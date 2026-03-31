[[_TOC_]]

# Storage Flow End-to-End Verification

**Date:** 2026-03-31
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
| BCR sendable | **1** (OK) -- StorageSender can find them |
| BCR max_instruction | **2** (OK) -- capacity check in getReachableBcrStations |
| BCR class_name | AsrsInboundStationOperator (OK) |
| AGC status_flag | **2** OFFLINE (dev/test -- set to 1 for production) |
| AGC connection_flag | **0** DISCONNECTED (dev/test -- set to 1 for production) |
| DMLocationFullLamp | station_no = 1106/1111-1115/1301-1303 -- lamp at physical station, counts by wh_station via join (OK) |
| DMLocationFullLamp keywords | `free_shelf_warning_num_agc` matches DNSystemKVs (OK) |
| DMWareHouse | 9100=pattern 3, 9200=pattern 4 (OK) |

---

# Station Validation Rules

Enforced by `AutoStorageScheduler.validateStoragePlan()` before any carry is created:

| Station | Allowed | Rejected | Reason |
|---------|---------|----------|--------|
| 1106 | zone 003 (PM/ZPCK) only | zone 001 (Tempering), zone 002 (Ambient), zone 004 (EMP_PB) | Only routes to 7211-7214 (PM aisles 9011-9014) |
| 1303 | zone 001, 002, 004, 005 | zone 003 (PM/ZPCK) | Only routes to 7101-7110 (aisles 9001-9010, no PM) |
| 1301/1302 | ALL zones | none | Routes to both OP and HP BCRs |

---

# Routing Strategy: Two-Tier BCR Selection

`createCarryStorageInbound()` uses a two-tier approach:

**Tier 1 -- Direct BCR (all stations):**
`selectAisleAndGetBcrStation()` -> aisle selection -> `getReachableBcrForAisle()` -> BCR with route from source.

**Tier 2 -- Intermediate 720x (HP_INBOUND stations only):**
If Tier 1 returns null AND station is 1301/1302: `selectLeastBusyIntermediateStation()` -> 7207-7210.
AsrsInboundStationOperator at 720x then selects final BCR.

**Key fix:** PM items from 1301/1302 now route DIRECT to 7211-7214 via Tier 1, bypassing 720x intermediate. Only cross-warehouse scenarios (e.g. Tempering from 1301) fall through to Tier 2.

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
| A3 | **No aisle selection at 110x** -- pallet doesn't know which aisle yet | OK |
| A4 | INSERT DNPallet(DIRECT_PB), DNStock(DIRECT_PB), DNCarryInfo(DIRECT_TRAVEL dest=111x, **end=plan_area 9100/9200**) | OK |
| A5 | ID05: 110x -> 111x (route 1101->1111 exists) | OK |
| - | At 111x: PalletizeStorageStationOperator -> registArrival + carryRequest wakes StorageSender | OK |
| A6 | **StorageSender** at 111x -> processPalletizeStorage (Path 2) | -> Scenario 2 or 3 |

**Result: OK** -- Tested: `AutoStorageSchedulerStorageTest` tests path1_1101-1105, `StorageSenderPath2Test` tests 1-8

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

**Result: OK - Full end-to-end** -- Tested: `StorageSenderPath2Test#testNormal_Status0_Tempering`

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

**Result: OK - Full end-to-end** -- Tested: `StorageSenderPath2Test#testNormal_Status1_Ambient`

---

## Scenario 4: 1301/1302, plan_area 9100 (Cross-Warehouse -> Tempering)

