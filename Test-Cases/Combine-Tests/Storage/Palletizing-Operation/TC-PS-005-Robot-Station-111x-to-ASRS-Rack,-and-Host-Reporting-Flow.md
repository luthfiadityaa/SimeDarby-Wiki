[[_TOC_]]
[[_TOSP_]]

# TC PS 006 Robot Station 111x to ASRS Rack, and Host Reporting Flow 

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional & Integration Test |
| **DFD Reference** | [05. Storage Process](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.6.xlsx?d=wcdb049b50bee4924bcc4b8625d4455dd&csf=1&web=1&e=LdJMEt) <br> [Data Flow Palletizing Operation](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1042/Data-Flow?anchor=a4.-phase-3-%E2%80%94-arrival-at-111x-(storagesender-path-2)) |
| **Screen** | Background Processing Services (`StorageSender`, `AsrsInboundStationOperator`, `CarryCompleteOperator`) |
| **Job Type** | 02:Storage|
| **Destination** | **Same WH**: ST710x (Inbound BCR) → `9100` <br> **Cross WH**: ST7107 - 7110 (Inbound BCR) → `9200` |
| **AGC Route** | **Same WH**: ST111x → ID05 (`DIRECT_TRAVEL` to 710x) → ST710x → ID05 (`STORAGE` to Rack Location) → ID33 (Complete) <br> **Cross WH**: ST111x → ID05 (`DIRECT_TRAVEL` to 720x) → ST720x → ID05 (`STORAGE` to Rack Location) → ID33 (Complete) |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

# Objective

To verify the multi-phase background inventory transformation, physical storage lane execution, shelf management reservation, and final host synchronization reporting when a completed pallet drops from the robotic workstation (ST111x) into the automated warehouse system.

# Pre-Conditions

*   WMS Backend Engine and background poll loop tasks (`StorageSender`, `ShelfMonitor`) are active.    
*   The empty pallet placeholder item (`DIRECT_PB`) exists under the tracking pallet profile at station `110x`.    
*   Active batch record exists in `DNReceivingPlan` pointing to `BATCH_STATION_NO = '111x'`.    
*   ASRS Inbound BCR (`710x` or `7107-7110`) and STV/SRM crane sub-systems are online and operating without faults.    
*   At least one storage bin matching the designated target `soft_zone (001)` or `soft_zone (002)` and aisle configuration is free (`status = EMPTY`).   

# Test Data

| **Key** | **Value** |
| --- | --- |
| **Robot Station (Source)** | 1111 (or applicable until 1115) |
| **ASRS Inbound BCR Station** | **Same WH**: 7101 (or applicable until 7106) <br> **Cross WH**: 7107 (or applicable until 7110) |
| **Temporary Storage Location** | VT01 |
| **Target Storage Area Range** | `9001-9010` |
| **Target Aisle Node** | **Same WH**: 9001-9006 , <br> **Cross WH**: 9007-9010
| **SAP Corporate Plant Code** | 9908 |

# Test Steps

## Phase 1: Arrival at 111x (StorageSender Path 2 Execution) (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 1 | Simulate pallet arrival at the robot cell station line by inserting a record into the tracking log table (`PalletizeStorageStationOperator.arrival`). | Record successfully hits `DNArrival` with fields: `station = 111x`, `carry_key = 00000001`, and `sch_flag = 9:No Need to Schedule`. |  |  |
| 2 | Allow `StorageSender.dispatchPalletizeStorage` to pick up and process the non-dummy tracking payload from the station loop. | System polls and filters matching `station_no = 111x`, `send_flag = NOT_SEND`, and `sch_flag = 9:No Need to Schedule`. |  |  |
| 3 | Evaluate `StorageSender.processPalletizeStorage` functional validation points (Control Flags, Quantities, and Data Conversions). | • `control_info` validation checks out (`000` = normal / `001` = force completion).<br>  <br>• `DNReceivingPlan` details matched via `batch_station_no = 111x` (`status` in 0, 1, 2).<br>  <br>• Correct `plan_qty` evaluated (If status 0 or 1, matches `batch_qty_ctrn_pl`; if status 2, defaults to `batch_last_pallet_qty`).<br> |  |  |
| 4 | Monitor database side-effects during real inventory generation phase. | • **DELETE executed** on placeholder stock record from `DNStock` (`item = DIRECT_PB`) at source `110x`.<br>  <br>• **INSERT executed** on `DNStoragePlan (00000001)` (inherits status=0, plan_qty, bcr_data, etc.; maps `storage_location_from = VT01`).<br>  <br>• **INSERT executed** on `DNStock (00000001)` (real item details mapped, `stock_qty = 0`, `storage_type = 2 (NEW)`).<br> |  |  |
| 5 | Verify physical unit tracking and task creation profiles before transmission to the system line. | • **INSERT executed** on `DNWorkInfo (00000001)` (stores system transaction values: `user_id = AS21Param.SYS_USER_ID`, `terminal_no = SYS_TERMINAL_NO`, `storage_location_from = VT01`).<br>  <br>• `DNPallet (000000000000000001)` updates tracking variables (`soft_zone`, `aisle_collect_key`).<br>  <br>• `DNCarryInfo (00000001)` re-routed (`dest = 710x/7107-7110`, `source = 111x`, `cmd_status = START`, `wait_reason = OFF`).<br> |  |  |
| 6 | Observe progress accumulator and physical dispatch command via `StorageSender`. | `DNReceivingPlan.progress_qty` increments by evaluated plan qty (flips status to 4 if final pallet). WMS fires an outbound **ID05** payload to the AGC line (`dest = 710x/7107-7110`, `work_type = 26: Direct Transfer`). |  |  |

