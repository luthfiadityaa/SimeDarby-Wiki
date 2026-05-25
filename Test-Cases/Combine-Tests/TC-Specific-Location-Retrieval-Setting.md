[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Combine Test — Specific Location Retrieval Setting</span>

**DFD Reference:** [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting)
**User Story:** [DFD Location Retrieval — 5792](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5792)
**Purpose:** End-to-end combine tests covering every flow combination of the Specific Location Retrieval Setting screen (`LocationRetrievalSCH`) — from screen submission, through AS21 messaging (ID12/ID32/ID33/ID64/ID26/ID25/ID68/ID45/ID63), to final pallet/stock/host-send updates.

##<span style="color:skyblue; font-weight:bold">Coverage Matrix</span>

| TC ID                | Removal Type        | Destination ST | AGC Route                                    | Continues To                          |
|----------------------|---------------------|----------------|----------------------------------------------|---------------------------------------|
| TC-SLRS-C001         | Unit Retrieval      | 1201-1204      | Flow 3 (direct to ID68)                      | ID26 completion                       |
| TC-SLRS-C002         | Unit Retrieval      | 1205-1209      | Flow 2 (STV 9001-9014 → ID68)                | ID26 completion                       |
| TC-SLRS-C003         | Unit Retrieval      | 1301 / 1302    | ID63 mode change → Flow 2 → ID68             | ID26 completion (InOutStationOperator)|
| TC-SLRS-C004         | Unit Retrieval      | 1303           | ID63 → Flow 1 (crane 7-10 hop) → ID68        | ID26 completion (InOutStationOperator)|
| TC-SLRS-C005         | Unit Retrieval      | 1303           | ID63 → STV 9007-9010 direct → ID68           | ID26 completion (InOutStationOperator)|
| TC-SLRS-C006         | Only Confirmation   | 1301 / 1302    | ID63 → Flow 2 → ID68 → ID45                  | [Unplanned Storage from HP](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-HP)        |
| TC-SLRS-C007         | Only Confirmation   | 1303           | ID63 → Flow 1 → ID68 → ID45                  | [Unplanned Storage from OP-Area](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-OP-Area)  |

##<span style="color:skyblue; font-weight:bold">Common Pre-Conditions</span>

Applies to every TC unless explicitly overridden.

- AGC is **ONLINE** (`DMGroupController.STATUS_FLAG = ONLINE`).
- Destination station is **NORMAL** and **NOT SUSPENDED** (`DMStation.STATUS = NORMAL`, `DMStation.SUSPEND = OFF`, `DMMachine.STATUS_FLAG = ACTIVE`).
- For Bi-Direction stations (1301-1303): station is already in **Retrieval Mode** (`DMStation.CURRENT_MODE = 2`) — if not, [TC-SLRS-C003/C004/C006/C007] must execute the ID63 step first.
- Daily Update is **not running**.
- Retrieval and Transport Data Clear Flag is **not in progress**.
- Inventory-to-Retrieval allocation has **not** locked the target stock.
- Target shelf condition is **NOT** in `NG / Prohibited / Reserved for Storage / Empty`.
- A valid `DNSTOCK` record exists for the pallet to be retrieved (status allows retrieval).
- Mandatory screen fields (red asterisk inputs) are populated.

##<span style="color:skyblue; font-weight:bold">Common Verification Helpers (SQL)</span>

```sql
-- station mode (1301-1303 only)
SELECT station_no, current_mode FROM dmstation WHERE station_no IN (1301,1302,1303);

-- carry record lifecycle
SELECT carry_key, work_type, carry_flag, cmd_status, source_station_no,
       aisle_station_no, dest_station_no, end_station_no
  FROM dncarryinfo WHERE pallet_id = :pallet_id;

-- pallet status
SELECT pallet_id, status_flag, current_station_no, wh_station_no FROM dnpallet WHERE pallet_id = :pallet_id;

-- arrival (Flow 1)
SELECT station_no, carry_key, bcr_data, controlinfo, send_flag FROM dnarrival WHERE bcr_data = :bcr;

-- work info lifecycle
SELECT job_no, job_type, status_flag, removal_type, plan_qty, result_qty, plan_area_no, plan_location_no
  FROM dnworkinfo WHERE job_no = :job_no;

-- host send (after ID26 completion)
SELECT job_no, job_type, status_flag, result_qty, report_flag FROM dnhostsend WHERE job_no = :job_no;

-- in/out result (1301-1303 only)
SELECT result_kind, station_no, location_no, wh_station_no, work_type, pallet_id, carry_key
  FROM dninoutresult WHERE pallet_id = :pallet_id;

-- shelf release
SELECT area_no, location_no, status_flag FROM dmshelf WHERE area_no = :area AND location_no = :loc;
```

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C001 — Unit Retrieval to ST 1201-1204 (Flow 3 direct)</span>

**Objective:** Verify a Unit Retrieval issued from `LocationRetrievalSCH` to an Ambient/Tempering station 1201-1204 flows ID12 → ID32 → ID33 → ID68 → ID26 with correct DB transitions and no aisle 7-10 hop.

##Pre-Conditions
- Pallet `P-C001` resides at a shelf served by an aisle station in `9001-9006` or `9011-9014` (within the same WH that owns 1201-1204).
- Destination station = `1201` (or 1202/1203/1204) — unidirectional retrieval station.

##Test Steps

| #  | Step                                                                                  | Expected Result |
|----|---------------------------------------------------------------------------------------|-----------------|
| 1  | Open **Specific Location Retrieval Setting** screen (`LocationRetrievalSCH`).         | Screen loads; mandatory red-asterisk fields visible. |
| 2  | Select Pallet `P-C001`, Station `1201`, Removal Type = **Unit Retrieval**, Priority = Normal, fill Material/Batch fields. Press **Set (F2)**. | Validations pass. `WebUnplannedRetrievalScheduler.schedule()` is invoked. |
| 3  | Verify DB right after Set(F2).                                                        | `DNWORKINFO`: row inserted, `JOB_TYPE=23`, `STATUS_FLAG=0`, `REMOVAL_TYPE=1`, `PLAN_QTY=Stock Qty`.<br>`DNWORKLIST`: row inserted with `RETRIEVAL_DETAIL=1`.<br>`DNCARRYINFO`: inserted, `WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1201`, `END_STATION_NO=1201`, `AISLE_STATION_NO=DMSHELF.PARENT_STATION_NO`.<br>`DNPALLET.STATUS_FLAG=3` (Reserved for Retrieval).<br>`DNSTOCKHISTORY`: row inserted with `LAST_UPDATE_PNAME=LocationRetrievalSCH`. |
| 4  | `RetrievalSender` picks up the new carry → ID12 dispatched.                           | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`. AS21 ID12 frame observed. |
| 5  | Inject **ID32** (AGC acceptance).                                                     | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`. |
| 6  | Inject **ID33** (pallet out of rack).                                                 | `DMSHELF.STATUS_FLAG=0` (Empty) for source location. `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO` set to source shelf station_no. |
| 7  | Inject **ID68** (arrived at 1201).                                                    | `DNOPERATIONDISPLAY` inserted with `STATION_NO=1201`, `MATERIAL_CODE=DNWORKINFO.MATERIAL_CODE`, `RETRIEVAL_QTY=DNWORKINFO.RESULT_QTY`. LED at 1201 shows Job Type / Material Code / Qty / Dock / Truck. |
| 8  | Operator presses **Completion** at terminal → **ID26** received.                      | Path = `RetrievalStationOperator` (1201-1209 branch).<br>`DNWORKINFO.STATUS_FLAG=4`, `RESULT_QTY=PLAN_QTY`, `RESULT_AREA_NO/LOCATION_NO` populated.<br>`DNWORKLIST` row finalized.<br>`DNHOSTSEND` row inserted (`REPORT_FLAG=0`).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` **deleted**. |

##Post-Conditions / Assertions
- `dncarryinfo`, `dnpallet`, `dnstock` for `P-C001` removed.
- `dnhostsend` row present and unreported.
- Source `dmshelf` row `STATUS_FLAG=0`.
- No `dninoutresult` row (only created for 1301-1303).

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C002 — Unit Retrieval to ST 1205-1209 (Flow 2 STV hop)</span>

**Objective:** Same end-to-end path as C001 but exercising the Flow 2 STV ID64 hop (9001-9014, not through crane 7-10).

##Pre-Conditions
- Pallet `P-C002` resides at a shelf served by `9001-9014`.
- Destination station = `1207`.

##Test Steps (delta vs C001)

| #  | Step | Expected Result |
|----|------|-----------------|
| 1-6 | Same as C001 steps 1-6 (Set F2 → ID12 → ID32 → ID33). DEST/END station = 1207. | DB rows identical to C001 with `DEST_STATION_NO=1207`. |
| 7  | Inject **ID64** at STV (one of 9001-9014 — pallet picked from rack).                  | `DNCARRYINFO.CMD_STATUS=4` (Pickup completed). |
| 8  | Inject **ID68** at 1207.                                                              | `DNOPERATIONDISPLAY` inserted; LED at 1207 displays work info. |
| 9  | Operator Completion → **ID26**.                                                       | Same RetrievalStationOperator outcome as C001 step 8 (delete pallet/carry/stock; insert hostsend). |

##Post-Conditions
- Identical to C001 except `DEST_STATION_NO=1207`.

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C003 — Unit Retrieval to ST 1301/1302 (ID63 mode change + Flow 2)</span>

**Objective:** Exercise the Bi-Direction station path: ID63 switches station to Retrieval Mode, then a Unit Retrieval at 1301/1302 closes through `InOutStationOperator` (not `RetrievalStationOperator`) and writes a `DNINOUTRESULT` row.

##Pre-Conditions
- `DMSTATION.CURRENT_MODE` for `1301` = **1: Storage Mode** initially (to exercise ID63).
- Pallet `P-C003` resides at a shelf served by `9001-9014`.
- Destination station = `1301`.

##Test Steps

| #  | Step | Expected Result |
|----|------|-----------------|
| 1  | Operator presses **Retrieval Mode** button on 1301 operation box → **ID63** received. | `Id63Process` updates `DMSTATION.CURRENT_MODE=2` (Retrieval) for `STATION_NO=1301`. |
| 2  | Open `LocationRetrievalSCH`, choose Pallet `P-C003`, Station `1301`, Removal Type = **Unit Retrieval**, press **Set (F2)**. | Validations pass (station mode == Retrieval; otherwise screen rejects). DB writes match C001 step 3 with `DEST_STATION_NO=1301`. |
| 3  | ID12 → ID32 → ID33 sequence (same as C001 steps 4-6).                                 | Identical lifecycle transitions. |
| 4  | Inject **ID64** at STV (Flow 2 — 1301/1302 do not use crane 7-10).                    | `DNCARRYINFO.CMD_STATUS=4`. |
| 5  | Inject **ID68** at 1301.                                                              | `DNOPERATIONDISPLAY` inserted; LED shows work. |
| 6  | Operator Completion → **ID26**.                                                       | Path = `InOutStationOperator.arrival.updateArrival` (1301-1303 branch) → `ReturnStorageManager.updateForReturnStorageArrival` → `CarryCompleteOperator.completeRetrievalWorkAndUpdateStock`.<br>`DNWORKINFO.STATUS_FLAG=4`, result fields populated.<br>`DNWORKLIST`/`DMSHELF` updated; `DNHOSTSEND` inserted.<br>**`DNINOUTRESULT` inserted** with `RESULT_KIND=2`, `STATION_NO=1301`, `LOCATION_NO=DNPALLET.CURRENT_STATION_NO`, `WH_STATION_NO=DNPALLET.WH_STATION_NO`, `AISLE_STATION_NO=DNCARRYINFO.AISLE_STATION_NO`, `REMOVE_FLAG=00`.<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed (via `AsStockController.retrieval`). |

##Negative Sub-Case (C003.N1)
- Repeat step 2 while `DMSTATION.CURRENT_MODE=1` (skip ID63 step). **Expected:** screen-level validation rejects with "station mode must be Retrieval"; no `DNWORKINFO`/`DNCARRYINFO` rows are created.

##Post-Conditions
- `DMSTATION.CURRENT_MODE` for `1301` = 2.
- One `DNINOUTRESULT` row created.
- `DNHOSTSEND.REPORT_FLAG=0`.

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C004 — Unit Retrieval to ST 1303 via crane 7-10 (Flow 1)</span>

**Objective:** Verify the full crane-7-10 hop for 1303 when source is at WH9100 aisle (9001-9006 or 9011-9014): ID33 → ID64 STV → ID26 at 7207-7210 → RetrievalSender → ID25 → ID64 SRM → ID64 STV from 9007-9010 → ID68 → ID26 final.

##Pre-Conditions
- `DMSTATION.CURRENT_MODE` for `1303` = 2 (Retrieval) — execute ID63 first if needed.
- Pallet `P-C004` resides at an aisle in `9001-9006` or `9011-9014`.
- Destination station = `1303`.
- Transfer route entries exist for the source aisle → `720x` → `9007-9010` → `1303`.

##Test Steps

| #  | Step | Expected Result |
|----|------|-----------------|
| 1  | (If needed) Trigger **ID63** for 1303. | `DMSTATION.CURRENT_MODE=2`. |
| 2  | Set(F2) on `LocationRetrievalSCH` for `P-C004` → station 1303, Removal Type = Unit Retrieval. | Per the **For 1303** carry definition: `DNCARRYINFO.WORK_TYPE=26` (Direct Transfer), `CARRY_FLAG=3` (Direct Transfer), `DEST_STATION_NO ∈ {7207,7208,7209,7210}`, `END_STATION_NO=DEST_STATION_NO`. |
| 3  | RetrievalSender → ID12 → ID32 → ID33.                                                 | Standard lifecycle. After ID33: `DMSHELF.STATUS_FLAG=0` for source. |
| 4  | Inject **ID64 at STV** (pickup at 9001-9006 / 9011-9014).                             | `DNCARRYINFO.CMD_STATUS=4`. |
| 5  | Pallet arrives at conveyor station 7207-7210 → **ID26**.                              | `DNARRIVAL` inserted (`CARRY_KEY=99999999`, `SEND_FLAG=0`, height/width from ID26). `DNCARRYINFO` updated: `WORK_TYPE=23`, `CARRY_FLAG=2`, `SOURCE_STATION_NO=DNARRIVAL.STATION_NO`, `DEST_STATION_NO=1303`. |
| 6  | RetrievalSender @ 7207-7210 → **ID12** dispatched.                                    | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`. |
| 7  | Inject **ID25** (AGC acceptance for 7207-7210 → SRM).                                  | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`. `DNARRIVAL` row deleted. |
| 8  | Inject **ID64 at SRM** (pickup from 7207-7210).                                        | `DNCARRYINFO.CMD_STATUS=4`. |
| 9  | Inject **ID64 at STV** from 9007-9010 (transit through aisle 7-10).                   | `DNCARRYINFO.CMD_STATUS=4`. |
| 10 | Inject **ID68** at 1303.                                                              | `DNOPERATIONDISPLAY` inserted. |
| 11 | Operator Completion → **ID26**.                                                       | `InOutStationOperator` path (1301-1303 branch). Same final-state assertions as C003 step 6, with `DEST_STATION_NO=1303` and `DNINOUTRESULT.STATION_NO=1303`. |

##Post-Conditions
- No `DNARRIVAL` rows remain for the test BCR.
- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for `P-C004` deleted.
- One `DNINOUTRESULT` and one `DNHOSTSEND` row inserted.

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C005 — Unit Retrieval to ST 1303 direct from 9007-9010</span>

**Objective:** Verify the alternate 1303 path where the source aisle is already 9007-9010 — no crane 7-10 hop required; behaves like Flow 2 + InOutStationOperator completion.

##Pre-Conditions
- `DMSTATION.CURRENT_MODE` for `1303` = 2.
- Pallet `P-C005` resides at an aisle in `9007-9010`.
- Destination station = `1303`.

##Test Steps

| #  | Step | Expected Result |
|----|------|-----------------|
| 1-3 | Set(F2) → ID12 → ID32 → ID33 (same shape as C002 with `DEST_STATION_NO=1303`). NOTE: per DFD the **For 1303** carry block is used, so `WORK_TYPE=26`, `CARRY_FLAG=3`, but no 720x hop is needed. | DB transitions match the **For 1303** DNCarryInfo definition; final `DEST_STATION_NO=1303`. |
| 4  | Inject **ID64 at STV** from 9007-9010 directly.                                        | `DNCARRYINFO.CMD_STATUS=4`. |
| 5  | Inject **ID68** at 1303.                                                              | `DNOPERATIONDISPLAY` inserted. |
| 6  | Operator Completion → **ID26**.                                                       | `InOutStationOperator` path. `DNINOUTRESULT` inserted with `STATION_NO=1303`. Pallet/carry/stock removed; hostsend inserted. |

##Post-Conditions
- No `DNARRIVAL` row was ever created (no 7207-7210 hop).
- Exactly one `DNINOUTRESULT` row.

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C006 — Only Confirmation (Inventory Check) at ST 1301/1302</span>

**Objective:** Verify that selecting **Only Confirmation** (`REMOVAL_TYPE=2`, `RETRIEVAL_QTY=0`) treats the operation as Inventory Check (`JOB_TYPE=40`) and after ID45 the workflow continues into **Unplanned Storage from HP** (not into ID26 completion).

##Pre-Conditions
- `DMSTATION.CURRENT_MODE` for `1301` = 2.
- Pallet `P-C006` resides at an aisle in `9001-9014`.
- Destination station = `1301`.

##Test Steps

| #  | Step | Expected Result |
|----|------|-----------------|
| 1  | Set(F2): Pallet `P-C006`, Station `1301`, Removal Type = **Only Confirmation**, Retrieval Qty = 0. | `DNWORKINFO`: inserted with `JOB_TYPE=40`, `REMOVAL_TYPE=2`, `PLAN_QTY=0`.<br>`DNWORKLIST` inserted.<br>`DNCARRYINFO` inserted with `WORK_TYPE=40`, `CARRY_FLAG=2`, `DEST_STATION_NO=1301`.<br>`DNPALLET.STATUS_FLAG=3`.<br>`DNSTOCKHISTORY` row inserted. |
| 2  | RetrievalSender → ID12 → ID32 → ID33 → ID64 STV → ID68 at 1301.                       | Standard lifecycle through ID68; LED shows Inventory Check job. `DNOPERATIONDISPLAY` inserted. |
| 3  | Operator presses **Complete** at Work Display → **ID45** sent (`As21Id45`).           | Completion button at 1301 control box starts blinking. **No ID26 RetrievalStationOperator path is taken.** |
| 4  | Operator presses blinking Completion button. | Workflow transitions to **Unplanned Storage from HP** (see [Unplanned Storage from HP](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-HP)). Validate per that DFD's assertions. |

##Post-Conditions
- `DNWORKINFO.JOB_TYPE=40` row persists into the Unplanned Storage flow (not deleted by retrieval completion).
- Pallet remains physically at 1301 until Unplanned Storage finishes.
- Continue verification with the Unplanned Storage from HP test cases.

<hr>

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C007 — Only Confirmation (Inventory Check) at ST 1303</span>

**Objective:** Same as C006 but at 1303 — after ID45 the workflow continues into **Unplanned Storage from OP-Area**.

##Pre-Conditions
- `DMSTATION.CURRENT_MODE` for `1303` = 2.
- Pallet `P-C007` source aisle = `9001-9006` or `9011-9014` (exercises Flow 1 crane hop).
- Destination station = `1303`.

##Test Steps

| #  | Step | Expected Result |
|----|------|-----------------|
| 1  | Set(F2): Pallet `P-C007`, Station `1303`, Removal Type = **Only Confirmation**, Retrieval Qty = 0. | `DNWORKINFO.JOB_TYPE=40`, `REMOVAL_TYPE=2`, `PLAN_QTY=0`.<br>`DNCARRYINFO` uses the **For 1303** carry definition: `WORK_TYPE=26`, `CARRY_FLAG=3`, `DEST_STATION_NO ∈ {7207..7210}`. |
| 2  | Run the full Flow 1 chain (ID12 → ID32 → ID33 → ID64 STV → ID26 @ 7207-7210 → RetrievalSender → ID25 → ID64 SRM → ID64 STV from 9007-9010 → ID68 at 1303). | All transitions per C004 steps 3-10. |
| 3  | Operator **Complete** at Work Display → **ID45** sent.                                | Completion button at 1303 control box starts blinking. **ID26 RetrievalStationOperator path NOT taken.** |
| 4  | Operator presses blinking Completion button. | Workflow transitions to **Unplanned Storage from OP-Area** (see [Unplanned Storage from OP-Area](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-OP-Area)). |

##Post-Conditions
- `DNWORKINFO.JOB_TYPE=40` row persists into the Unplanned Storage from OP-Area flow.
- Continue verification with that DFD's test cases.

<hr>

#<span style="color:skyblue; font-weight:bold">Validation / Negative Test Cases</span>

These exercise the validation rules listed on the Specific Location Retrieval Setting screen.

| TC ID         | Pre-Condition Violated                                                       | Expected Result |
|---------------|------------------------------------------------------------------------------|-----------------|
| TC-SLRS-N001  | `DMGroupController.STATUS_FLAG != ONLINE`                                    | Screen blocks Set(F2) with "AGC offline" message; no DB writes. |
| TC-SLRS-N002  | Bi-Direction station (1301-1303) in Storage Mode (`CURRENT_MODE=1`)          | Screen blocks Set(F2) with mode error; no DB writes. |
| TC-SLRS-N003  | `DMStation.SUSPEND=ON` for selected destination station                      | Validation rejects; no DB writes. |
| TC-SLRS-N004  | `DMStation.STATUS != NORMAL` or `DMMachine.STATUS_FLAG != ACTIVE`            | Validation rejects; no DB writes. |
| TC-SLRS-N005  | Daily Update is running                                                      | Validation rejects with "Daily Update in progress". |
| TC-SLRS-N006  | Retrieval/Transport Data Clear Flag in progress                              | Validation rejects. |
| TC-SLRS-N007  | Inventory-to-Retrieval already allocated for target stock                    | Validation rejects with "stock already reserved". |
| TC-SLRS-N008  | Shelf `STATUS_FLAG ∈ {NG, Prohibited, Reserved for Storage, Empty}`          | Validation rejects; pallet not eligible. |
| TC-SLRS-N009  | Mandatory (red-asterisk) input left empty                                    | Field-level validation triggers; submit blocked. |
| TC-SLRS-N010  | Only Confirmation chosen but `Retrieval Qty != 0`                            | Screen forces Qty=0 or rejects (per `DNWORKINFO.REMOVAL_TYPE` definition). |

<hr>

#<span style="color:skyblue; font-weight:bold">Data Cleanup (after each TC)</span>

```sql
-- restore station to Storage Mode if test toggled it
UPDATE dmstation SET current_mode = 1 WHERE station_no IN (1301,1302,1303) AND :restore = 1;

-- if a test failed mid-flow, manually clean rows for the test pallet
DELETE FROM dninoutresult  WHERE pallet_id = :pallet_id;
DELETE FROM dnhostsend     WHERE job_no IN (SELECT job_no FROM dnworkinfo WHERE plan_ukey = :plan_ukey);
DELETE FROM dnworklist     WHERE job_no IN (SELECT job_no FROM dnworkinfo WHERE plan_ukey = :plan_ukey);
DELETE FROM dnworkinfo     WHERE plan_ukey = :plan_ukey;
DELETE FROM dncarryinfo    WHERE pallet_id = :pallet_id;
DELETE FROM dnarrival      WHERE bcr_data  = :bcr_data;
-- re-seed pallet/stock from the appropriate 3.initdata fixture if needed
COMMIT;
```

<hr>

#<span style="color:skyblue; font-weight:bold">Traceability</span>

| Source Section                                            | Covered By                                |
|-----------------------------------------------------------|-------------------------------------------|
| Stage 1 — ID63                                            | C003, C004, C006, C007, N002              |
| Stage 2 — Set(F2) `LocationRetrievalSCH`                  | C001-C007, N001-N010                      |
| Normal Flow — RetrievalSender / ID12 / ID32 / ID33        | C001-C007                                 |
| Flow 1 — through crane 7-10 (1303)                        | C004, C007                                |
| Flow 2 — not through crane 7-10                           | C002, C003, C006                          |
| Flow 3 — direct to ID68 (1201-1204)                       | C001                                      |
| Flow 4 — Only Confirmation → ID45                         | C006 (1301/1302), C007 (1303)             |
| Last Process — ID26 (RetrievalStationOperator)            | C001, C002                                |
| Last Process — ID26 (InOutStationOperator + DNINOUTRESULT)| C003, C004, C005                          |
| Continue Process — Unplanned Storage from HP / OP-Area    | C006 → HP, C007 → OP-Area                 |

#Related
- [Specific Location Retrieval Setting (DFD)](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting)
- [Unplanned Storage from HP](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-HP)
- [Unplanned Storage from OP-Area](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-OP-Area)
- [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)
