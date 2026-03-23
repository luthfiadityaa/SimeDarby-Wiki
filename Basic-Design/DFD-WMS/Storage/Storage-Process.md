ASRS MC Storage System — Wiki
=============================

**System:** MC (Java Middleware) between WMS (Warenavi/SAP) and AGC/ASRS  
**Date:** 2026-03-23  
**Version:** 3.0

* * *

1. System Overview
------------------

        SAP/Host
           |
           | (GR Notification / Storage Plan)
           v
      Warenavi (WMS)
           |
           | TCP/IP Socket (ID messages)
           v
          MC  <-----> Oracle DB (DNxx / DMxx tables)
           |
           | TCP/IP Socket (ID05 transport command)
           v
          AGC
           |
      STV / SRM / Conveyors
           |
      Physical ASRS Shelves
    

### Station Map

    OP Area:
      1101-1105  Robot Input Stations (palletizing input)
      1111-1115  Palletizing Stations (robot palletizes cartons)
      1210       Empty Pallet Buffer
      1220       Destacker
      1303       QC/Reject Station
      7101-7110  OP ASRS Inbound BCR Stations
      SRM 9001-9006  FGW2 Tempering (single deep)
      SRM 9007-9010  FGW1 Ambient (double deep)
    
    HP Area:
      1106       PM (Packaging Material) Inbound
      1301       FG Storage/Retrieval 1 (bi-directional)
      1302       FG Storage/Retrieval 2 (bi-directional)
      7207-7214  HP ASRS Inbound BCR Stations
      SRM 9007-9010  FGW1 Ambient (shared with OP)
      SRM 9011-9014  PM Ambient (double deep)
    

### Soft Zone to Aisle Mapping

    Soft Zone 001 (FG Tempering) → Aisles 01-06 → SRM 9001-9006 → BCR 7101-7106
    Soft Zone 002 (FG Ambient)   → Aisles 07-10 → SRM 9007-9010 → BCR 7107-7110 / 7207-7210
    Soft Zone 003 (PM Ambient)   → Aisles 11-14 → SRM 9011-9014 → BCR 7211-7214
    Soft Zone 004 (Empty Pallet) → Aisles 07-10 → SRM 9007-9010 → BCR 7107-7110 / 7207-7210
    
    Alternative zones (DMSoftZonePriority):
      002 → fallback: 004 → 003
      004 → fallback: 002 (SRM 7-10 only)
    

* * *

2. Storage Flows
----------------

### 2.1 Palletization Storage Flow

    USER                MC                    AGC/SRM
     |                   |                       |
     |--[Input batch]    |                       |
     |  DNReceivingPlan  |                       |
     |  status=0         |                       |
     |                   |                       |
     |                   |<--[ID66: 1210 empty]--|
     |                   |                       |
     |                   |--[ID05: retrieve EMP_PB from SRM]-->|
     |                   |                       |-->[1210->1220->110x]
     |                   |                       |
     |                   |<--[ID26: pallet at 110x, carry=DUMMY]--|
     |                   |                       |
     |                   | AutoStorageScheduler polls DNArrival
     |                   | No plan? -> silent wait
     |                   | Plan found:
     |                   |   INSERT DNPallet (DIRECT_PB)
     |                   |   INSERT DNStock (DIRECT_PB)
     |                   |   INSERT DNCarryInfo (DIRECT_TRAVEL)
     |                   |     dest=111x, end=710x
     |                   |   UPDATE DNReceivingPlan 0->1
     |                   |   UPDATE DNArrival SCHEDULED
     |                   |                       |
     |                   |--[ID05: 110x->111x]-->|
     |                   |                       |-->[pallet moves]
     |                   |<--[ID26: pallet at 111x, carry=CK001]--|
     |                   |                       |
     |                   | PalletizeStorageStationOperator:
     |                   |   dest was 111x, end=710x, reject_factor=00
     |                   |   LoadRemover.remove() (cleans direct carry)
     |                   |   re-register as DUMMY DNArrival
     |                   |                       |
     |                   | [Robot palletizes cartons]
     |                   |                       |
     |                   |<--[ID26: DUMMY, control_info='0']--|
     |                   |                       |
     |                   | AutoStorageScheduler:
     |                   |   insertStoragePlan from DNReceivingPlan
     |                   |   selectAisle -> AisleShelfDecider
     |                   |   WNCollectAisleSelector -> DNCollectInfo
     |                   |   INSERT DNPallet (real item)
     |                   |   INSERT DNCarryInfo (DIRECT_TRAVEL)
     |                   |     dest=710x, end=wh_station
     |                   |   INSERT DNStock (UU)
     |                   |   INSERT DNWorkInfo
     |                   |   UPDATE DNStoragePlan status=1
     |                   |   UPDATE DNArrival SCHEDULED
     |                   |                       |
     |                   |--[ID05: 111x->710x]-->|
     |                   |                       |-->[pallet moves]
     |                   |<--[ID26: pallet at 710x, carry=CK002]--|
     |                   |                       |
     |                   | AsrsInboundStationOperator:
     |                   |   wh_station=9100 = end_station=9100
     |                   |   -> FINAL BCR station
     |                   |   UPDATE DNCarryInfo:
     |                   |     carry_flag -> STORAGE
     |                   |     dest -> 9100
     |                   |                       |
     |                   | StorageSender:
     |                   |   LocationManager.searchLocation()
     |                   |   finds bin SHF001 in aisle 9001
     |                   |   UPDATE DMShelf -> RESERVED
     |                   |                       |
     |                   |--[ID05: storage to SHF001]-->|
     |                   |                       |-->[SRM stores]
     |                   |<--[ID33: storage complete]--|
     |                   |                       |
     |                   | ID33 handler:
     |                   |   UPDATE DNStock location, storage_date
     |                   |   UPDATE DNWorkInfo status=4
     |                   |   UPDATE DNStoragePlan status=4
     |                   |   UPDATE DMShelf -> OCCUPIED
     |                   |   INSERT DNHostSend (report_flag=0)
     |                   |   INSERT DNInOutResult
     |                   |   DELETE DNCarryInfo
     |                   |                       |
     |                   | ShelfMonitor (next cycle):
     |                   |   count empty shelves
     |                   |   -> ID54 lamp update if threshold
    

