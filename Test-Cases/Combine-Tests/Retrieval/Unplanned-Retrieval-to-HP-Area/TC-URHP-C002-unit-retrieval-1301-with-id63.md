[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-URHP-C002 — Unplanned Retrieval to ST 1301 (Bi-Direction, ID63 mode switch)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Unplanned Retrieval to HP Area](/Basic-Design/DFD-WMS/Retrieval/Unplanned-Retrieval-to-HP-Area) |
| **Screen**       | `InquiryRetrievalSettingSCH`                                                                  |
| **Job Type**     | Unplanned Retrieval (`JOB_TYPE=23`)                                                           |
| **Destination**  | 1301 (Bi-Direction `STATION_TYPE=3:INOUT`)                                                    |
| **AGC Route**    | **ID63** → RetrievalSender → ID12 → ID32 → ID33 → ID64 → ID68 → ID26 (`InOutStationOperator`) |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the Bi-Direction station retrieval path. Operator presses Retrieval Mode button → ID63 switches `DMSTATION.CURRENT_MODE` from 1 (STORAGE) to 2 (RETRIEVAL). Retrieval then runs through `InOutStationOperator` closure, producing a `DNINOUTRESULT` row distinguishable from the 1205-1209 `RetrievalStationOperator` flow.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1301 station `NORMAL`, not suspended; machine `ACTIVE`.
- `DMSTATION.CURRENT_MODE` for 1301 = **1 (STORAGE)** initially.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-URHP-002** present at a shelf served by aisle `9001-9014`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-URHP-002             |
| Destination Station| 1301                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9001-9014              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                              | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Operator presses **Retrieval Mode** button on 1301 operation box → **ID63** received. | `Id63Process` runs. `DMSTATION.CURRENT_MODE=2` for `STATION_NO=1301`.                                                                                                                                                                          |               |                      |
| 2  | Open `InquiryRetrievalSettingSCH`, select Pallet `P-URHP-002`, Station `1301`, Removal Type = Unit Retrieval. Press **Set (F2)**. | Validations pass (mode == Retrieval).<br>`DNWORKINFO`, `DNWORKLIST`, `DNCARRYINFO` inserted (`DEST_STATION_NO=1301`, `END_STATION_NO=1301`).<br>`DNPALLET.STATUS_FLAG=3`.                                                                       |               |                      |
| 3  | RetrievalSender → **ID12** → **ID32** → **ID33**.                                      | Standard lifecycle. Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`.                                                                                                                                                              |               |                      |
| 4  | Inject **ID64 at STV** from source aisle.                                              | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                  |               |                      |
| 5  | Inject **ID68** at 1301.                                                              | `DNOPERATIONDISPLAY` inserted (`STATION_NO=1301`).                                                                                                                                                                                            |               |                      |
| 6  | Operator presses Completion → **ID26**.                                                | Path = `InOutStationOperator` (1301-1303 branch).<br>`DNWORKINFO.STATUS_FLAG=4`; `DNHOSTSEND` inserted; **`DNINOUTRESULT` inserted** (`RESULT_KIND=2`, `STATION_NO=1301`, `WH_STATION_NO=DNPALLET.WH_STATION_NO`, `REMOVE_FLAG=00`).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed. |               |                      |
| 7  | **Negative sub-case** — repeat from step 2 while `CURRENT_MODE=1`.                     | Screen rejects with "station mode must be Retrieval". No DB writes.                                                                                                                                                                          |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 1
SELECT station_no, current_mode FROM dmstation WHERE station_no = 1301;   -- expect 2

-- after step 6
SELECT result_kind, station_no, location_no, wh_station_no, aisle_station_no, remove_flag
  FROM dninoutresult WHERE pallet_id = 'P-URHP-002';
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-URHP-002';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-URHP-002';   -- expect 0
SELECT job_no, status_flag, report_flag FROM dnhostsend WHERE job_no = :job_no;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- `DMSTATION.CURRENT_MODE` for 1301 = 2.
- One `DNINOUTRESULT` row inserted.
- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-URHP-002 removed.
- One `DNHOSTSEND` row inserted, `REPORT_FLAG=0`.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
