[[_TOC_]]
[[_TOSP_]]

# TC-PS-006 — Batch End, Force Completion Handling & Exception Recovery

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional, UI Validation & Exception Test |
| **DFD Reference** | [3. Batch End Database Flow](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1018/3.-Batch-End) <br> [06. Stop because Batch End is not executed although Force Completion. Consider as Data error (1101, 1102, 1103, 1104, 1105)](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.6.xlsx?d=wcdb049b50bee4924bcc4b8625d4455dd&csf=1&web=1&e=biBpga)
| **Screen** | Warenavi Interface (Batch End Operation Screen) |
| **Job Type** | 02:Storage |
| **Destination** | ST111X (Robot Station) → Assigned Storage Location / Reject Station ST1303 |
| **AGC Route** | **Normal Completion**: ID26 (OP Station) → ID05 → ST111X → ID26 → ID05 (Storage) → ID33 (Complete)  <br> **Force Completion**: ID26 (OP Station) → ID05 → ST111X → ID26 (Force Complete) → ID54 (Alarm) → ID54 (Clear) → ID05 (Storage → ID33 (Complete)  / Reject → ID26 (QC Station)) 
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

# Objective

To verify front-end validation boundaries, background database updates, and physical hardware signaling during the **Batch End** cycle. This test covers standard force completion flows, alarm locking mechanisms when a Force Completion arrives before a Batch End is executed, and routing handling for empty pallet exceptions.

# Pre-Conditions

*   WMS backend processing engine and station polling tasks are active.    
*   An active production batch exists in `DNReceivingPlan` (`STATUS_FLAG = 1: Working`).    
*   Physical conveyor lines, Robot Station cells (`ST111x`), and ASRS storage zones are online and communicating.
    
# Test Data

| **Key** | **Value** |
| --- | --- |
| **Target Robot Station Range** | 111x |
| **Reject Station Location** | 1303 |
| **Lamp Control Variable** | 17: Batch End Incomplete / Data Error |
| **Abnormal Palletizing Completion Code** | 1: Force Completion |

# Test Steps
## Phase 1: Screen-Level UI Input Validation (Boundary Verification)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 1 | Navigate to the Warenavi Batch End Screen, leave the **Storage Qty** input blank, and click submit. | System blocks transaction and throws a validation error: _"Storage Qty is required to be filled"_. |  |  |
| 2 | Input a value of **0** for **Total Actual Carton Qty** and execute the submit transaction. | Transaction fails on-screen validation rule: _"Total Actual Carton Qty must be greater than 0"_. |  |  |
| 3 | Input a **Storage Qty** value that exceeds the value assigned to the **Total Actual Carton Qty**. | System blocks transaction: _"Storage Qty cannot be greater than Total Actual Carton Qty"_. |  |  |
| 4 | Input a **Storage Qty** value that is greater than the original **Planned Carton Qty**. | System blocks submission: _"Storage Qty cannot be greater than Planned Carton Qty"_. |  |  |

## Phase 2: Standard Batch End & Force Completion (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 5 | Input a valid **Storage Qty (>= 0)** matching parameters on the terminal and execute **Batch End - Set (F2)**. | `DNReceivingPlan` updates: `STATUS_FLAG = 2: Waiting for Last Pallet`, `BATCH_LAST_PALLET_QTY = [Input]`, and logs the `BATCH_END_TIME` timestamp. |  |  |
| 6 | Trigger a hardware Force Completion event. AGC sends an **ID26** from Robot Station ST111X with `Control Information = "001"` and `Palletizing Completion = "1": Force Completion`. | WMS matches the inbound signal against the active batch record and confirms the `STATUS_FLAG = 2` wait state. |  |  |
| 7 | Check internal relational tables immediately following the execution of the ID26 process loop. | **Simultaneous updates hit the database:**<br>  <br>• `DNStock` matches product details and updates `PLAN_QTY`.<br>  <br>• `DNPallet.EMPTY_FLAG` drops to **0:Normal Pallet**.<br>  <br>• `DNStoragePlan` sets status to **4 (Completed)** and increments total tracking metrics.<br> |  |  |
| 8 | Monitor downstream physical routing commands. | System allocates an open bin storage location, establishes a valid record set under tracking contexts, and issues an outbound **ID05** transport command. |  |  |

## Phase 3: Force Completion Arrives Before Batch End Execution (🔴 Negative)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 9 | Simulate operational desynchronization. Trigger a Force Completion at the robot station _before_ executing the screen transaction. AGC sends an **ID26** with `Palletizing Completion = "1"` while batch status is **1: Working**. | WMS evaluates background state rules, identifies a data sequence exception, and intercepts the operational flow. Normal storage allocation workflows are restricted. |  |  |
| 10 | Inspect the communication queue for outgoing station message frames. | WMS builds and writes an automated **ID54 DO Output Instruction** to the AGC with `Lamp number = "17": Batch End Incomplete` and `Instruction section = "1": ON`. |  |  |
| 11 | Verify physical facility indicators and material transit tracking behavior on the line. | The physical station signal tower flashes and the audible buzzer sounds. **The physical conveyor line immediately issues a stop instruction**. |  |  |

