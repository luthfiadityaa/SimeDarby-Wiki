# Restorage Plan (再入庫予定)

## Overview

Restorage Plan controls the re-storage of pallets after partial picking. When a pallet is retrieved for picking and only a portion of the quantity is consumed, the remaining stock needs to be stored back into the ASRS with its quality attributes preserved.

## Table: DNReStoringPlan

| Column | Type | Description |
|--------|------|-------------|
| PLAN_UKEY | varchar2(10) PK | "27" + sequence |
| STATUS_FLAG | varchar2(1) | 0:Not Started, 1:Working, 4:Completed, 9:Deleted |
| STATION_NO | varchar2(16) | Restorage station number |
| WORK_NO | varchar2(16) | Original retrieval work number |
| PLAN_DAY | varchar2(8) | Plan date |
| PLAN_AREA_NO | varchar2(4) | Target warehouse (9100/9200) |
| PLAN_LOCATION_NO | varchar2(11) | Target location |
| ITEM_CODE | varchar2(40) | Material code |
| PLAN_LOT_NO | varchar2(60) | Lot number |
| STORAGE_TYPE | varchar2(3) | 1:Restorage, 2:New |
| NOTE | varchar2(180) | Remarks |
| STORAGE_DAY | varchar2(8) | Original storage date |
| STORAGE_DATE | date | Original storage date/time |
| RETRIEVAL_DAY | varchar2(8) | Last retrieval date |
| REMOVE_DATE | date | Removal timestamp |
| PLAN_QTY | number(7) | Remaining quantity to restorage |
| RESULT_QTY | number(7) | Actual restored quantity |
| SHORTAGE_QTY | number(7) | Plan - Result |
| REPORT_FLAG | varchar2(1) | 0:Not Reported, 1:Reported to SAP |
| BCR_DATA | varchar2(30) | Pallet barcode |
| AISLE_DISPERSE_KEY | varchar2(100) | Aisle distribution key |
| AISLE_COLLECT_KEY | varchar2(100) | Aisle group key |

### Missing Columns (Need ALTER TABLE)

These stock quality attributes exist in DNStock but are **NOT YET** in DNReStoringPlan:

| Column | Type | Purpose |
|--------|------|---------|
| STOCK_STATUS | varchar2(2) | UU:Unrestricted, BS:Block, QI:Quality Inspection |
| TEMPERING_PERIOD | number(7) | Tempering duration in hours |
| TEMPERING_FLAG | varchar2(1) | 0:Not Reached, 1:Reached |
| QC_FLAG | varchar2(1) | 0:Not Done, 1:Done |
| QC_DURATION | number(7) | QC duration |
| EXPIRY_DATE | timestamp(3) | Product expiry date |
| STORAGE_LOCATION | varchar2(10) | FGW1/FGW2/VT01/QC01/SD01/SG01/ZPCK |

**These columns MUST be added** so that when restorage storage happens, the stock is recreated with the correct quality/tempering/QC/expiry data.

## Job Types

| Job Type | Code | Japanese | Description |
|----------|------|----------|-------------|
| STORAGE | 02 | 入庫 | Planned storage (from DNReceivingPlan) |
| RETRIEVAL | 03 | 出庫 | Planned retrieval (picking uses this) |
| NOPLAN_STORAGE | 22 | 予定外入庫 | Unplanned storage |
| NOPLAN_RETRIEVAL | 23 | 予定外出庫 | Unplanned retrieval |
| RESTORING | 27 | 再入庫 | Restorage (this feature) |

## Trigger Mechanism: DMStation.restoring_operation

The **critical setting** that controls whether DNReStoringPlan is created is:

**Table:** `DMStation`
**Column:** `restoring_operation`

| Value | Constant | Meaning |
|-------|----------|---------|
| 0 | NOT_CREATE | Do NOT create restorage plan data |
| 1 | CREATE | Create restorage plan data |

### Current Values (ALL disabled)

All retrieval stations currently have `restoring_operation = '0'`:

