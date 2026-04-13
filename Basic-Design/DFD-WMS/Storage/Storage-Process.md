[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">System Overview</span>

::: mermaid
flowchart TD
    SAP[SAP / Host]
    WMS[Warenavi - WMS]
    MC[MC - Java Middleware]
    DB[(Oracle DB
    DNxx / DMxx)]
    AGC[AGC]
    HW[STV / SRM / Conveyors
    Physical ASRS Shelves]

    SAP -->|GR Notification / Storage Plan| WMS
    WMS -->|TCP/IP Socket - ID messages| MC
    MC <-->|JDBC| DB
    MC -->|TCP/IP Socket - ID05 transport command| AGC
    AGC --> HW
:::

<hr>

#<span style="color:skyblue; font-weight:bold">Station Map</span>

##<span style="color:skyblue; font-weight:bold">OP Area</span>
| Station | Description |
|---------|-------------|
| 1101-1105 | Robot Input Stations (palletizing input) |
| 1111-1115 | Palletizing Stations (robot palletizes cartons) |
| 1210 | Empty Pallet Buffer |
| 1220 | Destacker |
| 1303 | QC / Reject Station |
| 7101-7106 | OP ASRS Inbound BCR - FGW2 Tempering (wh_station=9100) |
| 7107-7110 | OP ASRS Inbound BCR - FGW1 Ambient (wh_station=9200) |
| SRM 9001-9006 | FGW2 Tempering (single deep) |
| SRM 9007-9010 | FGW1 Ambient (double deep) |

##<span style="color:skyblue; font-weight:bold">HP Area</span>
| Station | Description |
|---------|-------------|
| 1106 | PM (Packaging Material) Inbound |
| 1301 | FG Storage/Retrieval 1 (bi-directional InOut) |
| 1302 | FG Storage/Retrieval 2 (bi-directional InOut) |
| 7207-7210 | HP ASRS Inbound BCR - FGW1 Ambient (wh_station=9200) |
| 7211-7214 | HP ASRS Inbound BCR - PM Ambient (wh_station=9200) |
| SRM 9007-9010 | FGW1 Ambient (shared with OP) |
| SRM 9011-9014 | PM Ambient (double deep) |

##<span style="color:skyblue; font-weight:bold">Soft Zone to Aisle Mapping</span>
| Soft Zone | Description | Aisles | SRM | BCR |
|-----------|-------------|--------|-----|-----|
| 001 | FG Tempering | 01-06 | 9001-9006 | 7101-7106 |
| 002 | FG Ambient | 07-10 | 9007-9010 | 7107-7110 / 7207-7210 |
| 003 | PM Ambient | 11-14 | 9011-9014 | 7211-7214 |
| 004 | Empty Pallet | 07-10 | 9007-9010 | 7107-7110 / 7207-7210 |
| 005 | ZNFP Product | 01-10 | 9001-9010 | 7101-7110 / 7207-7210 |

**DMSoftZonePriority fallback order:**
| Primary Zone | Fallback |
|--------------|----------|
| 002 (FG Ambient) | 004 -> 003 |
| 004 (Empty Pallet) | 002 |
| 005 (ZNFP Product) | 001 -> 002 |

<hr>

#<span style="color:skyblue; font-weight:bold">Storage Flows</span>

**Abbreviation:**
- **RECP** : DNRECEIVINGPLAN
- **STRP** : DNSTORAGEPLAN
- **PLLT** : DNPALLET
- **CRYI** : DNCARRYINFO
- **STCK** : DNSTOCK
- **WRKI** : DNWORKINFO
- **HSTS** : DNHOSTSEND
- **ARVL** : DNARRIVAL
- **WRHS** : DMWAREHOUSE
- **SHLF** : DMSHELF
- **COLI** : DNCOLLECTINFO
- **INOUT** : DNINOUTRESULT

* **Operation Name**
  - **I** : INSERT
  - **U** : UPDATE
  - **D** : DELETE
  - **S** : SELECT

<hr>

## 1. Palletization Storage Flow (110x -> 111x -> 710x -> SRM)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.PalletizeRobotStationOperator (110x) &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.PalletizeStorageStationOperator (111x) &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.AsrsInboundStationOperator (710x) &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutoStorageScheduler (110x) &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender (111x) &nbsp;</span>

Only **ZFNP** item_type (soft_zone 005). User chooses plan_area_no 9100 or 9200.

| Step | Action |RECP|STRP|PLLT|CRYI|STCK|WRKI|HSTS|ARVL|SHLF|COLI|INOUT|
|------|--------|----|----|----|----|----|----|----|----|----|----|-----|
| 1 | ID26 at 110x - AutoStorageScheduler (plan found) | U | | I | I | I | | | U | | | |
| 2 | ID26 at 111x - PalletizeStorageStationOperator (pass-through) | | | | | | | | I | | | |
| 3 | StorageSender at 111x (reuses carry from step 1) | | I | U | U | D,I | I | | | | I | |
| 4 | ID26 at 710x - AsrsInboundStationOperator (FINAL BCR) | | | | U | | | | | | | |
| 5 | StorageSender at 710x (bin selection) | | | | | | | | | U | | |
| 6 | ID33 (storage complete) | | U | U | D | U | U | I | | U | | I |

###<span style="color:skyblue; font-weight:bold">Step 1 - ID26 at 110x (Robot Input)</span>

Pallet arrives at robot input station (1101-1105) with DUMMY mckey. `PalletizeRobotStationOperator` registers DNArrival and calls `autoScheduleRequest()`.

`AutoStorageScheduler` polls DNArrival (status 0,1 only — not WAITING_LAST_PALLET):
- **No plan found** -> silent wait, DNArrival stays NOT_SCHEDULED
- **Plan found** -> No aisle selection at 110x (pallet doesn't know which aisle yet)

::: mermaid
flowchart LR
    A["getReceivingPlanForRobot
    (next_station_no, status 0,1)"] --> B{Plan found?}
    B -->|NO| C[Silent wait]
    B -->|YES| E["INSERT DNPallet
    DIRECT_PB + collect_key"]
    E --> F["INSERT DNStock
    DIRECT_PB"]
    F --> G["INSERT DNCarryInfo
    DIRECT_TRAVEL
    dest=111x, end=111x (next station)"]
    G --> H["UPDATE DNReceivingPlan
    status 0->1 (only if status=0)"]
    H --> I["UPDATE DNArrival
    SCHEDULED"]
    I --> J["ID05: 110x->111x"]
:::

###<span style="color:skyblue; font-weight:bold">Step 2 - ID26 at 111x (Palletizing Station - Pass-through)</span>

`PalletizeStorageStationOperator.arrival()`:
- `reject_factor == "00"` (normal) -> `registArrival()` + `carryRequest()` wakes StorageSender
- Robot palletizes cartons onto the pallet

###<span style="color:skyblue; font-weight:bold">Step 3 - StorageSender at 111x (Path 2 — Storage Plan + Aisle Selection)</span>

`StorageSender.processPalletizeStorage()` reuses the same carry/mckey from step 1:

::: mermaid
flowchart LR
    A["getPalletizingCompletion
    from DNArrival"] --> B{control_info?}
    B -->|"not 000/001"| R1["rejectCarryTo1303"]
    B -->|"000 or 001"| C["findReceivingPlan
    (status 0,1,2)"]
    C --> D{Plan found?}
    D -->|NO| R2["rejectCarryTo1303"]
    D -->|YES + 001 + status!=2| W["updateWaitReason=17
    hold pallet"]
    D -->|YES + 001 + status=2 + qty=0| R3["rejectCarryTo1303"]
    D -->|"YES (normal)"| E["deleteDirectPbStock
    (remove DIRECT_PB placeholder)"]
    E --> F["createStoragePlan
    plan_qty=batch_qty_ctrn_pl
    (or batch_last_pallet_qty if status=2)"]
    F --> G["createStock
    (real item from DNReceivingPlan)"]
    G --> H["createWorkInfo"]
    H --> I["updatePalletSoftZone"]
    I --> J["selectAisleAndUpdateCarry
    dest=710x (7101-7110)
    end=plan_area (9100/9200)"]
    J --> K["updateReceivingPlanProgress
    progress_qty += plan_qty
    (status->4 if last pallet)"]
    K --> L["ID05: 111x->710x"]
:::

###<span style="color:skyblue; font-weight:bold">Step 4 - ID26 at 710x (FINAL BCR)</span>

`AsrsInboundStationOperator.arrival()`:
- `wh_station == end_station` -> **FINAL BCR**
- UPDATE DNCarryInfo: `carry_flag` DIRECT_TRAVEL -> **STORAGE**, `dest` -> wh_station (e.g. 9100)

###<span style="color:skyblue; font-weight:bold">Step 5 - StorageSender (Bin Selection)</span>

`StorageSender` at 710x:
- `LocationManager.searchLocation()` -> finds empty bin (e.g. SHF001)
- UPDATE DMShelf status -> **RESERVED**
- Send **ID05** storage command to SRM

###<span style="color:skyblue; font-weight:bold">Step 6 - ID33 (Storage Completion)</span>

See [Storage Completion (ID33)](#Storage-Completion-ID33) section below.

<hr>

## 2. PM / FG Inbound Storage - Same Warehouse (1106/1301/1302 -> 720x -> SRM)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutoStorageScheduler &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.AsrsInboundStationOperator (720x) &nbsp;</span>

| Step | Action |STRP|PLLT|CRYI|STCK|WRKI|HSTS|ARVL|SHLF|COLI|INOUT|
|------|--------|----|----|----|----|----|----|----|----|----|----|
| 1 | ID26 at 1106/1301/1302 - AutoStorageScheduler | U | I | I | I | I | | U | | I | |
| 2 | ID26 at 720x - AsrsInboundStationOperator (FINAL BCR) | | | U | | | | | | | |
| 3 | StorageSender (bin selection) | | | | | | | | U | | |
| 4 | ID33 (storage complete) | U | U | D | U | U | I | | U | | I |

###<span style="color:skyblue; font-weight:bold">Step 1 - ID26 at 1106/1301/1302 (Inbound Station)</span>

Pallet placed at inbound station. `InOutStationOperator` / `StorageStationOperator` registers DNArrival and calls `autoScheduleRequest()`.

`AutoStorageScheduler` — DNStoragePlan already exists from screen input (NOT created here):

**Station-specific routing:**
| Station | Allowed Items | Destination | Notes |
|---------|--------------|-------------|-------|
| 1106 | ZPCK only | 7207-7210 | PM Inbound. No empty pallet, no Tempering |
| 1301 | ALL (ZFNP, ZPCK, EMP_PB) | 7207-7214 | PM direct to 721x, Ambient to 720x, Tempering via 720x intermediate |
| 1302 | ALL | 7207-7214 | Same as 1301 |
| 1303 | ZFNP, EMP_PB (no ZPCK) | 7101-7110 | QC/Reject station. plan_area 9100 or 9200 |

::: mermaid
flowchart LR
    A["existsCarryInfo?"] --> B{Exists?}
    B -->|YES| C["updateArrival + carryRequest
    continue"]
    B -->|NO| D["validateStoragePlan
    (zone + shelf check with
    DMSoftZonePriority fallback)"]
    D --> E["selectAisleAndGetBcrStation
    -> BCR reachable from source"]
    E --> F["INSERT DNPallet, DNCarryInfo
    DIRECT_TRAVEL
    dest=BCR, end=plan_area"]
    F --> G["INSERT DNStock + DNWorkInfo
    job_type from DNStoragePlan"]
    G --> H["UPDATE DNStoragePlan
    status 0->1, process_qty += plan_qty"]
    H --> I["UPDATE DNArrival SCHEDULED"]
    I --> J["ID05: source->BCR"]
:::

###<span style="color:skyblue; font-weight:bold">Step 2 - ID26 at 720x (FINAL BCR - Same Warehouse)</span>

`AsrsInboundStationOperator.arrival()`:
- `wh_station (9200) == end_station (9200)` -> **FINAL BCR**
- UPDATE DNCarryInfo: `carry_flag` -> **STORAGE**, `dest` -> wh_station (9200)

###<span style="color:skyblue; font-weight:bold">Steps 3-4</span>

Same as Palletization flow Steps 5-6 (StorageSender bin selection -> ID33 completion).

<hr>

## 3. FG Inbound Cross-Warehouse Storage (1301/1302 -> 720x -> 710x -> SRM)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutoStorageScheduler &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.AsrsInboundStationOperator (720x + 710x) &nbsp;</span>

Example: 1301 -> FGW2 Tempering (plan_area = 9100)

| Step | Action |STRP|PLLT|CRYI|STCK|WRKI|HSTS|ARVL|SHLF|COLI|INOUT|
|------|--------|----|----|----|----|----|----|----|----|----|----|
| 1 | ID26 at 1301 - AutoStorageScheduler (cross-wh) | U | I | I | I | I | | U | | | |
| 2 | ID26 at 720x - AsrsInboundStationOperator (INTERMEDIATE) | | | U | | | | | | I | |
| 3 | ID26 at 710x - AsrsInboundStationOperator (FINAL BCR) | | | U | | | | | | | |
| 4 | StorageSender (bin selection) | | | | | | | | U | | |
| 5 | ID33 (storage complete) | U | U | D | U | U | I | | U | | I |

###<span style="color:skyblue; font-weight:bold">Step 1 - ID26 at 1301 (Cross-Warehouse Routing)</span>

`AutoStorageScheduler` routing for HP inbound stations (1301/1302):

::: mermaid
flowchart LR
    A["selectAisleAndGetBcrStation()
    Try direct BCR first"] --> B{Direct BCR
    reachable?}
    B -->|YES: PM item zone 003
    route 1301->7211 exists| C["dest=721x (direct)
    end=9200"]
    B -->|NO: Ambient/Tempering
    no direct BCR route| D["selectLeastBusyIntermediateStation
    720x intermediate"]
    D --> E["Count active DNCarryInfo
    per 720x, select min"]
    E --> F["dest=720x (e.g. 7208)
    end=wh_station (9100)"]
    C --> G[INSERT DNPallet, DNStock, DNWorkInfo]
    F --> G
    G --> H[UPDATE DNArrival SCHEDULED]
    H --> I["ID05: source->dest"]
:::

**Key:** PM items (zone 003) from 1301/1302 route DIRECTLY to 721x BCR (7211-7214) because 720x (7207-7210) can only physically reach aisles 9007-9010, not PM aisles 9011-9014.

###<span style="color:skyblue; font-weight:bold">Step 2 - ID26 at 720x (INTERMEDIATE Station)</span>

`AsrsInboundStationOperator.arrival()`:
- `wh_station (9200) != end_station (9100)` -> **INTERMEDIATE**

::: mermaid
flowchart LR
    A["wh_station (9200) !=
    end_station (9100)
    -> INTERMEDIATE"] --> B[selectTargetAisleStation]
    B --> C["getReachableBcrStations(720x, target_wh)
    DMRouteId: 720x -> 710x routes"]
    C --> D["mapBcrToAisleStations
    DMAisle: BCR -> Aisle"]
    D --> E["AisleShelfDecider.decideAisle
    (filtered reachable aisles only)"]
    E --> F["getReachableBcrForAisle
    -> BCR 710x (e.g. 7103)"]
    F --> G["UPDATE DNCarryInfo
    dest=7103, end stays 9100"]
    G --> H["ID05: 7208->7103"]
:::

###<span style="color:skyblue; font-weight:bold">Step 3 - ID26 at 710x (FINAL BCR)</span>

`AsrsInboundStationOperator.arrival()`:
- `wh_station (9100) == end_station (9100)` -> **FINAL BCR**
- UPDATE DNCarryInfo: `carry_flag` -> **STORAGE**, `dest` -> 9100

###<span style="color:skyblue; font-weight:bold">Steps 4-5</span>

Same as Palletization flow Steps 5-6 (StorageSender bin selection -> ID33 completion).

<hr>

#<span style="color:skyblue; font-weight:bold">Reject Flows</span>

## 1. BCR Fail at 110x (No-Read / Multi-Read)

::: mermaid
flowchart LR
    A["ID26 at 110x
    BCR = '?'"] --> B[AutoStorageScheduler
    checkBcr FAIL]
    B --> C["createCarryDirect
    dest=111x, end=1303
    reject_factor=NO_READ/MULTI_READ"]
    C --> D["ID05: 110x->111x"]
    D --> E["PalletizeStorageStationOperator
    reject_factor != '00'
    updateDestToEndStation
    dest=1303"]
    E --> F["ID05: 111x->1303"]
    F --> G["LoadRemover.remove()
    Pallet sits at 1303"]
:::

> **Note:** No DNPallet/DNStock exist at 110x if no plan was found - nothing to clean up.

## 2. Invalid Control Info at 111x (control_info not 000/001)

Handled by `StorageSender.processPalletizeStorage()`:

::: mermaid
flowchart LR
    A["StorageSender at 111x
    control_info not 000/001"] --> B["rejectCarryTo1303
    dest=1303, end=1303"]
    B --> C["ID05: 111x->1303"]
:::

## 3. No Receiving Plan at 111x

::: mermaid
flowchart LR
    A["StorageSender at 111x
    No DNReceivingPlan found"] --> B["rejectCarryTo1303
    dest=1303, end=1303"]
    B --> C["ID05: 111x->1303"]
:::

## 4. Force Completion at 111x (control_info=001)

Handled by `StorageSender.processPalletizeStorage()`:

| DNReceivingPlan Status | Condition | Action |
|------------------------|-----------|--------|
| status=0 or 1 | control_info=001 | `updateWaitReason=17` (batch end incomplete), hold pallet, lamp 17 ON |
| | | User does batch end -> DNReceivingPlan status=2, AGC resends with control_info=001 |
| status=2 (Last Pallet Wait), qty > 0 | Last pallet | `createStoragePlan(plan_qty=batch_last_pallet_qty)`, normal storage flow, UPDATE DNReceivingPlan status=4 (COMPLETION) |
| status=2 (Last Pallet Wait), qty = 0 | Zero qty | `rejectCarryTo1303` -> 1303 |

<hr>

#<span style="color:skyblue; font-weight:bold">Aisle Selection Logic</span>

##<span style="color:skyblue; font-weight:bold">Aisle Decision Pattern by Warehouse</span>

The aisle selector is chosen based on `DMWareHouse.aisle_decision_pattern`:

| Warehouse | Pattern | Value | Selector Class | Reason |
|-----------|---------|-------|----------------|--------|
| 9100 (FGW2 Tempering) | Aisle Distributed | **3** | `DisperseAisleSelector` | Single deep racks - distribute evenly across aisles |
| 9200 (FGW1 Ambient / PM) | Aisle Aggregation (WN) | **4** | `WNCollectAisleSelector` | Double deep racks - aggregate same batch into same aisle via DNCollectInfo |

**All available patterns:**
| Value | Name | Selector | Description |
|-------|------|----------|-------------|
| 0 | OFF | `ConnectedAisleSelector` | Default / no special logic |
| 1 | LOCAL_BRANCH | (local branch) | Local branch routing |
| 2 | PATTERN_BRANCH | `PatternAisleSelector` | Pattern-based branch routing |
| 3 | AISLE_DISTRIBUTED | `DisperseAisleSelector` | Distribute pallets evenly across aisles |
| 4 | AISLE_AGGREGATION_WN | `WNCollectAisleSelector` | Aggregate same batch into same aisle (WN decides) |
| 5 | AISLE_AGGREGATION_HOST | `HostCollectAisleSelector` | Aggregate same batch into same aisle (Host decides) |

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.decide.AbstractShelfDecider &nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.decide.AisleShelfDecider &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">9100 — DisperseAisleSelector (Pattern 3, Single Deep)</span>

For single deep racks, pallets are distributed evenly across aisles to maximize throughput.

::: mermaid
flowchart TD
    A["AisleShelfDecider.decideAisle
    Pallet, WareHouse (9100)"] --> B["DisperseAisleSelector"]
    B --> C["Distribute pallets evenly
    across available aisles 9001-9006"]
    C --> D["SoftZoneSelector
    primary zone from DMItem
    fallback via DMSoftZonePriority"]
    D --> E["checkStorageAisle(aisle, soft_zone)
    DMShelf has empty locations?"]
    E --> F["determin()
    Update DMWareHouse.last_used_station_no"]
:::

##<span style="color:skyblue; font-weight:bold">9200 — WNCollectAisleSelector (Pattern 4, Double Deep)</span>

For double deep racks, same batch pallets are aggregated into the same aisle to optimize double-deep placement and retrieval efficiency.

::: mermaid
flowchart TD
    A["AisleShelfDecider.decideAisle
    Pallet, WareHouse (9200)"] --> B["WNCollectAisleSelector"]
    B --> C{DNCollectInfo
    aisle_collect_key exists?}
    C -->|YES| D["Use same aisle
    (batch grouping)"]
    C -->|NO| E["Balance query:
    ORDER BY ALL_BATCH_COUNT ASC,
    ALL_STOCK_COUNT ASC,
    AISLE_STATION_NO ASC
    (counts in-transit + stored pallets)"]
    D --> F["SoftZoneSelector
    primary zone from DMItem
    fallback via DMSoftZonePriority"]
    E --> F
    F --> G["checkStorageAisle(aisle, soft_zone)
    DMShelf has empty locations?"]
    G --> H["determin()
    Write/update DNCollectInfo: aisle_collect_key -> aisle_no
    Update DMWareHouse.last_used_station_no"]
:::

**Balance query counts (WNCollectAisleSelector):**
| State | Description |
|-------|-------------|
| BC1/SC1 | Already stored (DNStock + DMShelf OCCUPIED) |
| BC2/SC2 | In transit main -> BCR (DNCarryInfo DIRECT_TRAVEL, dest=BCR) |
| BC3/SC3 | In transit BCR -> shelf (DNCarryInfo STORAGE, dest=shelf) |
| BC4/SC4 | Aisle decided not yet sent (DNStoragePlan status=0, DNCollectInfo set) |

##<span style="color:skyblue; font-weight:bold">Cross-Warehouse Intermediate Selection (1301 -> FGW2)</span>

::: mermaid
flowchart TD
    A["selectLeastBusyIntermediateStation()"] --> B["Query DMRouteId:
    source -> HP_INTERMEDIATE_STATION_NOS"]
    B --> C["Filter: DMStation
    online, not suspended, connected"]
    C --> D["Count DNCarryInfo
    active per 720x station"]
    D --> E["Select min count
    tiebreak: station_no ASC
    -> 720x intermediate (e.g. 7208)"]
:::

##<span style="color:skyblue; font-weight:bold">At 720x - Reachable Aisle Selection</span>

::: mermaid
flowchart TD
    A["selectTargetAisleStation()
    at 720x INTERMEDIATE"] --> B["getReachableBcrStations(720x, target_wh)
    DMRouteId: 720x -> 710x
    Filter: online, max_instruction not full"]
    B --> C["mapBcrToAisleStations()
    DMAisle.bcr_station_no -> station_no"]
    C --> D["AisleShelfDecider.decideAisle
    (restricted to reachable aisles only)"]
    D --> E{Aisle reachable
    from this 720x?}
    E -->|YES| F["getReachableBcrForAisle
    -> target BCR 710x"]
    E -->|NO| G["Return null
    hold, retry next cycle"]
:::

> **Important:** `decideAisle()` considers ALL aisles in the warehouse. After calling it, validate the returned aisle is reachable from the current 720x station. If not reachable, return null and hold for retry.

<hr>

#<span style="color:skyblue; font-weight:bold">Storage Completion (ID33)</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.CarryCompleteOperator &nbsp;</span>

`CarryCompleteOperator.completeStorageWorkAndUpdateStock(ci)` handles all post-storage updates:

| Order | Table | Operation | Details |
|-------|-------|-----------|---------|
| 1 | DNWorkInfo | UPDATE | status_flag -> **4** (COMPLETION), result_location_no = actual bin, result_qty = stored qty |
| 2 | DNHostSend | INSERT | job_type from DNWorkInfo (02 or 22), result_location_no = actual bin, report_flag = **0** (pending SAP send) |
| 3 | DNStock | UPDATE | location_no = actual bin, storage_date = now, stock_qty += qty, plan_qty -> 0 |
| 4 | DMShelf | UPDATE | status_flag -> **1** (OCCUPIED) |
| 5 | DNPallet | UPDATE | current_station_no = shelf, status_flag = STORED |
| 6 | DNStockHistory | INSERT | Audit trail of stock change |
| 7 | DNStoragePlan | UPDATE | status_flag -> **4** (COMPLETE) via `PlanControllerFactory.makeInstance(work.job_type)` |
| 8 | DNInOutResult | INSERT | Audit trail of storage operation |
| 9 | DNCarryInfo | DELETE | Carry instruction removed |
| 10 | ShelfMonitor | (next cycle) | Count empty DMShelf -> ID54 lamp update if threshold changed |

###<span style="color:skyblue; font-weight:bold">Verification Checklist</span>
- `DNHostSend.job_type` = 02 (Storage) or 22 (Unplanned) - must come from DNStoragePlan, never hardcoded
- `DNHostSend.result_location_no` = actual bin address
- `DNStock.location_no` = actual bin address
- `DNStoragePlan.status_flag` = 4

<hr>

#<span style="color:skyblue; font-weight:bold">Location Full Lamp (ShelfMonitor)</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.ShelfMonitor &nbsp;</span>

`ShelfMonitor` runs every `SHELF_MONITOR_SLEEP_SEC`:

::: mermaid
flowchart TD
    A[ShelfMonitor polls] --> B["For each DMLocationFullLamp record:
    Count empty DMShelf
    WHERE wh_station_no = configured wh
    AND status = EMPTY
    AND aisle not DISCONNECTED
    AND prohibition = OK, access_ng = OK
    minus reserved_qty"]
    B --> C{Compare against
    DNSystemKVs thresholds}
    C -->|"emptyCount = 0"| D["LAMP_FULL ON
    WARNING OFF"]
    C -->|"0 < count < warningQty (50)"| E["WARNING ON
    LAMP_FULL OFF"]
    C -->|"warningQty <= count <= lightOffQty (60)"| F["Hysteresis
    stays WARNING if was FULL"]
    C -->|"count > lightOffQty (60)"| G["Both OFF"]
    D --> H{Status changed?}
    E --> H
    F --> H
    G --> H
    H -->|YES| I["Send ID54
    UPDATE DMLocationFullLamp.status_flag"]
    H -->|NO| J[No action]
:::

| Threshold | Source | Default |
|-----------|--------|---------|
| Warning ON | `free_shelf_warning_num_agc5` | 50 (WARNING when empty < 50) |
| Both OFF | `free_shelf_warning_release_num_agc5` | 60 (both OFF when empty > 60) |

> **Important:** MC code does **NOT** send `LAMP_FULL` or `LAMP_FULL_WARNING` directly from storage code. `ShelfMonitor` is the **ONLY** source for location full lamp control.

<hr>

#<span style="color:skyblue; font-weight:bold">Id35 — Removal/Error Recovery</span>

| Scenario | sch_carry_key | Records exist | Id35 action |
|----------|--------------|---------------|-------------|
| 110x, no plan found | blank | none | DELETE DNArrival only |
| 110x, plan found | CK001 | DIRECT_PB pallet+stock | StorageTrackingDeleter |
| 111x, carry created | CK002 | real item pallet+stock | StorageTrackingDeleter + DNStoragePlan status -> 0 |
| 710x / 720x | real key | all records | StorageTrackingDeleter |

> **Note:** `DNReceivingPlan` status is **never** rolled back in any scenario.

<hr>

#<span style="color:skyblue; font-weight:bold">Key Tables Reference</span>

| Table | Description | Key Status Values |
|-------|-------------|-------------------|
| DNReceivingPlan | Palletization batch plan (from Warenavi screen) | 0=Not started, 1=Working, 2=Last pallet wait, 4=Complete |
| DNStoragePlan | Storage plan per pallet (from system or user) | 0=Unstart, 1=Working, 4=Complete. job_type: 02=Storage, 22=Unplanned |
| DNArrival | Pallet arrival record per station | sch_flag: 0=Not scheduled, 1=Scheduled |
| DNCarryInfo | Transport command record | carry_flag: 1=Storage, 3=Direct Travel. cmd_status: 1=Start, 2=Arrival, 3=Instruction, 4=Wait response |
| DNPallet | Pallet tracking record | status: DIRECT_PB, STORED |
| DNStock | Inventory record (location, qty, status) | stock_status: UU=Unrestricted, QI=Quality Inspection, BS=Block |
| DNWorkInfo | Work instruction linked to carry + stock | status_flag: 0=Unstart, 1=Working, 4=Complete |
| DNCollectInfo | Aisle grouping for batch | aisle_collect_key -> aisle_no |
| DNHostSend | Result notification to SAP/Host | report_flag: 0=pending, 1=sent |
| DNInOutResult | Audit trail of storage/retrieval operations | |
| DMShelf | Physical shelf location | status: 0=Empty, 1=Occupied, 2=Reserved |
| DMLocationFullLamp | Tower light configuration per warehouse | |
| DMWareHouse | Warehouse settings | aisle_decision_pattern, zone_manage_type |
| DNSystemKVs | System parameters | lamp thresholds etc. |

<hr>

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Palletize Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/868/0.-Palletize-Start)
- [Batch End](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End)
- [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)
