[[_TOC_]]

# Storage Flow End-to-End Verification

**Date:** 2026-03-29
**Branch:** `6438_6524_6441_5801_control_for_storage`
**Oracle:** wms/wms@0.0.0.0:1521/orcl

---

# Physical Constraint: BCR to Aisle is 1:1

Each BCR station can ONLY deliver to its own aisle:

| BCR | Aisle | Warehouse | Area |
|-----|-------|-----------|------|
| 7101 | 9001 | 9100 | FGW2 Tempering |
| 7102 | 9002 | 9100 | FGW2 Tempering |
| 7103 | 9003 | 9100 | FGW2 Tempering |
| 7104 | 9004 | 9100 | FGW2 Tempering |
| 7105 | 9005 | 9100 | FGW2 Tempering |
| 7106 | 9006 | 9100 | FGW2 Tempering |
| 7107 | 9007 | 9200 | FGW1 Ambient |
| 7108 | 9008 | 9200 | FGW1 Ambient |
| 7109 | 9009 | 9200 | FGW1 Ambient |
| 7110 | 9010 | 9200 | FGW1 Ambient |
| 7207 | 9007 | 9200 | FGW1 Ambient |
| 7208 | 9008 | 9200 | FGW1 Ambient |
| 7209 | 9009 | 9200 | FGW1 Ambient |
| 7210 | 9010 | 9200 | FGW1 Ambient |
| 7211 | 9011 | 9200 | PM Ambient |
| 7212 | 9012 | 9200 | PM Ambient |
| 7213 | 9013 | 9200 | PM Ambient |
| 7214 | 9014 | 9200 | PM Ambient |

**7207 CANNOT reach 9011.** This is critical for PM routing from 1301/1302.

---

# DB State (After Fixes)

| Config | Status |
|--------|--------|
| BCR sendable | **1** (OK) — StorageSender can find them |
| BCR max_instruction | **2** (OK) |
| BCR class_name | AsrsInboundStationOperator (OK) |
| AGC status_flag | **1** ONLINE (OK) |
| AGC connection_flag | **1** CONNECTED (OK) |
| DMLocationFullLamp | station_no = 1106/1111-1115/1301-1303 — lamp at physical station, counts by wh_station via join (OK) |
| DMLocationFullLamp keywords | `free_shelf_warning_num_agc` matches DNSystemKVs (OK) |
| DMWareHouse | 9100=pattern 3, 9200=pattern 4 (OK) |

---

# Shelf Soft Zone Distribution

| Aisle | Zone 001 (Temp) | Zone 002 (Amb) | Zone 003 (PM) | Zone 004 (EmpPB) |
|-------|----------------|----------------|---------------|------------------|
| 9001-9006 | 1040 each | - | - | - |
| 9007-9010 | - | 2028 each | - | 520 each |
| 9011-9012 | - | 1508 each | 1040 each | - |
| 9013 | - | 1768 | 780 | - |
| 9014 | - | 1508 | 1040 | - |

---

# End-to-End Flow per Scenario

## Scenario 1: 110x -> 111x (Robot Input)

| Stage | Description | Result |
|-------|-------------|--------|
| A1 | ID26 at 110x -> PalletizeRobotStationOperator -> registArrival + autoScheduleRequest | OK |
| A2 | AutoStorageScheduler -> getReceivingPlanForRobot(next_station=1111, **status 0,1 only**) -> plan found | OK |
| A3 | **No aisle selection at 110x** — pallet doesn't know which aisle yet | OK |
| A4 | INSERT DNPallet(DIRECT_PB), DNStock(DIRECT_PB), DNCarryInfo(DIRECT_TRAVEL dest=111x, **end=plan_area 9100/9200**) | OK |
| A5 | ID05: 110x -> 111x (route 1101->1111 exists) | OK |
| - | At 111x: PalletizeStorageStationOperator -> registArrival + carryRequest wakes StorageSender | OK |
| A6 | **StorageSender** at 111x -> processPalletizeStorage (Path 2) | -> Scenario 2 or 3 |