| Station | Name | restoring_operation |
|---------|------|---------------------|
| 1205 | FG Retrieval 1 | 0 (disabled) |
| 1206 | FG Retrieval 2 | 0 (disabled) |
| 1207 | FG Retrieval 3 | 0 (disabled) |
| 1208 | FG Retrieval 4 | 0 (disabled) |
| 1209 | FG Retrieval 5 | 0 (disabled) |
| 1210 | EP Retrieval | 0 (disabled) |
| 1220 | EP Retrieval End | 0 (disabled) |
| 1201 | PM Retrieval 1 | 0 (disabled) |
| 1202 | PM Retrieval 2 | 0 (disabled) |
| 1203 | PM Retrieval 3 | 0 (disabled) |
| 1204 | PM Retrieval 4 | 0 (disabled) |

### To Enable Restorage Plan Creation

```sql
-- Enable restorage plan for FG retrieval stations (partial picking)
UPDATE DMStation SET restoring_operation = '1'
WHERE station_no IN ('1205','1206','1207','1208','1209');

-- Optional: Enable for PM retrieval stations
UPDATE DMStation SET restoring_operation = '1'
WHERE station_no IN ('1201','1202','1203','1204');
```

## Two Paths at ID26 (load_present=0)

There are **two completely separate code paths** depending on retrieval_detail:

### Path A: Unit Retrieval (retrieval_detail = UNIT)

```
ID26 load_present=0 + UNIT
  ↓
LoadRemover (RetrievalRemover)
  ↓
removeLoad()
  ├─ doStockDeletion()        → DELETE DNStock
  ├─ releaseSourceShelf()     → UPDATE DMShelf status
  └─ needsRestorage(ci, workType)  ← CHECKS restoring_operation
      ├─ "0" → NO restorage plan created
      └─ "1" → createRestorageData() → INSERT DNReStoringPlan
```

**Code:** `RetrievalRemover.java:340-358`
**Gate:** `SettingRestorageOperator.needsRestorage()` line 124
**Setting:** `DMStation.restoring_operation` on dest station

### Path B: Picking Return (retrieval_detail = PICKING/INVENTORY_CHECK/ADD_STORING)

```
ID26 load_present=0 + PICKING
  ↓
needsReturnStorage() = TRUE   (Id26SubThread:400-414)
  ↓
ReturnStorageManager.updateForReturnStorageArrival()
  ├─ completeRetrievalWorkAndUpdateStock()  → UPDATE DNStock (subtract picked qty)
  ├─ If remaining qty > 0: stock survives
  ├─ If remaining qty == 0: stock deleted
  ├─ Update carry: RETRIEVAL → STORAGE
  └─ Route pallet back to warehouse
  
  *** NO DNReStoringPlan is created ***
  *** restoring_operation is NOT checked ***
```

**Code:** `ReturnStorageManager.java:147-191`
**Note:** This path does NOT create DNReStoringPlan regardless of settings

### Gap: Path B Does Not Create DNReStoringPlan

For the **partial picking scenario** (Path B), the pallet returns to storage via normal carry conversion. But:
- No DNReStoringPlan record is created
- When the same pallet comes back for storage at an inbound station, AutoStorageScheduler only checks DNStoragePlan — it doesn't check DNReStoringPlan
- Quality attributes (stock_status, tempering, QC, expiry) are preserved in the surviving DNStock record since the carry is just converted from RETRIEVAL to STORAGE

**For the partial picking use case where pallet goes to inbound station instead of return storage:**
- DNReStoringPlan creation needs to be added to ReturnStorageManager (or a new path)
- AutoStorageScheduler needs to check DNReStoringPlan as fallback when DNStoragePlan not found

## Implementation Changes Required

### 1. Schema Change

```sql
ALTER TABLE DNReStoringPlan ADD (
    stock_status        varchar2(2),
    tempering_period    number(7),
    tempering_flag      varchar2(1),
    qc_flag             varchar2(1),
    qc_duration         number(7),
    expiry_date         timestamp(3),
    storage_location    varchar2(10)
);
```

### 2. Code Changes

