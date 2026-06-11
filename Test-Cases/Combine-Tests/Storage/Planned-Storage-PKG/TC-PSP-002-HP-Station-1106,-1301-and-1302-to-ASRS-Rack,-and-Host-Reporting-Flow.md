[[_TOC_]]
[[_TOSP_]]

# TC-PSP-002 - HP Station 1106 to ASRS Rack, and Host Reporting Flow

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional, Routing Protocol, Boundary Constraint & Error Handling Integration Test |
| **DFD Reference** | 05. Storage Process both side (HP and OP side), Station Validation Rules, & HP Side Error Flow |
| **Screen** | Background Processing Engine (`AutoStorageScheduler`, `StorageSender`, `AsrsInboundStationOperator`) |
| **Job Type** | 02: Storage |
| **Destination** | Warehouse Area 9200 (FGW1 Ambient) / Aisle 9011-9014 |
| **AGC Route Sequences** | 1106/1301/1302 → STV Transport → BCR 7211-7214 → SRM Transport → Aisle 9011-9014 |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

# Objective

To verify the inbound storage transaction processing sequence, data table state adjustments, and strict station-to-aisle matching rules for items entering via HP Stations 1106, 1301, and 1302. This ensures that authorized Zone 003 (PM/ZPCK) pallets correctly complete direct routing, while verifying that **Station 1106 systematically blocks** invalid profiles (Zones 001, 002, 004) at entry. It additionally validates hardware error handling for unreadable or invalid barcodes.

# Pre-Conditions

*   Warehouse Area 9200 contains unallocated storage rack locations within Aisles 9011–9014.    
*   AGC communication channels are online.

# Test Data
_This data should be reserved during the process_

| **Target Parameter** | **Operational Test Value** |
| --- | --- |
| **Station No** | Station 1106, 1301, 1302 (HP Side Station) |
| **Allowed SoftZone / Material Type** | **1106**: Zone 003 (PM/ZPCK) **ONLY**<br>  <br>**1301/1302**: ALL Zones<br> |
| **Prohibited SoftZone** | **1106**: Zone 001 (Tempering), Zone 002 (Ambient), Zone 004 (EMP_PB)<br>  <br>**1301/1302**: None<br> |
| **Target Area No** | Plan Area No 9200 (FGW1 Ambient Racks) |
| Document No / Receive Ticket No | RT-PKG-20260604 |
| Receive Line No | 0001 |
| Material Code (`ITEM_TYPE=ZPCK`) | PM-ZPCK-009 |
| Batch # | 0000000001 |
| BCR Data | 0000000001 |

##Test Steps
### Phase 1: Prohibited Product Zone Deviations (🔴 Negative)

_This phase verifies that system safety rules prevent non-PM products from entering dedicated PM lines specifically at Station 1106._
| **#** | **Action / Step** | **Expected Operational Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| **1** | Trigger physical arrival logic for a Zone 001, 002, or 004 profile pallet at **HP Station 1106**. | `validateStoragePlan()` flags the violation upfront. Station 1106 strictly rejects non-PM zones because it only routes to PM aisles 9011-9014. |  |  |
| **2** | Monitor downstream transport generation modules. | Processing aborts immediately. No tracking records are assigned, and no `[ID05]` STV instructions are generated, preventing undeliverable freight from entering the loop. |  |  |

### Phase 2: Barcode No Read / Data Error (🔴 Negative)

_This phase validates the system's ability to halt processing and trigger physical hardware alerts when an invalid or unreadable barcode is scanned at the HP P&D Station._
| **#** | **Action / Step** | **Expected Operational Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| **3** | Trigger physical arrival with a damaged, missing, or unregistered barcode at HP Station 1106, 1301, or 1302. | WMS receives an **[ID26] Arrival report** (`MC key = All "9"`, `Load information = "1": Present`, `BC data = Scanned BC Data`). |  |  |
| **4** | WMS internal validation module checks the payload data. | The system flags the payload as "Error detected" due to a No Read or Data Error. |  |  |
| **5** | WMS dispatches hardware alert triggers. | WMS sends an **[ID54] DO output instruction** to the origin station. Parameters: `Lamp number = "02": No Read` or `"03": Data Error`, `Instruction section = "1": ON`. The physical signal tower lights up and the buzzer sounds. |  |  |
| **6** | Operator intervenes physically: corrects the BC label (importing the storage plan), allows the conveyor to reverse back, and presses the completion button. | Operator input is registered by the system. |  |  |
| **7** | WMS clears the hardware alert state. | WMS sends a secondary **[ID54] DO output instruction** with `Instruction section = "0": OFF`. The signal tower turns off and the buzzer stops. |  |  |
| **8** | Verify post-error routing. | The pallet either continues to the normal `[ID05]` flow (if corrected) or the flow ends (if removed). |  |  |

