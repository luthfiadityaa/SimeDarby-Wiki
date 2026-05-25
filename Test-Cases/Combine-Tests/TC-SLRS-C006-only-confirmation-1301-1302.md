[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C006 — Only Confirmation (Inventory Check) at ST 1301/1302</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | **Only Confirmation** (`REMOVAL_TYPE=2`, `RETRIEVAL_QTY=0`, `JOB_TYPE=40` Inventory Check)    |
| **Destination**  | 1301 / 1302                                                                                   |
| **AGC Route**    | **ID63** → Flow 2 → ID68 → **ID45** → continues into **Unplanned Storage from HP**            |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify that selecting **Only Confirmation** at 1301/1302 is treated as Inventory Check (`JOB_TYPE=40`), and after ID45 the workflow transitions into **Unplanned Storage from HP** — i.e. the `RetrievalStationOperator` ID26 path is **NOT** taken.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1301 `NORMAL`, not suspended; machine `ACTIVE`.
- `DMSTATION.CURRENT_MODE` for `1301` = 2 (Retrieval).
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-C006** resides at an aisle in `9001-9014`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-C006                 |
| Destination Station| 1301                   |
| Removal Type       | **Only Confirmation**  |
| Retrieval Qty      | 0                      |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9014              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                              | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Set(F2) on `LocationRetrievalSCH`: Pallet `P-C006`, Station `1301`, Removal Type = **Only Confirmation**, Retrieval Qty = 0. | `DNWORKINFO` inserted: `JOB_TYPE=40`, `REMOVAL_TYPE=2`, `PLAN_QTY=0`, `STATUS_FLAG=0`.<br>`DNWORKLIST` inserted (`RETRIEVAL_DETAIL=1`).<br>`DNCARRYINFO` inserted: `WORK_TYPE=40`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1301`.<br>`DNPALLET.STATUS_FLAG=3`.<br>`DNSTOCKHISTORY` inserted. |               |                      |
| 2  | `RetrievalSender` → **ID12**.                                                          | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                        |               |                      |
| 3  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                  |               |                      |
| 4  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`.                                                                                                                                                                                                  |               |                      |
| 5  | Inject **ID64 at STV** (Flow 2 — 1301/1302 do not use crane 7-10).                    | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                  |               |                      |
| 6  | Inject **ID68** at 1301.                                                              | `DNOPERATIONDISPLAY` inserted; LED at 1301 shows Inventory Check work.                                                                                                                                                                                       |               |                      |
| 7  | Operator presses **Complete** on Work Display → **ID45** sent (`As21Id45`).            | Completion button at 1301 control box starts **blinking**. **No** ID26 `RetrievalStationOperator` path taken. `DNWORKINFO`, `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` rows **persist** (not deleted by retrieval completion).                                       |               |                      |
| 8  | Operator presses the blinking Completion button.                                       | Workflow transitions to **Unplanned Storage from HP** (see [Unplanned Storage from HP](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-HP)). Continue verification using that DFD's test cases.                                                          |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 1
SELECT job_no, job_type, removal_type, plan_qty, status_flag FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT work_type, carry_flag, cmd_status, dest_station_no FROM dncarryinfo WHERE pallet_id = 'P-C006';

-- after step 7 (before pressing blinking button)
-- ID26 RetrievalStationOperator path should NOT have executed
SELECT status_flag, result_qty FROM dnworkinfo WHERE plan_ukey = :plan_ukey;   -- STATUS_FLAG != 4
SELECT COUNT(*) FROM dnpallet WHERE pallet_id = 'P-C006';                       -- expect 1 (still present)
SELECT COUNT(*) FROM dnhostsend WHERE job_no = :job_no;                         -- expect 0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DNWORKINFO.JOB_TYPE=40` row persists into the Unplanned Storage from HP flow (not deleted by retrieval completion).
- Pallet P-C006 remains physically at 1301 until Unplanned Storage finishes.
- **No** `DNINOUTRESULT` row was created by this TC (it is created later by Unplanned Storage from HP).
- Continue verification with the Unplanned Storage from HP test cases.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
