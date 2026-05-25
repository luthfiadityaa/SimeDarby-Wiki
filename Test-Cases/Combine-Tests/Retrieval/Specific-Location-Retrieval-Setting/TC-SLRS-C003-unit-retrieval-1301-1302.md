[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C003 — Unit Retrieval to ST 1301/1302 (ID63 mode change + Flow 2)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | Unit Retrieval (`REMOVAL_TYPE=1`, `JOB_TYPE=23`)                                              |
| **Destination**  | 1301 / 1302 (Bi-Direction)                                                                    |
| **AGC Route**    | **ID63** mode change → Flow 2 (STV ID64 9001-9014) → ID68 → **ID26 InOutStationOperator**     |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the Bi-Direction station path: **ID63** switches station to Retrieval Mode, a Unit Retrieval at 1301/1302 closes through `InOutStationOperator` (not `RetrievalStationOperator`), and a `DNINOUTRESULT` row is created.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; machine `ACTIVE`; station `NORMAL`, not suspended.
- Daily Update and Data Clear Flag not in progress.
- `DMSTATION.CURRENT_MODE` for `1301` = **1: Storage Mode** initially (so step 1 exercises ID63).
- Pallet **P-C003** resides at a shelf served by an aisle in `9001-9014`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-C003                 |
| Destination Station| 1301                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9014              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Operator presses **Retrieval Mode** button on 1301 operation box → **ID63** received. | `Id63Process` runs. `DMSTATION.CURRENT_MODE=2` (Retrieval) for `STATION_NO=1301`.                                                                                                                                                                                                                                                                                                                                                                                                                          |               |                      |
| 2  | Open `LocationRetrievalSCH`, select Pallet `P-C003`, Station `1301`, Removal Type = **Unit Retrieval**, press **Set (F2)**. | Validations pass (mode == Retrieval). DB writes:<br>`DNWORKINFO` inserted (`JOB_TYPE=23`, `STATUS_FLAG=0`, `REMOVAL_TYPE=1`).<br>`DNWORKLIST` inserted.<br>`DNCARRYINFO` inserted (`WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1301`, `END_STATION_NO=1301`, `AISLE_STATION_NO=DMSHELF.PARENT_STATION_NO`).<br>`DNPALLET.STATUS_FLAG=3`.<br>`DNSTOCKHISTORY` inserted.                                                                                              |               |                      |
| 3  | `RetrievalSender` → **ID12** dispatched.                                              | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |               |                      |
| 4  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |               |                      |
| 5  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO=DMSHELF.STATION_NO`.                                                                                                                                                                                                                                                                                                                                                                                                    |               |                      |
| 6  | Inject **ID64 at STV** (Flow 2 — 1301/1302 do not use crane 7-10).                    | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |               |                      |
| 7  | Inject **ID68** at 1301.                                                              | `DNOPERATIONDISPLAY` inserted; LED shows work info.                                                                                                                                                                                                                                                                                                                                                                                                                                                        |               |                      |
| 8  | Operator presses **Completion** → **ID26**.                                            | Path = `InOutStationOperator.arrival.updateArrival` (1301-1303 branch) → `ReturnStorageManager.updateForReturnStorageArrival` → `CarryCompleteOperator.completeRetrievalWorkAndUpdateStock` → `WorkInfoController.completeWorkInfo` + `AsStockController.retrieval`.<br>`DNWORKINFO.STATUS_FLAG=4`, result fields populated.<br>`DNWORKLIST`/`DMSHELF` updated; `DNHOSTSEND` inserted.<br>**`DNINOUTRESULT` inserted** (`RESULT_KIND=2`, `STATION_NO=1301`, `LOCATION_NO=DNPALLET.CURRENT_STATION_NO`, `WH_STATION_NO=DNPALLET.WH_STATION_NO`, `AISLE_STATION_NO=DNCARRYINFO.AISLE_STATION_NO`, `REMOVE_FLAG=00`).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed. |               |                      |
| 9  | **Negative sub-case** — repeat step 2 while `DMSTATION.CURRENT_MODE=1` (skip ID63).   | Screen rejects with "station mode must be Retrieval". No `DNWORKINFO` / `DNCARRYINFO` rows created.                                                                                                                                                                                                                                                                                                                                                                                                        |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 1
SELECT station_no, current_mode FROM dmstation WHERE station_no = 1301;   -- expect current_mode = 2

-- after step 8
SELECT result_kind, station_no, location_no, wh_station_no, aisle_station_no, remove_flag
  FROM dninoutresult WHERE pallet_id = 'P-C003';
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-C003';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-C003';   -- expect 0
SELECT COUNT(*) FROM dnstock      WHERE pallet_id = 'P-C003';   -- expect 0
SELECT job_no, status_flag, report_flag FROM dnhostsend WHERE job_no = :job_no;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DMSTATION.CURRENT_MODE` for `1301` = 2.
- One `DNINOUTRESULT` row created.
- `DNHOSTSEND.REPORT_FLAG=0`.
- Pallet/carry/stock for P-C003 removed.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
