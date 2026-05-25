[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-QC-C003 — Extend Tempering Period</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [QC Setting — Process 6: Extend Tempering Period](/Basic-Design/DFD-WMS/Retrieval/QC-Setting) |
| **Scope**        | Increase `TEMPERING_PERIOD` for an in-QC stock and reset `TEMPERING_FLAG` to Not Reached      |
| **Priority**     | Medium                                                                                        |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the **Extend Tempering Period** screen increases `TEMPERING_PERIOD` by the operator-entered value and resets `TEMPERING_FLAG` to `0 (Not Reached)`, scoped to `STOCK_STATUS=QI` and `QC_CHECK_FLAG=Not Done`. Stock Qty is preserved.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- `DNSTOCK` row **S-QC-200** seeded:
  - `STOCK_STATUS='QI'`, `TEMPERING_FLAG=1 (Reached)`, `QC_CHECK_FLAG='Not Done'`, `STOCK_QTY=60`, `TEMPERING_PERIOD=72`.
- QC Extend Tempering screen accessible.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key              | Value                  |
|------------------|------------------------|
| Stock ID         | S-QC-200               |
| Initial Period   | 72 H                   |
| Extend Value     | 3 H                    |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open **Extend Tempering Period** screen. Filter `StockStatus=QI, QcCheckFlag=Not Done`. | S-QC-200 listed with Qty=60, Period=72H, Flag=Reached.                                                                                                                                          |               |                      |
| 2  | Enter Extended Value = `3`. Submit.                                                    | `DNSTOCK` for S-QC-200:<br>`TEMPERING_PERIOD = 72 + 3 = 75`<br>`TEMPERING_FLAG = 0` (Not Reached)<br>`STOCK_QTY = 60` (retained)<br>`STOCK_STATUS = 'QI'` (retained)<br>`QC_CHECK_FLAG = 'Not Done'` (retained) |               |                      |
| 3  | Next `TemperingChecker` tick (≈10s).                                                   | If new period not yet elapsed, `TEMPERING_FLAG` remains `0`. Once it elapses again, checker flips back to `1`.                                                                                  |               |                      |
| 4  | Negative — submit extend with stock `STOCK_STATUS='UU'`.                                | Screen rejects (filter excludes UU stock).                                                                                                                                                      |               |                      |
| 5  | Negative — submit extend with `QC_CHECK_FLAG='Done'`.                                   | Screen rejects (filter excludes Done stock).                                                                                                                                                    |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 2
SELECT stock_id, tempering_period, tempering_flag, stock_status, qc_check_flag, stock_qty
  FROM dnstock WHERE stock_id = 'S-QC-200';
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- S-QC-200: `TEMPERING_PERIOD=75`, `TEMPERING_FLAG=0`, others unchanged.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