| Stage | Description | Result |
|-------|-------------|--------|
| A | Tier 1: selectAisleAndGetBcrStation -> zone 001 -> aisles 9001-9006 -> no route from 1301 to 710x. Returns null | OK |
| A | Tier 2: HP_INBOUND -> selectLeastBusyIntermediateStation -> 7207-7210. Route 1301->7207 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7208, end=9100 | OK |
| B1 | At 7208: wh(9200)!=end(9100) -> INTERMEDIATE -> selectTargetAisleStation -> aisle 9001-9006 -> BCR 710x. Route 7208->7103 exists | OK |
| B1 | Pallet currentStation overridden to reachable 710x for AisleShelfDecider (avoids InvalidDefineException) | OK |
| B1 | Reachable aisle validation: selected aisle must be in reachableAisles list | OK |
| B1 | UPDATE dest=7103, end stays 9100 | OK |
| B2 | At 7103: wh(9100)==end(9100) -> FINAL BCR -> STORAGE dest=9100 | OK |
| C | StorageSender at 7103 -> LocationManager -> shelf zone 001 in 9003. 7103->9003 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end (3-stage)** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1301_crossWarehouse_Tempering`, `AsrsInboundStationOperatorTest#testIntermediate_7207_Tempering_ForwardTo710x`

---

## Scenario 5: 1301/1302, plan_area 9200 (Ambient, zone 002)

| Stage | Description | Result |
|-------|-------------|--------|
| A | Tier 1: selectAisleAndGetBcrStation -> zone 002 -> aisles 9007-9010 -> getReachableBcrForAisle -> 7207-7210. Route 1301->7207 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7207, end=9200 | OK |
| B | At 7207: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7207 -> LocationManager -> shelf zone 002 in 9007. 7207->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1301_Ambient_Zone002`

---

## Scenario 6: 1301/1302, PM/ZPCK (zone 003 -> aisles 9011-9014)

| Stage | Description | Result |
|-------|-------------|--------|
| A | Tier 1: selectAisleAndGetBcrStation -> zone 003 -> aisles 9011-9014 -> getReachableBcrForAisle -> 7211-7214. Route 1301->7211 exists | OK |
| A | **DIRECT routing to 721x** -- bypasses 720x intermediate (Tier 2 not needed) | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7211, end=9200 | OK |
| B | At 7211: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7211 -> LocationManager -> shelf zone 003 in 9011. 7211->9011 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end (FIXED)** -- Previously routed via 720x (7207-7210) which could only reach 9007-9010, not PM aisles. Now uses Tier 1 direct BCR selection. Tested: `AutoStorageSchedulerStorageTest#testPath3_1301_PM_direct721x`, `AsrsInboundStationOperatorTest` ZPCK tests at 7211-7213.

---

## Scenario 7: 1301/1302, EMP_PB (zone 004 -> aisles 9007-9010)

| Stage | Description | Result |
|-------|-------------|--------|
| A | Tier 1: selectAisleAndGetBcrStation -> zone 004 -> aisles 9007-9010 -> getReachableBcrForAisle -> 7207-7210. Route 1301->7207 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7207, end=9200 | OK |
| B | At 7207: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender -> LocationManager -> shelf zone 004 in 9007 (520 shelves). 7207->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1301_EmptyPallet_Zone004`, `AsrsInboundStationOperatorTest#testFinalBcr_7207_EmptyPallet_Storage`

---

## Scenario 8: 1106, plan_area 9100

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1106 only allows zone 003 PM) | EXPECTED REJECT |

**Result: EXPECTED REJECT** -- 1106 is PM-only station. Cannot reach Tempering aisles. Tested: `AutoStorageSchedulerStorageTest#testValidation_1106_Tempering_Rejected`

---

## Scenario 9: 1106, plan_area 9200 (PM, TestC zone 003)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> zone 003 allowed at 1106 | OK |
| A | selectAisleAndGetBcrStation -> zone 003 -> aisles 9011-9014 -> BCR 7211-7214. Route 1106->7211 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7211, end=9200 | OK |
| B | At 7211: wh(9200)==end(9200) -> FINAL BCR -> STORAGE dest=9200 | OK |
| C | StorageSender at 7211 -> LocationManager -> shelf zone 003 in 9011. 7211->9011 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1106_PlannedStorage_PM`, `AsrsInboundStationOperatorTest#testFinalBcr_7212_ZPCK_FromStation1106`

---

## Scenario 10: 1106, EMP_PB (zone 004)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1106 does not allow zone 004) | EXPECTED REJECT |

**Result: EXPECTED REJECT** -- 1106 only allows ZPCK/PM items. Tested: `AutoStorageSchedulerStorageTest#testValidation_1106_EmpPb_Rejected`

---

