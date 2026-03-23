Palletization Storage — Programmer Documentation
================================================

**Scope:** Palletization Storage Flow (OP Area) **Version:** 1.0 **Date:** 2026-03-23

* * *

Table of Contents
-----------------

1.  [System Thread Architecture](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#1-system-thread-architecture)
2.  [Palletization Storage Flow Overview](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#2-palletization-storage-flow-overview)
3.  [Phase 1 — User Setup](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#3-phase-1--user-setup)
4.  [Phase 2 — Empty Pallet Supply](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#4-phase-2--empty-pallet-supply)
5.  [Phase 3 — Arrival at 110x (BCR Scan)](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#5-phase-3--arrival-at-110x-bcr-scan)
6.  [Phase 4 — Palletizing at 111x](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#6-phase-4--palletizing-at-111x)
7.  [Phase 5 — Storage Destination Selection](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#7-phase-5--storage-destination-selection)
8.  [Phase 6 — Control Info Processing](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#8-phase-6--control-info-processing)
9.  [Phase 7 — Pallet Forced Removal (ID04)](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#9-phase-7--pallet-forced-removal-id04)
10.  [Data Flow Diagram](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#10-data-flow-diagram)
11.  [Table Operations Summary](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#11-table-operations-summary)
12.  [Error Handling Reference](https://claude.ai/chat/51ab9b49-c6ff-4c10-8edf-3c3f5e3cf4e1#12-error-handling-reference)

* * *

1. System Thread Architecture
-----------------------------

    TransmissionExecutor (main launcher — runs once on boot)
    │
    ├── AutoStorageScheduler[] (one thread per station group)
    │     Polls DNArrival continuously
    │     Handles: 110x, 111x, 1106, 1301, 1302, 1303
    │
    ├── StorageSender (or DoubleDeepStorageSender if double deep)
    │     Triggered via RMI by carryRequest()
    │     Reads DNCarryInfo, sends ID05 storage command to AGC
    │
    ├── RetrievalSender
    │     Handles retrieval commands
    │
    ├── TemperingChecker
    │     Polls DNStock for expired tempering periods
    │     Updates tempering_flag → 1
    │
    └── TimeKeeper, ShelfMonitor, RequestWatcher, etc.
    

**ID26 Processing Chain:**

    AGC sends ID26
        │
        ▼
    Id26Process
        │ parse station_no → get groupNo
        ▼
    Id26SubThread (per station group)
        │ create CarryInfo from mckey
        ▼
    Station Operator (by station_operator_class in DMStation)
        │
        ├── StorageStationOperator          → used at 1101-1107
        ├── PalletizeStorageStationOperator → used at 1111-1115
        └── InOutStationOperator            → used at 1301-1303
    

* * *

2. Palletization Storage Flow Overview
--------------------------------------

    ┌─────────────────────────────────────────────────────────────────┐
    │ PHASE 1: User Setup                                             │
    │ User inputs batch info at Palletize Start Screen               │
    │ → INSERT DNReceivingPlan (status=0)                            │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 2: Empty Pallet Supply                                    │
    │ AGC sends ID66 (1210 empty) → MC selects from SRM 9007-9010    │
    │ FIFO by DNStock.storage_date → sends to 1210 → 1220 → 110x    │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 3: Arrival at 110x                                        │
    │ AGC sends ID26 → BCR scan result                               │
    │ BCR OK → INSERT DNPallet, DNStock(DIRECT_PB)                   │
    │         UPDATE DNReceivingPlan 0→1 (first pallet only)         │
    │         store in DNArrival → AutoStorageScheduler sends ID05   │
    │         direct transfer → 111x                                 │
    │ BCR Fail → INSERT DNStock(IRREGULAR_PB) → route to 1303        │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 4: Arrival at 111x — Robot Palletizes                     │
    │ PalletizeStorageStationOperator receives ID26                   │
    │ Real carry arrives → LoadRemover cleans up direct carry        │
    │ Re-register as dummy → AutoStorageScheduler picks up           │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 5: AutoStorageScheduler at 111x                          │
    │ insertStoragePlan() → create DNStoragePlan from DNReceivingPlan│
    │ createCarryStorage() → INSERT DNPallet, DNCarryInfo,           │
    │                         DNStock, DNWorkInfo                    │
    │ carryRequest() → triggers StorageSender                        │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 6: StorageSender                                          │
    │ Reads DNCarryInfo, selects bin location                        │
    │ Sends ID05 storage command → 710x → SRM bin                   │
    └──────────────────────────┬──────────────────────────────────────┘
                               │
    ┌──────────────────────────▼──────────────────────────────────────┐
    │ PHASE 7: Storage Complete                                       │
    │ AGC sends ID33 completion → MC updates DNStock, DNCarryInfo    │
    │ Updates DNWorkInfo status → Complete                           │
    └─────────────────────────────────────────────────────────────────┘
    

* * *

3. Phase 1 — User Setup
-----------------------

### Actor: User at Warenavi (WMS) Palletize Start Screen

    User inputs:
      ┌─────────────────────────────────────────┐
      │ Station No      : 1111 - 1115           │
      │ Material Code   : (item_code)           │
      │ Batch/Lot No    : (plan_lot_no)         │
      │ Expiry Days     : (batch_expiry_days)   │
      │ Planned Qty     : (plan_qty)            │
      │ Storage Date    : (batch_storage_datetime)│
      │ Qty Ctrn/Pl     : (batch_qty_ctrn_pl)  │
      │ Tempering Period: (batch_tempering_period in hours) │
      │ Storage Location: FGW1 or FGW2         │
      └─────────────────────────────────────────┘
             │
             ▼
    INSERT DNReceivingPlan
      status_flag         = 0 (Not started)
      plan_ukey           = "01" + sequence
      batch_station_no    = selected station (1111-1115)
      plan_area_no        = FGW1 or FGW2
      item_code           = input value
      plan_lot_no         = input value
      batch_qty_ctrn_pl   = input value
      batch_tempering_period = input value
      batch_expiry_days   = input value
      batch_storage_datetime = input value
      batch_last_pallet_qty = 0 (default)
      storing_pair_key    = item_code + plan_lot_no
    

* * *

4. Phase 2 — Empty Pallet Supply
--------------------------------

### Trigger: AGC detects 1210 (Empty Pallet Buffer) is empty → sends ID66

    AGC sends ID66 to MC
      station_no = 1210 (empty)
             │
             ▼
    MC queries DNStock:
      WHERE item_code = 'EMP_PB'  (empty pallet)
      AND area in (9007, 9008, 9009, 9010)
      ORDER BY storage_date ASC  ← FIFO
             │
        ┌────┴────┐
      No stock   Stock found
        │              │
        ▼              ▼
      Send ID54   Send ID05 → AGC
      lamp 15     (retrieve from SRM → send to 1210)
      (Out of
      Empty Pallet)
             │
             ▼
      Station 1210 receives pallet stack
             │
             ▼
      1210 supplies single empty pallet
      → 1220 (Destacker)
      → 1101-1105 (controlled by AGC)
    

* * *

5. Phase 3 — Arrival at 110x (BCR Scan)
---------------------------------------

### Handler: StorageStationOperator → AutoStorageScheduler

    Pallet arrives at 110x (1101-1105)
             │
             ▼
    AGC scans BCR + detects pallet presence
    AGC sends ID26 to MC:
      station_no  = 110x
      carry_key   = 99999999 (dummy)
      bcr_data    = scan result
             │
             ▼
    Id26Process → Id26SubThread → StorageStationOperator.arrival()
             │
             ▼
    Duplicate check:
      carry_key = 99999999?
      → bcr_data already in DNArrival? → DISCARD
      → not exist → continue
             │
             ▼
    BCR check: bcr_data = '?' or '@'?
             │
        ┌────┴────┐
      FAIL       OK
        │          │
        ▼          ▼
    INSERT      INSERT DNPallet:
    DNStock       pallet_id         = new sequence
    (IRREGULAR    current_station_no = 110x
    _PB)          wh_station_no     = 9100
                  bcr_data          = from ID26
                  status_flag       = STORAGE_PLAN
                  empty_flag        = NORMAL
        │
        ▼         INSERT DNStock:
    Send ID05       item_code   = 'DIRECT_PB'
    transport       plan_qty    = 1
    classification  pallet_id   = new pallet_id
    = 3 (direct)    area_no     = area of wh_station
    dest = 1303     storage_location = FGW1 or FGW2
    via 111x
    controlinfo     UPDATE DNReceivingPlan:
    = 010           WHERE batch_station_no = next_station_no
                    AND status_flag = '0' (UNSTART only)
    reject_factor   → status_flag = '1' (NOWWORKING)
    = 01
                    INSERT DNArrival:
    DELETE          station_no   = 110x
    DNArrival       carry_key    = 99999999
                    bcr_data     = from ID26
                    sch_flag     = 0 (not scheduled)
                             │
                             ▼
                  AutoStorageScheduler polls DNArrival
                             │
                  Check DNReceivingPlan:
                  WHERE batch_station_no = next_station_no (111x)
                  AND status_flag IN ('0','1','2')
                             │
                        ┌────┴────┐
                      None      Found
                        │          │
                      Stay in    Send ID05:
                      DNArrival   transport_class = 3 (direct)
                      (silent     source = 110x
                      wait)       dest   = 111x (next_station_no)
                                  bc_data = from ID26
                                  UPDATE DNArrival sch_flag → SCHEDULED
                                  commit → carryRequest()
    

* * *

6. Phase 4 — Palletizing at 111x
--------------------------------

### Handler: PalletizeStorageStationOperator

    Pallet arrives at 111x physically
             │
             ▼
    AGC sends ID26:
      station_no = 111x
      carry_key  = real key (from direct transfer)
      bcr_data   = pallet barcode
      carry_flag = DIRECT_TRAVEL
             │
             ▼
    PalletizeStorageStationOperator.arrival()
             │
    carry_key = DUMMY (99999999)?
             │
        ┌────┴──────────────────────────────┐
       YES (dummy)               NO (real carry)
        │                                   │
    Register DNArrival             carry_flag = DIRECT_TRAVEL?
    autoScheduleRequest()          AND end_station_no = this 111x?
                                             │
                                        ┌────┴────┐
                                       YES        NO
                                        │          │
                                 LoadRemover    Re-register
                                 .remove(ci)   as dummy arrival
                                 (cleans up    → autoScheduleRequest
                                 direct carry  (AutoStorageScheduler
                                 DNCarryInfo)  handles reject)
                                        │
                                 set carry_key = DUMMY
                                 registArrival(ci, plt)
                                 carryRequest()
                                        │
                                        ▼
                                 AutoStorageScheduler
                                 picks up dummy arrival
                                 at 111x
    

### Robot Palletizing States

    Pallet at 111x
    Robot palletizes cartons
             │
        ┌────┴──────────────────────────────┐
    Full pallet                       Last pallet
    (qty = batch_qty_ctrn_pl)         (partial qty)
        │                                   │
    Auto released by robot            User: Batch End at
        │                             Palletization Screen
        │                             input last_pallet_qty (≥ 0)
        │                                   │
        │                             UPDATE DNReceivingPlan:
        │                               status 1 → 2
        │                               batch_last_pallet_qty = input
        │                                   │
        │                             User: Force completion
        │                             at robot side
        │
    AGC sends ID26 at 111x:
      station_no    = 111x
      carry_key     = 99999999 (dummy, re-registered)
      bcr_data      = pallet barcode
      control_info  = [x][x][N]  ← position 3:
                                   0 = normal completion
                                   1 = force completion
                                   2 = error completion
    

* * *

7. Phase 5 — Storage Destination Selection
------------------------------------------

### Handler: AutoStorageScheduler (PALLETIZATION_STORAGE_STATION_NOS path)

    AutoStorageScheduler picks up DNArrival at 111x
             │
             ▼
    checkBcr(arrival)
      bcr_data = '?' or '@'? → rejectProcess() → 1303
             │
             ▼
    checkControlInfo(arrival)
      controlInfo null or length < 3? → MALFUNCTION_CONTROL_DATA
      position 3 = '2'? → PALLETIZING_ERROR_COMPLETION
      position 3 = '1' (FORCE)?
        → check DNReceivingPlan status:
          status = '2' (WAITING_LAST_PALLET) → OK, continue
          status = '1' (NOWWORKING) → WAITING_LAST_PALLET (send lamp 17)
      position 3 = '0' (NORMAL)? → OK, continue
             │
             ▼
    insertStoragePlan(arrival):
      Find DNReceivingPlan WHERE:
        batch_station_no = 111x
        status IN ('0','1','2')
      Lock record (NOWAIT)
             │
      status = '2' AND batch_last_pallet_qty <= 0?
        → UPDATE DNReceivingPlan status → '4' (COMPLETE)
        → return (no StoragePlan created — zero qty last pallet)
        → send empty pallet to 1303 as DIRECT_PB
             │
      else → INSERT DNStoragePlan:
        plan_ukey      = "02" + sequence
        status_flag    = '0' (UNSTART)
        plan_area_no   = from DNReceivingPlan
        plan_qty       = batch_qty_ctrn_pl (normal)
                       OR batch_last_pallet_qty (force)
        bcr_data       = from DNArrival
        item_code      = from DNReceivingPlan
        plan_lot_no    = from DNReceivingPlan
        storing_pair_key = item_code + plan_lot_no
        batch_tempering_period = from DNReceivingPlan
        batch_expiry_days      = from DNReceivingPlan
             │
             ▼
    createCarryStorage(arrival):
      Find DNStoragePlan WHERE:
        status_flag = '0' (UNSTART)
        bcr_data = from arrival (trimmed match)
      Lock record (NOWAIT) → ROLLBACK if locked
             │
             ▼
    insertPalletStorage(arrival):
      NEW INSERT DNPallet:
        pallet_id          = new sequence
        current_station_no = 111x
        wh_station_no      = 9100
        status_flag        = STORAGE_PLAN
        empty_flag         = NORMAL
        bcr_data           = from arrival (trimmed)
        soft_zone_id       = from DMItem (first item's zone)
             │
             ▼
    insertCarryInfoStorage(pallet):
      INSERT DNCarryInfo:
        carry_key      = new sequence
        pallet_id      = new pallet_id
        work_type      = '02' (Storage)
        cmd_status     = '1' (Started)
        carry_flag     = '1' (Storage)
        source_station = 111x
        dest_station   = wh_station_no (9100)
        end_station    = wh_station_no (9100)
        reject_factor  = '00' (None)
             │
             ▼
    For each DNStoragePlan record:
      insertStockStorage(pallet, storagePlan):
        INSERT DNStock:
          stock_id         = new sequence
          area_no          = area of wh_station
          item_code        = from DNStoragePlan
          lot_no           = from DNStoragePlan
          storage_type     = '2' (New)
          plan_qty         = from DNStoragePlan
          pallet_id        = new pallet_id
          storage_location = from DNStoragePlan (FGW1/FGW2)
          stock_status     = 'UU' (Unrestricted)
          tempering_period = from DNStoragePlan
          expiry_date      = storage_datetime + expiry_days
          storing_pair_key = item_code + lot_no
             │
      insertWorkInfoStorage(carryInfo, stock, storagePlan):
        INSERT DNWorkInfo:
          job_no           = new sequence
          plan_ukey        = from DNStoragePlan
          stock_id         = new stock_id
          system_conn_key  = carry_key
          job_type         = '02' (Storage)
          status_flag      = '1' (Working)
          hardware_type    = '3' (ASRS)
          storing_pair_key = item_code + lot_no
             │
      UPDATE DNStoragePlan:
        status_flag  → '1' (NOWWORKING)
        process_qty  → plan_qty
             │
             ▼
    UPDATE DNArrival:
      sch_flag     → '1' (SCHEDULED)
      sch_carry_key → carry_key
             │
             ▼
    commit → carryRequest() → triggers StorageSender via RMI
    

* * *

8. Phase 6 — Control Info Processing Detail
-------------------------------------------

### Control Info = 0 (Normal Completion — Full Pallet)

    Full pallet auto released by robot
    AGC sends ID26 with control_info[2] = '0'
             │
             ▼
    checkControlInfo → OK
    insertStoragePlan → INSERT DNStoragePlan (qty = batch_qty_ctrn_pl)
    createCarryStorage → INSERT DNPallet, DNCarryInfo, DNStock, DNWorkInfo
             │
             ▼
    UPDATE DNArrival sch_flag → SCHEDULED
    commit → carryRequest()
             │
             ▼
    StorageSender selects bin → sends ID05 storage → 710x → SRM
    

### Control Info = 1 (Force Completion — Last Pallet)

    User force releases last pallet from robot
    AGC sends ID26 with control_info[2] = '1'
             │
             ▼
    checkControlInfo:
      DNReceivingPlan status = '2'? → OK
      DNReceivingPlan status = '1'? → WAITING_LAST_PALLET
             │
        ┌────┴──────────────────────────┐
      status = '1'            status = '2'
      (not yet batch end)          │
        │                    batch_last_pallet_qty = 0?
        ▼                         │
      Send ID54:            ┌─────┴──────┐
      lamp 17 ON           qty=0        qty>0
      (Batch End            │             │
      Incomplete)     UPDATE          INSERT DNStoragePlan
      Hold at 111x    DNReceivingPlan   (qty = last_pallet_qty)
                      status → '4'    INSERT DNPallet
                      Send empty      INSERT DNCarryInfo
                      pallet to 1303  INSERT DNStock
                      as DIRECT_PB    INSERT DNWorkInfo
                                      UPDATE DNReceivingPlan
                                        status → '4' (Complete)
                                      carryRequest()
             │
             ▼  (after user does batch end + error reset)
      AGC resends ID26 (control_info[2] = '1')
      → loop back to checkControlInfo
    

### Control Info = 2 (Error Completion)

    Robot reports error completion
    AGC sends ID26 with control_info[2] = '2'
             │
             ▼
    checkControlInfo → PALLETIZING_ERROR_COMPLETION
    rejectProcess():
      station has reject_station_no (1303)?
      → createCarryDirect(arrival) → route to 1303
      reject_factor = '06' (No Storage Data)
    

### Lamp 17 Recovery Flow

    ID54 lamp 17 ON sent by MC
             │
             ▼
    Tower light ON at 111x
    User sees alert on Warenavi screen
             │
             ▼
    User inputs actual last pallet qty
    at Palletization Screen → Batch End
             │
             ▼
    UPDATE DNReceivingPlan:
      status 1 → 2
      batch_last_pallet_qty = actual qty
             │
             ▼
    User presses Error Reset
    at operation box near station
             │
             ▼
    AGC turns lamp OFF
    AGC resends ID26 (control_info[2] = '1')
             │
             ▼
    MC processes → normal force completion flow
    

### Location Full Recovery Flow

    All aisles full / disconnected
             │
             ▼
    Send ID54 lamp 01 ON (Location Full)
    Pallet holds at 111x
    DNArrival record stays (not deleted)
             │
             ▼
    When slot becomes available:
    MC detects → Send ID54 lamp 01 OFF
    DNArrival polling resumes
    → aisle selection → carryRequest()
    

* * *

9. Phase 7 — Pallet Forced Removal (ID04)
-----------------------------------------

### Trigger: User removes pallet physically → AGC detects no-load → user does delete tracking in MOS

    AGC sends ID04 to MC:
      mc_key          = original carry key
      source_station  = station where removed
      dest_station    = original destination
      location_no     = original location
    

### Scenario A — Removed at 110x

    ID04 received (source = 110x)
             │
             ▼
    Lookup by mc_key + source_station
    DELETE DNStock WHERE item_code = 'DIRECT_PB' AND pallet matches
    DELETE DNPallet
    (AGC handles tower light OFF automatically)
    

### Scenario B — Removed at 111x BEFORE ID26

    ID04 received (source = 111x, no real stock yet)
             │
             ▼
    DELETE DNStock WHERE item_code = 'DIRECT_PB' AND pallet matches
    DELETE DNPallet
    

### Scenario C — Removed at 111x AFTER ID26 (real stock created)

    ID04 received (source = 111x, real stock exists)
             │
             ▼
    DELETE DNStock (real item)
    DELETE DNPallet
    DELETE DNCarryInfo
    DELETE DNStoragePlan
    DELETE DNWorkInfo
    (Aisle reservation released)
    (AGC handles tower light OFF)
    

**Note:** DNReceivingPlan status is NOT rolled back in any scenario. The batch continues — next pallet from 110x will arrive and continue. If the removed pallet needs to be re-stored, user uses station 1303 unplanned storage.

* * *

10. Data Flow Diagram
---------------------

    ┌─────────────────────────────────────────────────────────────────────────────┐
    │                         PALLETIZATION STORAGE DATA FLOW                     │
    └─────────────────────────────────────────────────────────────────────────────┘
    
    USER INPUT
        │
        │ INSERT
        ▼
    ┌───────────────┐
    │DNReceivingPlan│ status: 0→1→2→4
    └───────┬───────┘
            │ read (find plan by batch_station_no)
            │
    AGC ID26 (110x)
        │
        │ INSERT
        ▼
    ┌───────────┐     INSERT    ┌──────────┐
    │ DNArrival │──────────────▶│ DNPallet │ (DIRECT_PB placeholder)
    └─────┬─────┘               └──────────┘
          │ poll                     │
          │                    INSERT▼
          │               ┌──────────────┐
          │               │   DNStock    │ item_code=DIRECT_PB
          │               └──────────────┘
          │
          │ AutoStorageScheduler (110x path)
          │ sends ID05 direct transfer → 111x
          │
    AGC ID26 (111x — real carry)
          │
          │ PalletizeStorageStationOperator
          │ LoadRemover cleans direct carry
          │ re-register as dummy
          │
          │ AutoStorageScheduler (111x path)
          │
          │ INSERT
          ▼
    ┌─────────────────┐
    │  DNStoragePlan  │ status: 0→1
    └────────┬────────┘
             │
             │ INSERT (from DNStoragePlan data)
             ▼
    ┌───────────┐   INSERT   ┌──────────┐   INSERT   ┌──────────────┐
    │ DNPallet  │◄───────────│          │────────────▶│   DNStock    │
    │ (new)     │            │          │             │ (real item)  │
    └───────────┘            │  create  │             └──────────────┘
                             │  Carry   │                    │
    ┌───────────┐   INSERT   │  Storage │   INSERT           │
    │DNCarryInfo│◄───────────│          │──────────────▶┌────────────┐
    │           │            │          │               │ DNWorkInfo │
    └─────┬─────┘            └──────────┘               └────────────┘
          │
          │ UPDATE sch_flag→SCHEDULED
          ▼
    ┌───────────┐
    │ DNArrival │
    └───────────┘
    
          │ carryRequest() via RMI
          ▼
    StorageSender
          │ reads DNCarryInfo
          │ selects bin (via StorageRouteController)
          │ sends ID05 storage → 710x → SRM bin
          ▼
    AGC → SRM stores pallet
          │
          │ AGC sends ID33 (completion)
          ▼
    MC updates:
      DNCarryInfo  cmd_status → ARRIVAL/COMPLETE
      DNStock      location_no, storage_date (actual)
      DNWorkInfo   status → COMPLETE, result_location_no
    

* * *

11. Table Operations Summary
----------------------------

| Phase | Table | Operation | Key Values |
| --- | --- | --- | --- |
| User setup | DNReceivingPlan | INSERT | status=0 |
| 110x arrival (BCR OK) | DNPallet | INSERT | item=DIRECT_PB |
| 110x arrival (BCR OK) | DNStock | INSERT | item=DIRECT_PB |
| 110x arrival (BCR OK) | DNReceivingPlan | UPDATE | status 0→1 (first pallet) |
| 110x arrival (BCR OK) | DNArrival | INSERT | sch_flag=0 |
| 110x arrival (BCR fail) | DNStock | INSERT | item=IRREGULAR_PB |
| 110x→111x direct carry | DNCarryInfo | INSERT | carry_flag=3 (direct) |
| 110x→111x direct carry | DNArrival | UPDATE | sch_flag=SCHEDULED |
| 111x arrival (real) | DNCarryInfo | DELETE | direct carry cleanup |
| 111x arrival (dummy) | DNArrival | INSERT | sch_flag=0 |
| 111x processing | DNStoragePlan | INSERT | status=0, from DNReceivingPlan |
| 111x processing | DNPallet | INSERT (NEW) | replace DIRECT_PB concept |
| 111x processing | DNCarryInfo | INSERT | carry_flag=1 (storage) |
| 111x processing | DNStock | INSERT | real item, status=UU |
| 111x processing | DNWorkInfo | INSERT | job_type=02 |
| 111x processing | DNStoragePlan | UPDATE | status=1, process_qty |
| 111x processing | DNArrival | UPDATE | sch_flag=SCHEDULED |
| Force completion | DNReceivingPlan | UPDATE | status 2→4 |
| ID04 at 110x | DNStock | DELETE | DIRECT_PB |
| ID04 at 110x | DNPallet | DELETE |  |
| ID04 at 111x (after ID26) | DNStock | DELETE | real item |
| ID04 at 111x (after ID26) | DNPallet | DELETE |  |
| ID04 at 111x (after ID26) | DNCarryInfo | DELETE |  |
| ID04 at 111x (after ID26) | DNStoragePlan | DELETE |  |
| ID04 at 111x (after ID26) | DNWorkInfo | DELETE |  |
| Storage complete | DNStock | UPDATE | location_no, storage_date |
| Storage complete | DNCarryInfo | UPDATE | cmd_status→complete |
| Storage complete | DNWorkInfo | UPDATE | status→4, result_location |

* * *

12. Error Handling Reference
----------------------------

| Error | Trigger | MC Action | Lamp |
| --- | --- | --- | --- |
| BCR fail at 110x | bcr_data = ? or @ | Route to 1303, DNStock=IRREGULAR_PB | — |
| No DNReceivingPlan | status not in 0,1,2 | Silent wait in DNArrival | — |
| Control info = 2 (error completion) | Robot error | Route to 1303, reject_factor=06 | — |
| Force completion, status not 2 | User forgot batch end | Hold at 111x | ID54 lamp 17 ON |
| Zero qty last pallet | batch_last_pallet_qty = 0 | Complete plan, send empty to 1303 | — |
| All aisles full | No available aisle | Hold at 111x | ID54 lamp 01 ON |
| Aisle available again | MC detects slot | Resume, lamp OFF | ID54 lamp 01 OFF |
| Pallet removed at 110x | ID04 received | DELETE DNStock(DIRECT_PB), DNPallet | AGC handles |
| Pallet removed at 111x after ID26 | ID04 received | DELETE all related records | AGC handles |
| Lock timeout (DNStoragePlan) | Concurrent access | ROLLBACK, sleep, retry | — |
| Malformed control info | Length < 3 | rejectProcess() | ID54 lamp (data error) |