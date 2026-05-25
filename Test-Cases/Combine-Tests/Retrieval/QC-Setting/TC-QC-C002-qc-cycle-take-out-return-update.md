[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-QC-C002 — QC cycle: take-out → return → update status</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test (end-to-end QC sampling cycle)                                                   |
| **DFD Reference**| [QC Setting — Processes 2-5](/Basic-Design/DFD-WMS/Retrieval/QC-Setting)                      |
| **Scope**        | Take out QC sample → Return stock → Update QC Status to UU/DONE                               |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the full QC sampling cycle on a single stock row:
- **Process 2** — Retrieval for QC Start: take out qty `2` from stock `60` → stock `58`, status flips `UU → QI`, `QC_DURATION = Newest Storage Date + Now()`.
- **Process 4** — Return Stock: add qty `2` back → stock `60`, `QC_DURATION = QC_DURATION + Now()`.
- **Process 5** — Update QC Status: stock `60`, `QC_CHECK_FLAG → DONE`, status flips `QI → UU`, `QC_DURATION` stops.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- `DNSTOCK` row **S-QC-100** seeded:
  - `STOCK_STATUS='UU'`, `TEMPERING_FLAG=1 (Reached)`, `QC_CHECK_FLAG='Not Done'`, `STOCK_QTY=60`, `TEMPERING_PERIOD=72`.
- QC workstation user authenticated; QC retrieval screens accessible.

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key            | Value                  |
|----------------|------------------------|
| Stock ID       | S-QC-100               |
| Initial Qty    | 60                     |
| Take-Out Qty   | 2                      |
| Return Qty     | 2                      |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                  | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | Open **Retrieval for QC Start** screen. Filter `StockStatus=UU, TemperingFlag=1`.      | Stock S-QC-100 appears with Qty=60, Tempering Period=72H, QC Check Flag=Not Done.                                                                                                                                                                |               |                      |
| 2  | **Process 2** — Take Out Qty = `2`. Submit.                                            | `DNSTOCK` for S-QC-100 updated:<br>`STOCK_QTY = 60 - 2 = 58`<br>`STOCK_STATUS = 'QI'`<br>`TEMPERING_FLAG = 1` (retained)<br>`QC_DURATION = Newest Storage Date + Now()`<br>`TEMPERING_PERIOD = 72` (retained)<br>`QC_CHECK_FLAG = 'Not Done'` (retained) |               |                      |
| 3  | Open **Retrieval for Return Stock** screen. Filter `StockStatus=QI`.                   | S-QC-100 appears with Qty=58, Status=QI, `QC_DURATION` shows `(QC_DURATION + Now)`.                                                                                                                                                              |               |                      |
| 4  | **Process 4** — Qty To Add = `2`. Submit.                                              | `DNSTOCK` for S-QC-100 updated:<br>`STOCK_QTY = 58 + 2 = 60`<br>`QC_DURATION = QC_DURATION + Now()` (accumulated)<br>`STOCK_STATUS = 'QI'` (retained)                                                                                              |               |                      |
| 5  | Open **Update QC Status** screen. Filter `StockStatus=QI`.                             | S-QC-100 appears with Qty=60, Status=QI.                                                                                                                                                                                                          |               |                      |
| 6  | **Process 5** — Confirm QC pass; submit.                                               | `DNSTOCK` for S-QC-100 updated:<br>`STOCK_STATUS = 'UU'`<br>`QC_CHECK_FLAG = 'Done'`<br>`QC_DURATION` stops accumulating (final = `QC_DURATION + Now()`)<br>`STOCK_QTY = 60` (retained)<br>`TEMPERING_PERIOD = 72`, `TEMPERING_FLAG = 1` (retained) |               |                      |
| 7  | Verify history.                                                                       | `DNSTOCKHISTORY` rows recorded for each qty change (decrease at step 2, increase at step 4).                                                                                                                                                     |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after each process
SELECT stock_id, stock_status, tempering_flag, qc_check_flag, stock_qty, qc_duration, tempering_period
  FROM dnstock WHERE stock_id = 'S-QC-100';

-- after step 7
SELECT stock_id, inc_dec_type, work_day, stock_qty
  FROM dnstockhistory WHERE stock_id = 'S-QC-100' ORDER BY regist_date;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- S-QC-100: `STOCK_STATUS=UU`, `QC_CHECK_FLAG=Done`, `STOCK_QTY=60`, `TEMPERING_PERIOD=72`, `TEMPERING_FLAG=1`.
- 2 `DNSTOCKHISTORY` rows recorded.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
