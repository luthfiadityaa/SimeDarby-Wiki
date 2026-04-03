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
9001->7207   9001->7208   9001->7209   9001->7210   (routes 449-452, already existed)
9002->7207   9002->7208   9002->7209   9002->7210   (routes 453-456, already existed)
...
9006->7207   9006->7208   9006->7209   9006->7210   (routes 469-472, already existed)

9011->7207   9011->7208   9011->7209   9011->7210   (routes 501-504, NEW)
9012->7207   9012->7208   9012->7209   9012->7210   (routes 505-508, NEW)
9013->7207   9013->7208   9013->7209   9013->7210   (routes 509-512, NEW)
9014->7207   9014->7208   9014->7209   9014->7210   (routes 513-516, NEW)

7207->1303   7208->1303   7209->1303   7210->1303   (routes 497-500, NEW)
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

Selection at **send time** (in RetrievalSender) provides the most up-to-date workload information. The scheduler creates the carry with `dest=1303, end=1303` normally — no changes to QCStartSCH or the scheduling framework. RetrievalSender intercepts when the route check fails and dynamically picks the least-busy 720x.

## Implemented Flow

```
QCStartSCH.addnewConfirmationList()
  → scheduler.schedule() creates carry: dest=1303, end=1303, carry_flag=RETRIEVAL
  → RetrievalSender.destDetermine()
     → retrievalDetermin(pallet, endSt=1303): routeOK = false (no route 9001→1303)
     → isIntermediateRetrievalNeeded(): aisle IN (9001-9006, 9011-9014) → true
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
        work_type:  → 26 (DIRECT_TRAVEL)
        source:     → this 720x
        dest:       → 1303 (from end_station_no)
        end:        → 1303
        cmd_status: → START(1)
     → registArrival + carryRequest()
  → StorageSender picks up DIRECT_TRAVEL carry → ID05 from 720x to 1303
  → ID26 at 1303: InOutStationOperator handles arrival, completes work
```

## Modified Files

### `RetrievalSender.java` — Cross-warehouse intermediate routing

New methods added to `destDetermine()` flow:

| Method | Purpose |
|--------|---------|
| `isIntermediateRetrievalNeeded(con, cInfo)` | Returns true if aisle is 9001-9006 or 9011-9014 (no direct route to 1303) |
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
- `carry_flag`: RETRIEVAL(2) → DIRECT_TRAVEL(3)
- `work_type`: → DIRECT_TRAVEL(26)
- `source`: → this 720x station
- `dest`: → end_station_no (1303)
- `end`: → 1303
- `cmd_status`: → START(1)

**No conflict with storage flow:** StorageSender queries `carry_flag IN (STORAGE, DIRECT_TRAVEL)`. The new DIRECT_TRAVEL carry to 1303 is picked up by StorageSender, separate from storage carries.

## DB Changes

### Routes — `QCRetrievalRoutes.sql`

| Route IDs | Path | Count | Physical Detail |
|-----------|------|-------|-----------------|
| 497-500 | 7207-7210 → 1303 | 4 | BCR → SRM → conveyor → STV(8101) → conveyor → 1303 |
| 501-504 | 9011 → 7207-7210 | 4 | SRM → 1AA160 → 1BA220 → STV(SLA00111/8102) → BCR |
| 505-508 | 9012 → 7207-7210 | 4 | SRM → 1AA180 → 1BA240 → STV(SLA00112/8102) → BCR |
| 509-512 | 9013 → 7207-7210 | 4 | SRM → 1AA200 → 1BA260 → STV(SLA00113/8102) → BCR |
| 513-516 | 9014 → 7207-7210 | 4 | SRM → 1AA220 → 1BA280 → STV(SLA00101/8102) → BCR |

Total: 20 new routes (DMRouteId + DMRouteDetail).

Routes 9001-9006 → 7207-7210 already existed (routes 449-472).

### Station update

```sql
UPDATE DMStation SET max_pallet_qty = 1 WHERE station_no = '1303';
```

## Test Results — 8/8 Pass

**Test class:** `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCRetrievalTest.java`
**SQL files:** `WEB-INF/test/resources/qcretrieval/sql/`

