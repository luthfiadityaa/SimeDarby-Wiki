[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletizing Setting Set(F2)**</p>

Palletizing Start is used to set the information of stock which will be palletized by robot automatically. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as Production Storage Result.

![image.png](/.attachments/image-0617abe5-5a77-4ffa-a06f-55d39356473b.png)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Station<br>
        Current Status<br>
        Material Code<br>
        Material Name<br>
        Material Type<br>
        Batch #<br>
        Planned Craton Qty<br>
        Qty kg/crtn<br>
        Qty crtn/PL<br>
        UOM<br>
        Tempering Period<br>
        Storage Location<br>
        Expiry Days<br>
    ]

    tableList-insert[("
        DNSTORAGEPLAN<br>
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
    ")]

    className[PalletizingSettingSCH]

    input --> className --> tableList-insert

    classDef leftAlign text-align:left;
    class input leftAlign;

:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the `DNSTORAGEPLAN` data
- Material Code exists in **DMMaterialMaster**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Process</span>

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              |                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            |                                                       
| **CANCEL_FLAG**            |                                                       
| **PLAN_DAY**               |                                                       
| **VENDOR_CODE**            |                                                       
| **COMPANY_CODE**           |                                                       
| **RECEIVE_TICKET_NO**      |                                                       
| **RECEIVE_LINE_NO**        |                                                       
| **RECEIVE_TICKET_DATE**    |                                                       
| **BRANCH_NO**              |                                                       
| **PLAN_AREA_NO**           |                                                       
| **PLAN_LOCATION_NO**       |                                                       
| **MATERIAL_CODE**          |                                                       
| **PLAN_LOT_NO**            |                                                       
| **NOTE**                   |                                                       
| **PLAN_QTY**               |                                                       
| **PROCESS_QTY**            |                                                       
| **RESULT_QTY**             |                                                       
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            |                                                       
| **WORK_DAY**               |                                                       
| **REGIST_KIND**            |                                                       
| **BCR_DATA**               |                                                       
| **STORING_PAIR_KEY**       |   
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              |                                                       
| **CURRENT_STATION_NO**     |                                                       
| **WH_STATION_NO**          |                                                       
| **STATUS_FLAG**            |                                                       
| **ALLOCATION_FLAG**        |                                                       
| **EMPTY_FLAG**             |                                                       
| **HEIGHT**                 |                                                       
| **WIDTH**                  |                                                       
| **SOFTZONE_ID**            |                                                       
| **BCR_DATA**               |                                                       
| **LAST_STORED_DATE**       |                                                       
| **AISLE_DISPERSE_KEY**     |                                                       
| **AISLE_COLLECT_KEY**      |                                                       
| **LEVEL_DISPERSE_KEY**     |                                                       
| **COLLECT_BATCH_NO**       |                                                       
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNSTOCK</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **STOCK_ID**               |   
| **AREA_NO**                |
| **LOCATION_NO**            |
| **LOT_NO**                 | 
| **STORAGE_TYPE**           |
| **NOTE**                   |
| **STORAGE_DAY**            |
| **STORAGE_DATE**           |
| **NEWEST_STORAGE_DATE**    |
| **RETRIEVAL_DAY**          |
| **INVENTORY_DAY**          |
| **STOCK_QTY**              |
| **ALLOCATION_QTY**         |
| **PLAN_QTY**               |
| **PALLET_ID**              |
| **BATCH_NO**               |
| **STOCK_STATUS**           |
| **QC_DURATION**            |
| **TEMPERING_FLAG**         |
| **QC_FLAG**                |
| **TEMPERING_PERIOD**       |
| **STORING_PAIR_KEY**       |
| **EXPIRY_DATE**            |
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1|| **JOB_NO**                 |
|--|| **SETTING_UNIT_KEY**       |
|--|| **COLLECT_JOB_NO**         |
|--|| **JOB_TYPE**               |
|--|| **STATUS_FLAG**            |
|--|| **RFT_STATUS_FLAG**        |
|--|| **HARDWARE_TYPE**          |
|--|| **PLAN_UKEY**              |
|--|| **STOCK_ID**               |
|--|| **SYSTEM_CONN_KEY**        |
|--|| **PLAN_DAY**               |
|--|| **VENDOR_CODE**            |
| **RECEIVE_TICKET_NO**      |
| **RECEIVE_LINE_NO**        |
| **RECEIVE_BRANCH_NO**      |
| **COMPANY_CODE**           |
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               |
| **ORDER_NO**               |
| **ORDER_SERIAL_NO**        |
| **PLAN_AREA_NO**           |
| **PLAN_LOCATION_NO**       |
| **MATERIAL_CODE**          |
| **PLAN_LOT_NO**            |
| **NOTE**                   |
| **PLAN_QTY**               |
| **RESULT_QTY**             |
| **SHORTAGE_QTY**           |
| **RESULT_AREA_NO**         |
| **RESULT_LOCATION_NO**     |
| **RESULT_NO**              |
| **WORK_NOTE**              |
| **SKIP_CNT**               |
| **WORK_DAY**               |
| **USER_ID**                |
| **TERMINAL_NO**            |
| **WORK_SECOND**            |
| **STORAGE_LOCATION_TO**    |
| **STORAGE_LOCATION_FROM**  |
| **DOCK_NO**                |
| **TRUCK_PLATE_NO**         |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |   
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNWORKLIST</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              |   
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNHOSTSEND</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              |   
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |