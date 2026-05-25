[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-PROP-C001 — Planned PKG Retrieval to ST 1202 (ticket batch)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Planned Retrieval to OP-Area (PKG)](/Basic-Design/DFD-WMS/Retrieval/Planned-Retrieval-to-OP-Area-(PKG)) |
| **Screen**       | `PlannedRetrievalSettingSCH`                                                                  |
| **Job Type**     | Planned Retrieval (allocate via `RetrievalAllocateOperator`)                                  |
| **Destination**  | 1202 (OP Packaging — group 1200)                                                              |
| **Route Restriction** | Source aisle MUST be in **9011-9014** (PM only)                                          |
| **AGC Route**    | RetrievalAllocateOperator → RetrievalSender → ID12 → ID32 → ID33 → ID68 → ID26                |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify multi-pallet ticket-based PKG retrieval to 1202. Same allocation pipeline as PRHP but route restricted to PM aisles 9011-9014. Confirms `DNRETRIEVALPLAN` status update and that a non-PM source aisle is rejected by the route allocator.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1202 `NORMAL`, not suspended; machine `ACTIVE`.
- Daily Update not running; Retrieval Allocate Flag in progress.
- Inventory-to-Retrieval allocated; `DNRETRIEVALPLAN` rows pre-loaded.
- 2 PKG pallets in aisle 9011-9014: `P-PROP-001`, `P-PROP-002`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                       |
|--------------------|-----------------------------|
| Ticket #           | TICK-2026-0002              |
| Pallets            | P-PROP-001, P-PROP-002      |
| Destination Station| 1202                        |
| Dock #             | D-02                        |
| Truck Plate #      | BK 5678 YY                  |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                          | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open `PlannedRetrievalSettingSCH`, select Ticket `TICK-2026-0002`.                     | 2 ticket lines listed.                                                                                                                                                                                                                   |               |                      |
| 2  | Set destination Station=`1202`, Dock=`D-02`, Truck Plate=`BK 5678 YY`. Press **Set (F2)**. | `RetrievalAllocateOperator.allocate` invoked.<br>2 × `DNWORKINFO`, `DNWORKLIST`, `DNCARRYINFO` inserted (`DEST_STATION_NO=1202`).<br>2 × `DNPALLET.STATUS_FLAG=3`.<br>`DNRETRIEVALPLAN.STATUS_FLAG=1`. |               |                      |
| 3  | For each pallet — RetrievalSender → **ID12** → **ID32** → **ID33**.                    | Each carry: `CMD_STATUS 2 → 3 → 5`; source `DMSHELF.STATUS_FLAG=0`.                                                                                                                                                                      |               |                      |
| 4  | Inject **ID68** at 1202 for each pallet arrival.                                       | 2 × `DNOPERATIONDISPLAY` inserted.                                                                                                                                                                                                        |               |                      |
| 5  | Operator presses Completion → **ID26** (each).                                          | `DNWORKINFO.STATUS_FLAG=4`; `DNHOSTSEND` inserted; `DNINOUTRESULT` inserted (per OP-PKG DML table); `DNCARRYINFO`/`DNPALLET`/`DNSTOCK` deleted; `DMSHELF.STATUS_FLAG=0`.                                                                  |               |                      |
| 6  | **Negative sub-case** — re-run with a pallet on aisle 9001 (Tempering).                | Route allocator rejects ("no route to 1202 from this aisle"). No DB writes for that pallet.                                                                                                                                              |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT pallet_id, dest_station_no, aisle_station_no FROM dncarryinfo
  WHERE pallet_id IN ('P-PROP-001','P-PROP-002');

-- after step 5
SELECT COUNT(*) FROM dnpallet WHERE pallet_id IN ('P-PROP-001','P-PROP-002');     -- expect 0
SELECT job_no, report_flag FROM dnhostsend WHERE pallet_id IN ('P-PROP-001','P-PROP-002');
SELECT result_kind, station_no FROM dninoutresult WHERE pallet_id IN ('P-PROP-001','P-PROP-002');
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Both pallets removed from `DNCARRYINFO`/`DNPALLET`/`DNSTOCK`.
- 2 `DNHOSTSEND` rows + 2 `DNINOUTRESULT` rows present.
- `DNRETRIEVALPLAN` advanced per plan-completion logic.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
