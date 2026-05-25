[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SLRS-C004 — Unit Retrieval to ST 1303 via crane 7-10 (Flow 1)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Specific Location Retrieval Setting](/Basic-Design/DFD-WMS/Retrieval/Specific-Location-Retrieval-Setting) |
| **Removal Type** | Unit Retrieval (`REMOVAL_TYPE=1`)                                                             |
| **Destination**  | 1303 (Bi-Direction)                                                                           |
| **AGC Route**    | **ID63** → Flow 1: 9001-9006/9011-9014 → ID33 → ID64 STV → ID26@7207-7210 → RetrievalSender → ID25 → ID64 SRM → ID64 STV from 9007-9010 → ID68 → ID26 final |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the full crane 7-10 hop for 1303 when source is at WH9100 aisle (9001-9006 or 9011-9014). Pallet must travel via 720x conveyor before reaching 1303, and closure happens through `InOutStationOperator` writing a `DNINOUTRESULT` row.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1303 station `NORMAL`, `SUSPEND=OFF`, machine `ACTIVE`.
- `DMSTATION.CURRENT_MODE` for `1303` = 2 (Retrieval) — if not, step 1 runs ID63 first.
- Daily Update and Data Clear Flag not in progress.
- Pallet **P-C004** resides at an aisle in `9001-9006` or `9011-9014`.
- Transfer route entries exist: source aisle → `720x` (one of 7207-7210) → `9007-9010` → `1303`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                                |
|--------------------|--------------------------------------|
| Pallet ID          | P-C004                               |
| Destination Station| 1303                                 |
| Removal Type       | Unit Retrieval                       |
| Priority           | Normal                               |
| Source Aisle ST    | 9001-9006 or 9011-9014               |
| Conveyor Hop ST    | 7207-7210                            |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                                                                                          | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | (If `CURRENT_MODE=1`) Trigger **ID63** for 1303.                                       | `DMSTATION.CURRENT_MODE=2`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 2  | Set(F2) on `LocationRetrievalSCH` for `P-C004` → Station 1303, Removal Type = Unit Retrieval. | Per **For 1303** carry definition:<br>`DNCARRYINFO`: `WORK_TYPE=26` (Direct Transfer), `CARRY_FLAG=3`, `CMD_STATUS=1`, `DEST_STATION_NO ∈ {7207,7208,7209,7210}`, `END_STATION_NO = DEST_STATION_NO`, `AISLE_STATION_NO=DMSHELF.PARENT_STATION_NO`.<br>`DNWORKINFO`, `DNWORKLIST`, `DNSTOCKHISTORY` inserted (as per common Set(F2) DML).<br>`DNPALLET.STATUS_FLAG=3`. |               |                      |
| 3  | `RetrievalSender` → **ID12** → **ID32** → **ID33**.                                    | Standard lifecycle. After ID33: source `DMSHELF.STATUS_FLAG=0`, `DNCARRYINFO.CMD_STATUS=5`.                                                                                                                                                                                                                                                              |               |                      |
| 4  | Inject **ID64 at STV** (pickup at 9001-9006 / 9011-9014).                              | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 5  | Pallet arrives at conveyor station 7207-7210 → **ID26**.                               | `DNARRIVAL` inserted (`CARRY_KEY=99999999`, `SEND_FLAG=0`, height/width from ID26). `DNCARRYINFO` updated: `WORK_TYPE=23`, `CARRY_FLAG=2`, `SOURCE_STATION_NO=DNARRIVAL.STATION_NO`, `DEST_STATION_NO=1303`, `CMD_STATUS=1`.                                                                                                                            |               |                      |
| 6  | `RetrievalSender` at 7207-7210 → **ID12** dispatched.                                  | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                                                                                                                    |               |                      |
| 7  | Inject **ID25** (AGC acceptance at 7207-7210 → SRM).                                    | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`. `DNARRIVAL` row **deleted**.                                                                                                                                                                                                                                                                                 |               |                      |
| 8  | Inject **ID64 at SRM** (pickup from 7207-7210).                                         | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 9  | Inject **ID64 at STV** from 9007-9010 (transit through aisle 7-10).                     | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                                                                                              |               |                      |
| 10 | Inject **ID68** at 1303.                                                              | `DNOPERATIONDISPLAY` inserted (`STATION_NO=1303`, material/qty/dock/truck). LED at 1303 shows work info.                                                                                                                                                                                                                                                 |               |                      |
| 11 | Operator Completion → **ID26**.                                                        | Path = `InOutStationOperator` (1301-1303 branch). Same final-state assertions as TC-SLRS-C003 step 8 but with `DEST_STATION_NO=1303` and `DNINOUTRESULT.STATION_NO=1303`. `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed; `DNHOSTSEND` inserted.                                                                                                            |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 5
SELECT station_no, carry_key, bcr_data, send_flag FROM dnarrival WHERE bcr_data = :bcr;
SELECT carry_key, work_type, carry_flag, cmd_status, source_station_no, dest_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-C004';

-- after step 7
SELECT COUNT(*) FROM dnarrival WHERE bcr_data = :bcr;   -- expect 0

-- after step 11
SELECT result_kind, station_no, location_no, wh_station_no, aisle_station_no
  FROM dninoutresult WHERE pallet_id = 'P-C004';
SELECT COUNT(*) FROM dnpallet WHERE pallet_id = 'P-C004';     -- expect 0
SELECT COUNT(*) FROM dncarryinfo WHERE pallet_id = 'P-C004';  -- expect 0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- No `DNARRIVAL` rows remain for the test BCR.
- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` for P-C004 deleted.
- One `DNINOUTRESULT` row (`STATION_NO=1303`) and one `DNHOSTSEND` row inserted.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
