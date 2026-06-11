[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">TC-USS-C002 — Unplanned Storage Setting — Storage from HP Station</span>

| **Field**            | **Value**                                                                                                                                                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**     | TC-USS-C002-unplanned-storage-HP-Station                                                                                                                                                                                                                     |
| **Test Type**        | Combined Test (Screen Validation + DB Flow + AGC Control Flow)                                                                                                                                                                                               |
| **Feature**          | Unplanned Storage Setting                                                                                                                                                                                                                                    |
| **DFD Reference**    | [[SCREEN] Unplanned Storage](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/946/Unplanned-Storage)                                                                                                          |
| **Screen Class**     | `jp.co.daifuku.wms.web.display.storage.unplannedstorage.UnplannedStorageSCH`                                                                                                                                                                                 |
| **Job Type**         | Unplanned Storage — `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.NOPLAN_STORAGE` (value: `22`)                                                                                                                                                                        |
| **HP Stations**      | `ST1301` (FG Storage/Retrieval 1) · `ST1302` (FG Storage/Retrieval 2) · `ST1106` (PM/Packaging Material Inbound)                                                                                                                                             |
| **Soft Zone**        | Zone `002` (FG Ambient) → Aisles 9007–9010, `003` (PM Ambient) → Aisles 9011–9014, `005` (ZFNP) → fallback to zone 001/002                                                                                                                                  |
| **Target Warehouse** | `9200` (FGW1 Ambient / PM Ambient)                                                                                                                                                                                                                           |
| **Aisle Selector**   | `WNCollectAisleSelector` (Pattern 4, Double Deep — `DMWareHouse.aisle_decision_pattern = 4`)                                                                                                                                                                 |
| **AGC Flow**         | AGC Linkage Spec Flow **05** — Input Area Storage (Direct Transfer): BCRs and Load Size Detectors Are Used                                                                                                                                                   |
| **Physical Route (ZFNP / Ambient)** | `ST1301/1302` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `BCR 720x (7207–7210)` → `ID26` → `ID05` (**STORAGE**) → `SRM Bin (9007–9010)` → `ID33`                                                 |
| **Physical Route (ZPCK / PM)**      | `ST1301/1302 or ST1106` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `BCR 721x (7211–7214)` → `ID26` → `ID05` (**STORAGE**) → `SRM Bin (9011–9014)` → `ID33`                                      |
| **Physical Route (Cross-WH Tempering)** | `ST1301/1302` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `BCR 720x (7207–7210)` **[Intermediate]** → `ID26` → `ID05` (**DIRECT_TRAVEL**) → `BCR 710x (7101–7106)` → `ID05` (**STORAGE**) → `SRM Bin (9001–9006)` → `ID33` |
| **Related Stories**  | [US-5140](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5140) · [US-6184](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6184)                                                                           |
| **Tester**           | _____________                                                                                                                                                                                                                                                |
| **Test Date**        | _____________                                                                                                                                                                                                                                                |
| **Overall Status**   | [ ] Pass &nbsp;&nbsp;[ ] Fail                                                                                                                                                                                                                                |

---

## <span style="color:skyblue; font-weight:bold">Objective</span>

Verify that the **Unplanned Storage Setting** screen (`UnplannedStorageSCH`) creates a valid `DNSTORAGEPLAN` record (`JOB_TYPE = 22`) upon pressing **Set (F2)**, and that the subsequent automated control flow — `AutoStorageScheduler` → `AsrsInboundStationOperator` → `StorageSender` → AGC ID messages → `CarryCompleteOperator` — completes end-to-end for all HP station scenarios, with all transactional tables correctly updated.

HP station coverage includes three routing sub-scenarios:

| Sub-Scenario | Station     | Item Type | Target Area | Route                                   |
| ------------ | ----------- | --------- | ----------- | --------------------------------------- |
| **A**        | 1301 / 1302 | ZFNP (zone 002) | 9200 (FGW1 Ambient) | Direct → BCR 720x (FINAL BCR)  |
| **B**        | 1301 / 1302 / 1106 | ZPCK (zone 003) | 9200 (PM Ambient) | Direct → BCR 721x (FINAL BCR) |
| **C**        | 1301 / 1302 | ZFNP (zone 001/005) | 9100 (FGW2 Tempering) | Tier 2 → BCR 720x (Intermediate) → BCR 710x (FINAL BCR) |

Key assertions per design spec:
- `JOB_TYPE = 22` (NOPLAN_STORAGE) propagates from `DNSTORAGEPLAN` → `DNWORKINFO` → `DNHOSTSEND`.
- `validateStoragePlan()` rejects incompatible zone/station combinations **before** any carry is created.
- PM items (zone 003) from 1301/1302 route **directly** to BCR 721x (7211–7214) via Tier 1 — **not** through 720x intermediate.
- Station 1106 **only** allows zone 003 (PM/ZPCK). Zones 001, 002, 004 must be rejected.
- Cross-warehouse (1301/1302 → 9100 Tempering) uses Tier 2 `selectLeastBusyIntermediateStation()` → 720x intermediate → 710x final.
- `ShelfMonitor` — **not** storage code — is the **only** source of ID54 location-full lamp control.

---

## <span style="color:skyblue; font-weight:bold">Scope</span>

| Layer                | Scope                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Screen (SCH)**     | Input validation, `DNSTORAGEPLAN` INSERT with `JOB_TYPE = 22`                                                |
| **Scheduler**        | `AutoStorageScheduler` — `validateStoragePlan()`, aisle selection (Tier 1 / Tier 2), carry creation          |
| **Station Operator** | `AsrsInboundStationOperator` at BCR 720x (FINAL or INTERMEDIATE) and 721x (FINAL)                           |
| **Storage Sender**   | `StorageSender` — bin selection, `DMShelf` reservation, ID05 dispatch                                        |
| **AGC Linkage**      | ID26 (Arrival), ID05 (Transport Command), ID25 (Response), ID64 (Pick-up), ID33 (Completion), ID54 (Lamp)    |
| **Completion**       | `CarryCompleteOperator` — full post-ID33 table updates                                                       |
| **Lamp**             | `ShelfMonitor` — location-full lamp logic (ID54)                                                             |

---

## <span style="color:skyblue; font-weight:bold">System Architecture Reference</span>

```
SAP/User Screen
    │
    │  [Set F2] INSERT DNSTORAGEPLAN (STATUS=0, JOB_TYPE=22, ITEM=ZFNP/ZPCK, AREA=9200)
    ▼
AutoStorageScheduler  ◄── ID26 from AGC (pallet arrives at 130x / 1106)
    │  validateStoragePlan() → zone/station check
    │  selectAisleAndGetBcrStation() → Tier 1 direct BCR
    │  (fallback Tier 2: selectLeastBusyIntermediateStation() → 720x)
    │  INSERT DNPallet, DNStock, DNCarryInfo (DIRECT_TRAVEL, dest=BCR, end=plan_area)
    │  UPDATE DNStoragePlan STATUS→1, DNArrival SCHEDULED
    │  → ID05 (DIRECT_TRAVEL): source → BCR
    ▼
AGC/Conveyor transports pallet to BCR 720x or 721x
    │  → ID26 Arrival at BCR
    ▼
AsrsInboundStationOperator (720x / 721x)
    │  [Same-WH]: wh_station(9200) == end(9200) → FINAL BCR
    │            UPDATE carry_flag → STORAGE, dest → 9200
    │  [Cross-WH intermediate]: wh_station(9200) != end(9100) → INTERMEDIATE
    │            selectTargetAisleStation() → BCR 710x
    │            UPDATE dest → 710x, end stays 9100
    │  → ID26 at 710x → FINAL BCR → carry_flag STORAGE, dest=9100
    ▼
StorageSender (bin selection at FINAL BCR)
    │  LocationManager.searchLocation() → empty bin
    │  UPDATE DMShelf → RESERVED
    │  → ID05 (STORAGE): BCR → bin
    ▼
SRM physically stores pallet → ID33
    ▼
CarryCompleteOperator
    │  UPDATE DNWorkInfo, DNStock, DMShelf, DNPallet
    │  INSERT DNHostSend (job_type=22), DNStockHistory, DNInOutResult
    │  DELETE DNCarryInfo
    ▼
ShelfMonitor (next cycle) → ID54 if threshold crossed
```

---

## <span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

| No  | Pre-Condition                                                                                                          | How to Verify                                                                                                                    |
| --- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 1   | System (MC/WMS) is **ONLINE**. All background services active.                                                         | Check system status screen                                                                                                       |
| 2   | Daily database cleanup is **NOT** actively running                                                                     | `SELECT * FROM system_status WHERE daily_update_flag = '1'` → Expected: 0 rows                                                   |
| 3   | Stations 1301, 1302, and 1106 are **ONLINE** (`status_flag = 1`) and **NOT suspended** (`sendable = 1`)               | `SELECT station_no, status_flag, sendable FROM dmstation WHERE station_no IN ('1106','1301','1302')`                              |
| 4   | BCR stations 7207–7210 (720x) and 7211–7214 (721x) are online, `max_instruction = 2`, `class_name = AsrsInboundStationOperator` | `SELECT station_no, status_flag, max_instruction, sendable FROM dmstation WHERE station_no BETWEEN '7207' AND '7214'`    |
| 5   | AGC status is `ONLINE` (`status_flag = 1`) and `connection_flag = 1` (connected)                                      | `SELECT status_flag, connection_flag FROM dmagc`                                                                                 |
| 6   | `DMITEM` contains `ZFNPFG005` (zone 005) and `ZPCKPM003` (zone 003)                                                   | `SELECT item_code, soft_zone_id, item_type FROM dmitem WHERE item_code IN ('ZFNPFG005','ZPCKPM003')`                             |
| 7   | `DMWAREHOUSE`: WH 9200 has `aisle_decision_pattern = 4` (WNCollect), `zone_manage_type = 2`                           | `SELECT station_no, aisle_decision_pattern, zone_manage_type FROM dmwarehouse WHERE station_no = '9200'`                         |
| 8   | `DMAISLE` for aisles 9007–9014 has `status = 1` (NORMAL) and `double_deep_kind = 1` (DOUBLE)                          | `SELECT station_no, status, double_deep_kind FROM dmaisle WHERE station_no BETWEEN '9007' AND '9014'`                            |
| 9   | `DMSHELF` has empty shelves in Zone `002` (aisles 9007–9010) and Zone `003` (aisles 9011–9014)                        | `SELECT COUNT(*) FROM dmshelf WHERE status_flag = '0' AND soft_zone_id IN ('002','003') AND aisle_station_no BETWEEN '9007' AND '9014'` → Expected: > 0 |
| 10  | `DMROUTEID` routes exist: 1301/1302 → 720x (7207–7210), 1301/1302 → 721x (7211–7214), 1106 → 721x (7211–7214)        | `SELECT COUNT(*) FROM dmrouteid WHERE start_station_no IN ('1106','1301','1302') AND end_station_no LIKE '72%'` → Expected: ≥ 12 |
| 11  | No existing `DNSTORAGEPLAN` with `BCR_DATA = 'PLTTSS0002'` in status `0`, `1`, or `2`                                 | `SELECT COUNT(*) FROM dnstorageplan WHERE bcr_data = 'PLTTSS0002' AND status_flag IN ('0','1','2')` → Expected: 0                |

---

## <span style="color:skyblue; font-weight:bold">Test Data</span>

| **Field**                       | **Sub-Scenario A (ZFNP Ambient)** | **Sub-Scenario B (ZPCK PM)**   | **Sub-Scenario C (ZFNP Cross-WH Tempering)** |
| ------------------------------- | ---------------------------------- | ------------------------------ | --------------------------------------------- |
| **Pallet No**                   | `PLTTSS0002`                       | `PLTTSS0002`                   | `PLTTSS0002`                                  |
| **Material Code**               | `ZFNPFG005` (zone 005)             | `ZPCKPM003` (zone 003)         | `ZFNPFG001` (zone 001)                        |
| **Batch No**                    | `BATUSS0002`                       | (optional for ZPCK)            | `BATUSS0002`                                  |
| **Storage Qty**                 | `100`                              | `50`                           | `100`                                         |
| **Tempering Period**            | `7`                                | N/A (disabled)                 | `7`                                           |
| **Expiry Days**                 | `30`                               | N/A (disabled)                 | `30`                                          |
| **Storage Location From**       | `VT01`                             | `VT01`                         | `VT01`                                        |
| **To Location**                 | `FGW1`                             | `PACK`                         | `FGW2`                                        |
| **PLAN_AREA_NO (mapped)**       | `9002`                             | `9200`                         | `9001`                                        |
| **Source Station**              | `ST1301` or `ST1302`               | `ST1106`, `ST1301`, or `ST1302`| `ST1301` or `ST1302`                          |
| **Expected BCR**                | `7207–7210` (Tier 1, FINAL)        | `7211–7214` (Tier 1, FINAL)    | `7207–7210` (Tier 2 Intermediate) → `7101–7106` (FINAL) |
| **Expected SRM Aisles**         | `9007–9010`                        | `9011–9014`                    | `9001–9006`                                   |
| **Expected Soft Zone in Shelf** | Zone `002` (fallback from 005)     | Zone `003`                     | Zone `001` (fallback from 005→001 in WH 9100) |

---

## <span style="color:skyblue; font-weight:bold">Test Steps</span>

---

### Case 1: Station Validation Rules (🔴 Negative — Rejected Combinations)

**Purpose:** Confirm `validateStoragePlan()` in `AutoStorageScheduler` rejects incompatible zone/station combinations **before** any carry or pallet record is created.

| **No** | **Action**                                                                                                         | **Expected Result**                                                                                                                                           | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 1.1    | Create `DNSTORAGEPLAN` for `ZFNPFG005` (zone 005 → Ambient) at Station **1106**. Trigger `ID26` from Station 1106. | `validateStoragePlan()` **REJECTS** — Station 1106 only allows zone 003 (PM/ZPCK). No `DNCARRYINFO`, `DNPALLET`, or `DNSTOCK` created. Plan remains status 0. |                   | [ ] P <br>[ ] F |
| 1.2    | Create `DNSTORAGEPLAN` for EMP_PB (zone 004) at Station **1106**. Trigger `ID26` from Station 1106.                | `validateStoragePlan()` **REJECTS** — Zone 004 not allowed at 1106. No carry created.                                                                         |                   | [ ] P <br>[ ] F |
| 1.3    | Attempt to create `DNSTORAGEPLAN` for `ZPCKPM003` (zone 003 PM) at Station **1301**. Trigger `ID26`.               | `validateStoragePlan()` **PASSES** — 1301/1302 allow ALL zones including zone 003. Carry IS created (positive gate for zone 003 from 1301).                   |                   | [ ] P <br>[ ] F |

---

### Case 2: Sub-Scenario A — HP Station 1301/1302, ZFNP Ambient (Zone 002/005 → 9200) (🟢 Positive)

**Purpose:** Verify Tier 1 direct routing from HP station 1301/1302 to BCR 720x for FGW1 Ambient items.

#### Step 2.1 — Screen: DNSTORAGEPLAN INSERT

| **No** | **Action**                                                                                                             | **Expected Result**                                                                                                                     | **Actual Result** | **Status**      |
| ------ | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.1.1  | Enter test data for Sub-Scenario A (ZFNPFG005, BATUSS0002, Qty=100, Tempering=7, Expiry=30, FGW1). Click **Set (F2)**. | Success `6461009`. `DNSTORAGEPLAN` created: `JOB_TYPE = '22'`, `PLAN_AREA_NO = '9002'`, `STATUS_FLAG = '0'`. Run **SQL-01** to confirm. |                   | [ ] P <br>[ ] F |

#### Step 2.2 — AGC Linkage: ID26 Arrival at Station 1301/1302

| **No** | **Action**                                                                                                                                                              | **Expected Result**                                                                                                                                          | **Actual Result** | **Status**      |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 2.2.1  | Send **ID26 (Arrival Report)** from AGC for Station `1301` (or `1302`). `MC key = DUMMY (99999...)`, `BC data = PLTTSS0002`, `Load info = "1"`, `Control info = "000"`. | `AutoStorageScheduler` processes arrival. `validateStoragePlan()` passes for zone 002/005 at 1301. Aisle selector runs `WNCollectAisleSelector` for WH 9200. |                   | [ ] P <br>[ ] F |
| 2.2.2  | Query `DNCARRYINFO` after ID26.                                                                                                                                         | New carry record: `carry_flag = 3` (DIRECT_TRAVEL), `dest = 720x BCR (7207–7210)` (Tier 1 direct, route 1301→720x exists), `end = '9002'`. Run **SQL-02**.   |                   | [ ] P <br>[ ] F |
| 2.2.3  | Verify `DNSTORAGEPLAN.STATUS_FLAG` updated to `1` (NOWWORKING).                                                                                                         | `STATUS_FLAG = '1'`. `PROCESS_QTY` incremented by plan qty.                                                                                                  |                   | [ ] P <br>[ ] F |
| 2.2.4  | Verify `DNCOLLECTINFO` for batch grouping.                                                                                                                              | `DNCOLLECTINFO` record exists with `aisle_collect_key` mapping pallet to a target aisle in 9007–9010.                                                        |                   | [ ] P <br>[ ] F |

#### Step 2.3 — AGC Linkage: ID05 Transport Command (DIRECT_TRAVEL)

| **No** | **Action**                                                                  | **Expected Result**                                                                                                                           | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.3.1  | Verify MC sends **ID05** to AGC after `AutoStorageScheduler` creates carry. | ID05: `Transport class = "3"` (Direct transfer), `Source = 1301/1302`, `Dest = 720x BCR (7207–7210)`, `Location = All "0"` (not a shelf yet). |                   | [ ] P <br>[ ] F |
| 2.3.2  | AGC acknowledges with **ID25** (`Response section = "00"` — Normal).        | MC registers acknowledgement. No error.                                                                                                       |                   | [ ] P <br>[ ] F |

#### Step 2.4 — AGC Linkage: ID26 Arrival at BCR 720x (FINAL BCR)

| **No** | **Action**                                                                       | **Expected Result**                                                                                                                                                                     | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.4.1  | AGC transports pallet to BCR 720x. Send **ID26 (Arrival Report)** from BCR 720x. | `AsrsInboundStationOperator.arrival()`: `wh_station (9200) == end_station (9002)` (same warehouse) → **FINAL BCR**. `carry_flag` updated: DIRECT_TRAVEL → **STORAGE**. `dest` → `9200`. |                   | [ ] P <br>[ ] F |
| 2.4.2  | `StorageSender` at BCR 720x performs bin selection.                              | `LocationManager.searchLocation()` → empty bin in Zone 002, aisle 9007–9010. `DMShelf.status → RESERVED` (status 2). **SQL-03** verifies shelf reserved.                                |                   | [ ] P <br>[ ] F |

#### Step 2.5 — AGC Linkage: ID05 (STORAGE) and ID33 Completion

| **No** | **Action**                                                                                                 | **Expected Result**                                                                                                                   | **Actual Result** | **Status**      |
| ------ | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.5.1  | MC sends **ID05 (STORAGE)** to AGC after bin selection.                                                    | ID05: `Transport class = "1"` (Storage), `Source = 720x`, `Dest = aisle 9007–9010`, `Location = bin address`.                         |                   | [ ] P <br>[ ] F |
| 2.5.2  | SRM stores pallet. AGC sends **ID33 (Normal Completion)** (`Transport section = "1"`, `Completion = "0"`). | `CarryCompleteOperator` executes full completion. Run **SQL-04** to verify all post-ID33 table updates. `DNHOSTSEND.JOB_TYPE = '22'`. |                   | [ ] P <br>[ ] F |

---

### Case 3: Sub-Scenario B — HP Station 1106 / 1301 / 1302, ZPCK PM (Zone 003 → 9200) (🟢 Positive)

**Purpose:** Verify direct Tier 1 routing to BCR 721x (7211–7214) for ZPCK/PM items. Critical: PM items from 1301/1302 must use 721x **directly**, NOT via 720x intermediate.

#### Step 3.1 — Screen and Plan Creation

| **No** | **Action**                                                                                                                        | **Expected Result**                                                                                                     | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.1.1  | Enter test data Sub-Scenario B (ZPCKPM003, Qty=50, To Location: PACK). Confirm Tempering/Expiry are disabled. Click **Set (F2)**. | Success `6461009`. `DNSTORAGEPLAN`: `JOB_TYPE = '22'`, `PLAN_AREA_NO = '9200'` (mapped from PACK), `STATUS_FLAG = '0'`. |                   | [ ] P <br>[ ] F |

#### Step 3.2 — AGC Linkage from Station 1106 (PM Only)

| **No** | **Action**                                                                           | **Expected Result**                                                                                                                                                              | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.2.1  | Send **ID26** from Station **1106**. `BC data = PLTTSS0002`, `Control info = "000"`. | `validateStoragePlan()` passes (zone 003 is allowed at 1106). Tier 1: `selectAisleAndGetBcrStation()` → zone 003 → aisles 9011–9014 → BCR **7211–7214**. Route 1106→7211 exists. |                   | [ ] P <br>[ ] F |
| 3.2.2  | Verify `DNCARRYINFO` after ID26 at 1106.                                             | `carry_flag = 3` (DIRECT_TRAVEL), `dest = 721x BCR (7211–7214)`, `end = '9200'`. **720x (7207–7210) must NOT be the dest** — PM from 1106 routes directly to 721x.               |                   | [ ] P <br>[ ] F |

#### Step 3.3 — AGC Linkage from Station 1301/1302 (PM — Tier 1 Direct to 721x)

| **No** | **Action**                                         | **Expected Result**                                                                                                                                                              | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.3.1  | Send **ID26** from Station **1301** for ZPCK item. | Tier 1 `selectAisleAndGetBcrStation()` → zone 003 → aisles 9011–9014 → BCR **7211–7214**. Route 1301→7211 exists. `DNCARRYINFO.dest = 721x`. **7207–7210 must NOT be assigned**. |                   | [ ] P <br>[ ] F |
| 3.3.2  | Verify ID05 sent by MC.                            | ID05: `Transport class = "3"`, `Source = 1301`, `Dest = 7211–7214` (PM BCR), NOT 7207–7210. Tier 2 (intermediate) is **NOT triggered** for PM items.                             |                   | [ ] P <br>[ ] F |

#### Step 3.4 — BCR 721x FINAL and Completion

| **No** | **Action**                                                                                           | **Expected Result**                                                                                                                 | **Actual Result** | **Status**      |
| ------ | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.4.1  | Pallet arrives at BCR 721x. Send **ID26** from BCR 721x.                                             | `AsrsInboundStationOperator`: `wh_station (9200) == end (9200)` → **FINAL BCR**. `carry_flag → STORAGE`, `dest → 9200`.             |                   | [ ] P <br>[ ] F |
| 3.4.2  | `StorageSender` at 721x selects bin. MC sends **ID05 (STORAGE)**. SRM completes. AGC sends **ID33**. | Bin reserved in Zone **003**, aisle 9011–9014. ID33 triggers `CarryCompleteOperator`. `DNHOSTSEND.JOB_TYPE = '22'`. Run **SQL-04**. |                   | [ ] P <br>[ ] F |

---

### Case 4: Sub-Scenario C — HP Station 1301/1302, ZFNP Cross-Warehouse Tempering (Zone 001 → 9100) (🟢 Positive)

**Purpose:** Verify Tier 2 intermediate routing via BCR 720x → BCR 710x when HP stations target WH 9100 (FGW2 Tempering).

#### Step 4.1 — Plan Creation and Initial ID26

| **No** | **Action**                                                                                                        | **Expected Result**                                                                                                                                                                             | **Actual Result** | **Status**      |
| ------ | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 4.1.1  | Enter Sub-Scenario C data (ZFNPFG001, BATUSS0002, Tempering=7, Expiry=30, To Location: FGW2). Click **Set (F2)**. | `DNSTORAGEPLAN` created: `JOB_TYPE = '22'`, `PLAN_AREA_NO = '9001'` (FGW2 = 9001), `STORAGE_LOCATION_TO = 'FGW2'`, `STATUS_FLAG = '0'`.                                                         |                   | [ ] P <br>[ ] F |
| 4.1.2  | Send **ID26** from Station **1301**. `BC data = PLTTSS0002`, `Control info = "000"`.                              | Tier 1 `selectAisleAndGetBcrStation()`: zone 001 → aisles 9001–9006 → no direct route 1301→710x → **returns null**. Tier 2 triggers: `selectLeastBusyIntermediateStation()` → 720x (7207–7210). |                   | [ ] P <br>[ ] F |
| 4.1.3  | Verify `DNCARRYINFO` after Tier 2 selection.                                                                      | `carry_flag = 3` (DIRECT_TRAVEL), `dest = 720x (7207–7210)` (intermediate), `end = '9001'` (cross-warehouse target area). **710x must NOT be the initial dest**.                                |                   | [ ] P <br>[ ] F |
| 4.1.4  | Verify ID05 Transport Command sent.                                                                               | ID05: `Transport class = "3"`, `Source = 1301`, `Dest = 7207–7210` (intermediate 720x). Route 1301→7208 (or other 720x) confirmed.                                                              |                   | [ ] P <br>[ ] F |

#### Step 4.2 — Intermediate BCR 720x Processing

| **No** | **Action**                                                            | **Expected Result**                                                                                                                                                                        | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 4.2.1  | Pallet arrives at intermediate BCR 720x. Send **ID26** from BCR 720x. | `AsrsInboundStationOperator`: `wh_station (9200) ≠ end (9001)` → **INTERMEDIATE**. `selectTargetAisleStation()` runs: routes 720x → 710x (7101–7106) available.                            |                   | [ ] P <br>[ ] F |
| 4.2.2  | Verify `DNCARRYINFO` updated after intermediate processing.           | `dest` updated to `710x BCR (7101–7106)`. `end` remains `'9001'` (unchanged). `carry_flag` remains `3` (DIRECT_TRAVEL — not yet STORAGE). `DNCOLLECTINFO` inserted for aisle in 9001–9006. |                   | [ ] P <br>[ ] F |
| 4.2.3  | MC sends new **ID05 (DIRECT_TRAVEL)** to AGC.                         | ID05: `Source = 720x`, `Dest = 710x BCR (7101–7106)`. Route 7208→7103 (or similar) confirmed.                                                                                              |                   | [ ] P <br>[ ] F |

#### Step 4.3 — FINAL BCR 710x and Completion

| **No** | **Action**                                                                                                     | **Expected Result**                                                                                                                         | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 4.3.1  | Pallet arrives at BCR 710x. Send **ID26** from BCR 710x.                                                       | `AsrsInboundStationOperator`: `wh_station (9100) == end (9001/9100)` → **FINAL BCR**. `carry_flag → STORAGE`, `dest → 9100`.                |                   | [ ] P <br>[ ] F |
| 4.3.2  | `StorageSender` at 710x selects Tempering bin. MC sends **ID05 (STORAGE)**. SRM completes. AGC sends **ID33**. | Bin reserved in Zone **001**, aisle 9001–9006 (FGW2 Tempering). `CarryCompleteOperator` runs. `DNHOSTSEND.JOB_TYPE = '22'`. Run **SQL-04**. |                   | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">Verification SQL</span>

### SQL-01 — DNSTORAGEPLAN Record (After Set F2)

```sql
SELECT
    PLAN_UKEY, STATUS_FLAG, JOB_TYPE, BCR_DATA, ITEM_CODE,
    PLAN_QTY, PLAN_LOT_NO, PLAN_AREA_NO,
    STORAGE_LOCATION_FROM, STORAGE_LOCATION_TO,
    BATCH_TEMPERING_PERIOD, BATCH_EXPIRY_DAYS, STOCK_STATUS,
    REGIST_PNAME
FROM dnstorageplan
WHERE bcr_data = 'PLTTSS0002'
ORDER BY regist_date DESC;
-- Expected per sub-scenario:
-- JOB_TYPE          = '22'                   (NOPLAN_STORAGE — NOT '02')
-- STATUS_FLAG       = '0'                    (UNSTART on create, '1' after ID26)
-- Scenario A: PLAN_AREA_NO = '9002', STORAGE_LOCATION_TO = 'FGW1'
-- Scenario B: PLAN_AREA_NO = '9200', STORAGE_LOCATION_TO = 'PACK'
-- Scenario C: PLAN_AREA_NO = '9001', STORAGE_LOCATION_TO = 'FGW2'
```

### SQL-02 — DNCARRYINFO After AutoStorageScheduler

```sql
SELECT
    CARRY_KEY, CARRY_FLAG, SOURCE_STATION_NO,
    DEST_STATION_NO, END_STATION_NO, CMD_STATUS
FROM dncarryinfo
WHERE pallet_id = (SELECT pallet_id FROM dnpallet WHERE bcr_data = 'PLTTSS0002');
-- Scenario A: carry_flag=3, dest=720x (7207-7210), end='9002'
-- Scenario B: carry_flag=3, dest=721x (7211-7214), end='9200'
-- Scenario C: carry_flag=3, dest=720x (7207-7210), end='9001' (intermediate)
```

### SQL-03 — DMShelf Reservation Check

```sql
SELECT
    s.shelf_no, s.status_flag, s.soft_zone_id, s.aisle_station_no
FROM dmshelf s
WHERE s.status_flag = '2'  -- RESERVED
  AND s.aisle_station_no BETWEEN '9007' AND '9014'  -- adjust per scenario
ORDER BY s.aisle_station_no;
-- Expected: 1 row RESERVED for the pallet's assigned aisle
```

### SQL-04 — Full Post-ID33 Completion Verification

```sql
-- 1. Check storage plan completed
SELECT status_flag FROM dnstorageplan
WHERE bcr_data = 'PLTTSS0002';
-- Expected: '4' (COMPLETE)

-- 2. Check stock record updated
SELECT location_no, stock_qty, plan_qty, stock_status
FROM dnstock
WHERE bcr_data = 'PLTTSS0002';
-- Expected: location_no=actual bin, stock_qty=100 (or 50), plan_qty=0, stock_status='UU'

-- 3. Shelf occupied
SELECT status_flag, soft_zone_id FROM dmshelf
WHERE shelf_no = (
    SELECT location_no FROM dnstock WHERE bcr_data = 'PLTTSS0002'
);
-- Expected: status_flag='1' (OCCUPIED)

-- 4. Pallet stored
SELECT current_station_no, status_flag FROM dnpallet
WHERE bcr_data = 'PLTTSS0002';
-- Expected: current_station_no=bin, status_flag=STORED

-- 5. Carry deleted (no orphan)
SELECT COUNT(*) AS orphan_carry FROM dncarryinfo
WHERE pallet_id = (SELECT pallet_id FROM dnpallet WHERE bcr_data = 'PLTTSS0002');
-- Expected: 0

-- 6. HostSend prepared (JOB_TYPE must be 22)
SELECT job_type, report_flag, result_location_no, result_qty
FROM dnhostsend
WHERE bcr_data = 'PLTTSS0002';
-- Expected: job_type='22', report_flag='0', result_location_no=actual bin, result_qty=qty

-- 7. Audit trail
SELECT COUNT(*) FROM dnstockhistory WHERE bcr_data = 'PLTTSS0002';
SELECT COUNT(*) FROM dninoutresult WHERE carry_key = (/* carry_key from dnhostsend */);
```

---

## <span style="color:skyblue; font-weight:bold">DNHOSTSEND Verification Checklist (Post-ID33)</span>

| **Field**            | **Expected Value**              | **Source / Rule**                                                                              | **Pass / Fail** |
| -------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------- | --------------- |
| `job_type`           | `22` (Unplanned Storage)        | From `DNWorkInfo.job_type` ← `DNStoragePlan.JOB_TYPE.NOPLAN_STORAGE`. **Must NOT be `02`.**    | [ ] P <br>[ ] F |
| `status_flag`        | `4`                             | Hardcoded Completed by `CarryCompleteOperator`                                                 | [ ] P <br>[ ] F |
| `plan_ukey`          | Sequence key `22` + seq         | From `DNWorkInfo.plan_ukey`                                                                    | [ ] P <br>[ ] F |
| `stock_id`           | `STKxxx` format                 | From `DNWorkInfo.stock_id` → matches `DNSTOCK` record                                          | [ ] P <br>[ ] F |
| `system_conn_key`    | Carry key (`CKxxx`)             | The carry key from the deleted `DNCARRYINFO`                                                   | [ ] P <br>[ ] F |
| `result_qty`         | `100` (A, C) or `50` (B)        | Actual stored qty                                                                              | [ ] P <br>[ ] F |
| `result_area_no`     | `9200` (A, B) or `9100` (C)     | From `DNStock.area_no`                                                                         | [ ] P <br>[ ] F |
| `result_location_no` | Actual bin address              | Must be actual physical bin — NOT null, NOT temp location                                      | [ ] P <br>[ ] F |
| `result_lot_no`      | `BATUSS0002` (or blank for ZPCK)| From `DNStock.lot_no`                                                                          | [ ] P <br>[ ] F |
| `hardware_type`      | `3`                             | Hardcoded ASRS designation                                                                     | [ ] P <br>[ ] F |
| `report_flag`        | `0`                             | Pending SAP notification upload                                                                | [ ] P <br>[ ] F |
| `bcr_data`           | `PLTTSS0002`                    | Physical pallet barcode                                                                        | [ ] P <br>[ ] F |
| `storage_location_from` | `VT01`                       | Copied from `DNWorkInfo.storage_location_from` (set at plan creation)                          | [ ] P <br>[ ] F |
| `storage_location_to`   | `FGW1` / `PACK` / `FGW2`    | Copied from `DNWorkInfo.storage_location_to` (set at plan creation)                            | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">AGC ID Message Summary</span>

**Sub-Scenario A / B (Same-Warehouse — FINAL BCR):**

| **ID**   | **Direction** | **Trigger**                                        | **Key Fields**                                                                                        |
| -------- | ------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **ID26** | AGC → MC      | Pallet arrives at Station 1301/1302/1106           | `MC key = DUMMY`, `BC data = PLTTSS0002`, `Load info = "1"`, `Control info = "000"`                  |
| **ID05** | MC → AGC      | After `AutoStorageScheduler` creates carry         | `Transport class = "3"`, `Source = 1301/1302/1106`, `Dest = 720x or 721x BCR`, `Location = All "0"` |
| **ID25** | AGC → MC      | AGC acknowledges ID05                              | `Response section = "00"` (Normal)                                                                    |
| **ID64** | AGC → MC      | STV picks up pallet at P&D station                 | Pickup completion confirmation                                                                        |
| **ID26** | AGC → MC      | Pallet arrives at BCR 720x or 721x                 | `MC key = carry key`, `Station = 720x / 721x`                                                        |
| **ID05** | MC → AGC      | After `StorageSender` selects bin                  | `Transport class = "1"`, `Source = 720x/721x`, `Dest = 9007–9010 / 9011–9014`, `Location = bin`     |
| **ID33** | AGC → MC      | SRM completes storage                              | `Transport section = "1"`, `Completion = "0"` (Normal)                                               |
| **ID54** | MC → AGC      | `ShelfMonitor` cycle (not storage code)            | Lamp update if threshold crossed                                                                      |

**Sub-Scenario C (Cross-Warehouse — Intermediate BCR 720x then FINAL BCR 710x):**

| **ID**     | **Direction** | **Trigger**                                        | **Key Fields**                                                                                        |
| ---------- | ------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **ID26**   | AGC → MC      | Pallet arrives at Station 1301/1302                | `MC key = DUMMY`, `BC data = PLTTSS0002`                                                              |
| **ID05**   | MC → AGC      | Tier 2: Intermediate station selected (720x)       | `Transport class = "3"`, `Source = 1301`, `Dest = 7207–7210` (Intermediate)                         |
| **ID26**   | AGC → MC      | Pallet arrives at BCR 720x (INTERMEDIATE)          | `MC key = carry key`, `Station = 720x`                                                                |
| **ID05**   | MC → AGC      | After 720x selects final BCR 710x                  | `Transport class = "3"`, `Source = 720x`, `Dest = 7101–7106` (Final OP BCR)                          |
| **ID26**   | AGC → MC      | Pallet arrives at BCR 710x (FINAL BCR)             | `MC key = carry key`, `Station = 710x`                                                                |
| **ID05**   | MC → AGC      | After `StorageSender` selects Tempering bin        | `Transport class = "1"`, `Source = 710x`, `Dest = 9001–9006`, `Location = bin`                       |
| **ID33**   | AGC → MC      | SRM completes Tempering storage                    | `Transport section = "1"`, `Completion = "0"` (Normal)                                               |

---

## <span style="color:skyblue; font-weight:bold">Post-Conditions</span>

Upon successful completion of the full end-to-end flow, all of the following must hold:

| #   | Post-Condition                                                                                                                                              | Verification SQL      |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| 1   | `DNSTORAGEPLAN.STATUS_FLAG = 4` (COMPLETE) for `BCR_DATA = 'PLTTSS0002'`                                                                                    | SQL-04 block 1        |
| 2   | A: Bin in Zone 002 (aisles 9007–9010), WH 9200. B: Bin in Zone 003 (aisles 9011–9014), WH 9200. C: Bin in Zone 001 (aisles 9001–9006), WH 9100            | SQL-04 block 3        |
| 3   | `DNCARRYINFO` deleted — no orphan carry records                                                                                                             | SQL-04 block 5        |
| 4   | `DNSTOCK.location_no` = actual bin; `stock_qty` = plan qty; `plan_qty = 0`; `stock_status = 'UU'`                                                           | SQL-04 block 2        |
| 5   | `DMSHELF.STATUS_FLAG = 1` (OCCUPIED) for the assigned bin                                                                                                   | SQL-04 block 3        |
| 6   | `DNPALLET.STATUS_FLAG = STORED`; `current_station_no` = actual bin                                                                                          | SQL-04 block 4        |
| 7   | `DNHOSTSEND` contains 1 row: `report_flag = 0`, `job_type = '22'`, `result_location_no` populated, `storage_location_from = 'VT01'`                         | DNHOSTSEND Checklist  |
| 8   | `DNSTOCKHISTORY` contains audit entry for this storage                                                                                                      | SQL-04 block 7        |
| 9   | `DNINOUTRESULT` contains audit entry                                                                                                                        | SQL-04 block 7        |
| 10  | `DMWAREHOUSE.last_used_station_no` for WH 9200 (A/B) or 9100 (C) updated to the aisle used                                                                 | `SELECT last_used_station_no FROM dmwarehouse WHERE station_no IN ('9200','9100')` |
| 11  | No `DNWORKINFO` in status 0 or 1 for this carry                                                                                                             | All work completed (status = 4) |
| 12  | **Sub-Scenario C only:** No orphan intermediate `DNCARRYINFO` or `DNARRIVAL` at BCR 720x                                                                    | SQL-02 (count = 0)    |

---

## <span style="color:skyblue; font-weight:bold">Notes / Defects</span>

> ⚠️ **JOB_TYPE must be `22` throughout:** Unplanned storage sets `JOB_TYPE = '22'` in `DNSTORAGEPLAN`. This value propagates to `DNWORKINFO.job_type` via `insertWorkInfoStorage()` which reads `storagePlan.getJobType()`. It then propagates to `DNHOSTSEND.job_type` via `HostSendController.insertByWorkInfo()`. Any value of `02` in these tables for an unplanned pallet is a defect.

> ⚠️ **PM from 1301/1302 must use Tier 1 (721x direct), NOT Tier 2 (720x intermediate):** The key routing fix ensures `selectAisleAndGetBcrStation()` finds direct route 1301→7211 for zone 003. If `DNCARRYINFO.dest` = 7207–7210 for a PM item from 1301/1302, the routing is broken.

> ⚠️ **Cross-warehouse (Sub-Scenario C) requires 3-stage physical travel:** 1301 → 720x (intermediate, `carry_flag=3`) → 710x (final, `carry_flag=3` → `1`) → shelf. The carry record is updated twice before it becomes STORAGE type.

> ⚠️ **ShelfMonitor is the only ID54 source:** `CarryCompleteOperator` never sends ID54 directly. Only `ShelfMonitor` on its polling cycle triggers location-full lamp changes.

> ⚠️ **Station 1106 is PM-only:** Any non-zone-003 item at station 1106 must be rejected by `validateStoragePlan()` before any carry is created. The plan stays at `STATUS_FLAG = 0`.

```
[                                                                               ]
[                                                                               ]
[                                                                               ]
```
