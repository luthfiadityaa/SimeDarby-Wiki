Storage System — Developer Data Flow Reference
==============================================

**Version:** 3.0  
**Date:** 2026-03-23  
**Audience:** Developers implementing or maintaining storage flow

* * *

Section A — Palletization Storage Data Flow
-------------------------------------------

### A1. User Setup

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | Warenavi Screen | INSERT | DNReceivingPlan | plan_ukey="01"+seq, status=0, batch_station_no=111x, plan_area_no=9100/9200 |

* * *

### A2. Phase 1 — Empty Pallet Supply (1210 → 110x)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | Id66Process | SELECT | DNStock | item=EMP_PB, area IN(9007-9010), ORDER BY storage_date ASC |
| 2 | AutoStorageScheduler | INSERT DNCarryInfo | DNCarryInfo | carry_flag=3, dest=1210, end=1210 |
| 3 | AGC | — | — | Pallet moves 1210→1220→110x |

* * *

### A3. Phase 2 — Arrival at 110x

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | Id26Process→PalletizeRobotStationOperator | INSERT | DNArrival | station=110x, carry_key=99999999, sch_flag=0, sch_carry_key=blank |
**AutoStorageScheduler polling — ONE TRANSACTION:**
| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 2 | AutoStorageScheduler | SELECT | DNReceivingPlan | batch_station_no=111x, status IN(0,1,2) |
| — | — | — | — | If null → silent wait, continue |
| 3 | AisleShelfDecider.decideAisle() | SELECT | DMAisle, DMShelf | soft_zone, reachable aisles |
| 4 | WNCollectAisleSelector.determin() | INSERT | DNCollectInfo | aisle_collect_key, storage_aisle_no |
| 5 | AutoStorageScheduler | INSERT | DNPallet | PLT001, current=110x, item=DIRECT_PB, collect_batch_no=lot |
| 6 | AutoStorageScheduler | INSERT | DNStock | item=DIRECT_PB, pallet=PLT001, area=9100 |
| 7 | AutoStorageScheduler | INSERT | DNCarryInfo CK001 | carry_flag=3, dest=111x, end=710x |
| 8 | AutoStorageScheduler | UPDATE | DNReceivingPlan | status: 0→1 (only if status=0) |
| 9 | AutoStorageScheduler | UPDATE | DNArrival | sch_flag=1, sch_carry_key=CK001 |
| — | commit + carryRequest() | — | — | — |
| 10 | StorageSender | UPDATE | DNArrival | send_flag=SENDED |
| 10 | StorageSender | SEND | ID05 | dest=111x, carry=DIRECT_TRAVEL |

* * *

### A4. Phase 3 — Arrival at 111x (normal storage pass-through)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | PalletizeStorageStationOperator | UPDATE | DNCarryInfo CK001 | dest→710x (end_station), cmd_status→ARRIVAL |
| 2 | PalletizeStorageStationOperator | DELETE via LoadRemover | DNCarryInfo CK001 | (direct carry cleaned) |
| 3 | PalletizeStorageStationOperator | INSERT | DNArrival | station=111x, carry_key=DUMMY, sch_flag=0 |
**AutoStorageScheduler at 111x — ONE TRANSACTION:**
| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 4 | AutoStorageScheduler | SELECT+LOCK | DNReceivingPlan | NOWAIT |
| 5 | AutoStorageScheduler | INSERT | DNStoragePlan SP001 | status=0, plan_qty, bcr_data, job_type from plan |
| 6 | AisleShelfDecider.decideAisle() | SELECT | DMAisle, DMShelf | soft_zone, batch balance |
| 7 | WNCollectAisleSelector.determin() | INSERT/UPDATE | DNCollectInfo | aisle_collect_key, aisle_no |
| 8 | AutoStorageScheduler | SELECT+LOCK | DNStoragePlan | NOWAIT → ROLLBACK if locked |
| 9 | AutoStorageScheduler | INSERT | DNPallet PLT002 | current=111x, soft_zone, collect_key |
| 10 | AutoStorageScheduler | INSERT | DNCarryInfo CK002 | carry_flag=3, dest=710x, end=wh_station |
| 11 | AutoStorageScheduler | INSERT | DNStock STK001 | item=ZFNP, status=UU, tempering_period, expiry_date |
| 12 | AutoStorageScheduler | INSERT | DNWorkInfo WK001 | **job_type = DNStoragePlan.job_type** ← not hardcoded |
| 13 | AutoStorageScheduler | UPDATE | DNStoragePlan SP001 | status→1 (NOWWORKING) |
| 14 | AutoStorageScheduler | UPDATE | DNArrival | sch_flag=1, sch_carry_key=CK002 |
| — | commit + carryRequest() | — | — | — |
| 15 | StorageSender | UPDATE | DNArrival | send_flag=SENDED |
| 15 | StorageSender | SEND | ID05 | dest=710x, carry=DIRECT_TRAVEL |

