[[_TOC_]]
[[_TOSP_]]

# Aisle Selection & BCR Routing Reference

This page documents how the system selects aisles and BCR stations for different operations. **This is critical shared knowledge** — incorrect settings cause pallets to be routed to wrong stations or shelves to be unallocable.

---

# <span style="color:red; font-weight:bold">BCR Station Selection — Two Different Methods</span>

Each aisle has multiple BCR stations (OP side and HP side). The correct BCR depends on the **physical route** the pallet takes.

## Method 1: DMAisle.bcr_station_no (Normal Inbound Storage)

Used by: **AutoStorageScheduler**, **StorageSender**, **AsrsInboundStationOperator**

```
DMAisle.bcr_station_no stores ONE BCR per aisle:
  9007 -> 7107 (OP BCR)
  9008 -> 7108 (OP BCR)
  9009 -> 7109 (OP BCR)
  9010 -> 7110 (OP BCR)
  9011 -> 7211 (HP BCR)
  9012 -> 7212 (HP BCR)
  9013 -> 7213 (HP BCR)
  9014 -> 7214 (HP BCR)
```

**When to use:** Pallet travels from inbound station (1301, 1302, 1106) to BCR to aisle. The pallet enters from the **same physical side** as the BCR. Route exists: `1301 -> 7207 -> 9007`.

For aisles 9007-9010 reachable from both OP and HP sides, `getReachableBcrForAisle()` checks DMRouteId to find the correct BCR based on source station:
```
source=1111 (OP), aisle=9007 -> route 1111->7107 exists -> return 7107
source=1301 (HP), aisle=9007 -> route 1301->7107 NOT exists
                              -> route 1301->7207 exists -> return 7207
```

## Method 2: getBcrForTransfer via DMRouteId (Cross-Warehouse Transfer)

Used by: **processTransferWarehouse** in RetrievalSender

```
Transfer from 9100 (Tempering) to 9200 (Ambient):
  Pallet retrieved from shelf in aisle 9001-9006 (OP side, WH 9100)
  Must travel via STV 8102 to HP BCR 7207-7214 (HP side, WH 9200)
  
  DMAisle.bcr_station_no for 9007 = 7107 (OP BCR)
  BUT route 9001->7107 does NOT exist!
  Route 9001->7207 DOES exist (via STV 8102)
```

**`getBcrForTransfer(sourceAisle, targetAisle)`** logic:
```
1. Query DMRouteId WHERE start_station_no = sourceAisle (e.g. 9001)
2. For each route end_station_no:
   Check DMStation WHERE station_no = end AND aisle_station_no = targetAisle
3. First match = correct BCR

Example:
  sourceAisle=9001, targetAisle=9007
  -> DMRouteId: 9001->7207 exists
  -> DMStation 7207: aisle_station_no = 9007 -> MATCH
  -> return "7207" ✓
```

::: mermaid
flowchart TD
    A{Which operation?}
    A -->|Normal Storage<br/>1301/1302 -> BCR -> aisle| B[getReachableBcrForAisle<br/>uses DMRouteId from source station]
    A -->|Transfer 9100->9200<br/>shelf -> STV -> BCR -> aisle| C[getBcrForTransfer<br/>uses DMRouteId from source AISLE]
    A -->|Palletize Storage<br/>111x -> BCR -> aisle| D[selectAisleAndGetBcrStation<br/>uses DMAisle.bcr_station_no<br/>+ DMRouteId validation]
    
    B --> E[7107-7110 or 7207-7214<br/>depends on source]
    C --> F[Always 7207-7214<br/>HP side for cross-WH]
    D --> G[7101-7110<br/>OP side]
:::

---

# <span style="color:red; font-weight:bold">Aisle Round-Robin — DMWareHouse.last_used_station_no</span>

## What It Does
After a successful shelf decision, the aisle used is stored in `DMWareHouse.last_used_station_no`. The next shelf search starts from the **next** aisle, cycling through all aisles.

## Who Updates It

| Writer | When | Example |
|--------|------|---------|
| `AbstractAisleSelector.determin()` | After ANY successful shelf decision (storage, R2R, transfer) | Normal storage stores to 9007 -> last_used=9007 |
| `processTransferWarehouse.updateLastUsedAisle()` | After transfer shelf booking | Transfer books in 9008 -> last_used=9008 |

## Shared Across ALL Operations

`last_used_station_no` is **ONE value per DMWareHouse**, shared by:

- Normal inbound storage (StorageSender -> LocationManager -> AisleShelfDecider)
- Transfer from Tempering (processTransferWarehouse -> RackToRackDecider)
- DoubleDeep rack-to-rack (DoubleDeepChecker -> LocationManager.searchRackToRack)

```
Timeline example for WH 9200 (last_used_station_no):

T1: Normal storage     -> stores to 9007 -> last_used = 9007
T2: Transfer           -> books in 9008  -> last_used = 9008
T3: Normal storage     -> starts at 9009 (skips 9007, 9008) -> stores to 9009
T4: Transfer           -> starts at 9010 -> books in 9010
T5: DoubleDeep R2R     -> starts at 9011 -> moves to 9011
T6: Normal storage     -> starts at 9012 -> stores to 9012
```

This is **correct behavior** — ensures even distribution across all aisles regardless of operation type.

## How Round-Robin Works in Code

