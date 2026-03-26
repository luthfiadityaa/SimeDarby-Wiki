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
| 9001-9006 (FGW2 Tempering) | 9100 | Via STV to **any** of 7207-7210, then to 1303 | **NOT WORKING** |
| 9011-9014 (PM Ambient) | 9200 | Via STV to **any** of 7207-7210, then to 1303 | **NOT WORKING** |

### Key Physical Fact

Aisles 9001-9006 and 9011-9014 can physically reach **any** of 7207-7210 via STV (not 1:1 fixed). This means the relay station should be **dynamically selected** for load balancing.

## Existing Routes (DMRouteId)

**Direct routes that work:**
```
9007->1303   9008->1303   9009->1303   9010->1303
```

**Missing routes (no direct physical path, must go through 7207-7210):**
```
9001->1303   9002->1303   9003->1303   9004->1303   9005->1303   9006->1303
9011->1303   9012->1303   9013->1303   9014->1303
```

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

# 3. Intermediate Station Selection - Solution Options

## Shared Requirement (all options)

Regardless of which option is chosen:
- 7207-7210 need `max_instruction >= 1` for StorageSender to send ID05
- 7207-7210 need a station operator that handles RETRIEVAL carry arrival and creates DIRECT_TRAVEL carry to 1303
- Routes `7207->1303`, `7208->1303`, `7209->1303`, `7210->1303` must exist in DMRouteId (for the DIRECT_TRAVEL leg)

## Option A: Selection in QCStartSCH (at scheduling time)

**Concept:** Before calling `WebUnplannedRetrievalScheduler.schedule()`, QCStartSCH determines the intermediate 720x station and passes it as destStationNo. The scheduler creates DNCarryInfo with `dest=720x, end=1303`.

**Flow:**
```
QCStartSCH.addnewConfirmationList()
  -> selectLeastBusyIntermediate(sourceAisle, "1303")
     -> count active DNCarryInfo per 7207-7210 (both STORAGE + RETRIEVAL)
     -> pick station with lowest count
  -> RetrievalScheduleParamMaker.makeParam(whStationNo, dest=720x, ...)
  -> scheduler.schedule() creates carry: dest=720x, end=1303, carry_flag=RETRIEVAL
  -> RetrievalSender sends ID12 from aisle to 720x
  -> ID26 at 720x: operator creates DIRECT_TRAVEL carry to 1303
  -> StorageSender sends ID05 from 720x to 1303
```

**Pros:**
- Selection logic centralized in SCH, easy to understand
- Scheduler receives a concrete dest station, no framework modification needed
- Can reuse `selectLeastBusyIntermediateStation()` pattern from AutoStorageScheduler

**Cons:**
- QCStartSCH needs to know which aisles require intermediate routing (9001-9006, 9011-9014 vs 9007-9010 direct)
- Need to create direct `DMRouteId` entries: `9001->7207`, `9001->7208`, etc. (for RouteChecker validation) — or bypass route validation for intermediate selection
- Selection happens at scheduling time, not at retrieval time — if station becomes busy between scheduling and actual retrieval, the choice may be suboptimal

**DB routes needed:**
```sql
-- Intermediate leg: aisle -> 720x (one per combination, 10 aisles x 4 stations = 40 routes)
-- 9001->7207, 9001->7208, 9001->7209, 9001->7210
-- 9002->7207, 9002->7208, ... etc.
-- 9011->7207, 9011->7208, ... etc.

-- Final leg: 720x -> 1303
-- 7207->1303, 7208->1303, 7209->1303, 7210->1303
```

## Option B: Selection in RetrievalSender (at ID12 send time)

**Concept:** The scheduler creates DNCarryInfo with `dest=1303, end=1303` as if it's a direct route. But `RetrievalSender.getSendCarryArray()` detects that aisle 9001 cannot reach 1303 directly and selects an intermediate 720x station, updating the carry's dest before sending ID12.

**Flow:**
```
QCStartSCH.addnewConfirmationList()
  -> scheduler.schedule() creates carry: dest=1303, end=1303, carry_flag=RETRIEVAL
  -> RetrievalSender.getSendCarryArray()
     -> checks route 9001->1303: no direct route
     -> selectLeastBusyIntermediate(9001, 1303)
     -> updates carry: dest=720x, end=1303
     -> sends ID12 from aisle to 720x
  -> ID26 at 720x: operator creates DIRECT_TRAVEL carry to 1303
  -> StorageSender sends ID05 from 720x to 1303
```

**Pros:**
- SCH code stays simple, no routing awareness needed
- Selection happens at send time (more up-to-date workload info)
- Works for any future retrieval screen, not just QC Start

**Cons:**
- Requires modifying RetrievalSender (core framework class, high risk)
- RetrievalSender currently does simple carry_flag=RETRIEVAL search with fixed dest — adding intermediate routing logic increases complexity
- Need a way to detect "needs intermediate" vs "direct" — cannot rely on DMRouteId since no direct route exists