* * *

### A5. Phase 4 — Arrival at 710x (OP ASRS inbound BCR)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | AsrsInboundStationOperator | CHECK | DMStation | this.wh_station = end_station → FINAL BCR |
| 2 | AsrsInboundStationOperator | UPDATE | DNCarryInfo CK002 | carry_flag→1(STORAGE), source→710x, dest→9100, cmd_status→START |
| 3 | AsrsInboundStationOperator | INSERT | DNArrival | station=710x, carry_key=CK002 |
| — | carryRequest() | — | — | — |
**StorageSender at 710x (bin selection):**
| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 4 | StorageSender.soueceRightStation() | CHECK | DMStation | max_instruction, suspend, mode |
| 5 | LocationManager.searchLocation() | SELECT | DMShelf | status=EMPTY, soft_zone=001, aisle=9001 |
| 6 | ShelfController.reserveShelf() | UPDATE | DMShelf SHF001 | status→2 (RESERVED) |
| 7 | StorageSender | UPDATE | DNCarryInfo CK002 | dest→SHF001, aisle_station_no=9001 |
| 8 | StorageSender | UPDATE | DNArrival | send_flag=SENDED |
| 8 | StorageSender | SEND | ID05 | dest=SHF001, location_no=B1-L1-A1, carry=STORAGE |

* * *

### A6. Phase 5 — ID33 Storage Complete

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | CarryCompleteOperator | UPDATE | DNWorkInfo WK001 | ⚑ status→4, result_location=B1-L1-A1, result_qty=qty |
| 2 | WorkInfoController | INSERT | **DNHostSend** | ⚑ job_type from DNWorkInfo, report_flag=0, result_location=B1-L1-A1 |
| 3 | AsStockController.addStock() | UPDATE | **DNStock STK001** | ⚑ location_no=B1-L1-A1, storage_date=now, stock_qty+=qty, plan_qty=0 |
| 4 | AsStockController | UPDATE | DMShelf SHF001 | status→1 (OCCUPIED) |
| 5 | AsStockController | UPDATE | DNPallet PLT002 | current_station=SHF001, status=STORED |
| 6 | AsStockController | INSERT | DNStockHistory | storage audit |
| 7 | PlanController.updatePlan() | UPDATE | **DNStoragePlan SP001** | ⚑ status→4 (COMPLETE) |
| 8 | InOutResultController | INSERT | DNInOutResult | result_kind=1, carry_key=CK001, location=B1-L1-A1 |
| 9 | Id33Process | DELETE | DNCarryInfo CK002 | — |
| 10 | ShelfMonitor (next cycle) | SEND | ID54 | lamp update if empty count threshold crossed |

> ⚑ = Fields to verify during testing

* * *

Section B — HP/OP Inbound Storage Data Flow
-------------------------------------------

### B1. Same Warehouse (e.g. 1106 → FGW1)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | SAP/User | INSERT | DNStoragePlan | job_type=22 (unplanned), plan_area_no=9200, soft_zone=003 |
| 2 | InOutStationOperator/StorageStationOperator | INSERT | DNArrival | station=1106, carry_key=DUMMY, sch_flag=0 |
| 3 | AutoStorageScheduler | SELECT | DNStoragePlan | bcr_data match, status=0 |
| 4 | AisleShelfDecider.decideAisle() | — | — | soft_zone=003, aisles 11-14 |
| 5 | WNCollectAisleSelector | INSERT | DNCollectInfo | if new batch |
| 6 | AutoStorageScheduler | INSERT | DNPallet, DNCarryInfo | dest=720x, end=9200 |
| 7 | AutoStorageScheduler | INSERT | DNStock, DNWorkInfo | job_type = storagePlan.getJobType() |
| 8 | AutoStorageScheduler | UPDATE | DNStoragePlan, DNArrival | status=1, SCHEDULED |
| 9 | StorageSender at 1106 | SEND | ID05 | dest=720x DIRECT_TRAVEL |
| 10 | AsrsInboundStationOperator at 720x | UPDATE | DNCarryInfo | wh=end → STORAGE, dest=9200 |
| 11 | StorageSender at 720x | SELECT | DMShelf | bin in zone 003 |
| 12 | StorageSender at 720x | SEND | ID05 | storage to bin |
| 13 | ID33 | UPDATE/INSERT | DNStock, DNWorkInfo, DNStoragePlan, DNHostSend | completion |