```java
// getAvailableAisles() in RetrievalSender / AbstractAisleSelector:
String lastUsed = targetWh.getLastUsedStationNo();  // e.g. "9007"

// Query all aisles in WH, sorted by aisle_no:
//   [9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]

// Find index of lastUsed (0), start from next (1):
//   Rotated: [9008, 9009, 9010, 9011, 9012, 9013, 9014, 9007]
//   9007 is LAST (tried only if all others fail)
```

---

# <span style="color:red; font-weight:bold">Aisle Selector Comparison</span>

| Operation | Selector Class | Based On | Searches |
|-----------|---------------|----------|----------|
| Normal storage at BCR | Based on `DMWareHouse.aisle_decision_pattern` | Pattern 0: ConnectedAisleSelector<br/>Pattern 3: DisperseAisleSelector<br/>Pattern 4: WNCollectAisleSelector | All aisles in WH, round-robin |
| Transfer (R2R cross-WH) | Always `RackToRackAisleSelector` | Pallet current station set to target aisle | One aisle at a time, iterated by caller |
| DoubleDeep R2R | Always `RackToRackAisleSelector` | Pallet current station = source shelf -> aisle | Same WH aisles |

### Key Difference: RackToRackAisleSelector
When pallet `currentStationNo` is an **Aisle** instance, `RackToRackAisleSelector` returns **only that one aisle**. This is why `processTransferWarehouse` iterates aisles externally and calls `searchRearShelfInAisle(aisle)` per aisle — the decider searches only the aisle we tell it.

---

# <span style="color:red; font-weight:bold">Shelf Search — Double Deep Specifics</span>

Aisles 9007-9014 are **double deep**. Shelf search uses `DoubleDeepShelfSelector`.

## Normal Storage: `searchEmptyShelf()`
- Fills rear first, then front
- When rear is OCCUPIED and front is EMPTY -> front is selectable
- When rear is RESERVED (not yet stored) -> front is NOT selectable

## Transfer R2R: `searchShelfRackToRack()` -> `findEmptyShelfForRackToRack()`
- Only considers **empty pairs** (both rear + front = EMPTY)
- Requires `MIN_EMPTY_PAIR_DIFFERENT_ZONE = 2` empty pairs minimum
- Returns the **rear** shelf of the first available empty pair
- After reservation (status->RESERVED), that pair is excluded from future searches

## Why Transfer Needs Empty Pairs
Transfer books shelves **before retrieval starts** (pallet still in source WH). The shelf stays RESERVED during the entire retrieval + transport time. Requiring empty pairs ensures:
1. Buffer for normal storage operations (at least 1 pair stays available)
2. No conflict with front/rear ordering

---

# <span style="color:red; font-weight:bold">DB Settings Checklist</span>

Settings that MUST be correct for aisle/BCR selection to work:

| Setting | Table.Column | Required Value | Impact if Wrong |
|---------|-------------|---------------|-----------------|
| Aisle status | `DMAisle.status` | `1` (NORMAL) | Aisle skipped by ALL operations |
| Aisle double_deep_kind | `DMAisle.double_deep_kind` | `1` (DOUBLE) for 9007-9014 | Wrong ShelfSelector used |
| Aisle decision pattern | `DMWareHouse.aisle_decision_pattern` | `4` (WNCollect) for 9200 | Normal storage uses wrong selector |
| Zone manage type | `DMWareHouse.zone_manage_type` | `2` (SOFT_ZONE) for 9200 | Zone matching fails |
| Direction | `DMWareHouse.direction` | `13` (BAY_HP) for 9200 | Shelf search order wrong, may throw exception |
| Pallet soft_zone | `DNPallet.soft_zone_id` | Must match target WH zones | No shelf found (e.g. zone 001 has no fallback in 9200) |
| Soft zone priority | `DMSoftZonePriority` | Zone 002/004/005 entries for WH 9200 | Zone fallback fails |
| Transfer routes | `DMRouteId` | 9001-9006 -> 7207-7214 (route_type=2) | `getBcrForTransfer` returns null |
| BCR aisle mapping | `DMStation.aisle_station_no` | 7207->9007, 7208->9008, etc. | BCR-to-aisle match fails |
| BCR station in DMAisle | `DMAisle.bcr_station_no` | 9007->7107, 9011->7211, etc. | Normal storage gets wrong BCR (not transfer) |

## Verification SQL
```sql
-- Check aisle status
SELECT station_no, status, double_deep_kind FROM DMAISLE 
WHERE wh_station_no = '9200' ORDER BY station_no;

-- Check WH settings
SELECT station_no, aisle_decision_pattern, zone_manage_type, direction, last_used_station_no 
FROM DMWAREHOUSE WHERE station_no = '9200';

-- Check soft zone priorities for WH 9200
SELECT soft_zone_id, priority_soft_zone, priority 
FROM DMSOFTZONEPRIORITY WHERE wh_station_no = '9200' ORDER BY soft_zone_id, priority;

-- Check transfer routes exist
SELECT COUNT(*) AS transfer_routes FROM DMROUTEID 
WHERE start_station_no IN ('9001','9002','9003','9004','9005','9006') 
AND end_station_no LIKE '72%';  -- should be 48

-- Check BCR -> aisle mapping
SELECT station_no, aisle_station_no, wh_station_no FROM DMSTATION 
WHERE station_no LIKE '72%' ORDER BY station_no;
```

---

# Related DFDs
- [Storage Process](../Storage/Storage-Process)
- [Transfer Setting — Tempering to Ambient](../Retrieval/Transfer-Setting/Tempering-to-Ambient)
- [Transfer Setting — Ambient to Tempering](../Retrieval/Transfer-Setting/Ambient-to-Tempering)
