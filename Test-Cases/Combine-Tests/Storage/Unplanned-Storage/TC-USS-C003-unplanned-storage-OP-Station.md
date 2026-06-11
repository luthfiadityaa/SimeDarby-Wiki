[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">TC-USS-C003 — Unplanned Storage Setting — Storage from OP / QC Station</span>

| **Field**            | **Value**                                                                                                                                                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**     | TC-USS-C003-unplanned-storage-OP-Station                                                                                                                                                                                                                     |
| **Test Type**        | Combined Test (Screen Validation + DB Flow + AGC Control Flow)                                                                                                                                                                                               |
| **Feature**          | Unplanned Storage Setting                                                                                                                                                                                                                                    |
| **DFD Reference**    | [[SCREEN] Unplanned Storage](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/946/Unplanned-Storage)                                                                                                          |
| **Screen Class**     | `jp.co.daifuku.wms.web.display.storage.unplannedstorage.UnplannedStorageSCH`                                                                                                                                                                                 |
| **Job Type**         | Unplanned Storage — `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.NOPLAN_STORAGE` (value: `22`)                                                                                                                                                                        |
| **OP Station**       | `ST1303` (QC / Reject Station)                                                                                                                                                                                                                               |
| **Soft Zone**        | Zone `001` (FG Tempering) → Aisles 9001–9006 · Zone `002` (FG Ambient) → Aisles 9007–9010 · Zone `004` (EMP_PB) → Aisles 9007–9010 · Zone `005` (ZFNP) → fallback to 001/002                                                                               |
| **Allowed Zones**    | `001`, `002`, `004`, `005` — Zone `003` (PM/ZPCK) is **REJECTED** from station 1303                                                                                                                                                                          |
| **Target Warehouse** | `9100` (FGW2 Tempering) or `9200` (FGW1 Ambient) — direct routing, no intermediate station                                                                                                                                                                  |
| **Aisle Selector**   | `DisperseAisleSelector` (Pattern 3, Single Deep) for WH 9100 · `WNCollectAisleSelector` (Pattern 4, Double Deep) for WH 9200                                                                                                                                |
| **AGC Flow**         | AGC Linkage Spec Flow **05** — Input Area Storage (Direct Transfer): BCRs and Load Size Detectors Are Used                                                                                                                                                   |
| **Physical Route (Zone 001/Tempering)** | `ST1303` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `BCR 710x (7101–7106)` → `ID26` → `ID05` (**STORAGE**) → `SRM Bin (9001–9006)` → `ID33`                           |
| **Physical Route (Zone 002/Ambient)**   | `ST1303` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `BCR 710x (7107–7110)` → `ID26` → `ID05` (**STORAGE**) → `SRM Bin (9007–9010)` → `ID33`                           |
| **Related Stories**  | [US-5140](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5140) · [US-6184](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6184)                                                                           |
| **Tester**           | _____________                                                                                                                                                                                                                                                |
| **Test Date**        | _____________                                                                                                                                                                                                                                                |
| **Overall Status**   | [ ] Pass &nbsp;&nbsp;[ ] Fail                                                                                                                                                                                                                                |

---

## <span style="color:skyblue; font-weight:bold">Objective</span>

Verify that the **Unplanned Storage Setting** screen (`UnplannedStorageSCH`) creates a valid `DNSTORAGEPLAN` record (`JOB_TYPE = 22`) upon pressing **Set (F2)**, and that the subsequent automated control flow — `AutoStorageScheduler` → `AsrsInboundStationOperator` → `StorageSender` → AGC ID messages → `CarryCompleteOperator` — completes end-to-end for the OP QC station (1303), with all transactional tables correctly updated.

Station 1303 coverage includes three routing sub-scenarios:

| Sub-Scenario | Station | Item Type       | Target Area       | BCR Route                   |
| ------------ | ------- | --------------- | ----------------- | --------------------------- |
| **A**        | 1303    | ZFNP (zone 001) | 9100 (Tempering)  | Direct → BCR 710x (7101–7106) |
| **B**        | 1303    | ZFNP (zone 002) | 9200 (Ambient)    | Direct → BCR 710x (7107–7110) |
| **C (Rejected)** | 1303 | ZPCK (zone 003) | (any)            | REJECTED — no routes 1303→721x |

Key assertions per design spec:
- `JOB_TYPE = 22` (NOPLAN_STORAGE) propagates from `DNSTORAGEPLAN` → `DNWORKINFO` → `DNHOSTSEND`.
- `validateStoragePlan()` rejects zone 003 (PM/ZPCK) from station 1303 **upfront** — no carry, no pallet, no stock created.
- Station 1303 routes **directly** to BCR 710x (7101–7110) — **no intermediate station** (unlike 1301/1302 cross-warehouse flow).
- WH 9100 uses `DisperseAisleSelector` (Pattern 3) — single-deep, distributes evenly across aisles 9001–9006.
- WH 9200 uses `WNCollectAisleSelector` (Pattern 4) — double-deep, batches grouped via `DNCOLLECTINFO`.
- `ShelfMonitor` — **not** storage code — is the **only** source of ID54 location-full lamp control.

---

## <span style="color:skyblue; font-weight:bold">Scope</span>

| Layer                | Scope                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Screen (SCH)**     | Input validation, `DNSTORAGEPLAN` INSERT with `JOB_TYPE = 22`                                                |
| **Scheduler**        | `AutoStorageScheduler` — `validateStoragePlan()`, zone/station check, aisle selection, carry creation        |
| **Station Operator** | `AsrsInboundStationOperator` at BCR 710x (7101–7106 for Tempering, 7107–7110 for Ambient) — FINAL BCR        |
| **Storage Sender**   | `StorageSender` — bin selection, `DMShelf` reservation, ID05 dispatch                                        |
| **AGC Linkage**      | ID26 (Arrival), ID05 (Transport Command), ID25 (Response), ID64 (Pick-up), ID33 (Completion), ID54 (Lamp)    |
| **Completion**       | `CarryCompleteOperator` — full post-ID33 table updates                                                       |
| **Lamp**             | `ShelfMonitor` — location-full lamp logic (ID54)                                                             |

---

## <span style="color:skyblue; font-weight:bold">System Architecture Reference</span>

```
SAP/User Screen
    │
    │  [Set F2] INSERT DNSTORAGEPLAN (STATUS=0, JOB_TYPE=22, ITEM=ZFNP, AREA=9100 or 9200)
    ▼
AutoStorageScheduler  ◄── ID26 from AGC (pallet arrives at ST1303)
    │  validateStoragePlan():
    │     zone 001/002/004/005 → ALLOWED at 1303
    │     zone 003 (PM/ZPCK)   → REJECTED upfront
    │  selectAisleAndGetBcrStation():
    │     zone 001/005→9100: route 1303→7101-7106 → BCR 7101-7106 (Tier 1 direct)
    │     zone 002/005→9200: route 1303→7107-7110 → BCR 7107-7110 (Tier 1 direct)
    │  INSERT DNPallet, DNStock, DNCarryInfo (DIRECT_TRAVEL, dest=710x, end=9100/9200)
    │  UPDATE DNStoragePlan STATUS→1, DNArrival SCHEDULED
    │  → ID05 (DIRECT_TRAVEL): 1303 → 710x BCR
    ▼
AGC/Conveyor transports pallet to BCR 710x
    │  → ID26 Arrival at 710x
    ▼
AsrsInboundStationOperator (710x — FINAL BCR)
    │  wh_station == end_station → FINAL BCR
    │  carry_flag → STORAGE, dest → 9100 or 9200
    ▼
StorageSender (bin selection)
    │  LocationManager.searchLocation() → empty bin
    │  UPDATE DMShelf → RESERVED
    │  → ID05 (STORAGE): 710x → bin
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
| 3   | Station 1303 is **ONLINE** (`status_flag = 1`) and **NOT suspended** (`sendable = 1`)                                 | `SELECT station_no, status_flag, sendable FROM dmstation WHERE station_no = '1303'`                                              |
| 4   | BCR stations 7101–7110 (710x) are online, `max_instruction = 2`, `class_name = AsrsInboundStationOperator`            | `SELECT station_no, status_flag, max_instruction, sendable FROM dmstation WHERE station_no BETWEEN '7101' AND '7110'`            |
| 5   | AGC status is `ONLINE` (`status_flag = 1`) and `connection_flag = 1` (connected)                                      | `SELECT status_flag, connection_flag FROM dmagc`                                                                                 |
| 6   | `DMITEM` contains `ZFNPFG001` (zone 001), `ZFNPFG002` (zone 002) and `ZPCKPM003` (zone 003) for test data            | `SELECT item_code, soft_zone_id FROM dmitem WHERE item_code IN ('ZFNPFG001','ZFNPFG002','ZPCKPM003')`                           |
| 7   | `DMWAREHOUSE`: WH 9100 has `aisle_decision_pattern = 3` (Disperse). WH 9200 has `aisle_decision_pattern = 4` (WNCollect) | `SELECT station_no, aisle_decision_pattern FROM dmwarehouse WHERE station_no IN ('9100','9200')`                               |
| 8   | `DMAISLE` for aisles 9001–9006 (WH 9100) and 9007–9010 (WH 9200) has `status = 1` (NORMAL)                           | `SELECT station_no, status, double_deep_kind FROM dmaisle WHERE station_no BETWEEN '9001' AND '9010'`                           |
| 9   | `DMSHELF` has empty shelves in Zone `001` (aisles 9001–9006) and Zone `002` (aisles 9007–9010)                        | `SELECT COUNT(*) FROM dmshelf WHERE status_flag = '0' AND soft_zone_id IN ('001','002') AND aisle_station_no BETWEEN '9001' AND '9010'` → Expected: > 0 |
| 10  | `DMROUTEID` routes exist: 1303 → 710x (7101–7106) for Tempering, 1303 → 710x (7107–7110) for Ambient                 | `SELECT COUNT(*) FROM dmrouteid WHERE start_station_no = '1303' AND end_station_no LIKE '71%'` → Expected: ≥ 10                 |
| 11  | No direct route 1303 → 721x (7211–7214) exists (PM aisles are NOT reachable from 1303)                                | `SELECT COUNT(*) FROM dmrouteid WHERE start_station_no = '1303' AND end_station_no LIKE '721%'` → Expected: **0**               |
| 12  | No existing `DNSTORAGEPLAN` with `BCR_DATA = 'PLTTSS0003'` in status `0`, `1`, or `2`                                 | `SELECT COUNT(*) FROM dnstorageplan WHERE bcr_data = 'PLTTSS0003' AND status_flag IN ('0','1','2')` → Expected: 0                |

---

## <span style="color:skyblue; font-weight:bold">Test Data</span>

| **Field**                       | **Sub-Scenario A (ZFNP Tempering)** | **Sub-Scenario B (ZFNP Ambient)** | **Sub-Scenario C (ZPCK — Rejected)** |
| ------------------------------- | ------------------------------------ | ---------------------------------- | ------------------------------------- |
| **Pallet No**                   | `PLTTSS0003`                         | `PLTTSS0003`                       | `PLTTSS0003`                          |
| **Material Code**               | `ZFNPFG001` (zone 001)               | `ZFNPFG002` (zone 002)             | `ZPCKPM003` (zone 003)                |
| **Batch No**                    | `BATUSS0003`                         | `BATUSS0003`                       | (N/A — plan rejected before creation) |
| **Storage Qty**                 | `100`                                | `100`                              | N/A                                   |
| **Tempering Period**            | `7`                                  | `7`                                | N/A (disabled for ZPCK)               |
| **Expiry Days**                 | `30`                                 | `30`                               | N/A (disabled for ZPCK)               |
| **Storage Location From**       | `VT01`                               | `VT01`                             | N/A                                   |
| **To Location**                 | `FGW2`                               | `FGW1`                             | `PACK`                                |
| **PLAN_AREA_NO (mapped)**       | `9001`                               | `9002`                             | N/A — REJECTED                        |
| **Source Station**              | `ST1303`                             | `ST1303`                           | `ST1303`                              |
| **Expected BCR**                | `7101–7106` (Tier 1 direct FINAL)    | `7107–7110` (Tier 1 direct FINAL)  | No BCR — `validateStoragePlan()` rejects |
| **Expected SRM Aisles**         | `9001–9006` (Tempering, Single-Deep) | `9007–9010` (Ambient, Double-Deep) | N/A                                   |
| **Aisle Selector**              | `DisperseAisleSelector` (Pattern 3)  | `WNCollectAisleSelector` (Pattern 4)| N/A                                   |
| **Expected Soft Zone in Shelf** | Zone `001`                           | Zone `002`                         | N/A                                   |

---

## <span style="color:skyblue; font-weight:bold">Test Steps</span>

---

### Case 1: Station Validation Rules — Zone 003 Rejection at 1303 (🔴 Negative — FIXED)

**Purpose:** Confirm `validateStoragePlan()` in `AutoStorageScheduler` rejects zone 003 (PM/ZPCK) from station 1303 upfront, preventing any carry or pallet record creation. This was a known bug (previously failed at routing level with no routes 1303→7211-7214); it is now fixed to reject proactively.

| **No** | **Action**                                                                                                            | **Expected Result**                                                                                                                                                                                     | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 1.1    | Create `DNSTORAGEPLAN` for `ZPCKPM003` (zone 003) via screen Set (F2). Confirm plan is created (`STATUS_FLAG = '0'`). | `DNSTORAGEPLAN` record created successfully (screen-level validation passes for ZPCK — `validateAfterSelectItemCode` does not block ZPCK). `STATUS_FLAG = '0'`.                                         |                   | [ ] P <br>[ ] F |
| 1.2    | Send **ID26 (Arrival Report)** from Station **1303** for the ZPCK plan above.                                         | `AutoStorageScheduler.validateStoragePlan()` **REJECTS** zone 003 at station 1303. No `DNCARRYINFO`, `DNPALLET`, or `DNSTOCK` is created. `DNSTORAGEPLAN.STATUS_FLAG` remains `0` (not updated to `1`). |                   | [ ] P <br>[ ] F |
| 1.3    | Verify no orphan carry or pallet records exist after rejection.                                                       | `SELECT COUNT(*) FROM dncarryinfo WHERE ...` = 0. `SELECT COUNT(*) FROM dnpallet WHERE bcr_data = 'PLTTSS0003'` = 0. No physical transport command sent to AGC.                                         |                   | [ ] P <br>[ ] F |
| 1.4    | Verify no route from 1303 to PM BCR 721x exists (physical constraint confirming the rejection is correct).            | `SELECT COUNT(*) FROM dmrouteid WHERE start_station_no = '1303' AND end_station_no BETWEEN '7211' AND '7214'` → Expected: **0** (no such routes exist). Design is correct.                              |                   | [ ] P <br>[ ] F |

---

### Case 2: Sub-Scenario A — OP Station 1303, ZFNP Tempering (Zone 001 → 9100) (🟢 Positive)

**Purpose:** Verify direct Tier 1 routing from OP QC station 1303 to BCR 710x (7101–7106) for FGW2 Tempering storage. Uses `DisperseAisleSelector` (Pattern 3) — single-deep, distributes evenly.

#### Step 2.1 — Screen: DNSTORAGEPLAN INSERT

| **No** | **Action**                                                                                                                      | **Expected Result**                                                                                                                                          | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 2.1.1  | Enter test data Sub-Scenario A (ZFNPFG001, BATUSS0003, Qty=100, Tempering=7, Expiry=30, To Location: FGW2). Click **Set (F2)**. | Success `6461009`. `DNSTORAGEPLAN` created: `JOB_TYPE = '22'`, `PLAN_AREA_NO = '9001'`, `STORAGE_LOCATION_TO = 'FGW2'`, `STATUS_FLAG = '0'`. Run **SQL-01**. |                   | [ ] P <br>[ ] F |

#### Step 2.2 — AGC Linkage: ID26 Arrival at Station 1303

| **No** | **Action**                                                                                                                                                  | **Expected Result**                                                                                                                                                                       | **Actual Result** | **Status**      |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.2.1  | Send **ID26 (Arrival Report)** from AGC for Station `1303`. `MC key = DUMMY (99999...)`, `BC data = PLTTSS0003`, `Load info = "1"`, `Control info = "000"`. | `AutoStorageScheduler` processes. `validateStoragePlan()`: zone 001 allowed at 1303. **Passes**. Aisle selector: `DisperseAisleSelector` (Pattern 3) for WH 9100.                         |                   | [ ] P <br>[ ] F |
| 2.2.2  | Query `DNCARRYINFO` after ID26.                                                                                                                             | New carry record: `carry_flag = 3` (DIRECT_TRAVEL), `dest = 710x BCR (7101–7106)` (Tier 1 direct, route 1303→710x exists), `end = '9001'`. **No intermediate 720x used**. Run **SQL-02**. |                   | [ ] P <br>[ ] F |
| 2.2.3  | Verify aisle distribution for WH 9100.                                                                                                                      | `DisperseAisleSelector` distributes pallets evenly — selected aisle is from 9001–9006. `DMWareHouse.last_used_station_no` updated to the selected aisle.                                  |                   | [ ] P <br>[ ] F |
| 2.2.4  | Verify `DNSTORAGEPLAN.STATUS_FLAG` updated to `1`.                                                                                                          | `STATUS_FLAG = '1'` (NOWWORKING). `PROCESS_QTY` incremented.                                                                                                                              |                   | [ ] P <br>[ ] F |

#### Step 2.3 — AGC Linkage: ID05 Transport Command (DIRECT_TRAVEL)

| **No** | **Action**                                                           | **Expected Result**                                                                                                                         | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.3.1  | Verify MC sends **ID05** to AGC.                                     | ID05: `Transport class = "3"` (Direct transfer), `Source = 1303`, `Dest = 710x BCR (7101–7106)`, `Location = All "0"`. **No 720x in dest.** |                   | [ ] P <br>[ ] F |
| 2.3.2  | AGC acknowledges with **ID25** (`Response section = "00"` — Normal). | MC registers acknowledgement. No error.                                                                                                     |                   | [ ] P <br>[ ] F |

#### Step 2.4 — AGC Linkage: ID26 Arrival at BCR 710x (FINAL BCR — Tempering)

| **No** | **Action**                                                                  | **Expected Result**                                                                                                                                                                        | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 2.4.1  | AGC transports pallet to BCR 710x (7101–7106). Send **ID26** from BCR 710x. | `AsrsInboundStationOperator.arrival()`: `wh_station (9100) == end_station (9001/9100)` → **FINAL BCR**. `carry_flag` updated: DIRECT_TRAVEL → **STORAGE**. `dest` → `9100`.                |                   | [ ] P <br>[ ] F |
| 2.4.2  | `StorageSender` at BCR 710x performs bin selection for Tempering.           | `LocationManager.searchLocation()` → empty bin in Zone **001**, aisle 9001–9006. `DMShelf.status → RESERVED` (status 2). BCR-to-aisle 1:1 physical match (e.g. 7101→9001). Run **SQL-03**. |                   | [ ] P <br>[ ] F |

#### Step 2.5 — AGC Linkage: ID05 (STORAGE) and ID33 Completion

| **No** | **Action**                                                                                                 | **Expected Result**                                                                                                                                      | **Actual Result** | **Status**      |
| ------ | ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.5.1  | MC sends **ID05 (STORAGE)** to AGC after bin selection.                                                    | ID05: `Transport class = "1"` (Storage), `Source = 710x`, `Dest = aisle 9001–9006`, `Location = bin address`.                                            |                   | [ ] P <br>[ ] F |
| 2.5.2  | SRM stores pallet. AGC sends **ID33 (Normal Completion)** (`Transport section = "1"`, `Completion = "0"`). | `CarryCompleteOperator` executes full completion. Run **SQL-04** to verify all post-ID33 table updates. `DNHOSTSEND.JOB_TYPE = '22'`. Bin is in WH 9100. |                   | [ ] P <br>[ ] F |

---

### Case 3: Sub-Scenario B — OP Station 1303, ZFNP Ambient (Zone 002 → 9200) (🟢 Positive)

**Purpose:** Verify direct Tier 1 routing from OP QC station 1303 to BCR 710x (7107–7110) for FGW1 Ambient storage. Uses `WNCollectAisleSelector` (Pattern 4) — double-deep, batch grouping via `DNCOLLECTINFO`.

#### Step 3.1 — Screen: DNSTORAGEPLAN INSERT

| **No** | **Action**                                                                                                                      | **Expected Result**                                                                                                                  | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 3.1.1  | Enter test data Sub-Scenario B (ZFNPFG002, BATUSS0003, Qty=100, Tempering=7, Expiry=30, To Location: FGW1). Click **Set (F2)**. | Success `6461009`. `DNSTORAGEPLAN`: `JOB_TYPE = '22'`, `PLAN_AREA_NO = '9002'`, `STORAGE_LOCATION_TO = 'FGW1'`, `STATUS_FLAG = '0'`. |                   | [ ] P <br>[ ] F |

#### Step 3.2 — AGC Linkage: ID26 Arrival at Station 1303

| **No** | **Action**                                                                           | **Expected Result**                                                                                                                                                                    | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.2.1  | Send **ID26** from Station **1303**. `BC data = PLTTSS0003`, `Control info = "000"`. | `validateStoragePlan()`: zone 002 allowed at 1303. `WNCollectAisleSelector` (Pattern 4) runs for WH 9200. `DNCOLLECTINFO` record inserted for batch grouping to an aisle in 9007–9010. |                   | [ ] P <br>[ ] F |
| 3.2.2  | Query `DNCARRYINFO` after ID26.                                                      | `carry_flag = 3` (DIRECT_TRAVEL), `dest = 710x BCR (7107–7110)` (Tier 1 direct, route 1303→7107 exists), `end = '9002'`. **No 720x in dest. No intermediate**. Run **SQL-02**.         |                   | [ ] P <br>[ ] F |
| 3.2.3  | Verify `DNCOLLECTINFO` for double-deep batch grouping.                               | `DNCOLLECTINFO` record exists with `aisle_collect_key` mapping this batch to a target aisle in 9007–9010 (WNCollect logic groups same batch together for double-deep efficiency).      |                   | [ ] P <br>[ ] F |

#### Step 3.3 — AGC Linkage: ID05 (DIRECT_TRAVEL) and FINAL BCR 710x

| **No** | **Action**                                                                                              | **Expected Result**                                                                                                                                                 | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.3.1  | Verify MC sends **ID05** to AGC.                                                                        | ID05: `Transport class = "3"`, `Source = 1303`, `Dest = 710x BCR (7107–7110)`.                                                                                      |                   | [ ] P <br>[ ] F |
| 3.3.2  | Pallet arrives at BCR 710x (7107–7110). Send **ID26** from BCR 710x.                                    | `AsrsInboundStationOperator`: `wh_station (9200) == end (9002/9200)` → **FINAL BCR**. `carry_flag → STORAGE`. `dest → 9200`.                                        |                   | [ ] P <br>[ ] F |
| 3.3.3  | `StorageSender` at 710x selects Ambient bin. MC sends **ID05 (STORAGE)**. SRM completes. **ID33** sent. | Bin reserved in Zone **002**, aisle 9007–9010. BCR-to-aisle 1:1 match (e.g. 7107→9007). `CarryCompleteOperator` runs. `DNHOSTSEND.JOB_TYPE = '22'`. Run **SQL-04**. |                   | [ ] P <br>[ ] F |

---

### Case 4: Zone 005 (ZFNP — Fallback Routing) Verification (🟢 Positive)

**Purpose:** Confirm that ZFNP items (zone 005) from station 1303 correctly fall back to zone 001 or 002 depending on the `PLAN_AREA_NO` via `DMSoftZonePriority`.

| **No** | **Action**                                                                                                                       | **Expected Result**                                                                                                                                              | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 4.1    | Create `DNSTORAGEPLAN` for zone 005 item (`ZFNPFG005`), `To Location = FGW2` (→ `PLAN_AREA_NO = 9001`). Send **ID26** from 1303. | `DMSoftZonePriority`: zone 005 → fallback to zone 001 in WH 9100. `selectAisleAndGetBcrStation()` → aisles 9001–9006 → BCR 7101–7106. Route 1303→7101 confirmed. |                   | [ ] P <br>[ ] F |
| 4.2    | Create `DNSTORAGEPLAN` for zone 005 item, `To Location = FGW1` (→ `PLAN_AREA_NO = 9002`). Send **ID26** from 1303.               | `DMSoftZonePriority`: zone 005 → fallback to zone 002 in WH 9200. → aisles 9007–9010 → BCR 7107–7110. Route 1303→7107 confirmed.                                 |                   | [ ] P <br>[ ] F |
| 4.3    | Verify shelf assigned is in the correct fallback zone.                                                                           | Sub-scenario A (WH 9100): `DMSHELF.soft_zone_id = '001'`. Sub-scenario B (WH 9200): `DMSHELF.soft_zone_id = '002'`. Run **SQL-03** after each.                   |                   | [ ] P <br>[ ] F |

---

### Case 5: Aisle Round-Robin Verification at 1303 (🟡 System Behaviour)

**Purpose:** Verify that `DMWareHouse.last_used_station_no` advances correctly per aisle selection cycle, ensuring even distribution across Tempering aisles (DisperseAisleSelector, WH 9100).

| **No** | **Action**                                                                                                  | **Expected Result**                                                                                                                                        | **Actual Result** | **Status**      |
| ------ | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 5.1    | Record current `DMWareHouse.last_used_station_no` for WH 9100 before test.                                  | Note current value (e.g. `9003`).                                                                                                                          |                   | [ ] P <br>[ ] F |
| 5.2    | Submit two consecutive unplanned storage plans for Zone 001 items from station 1303. Trigger ID26 for each. | Each carry is assigned to a different aisle (round-robin starting from the aisle AFTER `last_used_station_no`). `last_used_station_no` advances each time. |                   | [ ] P <br>[ ] F |
| 5.3    | Query `DMWAREHOUSE.last_used_station_no` after both carries are created.                                    | Value advanced by 2 positions from initial. E.g. if initial = `9003`, after 2 submissions → `9005`.                                                        |                   | [ ] P <br>[ ] F |

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
WHERE bcr_data = 'PLTTSS0003'
ORDER BY regist_date DESC;
-- Expected per sub-scenario:
-- JOB_TYPE          = '22'                   (NOPLAN_STORAGE — NOT '02')
-- STATUS_FLAG       = '0'                    (UNSTART on create, '1' after ID26)
-- Scenario A: PLAN_AREA_NO = '9001', STORAGE_LOCATION_TO = 'FGW2'
-- Scenario B: PLAN_AREA_NO = '9002', STORAGE_LOCATION_TO = 'FGW1'
-- STOCK_STATUS      = 'UU'
```

### SQL-02 — DNCARRYINFO After AutoStorageScheduler

```sql
SELECT
    CARRY_KEY, CARRY_FLAG, SOURCE_STATION_NO,
    DEST_STATION_NO, END_STATION_NO, CMD_STATUS
FROM dncarryinfo
WHERE pallet_id = (SELECT pallet_id FROM dnpallet WHERE bcr_data = 'PLTTSS0003');
-- Scenario A: carry_flag=3, dest=710x (7101-7106), end='9001'
-- Scenario B: carry_flag=3, dest=710x (7107-7110), end='9002'
-- NOTE: 720x (7207-7210) must NEVER appear as dest for 1303
```

### SQL-03 — DMShelf Reservation Check

```sql
SELECT
    s.shelf_no, s.status_flag, s.soft_zone_id, s.aisle_station_no
FROM dmshelf s
WHERE s.status_flag = '2'   -- RESERVED
  AND (s.aisle_station_no BETWEEN '9001' AND '9006'    -- Tempering (Scenario A)
       OR s.aisle_station_no BETWEEN '9007' AND '9010') -- Ambient (Scenario B)
ORDER BY s.aisle_station_no;
-- Expected: 1 row RESERVED in correct zone
-- Scenario A: soft_zone_id = '001', aisle = 9001-9006
-- Scenario B: soft_zone_id = '002', aisle = 9007-9010
```

### SQL-04 — Full Post-ID33 Completion Verification

```sql
-- 1. Storage plan completed
SELECT status_flag FROM dnstorageplan
WHERE bcr_data = 'PLTTSS0003';
-- Expected: '4' (COMPLETE)

-- 2. Stock updated with actual bin
SELECT location_no, stock_qty, plan_qty, stock_status, area_no
FROM dnstock
WHERE bcr_data = 'PLTTSS0003';
-- Expected: location_no = actual bin, stock_qty = 100, plan_qty = 0
-- Scenario A: area_no = '9100'. Scenario B: area_no = '9200'

-- 3. Shelf occupied
SELECT status_flag, soft_zone_id, aisle_station_no FROM dmshelf
WHERE shelf_no = (
    SELECT location_no FROM dnstock WHERE bcr_data = 'PLTTSS0003'
);
-- Expected: status_flag = '1' (OCCUPIED)
-- Scenario A: soft_zone_id = '001', aisle = 9001-9006
-- Scenario B: soft_zone_id = '002', aisle = 9007-9010

-- 4. Pallet stored
SELECT current_station_no, status_flag FROM dnpallet
WHERE bcr_data = 'PLTTSS0003';
-- Expected: current_station_no = bin, status_flag = STORED

-- 5. Carry deleted
SELECT COUNT(*) AS orphan_carry FROM dncarryinfo
WHERE pallet_id = (SELECT pallet_id FROM dnpallet WHERE bcr_data = 'PLTTSS0003');
-- Expected: 0

-- 6. HostSend (JOB_TYPE must be 22)
SELECT job_type, report_flag, result_location_no, result_qty, result_area_no
FROM dnhostsend
WHERE bcr_data = 'PLTTSS0003';
-- Expected: job_type='22', report_flag='0', result_location_no=actual bin

-- 7. Audit trails
SELECT COUNT(*) FROM dnstockhistory WHERE bcr_data = 'PLTTSS0003';
-- Expected: ≥ 1

-- 8. Warehouse round-robin updated
SELECT last_used_station_no FROM dmwarehouse
WHERE station_no IN ('9100', '9200');
-- Expected: updated to the aisle used in this test
```

### SQL-05 — Validate Zone 003 Rejection (Case 1)

```sql
-- Confirm no carry/pallet created after rejection
SELECT COUNT(*) AS carry_count FROM dncarryinfo
WHERE pallet_id IN (SELECT pallet_id FROM dnpallet WHERE bcr_data = 'PLTTSS0003');
-- Expected: 0

SELECT COUNT(*) AS pallet_count FROM dnpallet WHERE bcr_data = 'PLTTSS0003';
-- Expected: 0

-- Confirm plan still at STATUS_FLAG=0
SELECT status_flag FROM dnstorageplan WHERE bcr_data = 'PLTTSS0003';
-- Expected: '0' (NOT updated to '1')
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
| `result_qty`         | `100`                           | Actual stored qty from plan                                                                    | [ ] P <br>[ ] F |
| `result_area_no`     | `9100` (A) or `9200` (B)        | From `DNStock.area_no`                                                                         | [ ] P <br>[ ] F |
| `result_location_no` | Actual bin address              | Must be actual physical bin — NOT null, NOT temp location                                      | [ ] P <br>[ ] F |
| `result_lot_no`      | `BATUSS0003`                    | From `DNStock.lot_no`                                                                          | [ ] P <br>[ ] F |
| `hardware_type`      | `3`                             | Hardcoded ASRS designation                                                                     | [ ] P <br>[ ] F |
| `report_flag`        | `0`                             | Pending SAP notification upload                                                                | [ ] P <br>[ ] F |
| `bcr_data`           | `PLTTSS0003`                    | Physical pallet barcode                                                                        | [ ] P <br>[ ] F |
| `storage_location_from` | `VT01`                       | Copied from `DNWorkInfo.storage_location_from`                                                 | [ ] P <br>[ ] F |
| `storage_location_to`   | `FGW2` (A) or `FGW1` (B)    | Copied from `DNWorkInfo.storage_location_to`                                                   | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">AGC ID Message Summary</span>

| **ID**   | **Direction** | **Trigger**                                        | **Key Fields for OP Station 1303**                                                                       |
| -------- | ------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **ID26** | AGC → MC      | Pallet arrives at Station 1303                     | `MC key = DUMMY (99999...)`, `BC data = PLTTSS0003`, `Load info = "1"`, `Control info = "000"`           |
| **ID05** | MC → AGC      | After `AutoStorageScheduler` creates carry         | `Transport class = "3"` (Direct transfer), `Source = 1303`, `Dest = 710x BCR (7101–7110)`, `Location = All "0"` |
| **ID25** | AGC → MC      | AGC acknowledges ID05                              | `Response section = "00"` (Normal)                                                                        |
| **ID64** | AGC → MC      | STV picks up pallet at P&D station                 | Pickup completion confirmation                                                                            |
| **ID26** | AGC → MC      | Pallet arrives at BCR 710x                         | `MC key = assigned carry key`, `Station = 7101–7110`                                                     |
| **ID05** | MC → AGC      | After `StorageSender` selects bin                  | `Transport class = "1"` (Storage), `Source = 710x`, `Dest = 9001–9006 (A) or 9007–9010 (B)`, `Location = bin` |
| **ID25** | AGC → MC      | AGC acknowledges storage ID05                      | `Response section = "00"` (Normal)                                                                        |
| **ID64** | AGC → MC      | SRM picks up pallet at aisle conveyor              | Pickup confirmation                                                                                       |
| **ID33** | AGC → MC      | SRM completes physical storage in bin              | `Transport section = "1"` (Storage), `Completion = "0"` (Normal)                                         |
| **ID54** | MC → AGC      | `ShelfMonitor` cycle detects threshold change      | `Lamp = "01"` (Full) or `"04"` (Warning), `Instruction = "0"` (Off) or `"1"` (On)                        |

---

## <span style="color:skyblue; font-weight:bold">Post-Conditions</span>

Upon successful completion of the full end-to-end flow, all of the following must hold:

| #   | Post-Condition                                                                                                                                             | Verification SQL      |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| 1   | `DNSTORAGEPLAN.STATUS_FLAG = 4` (COMPLETE) for `BCR_DATA = 'PLTTSS0003'`                                                                                   | SQL-04 block 1        |
| 2   | A: Bin in Zone 001 (aisles 9001–9006), WH 9100. B: Bin in Zone 002 (aisles 9007–9010), WH 9200.                                                           | SQL-04 block 3        |
| 3   | `DNCARRYINFO` deleted — no orphan carry records                                                                                                            | SQL-04 block 5        |
| 4   | `DNSTOCK.location_no` = actual bin; `stock_qty = 100`; `plan_qty = 0`; `stock_status = 'UU'`                                                               | SQL-04 block 2        |
| 5   | `DMSHELF.STATUS_FLAG = 1` (OCCUPIED) for the assigned bin                                                                                                  | SQL-04 block 3        |
| 6   | `DNPALLET.STATUS_FLAG = STORED`; `current_station_no` = actual bin                                                                                         | SQL-04 block 4        |
| 7   | `DNHOSTSEND` contains 1 row: `report_flag = 0`, `job_type = '22'`, `result_location_no` populated                                                          | DNHOSTSEND Checklist  |
| 8   | `DNSTOCKHISTORY` contains audit entry                                                                                                                       | SQL-04 block 7        |
| 9   | `DNINOUTRESULT` contains audit entry                                                                                                                        | SQL-04 block 7        |
| 10  | `DMWAREHOUSE.last_used_station_no` updated for WH 9100 (A) or 9200 (B)                                                                                     | SQL-04 block 8        |
| 11  | No `DNWORKINFO` in status 0 or 1 for this carry                                                                                                             | All work completed (status = 4) |
| 12  | **Case 1 (Rejection):** `DNSTORAGEPLAN.STATUS_FLAG` still `0`, no pallet/carry created for ZPCK item                                                       | SQL-05                |

---

## <span style="color:skyblue; font-weight:bold">Notes / Defects</span>

> ⚠️ **JOB_TYPE must be `22` throughout:** Same rule as TC-USS-C002. `DNSTORAGEPLAN.JOB_TYPE = '22'` → `DNWORKINFO.job_type = '22'` → `DNHOSTSEND.job_type = '22'`. Any `02` in this chain for an unplanned pallet is a defect.

> ⚠️ **Station 1303 does NOT use intermediate routing:** Unlike HP stations 1301/1302 which may use 720x as an intermediate for cross-warehouse Tempering storage, station 1303 routes **directly** to BCR 710x (7101–7110). Routes 1303→7101 through 1303→7110 all exist. There is no Tier 2 at 1303.

> ⚠️ **Zone 003 (PM/ZPCK) rejection from 1303 is an upfront guard (FIXED):** Previously the system would attempt routing and fail (no routes 1303→7211-7214). The fix adds `validateStoragePlan()` check that rejects zone 003 at 1303 before any carry creation. Test Case 1 verifies this fix is effective.

> ⚠️ **DisperseAisleSelector vs WNCollectAisleSelector:** Station 1303 can target either WH 9100 (DisperseAisleSelector — distributes evenly, no batch grouping, single-deep) or WH 9200 (WNCollectAisleSelector — batch grouped, `DNCOLLECTINFO` inserted, double-deep). Verify the correct selector is used per target area.

> ⚠️ **ShelfMonitor is the only ID54 source:** Storage code never sends ID54 directly. Only `ShelfMonitor` on its polling cycle triggers location-full lamp changes.

> ⚠️ **BCR-to-Aisle is a 1:1 physical constraint:** BCR 7101 can ONLY deliver to aisle 9001, 7102→9002, etc. If `StorageSender` selects a shelf in the wrong aisle for the assigned BCR, physical storage will fail. SQL-03 verifies the reserved shelf is in the expected aisle.

```
[                                                                               ]
[                                                                               ]
[                                                                               ]
```
