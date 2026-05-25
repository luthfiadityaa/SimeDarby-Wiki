[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C007 — Only Confirmation (Inventory Check) at ST 1303</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | **Only Confirmation** (`REMOVAL_TYPE=2`, `RETRIEVAL_QTY=0`, `JOB_TYPE=40` Inventory Check)    |
| **Destination**  | 1303                                                                                          |
| **AGC Route**    | **ID63** → Flow 1 (crane 7-10 hop) → ID68 → **ID45** → continues into **Unplanned Storage from OP-Area** |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Same as TC-SLRS-C006 but for 1303 — verify that after ID45 the workflow continues into **Unplanned Storage from OP-Area** (not HP). Exercises the Flow 1 crane 7-10 hop end-to-end.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1303 station `NORMAL`, not suspended; machine `ACTIVE`.
- `DMSTATION.CURRENT_MODE` for `1303` = 2.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-C007** resides at an aisle in `9001-9006` or `9011-9014` (so Flow 1 crane hop is exercised).
- Transfer route entries exist: source aisle → `720x` → `9007-9010` → `1303`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                                |
|--------------------|--------------------------------------|
| Pallet ID          | P-C007                               |
| Destination Station| 1303                                 |
| Removal Type       | **Only Confirmation**                |
| Retrieval Qty      | 0                                    |
| Priority           | Normal                               |
| Source Aisle ST    | 9001-9006 or 9011-9014               |
| Conveyor Hop ST    | 7207-7210                            |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Set(F2) on `LocationRetrievalSCH`: Pallet `P-C007`, Station `1303`, Removal Type = **Only Confirmation**, Retrieval Qty = 0. | `DNWORKINFO`: `JOB_TYPE=40`, `REMOVAL_TYPE=2`, `PLAN_QTY=0`.<br>`DNCARRYINFO` uses the **For 1303** definition: `WORK_TYPE=26`, `CARRY_FLAG=3`, `DEST_STATION_NO ∈ {7207..7210}`, `END_STATION_NO = DEST_STATION_NO`.<br>`DNPALLET.STATUS_FLAG=3`. |               |                      |
| 2  | `RetrievalSender` → **ID12** → **ID32** → **ID33**.                                    | Standard lifecycle. Source `DMSHELF.STATUS_FLAG=0` after ID33.                                                                                                                                                                                                |               |                      |
| 3  | Inject **ID64 at STV** (pickup at 9001-9006 / 9011-9014).                              | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                    |               |                      |
| 4  | Pallet arrives at 7207-7210 → **ID26**.                                                | `DNARRIVAL` inserted. `DNCARRYINFO` updated: `WORK_TYPE=23`, `CARRY_FLAG=2`, `SOURCE_STATION_NO=DNARRIVAL.STATION_NO`, `DEST_STATION_NO=1303`, `CMD_STATUS=1`.                                                                                                |               |                      |
| 5  | `RetrievalSender` at 7207-7210 → **ID12**.                                              | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                          |               |                      |
| 6  | Inject **ID25**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`. `DNARRIVAL` deleted.                                                                                                                                                                                                                |               |                      |
| 7  | Inject **ID64 at SRM** (pickup from 7207-7210).                                         | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                    |               |                      |
| 8  | Inject **ID64 at STV** from 9007-9010.                                                  | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                    |               |                      |
| 9  | Inject **ID68** at 1303.                                                              | `DNOPERATIONDISPLAY` inserted; LED shows Inventory Check work.                                                                                                                                                                                                 |               |                      |
| 10 | Operator presses **Complete** on Work Display → **ID45** sent.                          | Completion button at 1303 control box **blinks**. ID26 `RetrievalStationOperator` path **NOT** taken; `DNWORKINFO`, `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` persist.                                                                                              |               |                      |
| 11 | Operator presses blinking Completion button.                                            | Workflow transitions to **Unplanned Storage from OP-Area** (see [Unplanned Storage from OP-Area](/Basic-Design/DFD-WMS/Storage/Unplanned-Storage-from-OP-Area)). Continue verification with that DFD's test cases.                                              |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 1
SELECT job_type, removal_type, plan_qty FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT work_type, carry_flag, dest_station_no, end_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-C007';

-- after step 6
SELECT COUNT(*) FROM dnarrival WHERE bcr_data = :bcr;   -- expect 0

-- after step 10 (before pressing blinking button)
SELECT status_flag FROM dnworkinfo WHERE plan_ukey = :plan_ukey;    -- STATUS_FLAG != 4
SELECT COUNT(*) FROM dnpallet WHERE pallet_id = 'P-C007';            -- expect 1
SELECT COUNT(*) FROM dnhostsend WHERE job_no = :job_no;              -- expect 0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DNWORKINFO.JOB_TYPE=40` row persists into the Unplanned Storage from OP-Area flow.
- Pallet P-C007 remains physically at 1303 until Unplanned Storage finishes.
- **No** `DNINOUTRESULT` row was created by this TC (it is created later by Unplanned Storage from OP-Area).
- Continue verification with the Unplanned Storage from OP-Area test cases.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