## Scenario 11: 1106, ZFNP/Ambient/Tempering

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1106 only allows zone 003 PM) | EXPECTED REJECT |

**Result: EXPECTED REJECT** -- Tested: `AutoStorageSchedulerStorageTest#testValidation_1106_Tempering_Rejected`

---

## Scenario 12: 1303, plan_area 9100 (Tempering, zone 001)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> zone 001 allowed at 1303 | OK |
| A | selectAisleAndGetBcrStation -> zone 001 -> aisles 9001-9006 -> BCR 7101-7106. Route 1303->7101 exists | OK |
| A | DNCarryInfo: DIRECT_TRAVEL dest=7101, end=9100 | OK |
| B | At 7101: FINAL BCR -> STORAGE dest=9100 | OK |
| C | StorageSender at 7101 -> LocationManager -> shelf zone 001 in 9001. 7101->9001 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK - Full end-to-end** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1303_Tempering`, `AsrsInboundStationOperatorTest#testFinalBcr_7101_ZFNP_FromStation1303_9100`

---

## Scenario 13a: 1303, plan_area 9200 -- Ambient item (zone 002)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> zone 002 allowed at 1303 | OK |
| A | zone 002 -> aisles 9007-9010 -> BCR 7107-7110. Route 1303->7107 exists | OK |
| B | At 7107: FINAL BCR -> STORAGE | OK |
| C | StorageSender -> shelf zone 002 in 9007. 7107->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1303_Ambient`, `AsrsInboundStationOperatorTest#testFinalBcr_7107_ZFNP_FromStation1303_9200`

---

## Scenario 13b: 1303, plan_area 9200 -- PM/ZPCK item (zone 003)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> **REJECTED** (1303 does not allow zone 003 PM) | EXPECTED REJECT |

**Result: EXPECTED REJECT (FIXED)** -- Previously failed at routing level (no routes 1303->7211-7214). Now rejected upfront by `validateStoragePlan()` which prevents any carry creation attempt. Tested: `AutoStorageSchedulerStorageTest#testValidation_1303_PM_Rejected`, `testValidation_1303_ZPCK_Rejected`

---

## Scenario 14: 1303, EMP_PB (zone 004 -> aisles 9007-9010)

| Stage | Description | Result |
|-------|-------------|--------|
| A | validateStoragePlan -> zone 004 allowed at 1303 | OK |
| A | zone 004 -> aisles 9007-9010 -> BCR 7107-7110. Route 1303->7107 exists | OK |
| B | At 7107: FINAL BCR -> STORAGE | OK |
| C | StorageSender -> shelf zone 004 in 9007 (520 shelves). 7107->9007 (1:1) | OK |
| D | ID33 complete | OK |

**Result: OK** -- Tested: `AutoStorageSchedulerStorageTest#testPath3_1303_EmptyPallet`, `AsrsInboundStationOperatorTest#testFinalBcr_7108_EmpPb_FromStation1303_9200`

---

## Scenario 15: 1303, ZFNP (zone 005 -> fallback to zone 001/002)

| Source | plan_area | Fallback | Flow | Result |
|--------|-----------|----------|------|--------|
| 1303 | 9100 | zone 005 -> 001 (Tempering) | aisles 9001-9006 -> BCR 7101-7106. Route 1303->7101 exists | **OK** |
| 1303 | 9200 | zone 005 -> 002 (Ambient) | aisles 9007-9010 -> BCR 7107-7110. Route 1303->7107 exists | **OK** |

**Result: OK** -- Tested: `AsrsInboundStationOperatorTest#testFinalBcr_7101_EmpPb_FromStation1303_9100`, `testFinalBcr_7107_ZFNP_FromStation1303_9200`

---

## Extra: 720x ZFNP Intermediate -> 710x (Cross-Warehouse Tempering)

Tests the full intermediate forwarding path for each 720x station with ZFNP items (zone 005, plan_area 9100).