**Result: OK** — Tested: `AutoStorageSchedulerStorageTest` tests 9-13, `StorageSenderPath2Test` tests 1-8

---

## Scenario 2: 111x -> plan_area 9100 (Tempering, TestA zone 001)

| Stage | Description | Result |
|-------|-------------|--------|
| A | **StorageSender.processPalletizeStorage** at 1111: findReceivingPlan, deleteDirectPbStock, createStoragePlan(qty=batch_qty_ctrn_pl), createStock, createWorkInfo | OK |
| A | selectAisleAndUpdateCarry: DisperseAisleSelector(pattern 3) -> aisle 9001-9006 -> BCR 7101-7106. Route 1111->7101 exists | OK |
| A | UPDATE DNCarryInfo: dest=7101, end stays 9100. UPDATE DNReceivingPlan progress_qty | OK |
| B | At 7101: wh(9100)==end(9100) -> FINAL BCR -> carry_flag=STORAGE, dest=9100 | OK |
| C | StorageSender finds 7101 (sendable=1, type=IN). storageDetermine -> LocationManager.searchLocation -> shelf in zone 001, aisle 9001. DMShelf RESERVED | OK |
| C | ID05: 7101 -> shelf in 9001 (1:1 physical match) | OK |
| D | ID33 -> DNWorkInfo status=4, DNHostSend(job_type from StoragePlan), DNStock.location_no=shelf, DMShelf=OCCUPIED, DNPallet=STORED, DNStoragePlan status=4, DNCarryInfo DELETE | OK |

**Result: OK - Full end-to-end** — Tested: `StorageSenderPath2Test#testNormal_Status0_Tempering`

---

## Scenario 3: 111x -> plan_area 9200 (Ambient, TestB zone 002)

| Stage | Description | Result |
|-------|-------------|--------|
| A | **StorageSender.processPalletizeStorage** at 1112: cross-warehouse override (1112 wh=9100, plan_area=9200) -> pallet.currentStation set to BCR in WH 9200 | OK |
| A | WNCollectAisleSelector(pattern 4) -> aisle 9007-9010 -> BCR 7107-7110. Route 1112->7107 exists | OK |
| A | UPDATE DNCarryInfo: dest=7107, end stays 9200 | OK |
| B | At 7107: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender -> storageDetermine -> LocationManager -> shelf zone 002 in 9007. Shelf in 9007 reachable from 7107 (1:1) | OK |
| C | ID05: 7107 -> shelf in 9007 | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** — Tested: `StorageSenderPath2Test#testNormal_Status1_Ambient`

---

## Scenario 4: 1301/1302, plan_area 9100 (Cross-Warehouse -> Tempering)

| Stage | Description | Result |
|-------|-------------|--------|
| A | HP_INBOUND -> selectLeastBusyIntermediateStation -> 7207-7210. Route 1301->7207 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7208, end=9100 | OK |
| B1 | At 7208: wh(9200)!=end(9100) -> INTERMEDIATE -> selectTargetAisleStation -> aisle 9001-9006 -> BCR 710x. Route 7208->7103 exists | OK |
| B1 | UPDATE dest=7103, end stays 9100 | OK |
| B2 | At 7103: wh(9100)==end(9100) -> FINAL BCR -> STORAGE dest=9100 | OK |
| C | StorageSender at 7103 -> LocationManager -> shelf zone 001 in 9003. 7103->9003 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end (3-stage)**

---

## Scenario 5: 1301/1302, plan_area 9200 (Ambient, zone 002)

| Stage | Description | Result |
|-------|-------------|--------|
| A | HP_INBOUND -> 720x intermediate. dest=7207, end=9200 | OK |
| B | At 7207: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7207 -> LocationManager -> shelf zone 002 in 9007. 7207->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end**

---

## Scenario 6: 1301/1302, ZPCK (zone 004 -> aisles 9007-9010)

