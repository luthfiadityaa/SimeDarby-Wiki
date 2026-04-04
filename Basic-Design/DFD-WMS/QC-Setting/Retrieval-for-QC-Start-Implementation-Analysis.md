[[_TOC_]]

# Retrieval for QC Start - Implementation Analysis

**Date:** 2026-03-27
**Branch:** `5137-qcsetting-retrievalforstartqc`
**Related DFD:** [Retrieval for QC Start](Retrieval-for-QC-Start)

---

# 1. QCStartSCH.addnewConfirmationList - Implemented

## Overview

Implemented `addnewConfirmationList()` in `QCStartSCH.java` to validate selected stocks and trigger ASRS retrieval to station **1303** (QC Station).

**File:** `WEB-INF/src/jp/co/daifuku/wms/web/display/qcsetting/qcstartretrieval/QCStartSCH.java`

## Flow

```
addnewConfirmationList()
  1. System checks: isDailyUpdate(), isProcessing()
  2. Per-stock validation: validateForRetrieval()
  3. Build RetrievalScheduleParam[] (job_type=40, dest=1303)
  4. WebUnplannedRetrievalScheduler.schedule()
  5. QC-specific: update DNStock (status->QI, storage_location->QC01) + insert DNStockHistory
  6. doCommit()
  7. SendRequestor.retrieval() -> triggers RetrievalSender -> ID12
```

## Validation (validateForRetrieval - separate method)

Single fresh DB query per stock (stock + pallet join by stock_id + last_update_date):

| Check | Condition | Message |
|-------|-----------|---------|
| Duplicate | stock_id already in setting list | 6463164 |
| Freshness | stock_id + last_update_date must match DB | 6463019 |
| Stock Status | Must be **UU** (Unrestricted Used) | 6463049 |
| Tempering Flag | Must be **1** (Reached) | 6463049 |
| Allocation | Pallet must NOT be allocated | 6463083 |
| Station | retrievalStationCheck (AGC online, retrieval mode, not suspended) | varies |

## Retrieval Scheduling

Uses same pattern as `InquiryRetrievalSCH.addnewStart()`:

- `RetrievalScheduleParamMaker.makeParam()` with `job_type = ASRS_INVENTORYCHECK` (40)
- `SchedulerFactory.makeRetrievalScheduler(UNPLANNED_RETRIEVAL, ...)`
- `WebUnplannedRetrievalScheduler.schedule()` creates: DNWorkInfo, DNWorkList, DNCarryInfo, updates DNPallet
- After scheduler: update DNStock (status->QI, storage_location->QC01), insert DNStockHistory
- `SendRequestor.retrieval()` triggers RetrievalSender

---

# 2. Routing Analysis - Cross-Warehouse Retrieval to 1303

## Physical Layout

All QC retrieval goes to station **1303** (QC Station, `wh_station_no=9200`).

| Source Aisles | Warehouse | Physical Path to 1303 | Status |
|---|---|---|---|
| 9007-9010 (FGW1 Ambient) | 9200 | Direct via STV | **WORKING** - routes exist |
| 9001-9006 (FGW2 Tempering) | 9100 | Via STV to **any** of 7207-7210, then to 1303 | **IMPLEMENTED** - Option B |
| 9011-9014 (PM Ambient) | 9200 | Via STV to **any** of 7207-7210, then to 1303 | **IMPLEMENTED** - Option B |

### Key Physical Fact

Aisles 9001-9006 and 9011-9014 can physically reach **any** of 7207-7210 via STV (not 1:1 fixed). This means the relay station should be **dynamically selected** for load balancing.

## Existing Routes (DMRouteId)

**Direct routes that work:**
```
9007->1303   9008->1303   9009->1303   9010->1303
```

**Cross-warehouse routes (added for QC retrieval):**
```
9001->7207   9001->7208   9001->7209   9001->7210   (already existed)
9002->7207   9002->7208   9002->7209   9002->7210   (already existed)
...
9006->7207   9006->7208   9006->7209   9006->7210   (already existed)

9011->7207   9011->7208   9011->7209   9011->7210   (NEW — physical routes)
9012->7207   9012->7208   9012->7209   9012->7210   (NEW — physical routes)
9013->7207   9013->7208   9013->7209   9013->7210   (NEW — physical routes)
9014->7207   9014->7208   9014->7209   9014->7210   (NEW — physical routes)

7207->1303   7208->1303   7209->1303   7210->1303   (NEW — physical routes)

9001->1303v  9002->1303v  9003->1303v  9004->1303v  (NEW — virtual routes for scheduler)
9005->1303v  9006->1303v                              (NEW — virtual routes for scheduler)
9011->1303v  9012->1303v  9013->1303v  9014->1303v  (NEW — virtual routes for scheduler)
```

