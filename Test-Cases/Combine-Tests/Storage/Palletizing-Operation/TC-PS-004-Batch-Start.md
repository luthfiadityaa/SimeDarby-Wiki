[[_TOC_]]
[[_TOSP_]]

# TC-PS-004 Batch Start

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional Test |
| **DFD Reference** | [2. Batch Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1017/2.-Batch-Start) <br> [05. OP Side Palletizing Flow / 06. Wait until Batch Start is executed. Consider as Data error (1101, 1102, 1103, 1104, 1105) ](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.6.xlsx?d=wcdb049b50bee4924bcc4b8625d4455dd&csf=1&web=1&e=uqJXjJ) |
| **Screen** | Background Process (PrePalletizeStationOperator) & Warenavi Interface |
| **Job Type** | 26: Direct Transfer (Empty Pallet) |
| **Destination** | ST111X (Robot Station - e.g., ST1111) |
| **AGC Route** | • **Batch Executed**: ID26 (@110x) → ID05 (110x → 111x) → ID25 → ID26 (@111x) <br> • **Batch Unexecuted**: ID26 (@110x) → ID54 (Lamp ON) → ID54 (Lamp OFF) → ID05 (110x → 111x) → ID25 → ID26 (@111x) <br> • **Error Completion:** ID26 (@1101) → ID05 (1101 → 1111) → ID25 → ID26 (@111x, Comp="2") → ID30 (Status="2: Under breakdown") → ID35 (Transport data deletion processing) |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass [ ] Fail |

# Objective

To verify the full operational data flow and state transitions of the Empty Pallet Supply cycle from the OP Side Palletizing Station (ST110X) to the Robot Station (ST111X). This includes confirming the negative scenario where an arrival report (ID26) is received before a "Batch Start" is executed (triggering a data error and ID54 alarm), followed by the system's recovery and successful generation of the transfer command (ID05) once the user inputs the Batch Start data into the `DNReceivingPlan`.

# Pre-Conditions

*   AGC/Hardware sub-systems are online and marked as ONLINE.    
*   Physical Station ST110x is operational and physically occupied by an empty pallet.    
*   Physical Station ST111x (Target Robot Station) is currently EMPTY.    
*   There is currently **NO** active record in `DNReceivingPlan` for Station 110x (Status = 0: Not Started).
    
# Test Data

| **Key** | **Value** |
| --- | --- |
| **Source Station (OP Side)** | 110x |
| **Destination Station (Robot)** | 111x |
| **Lamp Number** | 16: Batch Start Incomplete |
| **Material Code / Batch No** | MAT-01 / LOT-999 **(User Input)** |

# Test Steps

## Phase 1: When "Batch Start" is not executed yet (🔴 Negative)
_This phase verifies the system's alert state when a pallet arrives but no batch plan has been initiated._
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 1 | AGC sends an Arrival Report [ID26] at OP Station 110x (MC Key = All "9", Load size = 000, Load info = "1": Present). | WMS intercepts the signal and checks `DNReceivingPlan`. System identifies that "Batch Start" is not executed yet, logging a Data Error for Station 110x. |  |  |
| 2 | Monitor the outbound communication buffer for the AGC system. | WMS generates and sends an [ID54] DO Output Instruction to Station 111x with `Lamp number = "16"` and `Instruction section = "1": ON`. |  |  |
| 3 | Observe physical hardware behavior. | The signal tower lights up and the buzzer sounds to alert the operator. |  |