### Phase 3: Inbound Storage (🟢 Positive)

_This phase verifies the complete STV-to-SRM data handover for pallets staying within the storage facility (e.g., Area 9200)._
| **#** | **Action / Step** | **Expected Operational Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| **9** | Trigger physical arrival of a valid pallet (Zone 003) at HP P&D Station 1106, 1301, or 1302. | WMS receives an **[ID26] Arrival report** (`MC key = All "9"`, `Control Information = "000"`). Because it is the first entry point for 1106/1301/1302, WMS assigns a new MC Key to the load. |  |  |
| **10** | System validates the plan via the `AutoStorageScheduler` background task. | Validation passes successfully. WMS matches the profile to the appropriate Warehouse Aisles (9011-9014). |  |  |
| **11** | WMS dispatches initial STV routing instructions. | WMS sends an **[ID05] Transport command** with parameters: `Transport classification = "3": Direct transfer`, `Destination = 7211-7214`. |  |  |
| **12** | STV vehicle picks up the load from the P&D station. | WMS receives an **[ID64] Pick-up completion report**, indicating STV transport has started. |  |  |
| **13** | The load arrives at the intermediate Aisle BCR station for pickup. | WMS receives an **[ID26] Arrival report** confirming transit to the assigned BCR (7211-7214). |  |  |
| **14** | WMS allocates the final bin and dispatches the SRM (Crane) storage instruction. | WMS sends a secondary **[ID05] Transport command** (`Transport classification = "1": Storage`) assigning the final Location number in Aisles 9011-9014. |  |  |
| **15** | SRM crane mechanism picks up the load and completes physical rack placement. | WMS receives **[ID64] Pick-up completion report** followed by the **[ID33] Operation completion report** (`Transport section = "1": Storage`, `Completion section = "0": Normal completion`). |  |  |
| **16** | Verify internal inventory integration. | Inventory data is updated and active transport tracking data is successfully deleted from WMS memory. |  |  |
| **17** | Wait for the next background execution cycle of `HostCommExecutor - StorageReportData`. | System executes the Planned Storage Result and formats a payload for the host. |  |  |

# SQL Verification Script
```sql
    -- 1. Check Upfront Validation Rejection Enforcement STRICTLY for Station 1106 (After Step 2)
    SELECT COUNT(*) 
    FROM DNCarryInfo 
    WHERE SOURCE_STATION_NO = '1106' AND DEST_STATION_NO NOT IN ('7211','7212','7213','7214');
    -- Expected Result: 0 (Validation must block all unauthorized transport creation attempts specifically from 1106).
    
    -- 2. Check Same Warehouse Transport Instantiation (After Step 11)
    SELECT DEST_STATION_NO, CARRY_FLAG, MC_KEY 
    FROM DNCarryInfo 
    WHERE SOURCE_STATION_NO IN ('1106', '1301', '1302');
    -- Expected Result: DEST_STATION_NO is between '7211' and '7214'. New MC_KEY assigned.
    
    -- 3. Check Final Route Leg Assignment for SRM Storage (After Step 14)
    SELECT DEST_STATION_NO, LOCATION_NO 
    FROM DNCarryInfo 
    WHERE SOURCE_STATION_NO IN ('7211','7212','7213','7214');
    -- Expected Result: DEST_STATION_NO is within '9011' to '9014'. Storage location mapped correctly.
```

# Post-Conditions

*   Tracking contexts between STV and SRM systems, ultimately clearing active `DNCarryInfo` tracking following the `ID33` process.    
*   A target reporting event is dropped into `DNHostSend`, holding correct structural tracking values (`VT01` storage origins and `9908` plant identifiers) required for automated host transmission processes.
* [Planned Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1140/9.-Planned-Storage-Result) will create after process was completed.
    
# Notes / Defects
```
[1. Physical location coordinate patterns (`DMShelf.station_no`) map dynamically following variable rack geometry definitions; ensure target database mock arrays line up with real floor layouts during execution.]
[2. The verification data on the TC just an example, follow the actual data when to do it.]
[3. This operation requires generating result data for host transmission]
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1164/TC-PSP-001-Planned-Storage-Setting-(PKG)" style="font-weight: bold; font-size: 1.1em;">📝 TC PSP 001 Planned Storage Setting (PKG)</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      &nbsp;
    </td>
  </tr>
</table>