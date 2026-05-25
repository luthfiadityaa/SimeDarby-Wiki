[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-URHP-C001 — Unplanned Retrieval to ST 1207 (direct, no ID63)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Unplanned Retrieval to HP Area](/Basic-Design/DFD-WMS/Retrieval/Unplanned-Retrieval-to-HP-Area) |
| **Screen**       | `jp.co.daifuku.wms.web.display.retrieval.InquiryRetrievalSettingSCH`                          |
| **Job Type**     | Unplanned Retrieval (`JOB_TYPE=23`, `REMOVAL_TYPE=1`)                                         |
| **Destination**  | 1207 (Unidirectional HP retrieval station)                                                    |
| **AGC Route**    | RetrievalSender → ID12 → ID32 → ID33 → ID64 → ID68 → ID26 (`RetrievalStationOperator`)        |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify a manual Unplanned Retrieval issued via `InquiryRetrievalSettingSCH` to a non-Bi-Direction station (1207) executes the full ID12 → ID32 → ID33 → ID64 → ID68 → ID26 chain with correct DB transitions and **without** invoking `Id63Process`.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE` (`DMGroupController.STATUS_FLAG = ONLINE`).
- 1207 station `NORMAL`, `SUSPEND=OFF`, machine `ACTIVE`.
- Daily Update and Retrieval/Transport Data Clear Flag not in progress.
- Pallet **P-URHP-001** present at a shelf served by aisle `9001-9014` in the WH that owns 1207.
- Source `DMSHELF.STATUS_FLAG` not in `NG / Prohibited / Reserved for Storage / Empty`.
- Stock not already locked by Inventory-to-Retrieval allocation.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-URHP-001             |
| Destination Station| 1207                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9014              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                                                              | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open **Inquiry Retrieval Setting** screen (`InquiryRetrievalSettingSCH`).             | Screen loads; mandatory red-asterisk fields visible.                                                                                                                                                                                                                                                                                                          |               |                      |
| 2  | Select Pallet `P-URHP-001`, Station `1207`, Removal Type = Unit Retrieval, fill mandatory fields. Press **Set (F2)**. | Validations pass. DB writes:<br>`DNWORKINFO` inserted (`JOB_TYPE=23`, `STATUS_FLAG=0`, `SYSTEM_CONN_KEY=DNCARRYINFO.CARRY_KEY`).<br>`DNWORKLIST` inserted (`RETRIEVAL_DETAIL=1`).<br>`DNCARRYINFO` inserted (`WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `DEST_STATION_NO=1207`, `END_STATION_NO=1207`, `AISLE_STATION_NO=DMSHELF.PARENT_STATION_NO`).<br>`DNPALLET.STATUS_FLAG=3`. |               |                      |
| 3  | `RetrievalSender` picks up carry → **ID12** dispatched.                                | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`. ID12 frame observed.                                                                                                                                                                                                                                                                                  |               |                      |
| 4  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                                                                                                                  |               |                      |
| 5  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0` (Empty). `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO=DMSHELF.STATION_NO`.                                                                                                                                                                                                                                              |               |                      |
| 6  | Inject **ID64 at STV** (pickup at source aisle).                                       | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                                  |               |                      |
| 7  | Inject **ID68** at 1207.                                                              | `DNOPERATIONDISPLAY` inserted (`STATION_NO=1207`, material/qty/dock/truck from `DNWORKINFO`). LED at 1207 shows work info.                                                                                                                                                                                                                                  |               |                      |
| 8  | Operator presses **Completion** at 1207 → **ID26** received.                           | Path = `RetrievalStationOperator` (1201-1209 branch).<br>`DNWORKINFO.STATUS_FLAG=4`, `RESULT_QTY=PLAN_QTY`, `RESULT_AREA_NO/LOCATION_NO` populated.<br>`DNWORKLIST` finalized.<br>`DNHOSTSEND` inserted (`REPORT_FLAG=0`).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-URHP-001 **deleted**.                                                              |               |                      |
| 9  | Verify **no** ID63 frame was observed at any step.                                     | `Id63Process` not invoked for 1207 (non-Bi-Direction). `DMSTATION.CURRENT_MODE` for 1207 not relevant / unchanged.                                                                                                                                                                                                                                          |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT job_no, job_type, status_flag, system_conn_key FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT carry_key, work_type, carry_flag, cmd_status, dest_station_no, end_station_no, aisle_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-URHP-001';

-- after step 8 — pallet/carry/stock removed; hostsend present
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-URHP-001';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-URHP-001';   -- expect 0
SELECT COUNT(*) FROM dnstock      WHERE pallet_id = 'P-URHP-001';   -- expect 0
SELECT job_no, status_flag, result_qty, report_flag FROM dnhostsend WHERE job_no = :job_no;
SELECT COUNT(*) FROM dninoutresult WHERE pallet_id = 'P-URHP-001';  -- expect 0 (1207 not in 1301-1303)
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` rows for P-URHP-001 removed.
- One `DNHOSTSEND` row inserted, `REPORT_FLAG=0`.
- Source `DMSHELF.STATUS_FLAG=0`.
- **No** `DNINOUTRESULT` row (1207 is not Bi-Direction).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
