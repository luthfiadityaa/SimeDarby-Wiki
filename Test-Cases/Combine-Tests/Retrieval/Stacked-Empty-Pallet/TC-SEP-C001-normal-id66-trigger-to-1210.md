[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">TC-SEP-C001 — Stacked Empty Pallet retrieval to ST 1210 (normal ID66 trigger)</span>

| Field            | Value                                                                                         |
|------------------|-----------------------------------------------------------------------------------------------|
| **Test Type**    | Combine Test                                                                                  |
| **DFD Reference**| [Stacked Empty Pallet](/Basic-Design/DFD-WMS/Retrieval/Stacked-Empty-Pallet)                  |
| **Trigger**      | AGC sends **ID66** when MC has empty-pallet stock available                                   |
| **Destination**  | 1210 (`STATION_TYPE=2:OUT`, `RETRIEVAL_TRIGGER_REQUEST=1:AGC_REQUEST`, `WH_STATION_NO=9200`)  |
| **Source Aisles**| 9007 / 9008 / 9009 / 9010 only (9001-9006 and 9011-9014 have NO route to 1210)                |
| **AGC Route**    | ID66 → Id66Process → RetrievalTriggerAllocator → ID12 → ID32 → ID33 → ID64 → ID26             |
| **Priority**     | High                                                                                          |
| **Tester**       | _____________                                                                                 |
| **Test Date**    | _____________                                                                                 |
| **Overall Status** | [ ] Pass &nbsp;&nbsp; [ ] Fail                                                              |

##<span style="color:skyblue; font-weight:bold">Objective</span>

Verify the AGC-triggered empty-pallet retrieval flow. AGC sends ID66 → WareNavi marks station 1210 as `RETRIEVAL_TRIGGER_REQUEST=2` and immediately runs `RetrievalTriggerAllocator` → `allocateStackPallet` → `WebUnplannedRetrievalScheduler.schedule` to insert work info / carry info, then sends ID46 ack + ID12. Pallet completes through the standard chain with `MATERIAL_CODE=EMP_PB`, `MATERIAL_NAME=Empty Pallet`.

##<span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

- AGC `ONLINE`; 1210 `STATUS=NORMAL`, `SENDABLE=1`, `SUSPEND=OFF`, machine `ACTIVE`.
- Empty pallet stock available in `DNSTOCK` (material `EMP_PB`) on a shelf served by aisle `9007-9010`.
- Route entry exists in `DMRouteId` for source aisle → 1210.
- `DMSTATION` for 1210: `RETRIEVAL_TRIGGER_REQUEST=1` (idle, awaiting request).

##<span style="color:skyblue; font-weight:bold">Test Data</span>

| Key                | Value                  |
|--------------------|------------------------|
| Pallet ID          | P-SEP-001 (EMP_PB)     |
| Destination Station| 1210                   |
| Source Aisle ST    | 9007-9010              |

##<span style="color:skyblue; font-weight:bold">Test Steps</span>

| #  | Step                                                                                  | Expected Result                                                                                                                                                                                                                                                              | Actual Result | Status (Pass / Fail) |
|----|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------------|
| 1  | AGC sends **ID66** to WareNavi for station 1210.                                       | `Id66Process` runs. `DMSTATION.RETRIEVAL_TRIGGER_REQUEST=2` for STATION_NO=1210. `LAST_UPDATE_PNAME=Id66Process`.                                                                                                                                                            |               |                      |
| 2  | `Id66Process.processReceivedInfo` sends **ID46** (ack) back to AGC and invokes `RetrievalTriggerAllocator`. | ID46 frame observed on link.                                                                                                                                                                                                                                                |               |                      |
| 3  | `RetrievalTriggerAllocator → allocateStackPallet → WebUnplannedRetrievalScheduler.schedule` runs. | DB writes:<br>`DNWORKINFO` inserted (`JOB_TYPE=23`, `STATUS_FLAG=0`, `MATERIAL_CODE='EMP_PB'`, `MATERIAL_NAME='Empty Pallet'`).<br>`DNWORKLIST` inserted (`RETRIEVAL_DETAIL=1`).<br>`DNCARRYINFO` inserted (`WORK_TYPE=23`, `CARRY_FLAG=2`, `CMD_STATUS=1`, `SOURCE_STATION_NO ∈ {9007,9008,9009,9010}`, `DEST_STATION_NO=1210`, `END_STATION_NO=1210`).<br>`DNPALLET.STATUS_FLAG=3`, `ALLOCATION_FLAG=1`. |               |                      |
| 4  | `RetrievalSender` → **ID12**.                                                          | `DNCARRYINFO.CMD_STATUS=2`. `DNPALLET.STATUS_FLAG=4`.                                                                                                                                                                                                                        |               |                      |
| 5  | Inject **ID32**.                                                                      | `DNCARRYINFO.CMD_STATUS=3`, `ERROR_CODE=0`.                                                                                                                                                                                                                                  |               |                      |
| 6  | Inject **ID33**.                                                                      | Source `DMSHELF.STATUS_FLAG=0`. `DNCARRYINFO.CMD_STATUS=5`, `RETRIEVAL_STATION_NO=DMSHELF.STATION_NO`.                                                                                                                                                                       |               |                      |
| 7  | Inject **ID64**.                                                                      | `DNCARRYINFO.CMD_STATUS=4`.                                                                                                                                                                                                                                                  |               |                      |
| 8  | Pallet arrives at 1210; operator triggers Completion → **ID26**.                       | `RetrievalStationOperator` path:<br>`DNWORKINFO.STATUS_FLAG=4`, `SHORTAGE_QTY=PLAN_QTY-RESULT_QTY`, `RESULT_AREA_NO/LOCATION_NO` populated.<br>`DNWORKLIST` finalized (`RETRIEVAL_STATION_NO=1210`).<br>`DNHOSTSEND` + `DNINOUTRESULT` inserted (`RESULT_KIND=2`, `STATION_NO=1210`, `WH_STATION_NO=9200`).<br>`DNCARRYINFO`, `DNPALLET`, `DNSTOCK` deleted. `DMSHELF` updated. |               |                      |

##<span style="color:skyblue; font-weight:bold">Verification SQL</span>

```sql
-- after step 1
SELECT station_no, retrieval_trigger_request FROM dmstation WHERE station_no = 1210;   -- expect 2

-- after step 3
SELECT material_code, material_name, job_type FROM dnworkinfo WHERE plan_ukey = :plan_ukey;
SELECT carry_key, source_station_no, dest_station_no, end_station_no, carry_flag
  FROM dncarryinfo WHERE pallet_id = 'P-SEP-001';
SELECT status_flag, allocation_flag FROM dnpallet WHERE pallet_id = 'P-SEP-001';

-- after step 8
SELECT COUNT(*) FROM dnpallet     WHERE pallet_id = 'P-SEP-001';   -- expect 0
SELECT COUNT(*) FROM dncarryinfo  WHERE pallet_id = 'P-SEP-001';   -- expect 0
SELECT result_kind, station_no, wh_station_no FROM dninoutresult WHERE pallet_id = 'P-SEP-001';
SELECT job_no, status_flag, report_flag FROM dnhostsend WHERE job_no = :job_no;
```

##<span style="color:skyblue; font-weight:bold">Post-Conditions</span>

- Empty pallet removed from rack; arrived at 1210.
- `DNCARRYINFO`, `DNPALLET`, `DNSTOCK` removed.
- One `DNHOSTSEND` + one `DNINOUTRESULT` row (`STATION_NO=1210`, `WH_STATION_NO=9200`).
- `DMSTATION.RETRIEVAL_TRIGGER_REQUEST` for 1210 can return to 1 (idle) by next AGC cycle.

##<span style="color:skyblue; font-weight:bold">Notes / Defects</span>

```
[                                                                                ]
[                                                                                ]
```
