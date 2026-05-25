[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C001 — Unit Retrieval to ST 1201-1204 (Flow 3 direct)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | Unit Retrieval (`REMOVAL_TYPE=1`, `JOB_TYPE=23`)                                              |
| **Destination**  | 1201 / 1202 / 1203 / 1204                                                                     |
| **AGC Route**    | Flow 3 — direct to ID68 (no aisle 7-10 hop)                                                   |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify a Unit Retrieval issued from `LocationRetrievalSCH` to an Ambient/Tempering station 1201-1204 flows correctly through **ID12 → ID32 → ID33 → ID68 → ID26** and that all DB transitions occur without an aisle 7-10 hop.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE` (`DMGroupController.STATUS_FLAG = ONLINE`).
- Destination station (1201-1204) is `NORMAL`, `SUSPEND=OFF`, machine `ACTIVE`.
- Daily Update and Data Clear Flag are NOT in progress.
- Pallet **P-C001** resides at a shelf served by an aisle station in `9001-9006` or `9011-9014` (within the WH that owns 1201-1204).
- `DNSTOCK` row exists for P-C001; not reserved by Inventory-to-Retrieval.
- Source `DMSHELF.STATUS_FLAG` is **not** in `NG / Prohibited / Reserved for Storage / Empty`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-C001                 |
| Destination Station| 1201                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9006 / 9011-9014  |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                                                                                                                                                | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open **Specific Location Retrieval Setting** screen (`LocationRetrievalSCH`).         | Screen loads; mandatory red-asterisk fields visible.                                                                                                                                                                                                                                                                                                                                                                                          |               |                      |
| 2  | Select Pallet `P-C001`, Station `1201`, Removal Type = **Unit Retrieval**, Priority = Normal. Press **Set (F2)**. | Validations pass. `WebUnplannedRetrievalScheduler.schedule()` invoked.                                                                                                                                                                                                                                                                                                                                                                |               |                      |
| 3  | Query DB right after Set(F2).                                                         | `DNWORKINFO`: row inserted, `JOB_TYPE=23`, `STATUS_FLAG=0`, `REMOVAL_TYPE=1`, `PLAN_QTY=Stock Qty`.<br>`DNWORKLIST`: row inserted, `RETRIEVAL_DETAIL=1`.<br>`DNCARRYINFO`: inserted, `WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1201`, `END_STATION_NO=1201`, `AISLE_STATION_NO=DMSHELF.PARENT_STATION_NO`.<br>`DNPALLET.STATUS_FLAG=3` (Reserved for Retrieval).<br>`DNSTOCKHISTORY` row inserted, `LAST_UPDATE_PNAME=LocationRetrievalSCH`. |               |                      |
| 4  | `RetrievalSender` processes carry → AS21 **ID12** dispatched.                         | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4` (Being retrieved). ID12 frame observed on link.                                                                                                                                                                                                                                                                                                                                          |               |                      |
| 5  | Inject **ID32** (AGC job acceptance).                                                 | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                                                                                                                                                                                                   |               |                      |
| 6  | Inject **ID33** (pallet out of rack).                                                 | Source `DMSHELF.STATUS_FLAG=0` (Empty). `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO=DMSHELF.STATION_NO`.                                                                                                                                                                                                                                                                                                                                |               |                      |
| 7  | Inject **ID68** (pallet arrives at 1201).                                             | `DNOPERATIONDISPLAY` inserted with `STATION_NO=1201`, `MATERIAL_CODE`, `RETRIEVAL_QTY`, `DOCK_NO`, `TRUCK_PLATE_NO` from `DNWORKINFO`. LED at 1201 shows Job Type / Material Code / Qty / Dock / Truck.                                                                                                                                                                                                                                       |               |                      |
| 8  | Operator presses **Completion** button at 1201 terminal → **ID26** received.          | Path = `RetrievalStationOperator` (1201-1209 branch).<br>`DNWORKINFO.STATUS_FLAG=4`, `RESULT_QTY=PLAN_QTY`, `RESULT_AREA_NO/LOCATION_NO` populated.<br>`DNWORKLIST` finalized.<br>`DNHOSTSEND` row inserted, `REPORT_FLAG=0`.<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-C001 **deleted**.                                                                                                                                                  |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 3
SELECT job_no, job_type, status_flag, removal_type, plan_qty FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT carry_key, work_type, carry_flag, cmd_status, dest_station_no, aisle_station_no, end_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-C001';
SELECT status_flag FROM dnpallet WHERE pallet_id = 'P-C001';

-- after step 8 — pallet and carry should be gone
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-C001';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-C001';   -- expect 0
SELECT COUNT(*) FROM dnstock      WHERE pallet_id = 'P-C001';   -- expect 0
SELECT job_no, status_flag, result_qty, report_flag FROM dnhostsend WHERE job_no = :job_no;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `dncarryinfo`, `dnpallet`, `dnstock` rows for P-C001 removed.
- One `dnhostsend` row present, `REPORT_FLAG=0`.
- Source `dmshelf` row `STATUS_FLAG=0`.
- **No** `dninoutresult` row created (this table is populated only for 1301-1303).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

_Record any deviation, screenshot reference, defect ID, or observation here._

```
[                                                                                ]
[                                                                                ]
```