**SQL file:** `sqlscript/oracle/3.initdata/wms/QCRetrievalRoutes.sql`

**Existing storage routes (for reference):**
```
7207->9007   7208->9008   7209->9009   7210->9010   (1:1 storage inbound)
7211->9011   7212->9012   7213->9013   7214->9014   (1:1 storage inbound)
```

## Why DMRouteDetail Relay Approach Is Not Sufficient

`RouteCheckerImpl` supports relay points via `DMRouteDetail`, but each `DMRouteId` has a **fixed** relay station. Example: `DMRouteId(9001->1303)` with relay `7207` means 9001 **always** goes through 7207.

Problems:
- No load balancing across 7207-7210
- If 7207 is busy with storage, all retrieval from 9001 stacks behind it
- Cannot consider both storage and retrieval workload when choosing relay

**Conclusion:** Relay station selection must be handled **in code**, not by fixed DB routing.

---

# 3. Chosen Approach: Option B — Selection in RetrievalSender

**Decision date:** 2026-04-04
**Branch:** `6619-control-transfer-9100-1303`

## Why Option B

Selection at **send time** (in RetrievalSender) provides the most up-to-date workload information. The scheduler creates the carry with `dest=1303, end=1303` using virtual routes (`9001->1303v` etc.) for route validation — no changes to QCStartSCH or the scheduling framework. RetrievalSender intercepts **before** the route check via `isIntermediateRetrievalNeeded()` and dynamically picks the least-busy 720x.

### Virtual Routes

Aisles 9001-9006 and 9011-9014 have no physical path to 1303. Virtual DMRouteId entries (`9001->1303v` .. `9014->1303v`) exist **only** for scheduler validation (`WebUnplannedRetrievalScheduler`). They have minimal DMRouteDetail (source station only) so `RouteDB.check()` returns ACTIVE. RetrievalSender never uses these routes — it intercepts before the route check and redirects to 720x.

## Implemented Flow

```
QCStartSCH.addnewConfirmationList()
  → scheduler.schedule() creates carry: dest=1303, end=1303, carry_flag=RETRIEVAL
  → RetrievalSender.destDetermine()
     → isIntermediateRetrievalNeeded(): aisle IN (9001-9006, 9011-9014) → true
       (checked BEFORE retrievalDetermin — virtual routes exist but are not used)
     → handleIntermediateRetrieval():
        1. countCarriesHeadingToStation("1303") — count dest=1303 OR end=1303
           If count >= 1303.max_pallet_qty(1) → DEST_FULL, ROLLBACK (deadlock prevention)
        2. selectLeastBusyIntermediate(aisle) — find 720x routes, count dest OR source per 720x
           Pick station with lowest active carry count
        3. Validate DMRouteId: aisle → selected 720x
        4. Update carry: dest=720x, end stays 1303
     → sends ID12 from aisle to 720x
  → ID26 at 720x: AsrsInboundStationOperator.arrival()
     → carry_flag=RETRIEVAL → updateCarryInfoForRetrievalForwarding()
        carry_flag: RETRIEVAL(2) → DIRECT_TRAVEL(3)
        work_type:  stays 40 (INVENTORY_CHECK — NOT changed to 26)
        source:     → this 720x
        dest:       → 1303 (from end_station_no)
        end:        → 1303
        cmd_status: → START(1)
     → registArrival + carryRequest()
  → StorageSender picks up DIRECT_TRAVEL carry → ID05 from 720x to 1303
  → ID68 at 1303: OperationDisplay created (operation_display=3 AUTO_COMP)
  → [QC Work screen — not yet implemented]
  → ID45 sent to AGC (payout type decided by WareNavi)
  → ID26 at 1303: InOutStationOperator.arrival()
     → DIRECT_TRAVEL + end=1303 + INSTRUCTION mode:
       work_type=40 → updateArrival() → ReturnStorageManager
       work_type=26 → LoadRemover.remove() (reject)
     → ReturnStorageManager: carry→STORAGE, dest=pallet.wh (9100 or 9200)
     → registArrival + carryRequest → StorageSender returns pallet to ASRS
```

## Modified Files

### `RetrievalSender.java` — Cross-warehouse intermediate routing

New methods added to `destDetermine()` flow:

| Method | Purpose |
|--------|---------|
| `isIntermediateRetrievalNeeded(con, cInfo)` | Returns true if aisle is 9001-9006 or 9011-9014. Called **before** `retrievalDetermin()` to intercept cross-warehouse carries |
| `handleIntermediateRetrieval(con, cInfo, destSt)` | Orchestrator: checks 1303 capacity → selects 720x → validates route → updates carry dest |
| `countCarriesHeadingToStation(con, stationNo)` | Counts active carries where `dest=stationNo OR end=stationNo`. Used for 1303 deadlock prevention |
| `selectLeastBusyIntermediate(con, aisleNo)` | Finds routes aisle→720x, filters by HP_INTERMEDIATE + NORMAL + not suspended. Counts active carries where `dest=720x OR source=720x`, picks lowest |

**Deadlock prevention:** Station 1303 has `max_pallet_qty=1`. Before routing any cross-warehouse retrieval, `countCarriesHeadingToStation("1303")` counts ALL active carries heading to 1303 (direct retrievals, in-transit cross-WH carries, DIRECT_TRAVEL forwards). If count >= max, carry waits with `wait_reason=DEST_FULL(09)`.

**Least-busy logic:** Counts carries by `dest OR source` for each 720x candidate. This captures both inbound carries heading TO the station and outbound carries currently AT it (e.g., STORAGE carries with source=720x).

### `AsrsInboundStationOperator.java` — RETRIEVAL forwarding at 720x

New `RETRIEVAL` case in `arrival()` method, before existing STORAGE case:

```java
if (DNCarryInfo.CARRY_FLAG.RETRIEVAL.equals(ci.getCarryFlag()))
{
    updateCarryInfoForRetrievalForwarding(ci);
    registArrival(ci, plt);
    carryRequest();
}
```

`updateCarryInfoForRetrievalForwarding()` converts the carry in-place (same carry_key):
- `carry_flag`: RETRIEVAL(2) → DIRECT_TRAVEL(3) — for StorageSender transport
- `work_type`: stays **40 (INVENTORY_CHECK)** — NOT changed to 26. This distinguishes QC from reject at 1303
- `source`: → this 720x station
- `dest`: → end_station_no (1303)
- `end`: → 1303
- `cmd_status`: → START(1)

**No conflict with storage flow:** StorageSender queries `carry_flag IN (STORAGE, DIRECT_TRAVEL)`. The DIRECT_TRAVEL carry to 1303 is picked up by StorageSender, separate from storage carries.

### `InOutStationOperator.java` — DIRECT_TRAVEL + INSTRUCTION at 1303

Added `else` block in `arrival()` for DIRECT_TRAVEL when `operation_display` is INSTRUCTION (not NONE/DISP_ONLY):

```java
if (NONE || DISP_ONLY) {
    LoadRemover.remove(ci);     // reject — unchanged
} else {
    if (work_type == INVENTORY_CHECK) {
        updateArrival(ci);      // QC → ReturnStorageManager → return to ASRS
    } else {
        LoadRemover.remove(ci); // reject with display
    }
}
```

This ensures:
- **QC cross-warehouse** (work_type=40): pallet returns to ASRS via ReturnStorageManager
- **Reject** (work_type=26): pallet removed by LoadRemover — same as before

### `ReturnStorageManager.java` — Return to correct warehouse

`determineDestStationNo()` changed to use `pallet.getWhStationNo()` instead of `station.getWhStationNo()`:

```java
// Old: always returns "9200" (station 1303's wh)
return station.getWhStationNo();

// New: returns pallet's original warehouse
return pallet.getWhStationNo();  // "9100" for Tempering, "9200" for Ambient/PM
```

| Pallet from | pallet.wh | Returns to | Route |
|-------------|-----------|------------|-------|
| 9001-9006 (Tempering) | 9100 | 9100 | 1303→9001-9006 |
| 9007-9010 (Ambient) | 9200 | 9200 | 1303→9007-9010 |
| 9011-9014 (PM) | 9200 | 9200 | 1303→9007-9010 (no route to 9011-9014 from 1303) |

## DB Changes

### Routes — `QCRetrievalRoutes.sql`

