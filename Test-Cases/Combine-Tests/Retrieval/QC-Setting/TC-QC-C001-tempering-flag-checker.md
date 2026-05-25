[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-QC-C001 — Tempering Flag Checker auto-update</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test (background process)                                                             |
| **DFD Reference**| [QC Setting — Tempering Flag Checker](/Basic-Design/DFD-WMS/Retrieval/QC-Setting)             |
| **Class**        | `jp.co.daifuku.asrs.transmission.TemperingChecker`                                            |
| **Interval**     | Every `TEMPERING_CHECKER_SEC = 10000` ms (10 seconds)                                         |
| **Priority**     | Medium                                                                                        |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the background `TemperingChecker` flips `DNSTOCK.TEMPERING_FLAG` from `0 (Not Reached)` to `1 (Reached)` once `current date >= Storage Date + Tempering Period`, scoped to stock with `STOCK_STATUS='UU'` and `TEMPERING_FLAG=0`.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- `TemperingChecker` background process running.
- 2 `DNSTOCK` rows seeded for the test:
  - **S-QC-001**: `STOCK_STATUS='UU'`, `TEMPERING_FLAG=0`, `STORAGE_DATE='2026-05-20 00:00:00'`, `TEMPERING_PERIOD=72` (period **already elapsed** at test time).
  - **S-QC-002**: `STOCK_STATUS='UU'`, `TEMPERING_FLAG=0`, `STORAGE_DATE='2026-05-25 23:00:00'`, `TEMPERING_PERIOD=72` (period **not yet** elapsed).

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                       | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Wait one `TEMPERING_CHECKER_SEC` tick (≈10s).                                          | `TemperingChecker` runs.                                                                                                                                                                                                              |               |                      |
| 2  | Query `DNSTOCK` for S-QC-001.                                                          | `TEMPERING_FLAG=1` (Reached).                                                                                                                                                                                                         |               |                      |
| 3  | Query `DNSTOCK` for S-QC-002.                                                          | `TEMPERING_FLAG=0` unchanged (period not yet elapsed).                                                                                                                                                                                |               |                      |
| 4  | Wait further until S-QC-002's tempering period elapses, then one more tick.            | S-QC-002 `TEMPERING_FLAG=1`.                                                                                                                                                                                                          |               |                      |
| 5  | Negative — seed S-QC-003 with `STOCK_STATUS='QI'`, `TEMPERING_FLAG=0`, elapsed period. | After tick, S-QC-003 `TEMPERING_FLAG` remains `0` (checker only looks at `STOCK_STATUS='UU'`).                                                                                                                                       |               |                      |
| 6  | Negative — already `TEMPERING_FLAG=1`. Verify checker does not "downgrade".            | Stock with `TEMPERING_FLAG=1` remains `1`.                                                                                                                                                                                            |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- before & after each tick
SELECT stock_id, stock_status, tempering_flag, storage_date, tempering_period
  FROM dnstock WHERE stock_id IN ('S-QC-001','S-QC-002','S-QC-003');
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- S-QC-001 `TEMPERING_FLAG=1`.
- S-QC-002 `TEMPERING_FLAG=1` once its period elapses.
- S-QC-003 unchanged (`STOCK_STATUS='QI'` excluded by filter).

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
