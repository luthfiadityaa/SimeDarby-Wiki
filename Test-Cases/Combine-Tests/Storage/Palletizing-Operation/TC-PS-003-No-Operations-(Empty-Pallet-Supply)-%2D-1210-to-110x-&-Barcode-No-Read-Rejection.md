[[_TOC_]]
[[_TOSP_]]

# TC PS 003 No Operations (Supply Empty Pallet) - 1210 to 110x & Barcode No Read Rejection

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional Test |
| **DFD Reference** | [1. No Operations](/Basic-Design/DFD-WMS/Storage/PALLETIZE-OPERATION-\(HP\)/1.-NO-Operation-\(Supply-Empty-Pallet\)) <br> [06. OP Side: No Read (Go to Reject station)](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.6.xlsx?d=wcdb049b50bee4924bcc4b8625d4455dd&csf=1&web=1&e=af4Spx) |
| **Screen** | `N/A` (Background Process / AGC Triggered) |
| **Job Type** | 26: Direct Transfer |
| **Destination** | 1101-1105 (Normal) / 1303 (Reject) |
| **AGC Route** | **Normal**: ST1210 → ST110X → ST111X <br> **No-Read**: ST1210 → ST110X → ST111X → ST1303 (Reject Flow) |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

# Objective

To verify that the WMS Logic successfully processes the empty pallet supply flow triggered by the AGC system.
1.  **Positive Flow:** When a readable empty pallet arrives at input stations ST1101-ST1105, the system intercepts the ID26 message, inserts a new pallet record into `DNPallet`, and generates the corresponding stock record in `DNStock` (Item: `DIRECT_PB`).
    
2.  **Negative Flow (No Read):** When a pallet arrives with an unreadable barcode (All "?"), the system flags an error, skips palletizing, and routes the empty pallet to the QC Reject station (1303).
    

# Pre-Conditions

*   AGC is ONLINE.    
*   Supply station ST1210 and destination stations ST1101-ST1105 are NORMAL, available, and not suspended.    
*   QC Reject Station 1303 is operational and available.    
*   WMS Logic service is active and listening for messages from the AGC.
    
#### **Test Data**

| **Key** | **Value** |
| --- | --- |
| **Supply Station** | 1210 |
| **Destination Station (Normal)** | 1101, 1102, 1103, 1104, 1105 |
| **Reject Station** | 1303 |
| **Normal BCR Data** | 000000110X |
| **Error BCR Data** | All "?" (e.g., ?????????) |
| **Item Code** | DIRECT_PB |

#Test Steps

## Phase 1: Normal Barcode Read (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 1 | Simulate the AGC transferring an empty pallet from ST1210 to ST110x. AGC sends an ID26 message containing Station No = 110x and `BC_DATA` = EP-1101. | WMS successfully receives and acknowledges the ID26 message. |  |  |
| 2 | WMS Logic processes the ID26 payload to create the Pallet Record. | 1 record inserted into `DNPallet` (`STATION_NO = 110x`, `BCR_DATA = 000000110X`, `EMPTY_FLAG = 1`). Database returns a new `PALLET_ID`. |  |  |
| 3 | WMS Logic processes the Stock Record creation using the newly returned `PALLET_ID`. | 1 record inserted into `DNStock` (`PALLET_ID = 000000000000000001`, `LOCATION_NO = 110X`, `ITEM_CODE = 'DIRECT_PB'`). |  |  |
| 4 | WMS finalizes the transaction and responds to the AGC. | WMS sends ACK (Ready for Operation) back to the AGC. |  |  |
| 5 | Repeat Steps 1-4 for the remaining destination stations: 110X, providing their respective `BC_DATA`. | `DNPallet` and `DNStock` records are successfully created for each respective station. |  |  |

## Phase 2: Barcode No Read & Reject Routing (🔴 Negative)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status** |
| --- | --- | --- | --- | --- |
| 6 | AGC sends Arrival Report [ID26] at OP Station 1101 with `BC data = All "?"` (No Read) and `Load information = "1": Present`. | WMS intercepts the signal, detects the Barcode data check failure, and prevents normal DB insertion. |  |  |
| 7 | WMS sends Transport Command [ID05] to AGC for direct transfer to Robot Station 111X. | Command includes `Control Information = "11_ _"`, `Palletizing Skip = "1": Skip`, and `No Read = "1": Error`. AGC turns ON the signal tower on QC Reject Station based on Control Info. AGC responds with [ID25] Normal. |  |  |
| 8 | Conveyor transport starts, palletizing work is physically skipped. AGC sends Arrival Report [ID26] from ST111X. | Arrival report confirms `Palletizing Skip = "1": Skip`, `No Read = "1": Error`, and `Palletizing Completion = "0": Normal Completion`. |  |  |
| 9 | WMS sends final Transport Command [ID05] to AGC. | Command sets `Destination station number = 1303` (Reject Station) with identical error control flags. AGC responds with [ID25] Normal. |  |  |
| 10 | Pallet arrives at ST1303. Completion button flashes. Operator physically removes the pallet and presses the completion button. AGC sends Arrival Report [ID26]. | Arrival report payload sends `Load information = "0": None`. The completion button and signal tower lights turn OFF. Operation indication is cleared. |  |  |

## Verification SQL

```sql
    -- 1. Verify Positive Scenario: Pallet records are created correctly with the Empty Flag
    SELECT 
        PALLET_ID, 
        STATION_NO, 
        BCR_DATA, 
        EMPTY_FLAG 
    FROM DNPallet 
    WHERE STATION_NO IN ('1101', '1102', '1103', '1104', '1105') 
    AND EMPTY_FLAG = '1';
    
    -- 2. Verify Positive Scenario: Stock records are tied to the newly created pallets (DIRECT_PB)
    SELECT 
        s.PALLET_ID, 
        s.LOCATION_NO, 
        s.ITEM_CODE 
    FROM DNStock s
    JOIN DNPallet p ON s.PALLET_ID = p.PALLET_ID
    WHERE p.STATION_NO IN ('1101', '1102', '1103', '1104', '1105') 
    AND s.ITEM_CODE = 'DIRECT_PB';
    
    -- 3. Verify Negative Scenario: Ensure NO invalid stock/pallet records exist for unread barcodes
    SELECT 
        COUNT(*) AS Invalid_Records
    FROM DNPallet
    WHERE BCR_DATA LIKE '%?%'; 
    -- Expected Result: 0 (System should not save records for "No Read" pallets)
```

## Post-Conditions

*   **Positive:** Empty pallets are officially registered into the warehouse system at locations 1101-1105 and are ready for operations.    
*   **Negative:** The unreadable pallet is physically ejected to ST1303, removed by the operator, and no corrupted inventory data is saved to the WMS. Alarms and signal towers are safely cleared.
    
## Notes / Defects
```
[1. When executing the Positive Scenario, the Negative Scenario can be run immediately after to confirm system resilience.]
[2. This operation does not require generating result data for host transmission.]
[3. The Test data on the TC just an example, follow the actual data when to do it.]
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1189/TC-PS-002-Palletizing-Start-Screen" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 002 Palletizing Start Screen</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      Next →<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1160/TC-PS-004-Batch-Start" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 004 Batch Start</a>
    </td>
  </tr>
</table>