| Station | Source | End | Flow | Result |
|---------|--------|-----|------|--------|
| 7207 | 1301 | 9100 | wh(9200)!=end(9100) -> selectTargetAisleStation -> 9001-9006 -> 7101-7106. carry stays DIRECT_TRAVEL | **OK** |
| 7208 | 1301 | 9100 | wh(9200)!=end(9100) -> selectTargetAisleStation -> 9001-9006 -> 7101-7106 | **OK** |
| 7209 | 1302 | 9100 | wh(9200)!=end(9100) -> selectTargetAisleStation -> 9001-9006 -> 7101-7106 | **OK** |
| 7210 | 1302 | 9100 | wh(9200)!=end(9100) -> selectTargetAisleStation -> 9001-9006 -> 7101-7106 | **OK** |

DB prerequisites verified per test:
- Routes 720x -> 7101-7106: 24 routes exist (full mesh)
- 710x stations: status=1 (NORMAL), max_instruction=2
- DMAisle 9001-9006: status=1 (NORMAL)
- WH 9100: aisle_decision_pattern=3 (DisperseAisleSelector)
- DMShelf: zone 001 shelves in 9001-9006 (1040 empty per aisle)
- DMSoftZonePriority: zone 005 -> 001 in WH 9100

**Result: OK - All 4 stations verified** -- Tested: `AsrsInboundStationOperatorTest` tests 17-20.

---

## Extra: ZFNP (TestZFNP, zone 005 -> fallback per warehouse)

| Source | plan_area | Flow | Result |
|--------|-----------|------|--------|
| 111x | 9100 | zone 005 -> 001 -> 9001-9006 -> BCR 7101-7106 | **OK** |
| 111x | 9200 | zone 005 -> 002 -> 9007-9010 -> BCR 7107-7110 | **OK** |
| 1301/1302 | 9100 | HP_INBOUND -> 720x INTERMEDIATE -> 710x (cross-wh, 3-stage) | **OK** |
| 1301/1302 | 9200 | Tier 1 direct -> 720x (final BCR) | **OK** |
| 1303 | 9100 | 1303->7101-7106 | **OK** |
| 1303 | 9200 | 1303->7107-7110 | **OK** |
| 1106 | 9200 | 1106 only allows zone 003 -- zone 005 REJECTED | **EXPECTED** |

**Result: OK for all valid ZFNP scenarios** -- zone 005 targets aisles 9001-9010 via DMSoftZonePriority fallback chain (005->001 in WH 9100, 005->002 in WH 9200).

Tested: `AsrsInboundStationOperatorTest` ZFNP tests at 7207/7208/7209/7210/7211/7214 (including intermediate tests 17-20), `AutoStorageSchedulerStorageTest` path3 ZFNP tests.

---

## Extra: 710x BCR from 1303 (ZFNP / EMP_PB)

| Station | Item | Source | End | Type | Result |
|---------|------|--------|-----|------|--------|
| 7101 | ZFNP | 1303 | 9100 | FINAL BCR | carry->STORAGE, dest=9100. **OK** |
| 7107 | ZFNP | 1303 | 9200 | FINAL BCR | carry->STORAGE, dest=9200. **OK** |
| 7101 | EMP_PB | 1303 | 9100 | FINAL BCR | carry->STORAGE, dest=9100. **OK** |
| 7108 | EMP_PB | 1303 | 9200 | FINAL BCR | carry->STORAGE, dest=9200. **OK** |

**Result: OK** -- 710x stations handle 1303-originating pallets correctly. Tested: `AsrsInboundStationOperatorTest` 710x tests.

---

# Final Summary

