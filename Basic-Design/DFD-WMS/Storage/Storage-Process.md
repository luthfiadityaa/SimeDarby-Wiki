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

**DMSoftZonePriority fallback order:**
| Primary Zone | Fallback |
|--------------|----------|
| 002 (FG Ambient) | 004 -> 003 |
| 004 (Empty Pallet) | 002 |

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
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutoStorageScheduler &nbsp;</span>

| Step | Action |RECP|STRP|PLLT|CRYI|STCK|WRKI|HSTS|ARVL|SHLF|COLI|INOUT|
|------|--------|----|----|----|----|----|----|----|----|----|----|-----|
| 1 | ID26 at 110x - AutoStorageScheduler (plan found) | U | | I | I | I | | | U | | | |
| 2 | ID26 at 111x - PalletizeStorageStationOperator (pass-through) | | | | D | | | | D,I | | | |
| 3 | ID26 at 111x - AutoStorageScheduler (storage carry) | | U | I | I | I | I | | U | | I | |
| 4 | ID26 at 710x - AsrsInboundStationOperator (FINAL BCR) | | | | U | | | | | | | |
| 5 | StorageSender (bin selection) | | | | | | | | | U | | |
| 6 | ID33 (storage complete) | | U | U | D | U | U | I | | U | | I |

###<span style="color:skyblue; font-weight:bold">Step 1 - ID26 at 110x (Robot Input)</span>

Pallet arrives at robot input station (1101-1105) with DUMMY mckey. `PalletizeRobotStationOperator` registers DNArrival and calls `autoScheduleRequest()`.

`AutoStorageScheduler` polls DNArrival:
- **No plan found** -> silent wait, DNArrival stays NOT_SCHEDULED
- **Plan found** ->

::: mermaid
flowchart LR
    A[getReceivingPlan
    next_station_no] --> B{Plan found?}
    B -->|NO| C[Silent wait]
    B -->|YES| D[selectAisleAndGetBcrStation
    -> BCR 710x]
    D --> E[INSERT DNPallet
    DIRECT_PB + collect_key]
    E --> F[INSERT DNStock
    DIRECT_PB]
    F --> G[INSERT DNCarryInfo
    DIRECT_TRAVEL
    dest=111x, end=710x]
    G --> H[UPDATE DNReceivingPlan
    status 0->1]
    H --> I[UPDATE DNArrival
    SCHEDULED]
    I --> J[ID05: 110x->111x]
:::

###<span style="color:skyblue; font-weight:bold">Step 2 - ID26 at 111x (Palletizing Station - Pass-through)</span>

`PalletizeStorageStationOperator.arrival()`:
- `reject_factor == "00"` (normal) -> `LoadRemover.remove()` cleans direct carry, re-registers as DUMMY DNArrival
- Robot palletizes cartons onto the pallet

###<span style="color:skyblue; font-weight:bold">Step 3 - ID26 at 111x (AutoStorageScheduler - Storage Carry)</span>

After palletizing, AGC sends ID26 with DUMMY, `control_info='0'`.

`AutoStorageScheduler`:

::: mermaid
flowchart LR
    A[checkControlInfo] --> B{control_info?}
    B -->|0: Normal| C[insertStoragePlan]
    C --> D[selectAisleAndGetBcrStation
    -> BCR 710x]
    D --> E[INSERT DNPallet - real item]
    E --> F[INSERT DNCarryInfo
    DIRECT_TRAVEL
    dest=710x, end=wh_station]
    F --> G[INSERT DNStock + DNWorkInfo
    job_type from DNStoragePlan]
    G --> H[UPDATE DNStoragePlan
    status 0->1]
    H --> I[UPDATE DNArrival
    SCHEDULED]
    I --> J[ID05: 111x->710x]
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

`AutoStorageScheduler`:

::: mermaid
flowchart LR
    A[existsCarryInfo?] --> B{Exists?}
    B -->|YES| C[updateArrival + carryRequest
    continue]
    B -->|NO| D[insertStoragePlan]
    D --> E[selectAisleAndGetBcrStation
    -> BCR 720x]
    E --> F[INSERT DNPallet, DNCarryInfo
    DIRECT_TRAVEL
    dest=720x, end=wh_station]
    F --> G[INSERT DNStock + DNWorkInfo
    job_type from DNStoragePlan]
    G --> H[UPDATE DNStoragePlan
    status 0->1]
    H --> I[UPDATE DNArrival
    SCHEDULED]
    I --> J[ID05: source->720x]
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

`AutoStorageScheduler` detects `isCrossWarehouseRoute(planAreaNo)` = YES:

::: mermaid
flowchart LR
    A[isCrossWarehouseRoute?
    YES] --> B[selectLeastBusyIntermediateStation]
    B --> C[Count active DNCarryInfo
    per 720x station
    Select min count]
    C --> D["INSERT DNCarryInfo
    DIRECT_TRAVEL
    dest=720x (e.g. 7208)
    end=wh_station (9100)"]
    D --> E[INSERT DNPallet, DNStock, DNWorkInfo]
    E --> F[UPDATE DNArrival SCHEDULED]
    F --> G["ID05: 1301->7208"]
:::

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

## 2. Error Completion at 111x (control_info=2)

::: mermaid
flowchart LR
    A["ID26 at 111x
    control_info='2'"] --> B["checkControlInfo
    -> PALLETIZING_ERROR_COMPLETION
    (called BEFORE insertStoragePlan)"]
    B --> C["createCarryDirect
    dest=1303, end=1303
    reject_factor=ERROR_COMPLETION"]
    C --> D["ID05: 111x->1303"]
    D --> E["LoadRemover.remove()"]
:::

> **Note:** `checkControlInfo()` is called **before** `insertStoragePlan()` at 111x. No DNPallet/DNStock exist yet - no orphan cleanup needed.

## 3. Force Completion at 111x (control_info=1)

| DNReceivingPlan Status | Condition | Action |
|------------------------|-----------|--------|
| status=1 (Working) | control_info=1 | Lamp 17 ON -> user inputs last qty |
| | | User submits -> DNReceivingPlan status=2, AGC resends |
| status=2 (Last Pallet Wait), qty > 0 | Last pallet | insertStoragePlan (last pallet), createCarryStorage -> normal flow, UPDATE DNReceivingPlan status=4 |
| status=2 (Last Pallet Wait), qty = 0 | Empty pallet | No DNStoragePlan, createCarryDirect to 1303 (empty pallet disposal) |

<hr>

#<span style="color:skyblue; font-weight:bold">Aisle Selection Logic</span>

##<span style="color:skyblue; font-weight:bold">Same-Warehouse Aisle Selection</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.location.AisleShelfDecider &nbsp;</span>

::: mermaid
flowchart TD
    A[AisleShelfDecider.decideAisle
    Pallet, WareHouse] --> B[WNCollectAisleSelector]
    B --> C{DNCollectInfo
    aisle_collect_key exists?}
    C -->|YES| D[Use same aisle
    batch grouping]
    C -->|NO| E["Balance query:
    ORDER BY ALL_BATCH_COUNT ASC,
    ALL_STOCK_COUNT ASC,
    AISLE_STATION_NO ASC"]
    D --> F[SoftZoneSelector
    primary zone from DMItem
    fallback via DMSoftZonePriority]
    E --> F
    F --> G["checkStorageAisle(aisle, soft_zone)
    DMShelf has empty locations?"]
    G --> H["determin()
    Write/update DNCollectInfo
    Update DMWareHouse.last_used_station_no"]
:::

**Balance query counts:**
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
