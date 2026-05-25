[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C002 — Unit Retrieval to ST 1205-1209 (Flow 2 STV hop)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | Unit Retrieval (`REMOVAL_TYPE=1`, `JOB_TYPE=23`)                                              |
| **Destination**  | 1205 / 1206 / 1207 / 1208 / 1209                                                              |
| **AGC Route**    | Flow 2 — STV ID64 from 9001-9014 (no aisle 7-10 hop)                                          |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify a Unit Retrieval to a 1205-1209 destination exercises the Flow 2 path (STV ID64 from 9001-9014) and completes through `RetrievalStationOperator` on ID26.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; destination station `NORMAL`, not suspended; machine `ACTIVE`.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-C002** resides at a shelf served by an aisle in `9001-9014`.
- Source `DMSHELF.STATUS_FLAG` not in `NG / Prohibited / Reserved for Storage / Empty`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-C002                 |
| Destination Station| 1207                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9014              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                  | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open `LocationRetrievalSCH`.                                                          | Screen loads.                                                                                                                                                                                                                                                                                                    |               |                      |
| 2  | Select Pallet `P-C002`, Station `1207`, Removal Type = **Unit Retrieval**, Priority = Normal. Press **Set (F2)**. | Validations pass.                                                                                                                                                                                                                                                                                                |               |                      |
| 3  | Query DB after Set(F2).                                                               | `DNWORKINFO` inserted (`JOB_TYPE=23`, `STATUS_FLAG=0`, `REMOVAL_TYPE=1`).<br>`DNWORKLIST` inserted (`RETRIEVAL_DETAIL=1`).<br>`DNCARRYINFO` inserted (`WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1207`, `END_STATION_NO=1207`).<br>`DNPALLET.STATUS_FLAG=3`.<br>`DNSTOCKHISTORY` inserted. |               |                      |
| 4  | `RetrievalSender` → **ID12** dispatched.                                              | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                                                                            |               |                      |
| 5  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                                                                      |               |                      |
| 6  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO=DMSHELF.STATION_NO`.                                                                                                                                                                                                          |               |                      |
| 7  | Inject **ID64 at STV** (pickup at 9001-9014).                                          | `DNCARRYINFO.CMD_STATUS=4` (Pickup completed).                                                                                                                                                                                                                                                                    |               |                      |
| 8  | Inject **ID68** at 1207.                                                              | `DNOPERATIONDISPLAY` inserted (`STATION_NO=1207`, `MATERIAL_CODE`, `RETRIEVAL_QTY`, `DOCK_NO`, `TRUCK_PLATE_NO`). LED at 1207 shows work info.                                                                                                                                                                  |               |                      |
| 9  | Operator presses **Completion** → **ID26**.                                            | Path = `RetrievalStationOperator` (1201-1209 branch).<br>`DNWORKINFO.STATUS_FLAG=4`, `RESULT_QTY=PLAN_QTY`.<br>`DNWORKLIST` finalized. `DNHOSTSEND` inserted, `REPORT_FLAG=0`.<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` deleted.                                                                                  |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
SELECT carry_key, cmd_status, source_station_no, dest_station_no, aisle_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-C002';
SELECT status_flag FROM dnpallet WHERE pallet_id = 'P-C002';
SELECT COUNT(*) FROM dninoutresult WHERE pallet_id = 'P-C002';   -- expect 0 (not 1301-1303)
SELECT job_no, status_flag, report_flag FROM dnhostsend WHERE job_no = :job_no;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed.
- `DNHOSTSEND` inserted, unreported.
- **No** `DNINOUTRESULT` row (1207 is not in 1301-1303 range).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
