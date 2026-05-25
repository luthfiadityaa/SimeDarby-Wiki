[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-TSAT-C001 — Transfer Ambient → Tempering (Flow 1: via HP BCR 7207-7210)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Transfer Setting Ambient → Tempering](/Basic-Design/DFD-WMS/Retrieval/Transfer-Setting/Ambient-to-Tempering) |
| **Screen**       | `jp.co.daifuku.wms.web.display.retrieval.transfersetting.TransferSettingSCH`                  |
| **Job Type**     | Retrieval (`JOB_TYPE=03`) — but routed back into storage at SRM 9007-9010                     |
| **Source**       | Ambient shelf in aisle `9011-9014`                                                            |
| **Path**         | Stage 1 SRM retrieval → STV HP → BCR 7207-7210 → StorageSender → SRM 9007-9010 (intermediate) → **Continue to Flow 2** |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify Flow 1 of Ambient → Tempering transfer: a pallet sourced from a 9011-9014 ambient aisle is retrieved through STV HP (Stage 2 HP branch), arrives at BCR 7207-7210, gets re-injected as Storage by `StorageSender` (ID05), and is finally placed back at an intermediate SRM 9007-9010 aisle. Process continues to Flow 2 for actual tempering placement.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; source-aisle SRM `ACTIVE`; BCR 7207-7210 reachable.
- Pallet **P-TSAT-001** at a shelf served by aisle `9011-9014`.
- Stock validation per screen: `STOCK_STATUS='UU'`, `TEMPERING_FLAG=Reached`, `QC_CHECK_FLAG=Done`, storage location is **Ambient**.
- Daily Update and Data Clear Flag not in progress.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-TSAT-001             |
| Source Aisle ST    | 9011-9014              |
| BCR Hop ST         | 7207-7210              |
| Intermediate SRM ST| 9007-9010              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                  | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open **Transfer Setting** screen (`TransferSettingSCH`). Filter Storage Location = Ambient. | S-TSAT-001 listed.                                                                                                                                                                                                                                                              |               |                      |
| 2  | Select Pallet `P-TSAT-001` and press **Set (F2)**.                                     | Validations pass.<br>`DNWORKINFO` inserted (`JOB_TYPE=03 Retrieval`, `STATUS_FLAG=1 Working`, `HARDWARE_TYPE=3 ASRS`).<br>`DNWORKLIST` inserted (`RETRIEVAL_DETAIL=1`).<br>`DNCARRYINFO` inserted (`CARRY_FLAG=2 Retrieval`, `CMD_STATUS=1`).                                |               |                      |
| 3  | RetrievalSender → **ID12** → **ID32** → **ID33** at SRM (retrieval from rack).         | `DNCARRYINFO.CMD_STATUS` cycles `2 → 3 → 5`. Source `DMSHELF.STATUS_FLAG=0`.                                                                                                                                                                                                    |               |                      |
| 4  | Inject **ID64 at STV HP** (pickup at source aisle).                                    | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                      |               |                      |
| 5  | Pallet arrives at BCR 7207-7210 → **ID26**.                                            | `DNARRIVAL` inserted. `DNCARRYINFO` updated (transitions toward storage carry).                                                                                                                                                                                                  |               |                      |
| 6  | `StorageSender` at 7207-7210 → **ID05** dispatched.                                    | `DNCARRYINFO` updated for storage leg; `DNPALLET` updated; `DMSHELF` reserved at SRM 9007-9010.                                                                                                                                                                                  |               |                      |
| 7  | Inject **ID25** at 7207-7210.                                                          | `DNCARRYINFO.CMD_STATUS=3` (storage commanded). `DNARRIVAL` row **deleted**.                                                                                                                                                                                                     |               |                      |
| 8  | Inject **ID64 at SRM 9007-9010** (deposit-pickup at intermediate SRM).                 | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                      |               |                      |
| 9  | Continue to **Flow 2** for final placement at Tempering aisle 9001-9006.               | Hand off to TC-TSAT-C002 — pallet now resides at intermediate SRM 9007-9010 awaiting Flow 2 (STV OP → BCR 7101-7106 → SRM 9001-9006).                                                                                                                                            |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT job_no, job_type, status_flag FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT carry_key, carry_flag, cmd_status, dest_station_no, aisle_station_no
  FROM dncarryinfo WHERE pallet_id = 'P-TSAT-001';

-- after step 5
SELECT station_no, bcr_data FROM dnarrival WHERE bcr_data = :bcr;

-- after step 7
SELECT COUNT(*) FROM dnarrival WHERE bcr_data = :bcr;   -- expect 0

-- after step 8
SELECT cmd_status, dest_station_no FROM dncarryinfo WHERE pallet_id = 'P-TSAT-001';
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Source ambient shelf `DMSHELF.STATUS_FLAG=0`.
- Pallet physically located at SRM 9007-9010 (intermediate); ready for Flow 2.
- `DNARRIVAL` row removed.
- `DNCARRYINFO` row updated, not deleted (continuation to Flow 2).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
