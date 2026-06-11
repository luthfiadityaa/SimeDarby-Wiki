[[_TOC_]]
[[_TOSP_]]
#TC-PS-002 — Palletize Start - Set(F2)

| **Field** | **Value** |
| --- | --- |
| **Test Type** | Functional Test |
| **DFD Reference** | [Palletizing Start](/Basic-Design/DFD-WMS/Storage/PALLETIZE-OPERATION-\(HP\)/0.-Palletize-Start-[SCREEN-ONLY]) |
| **Screen** | `PalletizingSettingSCH` |
| **Job Type** | 02:Storage |
| **Destination** | Storage Location: `FGW1 (Ambient/9002)` or `FGW2 (Tempering/9001)` |
| **AGC Route** | `N/A` (Pre-Storage Planning) |
| **Tester** | _____________ |
| **Test Date** | _____________ |
| **Overall Status** | [ ] Pass    [ ] Fail |

##Objective
To verify that the `PalletizingSettingSCH` screen successfully validates user inputs and inserts a valid record into `DNReceivingPlan`. The process must ensure all business validation rules **(such as qty limits, tempering vs. expiry dates, and station statuses)** are met before setting the stock information for automatic robot palletization.

##Pre-Conditions
*   AGC is `ONLINE`.    
*   Daily cleanup is **not** processing.    
*   The selected Station Number is available (`DMStation.STATUS.NORMAL` and `DMStation.STATUS_FLAG.ACTIVE`).    
*   The selected Station Number is **not** suspended (`DMStation.SUSPEND.OFF`).    
*   The selected Station Number does **not** have any existing work in progress.    
*   The Material Code used for testing exists in **DMITEM** and is of type must **ZFNP**.
    
##Test Data
| **Key** | **Value** |
| --- | --- |
| Station | 1101 (or any available valid station) |
| Material Code | Valid `ZFNP` type item code |
| Planned Carton Qty | 1000 |
| Qty kg/crtn | 10 |
| Qty crtn/PL | 50 |
| Tempering Period | 5 |
| Expiry Days | 30 |
| Storage Location | FGW1 (Ambient/9002) |
| Storage Date/Time | [Current System Date/Time] |

##Test Steps

### Phase 1: When input passed (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 1 | Open `PalletizingSettingSCH`, select an available Station (e.g., `1111`). | Station is selectable and displays no existing WIP. |  |  |
| 2 | Enter valid values for Material Code (ZFNP type), Batch #, and Storage Location. | Inputs are accepted. |  |  |

### Phase 2: When validation not passed (🔴 Negative)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 3 | Input quantity and period values ensuring:<br>  <br>- Planned Carton Qty (10) <= Qty kg/crtn (10)<br>  <br>- Planned Carton Qty (1000) <= Qty crtn/PL (50)<br>  <br>- Expiry Days (30) <= Tempering Period (5)<br> | Validation logic unpasses with error warnings. |  |  |
| 4 | Set Storage Date/Time not to the current day. | Date is rejected. with error message |  |  |

### Phase 3: When validation passed (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 5 | Input quantity and period values ensuring:<br>  <br>- Planned Carton Qty (1000) >= Qty kg/crtn (10)<br>  <br>- Planned Carton Qty (1000) >= Qty crtn/PL (50)<br>  <br>- Expiry Days (30) >= Tempering Period (5)<br> |  Validation logic passes without error warnings. |  |  |
| 6 | Set Storage Date/Time to the current day. | Date is accepted. |  |  |

### Phase 4: Final Check (🟢 Positive)
| **#** | **Step** | **Expected Result** | **Actual Result** | **Status (Pass / Fail)** |
| --- | --- | --- | --- | --- |
| 7 | Press **Set (F2)**. | 1 record inserted into `DNRECEIVINGPLAN` (`STATUS_FLAG = 1`, `CANCEL_FLAG = 0`, `REGIST_KIND = 1`). |  |  |

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
    WHERE batch_station_no = '1101' 
    ORDER BY REGIST_DATE DESC 
    FETCH FIRST 1 ROWS ONLY;
    
    -- Expected result check:
    -- STATUS_FLAG = '1'
    -- CANCEL_FLAG = '0'
    -- REPORT_FLAG = '0'
```

##Post-Conditions
*   The plan for palletizing information is successfully recorded in the database.    
*   The system is queued to proceed to the Palletization Storage Flow (110x -> 111x -> (7101-7110 -> (9101-9110) -> (9100/9200).    
*   The transaction is ready to be reported to SAP as a Production Storage Result once the process completes.
    
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
      ← Previous<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1167/TC-PS-001-Empty-Pallet-Supply-via-Standard-Retrieval-Cycle-(ST1210-Triggered)" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 001 Empty Pallet Supply via Standard Retrieval Cycle (ST1210 Triggered)</a>
    </td>
    <td style="text-align: left; width: 0%; border: none;">
      &nbsp;
    </td>
    <td style="text-align: right; width: 50%; border: none;">
      Next →<br>
      <a href="https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1158/TC-PS-003-No-Operations-(Empty-Pallet-Supply)-1210-to-110x-Barcode-No-Read-Rejection" style="font-weight: bold; font-size: 1.1em;">📝 TC PS 003 No Operations (Empty Pallet Supply) - 1210 to 110x & Barcode No Read Rejection</a>
    </td>
  </tr>
</table>