* * *

### B2. Cross Warehouse (e.g. 1301 → FGW2)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | User | INSERT | DNStoragePlan | job_type=22, plan_area_no=9100 (FGW2) |
| 2 | InOutStationOperator | INSERT | DNArrival | station=1301, carry_key=DUMMY |
| 3 | AutoStorageScheduler | isCrossWarehouseRoute? | — | 1301 → 9100 = YES |
| 4 | AutoStorageScheduler | SELECT | DMRouteId | routes 1301 → 720x candidates |
| 5 | AutoStorageScheduler | SELECT | DNCarryInfo | count active carries per 720x |
| 6 | AutoStorageScheduler | INSERT | DNPallet, DNCarryInfo | dest=7208 (least busy), end=9100 |
| 7 | AutoStorageScheduler | INSERT | DNStock, DNWorkInfo | job_type=22 |
| 8 | AutoStorageScheduler | UPDATE | DNStoragePlan, DNArrival | status=1, SCHEDULED |
| 9 | StorageSender at 1301 | SEND | ID05 | dest=7208 DIRECT_TRAVEL |
| 10 | AsrsInboundStationOperator at 7208 | CHECK | DMStation | wh=9200 ≠ end=9100 → INTERMEDIATE |
| 11 | AsrsInboundStationOperator at 7208 | SELECT | DMRouteId | reachable 710x from 7208 |
| 12 | AsrsInboundStationOperator at 7208 | SELECT | DMAisle | map BCR → aisle stations |
| 13 | AisleShelfDecider.decideAisle() | — | — | filtered aisles, soft_zone, batch balance |
| 14 | WNCollectAisleSelector | INSERT | DNCollectInfo | aisle decided |
| 15 | AsrsInboundStationOperator at 7208 | UPDATE | DNCarryInfo | dest=7103 (target BCR), source=7208 |
| 16 | AsrsInboundStationOperator at 7208 | INSERT | DNArrival | station=7208 |
| 17 | StorageSender at 7208 | SEND | ID05 | dest=7103 DIRECT_TRAVEL |
| 18 | AsrsInboundStationOperator at 7103 | UPDATE | DNCarryInfo | carry_flag→STORAGE, dest→9100 |
| 19 | StorageSender at 7103 | SELECT | DMShelf | bin in 9001-9006, zone=001 |
| 20 | StorageSender at 7103 | SEND | ID05 | storage |
| 21 | ID33 | UPDATE/INSERT | DNStock, DNWorkInfo, DNStoragePlan, DNHostSend | completion |

* * *

Section C — Reject Data Flow
----------------------------

### C1. BCR Fail at 110x

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | AutoStorageScheduler.checkBcr | — | — | FAIL → rejectProcess |
| 2 | AutoStorageScheduler | INSERT | DNCarryInfo | DIRECT_TRAVEL, dest=111x, end=1303, reject_factor=01 |
| 3 | AutoStorageScheduler | UPDATE | DNArrival | SCHEDULED |
| 4 | StorageSender | SEND | ID05 | 110x → 111x DIRECT_TRAVEL |
| 5 | PalletizeStorageStationOperator | CHECK | DNCarryInfo | reject_factor≠00 → isRejectDestination=true |
| 6 | PalletizeStorageStationOperator | UPDATE | DNCarryInfo | dest→1303 (end_station), cmd_status→ARRIVAL |
| 7 | StorageSender | SEND | ID05 | 111x → 1303 DIRECT_TRAVEL |
| 8 | InOutStationOperator at 1303 | DELETE via LoadRemover | DNCarryInfo | end=1303=this → remove |

> Note: No DNPallet or DNStock created in BCR fail path.

* * *

### C2. Error Completion at 111x (control_info=2)

| Step | Class | DB Operation | Table | Key Values |
| --- | --- | --- | --- | --- |
| 1 | AutoStorageScheduler.checkControlInfo | — | — | '2' → PALLETIZING_ERROR_COMPLETION |
| 2 | AutoStorageScheduler | INSERT | DNCarryInfo | DIRECT_TRAVEL, dest=1303, end=1303, reject_factor=06 |
| 3 | AutoStorageScheduler | UPDATE | DNArrival | SCHEDULED |
| 4 | StorageSender | SEND | ID05 | 111x → 1303 DIRECT_TRAVEL |
| 5 | InOutStationOperator at 1303 | DELETE via LoadRemover | DNCarryInfo | remove |

> Note: checkControlInfo called BEFORE insertStoragePlan → no orphan records.

* * *

### C3. Forced Removal via MOS (Id35Process)