| Stage | Description | Result |
|-------|-------------|--------|
| A | HP_INBOUND -> 720x intermediate. end=9200 | OK |
| B | At 720x: FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender -> LocationManager -> shelf zone 004. Zone 004 exists in 9007-9010 (520 per aisle). 720x->900x (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end**

---

## Scenario 7: 1301/1302, EMP_PB (zone 004 -> aisles 9007-9010)

Same as scenario 6.

**Result: OK - Full end-to-end**

---

## Scenario 8: 1106, plan_area 9100

| Stage | Description | Result |
|-------|-------------|--------|
| A | selectAisleAndGetBcrStation -> zone 001 -> aisles 9001-9006 -> BCR 7101-7106. No route from 1106 | EXPECTED FAIL |

**Result: EXPECTED FAIL** — 1106 is PM-only station. Should never receive plan_area 9100.

---

## Scenario 9: 1106, plan_area 9200 (PM, TestC zone 003)

| Stage | Description | Result |
|-------|-------------|--------|
| A | selectAisleAndGetBcrStation -> zone 003 -> aisles 9011-9014 -> BCR 7211-7214. Route 1106->7211 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7211, end=9200 | OK |
| B | At 7211: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7211 -> LocationManager -> shelf zone 003 in 9011. 7211->9011 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** — Tested: `AutoStorageSchedulerStorageTest#testPath3_1106_PlannedStorage_PM`

---

## Scenario 10: 1106, EMP_PB (zone 004)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1106 does not allow EMP_PB/zone 004) | **EXPECTED** |

**Result: EXPECTED REJECT** — 1106 only allows ZPCK items. Tested: `AutoStorageSchedulerStorageTest#testValidation_1106_EmpPb_Rejected`

---

## Scenario 11: 1106, ZFNP/Ambient/Tempering

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1106 only allows ZPCK) | **EXPECTED** |

**Result: EXPECTED REJECT** — Tested: `AutoStorageSchedulerStorageTest#testValidation_1106_Tempering_Rejected`

---

## Scenario 12: 1303, plan_area 9100 (Tempering, zone 001)

| Stage | Description | Result |
|-------|-------------|--------|
| A | selectAisleAndGetBcrStation -> zone 001 -> aisles 9001-9006 -> BCR 7101-7106. Route 1303->7101 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7101, end=9100 | OK |
| B | At 7101: FINAL BCR -> STORAGE dest=9100 | OK |
| C | StorageSender at 7101 -> LocationManager -> shelf zone 001 in 9001. 7101->9001 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end**

---

## Scenario 13a: 1303, plan_area 9200 — Ambient item (zone 002)

| Stage | Description | Result |
|-------|-------------|--------|
| A | zone 002 -> aisles 9007-9010 -> BCR 7107-7110. Route 1303->7107 exists | OK |
| B | At 7107: FINAL BCR -> STORAGE | OK |
| C | StorageSender -> shelf zone 002 in 9007. 7107->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK**

---

## Scenario 13b: 1303, plan_area 9200 — PM item (zone 003)

| Stage | Description | Result |
|-------|-------------|--------|
| A | zone 003 -> aisles 9011-9014 -> BCR 7211-7214. Route 1303->7211: **DOES NOT EXIST** | **FAIL** |

**ISSUE:** 1303 only has routes to 7101-7110. Cannot reach PM BCR 7211-7214.

**Fix:** Add routes 1303->7211, 7212, 7213, 7214.

---

## Scenario 14: 1303, ZPCK (zone 004 -> aisles 9007-9010)

| Stage | Description | Result |
|-------|-------------|--------|
| A | zone 004 -> aisles 9007-9010 -> BCR 7107-7110. Route 1303->7107 exists | OK |
| B | At 7107: FINAL BCR -> STORAGE | OK |
| C | StorageSender -> shelf zone 004 in 9007 (520 shelves). 7107->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK**

---

## Scenario 15: 1303, EMP_PB (zone 004 -> aisles 9007-9010)

Same as scenario 14. **OK**

---

## Extra: 1301/1302 + PM item (zone 003 -> aisles 9011-9014)