## Phase 4: Operational Exception Recovery from Phase 3 State
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 12 | While the conveyor is interlocked and flashing, navigate to the terminal interface and execute **Batch End**. | System processes the data correction and transitions the active tracking indices to a normal completion path. |  |  |
| 13 | Monitor the real-time communication buffer immediately following screen entry validation. | WMS updates equipment nodes by sending an **ID54 instruction** with `Lamp number = "17": Data Error` and `Instruction section = "0": OFF`. |  |  |
| 14 | Observe material transit lines and system equipment performance. | The signal tower turns off and the audio buzzer stops. **The conveyor line automatically unlocks and restarts transit**, processing normal storage commands. |  |  |
| 15 | Processing normal storage commands. | Continue to [TC PS 005 Robot Station 111x to ASRS Rack, and Host Reporting Flow](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1191/TC-PS-005-Robot-Station-111x-to-ASRS-Rack-and-Host-Reporting-Flow) |

## Phase 5: Force Completion with Empty Pallet Condition (🔴 Negative)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 16 | Trigger a Force Completion on an empty pallet container while `Status_Flag = 1`. When the alert block triggers, access the screen tool, input an explicit quantity value of **0**, and submit. | Relational tracking tables handle the state override profile: `DNStoragePlan.STATUS_FLAG` is changed to **4 (Completed)**. |  |  |
| 17 | Trace physical pallet routing data profiles inside background tracking logs. | The warehouse engine skips standard rack allocation loops, prepares an interface payload change for `DNHostSend`, and dispatches an **ID05 command directing the empty unit to Reject Station 1303**. |  |  |

# Verification SQL

```sql
    -- 1. Verify Phase 2 State Transitions (After Step 7)
    SELECT STATUS_FLAG, BATCH_LAST_PALLET_QTY, BATCH_END_TIME 
    FROM DNReceivingPlan 
    WHERE BATCH_STATION_NO = '111x';
    -- Expected Result: STATUS_FLAG = '2', BATCH_LAST_PALLET_QTY = Entered Storage Qty, Timestamp populated.
    
    SELECT EMPTY_FLAG, STATUS_FLAG FROM DNPallet WHERE CURRENT_STATION_NO = '111x';
    -- Expected Result: EMPTY_FLAG = '0' (Loaded/Full), STATUS_FLAG variable updated.
    
    SELECT STATUS_FLAG, PROCESS_QTY, RESULT_QTY FROM DNStoragePlan WHERE BATCH_STATION_NO = '111x';
    -- Expected Result: STATUS_FLAG = '4' (Completed), PROCESS_QTY matches RESULT_QTY tracking fields.
    
    -- 2. Verify Phase 3 Alarm Locks & Intercept Processing (After Step 11)
    SELECT COUNT(*) AS Storage_Commands_Generated
    FROM DNCarryInfo 
    WHERE SOURCE_STATION_NO = '111x' AND CMD_STATUS = 'START';
    -- Expected Result: 0 (System must not issue storage transport commands while locked in alarm state 17).
    
    -- 3. Verify Phase 5 Storage Target Redirects for Empty Container Exceptions (After Step 16)
    SELECT DEST_STATION_NO, TRANSPORT_CLASSIFICATION 
    FROM DNCarryInfo 
    WHERE SOURCE_STATION_NO = '111x';
    -- Expected Result: DEST_STATION_NO = '1303' (Pallet is rerouted to Reject Station instead of storage racks).
```

# Post-Conditions

*   **Normal Storage Path:** Production plans are finalized, total item balances register within active `DNStock` profiles, and tracking references are dropped after transit commands execute successfully.    
*   **Abnormal Path:** Out-of-sequence exceptions create safe hardware locks (Conveyor Stop) via `ID54` to protect tracking integrity until corrected by on-screen user actions.    
*   **Empty Deviation Path:** Invalid product cycles are terminated, data updates are logged for upstream host reporting networks (`DNHostSend`), and structural hardware directs the empty pallet target out of the automation lane to location 1303.
    
# Notes / Defects
```
[1. Executing Phase 3 requires the automated background tasks to be active on the test database instance to capture and trigger the `ID54` indicator sequence instantly.]
[2. Daily cleanup routine dependencies must remain inactive during validation to prevent transaction record purge interference.]
[3. This operation requires generating result data for host transmission- Production Storage Result in TC PS 005 Robot Station 111x to ASRS Rack, and Host Reporting Flow, unless checking Phase 5 empty pallet termination protocols.]
[4. The verification data on the TC just an example, follow the actual data when to do it.]
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1191/TC-PS-005-Robot-Station-111x-to-ASRS-Rack-and-Host-Reporting-Flow" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 005 Robot Station 111x to ASRS Rack, and Host Reporting Flow</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      &nbsp;
    </td>
  </tr>
</table>