| Scenario | Trigger | DNArrival.sch_carry_key | Id35 Action |
| --- | --- | --- | --- |
| 110x, no plan yet | ID35 DUMMY mckey | blank | DELETE DNArrival only |
| 110x, plan found | ID35 DUMMY mckey | CK001 | StorageTrackingDeleter: DELETE DNCarryInfo + DNPallet + DNStock |
| 111x after scheduling | ID35 real CK002 | — | StorageTrackingDeleter: DELETE all + DNStoragePlan→status=0 |
| 710x/720x | ID35 real key | — | StorageTrackingDeleter: DELETE all |

> DNReceivingPlan status is NOT rolled back in any scenario.

* * *

Section D — Settings Reference ⚙️
---------------------------------

> All settings below must be configured before system operation.

### D1. DMWareHouse

| station_no | aisle_decision_pattern | Reason |
| --- | --- | --- |
| 9100 | **4** (WN Collect) | Enables WNCollectAisleSelector + DNCollectInfo |
| 9200 | **4** (WN Collect) | Same |
Current value: `0` (wrong — ConnectedAisleSelector, no DNCollectInfo write)

### D2. DMStation — max_instruction

| station_no range | max_instruction | Reason |
| --- | --- | --- |
| 7101-7110, 7207-7214 | **1** | Allow StorageSender to send commands |
Current value: `0` (wrong — blocks all storage commands)

### D3. DMStation — station_operator_class

| station_no | class_name |
| --- | --- |
| 7101-7110, 7207-7214 | `jp.co.daifuku.asrs.location.AsrsInboundStationOperator` |
| 1101-1105 | `jp.co.daifuku.asrs.location.PalletizeRobotStationOperator` |

### D4. DMLocationFullLamp

| Field | Value | Notes |
| --- | --- | --- |
| station_no | 9100 or 9200 | Station where tower light is installed (confirm with engineer) |
| hard_zone_id | null | All hard zones |
| status_flag | 0 | STOP (initial) |
| reserved_qty | 0 | No reserved locations |
| warning_keyword | free_shelf_warning_num_agc5 | → DNSystemKVs → 50 |
| light_off_keyword | free_shelf_warning_release_num_agc5 | → DNSystemKVs → 60 |

### D5. DNSystemKVs — Lamp Thresholds

| keyword | number_value | Meaning |
| --- | --- | --- |
| free_shelf_warning_num_agc5 | 50 | WARNING lamp ON when empty < 50 |
| free_shelf_warning_release_num_agc5 | 60 | Both lamps OFF when empty > 60 |

### D6. DMRouteId — Missing Routes (must add)

Routes required for 1301/1302 → FGW2 cross-warehouse storage:
| start_station_no | end_station_no | Notes |
| --- | --- | --- |
| 7207 | 7101-7106 | After route fix by engineer |
| 7208 | 7103-7106 | After route fix |
| 7209 | 7104-7106 | After route fix |
| 7210 | 7105-7106 | After route fix |
Corresponding `DMRouteDetail` rows with machine path must also be added.

### D7. Constant.java — New Constants

    INBOUND_STORAGE_STATION_NOS  = ["1106","1301","1302","1303"]
    ASRS_INBOUND_STATION_NOS     = ["7101"..."7110","7207"..."7214"]
    HP_INTERMEDIATE_STATION_NOS  = ["7207","7208","7209","7210"]
    

* * *

Section E — DNHostSend Verification Checklist
---------------------------------------------

After first test storage run, verify `DNHostSend` record:
| Field | Expected Value | Source |
| --- | --- | --- |
| job_type | 02 (planned) or 22 (unplanned) | DNWorkInfo.job_type ← DNStoragePlan.job_type |
| status_flag | 4 | hardcoded (Completed) |
| plan_ukey | "02"+seq | DNWorkInfo.plan_ukey |
| stock_id | STKxxx | DNWorkInfo.stock_id |
| system_conn_key | CKxxx (carry key) | carry_key |
| result_qty | actual qty | DNWorkInfo.result_qty |
| result_area_no | 9100 or 9200 | DNStock.area_no |
| result_location_no | e.g. B1-L1-A1 | DNStock.location_no (actual bin) |
| result_lot_no | e.g. LOT001 | DNStock.lot_no |
| hardware_type | 3 | hardcoded (ASRS) |
| report_flag | 0 | pending SAP notification |
| bcr_data | pallet barcode | DNPallet.bcr_data |

> If `result_location_no` is blank or wrong → check `DNWorkInfo.result_location_no` after ID33. If `job_type` is wrong → check `insertWorkInfoStorage()` reads `storagePlan.getJobType()`.