### 2.2 PM / FG Inbound Storage (same warehouse)

    USER                MC                    AGC/SRM
     |                   |                       |
     |--[Input plan]     |                       |
     |  DNStoragePlan    |                       |
     |  (PM/unplanned)   |                       |
     |                   |                       |
     |--[Place pallet]   |                       |
     |  at 1106/1301     |<--[ID26: DUMMY at 1106/1301]--|
     |                   |                       |
     |                   | InOutStationOperator / StorageStationOperator:
     |                   |   registArrival -> INSERT DNArrival
     |                   |   autoScheduleRequest
     |                   |                       |
     |                   | AutoStorageScheduler:
     |                   |   existsCarryInfo? No
     |                   |   insertStoragePlan
     |                   |   selectAisleAndGetBcrStation()
     |                   |     soft_zone from DMItem
     |                   |     DMSoftZonePriority fallback
     |                   |     AisleShelfDecider -> BCR 720x
     |                   |   INSERT DNPallet, DNCarryInfo
     |                   |     dest=720x, end=wh_station
     |                   |   INSERT DNStock, DNWorkInfo
     |                   |     job_type = DNStoragePlan.job_type
     |                   |   UPDATE DNStoragePlan status=1
     |                   |   UPDATE DNArrival SCHEDULED
     |                   |                       |
     |                   |--[ID05: 1106->720x]-->|
     |                   |                       |
     |                   |<--[ID26: at 720x]-----|
     |                   |                       |
     |                   | AsrsInboundStationOperator at 720x:
     |                   |   wh=9200 = end=9200 -> FINAL BCR
     |                   |   -> STORAGE, dest=9200
     |                   |                       |
     |                   | StorageSender -> bin selection -> ID05
     |                   |<--[ID33: complete]----|
     |                   | -> DNStock/WorkInfo/StoragePlan/HostSend updated
    