## Option C: Selection in a new RetrievalIntermediateSelector (called by scheduler)

**Concept:** Create a dedicated class that the retrieval scheduler calls when it detects the route requires intermediate stations. Similar to how `AisleShelfDecider` works for storage.

**Flow:**
```
QCStartSCH.addnewConfirmationList()
  -> scheduler.schedule()
     -> RouteChecker: 9001->1303 no direct route -> returns false
     -> scheduler detects cross-warehouse retrieval
     -> RetrievalIntermediateSelector.selectStation(sourceAisle, finalDest)
        -> find candidate 720x stations reachable from sourceAisle
        -> count active DNCarryInfo per candidate (STORAGE + RETRIEVAL + DIRECT_TRAVEL)
        -> pick least busy
        -> return 720x
     -> creates carry: dest=720x, end=1303, carry_flag=RETRIEVAL
  -> RetrievalSender sends ID12 from aisle to 720x
  -> ID26 at 720x: operator creates DIRECT_TRAVEL carry to 1303
  -> StorageSender sends ID05 from 720x to 1303
```

**Pros:**
- Clean separation of concerns (selector class is testable, reusable)
- Works within existing scheduler framework (just adds a fallback when direct route fails)
- Can share counting logic with storage `selectLeastBusyIntermediateStation()`
- Easy to extend for other intermediate routing scenarios

**Cons:**
- Requires modifying `WebUnplannedRetrievalScheduler` or `RetrievalScheduleManager` to call the selector when direct route fails
- Need to define which stations are "intermediate candidates" — either by config or by querying DMStation/DMRouteId
- More classes to maintain

**DB routes needed:**
```sql
-- Aisle -> 720x routes (for route validation)
-- 9001->7207, 9001->7208, 9001->7209, 9001->7210, etc.

-- Final leg: 720x -> 1303
-- 7207->1303, 7208->1303, 7209->1303, 7210->1303
```

## Option D: Selection in AsrsInboundStationOperator (no intermediate routing in scheduler)

**Concept:** The scheduler routes 9001->1303 as a "virtual" direct route using a dummy DMRouteId entry. RetrievalSender sends ID12 with dest=1303. AGC physically delivers the pallet to one of 7207-7210 (AGC decides). When ID26 arrives at 720x, the operator handles forwarding.

**Flow:**
```
QCStartSCH.addnewConfirmationList()
  -> scheduler.schedule() creates carry: dest=1303, end=1303
  -> RetrievalSender sends ID12 from aisle, AGC routes to 720x physically
  -> ID26 at 720x: operator sees dest=1303 != this station
     -> creates DIRECT_TRAVEL carry: source=720x, dest=1303
     -> StorageSender sends ID05 from 720x to 1303
```

**Pros:**
- Simplest code change — no intermediate selection logic in WMS at all
- AGC handles physical routing and station selection (AGC already knows STV traffic)
- WMS only needs to handle the arrival at 720x and forward
- No new routes needed between aisles and 720x

**Cons:**
- Depends on AGC being smart enough to pick the right 720x (may not balance load)
- WMS loses visibility of which intermediate station will be used
- Need a "virtual" route 9001->1303 in DMRouteId for scheduler validation, but the physical path goes through 720x
- May not work if AGC requires WMS to specify the exact intermediate station in ID12

---

# 4. Comparison Matrix

| Criteria | Option A (SCH) | Option B (Sender) | Option C (Selector) | Option D (AGC decides) |
|---|---|---|---|---|
| Load balance accuracy | Good (at schedule time) | Best (at send time) | Good (at schedule time) | Depends on AGC |
| Considers storage+retrieval | Yes (count all carries) | Yes | Yes | AGC decides |
| Code complexity | Low | High (modify Sender) | Medium | Low |
| Framework risk | Low | High | Medium | Low |
| DB routes needed | 40 aisle->720x + 4 720x->1303 | 4 720x->1303 only | 40 aisle->720x + 4 720x->1303 | 10 aisle->1303 (virtual) + 4 720x->1303 |
| Reusable for other screens | No (SCH-specific) | Yes (all retrieval) | Yes (scheduler level) | Yes (operator level) |
| AGC dependency | Low | Low | Low | High |

---

# 5. Station Operator at 7207-7210 (Required for all options)

## Problem

When RETRIEVAL carry arrives at 7207-7210 via ID26, current `StorageStationOperator` just does `updateArrival()` — does NOT create a forward carry to 1303.

`InOutStationOperator` was tested — it calls `ReturnStorageManager` which **completes the retrieval work** at 7207-7210 and tries to re-store the pallet. This is wrong because work should complete at 1303, not 7207-7210.

## Solution: Add retrieval forwarding to AsrsInboundStationOperator

`AsrsInboundStationOperator` is already planned for 7207-7210 (handles storage inbound per CLAUDE.md). Add RETRIEVAL carry handling:

```java
// In AsrsInboundStationOperator.arrival():
if (DNCarryInfo.CARRY_FLAG.RETRIEVAL.equals(ci.getCarryFlag()))
{
    // QC retrieval arrived at intermediate 720x
    // 1. Complete old retrieval carry (delete or update cmd_status)
    // 2. Update DNPallet current_station_no = this station
    // 3. Create DNARRIVAL
    // 4. Create NEW DNCarryInfo:
    //      carry_flag   = DIRECT_TRAVEL (3)
    //      work_type    = 26 (Direct Transfer)
    //      cmd_status   = START (1)
    //      source       = this station (720x)
    //      dest         = end_station_no from old carry (1303)
    //      end          = same as dest (1303)
    // 5. carryRequest() -> StorageSender sends ID05
    handleRetrievalForwarding(ci, pallet);
}
else if (DNCarryInfo.CARRY_FLAG.DIRECT_TRAVEL.equals(ci.getCarryFlag()))
{
    // Existing storage inbound logic (Rule 5 from CLAUDE.md)
    ...
}
```

**No conflict with storage:** StorageSender queries `carry_flag IN (STORAGE, DIRECT_TRAVEL)` while RetrievalSender queries `carry_flag = RETRIEVAL`. The new DIRECT_TRAVEL carry to 1303 is picked up by StorageSender, completely separate from storage carries.

---

# 6. DB Changes Required (all options need these)

```sql
-- 1. Routes: 720x -> 1303 (for DIRECT_TRAVEL leg)
INSERT INTO DMRouteId (...) VALUES ('xxx', '01', '7207->1303', '7207', '1303', ...);
INSERT INTO DMRouteId (...) VALUES ('xxx', '01', '7208->1303', '7208', '1303', ...);
INSERT INTO DMRouteId (...) VALUES ('xxx', '01', '7209->1303', '7209', '1303', ...);
INSERT INTO DMRouteId (...) VALUES ('xxx', '01', '7210->1303', '7210', '1303', ...);
-- Plus DMRouteDetail entries for each

-- 2. max_instruction for StorageSender to send ID05
UPDATE DMStation SET max_instruction = 1
WHERE station_no IN ('7207','7208','7209','7210');

-- 3. class_name for retrieval + storage handling
UPDATE DMStation SET class_name = 'jp.co.daifuku.asrs.location.AsrsInboundStationOperator'
WHERE station_no IN ('7207','7208','7209','7210');
```

---

# 7. Action Items Summary

| # | Item | Status | Notes |
|---|------|--------|-------|
| 1 | QCStartSCH validation + retrieval | DONE | Branch: 5137-qcsetting |
| 2 | Choose intermediate selection strategy | **PENDING** | Option A / B / C / D — see comparison above |
| 3 | DMRouteId: 720x -> 1303 routes | TODO | Required for all options |
| 4 | AsrsInboundStationOperator: RETRIEVAL forwarding | TODO | Required for all options |
| 5 | DMStation: max_instruction, class_name for 720x | TODO | Required for all options |
| 6 | Aisle->720x routes (if Option A or C) | TODO | 40 routes if needed |
| 7 | Verify with AGC team: can WMS specify intermediate? | TODO | Affects Option D viability |
| 8 | selectLeastBusy: count both STORAGE + RETRIEVAL + DIRECT_TRAVEL | TODO | Shared concern for storage and retrieval |
| 9 | Test direct retrieval (9007-9010 -> 1303) first | TODO | Verify scheduler + SendRequestor flow |

---

# References

- [Retrieval for QC Start - DFD](Retrieval-for-QC-Start)
- [QC Work from Retrieval for QC Start](QC-Work-from-Retrieval-for-QC-Start)
- RetrievalRouteControllerImpl: `WEB-INF/src/jp/co/daifuku/asrs/location/route/controller/RetrievalRouteControllerImpl.java`
- RouteCheckerImpl (relay logic): `WEB-INF/src/jp/co/daifuku/asrs/location/route/RouteCheckerImpl.java`
- RouteDB (DMRouteDetail): `WEB-INF/src/jp/co/daifuku/asrs/location/route/RouteDB.java`
- InOutStationOperator: `WEB-INF/src/jp/co/daifuku/asrs/location/InOutStationOperator.java`
- ReturnStorageManager: `WEB-INF/src/jp/co/daifuku/asrs/location/ReturnStorageManager.java`
- StorageSender (ID05): `WEB-INF/src/jp/co/daifuku/asrs/transmission/StorageSender.java`
- RetrievalSender (ID12): `WEB-INF/src/jp/co/daifuku/asrs/transmission/RetrievalSender.java`
- AutoStorageScheduler (selectLeastBusyIntermediateStation): `WEB-INF/src/jp/co/daifuku/asrs/transmission/AutoStorageScheduler.java`