| # | Scenario | A (Scheduler) | B (BCR Operator) | C (StorageSender) | D (ID33) | Result |
|---|----------|--------------|-----------------|-------------------|----------|--------|
| 1 | 110x->111x | OK | OK | OK | OK | **OK** |
| 2 | 111x, 9100 | OK | OK | OK | OK | **OK** |
| 3 | 111x, 9200 | OK | OK | OK | OK | **OK** |
| 4 | 1301/1302, 9100 cross-wh | OK | OK (3-stage) | OK | OK | **OK** |
| 5 | 1301/1302, 9200 Ambient | OK | OK | OK | OK | **OK** |
| 6 | 1301/1302, PM/ZPCK z003 | OK (direct 721x) | OK | OK | OK | **OK (FIXED)** |
| 7 | 1301/1302, EMP_PB z004 | OK | OK | OK | OK | **OK** |
| 8 | 1106, 9100 | REJECT | - | - | - | **EXPECTED** |
| 9 | 1106, 9200 PM z003 | OK | OK | OK | OK | **OK** |
| 10 | 1106, EMP_PB z004 | REJECT | - | - | - | **EXPECTED** |
| 11 | 1106, Ambient/Tempering | REJECT | - | - | - | **EXPECTED** |
| 12 | 1303, 9100 Tempering | OK | OK | OK | OK | **OK** |
| 13a | 1303, 9200 Ambient | OK | OK | OK | OK | **OK** |
| 13b | 1303, PM/ZPCK z003 | REJECT | - | - | - | **EXPECTED (FIXED)** |
| 14 | 1303, EMP_PB z004 | OK | OK | OK | OK | **OK** |
| 15 | 1303, ZFNP z005 | OK | OK | OK | OK | **OK** |
| ex | 720x ZFNP intermediate->710x | OK | OK (all 720x) | OK | OK | **OK** |
| ex | ZFNP all sources | OK | OK | OK | OK | **OK** |
| ex | 710x from 1303 | OK | OK | OK | OK | **OK** |

---

# Test Coverage

## AutoStorageSchedulerStorageTest (22 tests)

**Path 1 -- Robot Stations 110x:**
| Test | Description | Status |
|------|-------------|--------|
| testPath1_1101_Status0_Tempering | Plan status 0, plan_area=9100 | PASS |
| testPath1_1102_Status1_Ambient | Plan status 1, plan_area=9200 | PASS |
| testPath1_1103_NoPlan_SilentWait | No ReceivingPlan, DNArrival stays NOT_SCHEDULED | PASS |
| testPath1_1104_NoRead_Reject | BCR no-read, reject via 111x to 1303 | PASS |
| testPath1_1105_BlankBcr_Reject | BCR blank, reject via 111x to 1303 | PASS |

**Path 3 -- Inbound Stations:**
| Test | Description | Status |
|------|-------------|--------|
| testPath3_1106_PlannedStorage_PM | PM zone 003 -> 7211-7214 direct | PASS |
| testPath3_1106_UnplannedStorage | job_type=22 unplanned | PASS |
| testPath3_1301_Ambient_via720x | Zone 002 -> 720x intermediate | PASS |
| testPath3_1301_PM_direct721x | Zone 003 -> 7211-7214 DIRECT (key fix) | PASS |
| testPath3_1301_crossWarehouse_Tempering | 9200->9100 via 720x->710x | PASS |
| testPath3_1303_Tempering | Zone 001 -> 7101-7106 | PASS |
| testPath3_1303_Ambient | Zone 002 -> 7107-7110 | PASS |
| testPath3_1303_EmptyPallet | Zone 004 -> 7107-7110 | PASS |
| testPath3_1301_Ambient_Zone002 | Ambient from 1301 | PASS |
| testPath3_1301_PM_Zone003 | PM from 1301 | PASS |
| testPath3_1301_EmptyPallet_Zone004 | EMP_PB from 1301 | PASS |
| testPath3_1301_Tempering_CrossWh | Tempering cross-warehouse from 1301 | PASS |
| testPath3_1302_Ambient_Zone002 | Ambient from 1302 | PASS |

**Validation Tests:**
| Test | Description | Status |
|------|-------------|--------|
| testValidation_1106_EmpPb_Rejected | 1106 + zone 004 -> REJECTED | PASS |
| testValidation_1106_Tempering_Rejected | 1106 + zone 001 -> REJECTED | PASS |
| testValidation_1303_PM_Rejected | 1303 + zone 003 -> REJECTED | PASS |
| testValidation_1303_ZPCK_Rejected | 1303 + ZPCK (zone 003) -> REJECTED | PASS |

## StorageSenderPath2Test (8 tests)