### 2.3 FG Inbound Cross Warehouse (1301 → FGW2)

    USER                MC                    AGC/SRM
     |                   |                       |
     |--[Unplanned]      |                       |
     |  DNStoragePlan    |                       |
     |  plan_area=FGW2   |                       |
     |                   |                       |
     |--[Place pallet]   |<--[ID26: DUMMY at 1301]--|
     |  at 1301          |                       |
     |                   | AutoStorageScheduler:
     |                   |   isCrossWarehouseRoute? YES
     |                   |   selectLeastBusyIntermediateStation()
     |                   |     DMRouteId: 1301 -> 7207-7210
     |                   |     count active carries per 720x
     |                   |     select least busy: e.g. 7208
     |                   |   INSERT DNCarryInfo
     |                   |     dest=7208, end=9100
     |                   |   INSERT DNPallet, DNStock, DNWorkInfo
     |                   |   UPDATE DNArrival SCHEDULED
     |                   |                       |
     |                   |--[ID05: 1301->7208]-->|
     |                   |                       |
     |                   |<--[ID26: at 7208]-----|
     |                   |                       |
     |                   | AsrsInboundStationOperator at 7208:
     |                   |   wh=9200 != end=9100 -> INTERMEDIATE
     |                   |   selectTargetAisleStation():
     |                   |     getReachableBcrStations(7208, 9100)
     |                   |     -> DMRouteId: 7208->710x: [7103,7104]
     |                   |     mapBcr->Aisle: [9003, 9004]
     |                   |     AisleShelfDecider(filtered aisles)
     |                   |     soft_zone + batch balance + DNCollectInfo
     |                   |     -> BCR 7103
     |                   |   UPDATE DNCarryInfo dest=7103, end=9100
     |                   |                       |
     |                   |--[ID05: 7208->7103]-->|
     |                   |                       |
     |                   |<--[ID26: at 7103]-----|
     |                   |                       |
     |                   | AsrsInboundStationOperator at 7103:
     |                   |   wh=9100 = end=9100 -> FINAL BCR
     |                   |   -> STORAGE, dest=9100
     |                   |                       |
     |                   | StorageSender -> bin in 9003
     |                   |--[ID05: storage]----->|
     |                   |<--[ID33: complete]----|
     |                   | -> all tables updated
    

* * *

3. Reject Flows
---------------

### 3.1 BCR Fail at 110x

    110x                 111x                1303
     |                    |                    |
     | AGC: ID26 bcr='?'  |                    |
     | AutoStorageScheduler checkBcr FAIL      |
     | createCarryDirect: dest=111x, end=1303  |
     |--[ID05: direct]-->|                     |
     |                   | PalletizeStorageStationOperator:
     |                   | reject_factor!=00   |
     |                   | updateDestToEndStation: dest=1303
     |                   |--[ID05: direct]---->|
     |                   |                    | LoadRemover.remove()
     |                   |                    | pallet sits at 1303
    

### 3.2 Error Completion at 111x (control_info=2)

    111x                1303
     | AGC: ID26 control_info[2]='2'
     | checkControlInfo -> PALLETIZING_ERROR_COMPLETION
     | (called BEFORE insertStoragePlan -> no orphan data)
     | createCarryDirect: dest=1303, end=1303
     |--[ID05: direct]-->|
                        | LoadRemover.remove()
    

### 3.3 Force Completion (control_info=1)

    status=1: lamp 17 ON -> user inputs last qty
             -> DNReceivingPlan status=2
             -> AGC resends -> normal storage
    
    status=2, qty>0: insertStoragePlan (last pallet)
                     createCarryStorage -> normal flow
                     UPDATE DNReceivingPlan status=4
    
    status=2, qty=0: no DNStoragePlan
                     createCarryDirect to 1303 (empty pallet)
    

* * *

4. Aisle Selection Logic
------------------------

### 4.1 Same-warehouse aisle selection

    AisleShelfDecider.decideAisle(pallet, warehouse):
    
    Step 1: WNCollectAisleSelector
      Check DNCollectInfo for aisle_collect_key:
        EXISTS -> use that aisle (same batch grouping)
        NOT EXISTS -> run balance query:
          ORDER BY ALL_BATCH_COUNT ASC,
                   ALL_STOCK_COUNT ASC,
                   AISLE_STATION_NO ASC
          (counts in-transit + stored pallets)
    
    Step 2: SoftZoneSelector
      primary soft_zone from DMItem
      fallback via DMSoftZonePriority (in priority order)
    
    Step 3: checkStorageAisle(aisle, soft_zone)
      DMShelf has empty locations in this aisle+zone?
    
    Step 4: determin()
      Write/update DNCollectInfo: aisle_collect_key -> aisle_no
      Update DMWareHouse last_used_station_no (round-robin)
    

### 4.2 Cross-warehouse intermediate selection (1301→FGW2)

    Step 1: selectLeastBusyIntermediateStation()
      Query DMRouteId: source -> HP_INTERMEDIATE_STATION_NOS
      Filter: DMStation online, not suspended, connected
      Count DNCarryInfo active per 720x station
      Select min count (tiebreak: station_no ASC)
      -> 720x intermediate (e.g. 7208)
    
    Step 2: At 720x - selectTargetAisleStation()
      getReachableBcrStations(720x, target_wh):
        DMRouteId: 720x -> 710x routes
        Filter: DMStation status online, max_instruction not full
    
      mapBcrToAisleStations():
        DMAisle.bcr_station_no -> aisle station_no
    
      AisleShelfDecider.decideAisle(filtered_aisles):
        Same as 4.1 but restricted to reachable aisles only
      -> BCR 710x (e.g. 7103)
    