| Route Name | Path | Count | Physical Detail |
|-----------|------|-------|-----------------|
| 7207->1303 .. 7210->1303 | 7207-7210 → 1303 | 4 | BCR → SRM → conveyor → STV(8101) → conveyor → 1303 |
| 9011->7207 .. 9011->7210 | 9011 → 7207-7210 | 4 | SRM → 1AA160 → 1BA220 → STV(SLA00111/8102) → BCR |
| 9012->7207 .. 9012->7210 | 9012 → 7207-7210 | 4 | SRM → 1AA180 → 1BA240 → STV(SLA00112/8102) → BCR |
| 9013->7207 .. 9013->7210 | 9013 → 7207-7210 | 4 | SRM → 1AA200 → 1BA260 → STV(SLA00113/8102) → BCR |
| 9014->7207 .. 9014->7210 | 9014 → 7207-7210 | 4 | SRM → 1AA220 → 1BA280 → STV(SLA00101/8102) → BCR |
| 9001->1303v .. 9006->1303v | Virtual: 9001-9006 → 1303 | 6 | Scheduler validation only (minimal detail) |
| 9011->1303v .. 9014->1303v | Virtual: 9011-9014 → 1303 | 4 | Scheduler validation only (minimal detail) |

Total: 30 new routes — 20 physical (DMRouteId + DMRouteDetail) + 10 virtual.

Routes 9001-9006 → 7207-7210 already existed (9001->7207 .. 9006->7210).

### Station 1303 config — `QCRetrievalRoutes.sql`

```sql
UPDATE DMStation SET max_pallet_qty = 1 WHERE station_no = '1303';
UPDATE DMStation SET operation_display = '3' WHERE station_no = '1303';       -- AUTO_COMP (MC decides)
UPDATE DMStation SET restoring_instruction = '1' WHERE station_no = '1303';   -- WN_STORAGE_SEND
UPDATE DMStation SET max_instruction = 1 WHERE station_no = '1303';           -- StorageSender needs >=1
```

| Setting | Value | Why |
|---------|-------|-----|
| max_pallet_qty | 1 | Deadlock prevention — only 1 pallet heading to 1303 at a time |
| operation_display | 3 (AUTO_COMP) | Enables ID68→OperationDisplay→ID45→ID26 flow. MC decides payout type |
| restoring_instruction | 1 (WN_STORAGE_SEND) | StorageSender accepts ARRIVAL carries for return-to-ASRS |
| max_instruction | 1 | StorageSender requires >=1 to send carries from 1303 |

## Test Results — 21/21 Pass

**SQL files:** `WEB-INF/test/resources/qcretrieval/sql/`

### QCRetrievalTest (8 tests) — unit tests for routing methods

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | `testDbConfig_Routes` | 30 routes exist + 1303 max_pallet=1 |
| 2 | `testIsIntermediateNeeded` | 9001/9011 → true, 9007/9010 → false |
| 3 | `testCrossWarehouse_9001_Via720x` | dest updated to 7207-7210, end stays 1303 |
| 4 | `testCrossWarehouse_9011_Via720x` | PM aisle via 720x |
| 5 | `test1303Full_DestFull` | Deadlock prevention: DEST_FULL(09) |
| 6 | `testLeastBusy_720x_Selection` | Load balancing across 720x |
| 7 | `testArrival_7207_RetrievalForward` | RETRIEVAL→DIRECT_TRAVEL, work_type stays 40 |
| 8 | `testArrival_7209_RetrievalForward_PM` | PM aisle forwarding |

### QCRetrievalFullFlowTest (5 tests) — end-to-end chain

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | `testDbPrerequisites` | Virtual + physical + direct routes all exist |
| 2 | `testFullFlow_9001_CrossWarehouse` | Scheduler→Sender(dest→720x)→Operator(DIRECT_TRAVEL dest=1303, work_type=40) |
| 3 | `testFullFlow_9011_CrossWarehouse` | Same for PM aisle 9011 |
| 4 | `testFullFlow_9007_Direct` | 9007→1303 direct, no intermediate |
| 5 | `testFullFlow_9010_Direct` | 9010→1303 direct |

### QCReturnStorageTest (5 tests) — return storage routing at 1303

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | `testDbConfig_1303_ReturnStorage` | operation_display=3, restoring_instruction=1, max_instruction>=1 |
| 2 | `testDbConfig_Routes_1303_To_Aisles` | Routes 1303→9001-9010 exist |
| 3 | `testDirectTravel_WorkType40_NotRemoved` | QC carry (work_type=40) NOT removed at 1303 |
| 4 | `testDirectTravel_WorkType26_Removed` | Reject carry (work_type=26) removed at 1303 |
| 5 | `testReturnDestination_PalletWh9100` | Pallet wh determines return warehouse |

### QCStorageSenderReturnTest (3 tests) — StorageSender picks up return carry

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | `testStorageSenderFindsReturnCarry_9100` | STORAGE carry at 1303 found by StorageSender search |
| 2 | `testStorageSenderDestDetermine_Return9100` | destDetermine: route 1303→WH 9100 works (no NO_ROUTE) |
| 3 | `testStorageSenderDestDetermine_Return9200` | destDetermine: route 1303→WH 9200 works (no NO_ROUTE) |