## Phase 2: "Batch Start" is Executed and Supply Resumes (🟢 Positive)
_This phase confirms that once the operator initiates the batch, the system resolves the error, halts the alarm, and commands the pallet transfer._
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 4 | Operator inputs Batch Data (Material, Batch No, Station 110x) into the Warenavi Interface and executes "Batch Start". Go to [TC PS 002](/Test-Cases/Combine-Tests/Storage/Palletizing-Operation/TC-PS-002-Palletizing-Start-Screen) for details. | System creates a new record in `DNReceivingPlan` with user inputs, System Timestamp, and `STATUS_FLAG = 0`. |  |  |
| 5 | Monitor the outbound communication buffer for the AGC system immediately after Batch Start. | WMS detects the active batch and generates an [ID54] instruction with `Lamp number = "16"` and `Instruction section = "0": OFF`. The signal tower turns off and the buzzer stops. |  |  |
| 6 | Background Process (PrePalletizeStationOperator) polling cycle triggers. | WMS evaluates logic:<br>  <br>1. Active Batch in `DNReceivingPlan`? (Yes)<br>  <br>2. Any pallet at ST111x? (No)<br>  <br>3. Empty pallet at ST110x? (Yes).<br> |  |  |
| 7 | WMS updates database and dispatches transport command to AGC. | System updates `DNPallet` (`CURRENT_STATION_NO = 111x`). Generates and sends [ID05] Transport Command with `Source = 110x`, `Dest = 111x`, `Transport classification = "3"`. |  |  |
| 8 | AGC receives the command and begins physical transfer. | AGC responds with [ID25] (Response section = "00": Normal). Conveyor transport starts. Palletizing work begins. |  |  |
| 9 | AGC sends Arrival Report [ID26] from the Robot Station (ST111x) with:<br>  <br>• `Control Information = "000"`<br>  <br>• `Palletizing Skip = "0": None"`<br>  <br>• `No Read = "0": None"`<br>  <br>• `Palletizing Completion = "0": Normal Completion`<br> | WMS successfully receives the ID26 signal. It identifies the pallet at ST111x, updates `DNStock` with full carton quantities, marks `DNPallet.EMPTY_FLAG = 0` (Loaded), and increments `DNStoragePlan.PROCESS_QTY`. |  |  |
| 10 | Allow the storage allocation logic to execute immediately after the normal completion is recorded. | WMS continues to the normal flow:<br>  <br>1. Searches and finds an empty location in `PLAN_AREA`.<br>  <br>2. Updates `DNPallet` (`STATUS_FLAG = 1`, `CURRENT_STATION_NO = Station No`).<br>  <br>3. Generates `DNWorkInfo` & `DNCarryInfo` records.<br>  <br>4. Dispatches an [ID05] Transfer Command to move the full pallet to the assigned storage rack location.<br> |

## Phase 3: Error Completion & Manual Pallet Deletion (🔴 Negative)
This phase verifies tracking termination and line fault resets when the robot cell encounters an abnormal breakdown state.

| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 11 | During active palletizing execution at the cell, simulate a critical hardware/process fault. AGC transmits an Arrival Report **[ID26]** from Robot Station 111x. | WMS intercepts the message containing explicit fault properties:<br>  <br>• `Load size information = 000`<br>  <br>• `Load information = "1": Present`<br>  <br>• `Control Information = "002"`<br>  <br>• `Palletizing Skip = "0": None`, `No Read = "0": None`<br>  <br>• `Palletizing Completion = "2": Error Completion`.<br> |  |  |
| 12 | Check the diagnostic log and tracking screen interface following the error arrival signal. | AGC issues a **[ID30] Machine Status Report** showing `Status = "2": Under breakdown`. The system throws a breakdown alert on the MOS view. Operators use this window to remove the physical pallet container and clear out the hardware line fault. |  |  |
| 13 | Once the physical clearance and reset work finishes, simulate line clearance confirmation. AGC transmits a Transport Data Deletion Report **[ID35]**. | WMS accepts the ID35 instruction, executes internal **Transport data deletion processing**, and completely flushes out all active conveyor tracking records from the MOS node memory to reset the line state. |

# Verification SQL

