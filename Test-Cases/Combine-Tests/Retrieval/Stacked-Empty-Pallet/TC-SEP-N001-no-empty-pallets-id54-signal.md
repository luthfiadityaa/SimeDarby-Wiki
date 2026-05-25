[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SEP-N001 — No empty pallets in ASRS — ID54 signal tower at 1107</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Irregular / Negative Combine Test                                                             |
| **DFD Reference**| [Stacked Empty Pallet — Irregular Cases](/Basic-Design/DFD-WMS/Retrieval/Stacked-Empty-Pallet) |
| **Trigger**      | AGC sends **ID66** but no eligible empty-pallet stock exists                                  |
| **Signal**       | **ID54** to signal tower at station 1107, lamp 15 (`As21Id54.LAMP_EMPTY_PALLET_NOT_FOUND`)    |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the "no empty pallet found" irregular path. After ID66, `RetrievalTriggerAllocator` finds no allocatable stock → sends ID54 with `instruction=1` (lamp ON) to station 1107, MC saves the trigger in memory. When an empty pallet later becomes available, ID54 is re-sent with `instruction=0` (lamp OFF) and normal flow resumes.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1210 station `NORMAL`.
- `DNSTOCK` **does not** contain any allocatable empty pallet (`MATERIAL_CODE='EMP_PB'`) on aisles 9007-9010.
- Signal tower station 1107 reachable; lamp 15 currently OFF.

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                          | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | AGC sends **ID66** for station 1210.                                                   | `Id66Process` runs. `DMSTATION.RETRIEVAL_TRIGGER_REQUEST=2` for 1210.                                                                                                                                    |               |                      |
| 2  | `RetrievalTriggerAllocator` executes — finds no allocatable stock.                     | No DB writes to `DNWORKINFO`/`DNCARRYINFO`/`DNWORKLIST`.                                                                                                                                                 |               |                      |
| 3  | WareNavi sends **ID54** to station 1107 (lamp 15, `instruction=1`).                    | `As21Id54` invoked. Signal tower at 1107 lights up (lamp 15 ON). MC saves the retrieval trigger in memory and waits.                                                                                     |               |                      |
| 4  | Operator (or system) introduces an empty pallet into a 9007-9010 shelf (stock now available). | `DNSTOCK` now contains an allocatable `EMP_PB` pallet. `RetrievalTriggerAllocator` next cycle picks it up.                                                                                            |               |                      |
| 5  | WareNavi sends **ID54** to station 1107 with `instruction=0` (lamp OFF).                | Signal tower at 1107 turns off (lamp 15 OFF). Flow continues into normal allocation per TC-SEP-C001 step 3+.                                                                                            |               |                      |
| 6  | Normal chain continues: ID12 → ID32 → ID33 → ID64 → ID26 → pallet at 1210.             | Same DB transitions as TC-SEP-C001 steps 4-8.                                                                                                                                                            |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL / Log Checks</span>

```sql
-- after step 1
SELECT station_no, retrieval_trigger_request FROM dmstation WHERE station_no = 1210;   -- expect 2

-- after step 2 — no work / carry yet
SELECT COUNT(*) FROM dnworkinfo  WHERE registered IN (now - 10 sec);   -- expect 0
SELECT COUNT(*) FROM dncarryinfo WHERE registered IN (now - 10 sec);   -- expect 0
```

Log checks:
- `As21Id54` debug log shows `LAMP_EMPTY_PALLET_NOT_FOUND` with `instruction=1` after step 3.
- Same lamp with `instruction=0` after step 5.

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Signal tower at 1107 returns to OFF state once stock is available.
- Empty pallet is delivered to 1210 via normal flow.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