---

# 4. Action Items Summary

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | QCStartSCH validation + retrieval | **DONE** | Branch: 5137-qcsetting |
| 2 | Choose intermediate selection strategy | **DONE** | Option B — RetrievalSender (at send time) |
| 3 | DMRouteId: 720x → 1303 routes | **DONE** | 7207->1303 .. 7210->1303 |
| 4 | DMRouteId: 9011-9014 → 720x routes | **DONE** | 9011->7207 .. 9014->7210 |
| 4a | DMRouteId: Virtual routes for scheduler | **DONE** | 9001->1303v .. 9014->1303v |
| 5 | AsrsInboundStationOperator: forwarding (keep work_type=40) | **DONE** | `updateCarryInfoForRetrievalForwarding()` |
| 6 | RetrievalSender: intermediate selection + 1303 deadlock | **DONE** | `handleIntermediateRetrieval()` |
| 7 | DMStation 1303: max_pallet, operation_display, restoring, max_instruction | **DONE** | `QCRetrievalRoutes.sql` |
| 8 | InOutStationOperator: DIRECT_TRAVEL + INSTRUCTION (work_type=40 vs 26) | **DONE** | QC → return, reject → remove |
| 9 | ReturnStorageManager: pallet.wh for return destination | **DONE** | 9100 pallets return to 9100, not 9200 |
| 10 | StorageSender: verify return carry pickup + destDetermine | **DONE** | Routes 1303→9001-9010 work |
| 11 | Integration test: 21/21 pass | **DONE** | 4 test classes |
| 12 | QC Work screen (operator input) | TODO | Takes sample qty, decides return vs removal |
| 13 | As21Id45 dynamic payout type | TODO | RETURN_STORAGE vs PAYOUT based on operator decision |
| 14 | End-to-end test with AGC simulator | TODO | Verify ID12/ID26/ID05/ID45/ID68 message exchange |

---

# 5. Comparison Matrix (for reference)

| Criteria | Option A (SCH) | **Option B (Sender)** ✓ | Option C (Selector) | Option D (AGC decides) |
|---|---|---|---|---|
| Load balance accuracy | Good (at schedule time) | **Best (at send time)** | Good (at schedule time) | Depends on AGC |
| Considers storage+retrieval | Yes (count all carries) | **Yes** | Yes | AGC decides |
| Code complexity | Low | **Medium** | Medium | Low |
| Framework risk | Low | **Medium** (RetrievalSender) | Medium | Low |
| DB routes needed | 40+4 | **16+4+10 virtual** (9011-9014→720x + 720x→1303 + scheduler validation) | 40+4 | 10 virtual + 4 |
| Reusable for other screens | No (SCH-specific) | **Yes (all retrieval)** | Yes (scheduler level) | Yes (operator level) |
| AGC dependency | Low | **Low** | Low | High |

---

# References

- [Retrieval for QC Start - DFD](Retrieval-for-QC-Start)
- [QC Work from Retrieval for QC Start](QC-Work-from-Retrieval-for-QC-Start)
- RetrievalSender: `WEB-INF/src/jp/co/daifuku/asrs/transmission/RetrievalSender.java`
- AsrsInboundStationOperator: `WEB-INF/src/jp/co/daifuku/asrs/location/AsrsInboundStationOperator.java`
- InOutStationOperator: `WEB-INF/src/jp/co/daifuku/asrs/location/InOutStationOperator.java`
- ReturnStorageManager: `WEB-INF/src/jp/co/daifuku/asrs/location/ReturnStorageManager.java`
- StorageSender: `WEB-INF/src/jp/co/daifuku/asrs/transmission/StorageSender.java`
- QCRetrievalRoutes SQL: `sqlscript/oracle/3.initdata/wms/QCRetrievalRoutes.sql`
- Tests: `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCRetrievalTest.java`
- Tests: `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCRetrievalFullFlowTest.java`
- Tests: `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCReturnStorageTest.java`
- Tests: `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCStorageSenderReturnTest.java`
- RetrievalRouteControllerImpl: `WEB-INF/src/jp/co/daifuku/asrs/location/route/controller/RetrievalRouteControllerImpl.java`
- RouteCheckerImpl: `WEB-INF/src/jp/co/daifuku/asrs/location/route/RouteCheckerImpl.java`
- AutoStorageScheduler (selectLeastBusyIntermediateStation): `WEB-INF/src/jp/co/daifuku/asrs/transmission/AutoStorageScheduler.java`
