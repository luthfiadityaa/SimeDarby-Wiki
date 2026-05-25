[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-UROP-C001 — Unplanned PKG Retrieval to ST 1201</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Unplanned Retrieval from OP-Area (PKG)](/Basic-Design/DFD-WMS/Retrieval/Unplanned-Retrieval-from-OP-Area-(PKG)) |
| **Screen**       | `InquiryRetrievalSettingSCH` (PKG variant)                                                    |
| **Job Type**     | Unplanned Retrieval (`JOB_TYPE=23`)                                                           |
| **Destination**  | 1201 / 1202 / 1203 / 1204 (OP Packaging — group 1200)                                         |
| **Route Restriction** | Source aisle MUST be in **9011-9014** (PM). 9001-9006 / 9007-9010 have NO route.         |
| **AGC Route**    | RetrievalSender → ID12 → ID32 → ID33 → ID68 → ID26 (`RetrievalStationOperator`) — no ID64 row in DML table |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the manual PKG (Packaging Material) retrieval from a PM aisle (9011-9014) reaches an OP packaging station (1201-1204) with correct DB transitions and that retrievals from non-PM aisles are rejected (route restriction).

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1201 station `NORMAL`, not suspended; machine `ACTIVE`.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-UROP-001** (PKG material, e.g. soft-zone PM) present at a shelf served by aisle in `9011-9014`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-UROP-001             |
| Destination Station| 1201                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9011-9014 (PM only)    |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                            | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open `InquiryRetrievalSettingSCH`.                                                    | Screen loads.                                                                                                                                                                                                                                              |               |                      |
| 2  | Select Pallet `P-UROP-001`, Station `1201`, Removal Type = Unit Retrieval. Press **Set (F2)**. | Validations pass. `DNWORKINFO` inserted (`JOB_TYPE=23`, `STATUS_FLAG=0`, `MATERIAL_CODE=DNSTOCK.MATERIAL_CODE`, `PLAN_QTY=Retrieval Qty`).<br>`DNWORKLIST` inserted.<br>`DNCARRYINFO` inserted (`WORK_TYPE=23`, `CARRY_FLAG=2`, `DEST_STATION_NO=1201`).<br>`DNPALLET.STATUS_FLAG=3`. |               |                      |
| 3  | RetrievalSender → **ID12** dispatched.                                                | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                       |               |                      |
| 4  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                |               |                      |
| 5  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`.                                                                                                                                                                                                |               |                      |
| 6  | Inject **ID68** at 1201.                                                              | `DNOPERATIONRESULT` inserted with `STATION_NO=1201`, material/qty.                                                                                                                                                                                          |               |                      |
| 7  | Operator presses Completion → **ID26**.                                                | Path = `RetrievalStationOperator`.<br>`DNWORKINFO.STATUS_FLAG=4`.<br>`DNWORKLIST` finalized.<br>`DNHOSTSEND` inserted, `REPORT_FLAG=0`.<br>`DNINOUTRESULT` inserted (per OP-PKG DML table).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` deleted.<br>`DMSHELF.STATUS_FLAG=0`. |               |                      |
| 8  | **Negative sub-case** — re-run from step 2 with pallet on aisle `9007` (non-PM).        | Route lookup fails. Screen rejects with "no route to 1201 from this aisle". No DB writes.                                                                                                                                                                  |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT carry_key, work_type, carry_flag, dest_station_no, aisle_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-UROP-001';

-- after step 7
SELECT COUNT(*) FROM dnpallet    WHERE pallet_id = 'P-UROP-001';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo WHERE pallet_id = 'P-UROP-001';   -- expect 0
SELECT job_no, status_flag, report_flag FROM dnhostsend WHERE job_no = :job_no;
SELECT result_kind, station_no FROM dninoutresult WHERE pallet_id = 'P-UROP-001';
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-UROP-001 removed.
- One `DNHOSTSEND` row, `REPORT_FLAG=0`.
- One `DNINOUTRESULT` row (`RESULT_KIND=2`, `STATION_NO=1201`).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
