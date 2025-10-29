[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletizing to Tempering/Ambient database flow</span>

| Action Name                        | DNSTOCK | DNPALLET | DNWORKINFO | DNWORKLIST | DNHOSTSEND | DNSTORAGEPLAN | 
|------------------------------------|---------|----------|------------|------------|------------|---------------|
| [Palletizing Setting - Set(F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Setting&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Epalletizing-setting-set(f2)%3C/span%3E)      |  INSERT |  INSERT  |   INSERT   |   INSERT   |   INSERT   |     INSERT    |

#<span style="color:skyblue; font-weight:bold">Palletizing Setting - Set(F2)</span>

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
| **PLAN_UKEY**              | Sequence Object                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | Value from screen (Storage Date/Time)                                                      
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
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)                                                       
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
|1 | **JOB_NO**                 |
|2 | **SETTING_UNIT_KEY**       |
|3 | **COLLECT_JOB_NO**         |
|4 | **JOB_TYPE**               |
|5 | **STATUS_FLAG**            |
|6 | **RFT_STATUS_FLAG**        |
|7 | **HARDWARE_TYPE**          |
|8 | **PLAN_UKEY**              |
|9 | **STOCK_ID**               |
|10| **SYSTEM_CONN_KEY**        |
|11| **PLAN_DAY**               |
|12| **VENDOR_CODE**            |
|13| **RECEIVE_TICKET_NO**      |
|14| **RECEIVE_LINE_NO**        |
|15| **RECEIVE_BRANCH_NO**      |
|16| **COMPANY_CODE**           |
|17| **SHIP_TICKET_NO**         |
|18| **SHIP_LINE_NO**           |
|19| **SHIP_BRANCH_NO**         |
|20| **BATCH_NO**               |
|21| **ORDER_NO**               |
|22| **ORDER_SERIAL_NO**        |
|23| **PLAN_AREA_NO**           |
|24| **PLAN_LOCATION_NO**       |
|25| **MATERIAL_CODE**          |
|26| **PLAN_LOT_NO**            |
|27| **NOTE**                   |
|28| **PLAN_QTY**               |
|29| **RESULT_QTY**             |
|30| **SHORTAGE_QTY**           |
|31| **RESULT_AREA_NO**         |
|32| **RESULT_LOCATION_NO**     |
|33| **RESULT_NO**              |
|34| **WORK_NOTE**              |
|35| **SKIP_CNT**               |
|36| **WORK_DAY**               |
|37| **USER_ID**                |
|38| **TERMINAL_NO**            |
|39| **WORK_SECOND**            |
|40| **STORAGE_LOCATION_TO**    |
|41| **STORAGE_LOCATION_FROM**  |
|42| **DOCK_NO**                |
|43| **TRUCK_PLATE_NO**         |
|44| **STORING_PAIR_KEY**       |
|45| **NEW_STORING_PAIR_KEY**   |
|46| **PRIORITY_TYPE**          |
|47| **MSG_ID**                 |
|48| **MSG_TYPE**               |
|49| **MSG_ID_ORI**             |
|50| **ERROR_INDICATION**       |
|51| **TYPE**                   |
|52| **MESSAGE_DESC**           |   
|53| **REGIST_DATE**            |                                                    
|54| **REGIST_PNAME**           |
|55| **LAST_UPDATE_DATE**       |
|56| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNWORKLIST</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **JOB_NO**                 |
|2 | **CARRY_KEY**              |
|3 | **SETTING_UNIT_KEY**       |
|4 | **COLLECT_JOB_NO**         |
|5 | **JOB_TYPE**               |
|6 | **PLAN_UKEY**              |
|7 | **STOCK_ID**               |
|8 | **PALLET_ID**              |
|9 | **PLAN_DAY**               |
|10| **COMPANY_CODE**           |
|11| **COMPANY_NAME**           |
|12| **SHIP_TICKET_NO**         |
|13| **SHIP_LINE_NO**           |
|14| **SHIP_BRANCH_NO**         |
|15| **BATCH_NO**               |
|16| **ORDER_NO**               |
|17| **PLAN_AREA_NO**           |
|18| **PLAN_LOCATION_NO**       |
|19| **MATERIAL_CODE**          |
|20| **MATERIAL_NAME**          |
|21| **JAN**                    |
|22| **CASE_ITF**               |
|23| **BUNDLE_ITF**             |
|24| **ENTERING_QTY**           |
|25| **BUNDLE_ENTERING_QTY**    |
|26| **PLAN_LOT_NO**            |
|27| **STORAGE_TYPE**           |
|28| **STORAGE_DATE**           |
|29| **NOTE**                   |
|30| **PLAN_QTY**               |
|31| **STOCK_QTY**              |
|32| **ALLOCATION_QTY**         |
|33| **WORK_NOTE**              |
|34| **PRIORITY**               |
|35| **RETRIEVAL_STATION_NO**   |
|36| **RETRIEVAL_DETAIL**       |
|37| **WORK_NO**                |
|38| **SOURCE_STATION_NO**      |
|39| **DEST_STATION_NO**        |
|40| **SCHEDULE_NO**            |
|41| **SCHEDULE_NO**            |
|42| **USER_ID**                |
|43| **USER_NAME**              |
|44| **TERMINAL_NO**            |
|45| **DOCK_NO**                |
|46| **TRUCK_PLATE_NO**         |
|47| **STORING_PAIR_KEY**       |
|48| **NEW_STORING_PAIR_KEY**   |
|49| **MSG_ID**                 |
|50| **MSG_TYPE**               |
|51| **MSG_ID_ORI**             |
|52| **ERROR_INDICATION**       |
|53| **TYPE**                   |
|54| **MESSAGE_DESC**           |   
|55| **REGIST_DATE**            |                                                    
|56| **REGIST_PNAME**           |
|57| **LAST_UPDATE_DATE**       |
|58| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNHOSTSEND</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **WORK_DAY**               |   
|2 | **JOB_NO**                 |
|3 | **SETTING_UNIT_KEY**       |
|4 | **COLLECT_JOB_NO**         |
|5 | **JOB_TYPE**               |
|6 | **STATUS_FLAG**            |
|7 | **HARDWARE_TYPE**          |
|8 | **PLAN_UKEY**              |
|9 | **STOCK_ID**               |
|10| **SYSTEM_CONN_KEY**        |
|11| **PLAN_DAY**               |
|12| **VENDOR_CODE**            |
|13| **VENDOR_NAME**            |
|14| **RECEIVE_TICKET_NO**      |
|15| **RECEIVE_LINE_NO**        |
|16| **RECEIVE_BRANCH_NO**      |
|17| **COMPANY_CODE**           |
|18| **COMPANY_NAME**           |
|19| **SHIP_TICKET_NO**         |
|20| **SHIP_LINE_NO**           |
|21| **SHIP_BRANCH_NO**         |
|22| **BATCH_NO**               |
|23| **ORDER_NO**               |
|24| **PLAN_AREA_NO**           |
|25| **PLAN_LOCATION_NO**       |
|26| **MATERIAL_CODE**          |
|27| **MATERIAL_NAME**          |
|28| **JAN**                    |
|29| **CASE_ITF**               |
|30| **BUNDLE_ITF**             |
|31| **ENTERING_QTY**           |
|32| **BUNDLE_ENTERING_QTY**    |
|33| **PLAN_LOT_NO**            |
|34| **NOTE**                   |
|35| **PLAN_QTY**               |
|36| **RESULT_QTY**             |
|37| **SHORTAGE_QTY**           |
|38| **RESULT_AREA_NO**         |
|39| **RESULT_LOCATION_NO**     |
|40| **RESULT_LOT_NO**          |
|41| **WORK_NOTE**              |
|42| **REPORT_FLAG**            |
|43| **USER_ID**                |
|44| **USER_NAME**              |
|45| **TERMINAL_NO**            |
|46| **WORK_SECOND**            |
|47| **MSG_ID**                 |
|48| **MSG_TYPE**               |
|49| **MSG_ID_ORI**             |
|50| **ERROR_INDICATION**       |
|51| **TYPE**                   |
|52| **MESSAGE_DESC**           |
|53| **REGIST_DATE**            |                                                    
|54| **REGIST_PNAME**           |
|55| **LAST_UPDATE_DATE**       |
|56| **LAST_UPDATE_PNAME**      |