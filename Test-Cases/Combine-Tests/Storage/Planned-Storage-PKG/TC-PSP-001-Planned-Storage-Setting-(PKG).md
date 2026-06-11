[[_TOC_]]
[[_TOSP_]]

# TC-PSP-001 — Planned Storage Setting (PKG)

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional Test |
| **DFD Reference** | [Planned Storage Setting (PKG)](/Basic-Design/DFD-WMS/Storage/[SCREEN-ONLY]-Planned-Storage-Packaging-Material) |
| **Screen** | `PlannedStoragePkgSCH` |
| **Job Type** | 02:Storage |
| **AGC Route** | `N/A` (Pre-Storage Planning) |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

# Objective
To verify that the `PlannedStoragePkgSCH` screen successfully validates user inputs and inserts a valid record into `DNReceivingPlan`. The process must ensure all business validation rules and creates associated transactional entries in `DNSTORAGEPLAN` upon executing **Set(F2)**.

# Pre-Conditions
*   AGC is `ONLINE`.    
*   Daily cleanup is **not** processing.    
*   The selected Station Number is available (`DMStation.STATUS.NORMAL` and `DMStation.STATUS_FLAG.ACTIVE`).    
*   The selected Station Number is **not** suspended (`DMStation.SUSPEND.OFF`).    
*   The selected Station Number does **not** have any existing work in progress.    
*   The Material Code used for testing exists in **DMITEM** and is of type must **ZPCK**.
*   A valid host record exists inside `DNRECEIVINGPLAN` with an unstarted or working tracking status. The host record for receiving [material master data](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1102/1.-Material-Master-Data) should be processed first.
    
# Test Data 
| **Key** | **Value** |
| --- | --- |
| Document No / Receive Ticket No | RT-PKG-20260604 |
| Receive Line No | 0001 |
| Material Code (`ITEM_TYPE=ZPCK`) | PM-ZPCK-009 |
| Planned Qty | 1000 |
| Storage Qty (Input) | 250 |
| Batch # | 0000000001 |
| BCR Data | 0000000001 |

##Test Steps

### Phase 1: When input passed (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 1 | Open `PlannedStoragePkgSCH`, Enter valid values for Material Code (ZPCK type), Batch #, and Storage Qty. | Input accetable. |  |  |

### Phase 2: When validation not passed (🔴 Negative)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 3 | Input quantity and period values ensuring:<br>  <br> - Storage Qty <= 0. <br><br> - (Storage Qty + Stored Qty) > Planned Qty.<br> | Validation logic passes without error warnings. |  |  |

### Phase 3: When validation passed (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 5 | Input quantity and period values ensuring:<br>  <br> - Storage Qty > 0. <br><br> - (Storage Qty + Stored Qty) <= Planned Qty.<br> | Validation logic unpasses with error warnings. |  |  |

### Phase 4: Final Check (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 7 | Press **Set (F2)**. | 1 record inserted into `DNRECEIVINGPLAN` (`STATUS_FLAG = 1`, `CANCEL_FLAG = 0`, `REGIST_KIND = 1`), 1 record inserted into `DNStoragePlan` associated with `Pallet #` and `Batch #`. |  |  |

### Phase 5: Finish & Completion Processing Operations

| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 8 | Initiate a **Finish** request scenario where the current accumulated **Stored Qty** is not equal to the original **Planned Qty** limit. | Screen detects the discrepancy and flags the tracking record indicating that a shortage occurred. |  |  |
| 9 | Complete execution validation requirements and commit the transactional **Finish** request. | - `DNRECEIVINGPLAN` record transitions its `STATUS_FLAG` to `4` (Completion).  <br>  <br>- Final updates log the active class identifier name and system timestamps.  <br> |  |  |

##Verification SQL
```sql
    -- 1. Verify the successful insertion and correct column mapping in DNReceivingPlan
    SELECT 
        PLAN_UKEY, 
        STATUS_FLAG, 
        CANCEL_FLAG, 
        BATCH_STATION_NO, 
        ITEM_CODE, 
        PLAN_QTY, 
        SAP_TO_LOCATION, 
        REPORT_FLAG
    FROM dnreceivingplan 
    WHERE receive_ticket_no = 'RT-PKG-20260604'
    AND receive_line_no = '0001' 
    ORDER BY REGIST_DATE DESC 
    FETCH FIRST 1 ROWS ONLY;
    
    -- Expected result check:
    -- STATUS_FLAG = '1'
    -- CANCEL_FLAG = '0'
    -- REPORT_FLAG = '0'

     -- 2. Verify the successful insertion and correct column mapping in DNStoragePlan
    SELECT 
        PLAN_UKEY, 
        STATUS_FLAG, 
        CANCEL_FLAG, 
        BATCH_STATION_NO, 
        ITEM_CODE, 
        PLAN_QTY, 
        REPORT_FLAG
    FROM dnstorageplan 
    WHERE bcr_data = '00000000001' 
    ORDER BY REGIST_DATE DESC 
    FETCH FIRST 1 ROWS ONLY;
    
    -- Expected result check:
    -- STATUS_FLAG = '0'
    -- CANCEL_FLAG = '0'
    -- REPORT_FLAG = '0'

    -- 4. Verify Plan Close Out & Final Completion Status Flags (After Step 7 / Step 17)
    SELECT 
       STATUS_FLAG, 
       PROCESS_QTY, 
       LAST_UPDATE_PNAME 
    FROM 
       dnreceivingplan 
    WHERE 
       receive_ticket_no = 'RT-PKG-20260604' 
    AND receive_line_no = '0001';
```

##Post-Conditions
*   The plan for planned storage PKG information is successfully recorded in the database.    
*   The system is queued to proceed to the Palletization Storage Flow (1106/1301/1302 -> (7207-7214 -> (9007-9014) -> (9200).    
*   The transaction is ready to be reported to SAP as a Storage Result once the process completes.
    
##Notes / Defects
```
[1. This Test Case when executing the Positive Scenario, the Negative Scenario can be skipped.]
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
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1199/TC-PSP-002-HP-Station-1106-1301-1302-to-ASRS-Rack-and-Host-Reporting-Flow" style="font-weight: bold; font-size: 1.1em;">📝 TC PSP 002 HP Station 1106, 1301, 1302 to ASRS Rack, and Host Reporting Flow</a>
    </td>
  </tr>
</table>