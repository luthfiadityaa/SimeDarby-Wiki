[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C005 — Unit Retrieval to ST 1303 direct from 9007-9010</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | Unit Retrieval (`REMOVAL_TYPE=1`)                                                             |
| **Destination**  | 1303 (Bi-Direction)                                                                           |
| **AGC Route**    | **ID63** → direct STV ID64 from 9007-9010 → ID68 → ID26 (`InOutStationOperator`)              |
| **Priority**     | Medium                                                                                        |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the alternate 1303 path where the source aisle is already in `9007-9010`. **No** crane 7-10 hop is required, so no `DNARRIVAL` row is ever inserted. Closure still happens through `InOutStationOperator` writing a `DNINOUTRESULT` row.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1303 station `NORMAL`, not suspended; machine `ACTIVE`.
- `DMSTATION.CURRENT_MODE` for `1303` = 2.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-C005** resides at an aisle in `9007-9010`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-C005                 |
| Destination Station| 1303                   |
| Removal Type       | Unit Retrieval         |
| Priority           | Normal                 |
| Source Aisle ST    | 9007-9010              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                            | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Confirm `DMSTATION.CURRENT_MODE` for 1303 = 2 (or trigger **ID63** if not).            | Mode = Retrieval.                                                                                                                                                                                                                                          |               |                      |
| 2  | Open `LocationRetrievalSCH`, select Pallet `P-C005`, Station `1303`, Removal Type = **Unit Retrieval**, press **Set (F2)**. | Per **For 1303** carry definition: `DNCARRYINFO.WORK_TYPE=26`, `CARRY_FLAG=3`, `CMD_STATUS=1`, `DEST_STATION_NO=1303` (direct — no 720x hop needed), `END_STATION_NO=1303`. `DNWORKINFO`, `DNWORKLIST`, `DNSTOCKHISTORY` inserted. `DNPALLET.STATUS_FLAG=3`. |               |                      |
| 3  | `RetrievalSender` → **ID12** → **ID32** → **ID33**.                                    | Standard lifecycle. After ID33: source `DMSHELF.STATUS_FLAG=0`, `DNCARRYINFO.CMD_STATUS=5`.                                                                                                                                                                |               |                      |
| 4  | Inject **ID64 at STV** from 9007-9010 directly.                                        | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                 |               |                      |
| 5  | Inject **ID68** at 1303.                                                              | `DNOPERATIONDISPLAY` inserted.                                                                                                                                                                                                                              |               |                      |
| 6  | Operator Completion → **ID26**.                                                        | Path = `InOutStationOperator` (1301-1303 branch). `DNWORKINFO.STATUS_FLAG=4`; `DNHOSTSEND` inserted; **`DNINOUTRESULT` inserted** (`STATION_NO=1303`); `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed.                                                        |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT carry_key, work_type, carry_flag, cmd_status, dest_station_no, end_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-C005';

-- during/after the run there must NEVER be a DNARRIVAL row
SELECT COUNT(*) FROM dnarrival WHERE bcr_data = :bcr;   -- expect 0 throughout

-- after step 6
SELECT result_kind, station_no, location_no, wh_station_no
  FROM dninoutresult WHERE pallet_id = 'P-C005';
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-C005';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-C005';   -- expect 0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- **No** `DNARRIVAL` row was ever created (no 7207-7210 hop).
- Exactly one `DNINOUTRESULT` row, `STATION_NO=1303`.
- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-C005 removed.
- One `DNHOSTSEND` row, `REPORT_FLAG=0`.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