## Phase 2: Arrival at 710x/7107-7110 (OP ASRS Inbound BCR Allocation) (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 7 | Pallet triggers scan event at the entry BCR point (`AsrsInboundStationOperator`). | Verification loop confirms `wh_station = end_station`. `DNCarryInfo (00000001)` updates (`carry_flag = 1 (STORAGE)`, `source = 710x/7101-7110`, `dest = 9100/9200`, `cmd_status = START`). `DNArrival` entry logged. |  |  |
| 8 | Allow bin location allocation service loops to query for open destination shelves. | `StorageSender` verifies station constraints (`max_instruction = 2`, normal mode). `LocationManager.searchLocation` looks for `DMShelf` entries with `status = EMPTY` and matching `soft_zone = 001`. |  |  |
| 9 | Reserve shelf space and dispatch transport command execution profiles. | • `ShelfController.reserveShelf` updates target coordinate `DMShelf.station_no` (`status = 2 (RESERVED)`).<br>  <br>• `DNCarryInfo (00000001)` assigns destination values (`dest = DMShelf.station_no`, `aisle_station_no = 9001-9010).<br>  <br>• `DNArrival` flags update (`send_flag = SENDED`).<br> |  |  |
| 10 | Verify outbound equipment routing instruction. | WMS outputs an **ID05** routing control to the machine transport network (`dest = DMShelf.station_no`, `location_no = DMShelf.station_no`, `carry = STORAGE`). |  |  |

## Phase 3: ID33 Storage Complete & Host Interface Notification (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 11 | Simulate warehouse storage completion by sending an inbound **ID33** complete from the crane controller line (`CarryCompleteOperator`). | WMS receives notification. **Updates execution target** `DNWorkInfo (00000001)` (⚑ `status = 4`, `result_location = DMShelf.station_no`, `result_qty = [real qty]`). |  |  |
| 12 | Monitor host communication logs for business reporting records (`HostSendController.insertByWorkInfo`). | **INSERT executed** on `DNHostSend` data store interface containing parsed values: ⚑ `job_type`, `report_flag = 0`, `result_location = DMShelf.station_no`, `plant = 9908`. Original tracking locations and quality control states are mirrored accurately. |  |  |
| 13 | Verify stock activation and physical ledger transformations (`AsStockController.addStock`). | • `DNStock (00000001)` updates live tracking data (⚑ `location_no = DMShelf.station_no`, `stock_qty += qty`, `plan_qty = 0`).<br>  <br>• `DMShelf (DMShelf.station_no)` changes status from reserved to ⚑ `1 (OCCUPIED)`.<br>  <br>• `DNPallet` updates tracking variables (`current_station = DMShelf.station_no`, `status = STORED`).<br> |  |  |
| 14 | Observe history updates and structural tracking cleanup processing blocks. | • Audit trail data populated into `DNStockHistory`.<br>  <br>• `DNStoragePlan (00000001)` transitions state to ⚑ `4 (COMPLETE)`.<br>  <br>• `DNInOutResult` transaction entry logged (`result_kind = 1`).<br>  <br>• `Id33Process` issues an unconditional **DELETE** on routing reference `DNCarryInfo (00000001)`.<br> |  |  |
| 16 | Wait for the next background execution cycle of `HostCommExecutor - ProductionStorageReportData`. | System execute the [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1134/5.-Production-Storage-Result) |  |  |

⚑ = Fields to verify during testing

# Verification SQL

```sql
    -- 1. Verify Phase 1 Data Conversions (After Step 6)
    SELECT ITEM_CODE, STOCK_QTY, STORAGE_TYPE FROM DNStock WHERE PALLET_ID = '000000000000000149';
    -- Expected Result: Real material item configuration loaded, STOCK_QTY = 0, STORAGE_TYPE = '2'.
    
    SELECT JOB_TYPE, USER_ID, TERMINAL_NO, STORAGE_LOCATION_FROM FROM DNWorkInfo WHERE WORK_KEY = '00000001';
    -- Expected Result: JOB_TYPE matched from plan, USER_ID = 'AS21Param.SYS_USER_ID', FROM = 'VT01'.
    
    -- 2. Verify Phase 2 Allocation Inbound States (After Step 10)
    SELECT DEST_STATION_NO, AISLE_STATION_NO, CMD_STATUS FROM DNCarryInfo WHERE CARRY_KEY = '00000001';
    -- Expected Result: DEST_STATION_NO points to allocated shelf station (DMShelf.station_no), AISLE_STATION_NO = '9001-9010', STATUS = 'START'.
    
    -- 3. Verify Phase 3 Storage Completion Ledger Updates (⚑ Critical Paths after Step 14)
    SELECT STATUS, RESULT_LOCATION, RESULT_QTY FROM DNWorkInfo WHERE WORK_KEY = '00000001';
    -- Expected Result: STATUS = '4', RESULT_LOCATION = 'DMShelf.station_no', Qty matches completed load values.
    
    SELECT JOB_TYPE, REPORT_FLAG, RESULT_LOCATION, PLANT FROM DNHostSend WHERE CARRY_KEY = '00000001';
    -- Expected Result: Job Type successfully populated from WorkInfo, REPORT_FLAG = 0, PLANT = '9908'.
    
    SELECT LOCATION_NO, STOCK_QTY, PLAN_QTY FROM DNStock WHERE PALLET_ID = '000000000000000149';
    -- Expected Result: LOCATION_NO updated to 'DMShelf.station_no', STOCK_QTY initialized to full transaction qty, PLAN_QTY reset to 0.
    
    SELECT STATUS FROM DMShelf WHERE LOCATION_NO = 'DMShelf.station_no';
    -- Expected Result: STATUS = '1' (Occupied).
    
    SELECT STATUS_FLAG FROM DNStoragePlan WHERE PLAN_UKEY = '02000001';
    -- Expected Result: STATUS_FLAG = '4' (Complete).
    
    SELECT COUNT(*) FROM DNCarryInfo WHERE CARRY_KEY = '00000001';
    -- Expected Result: 0 (The active transfer tracking reference record must be completely deleted).
```

# Post-Conditions

*   The empty placeholder (`DIRECT_PB`) is dropped and real validated product inventory is successfully established in the destination ASRS storage lane location.    
*   System internal configuration indices reflect a decrease in open location availability, triggering any necessary `ID54` signal column modifications.    
*   A target reporting event is dropped into `DNHostSend`, holding correct structural tracking values (`VT01` storage origins and `9908` plant identifiers) required for automated host transmission processes.
* [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1134/5.-Production-Storage-Result) will create after process was completed.
    
# Notes / Defects
```
[1. Step 5 maps specific user profiles (`SYS_USER_ID`) into work instructions. If an engineering maintenance account bypasses this cycle, verify that standard processing fallback defaults prevent system background terminal drops.]
[2. Physical location coordinate patterns (`DMShelf.station_no`) map dynamically following variable rack geometry definitions; ensure target database mock arrays line up with real floor layouts during execution.]
[3. The verification data on the TC just an example, follow the actual data when to do it.]
[4. This operation requires generating result data for host transmission]
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1160/TC-PS-004-Batch-Start" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 004 Batch Start</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      Next →<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1155/TC-PS-006-Batch-End-Force-Completion-Handling-Exception-Recovery" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 006 Batch End, Force Completion Handling & Exception Recovery</a>
    </td>
  </tr>
</table>