| File | Change |
|------|--------|
| `SettingRestorageOperator.createRestoringPlan()` | Map stock_status, tempering_*, qc_*, expiry_date, storage_location from Stock |
| `ReturnStorageManager.updateForReturnStorageArrival()` | After work completion, if remaining qty > 0, create DNReStoringPlan with quality data |
| `AutoStorageScheduler.createCarryStorageInbound()` | Fallback: check DNReStoringPlan when DNStoragePlan not found by bcr_data |
| `ReStoringPlan` entity (wms-handler.jar) | Add getter/setter for new columns |

### 3. DMStation Setting

```sql
UPDATE DMStation SET restoring_operation = '1'
WHERE station_no IN ('1205','1206','1207','1208','1209');
```

### 4. ReStoringPlanController (No Change Needed)

Already handles job_type "27" completion at ID33:
- `startPlan()`: status 0 → 1
- `updatePlan()`: status 1 → 4, records result_qty/shortage_qty

## Data Flow Diagram

```
╔══════════════════════════════════════════════════════════════╗
║  RETRIEVAL (Planned Picking, job_type=03)                   ║
║                                                              ║
║  DNStock(qty=100) → Pick 30 → remaining 70                 ║
╚══════════════════════════════════════════════════════════════╝
                        │
                        ▼
╔══════════════════════════════════════════════════════════════╗
║  ID26 ARRIVAL (load_present=0)                              ║
║                                                              ║
║  ReturnStorageManager:                                       ║
║    1. completeRetrievalWork (result_qty=30)                 ║
║    2. UPDATE DNStock (stock_qty = 100-30 = 70)             ║
║    3. [NEW] CREATE DNReStoringPlan                          ║
║       - plan_qty = 70 (remaining)                           ║
║       - bcr_data = pallet barcode                           ║
║       - stock_status, tempering, QC, expiry from DNStock    ║
║       - status = 0 (UNSTART)                                ║
║    4. Convert carry: RETRIEVAL → STORAGE                    ║
║    5. Route pallet back to warehouse                        ║
╚══════════════════════════════════════════════════════════════╝
                        │
                        ▼
╔══════════════════════════════════════════════════════════════╗
║  LATER: Same pallet at inbound station (1301/1302/1303)     ║
║                                                              ║
║  AutoStorageScheduler:                                       ║
║    1. Check DNStoragePlan by bcr_data → NOT FOUND           ║
║    2. [NEW] Check DNReStoringPlan by bcr_data → FOUND!      ║
║    3. Use restorage plan data for storage:                   ║
║       - job_type = "27" (RESTORING)                         ║
║       - stock_status, tempering, QC, expiry from plan       ║
║    4. UPDATE DNReStoringPlan status 0 → 1                   ║
╚══════════════════════════════════════════════════════════════╝
                        │
                        ▼
╔══════════════════════════════════════════════════════════════╗
║  ID33 STORAGE COMPLETION                                     ║
║                                                              ║
║  PlanControllerFactory(job_type="27")                       ║
║    → ReStoringPlanController.updatePlan()                    ║
║    → status 1 → 4, result_qty recorded                     ║
║                                                              ║
║  DNHostSend: job_type="27", reported to SAP                 ║
╚══════════════════════════════════════════════════════════════╝
```

## Key Classes Reference

| Class | File Path | Purpose |
|-------|-----------|---------|
| SettingRestorageOperator | `asrs/operator/restorage/SettingRestorageOperator.java` | Creates DNReStoringPlan records |
| RestorageOperator | `asrs/operator/restorage/RestorageOperator.java` | Interface for restorage operations |
| RetrievalRemover | `asrs/function/remove/RetrievalRemover.java` | Handles stock deletion + conditional restorage |
| ReturnStorageManager | `asrs/location/ReturnStorageManager.java` | Handles picking return storage |
| ReStoringPlanController | `asrs/controller/plan/ReStoringPlanController.java` | Updates plan status on completion |
| ReStorageSettingSCH | `wms/web/display/storage/restoragesetting/ReStorageSettingSCH.java` | Manual restorage setting screen |
| AutoStorageScheduler | `asrs/transmission/AutoStorageScheduler.java` | Processes storage at inbound stations |
| Id26SubThread | `asrs/communication/control/Id26SubThread.java` | ID26 arrival processing |
| FreeRetrievalStationOperator | `asrs/location/FreeRetrievalStationOperator.java` | Free retrieval station logic |
