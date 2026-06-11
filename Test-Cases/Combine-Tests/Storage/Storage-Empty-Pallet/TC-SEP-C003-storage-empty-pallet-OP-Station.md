[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">TC-SEP-C003 — Storage Empty Pallet — Storage from OP / QC Station</span>

| **Field**            | **Value**                                                                                                                                                                          |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**     | TC-SEP-C003-storage-empty-pallet-OP-Station                                                                                                                                        |
| **Test Type**        | Combined Test (Screen Validation + DB Flow + AGC Control Flow)                                                                                                                     |
| **Feature**          | Empty Pallet Storage Setting                                                                                                                                                       |
| **DFD Reference**    | [[SCREEN] Empty Pallet Storage](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/-SCREEN-Empty-Pallet-Storage)                  |
| **Screen Class**     | `jp.co.daifuku.wms.web.display.storage.emptypalletsetting.EmptyPalletSettingSCH`                                                                                                   |
| **Job Type**         | Planned Storage — `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.STORAGE` (value: `02`)                                                                                                        |
| **Item Code**        | Fixed: `DMITEM.EMP_PB` (Empty Pallet — Material Type 99)                                                                                                                           |
| **Soft Zone**        | `004` (Empty Pallet) → Aisles 9007–9010. Fallback: Zone `002` (FG Ambient) via `DMSoftZonePriority`                                                                                |
| **Target Warehouse** | `9200` (FGW1 Ambient) — `Constant.AMBIENT_AREA`                                                                                                                                    |
| **Aisle Selector**   | `WNCollectAisleSelector` (Pattern 4, Double Deep — `DMWareHouse.aisle_decision_pattern = 4`)                                                                                       |
| **AGC Flow**         | AGC Linkage Spec Flow **05** — Input Area Storage (Direct Transfer): BCRs and Load Size Detectors Are Used                                                                         |
| **Physical Route**   | `ST1303` → `ID26` → `AutoStorageScheduler` → `ID05` (**DIRECT_TRAVEL**) → `720x BCR` → `ID26` → `ID05` (**STORAGE**) → `SRM Bin (9007–9010)` → `ID33` (Completion)                 |
| **Related Stories**  | [US-5464](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5464) · [Bug-6751](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6751) |
| **Tester**           | _____________                                                                                                                                                                      |
| **Test Date**        | _____________                                                                                                                                                                      |
| **Overall Status**   | [ ] Pass &nbsp;&nbsp;[ ] Fail                                                                                                                                                      |

---

## <span style="color:skyblue; font-weight:bold">Objective</span>

Verify that the **Empty Pallet Setting** screen (`EmptyPalletSettingSCH`) correctly enforces all input validation rules, creates a valid `DNSTORAGEPLAN` record upon pressing **Set (F2)**, and that the subsequent automated control flow — `AutoStorageScheduler` → `AsrsInboundStationOperator` → `StorageSender` → AGC ID messages → `CarryCompleteOperator` — completes end-to-end, resulting in a physically stored empty pallet in Soft Zone 004 (aisles 9007–9010, warehouse 9200) with all transactional tables correctly updated.

Key assertions per design spec:
- `ITEM_CODE` is fixed to `DMITEM.EMP_PB` and cannot be changed by the user.
- `PLAN_QTY` is always hardcoded to `1`.
- `PLAN_AREA_NO` is always fixed to `Constant.AMBIENT_AREA` (= `9200`).
- `JOB_TYPE` = `DNSTORAGEPLAN.JOB_TYPE.STORAGE` (planned storage, **not** `22` unplanned).
- Duplicate pallet detection covers both `DNSTOCK` (via `BarcodeCheckOperator`) and active `DNSTORAGEPLAN` status `0`/`1` (via `checkBCR()`).
- `ShelfMonitor` — not storage code — is the **only** source of ID54 location-full lamp control.

---

## <span style="color:skyblue; font-weight:bold">Scope</span>

| Layer                | Scope                                                                                                     |
| -------------------- | --------------------------------------------------------------------------------------------------------- |
| **Screen (SCH)**     | Input validation, `DNSTORAGEPLAN` INSERT logic                                                            |
| **Scheduler**        | `AutoStorageScheduler` — pallet matching, aisle selection, carry creation                                 |
| **Station Operator** | `AsrsInboundStationOperator` at BCR 710x — FINAL BCR determination                                        |
| **Storage Sender**   | `StorageSender` — bin selection, `DMShelf` reservation, ID05 dispatch                                     |
| **AGC Linkage**      | ID26 (Arrival), ID05 (Transport Command), ID25 (Response), ID64 (Pick-up), ID33 (Completion), ID54 (Lamp) |
| **Completion**       | `CarryCompleteOperator` — full post-ID33 table updates                                                    |
| **Lamp**             | `ShelfMonitor` — location-full lamp logic (ID54)                                                          |

---

## <span style="color:skyblue; font-weight:bold">System Architecture Reference</span>

```
SAP/User Screen
    │
    │  [Set F2] INSERT DNSTORAGEPLAN (STATUS=0, JOB_TYPE=STORAGE, ITEM=EMP_PB, AREA=9200)
    ▼
AutoStorageScheduler  ◄── ID26 from AGC (pallet arrives at 1303)
    │  INSERT DNPallet, DNStock, DNCarryInfo (DIRECT_TRAVEL, dest=710x, end=9200)
    │  UPDATE DNStoragePlan STATUS→1, DNArrival SCHEDULED
    │  → ID05 (DIRECT_TRAVEL): 1303 → 710x
    ▼
AGC/Conveyor transports pallet to BCR 710x
    │  → ID26 Arrival at 710x
    ▼
AsrsInboundStationOperator (710x)
    │  wh_station(9200) == end_station(9200) → FINAL BCR
    │  UPDATE DNCarryInfo: carry_flag→STORAGE, dest→9200
    ▼
StorageSender (710x)
    │  LocationManager.searchLocation() → DMShelf (Zone 004, aisle 9007–9010)
    │  UPDATE DMShelf STATUS→RESERVED
    │  UPDATE DNCarryInfo dest→shelf_bin
    │  → ID05 (STORAGE): 710x → shelf_bin
    ▼
AGC SRM deposits pallet into shelf bin
    │  → ID33 (Operation Completion Report, Transport Section=1 Storage, Completion=0 Normal)
    ▼
CarryCompleteOperator
    │  UPDATE DNWorkInfo, INSERT DNHostSend, UPDATE DNStock, UPDATE DMShelf→OCCUPIED
    │  UPDATE DNPallet→STORED, INSERT DNStockHistory, UPDATE DNStoragePlan→COMPLETE
    │  INSERT DNInOutResult, DELETE DNCarryInfo
    ▼
ShelfMonitor (next cycle)
    │  Count empty DMShelf in WH 9200
    │  → ID54 (DO Output) if threshold crossed (warning: <50, full: =0, reset: >60)
```

---

## <span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

| #   | Pre-Condition                                                                                                              | How to Verify                                                                                                                                                   |
| --- | -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | System (MC/WMS) is **ONLINE** and all background services are active                                                       | Check system status screen                                                                                                                                      |
| 2   | AGC connection: `DMStation.connection_flag = 1` (**CONNECTED**), `status_flag = 1` (**NORMAL**) for stations 1303 and 710x | `SELECT station_no, status_flag, connection_flag FROM dmstation WHERE station_no IN ('1303','7107','7108','7109','7110')`                                       |
| 3   | `DMITEM` contains a valid `EMP_PB` item code                                                                               | `SELECT item_code, item_type FROM dmitem WHERE item_code = (SELECT emp_pb FROM wms_constants)`                                                                  |
| 4   | `DMWareHouse` for WH 9200: `aisle_decision_pattern = 4` (WNCollect), `zone_manage_type = 2` (SOFT_ZONE)                    | `SELECT aisle_decision_pattern, zone_manage_type FROM dmwarehouse WHERE station_no = '9200'`                                                                    |
| 5   | `DMAisle` for aisles 9007–9010: `status = 1` (NORMAL), `double_deep_kind = 1` (DOUBLE)                                     | `SELECT station_no, status, double_deep_kind FROM dmaisle WHERE wh_station_no = '9200' AND station_no BETWEEN '9007' AND '9010'`                                |
| 6   | `DMShelf` has at least 1 empty shelf in Zone 004 within aisles 9007–9010 (520 shelves per aisle per design)                | `SELECT COUNT(*) FROM dmshelf WHERE wh_station_no = '9200' AND soft_zone_id = '004' AND status_flag = '0'` → Expected: > 0                                      |
| 7   | `DMSoftZonePriority` for WH 9200 Zone 004 fallback to Zone 002 is configured                                               | `SELECT soft_zone_id, priority_soft_zone, priority FROM dmsoftzonepriority WHERE wh_station_no = '9200' AND soft_zone_id = '004'` → Should show fallback to 002 |
| 8   | `DMRouteId` routes exist from stations `ST1303` to **710x** BCRs (**7107–7110**)                                           | `SELECT COUNT(*) FROM dmrouteid WHERE start_station_no IN ('1303') AND end_station_no LIKE '71_'` → Expected: ≥ 8                                               |
| 10  | BCR stations **7107–7110**: `sendable = 1`, `max_instruction = 2`, `class_name = AsrsInboundStationOperator`               | `SELECT station_no, sendable, max_instruction, class_name FROM dmstation WHERE station_no BETWEEN '7107' AND '7110'`                                            |


---

## <span style="color:skyblue; font-weight:bold">Test Data</span>

| **Field**                | **Value**      | **Notes**                                             |
| ------------------------ | -------------- | ----------------------------------------------------- |
| **Pallet No**            | `EMP0000003`   | Must be ≥ 10 characters (PALLET_NO_LENGTH constraint) |
| **Item Code**            | `EMP_PB`       | Fixed; read-only on screen; from `DMITEM.EMP_PB`      |
| **Item Name**            | `Empty Pallet` | Display only; auto-populated                          |
| **Storage Qty**          | `1`            | Fixed; hardcoded; cannot be changed                   |
| **Soft Zone**            | `004`          | Empty Pallet zone; assigned from DMITEM               |
| **Plan Area No**         | `9200`         | `Constant.AMBIENT_AREA` — hardcoded in SCH            |
| **Job Type**             | `STORAGE` (02) | `DNSTORAGEPLAN.JOB_TYPE.STORAGE` — not unplanned (22) |
| **Target Warehouse**     | `9200`         | FGW1 Ambient                                          |
| **Target Aisles**        | `9007–9010`    | Double deep, Soft Zone 004                            |
| **OP Inbound BCR Range** | `7107–7110`    | OP ASRS Inbound BCR — FGW1 Ambient                    |
| **OP Inbound Station**   | `1303`         | OP / QC Station                                       |

---

## <span style="color:skyblue; font-weight:bold">Test Steps</span>

---

### Case 1 : Successful Set (F2) - DNSTORAGEPLAN INSERT Verification (🟢 Positive)

**Purpose:** Confirm that a valid submission creates the `DNSTORAGEPLAN` record with exactly the field values defined in the design spec, including the correct `JOB_TYPE`, `PLAN_AREA_NO`, and `STORING_PAIR_KEY`.

| **#** | **Action**                                                                                     | **Expected Result**                                                                                                    | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 1.1   | Ensure all pre-conditions are met. Enter `EMP0000003` in **Pallet No** and click **Set (F2)**. | Success message `6461009` displayed. Screen input field clears after successful registration.                          |                   | [ ] P <br>[ ] F |
| 1.2   | Query `DNSTORAGEPLAN` immediately after Set (F2).                                              | One new record exists. Run SQL in **Section: Verification SQL → SQL-01**. Verify all fields match the checklist below. |                   | [ ] P <br>[ ] F |

**DNSTORAGEPLAN Field Verification Checklist (Post Set F2):**

| **Column**          | **Expected Value**                         | **Source**                                                             | **Pass / Fail** |
| ------------------- | ------------------------------------------ | ---------------------------------------------------------------------- | --------------- |
| `PLAN_UKEY`         | Sequence-generated key (format `02` + seq) | `WMSSequenceHandler.nextStoragePlanukeySeq()`                          | [ ] P [ ] F     |
| `LOAD_UNIT_KEY`     | `SYSTIMESTAMP` at time of insert           | `DbDateUtil.getSystemDateTime()`                                       | [ ] P [ ] F     |
| `STATUS_FLAG`       | `0` (UNSTART)                              | `DBValueDefine.DNStoragePlan.STATUS_FLAG.UNSTART`                      | [ ] P [ ] F     |
| `CANCEL_FLAG`       | `0` (NORMAL)                               | `DBValueDefine.DNStoragePlan.CANCEL_FLAG.NORMAL`                       | [ ] P [ ] F     |
| `JOB_TYPE`          | `02` — Planned Storage                     | `DBValueDefine.DNStoragePlan.JOB_TYPE.STORAGE` ⚠️ NOT `22` (unplanned) | [ ] P [ ] F     |
| `PLAN_DAY`          | Today's date in `YYYYMMDD`                 | `WmsFormatUtil.toStringDate(new Date())`                               | [ ] P [ ] F     |
| `BCR_DATA`          | `EMP0000003`                               | Value from screen Pallet No field                                      | [ ] P [ ] F     |
| `ITEM_CODE`         | `EMP_PB` value from `DMITEM`               | Fixed: `dsCond.getItemCode()`                                          | [ ] P [ ] F     |
| `PLAN_QTY`          | `1`                                        | Fixed hardcoded value                                                  | [ ] P [ ] F     |
| `PLAN_AREA_NO`      | `9200`                                     | `Constant.AMBIENT_AREA` — hardcoded in SCH                             | [ ] P [ ] F     |
| `STORING_PAIR_KEY`  | `ITEM_CODE + PLAN_LOT_NO` (concatenation)  | `dsCond.getItemCode() + storagePlan.getPlanLotNo()`                    | [ ] P [ ] F     |
| `REGIST_DATE`       | Current timestamp                          | `new Date()` at insert time                                            | [ ] P [ ] F     |
| `LAST_UPDATE_DATE`  | Current timestamp                          | `new Date()` at insert time                                            | [ ] P [ ] F     |
| `REGIST_PNAME`      | `EmptyPalletSettingSCH`                    | `this.getClass().getSimpleName()`                                      | [ ] P [ ] F     |
| `LAST_UPDATE_PNAME` | `EmptyPalletSettingSCH`                    | `this.getClass().getSimpleName()`                                      | [ ] P [ ] F     |

---

### Case 2 : AutoStorageScheduler: Pallet Arrival & Carry Creation

**Purpose:** Verify that when a physical pallet with barcode `EMP0000002` arrives at inbound station **1303**, `AutoStorageScheduler` correctly matches the `DNSTORAGEPLAN`, selects the appropriate BCR, and creates all carry-related DB records in a single atomic transaction.

**AGC Reference:** AGC Linkage Spec Flow **05** — Steps `25–27` (HP side arrival and transport command).

| **#** | **Action**                                                                                                                                         | **Expected Result**                                                                                                                                                                                                                                                                                                              | **Actual Result** | **Status**      |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.1   | Physical/simulated pallet with BCR `EMP0000003` placed at inbound station `1303`. AGC reads the barcode and sends **ID26** (Arrival Report) to MC. | AGC sends ID26 with: `MC key = All "9"` (**DUMMY**), `Load information = "1" (Present)`, `BC data = EMP0000003`, `Control Information = "000"`. `InOutStationOperator` inserts `DNARRIVAL`: `station_no = 1303`, `carry_key = DUMMY`, `sch_flag = 0` (**NOT_SCHEDULED**).                                                        |                   | [ ] P <br>[ ] F |
| 2.2   | Allow `AutoStorageScheduler` polling cycle to execute.                                                                                             | Scheduler finds `DNSTORAGEPLAN` with `bcr_data = 'EMP0000003'`, `status_flag = 0`, `plan_area_no = 9200`. `validateStoragePlan()` passes: Zone `004` (**EMP_PB**) is allowed at station **1303**.                                                                                                                                |                   | [ ] P <br>[ ] F |
| 2.3   | Verify BCR selection logic (Tier 1 — `selectAisleAndGetBcrStation()`).                                                                             | Zone **004** maps to aisles 9007–9010. `getReachableBcrForAisle()` checks `DMRouteId` from source **1303**: route `1303 → 710x` exists. Selected BCR is one of `7107`, `7108`, `7109`, or `7110` (least busy or round-robin).                                                                                                    |                   | [ ] P <br>[ ] F |
| 2.4   | Verify aisle decision: `WNCollectAisleSelector` (Pattern 4) for WH 9200.                                                                           | If `DNCollectInfo` already has an `aisle_collect_key` for this `STORING_PAIR_KEY`, use the same aisle. If not, balance query selects aisle with fewest existing + in-transit pallets. `DNCollectInfo` record inserted if new batch.                                                                                              |                   | [ ] P <br>[ ] F |
| 2.5   | Verify `DNPALLET` is inserted.                                                                                                                     | Run **SQL-02a**. One row exists: `current_station_no = '1303'`, `item_code = EMP_PB`, `soft_zone_id = '004'`, `status = not STORED (in transit)`.                                                                                                                                                                                |                   | [ ] P <br>[ ] F |
| 2.6   | Verify `DNSTOCK` is inserted.                                                                                                                      | Run **SQL-02b**. One row: `area_no = '9200'`, `item_code = EMP_PB`, `plan_qty = 1`, `stock_qty = 0` (physical qty set at ID33).                                                                                                                                                                                                  |                   | [ ] P <br>[ ] F |
| 2.7   | Verify `DNCARRYINFO` is inserted.                                                                                                                  | Run **SQL-02c**. One row: `carry_flag = '3'` (**DIRECT TRAVEL**), `dest_station_no = '710x'` (selected BCR), `end_station_no = '9200'`.                                                                                                                                                                                          |                   | [ ] P <br>[ ] F |
| 2.8   | Verify `DNSTORAGEPLAN` status update.                                                                                                              | `STATUS_FLAG` updated `0 → 1` (**NOWWORKING**). `process_qty` incremented by 1.                                                                                                                                                                                                                                                  |                   | [ ] P <br>[ ] F |
| 2.9   | Verify `DNARRIVAL` update.                                                                                                                         | `sch_flag = 1` (**SCHEDULED**), `sch_carry_key` populated with the new carry key.                                                                                                                                                                                                                                                |                   | [ ] P <br>[ ] F |
| 2.10  | Verify `StorageSender` dispatches ID05.                                                                                                            | MC sends **ID05** (Transport Command) to AGC: `Transport classification = "3" (Direct transfer)`, `Source station = 1303`, `Destination station = 710x (e.g. 7108)`, `Location number = All "0"`, `Load size = 000`. AGC responds with **ID25** `Response section = "00" (Normal)`. `DNARRIVAL.send_flag` updated to **SENDED**. |                   | [ ] P <br>[ ] F |

---

### Case 3 : AGC Transport: 1303 → BCR 710x (ID26 at BCR)

**Purpose:** Verify the conveyor/STV transport from inbound station to the HP ASRS BCR, and the AGC Arrival Report at the BCR station.

**AGC Reference:** AGC Linkage Spec Flow **05** — Steps `33–42` (Flow 1, STV transport to P&D station, conveyor to BCR).

| **#** | **Action**                                                                                                                             | **Expected Result**                                                                                                                                                                        | **Actual Result** | **Status**      |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 3.1   | AGC begins STV transport of pallet from station **1303** toward BCR **710x**.                                                          | AGC sends **ID64** (Pick-up completion report) once STV picks up pallet at P&D station. MC acknowledges.                                                                                   |                   | [ ] P <br>[ ] F |
| 3.2   | Pallet travels via conveyor and arrives at the BCR inlet of the selected BCR (e.g., 7108). AGC sends **ID26** (Arrival Report) at BCR. | ID26 received at MC: `MC key = Number assigned by MC (existing carry key)`, `Station number = 7108`. `AsrsInboundStationOperator.arrival()` triggered at station 7108.                     |                   | [ ] P <br>[ ] F |
| 3.3   | Verify FINAL BCR determination in `AsrsInboundStationOperator`.                                                                        | `wh_station_no` for station **7108** = `9200`. `end_station_no` in `DNCARRYINFO` = `9200`. Since `wh_station == end_station` → classified as **FINAL BCR** (NOT intermediate).             |                   | [ ] P <br>[ ] F |
| 3.4   | Verify `DNCARRYINFO` update at FINAL BCR.                                                                                              | `carry_flag` updated: `3` (**DIRECT TRAVEL**) → `1` (**STORAGE**). `dest_station_no` updated to `9200` (warehouse station). `source_station_no` updated to `7108`. `cmd_status` → `START`. |                   | [ ] P <br>[ ] F |
| 3.5   | Verify `DNARRIVAL` insert at BCR station.                                                                                              | New `DNARRIVAL` record inserted: `station_no = '7208'`, `carry_key = <carry key>`. `carryRequest()` invoked to wake `StorageSender`.                                                       |                   | [ ] P <br>[ ] F |

---

### Case 4 : StorageSender: Bin Selection & Storage Command

**Purpose:** Verify `StorageSender` at BCR **710x** selects an empty shelf in Zone **004** (aisles 9007–9010), reserves it, and dispatches the final storage ID05 to AGC.

| **#** | **Action**                                                                                | **Expected Result**                                                                                                                                                                                                                                                                                                         | **Actual Result** | **Status**      |
| ----- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 4.1   | `StorageSender` runs `soueceRightStation()` capacity check for BCR **710x**.              | `DMStation.max_instruction = 2`, suspend = 0, mode = NORMAL → BCR is eligible to receive a storage command.                                                                                                                                                                                                                 |                   | [ ] P [ ] F     |
| 4.2   | `LocationManager.searchLocation()` executes shelf search for Zone **004** in WH **9200**. | `DoubleDeepShelfSelector.searchEmptyShelf()` scans aisles 9007–9010 (double deep, round-robin from `DMWareHouse.last_used_station_no`). Finds empty rear shelf in Zone **004**.                                                                                                                                             |                   | [ ] P <br>[ ] F |
| 4.3   | Verify Zone **004** fallback logic if Zone **004** is exhausted.                          | If no empty Zone **004** shelf found, `DMSoftZonePriority` fallback triggers Zone 002 (FG Ambient). Verify fallback by checking: `SELECT * FROM dmsoftzonepriority WHERE wh_station_no = '9200' AND soft_zone_id = '004'`.                                                                                                  |                   | [ ] P <br>[ ] F |
| 4.4   | Verify `DMShelf` is reserved.                                                             | Run **SQL-03a**. Target shelf `status_flag` = `2` (**RESERVED**). `soft_zone_id` = `004` (or `002` if fallback triggered).                                                                                                                                                                                                  |                   | [ ] P <br>[ ] F |
| 4.5   | Verify `DNCARRYINFO` destination is updated to the bin shelf.                             | `dest_station_no` = the reserved shelf location. `aisle_station_no` = corresponding aisle (e.g., `9007` ~ `9010`).                                                                                                                                                                                                          |                   | [ ] P <br>[ ] F |
| 4.6   | Verify `DNARRIVAL` send_flag update and ID05 dispatch.                                    | `DNARRIVAL.send_flag` set to SENDED. MC sends **ID05** (Transport Command) to AGC: `Transport classification = "1" (Storage)`, `Source station = 7208`, `Destination station = 9007 ~ 9010` (aisle/SRM), `Location number = bin address`, `Load size = 000`. AGC responds with **ID25** `Response section = "00" (Normal)`. |                   | [ ] P <br>[ ] F |
| 4.7   | Verify `DMWareHouse.last_used_station_no` is updated for WH 9200.                         | After `AbstractAisleSelector.determin()` completes, `last_used_station_no` reflects the aisle just used (e.g., `9007`). Subsequent storage operations start round-robin from next aisle.                                                                                                                                    |                   | [ ] P <br>[ ] F |

---

### Case 5 : Storage Completion (ID33): Full Table Update Verification

**Purpose:** Verify that when the SRM completes physical storage of the empty pallet and AGC sends **ID33**, `CarryCompleteOperator.completeStorageWorkAndUpdateStock()` performs all 10 table update operations in the correct order.

**AGC Reference:** AGC Linkage Spec Flow **05** — Steps `51–57`: SRM picks up load (ID64) → SRM storage complete → **ID33** (Operation completion report, Transport section = `"1"` Storage, Completion section = `"0"` Normal completion) → MC performs inventory update and transport data deletion.

| **#** | **Action**                                                               | **Expected Result**                                                                                                                                                                                                                             | **Actual Result** | **Status**      |
| ----- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 5.1   | AGC SRM physically places pallet in shelf bin. AGC sends **ID33** to MC. | ID33 received: `Transport section = "1" (Storage)`, `Completion section = "0" (Normal completion)`. `CarryCompleteOperator` triggered.                                                                                                          |                   | [ ] P <br>[ ] F |
| 5.2   | Verify `DNWORKINFO` update (Order 1).                                    | `status_flag → 4` (COMPLETION). `result_location_no` = actual bin address. `result_qty = 1`.                                                                                                                                                    |                   | [ ] P <br>[ ] F |
| 5.3   | Verify `DNHOSTSEND` insert (Order 2).                                    | New row inserted. See full checklist in **Section: DNHostSend Verification Checklist**. `report_flag = 0` (pending SAP). `job_type` sourced from `DNWORKINFO.job_type` ← `DNSTORAGEPLAN.JOB_TYPE.STORAGE` (= `02`) — **must not be hardcoded**. |                   | [ ] P <br>[ ] F |
| 5.4   | Verify `DNSTOCK` update (Order 3).                                       | `location_no` = actual bin. `storage_date` = now. `stock_qty += 1` (was 0 before ID33). `plan_qty → 0`.                                                                                                                                         |                   | [ ] P <br>[ ] F |
| 5.5   | Verify `DMSHELF` update (Order 4).                                       | `status_flag → 1` (**OCCUPIED**). Was `2` (**RESERVED**) before ID33.                                                                                                                                                                           |                   | [ ] P <br>[ ] F |
| 5.6   | Verify `DNPALLET` update (Order 5).                                      | `current_station_no` = shelf bin. `status_flag = STORED`.                                                                                                                                                                                       |                   | [ ] P <br>[ ] F |
| 5.7   | Verify `DNSTOCKHISTORY` insert (Order 6).                                | Audit trail row created for the storage operation.                                                                                                                                                                                              |                   | [ ] P <br>[ ] F |
| 5.8   | Verify `DNSTORAGEPLAN` update (Order 7).                                 | `status_flag → 4` (COMPLETE). Updated via `PlanControllerFactory.makeInstance(work.job_type)`.                                                                                                                                                  |                   | [ ] P <br>[ ] F |
| 5.9   | Verify `DNINOUTRESULT` insert (Order 8).                                 | Audit trail row: `result_kind = 1` (**storage**), `carry_key = <carry key>`, `location = actual bin`.                                                                                                                                           |                   | [ ] P <br>[ ] F |
| 5.10  | Verify `DNCARRYINFO` deletion (Order 9).                                 | `DNCARRYINFO` row for `BCR_DATA = 'EMP0000003'` is **deleted**. No orphan carry records remain.                                                                                                                                                 |                   | [ ] P <br>[ ] F |

---

### Case 6 : ShelfMonitor: Location Full Lamp (ID54)

**Purpose:** Confirm that lamp updates are triggered exclusively by `ShelfMonitor` polling — **not** from within the storage completion code path — per the design spec.

**AGC Reference:** AGC Linkage Spec Flow **11** — Notification of Location-full Error or Location-full Warning (ID54).

| **#** | **Action**                                                                      | **Expected Result**                                                                                                                                                                                                                                                                                        | **Actual Result** | **Status**      |
| ----- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 6.1   | Confirm that the storage path (**Case 5**) does **not** directly send any ID54. | No ID54 is sent by `CarryCompleteOperator` or `StorageSender`. Verify by checking MC communication log during and after ID33 processing.                                                                                                                                                                   |                   | [ ] P <br>[ ] F |
| 6.2   | Allow `ShelfMonitor` polling cycle to run after storage completes.              | `ShelfMonitor` counts empty `DMSHELF` for WH 9200 (where `status_flag = 0`, aisle not DISCONNECTED, prohibition = **OK**). Subtracts `reserved_qty`.                                                                                                                                                       |                   | [ ] P <br>[ ] F |
| 6.3   | Verify lamp threshold behavior — Warning ON.                                    | If empty count < `free_shelf_warning_num_agc5` (default: **50**), and lamp was not already FULL: MC sends **ID54** to AGC: `Lamp number = "04" (Location full warning)`, `Instruction section = "1" (On)`. Signal tower (blue) flashes. Simultaneously sends ID54 for Lamp `"01"` (Location full) **OFF**. |                   | [ ] P <br>[ ] F |
| 6.4   | Verify lamp threshold behavior — Location Full ON.                              | If empty count = **0**: MC sends **ID54**: `Lamp number = "01" (Location full)`, `Instruction section = "1" (On)`. Signal tower (blue) lights solid. Simultaneously sends ID54 for Lamp `"04"` (Warning) **OFF**.                                                                                          |                   | [ ] P <br>[ ] F |
| 6.5   | Verify lamp threshold behavior — Reset.                                         | If empty count > `free_shelf_warning_release_num_agc5` (default: **60**): MC sends **ID54** for both Lamp `"01"` and `"04"` **OFF**. Signal tower (blue) extinguishes.                                                                                                                                     |                   | [ ] P <br>[ ] F |
| 6.6   | Verify `DMLocationFullLamp.status_flag` is updated on status change.            | Only when lamp status changes does MC update `DMLocationFullLamp.status_flag` and send ID54. No action if status is unchanged.                                                                                                                                                                             |                   | [ ] P <br>[ ] F |

---

### Case 7 : Error Recovery: ID35 Forced Removal

**Purpose:** Verify `Id35Process` (forced removal via **MOS**) correctly cleans up all transactional records at each stage, as defined in the design spec.

| **#** | **Action**                                                                                                       | **Expected Result**                                                                                                                                                                                           | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 7.1   | Trigger ID35 while pallet is at inbound station **1303** with carry already created (`sch_carry_key` populated). | `StorageTrackingDeleter` is called. `DNCARRYINFO` deleted. `DNPALLET` deleted. `DNSTOCK` deleted. `DNARRIVAL` deleted. `DNSTORAGEPLAN` status reset to `0` (not deleted).                                     |                   | [ ] P <br>[ ] F |
| 7.2   | Trigger ID35 while pallet is in transit at BCR **710x** (carry exists with DIRECT_TRAVEL or STORAGE flag).       | `StorageTrackingDeleter` deletes all records: `DNCARRYINFO`, `DNPALLET`, `DNSTOCK`, `DNWORKINFO`, `DNARRIVAL`. `DNSTORAGEPLAN` status reset to `0`. `DMShelf` status reset to `0` (EMPTY) if it was RESERVED. |                   | [ ] P <br>[ ] F |
| 7.3   | Confirm `DNSTORAGEPLAN` is NOT deleted in any ID35 scenario.                                                     | `DNSTORAGEPLAN` survives ID35 with `status_flag = 0`. Available for re-scheduling on next arrival.                                                                                                            |                   | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">Verification SQL</span>

### SQL-01 — DNSTORAGEPLAN Record (After Set F2)

```sql
SELECT
    PLAN_UKEY,
    LOAD_UNIT_KEY,
    STATUS_FLAG,
    CANCEL_FLAG,
    JOB_TYPE,
    PLAN_DAY,
    BCR_DATA,
    ITEM_CODE,
    PLAN_QTY,
    PLAN_AREA_NO,
    STORING_PAIR_KEY,
    REGIST_DATE,
    REGIST_PNAME,
    LAST_UPDATE_DATE,
    LAST_UPDATE_PNAME
FROM dnstorageplan
WHERE bcr_data = 'EMP0000003';
-- Expected:
-- STATUS_FLAG       = '0'                  (UNSTART)
-- CANCEL_FLAG       = '0'
-- JOB_TYPE          = '02'                 (STORAGE — NOT '22' unplanned)
-- PLAN_DAY          = YYYYMMDD of today
-- ITEM_CODE         = 'EMP_PB'
-- PLAN_QTY          = 1
-- PLAN_AREA_NO      = '9200'               (Constant.AMBIENT_AREA)
-- STORING_PAIR_KEY  = 'EMP_PB' || plan_lot_no  (concatenation)
-- REGIST_PNAME      = 'EmptyPalletSettingSCH'
-- LAST_UPDATE_PNAME = 'EmptyPalletSettingSCH'
```

### SQL-02a — DNPALLET (After AutoStorageScheduler)

```sql
SELECT
    pallet_id,
    current_station_no,
    item_code,
    soft_zone_id,
    bcr_data,
    status_flag
FROM dnpallet
WHERE bcr_data = 'EMP0000003';
-- Expected: 1 row
-- CURRENT_STATION_NO = '1303'
-- ITEM_CODE          = 'EMP_PB'
-- SOFT_ZONE_ID       = '004'
-- STATUS_FLAG        != 'STORED' (pallet is in transit)
```

### SQL-02b — DNSTOCK (After AutoStorageScheduler)

```sql
SELECT
    area_no,
    item_code,
    plan_qty,
    stock_qty,
    location_no,
    bcr_data
FROM dnstock
WHERE bcr_data = 'EMP0000003';
-- Expected: 1 row
-- AREA_NO    = '9200'
-- ITEM_CODE  = 'EMP_PB'
-- PLAN_QTY   = 1
-- STOCK_QTY  = 0          (real qty set at ID33)
-- LOCATION_NO is NULL or temp at this stage
```

### SQL-02c — DNCARRYINFO (After AutoStorageScheduler)

```sql
SELECT
    carry_key,
    carry_flag,
    dest_station_no,
    end_station_no,
    cmd_status,
    bcr_data
FROM dncarryinfo
WHERE bcr_data = 'EMP0000003';
-- Expected: 1 row
-- CARRY_FLAG       = '3'    (DIRECT_TRAVEL)
-- DEST_STATION_NO  = '710x' (e.g. 7107, 7108, 7109, or 7110)
-- END_STATION_NO   = '9200'
```

### SQL-03a — DMSHELF Reservation (After StorageSender bin selection)

```sql
SELECT
    location_no,
    status_flag,
    soft_zone_id,
    wh_station_no,
    aisle_station_no
FROM dmshelf
WHERE location_no = (
    SELECT dest_station_no
    FROM dncarryinfo
    WHERE bcr_data = 'EMP0000003'
);
-- Expected:
-- STATUS_FLAG    = '2'     (RESERVED)
-- SOFT_ZONE_ID   = '004'  (or '002' if Zone 004 fallback triggered)
-- WH_STATION_NO  = '9200'
-- AISLE_STATION_NO IN ('9007','9008','9009','9010')
```

### SQL-04 — Post ID33: Full Completion Verification

```sql
-- DNWorkInfo
SELECT status_flag, result_location_no, result_qty
FROM dnworkinfo
WHERE bcr_data = 'EMP0000003';
-- Expected: STATUS_FLAG = '4', RESULT_QTY = 1, RESULT_LOCATION_NO = actual bin

-- DNHostSend
SELECT job_type, status_flag, result_qty, result_area_no,
       result_location_no, hardware_type, report_flag, bcr_data
FROM dnhostsend
WHERE bcr_data = 'EMP0000003';
-- Expected:
-- JOB_TYPE           = '02'   (from DNWorkInfo ← DNStoragePlan — NOT hardcoded)
-- STATUS_FLAG        = '4'    (hardcoded Complete)
-- RESULT_QTY         = 1
-- RESULT_AREA_NO     = '9200'
-- RESULT_LOCATION_NO = actual bin address
-- HARDWARE_TYPE      = '3'    (hardcoded ASRS)
-- REPORT_FLAG        = '0'    (pending SAP notification)

-- DNStock
SELECT location_no, stock_qty, plan_qty, storage_date
FROM dnstock
WHERE bcr_data = 'EMP0000003';
-- Expected: LOCATION_NO = actual bin, STOCK_QTY = 1, PLAN_QTY = 0

-- DMShelf
SELECT status_flag
FROM dmshelf
WHERE location_no = (
    SELECT location_no FROM dnstock WHERE bcr_data = 'EMP0000003'
);
-- Expected: STATUS_FLAG = '1' (OCCUPIED)

-- DNPallet
SELECT current_station_no, status_flag
FROM dnpallet
WHERE bcr_data = 'EMP0000003';
-- Expected: CURRENT_STATION_NO = actual bin, STATUS_FLAG = 'STORED'

-- DNStoragePlan
SELECT status_flag
FROM dnstorageplan
WHERE bcr_data = 'EMP0000003';
-- Expected: STATUS_FLAG = '4' (COMPLETE)

-- DNCarryInfo — must be DELETED
SELECT COUNT(*)
FROM dncarryinfo
WHERE bcr_data = 'EMP0000003';
-- Expected: 0

-- DNInOutResult — must exist
SELECT result_kind, carry_key, location_no
FROM dninoutresult
WHERE bcr_data = 'EMP0000003';
-- Expected: 1 row, RESULT_KIND = 1 (storage)

-- DNStockHistory — must exist
SELECT COUNT(*)
FROM dnstockhistory
WHERE bcr_data = 'EMP0000003';
-- Expected: ≥ 1
```

---

## <span style="color:skyblue; font-weight:bold">DNHostSend Verification Checklist (Post-ID33)</span>

| **Field**            | **Expected Value**         | **Source / Rule**                                                                         | **Pass / Fail** |
| -------------------- | -------------------------- | ----------------------------------------------------------------------------------------- | --------------- |
| `job_type`           | `02` (Planned Storage)     | From `DNWorkInfo.job_type` ← `DNStoragePlan.JOB_TYPE.STORAGE`. **Must NOT be hardcoded.** | [ ] P <br>[ ] F |
| `status_flag`        | `4`                        | Hardcoded as Completed by `CarryCompleteOperator`                                         | [ ] P <br>[ ] F |
| `plan_ukey`          | Sequence key `02` + seq    | From `DNWorkInfo.plan_ukey`                                                               | [ ] P <br>[ ] F |
| `stock_id`           | `STKxxx` format            | From `DNWorkInfo.stock_id` → matches `DNSTOCK` record                                     | [ ] P <br>[ ] F |
| `system_conn_key`    | Active carry key (`CKxxx`) | The carry key from the deleted `DNCARRYINFO`                                              | [ ] P <br>[ ] F |
| `result_qty`         | `1`                        | Fixed quantity for EMP_PB                                                                 | [ ] P <br>[ ] F |
| `result_area_no`     | `9200`                     | From `DNStock.area_no`                                                                    | [ ] P <br>[ ] F |
| `result_location_no` | Actual bin                 | Must be actual physical bin address — NOT null, NOT temp location                         | [ ] P <br>[ ] F |
| `result_lot_no`      | Plan lot number string     | From `DNStock.lot_no`                                                                     | [ ] P <br>[ ] F |
| `hardware_type`      | `3`                        | Hardcoded ASRS designation                                                                | [ ] P <br>[ ] F |
| `report_flag`        | `0`                        | Pending SAP notification upload                                                           | [ ] P <br>[ ] F |
| `bcr_data`           | `EMP0000003`               | Physical pallet barcode                                                                   | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">AGC ID Message Summary</span>

| **ID**   | **Direction** | **Trigger**                                   | **Key Fields for EMP_PB Storage**                                                                |
| -------- | ------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **ID26** | AGC → MC      | Pallet arrives at station 1303                | `MC key = "9999..."` (DUMMY), `BC data = EMP0000003`, `Load info = "1"`, `Control info = "000"`  |
| **ID05** | MC → AGC      | After `AutoStorageScheduler` creates carry    | `Transport class = "3"` (Direct transfer), `Source = 1303`, `Dest = 710x`, `Location = All "0"`  |
| **ID25** | AGC → MC      | AGC acknowledges ID05                         | `Response section = "00"` (Normal)                                                               |
| **ID64** | AGC → MC      | STV picks up pallet at P&D station            | Pickup completion confirmation                                                                   |
| **ID26** | AGC → MC      | Pallet arrives at BCR 710x                    | `MC key = assigned carry key`, `Station = 710x`                                                  |
| **ID05** | MC → AGC      | After `StorageSender` selects bin             | `Transport class = "1"` (Storage), `Source = 710x`, `Dest = 9007–9010`, `Location = bin address` |
| **ID25** | AGC → MC      | AGC acknowledges storage ID05                 | `Response section = "00"` (Normal)                                                               |
| **ID64** | AGC → MC      | SRM picks up pallet at aisle conveyor         | Pickup confirmation                                                                              |
| **ID33** | AGC → MC      | SRM completes physical storage in bin         | `Transport section = "1"` (Storage), `Completion = "0"` (Normal)                                 |
| **ID54** | MC → AGC      | `ShelfMonitor` cycle detects threshold change | `Lamp = "01"` (Full) or `"04"` (Warning), `Instruction = "0"` (Off) or `"1"` (On)                |

---

## <span style="color:skyblue; font-weight:bold">Post-Conditions</span>

Upon successful completion of the full end-to-end flow, all of the following must hold:

| #   | Post-Condition                                                                                                                                                | Verification                                                             |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 1   | `DNSTORAGEPLAN.STATUS_FLAG = 4` (COMPLETE) for `BCR_DATA = 'EMP0000003'`                                                                                      | SQL-04                                                                   |
| 2   | Pallet physically stored in a Zone **004** shelf within aisles 9007–9010, WH 9200. If Zone 004 exhausted, Zone 002 was used per `DMSoftZonePriority` fallback | SQL-03a + SQL-04 (DMShelf soft_zone_id check)                            |
| 3   | `DNCARRYINFO` deleted — no orphan carry records                                                                                                               | SQL-04 (COUNT = 0)                                                       |
| 4   | `DNSTOCK.location_no` = actual bin; `stock_qty = 1`; `plan_qty = 0`                                                                                           | SQL-04                                                                   |
| 5   | `DMSHELF.STATUS_FLAG = 1` (OCCUPIED) for the assigned bin                                                                                                     | SQL-04                                                                   |
| 6   | `DNPALLET.STATUS_FLAG = STORED`; `current_station_no` = bin                                                                                                   | SQL-04                                                                   |
| 7   | `DNHOSTSEND` contains 1 row with `report_flag = 0`, `job_type = 02`, `result_location_no` populated                                                           | DNHostSend checklist                                                     |
| 8   | `DNSTOCKHISTORY` contains an audit entry for this storage                                                                                                     | SQL-04                                                                   |
| 9   | `DNINOUTRESULT` contains an audit entry                                                                                                                       | SQL-04                                                                   |
| 10  | `DMWareHouse.last_used_station_no` for WH 9200 updated to the aisle used                                                                                      | `SELECT last_used_station_no FROM dmwarehouse WHERE station_no = '9200'` |
| 11  | No `DNWORKINFO` in status 0 or 1 for this carry key                                                                                                           | All work completed (status = 4)                                          |

---

## <span style="color:skyblue; font-weight:bold">Notes / Defects</span>

> ⚠️ **JOB_TYPE Clarification:** The `EmptyPalletSettingSCH` sets `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.STORAGE` (value `02` — Planned Storage). This is **not** `22` (NOPLAN_STORAGE / Unplanned). The test case header in earlier drafts was incorrectly labelled "No Plan Storage" — this test covers **planned** storage with `job_type = 02`.

> ⚠️ **PLAN_AREA_NO is hardcoded:** The screen sets `PLAN_AREA_NO = Constant.AMBIENT_AREA` (= `9200`) regardless of user selection. There is no To Location field in the Empty Pallet screen (unlike Unplanned Storage). Empty pallets always go to FGW1 Ambient (Zone 004 aisles 9007–9010).

> ⚠️ **ShelfMonitor is the only ID54 source:** Storage code never sends ID54 directly. Only `ShelfMonitor` on its polling cycle triggers location-full lamp changes. Do not confuse with `CarryCompleteOperator`.

```
[                                                                               ]
[                                                                               ]
[                                                                               ]
```
