[[_TOC_]]
[[_TOSP_]]

TC-PS-001 — Empty Pallet Supply via Standard Retrieval Cycle (ST1210 Triggered)
===============================================================================

| **Field**         | **Value** |
| ----------------- | --------- |
| **Test Type**     | Functional Test |
| **DFD Reference** | [1. No Operations (Empty Pallet Supply)](/Basic-Design/DFD-WMS/Storage/PALLETIZE-OPERATION-\(HP\)/1.-NO-Operation-(Supply-Empty-Pallet)) <br> [24. Automated Retrieval Trigger](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.6.xlsx?d=wcdb049b50bee4924bcc4b8625d4455dd&csf=1&web=1&e=GgvKub) |
| **Screen**        | Background Process (`RetrievalTriggerAllocator`) & Hardware Trigger |
| **Job Type**      | 23:Unplanned Retrieval |
| **Destination**   | ST1210 |
| **AGC Route**     | **ID66** (@1210) → **ID12** (@Shelf) → **ID26** (@S1210)|
| **Tester**        | _____________ |
| **Test Date**     | _____________ |
| **Overall Status**| [ ] Pass [ ] Fail |

Objective
---------

To verify the full operational data flow and state transitions of the Empty Pallet Supply cycle triggered at station `ST1210`, as defined in the hardware-to-system messaging specification. This includes confirming that when an empty pallet request (`ID66`) is raised at Station 1210, the system correctly searches for available stock (`EMP_PB`), allocates a bin within the designated empty areas (`9007–9010`), generates the `ID12` retrieval command, manages intermediate station handling (e.g., `710x BCR`), and completes the entire process flow accurately.

Pre-Conditions
--------------

*   AGC/Hardware sub-systems are online and marked as `ONLINE`.    
*   Daily maintenance cleanup routines are **not** currently processing.    
*   Physical Station `ST1210` is operational, not suspended, and is physically NOT occupied.   
*   The DMStation.RETRIEVAL_TRIGGER_REQUEST for `1210` initial value is 1: Not Requested 
*   Adequate empty pallets (`item=EMP_PB` or equivalent digital placeholder) are digitally tracked as `STORED` inside warehouse storage areas `9007-9010` with at least one location marked as `1` (OCCUPIED). 

Test Data
---------

| **Key** | **Value** |
| ------- | --------- |
| Target Supply Station | 1210 |
| Source ASRS Storage Areas | 9007 / 9008 / 9009 / 9010 |
| Target Empty Pallet Item Code | EMP_PB |
| Target BCR | ABCDE00001 or another| 

Test Steps
----------

### Phase 1: When MC has data of any stock items to be retrieved (🟢 Positive)
This phase verifies normal operation when the Material Control (MC) system receives initial allocation signals.

| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| ----- | -------- | ------------------- | ----------------- | ------------------------ |
| 1 | AGC automatically trigger, supply signal for Station `1210` and send to WareNavi. | WMS intercepts the signal, executes the inventory search routine, and responds immediately with an `ID46` message mapping a successful (`00`) NORMAL reception code. |  |  |
| 2 | Allow `RetrievalTriggerAllocator` polling to pick up and process the new supply request. | - Background loop initiates. System locates suitable empty stock (`item=EMP_PB`) from the defined source areas `9007-9010`.<br>  <br>- A `DNCarryInfo` record is inserted (mapping unique MC key, `CMD_STATUS=2`, `dest=1210`, `end=1210`).

### Phase 2: When MC has no data of stock items to be retrieved (🔴 Negative)
This phase simulates an unexpected process where no empty stock/pallets can be physically or digitally retrieved, forcing an alert state

| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 3 | Automatically or Manually fill the ASRS map (or simulate via SQL) so that the count of empty locations is 0. | No Empty Plalet Founds. |  |  |
| 4 | Monitor the outbound communication buffer for the AGC system. | WMS generates and sends an **ID54** containing the "Blink/On" instruction for the lamp at station `1107`. |  |  |
| 5 | Wait for the `ShelfMonitor` until Allocation of stock items that can be retrieved. | System internal counters reflect a recovery of available empty pallet. |  |  |
| 6 | Observe the next `ShelfMonitor` cycle. | WMS generates a second **ID54** with the "Off/Normal" code to stop the blinking lamp at `1107`. |  |


### Phase 3: Final Check (🟢 Positive) - Continuation
This phase confirms that once the negative bottleneck is resolved, the system smoothly recovers its data flow to finish the physical retrieval.

| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 7 | ADVANCE transactional tracking by dispatching the transport command to AGC. | System creates unique Work Information key, generates the `ID12` retrieval command mapping unique coordinates for the source bin (Transport='2', Category='1' or '2', Qty=0, Load Size=000), and updates command states. |
| 8 | Physical Pallet arrival at Station `1210`. | System registering the arrival log entries. |  |  |



### Verification SQL

```sql
    -- 1. Verify Supply Request Plan (Step 2)
    SELECT 
      PALLET_ID, 
      BCR_DATA, 
      EMPTY_FLAG, 
      STATION_NO 
    FROM 
      DNPallet 
    WHERE 
      STATION_NO = '1210';
    -- Expected result check: Returns 1 pallet record with item='EMP_PB'.
    
    SELECT 
      CARRY_KEY, 
      SOURCE_STATION_NO, 
      DEST_STATION_NO, 
      CMD_STATUS 
    FROM 
      DNCarryInfo 
    WHERE 
      DEST_STATION_NO = '1210';
    -- Expected result check: Source bin coordinate exists (e.g. 900x bin ID), status=2.

    -- 2. Verify Lamp Station mapping (After step 4)
    SELECT 
      STATION_NO, 
      LAMP_ID, 
      DESCRIPTION 
    FROM 
      DMLocationFullLamp 
    WHERE 
      STATION_NO = '1107';
```

### Post-Conditions

*   The physical target location position inside `DMShelf` updates its inventory operational field setting to `0` (Empty Location).    
*   The `ID54` instruction is logged in the communication history for audit purposes.    
*   Once resources are replenished, the system automatically sends a clearing `ID54` instruction to return the station to a normal state.
*   `DNCarryInfo` are deleted, clean-up processes, and system memory buffers related to the Retrieval Trigger are safely flushed.

## Notes / Defects
```
[1. This Test Case When executing the Positive Scenario, the Negative Scenario can be skipped.]
[2. This operation does not require generating result data for host transmission.]    
[3. The Test data on the TC just an example, follow the actual data when to do it.]                                                                         
```

<hr>
    
<table style="width: 100%; border-collapse: collapse;">
  <tr>
    <td style="text-align: left; width: 50%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      Next →<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1189/TC-PS-002-Palletizing-Start-Screen" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 002 Palletizing Start Screen</a>
    </td>
  </tr>
</table>