| # | Test | What it verifies |
|---|------|-----------------|
| 1 | `testDbConfig_Routes` | All 20 new routes exist + 24 existing 9001-9006→720x routes + 1303 max_pallet=1 |
| 2 | `testIsIntermediateNeeded` | 9001/9011 → true (need intermediate), 9007/9010 → false (direct route) |
| 3 | `testCrossWarehouse_9001_Via720x` | Aisle 9001 → handleIntermediateRetrieval → dest updated to 7207-7210, end stays 1303 |
| 4 | `testCrossWarehouse_9011_Via720x` | Aisle 9011 (PM) → same logic → dest updated to 7207-7210, end stays 1303 |
| 5 | `test1303Full_DestFull` | Blocking carry at 1303 → ROLLBACK, wait_reason=DEST_FULL(09) |
| 6 | `testLeastBusy_720x_Selection` | 7207 has 2 carries, 7208 has 1 → selects 7209 or 7210 (0 carries) |
| 7 | `testArrival_7207_RetrievalForward` | RETRIEVAL at 7207 → DIRECT_TRAVEL, source=7207, dest=1303, work_type=26 |
| 8 | `testArrival_7209_RetrievalForward_PM` | RETRIEVAL at 7209 from PM aisle 9013 → same forwarding |

---

# 4. Action Items Summary

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | QCStartSCH validation + retrieval | **DONE** | Branch: 5137-qcsetting |
| 2 | Choose intermediate selection strategy | **DONE** | Option B — RetrievalSender (at send time) |
| 3 | DMRouteId: 720x → 1303 routes | **DONE** | Routes 497-500, `QCRetrievalRoutes.sql` |
| 4 | DMRouteId: 9011-9014 → 720x routes | **DONE** | Routes 501-516, `QCRetrievalRoutes.sql` |
| 5 | AsrsInboundStationOperator: RETRIEVAL forwarding | **DONE** | `updateCarryInfoForRetrievalForwarding()` |
| 6 | RetrievalSender: intermediate selection + 1303 deadlock | **DONE** | `handleIntermediateRetrieval()`, `selectLeastBusyIntermediate()` |
| 7 | DMStation: 1303 max_pallet_qty=1 | **DONE** | `QCRetrievalRoutes.sql` |
| 8 | Integration test: 8/8 pass | **DONE** | `QCRetrievalTest.java` |
| 9 | Test direct retrieval (9007-9010 → 1303) | TODO | Verify full scheduler + RetrievalSender + InOutStationOperator flow |
| 10 | End-to-end test with AGC simulator | TODO | Verify ID12/ID26/ID05 message exchange |

---

# 5. Comparison Matrix (for reference)

| Criteria | Option A (SCH) | **Option B (Sender)** ✓ | Option C (Selector) | Option D (AGC decides) |
|---|---|---|---|---|
| Load balance accuracy | Good (at schedule time) | **Best (at send time)** | Good (at schedule time) | Depends on AGC |
| Considers storage+retrieval | Yes (count all carries) | **Yes** | Yes | AGC decides |
| Code complexity | Low | **Medium** | Medium | Low |
| Framework risk | Low | **Medium** (RetrievalSender) | Medium | Low |
| DB routes needed | 40+4 | **16+4** (9011-9014→720x + 720x→1303) | 40+4 | 10 virtual + 4 |
| Reusable for other screens | No (SCH-specific) | **Yes (all retrieval)** | Yes (scheduler level) | Yes (operator level) |
| AGC dependency | Low | **Low** | Low | High |

---

# References

- [Retrieval for QC Start - DFD](Retrieval-for-QC-Start)
- [QC Work from Retrieval for QC Start](QC-Work-from-Retrieval-for-QC-Start)
- RetrievalSender: `WEB-INF/src/jp/co/daifuku/asrs/transmission/RetrievalSender.java`
- AsrsInboundStationOperator: `WEB-INF/src/jp/co/daifuku/asrs/location/AsrsInboundStationOperator.java`
- QCRetrievalRoutes SQL: `sqlscript/oracle/3.initdata/wms/QCRetrievalRoutes.sql`
- QCRetrievalTest: `WEB-INF/test/jp/co/daifuku/asrs/transmission/QCRetrievalTest.java`
- RetrievalRouteControllerImpl: `WEB-INF/src/jp/co/daifuku/asrs/location/route/controller/RetrievalRouteControllerImpl.java`
- RouteCheckerImpl: `WEB-INF/src/jp/co/daifuku/asrs/location/route/RouteCheckerImpl.java`
- AutoStorageScheduler (selectLeastBusyIntermediateStation): `WEB-INF/src/jp/co/daifuku/asrs/transmission/AutoStorageScheduler.java`