```sql
    -- 1. Verify Batch Start Record Insertion (After Step 4)
    SELECT 
      ITEM_CODE, 
      PLAN_LOT_NO, 
      BATCH_STATION_NO, 
      STATUS_FLAG 
    FROM 
      DNReceivingPlan 
    WHERE 
      BATCH_STATION_NO = '110x' AND STATUS_FLAG = '0';
    -- Expected result: 1 record matching the user input data.
    
    -- 2. Verify Pallet Update (After Step 7)
    SELECT 
      PALLET_ID, 
      CURRENT_STATION_NO, 
      EMPTY_FLAG 
    FROM 
      DNPallet 
    WHERE 
      CURRENT_STATION_NO = '111x';
    -- Expected result: Returns the transferred pallet record showing it is now assigned to the Robot Station (111x).
    
    -- 3. Verify Inventory and Storage Plan Updates (After Step 9)
    SELECT 
      ITEM_CODE, 
      PLAN_QTY, 
      PROCESS_QTY 
    FROM 
      DNStoragePlan 
    WHERE 
      BATCH_STATION_NO = '110x';
    -- Expected result: PROCESS_QTY should be incremented by the BATCH_QTY_CTRN_PL value.

    SELECT 
      PALLET_ID, 
      EMPTY_FLAG, 
      STATUS_FLAG 
    FROM 
      DNPallet 
    WHERE 
      CURRENT_STATION_NO = '111x';
    -- Expected result: EMPTY_FLAG = '0' (Pallet is now loaded/full).

    -- 4. Verify Storage Tracking Generation (After Step 10)
    SELECT 
      CARRY_KEY, 
      SOURCE_STATION_NO, 
      DEST_STATION_NO, 
      CMD_STATUS 
    FROM 
      DNCarryInfo 
    WHERE 
      SOURCE_STATION_NO = '111x';
    -- Expected result: 1 record routing from ST111x to the newly allocated rack location coordinate, status = 2 (or active).

    -- 5. Verify Breakdown Intercept and Transport Data Deletion (After Step 13)
    SELECT 
       COUNT(*) 
    FROM 
       DNCarryInfo 
    WHERE 
       SOURCE_STATION_NO = '111x';
    -- Expected result: 0 (The ID35 transaction must execute a complete data purge for the target conveyor node from MOS memory).
```

# Post-Conditions

*   The physical empty pallet arrives and is stationed at ST111x (Robot Station).    
*   The `DNPallet` record confirms `CURRENT_STATION_NO = 111x`.    
*   The system is now ready for the **Pallet Full Process** (where Palletizing work is completed and ID26 Normal Completion is sent from 111x).    
*   The ID54 alarm instructions (both ON and OFF) are accurately logged in the WMS communication history.  
*   **Error Deletion Path:** Robot breakdowns interrupt normal routing, transition the equipment log to a breakdown status (`ID30`), and completely clear the active tracking reference layers upon receiving an `ID35` data deletion frame.  

# Notes / Defects

```
[1. This testing scenario is valuable from a negative perspective; however, it may be skipped if necessary. The process can proceed directly with Phase 2: "Batch Start" execution, after which the supply operation will resume.]
[2. When testing Phase 1, ensure the background processes are actively running to catch the data error instantly.]
[3. Source station and Destination stations are rigidly mapped (e.g., 110x always pairs with 111x). Ensure the test data strictly follows this 1-to-1 routing mapping.]
[4. This operation doesn't require generating result data for host transmission]
[5. The verification data on the TC just an example, follow the actual data when to do it.]
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1158/TC-PS-003-No-Operations-(Empty-Pallet-Supply)-1210-to-110x-Barcode-No-Read-Rejection" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 003 No Operations (Empty Pallet Supply) - 1210 to 110x & Barcode No Read Rejection</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      Next →<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1191/TC-PS-005-Robot-Station-111x-to-ASRS-Rack-and-Host-Reporting-Flow" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 005 Robot Station 111x to ASRS Rack, and Host Reporting Flow</a>
    </td>
  </tr>
</table>