| Stage | Description | Result |
|-------|-------------|--------|
| A | AutoStorageScheduler: try direct BCR first -> selectAisleAndGetBcrStation -> zone 003 -> 9011-9014 -> BCR 7211-7214. Route 1301->7211 exists | **OK (FIXED)** |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7211, end=9200 | OK |
| B | At 7211: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7211 -> LocationManager -> shelf zone 003 in 9011. 7211->9011 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK — FIXED.** AutoStorageScheduler now tries direct BCR selection first. PM items from 1301/1302 route directly to 7211-7214 (bypassing 720x intermediate). Tested: `AutoStorageSchedulerStorageTest#testPath3_1301_PM_Zone003`, `AsrsInboundStationOperatorTest` tests 7-9.

---

## Extra: ZFNP (TestZFNP, zone 005 -> aisles 9001-9010)

| Source | plan_area | Flow | Result |
|--------|-----------|------|--------|
| 111x | 9100 | zone 005 -> 9001-9006 -> BCR 7101-7106 | **OK** |
| 111x | 9200 | zone 005 -> 9007-9010 -> BCR 7107-7110 | **OK** |
| 1301/1302 | 9100 | HP_INBOUND -> 720x -> 710x (cross-wh) | **OK** |
| 1301/1302 | 9200 | HP_INBOUND -> 720x (final BCR) | **OK** |
| 1303 | 9100 | 1303->7101-7106 | **OK** |
| 1303 | 9200 | 1303->7107-7110 | **OK** |

**Result: OK for all ZFNP scenarios** — zone 005 targets aisles 9001-9010 which are reachable from all sources.

---

# Final Summary

| # | Scenario | A (Scheduler) | B (BCR Operator) | C (StorageSender) | D (ID33) | Result |
|---|----------|--------------|-----------------|-------------------|----------|--------|
| 1 | 110x->111x | OK | OK | OK | OK | **OK** |
| 2 | 111x, 9100 | OK | OK | OK | OK | **OK** |
| 3 | 111x, 9200 | OK | OK | OK | OK | **OK** |
| 4 | 1301/1302, 9100 | OK | OK (3-stage) | OK | OK | **OK** |
| 5 | 1301/1302, 9200 Ambient | OK | OK | OK | OK | **OK** |
| 6 | 1301/1302, ZPCK | OK | OK | OK | OK | **OK** |
| 7 | 1301/1302, EMP_PB | OK | OK | OK | OK | **OK** |
| 8 | 1106, 9100 | FAIL | - | - | - | **EXPECTED** |
| 9 | 1106, 9200 PM | OK | OK | OK | OK | **OK** |
| 10 | 1106, ZPCK | FAIL | - | - | - | **FAIL** |
| 11 | 1106, EMP_PB | FAIL | - | - | - | **FAIL** |
| 12 | 1303, 9100 | OK | OK | OK | OK | **OK** |
| 13a | 1303, 9200 Ambient | OK | OK | OK | OK | **OK** |
| 13b | 1303, 9200 PM | FAIL | - | - | - | **FAIL** |
| 14 | 1303, ZPCK | OK | OK | OK | OK | **OK** |
| 15 | 1303, EMP_PB | OK | OK | OK | OK | **OK** |
| ex | 1301/1302, PM | OK | OK | **FAIL** (1:1) | - | **FAIL** |
| ex | ZFNP all | OK | OK | OK | OK | **OK** |

---

# Open Issues

| # | Issue | Severity | Fix Type |
|---|-------|----------|----------|
| 1 | **1301/1302 + PM (zone 003):** HP_INBOUND routes to 7207-7210 which physically can only reach 9007-9010. StorageSender will select PM shelf in 9011-9014 but AGC cannot deliver. | **HIGH** | Code: skip HP_INBOUND for PM, use direct 721x BCR |
| 2 | **1303 + PM (zone 003):** No routes 1303->7211..7214 | **HIGH** | DB: add 4 routes |
| 3 | **1106 + ZPCK/EMP_PB (zone 004):** No routes to 7107-7110 or 7207-7210 | **MEDIUM** | DB: add routes if expected scenario |
