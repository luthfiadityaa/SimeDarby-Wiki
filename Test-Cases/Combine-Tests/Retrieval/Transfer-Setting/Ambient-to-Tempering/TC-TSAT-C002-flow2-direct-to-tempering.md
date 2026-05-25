[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-TSAT-C002 — Transfer Ambient → Tempering (Flow 2: STV OP → BCR 7101-7106 → SRM 9001-9006)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Transfer Setting Ambient → Tempering — Flow 2](/Basic-Design/DFD-WMS/Retrieval/Transfer-Setting/Ambient-to-Tempering) |
| **Job Type**     | Retrieval (`JOB_TYPE=03`) — re-injected as storage at SRM 9001-9006                            |
| **Source**       | Pallet at SRM 9007-9010 (either continuation from Flow 1, or new transfer directly from 9007-9010) |
| **Path**         | ID64 STV OP → BCR 7101-7106 → ID26 → StorageSender → ID05 → ID25 → ID64 SRM 9001-9006 → ID33 (final placement) |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the final tempering placement leg. Pallet at SRM 9007-9010 (Ambient) is transferred via STV OP through BCR 7101-7106 to a destination tempering shelf in aisle 9001-9006. Ends with `ID33 at 9001-9006` which closes the transfer (updates `DMSHELF.STATUS_FLAG=1 OCCUPIED`, deletes `DNCARRYINFO`, inserts `DNINOUTRESULT`/`DNSTOCKHISTORY`, updates `DNPALLET`/`DNSTOCK`).

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; tempering destination aisle 9001-9006 has empty shelves available.
- Either (a) continuation of TC-TSAT-C001 (pallet `P-TSAT-001` at SRM 9007-9010), or (b) standalone new pallet **P-TSAT-002** at a shelf served by aisle 9007-9010.
- Stock validation passed at Set(F2).

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-TSAT-002             |
| Source Aisle ST    | 9007-9010              |
| BCR Hop ST         | 7101-7106              |
| Destination Aisle  | 9001-9006              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                            | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | (Standalone) Open `TransferSettingSCH`, select Pallet `P-TSAT-002`. Press **Set (F2)**. | DB writes per the TransferSetting DML table (`DNWORKINFO` `JOB_TYPE=03`, `DNCARRYINFO` `CARRY_FLAG=2`).                                                                                                                                                    |               |                      |
| 2  | RetrievalSender → **ID12** → **ID32** → **ID33** at SRM 9007-9010.                     | `DNCARRYINFO.CMD_STATUS` cycles `2 → 3 → 5`. Source `DMSHELF.STATUS_FLAG=0`.                                                                                                                                                                              |               |                      |
| 3  | Inject **ID64 at STV OP** (pickup at 9007-9010).                                       | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                |               |                      |
| 4  | Pallet arrives at BCR 7101-7106 → **ID26**.                                            | `DNARRIVAL` inserted. `DNCARRYINFO` updated (re-injected as storage carry toward SRM 9001-9006).                                                                                                                                                          |               |                      |
| 5  | `StorageSender` at 7101-7106 → **ID05**.                                               | Destination shelf in aisle 9001-9006 reserved. `DNCARRYINFO`/`DNPALLET`/`DNARRIVAL`/`DMSHELF` updated.                                                                                                                                                    |               |                      |
| 6  | Inject **ID25** at 7101-7106.                                                          | `DNCARRYINFO.CMD_STATUS=3`. `DNARRIVAL` deleted.                                                                                                                                                                                                            |               |                      |
| 7  | Inject **ID64 at SRM 9001-9006** (pickup at SRM).                                       | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                |               |                      |
| 8  | Inject **ID33 at 9001-9006** (deposit complete at tempering shelf).                     | `DMSHELF.STATUS_FLAG=1` (Occupied) for destination shelf.<br>`DNPALLET` updated (`CURRENT_STATION_NO=destination shelf parent station_no`, location updated).<br>`DNSTOCK` updated (`AREA_NO/LOCATION_NO` reflect new tempering location).<br>`DNINOUTRESULT` inserted.<br>`DNWORKINFO.STATUS_FLAG=4`.<br>`DNCARRYINFO` deleted. |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT carry_key, cmd_status FROM dncarryinfo WHERE pallet_id = 'P-TSAT-002';

-- after step 4
SELECT station_no, bcr_data FROM dnarrival WHERE bcr_data = :bcr;

-- after step 8 — final placement
SELECT pallet_id, current_station_no, status_flag FROM dnpallet WHERE pallet_id = 'P-TSAT-002';
SELECT area_no, location_no, status_flag FROM dmshelf
  WHERE area_no = :dest_area_no AND location_no = :dest_loc_no;       -- expect status_flag=1
SELECT stock_id, area_no, location_no FROM dnstock WHERE pallet_id = 'P-TSAT-002';
SELECT result_kind, station_no FROM dninoutresult WHERE pallet_id = 'P-TSAT-002';
SELECT COUNT(*) FROM dncarryinfo WHERE pallet_id = 'P-TSAT-002';   -- expect 0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Pallet P-TSAT-002 physically at destination tempering shelf in aisle 9001-9006.
- Destination `DMSHELF.STATUS_FLAG=1`.
- `DNCARRYINFO`/`DNARRIVAL` rows for the transfer removed.
- `DNINOUTRESULT` recorded with `RESULT_KIND` per StorageSender close DML.
- `DNWORKINFO.STATUS_FLAG=4` (Completed).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
