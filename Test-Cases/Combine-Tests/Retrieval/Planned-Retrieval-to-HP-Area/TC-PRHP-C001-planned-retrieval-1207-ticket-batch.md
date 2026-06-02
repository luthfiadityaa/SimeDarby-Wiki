[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-PRHP-C001 — Planned Retrieval to ST 1207 (ticket batch)</span>

| Field              | Value                                                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| **Test Type**      | Combine Test                                                                                   |
| **DFD Reference**  | [Planned Retrieval to HP Area](/Basic-Design/DFD-WMS/Retrieval/Planned-Retrieval-to-HP-Area)   |
| **Screen**         | `jp.co.daifuku.wms.web.display.retrieval.PlannedRetrievalSettingSCH`                           |
| **Job Type**       | Planned Retrieval (`JOB_TYPE` per `DNRETRIEVALPLAN`, allocate via `RetrievalAllocateOperator`) |
| **Destination**    | 1207 (Unidirectional HP retrieval station)                                                     |
| **AGC Route**      | RetrievalAllocateOperator → RetrievalSender → ID12 → ID32 → ID33 → ID64 → ID68 → ID26          |
| **Priority**       | High                                                                                           |
| **Tester**         | _____________                                                                                  |
| **Test Date**      | _____________                                                                                  |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                                 |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify multi-pallet ticket-based Planned Retrieval. `PlannedRetrievalSettingSCH` invokes `RetrievalAllocateOperator.allocate` → `AbstractAllocateOperator.allocateStock`, which inserts `DNWORKINFO`/`DNWORKLIST`/`DNCARRYINFO` for each pallet and updates `DNRETRIEVALPLAN.STATUS_FLAG=1 (Working)`. Each pallet then flows independently through the AS21 chain to 1207, ending with one `DNHOSTSEND` row per pallet plus the `Planned Retrieval Result` host message.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1207 station `NORMAL`, not suspended; machine `ACTIVE`.
- Daily Update **not** running; Retrieval Allocate Flag **in progress** (per DFD validation note).
- Inventory-to-Retrieval **already allocated** for ticket pallets (`DNRETRIEVALPLAN` rows pre-loaded from Planned Retrieval Overview).
- 3 ticket-line pallets present at shelves served by `9001-9014`:
  - `P-PRHP-001`, `P-PRHP-002`, `P-PRHP-003`.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                                  |
|--------------------|----------------------------------------|
| Ticket #           | TICK-2026-0001                         |
| Pallets            | P-PRHP-001, P-PRHP-002, P-PRHP-003     |
| Destination Station| 1207                                   |
| Shipping Date      | 2026-05-27                             |
| Dock #             | D-01                                   |
| Truck Plate #      | BK 1234 XX                             |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                                | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open `PlannedRetrievalSettingSCH`, select Ticket `TICK-2026-0001` with 3 lines.        | Screen lists 3 pallets with status/material/batch/qty.                                                                                                                                                                                                                          |               |                      |
| 2  | Set destination Station=`1207`, Dock=`D-01`, Truck Plate=`BK 1234 XX`. Press **Set (F2)**. | `RetrievalAllocateOperator.allocate` invoked.<br>3 × `DNWORKINFO` inserted; 3 × `DNWORKLIST`; 3 × `DNCARRYINFO` (`CARRY_FLAG=2`, `DEST_STATION_NO=1207`, `END_STATION_NO=1207`).<br>3 × `DNPALLET` updated (`STATUS_FLAG=3`).<br>`DNRETRIEVALPLAN.STATUS_FLAG=1` (Working).<br>3 × `DNSTOCK` updated (allocation marked). |               |                      |
| 3  | For each pallet — RetrievalSender → **ID12** → **ID32** → **ID33**.                    | 3 carries cycle through `CMD_STATUS 2 → 3 → 5`; corresponding `DMSHELF.STATUS_FLAG=0` for each source shelf.                                                                                                                                                                  |               |                      |
| 4  | Inject **ID64 at STV** for each pallet.                                                | Each `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                |               |                      |
| 5  | Inject **ID68** at 1207 for each pallet arrival.                                       | 3 × `DNOPERATIONDISPLAY` inserted. LED at 1207 advances through pallets per AGC sequencing.                                                                                                                                                                                    |               |                      |
| 6  | For each pallet, operator presses Completion → **ID26**.                               | Each pallet: `DNWORKINFO.STATUS_FLAG=4`; `DNHOSTSEND` inserted; `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` deleted; `DMSHELF.STATUS_FLAG=0`.                                                                                                                                          |               |                      |
| 7  | After all 3 complete — verify host-send batch.                                         | 3 `DNHOSTSEND` rows present, all `STATUS_FLAG=4`, `REPORT_FLAG=0`. **Planned Retrieval Result** host process picks them up next cycle.                                                                                                                                          |               |                      |
| 8  | Verify `DNRETRIEVALPLAN`.                                                             | All 3 lines marked complete per plan-completion logic (verify per project's plan-status table).                                                                                                                                                                                |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT pallet_id, status_flag FROM dnpallet WHERE pallet_id IN ('P-PRHP-001','P-PRHP-002','P-PRHP-003');
SELECT COUNT(*) FROM dncarryinfo WHERE pallet_id IN ('P-PRHP-001','P-PRHP-002','P-PRHP-003');   -- expect 3
SELECT status_flag FROM dnretrievalplan WHERE ticket_no = 'TICK-2026-0001';                    -- expect 1

-- after step 6 (per pallet)
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = :pallet_id;   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = :pallet_id;   -- expect 0
SELECT COUNT(*) FROM dnstock      WHERE pallet_id = :pallet_id;   -- expect 0

-- after step 7
SELECT job_no, status_flag, result_qty, report_flag FROM dnhostsend
  WHERE pallet_id IN ('P-PRHP-001','P-PRHP-002','P-PRHP-003');     -- expect 3 rows, report_flag=0
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- All 3 pallets removed from `DNCARRYINFO`/`DNPALLET`/`DNSTOCK`.
- 3 `DNHOSTSEND` rows present, awaiting Planned Retrieval Result host send.
- `DNRETRIEVALPLAN` advanced per plan-completion rule.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