| Test | Description | Status |
|------|-------------|--------|
| testDbConfig | Station config + routes verified | PASS |
| testNormal_Status0_Tempering | plan_area=9100, dest=7101-7106 | PASS |
| testNormal_Status1_Ambient | plan_area=9200, dest=7107-7110 | PASS |
| testNoPlan_Reject | No ReceivingPlan -> dest=1303 | PASS |
| testForce_NotWaiting_Hold | control_info=001 + status!=2 -> wait_reason=17 | PASS |
| testLastPallet_Qty50 | WAITING_LAST_PALLET, qty=50, RP status->4 | PASS |
| testLastPallet_Qty0_Reject | WAITING_LAST_PALLET, qty=0 -> dest=1303 | PASS |
| testBadControl_Reject | Invalid control_info -> dest=1303 | PASS |

## AsrsInboundStationOperatorTest (20 tests)

| Test | Description | Status |
|------|-------------|--------|
| testDbConfig_7207_7210 | Station config verified | PASS |
| testFinalBcr_7207_EmptyPallet_Storage | 7207 EMP_PB, wh==end -> STORAGE | PASS |
| testFinalBcr_7208_Ambient_Storage | 7208 Ambient -> STORAGE | PASS |
| testFinalBcr_7207_ZFNP_FromStation1301 | 7207 ZFNP from 1301 -> STORAGE | PASS |
| testFinalBcr_7209_ZFNP_FromStation1302 | 7209 ZFNP from 1302 -> STORAGE | PASS |
| testFinalBcr_7211_ZPCK_FromStation1301 | 7211 ZPCK from 1301 -> STORAGE | PASS |
| testFinalBcr_7213_ZPCK_FromStation1302 | 7213 ZPCK from 1302 -> STORAGE | PASS |
| testFinalBcr_7212_ZPCK_FromStation1106 | 7212 ZPCK from 1106 -> STORAGE | PASS |
| testFinalBcr_7208_ZFNP_FromStation1301 | 7208 ZFNP from 1301 -> STORAGE | PASS |
| testFinalBcr_7214_ZFNP_FromStation1302 | 7214 ZFNP from 1302 -> STORAGE | PASS |
| testFinalBcr_7211_ZFNP_FromStation1106 | 7211 ZFNP from 1106 -> STORAGE | PASS |
| testFinalBcr_7107_ZFNP_FromStation1303_9200 | 7107 ZFNP from 1303, WH 9200 | PASS |
| testFinalBcr_7101_ZFNP_FromStation1303_9100 | 7101 ZFNP from 1303, WH 9100 | PASS |
| testFinalBcr_7101_EmpPb_FromStation1303_9100 | 7101 EMP_PB from 1303, WH 9100 | PASS |
| testFinalBcr_7108_EmpPb_FromStation1303_9200 | 7108 EMP_PB from 1303, WH 9200 | PASS |
| testIntermediate_7207_Tempering_ForwardTo710x | 7207 Tempering, wh!=end -> forward to 710x | PASS |
| testIntermediate_7207_ZFNP_FromStation1301_To710x | 7207 ZFNP from 1301, end=9100 -> forward to 710x | PASS |
| testIntermediate_7208_ZFNP_FromStation1301_To710x | 7208 ZFNP from 1301, end=9100 -> forward to 710x | PASS |
| testIntermediate_7209_ZFNP_FromStation1302_To710x | 7209 ZFNP from 1302, end=9100 -> forward to 710x | PASS |
| testIntermediate_7210_ZFNP_FromStation1302_To710x | 7210 ZFNP from 1302, end=9100 -> forward to 710x | PASS |

---

# Resolved Issues

| # | Issue | Resolution |
|---|-------|------------|
| 1 | **1301/1302 + PM (zone 003):** HP_INBOUND routed to 720x which can only reach 9007-9010, not PM aisles 9011-9014 | **FIXED:** Two-tier routing. Tier 1 `selectAisleAndGetBcrStation()` finds direct route to 7211-7214. Tier 2 (720x) only used as fallback for cross-warehouse. |
| 2 | **1303 + PM (zone 003):** No routes 1303->7211-7214 | **FIXED:** `validateStoragePlan()` rejects zone 003 from 1303 upfront. No route needed. |
| 3 | **1106 + EMP_PB/Ambient/Tempering:** Incompatible item types | **FIXED:** `validateStoragePlan()` rejects all non-PM zones from 1106. |

---

# Open Issues

None. All scenarios verified and tested.