### 4.3 Balance query states covered

    BC1/SC1: Already stored (DNStock + DMShelf OCCUPIED)
    BC2/SC2: In transit main->BCR (DNCarryInfo DIRECT_TRAVEL, dest=BCR)
    BC3/SC3: In transit BCR->shelf (DNCarryInfo STORAGE, dest=shelf)
    BC4/SC4: Aisle decided not yet sent (DNStoragePlan status=0, DNCollectInfo set)
    

* * *

5. Storage Completion (ID33)
----------------------------

    ID33 (storage complete) triggers:
    
    1. DNWorkInfo.status_flag -> 4 (COMPLETION)
       DNWorkInfo.result_location_no -> actual bin
       DNWorkInfo.result_qty -> stored qty
    
    2. DNStock.location_no -> actual bin
       DNStock.storage_date -> now
       DNStock.stock_qty += qty
       DNStock.plan_qty -> 0
    
    3. DMShelf.status_flag -> 1 (OCCUPIED)
    
    4. DNPallet.current_station_no -> shelf
       DNPallet.status_flag -> STORED
    
    5. INSERT DNHostSend:
       job_type = from DNWorkInfo (02 or 22)
       result_location_no = actual bin
       report_flag = 0 (pending SAP notification)
    
    6. UPDATE DNStoragePlan.status_flag -> 4 (COMPLETE)
       (via PlanControllerFactory by job_type)
    
    7. INSERT DNInOutResult (audit trail)
    
    8. DELETE DNCarryInfo
    
    9. ShelfMonitor (next cycle):
       count empty DMShelf -> ID54 lamp if threshold changed
    

* * *

6. Location Full Lamp (ShelfMonitor)
------------------------------------

    ShelfMonitor runs every SHELF_MONITOR_SLEEP_SEC:
    
    For each DMLocationFullLamp record:
      count empty DMShelf:
        WHERE wh_station_no = configured wh
        AND status = EMPTY (0)
        AND aisle.status != DISCONNECTED
        AND prohibition = OK, access_ng = OK
        minus reserved_qty
    
      Compare against DNSystemKVs thresholds:
        warningQty  = free_shelf_warning_num_agc5     (= 50)
        lightOffQty = free_shelf_warning_release_num_agc5 (= 60)
    
      emptyCount = 0:         LAMP_FULL ON, WARNING OFF
      0 < count < 50:         WARNING ON, LAMP_FULL OFF
      50 <= count <= 60:      hysteresis (stays WARNING if was FULL)
      count > 60:             both OFF
    
      If status changed from last check:
        Send ID54 to DMLocationFullLamp.station_no
        UPDATE DMLocationFullLamp.status_flag
    
    NOTE: MC code does NOT send LAMP_FULL directly.
          ShelfMonitor is the ONLY source for this lamp.
    

* * *

7. Key Tables Reference
-----------------------

    DNReceivingPlan  - Palletization batch plan (from Warenavi screen)
                       status: 0=Not started, 1=Working, 2=Last pallet wait, 4=Complete
    
    DNStoragePlan    - Storage plan for each pallet (from system or user)
                       status: 0=Unstart, 1=Working, 4=Complete
                       job_type: 02=Storage, 22=Unplanned
    
    DNArrival        - Pallet arrival record per station
                       sch_flag: 0=Not scheduled, 1=Scheduled
                       sch_carry_key: carry key after scheduling
    
    DNCarryInfo      - Transport command record
                       carry_flag: 1=Storage, 3=Direct Travel
                       cmd_status: 1=Start, 2=Arrival, 3=Instruction, 4=Wait response
    
    DNPallet         - Pallet tracking record
    DNStock          - Inventory record (location, qty, status)
    DNWorkInfo       - Work instruction linked to carry + stock
    DNCollectInfo    - Aisle grouping for batch (aisle_collect_key -> aisle_no)
    DNHostSend       - Result notification to SAP/Host (report_flag: 0=pending, 1=sent)
    DNInOutResult    - Audit trail of all storage/retrieval operations
    DMShelf          - Physical shelf location (status: 0=Empty, 1=Occupied, 2=Reserved)
    DMLocationFullLamp - Tower light configuration per warehouse
    DMWareHouse      - Warehouse settings (aisle_decision_pattern, zone_manage_type)
    DNSystemKVs      - System parameters (lamp thresholds etc.)