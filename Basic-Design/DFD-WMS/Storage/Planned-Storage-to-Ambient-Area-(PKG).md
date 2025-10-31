[[_TOC_]]

# Planned Storage Setting (PKG) database flow

|Action Name| DNSTORAGEPLAN | DNPALLET | DNSTOCK | DNWORKLIST | DNWORKINFO | DNHOSTSEND |
|-----------|--|--|--|--|--|--|
| Planned Storage from Host | INSERT| | | | | |
| Planned Storage - Set (F2) | UPDATE| INSERT | INSERT | INSERT | INSERT | INSERT |

# Planned Storage from Host
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.storage.plannedstoragepkg.PlannedStoragePkgSCH` &nbsp;</span>

::: mermaid
flowchart LR

subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanPkgData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2
    end

subgraph WareNavi7A
FileExchange[FileExchangeConverter]

FileExchange-insert[("
DNSTORAGEPLAN
")]
end

HostCommExecutor-->FileExchange
FileExchange--INSERT-->FileExchange-insert
:::

## StoragePlanPkgDataLoader
- Document Number
- Company Code
- Vendor
- Vendor Name
- Document Date
- Item No / Line No
- Plant
- Planned Quantity
- Uom
- Delivery Date

Upon receiving new Plan Storage from Host system, WareNavi will insert related planned information to DNSTORAGEPLAN database table.

### DNSTORAGEPLAN (INSERT)

|NO| **Column Name**            | **Description / Notes**                           |
|--|----------------------------|-------------------------------------------------------|
|1 | **PLAN_UKEY**              | Sequence Object                                                       
|2 | **LOAD_UNIT_KEY**          |                                                       
|3 | **FILE_LINE_NO**           |                                                       
|4 | **STATUS_FLAG**            | 0:Not Started                                                       
|5 | **CANCEL_FLAG**            | 0:Normal Data                                                      
|6 | **PLAN_DAY**               |                                                       
|7 | **VENDOR_CODE**            | Value from SAP (Vendor Code)
|8 | **VENDOR_NAME**            | Value from SAP (Vendor Name)                                                     
|9 | **COMPANY_CODE**           | Value from SAP (Company Code)                                                      
|10| **RECEIVE_TICKET_NO**      | Value from SAP (Document Number)                                                      
|11| **RECEIVE_LINE_NO**        | Value from SAP (Item No / Line No)                                                      
|12| **RECEIVE_TICKET_DATE**    | Value from SAP (Document Date)                                                     
|13| **BRANCH_NO**              |                                                         
|14| **PLAN_AREA_NO**           | Value from SAP (Plant)                                                      
|15| **PLAN_LOCATION_NO**       | DMWAREHOUSE.Warehose_no                                                      
|16| **MATERIAL_CODE**          |                                                      
|17| **PLAN_LOT_NO**            |                                                     
|18| **NOTE**                   |                                                       
|19| **PLAN_QTY**               | Planned Quantity                                                     
|20| **PROCESS_QTY**            |                                            
|21| **RESULT_QTY**             |                                                  
|22| **SHORTAGE_QTY**           |                                                       
|23| **REPORT_FLAG**            | 0:Not Reported                                                      
|24| **WORK_DAY**               |                                                       
|25| **REGIST_KIND**            | 0:File Loading                                                      
|26| **BCR_DATA**               |                                    
|27| **STORING_PAIR_KEY**       | 
|28| **MSG_ID**                 |
|29| **MSG_TYPE**               |
|30| **MSG_ID_ORI**             |
|31| **ERROR_INDICATION**       | 0:Successfull
|32| **TYPE**                   | S: Success
|33| **MESSAGE_DESC**           |
|34| **REGIST_DATE**            | Delivery Date                                                   
|35| **REGIST_PNAME**           | ClassName
|36| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|37| **LAST_UPDATE_PNAME**      | ClassName

# Planned Storage - Set (F2)
Planned Storage Setting (PKG) is used to set the information of stock which will be entered into ASRS. After **Set(F2)** all item in input text will be process and the result will be posted back to SAP.
![image.png](/.attachments/image-731303e9-dc65-4f98-ba82-ba6d13a6c58b.png)

::: mermaid
flowchart LR
    input[
        Pallet #<br>
        Document #<br>
        Company Code<br>
        Vendor Code / Vendor Name<br>
        Material Code<br>
        Material Name<br>
        Batch #<br>
        Storage Qty / Planned Qty / Stored Qty<br>
        UOM<br>
        Plant<br>
        Line #<br>
        Document Date<br>
        Delivery Date<br>
    ]

    tableList-insert[("
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
    ")]

    tableList-update[("
        DNSTORAGEPLAN
    ")]

    className[PlannedStoragePkgSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;

:::

# **Validations**
This section explains the validations for the whole proccess Palletize Start
- Material Code exists in **DMMaterialMaster**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

# **Table Value**

### DNSTORAGEPLAN (UPDATE)
|NO| **Column Name**            | **Description / Notes**                           |
|--|----------------------------|-------------------------------------------------------|
|1 | **PLAN_UKEY**              | Sequence Object                                                       
|2 | **LOAD_UNIT_KEY**          |                                                       
|3 | **FILE_LINE_NO**           |                                                       
|4 | **STATUS_FLAG**            | 1:Working                                                       
|5 | **CANCEL_FLAG**            | 0:Normal Data                                                      
|6 | **PLAN_DAY**               |                                                       
|7 | **VENDOR_CODE**            |                                                      
|8 | **COMPANY_CODE**           |                                                       
|9 | **RECEIVE_TICKET_NO**      |                                                       
|10| **RECEIVE_LINE_NO**        |                                                       
|11| **RECEIVE_TICKET_DATE**    |                                                       
|12| **BRANCH_NO**              |                                                         
|13| **PLAN_AREA_NO**           |                                                       
|14| **PLAN_LOCATION_NO**       |                                                       
|15| **MATERIAL_CODE**          |                                                      
|16| **PLAN_LOT_NO**            |                                                     
|17| **NOTE**                   |                                                       
|18| **PLAN_QTY**               | Value from screen (Planned Qty)                                                     
|19| **PROCESS_QTY**            | Value from screen (Storage Qty)                                                       
|20| **RESULT_QTY**             | Value from screen (Stored Qty)                                                      
|21| **SHORTAGE_QTY**           |                                                       
|22| **REPORT_FLAG**            | 0:Not Reported                                                      
|23| **WORK_DAY**               |                                                       
|24| **REGIST_KIND**            |                                                       
|25| **BCR_DATA**               | Value from screen (Pallet #)                                                      
|26| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
|27| **MSG_ID**                 |
|28| **MSG_TYPE**               |
|29| **MSG_ID_ORI**             |
|30| **ERROR_INDICATION**       |
|31| **TYPE**                   |
|32| **MESSAGE_DESC**           |
|33| **REGIST_DATE**            |                                                  
|34| **REGIST_PNAME**           | 
|35| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|36| **LAST_UPDATE_PNAME**      | ClassName

### DNPallet (INSERT)
|NO| **Column Name**            | **Description / Notes**                           |
|--|----------------------------|-------------------------------------------------------|
|1 | **PALLET_ID**              | Sequence Object                                                       
|2 | **CURRENT_STATION_NO**     | Only 1 route station (ST1106)                                                       
|3 | **WH_STATION_NO**          | 9002                                                      
|4 | **STATUS_FLAG**            | 1:Reserved for Storage                                                      
|5 | **ALLOCATION_FLAG**        | 1:Allocated                                                      
|6 | **EMPTY_FLAG**             | 0:Normal Pallet                                                       
|7 | **HEIGHT**                 |                                                       
|8 | **WIDTH**                  |                                                       
|9 | **SOFTZONE_ID**            |                                                       
|10| **BCR_DATA**               | Value from screen (Pallet #)                                                     
|11| **LAST_STORED_DATE**       |                                                       
|12| **AISLE_DISPERSE_KEY**     |                                                       
|13| **AISLE_COLLECT_KEY**      |                                                       
|14| **LEVEL_DISPERSE_KEY**     |                                                       
|15| **COLLECT_BATCH_NO**       |                                                       
|16| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|17| **REGIST_PNAME**           | ClassName
|18| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|19| **LAST_UPDATE_PNAME**      | ClassName

### DNStock (INSERT)
|NO| **Column Name**            | **Description / Notes**                           |
|--|----------------------------|-------------------------------------------------------|
|1 | **STOCK_ID**               | WORKINFO.STOCK_ID  
|2 | **AREA_NO**                | 9001
|3 | **LOCATION_NO**            | 99999999999
|4 | **MATERIAL CODE**          | 
|5 | **LOT_NO**                 | 
|6 | **STORAGE_TYPE**           | 2:New 
|7 | **NOTE**                   |
|8 | **STORAGE_DAY**            |
|9 | **STORAGE_DATE**           |
|10| **NEWEST_STORAGE_DATE**    |
|11| **RETRIEVAL_DAY**          |
|12| **INVENTORY_DAY**          |
|13| **STOCK_QTY**              | 0
|14| **ALLOCATION_QTY**         | 0
|15| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
|16| **PALLET_ID**              | Value from screen (Pallet #)
|17| **QTY_CRTN**               | 
|18| **BATCH_NO**               | Value from screen (Batch #)
|19| **STOCK_STATUS**           | UU: Unrestricted Used (Target for normal retrieval)
|20| **QC_DURATION**            | 
|21| **TEMPERING_FLAG**         | 
|22| **QC_FLAG**                | 
|23| **TEMPERING_PERIOD**       | 
|24| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)
|25| **EXPIRY_DATE**            | 
|26| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|27| **REGIST_PNAME**           | ClassName
|28| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|29| **LAST_UPDATE_PNAME**      | ClassName

### DNWorkInfo (INSERT)
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **JOB_NO**                 | Sequence Object
|2 | **SETTING_UNIT_KEY**       | Sequence Object
|3 | **COLLECT_JOB_NO**         | Sequence Object
|4 | **JOB_TYPE**               | 02:Storage
|5 | **STATUS_FLAG**            | 0:Not Started
|6 | **RFT_STATUS_FLAG**        |
|7 | **HARDWARE_TYPE**          | 3:ASRS
|8 | **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
|9 | **STOCK_ID**               | Sequence Object
|10| **SYSTEM_CONN_KEY**        |
|11| **PLAN_DAY**               | DMWARENAVISYSTEM.WORK_DAY
|12| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
|13| **RECEIVE_TICKET_NO**      | DNSTORAGEPLAN.RECEIVE_TICKET_NO
|14| **RECEIVE_LINE_NO**        | DNSTORAGEPLAN.RECEIVE_LINE_NO
|15| **RECEIVE_BRANCH_NO**      |
|16| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
|17| **SHIP_TICKET_NO**         |
|18| **SHIP_LINE_NO**           |
|19| **SHIP_BRANCH_NO**         |
|20| **BATCH_NO**               | Value from screen (Batch #)
|21| **ORDER_NO**               |
|22| **ORDER_SERIAL_NO**        |
|23| **PLAN_AREA_NO**           | DNSTORAGEPLAN.PLAN_AREA_NO
|24| **PLAN_LOCATION_NO**       | DNSTORAGEPLAN.PLAN_LOCATION_NO
|25| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
|26| **PLAN_LOT_NO**            | 
|27| **NOTE**                   |
|28| **PLAN_QTY**               | DNSTORAGEPLAN.PLAN_QTY
|29| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY
|30| **SHORTAGE_QTY**           |
|31| **RESULT_AREA_NO**         |
|32| **RESULT_LOCATION_NO**     |
|33| **RESULT_NO**              |
|34| **WORK_NOTE**              |
|35| **SKIP_CNT**               |
|36| **WORK_DAY**               |
|37| **USER_ID**                | Login Info
|38| **TERMINAL_NO**            | Login Terminal
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
|53| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|54| **REGIST_PNAME**           | ClassName
|55| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|56| **LAST_UPDATE_PNAME**      | ClassName

### DNWorkList (INSERT)
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **JOB_NO**                 | DNWORKINFO.JOB_NO
|2 | **CARRY_KEY**              |
|3 | **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
|4 | **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
|5 | **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
|6 | **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
|7 | **STOCK_ID**               | DNWORKINFO.STOCK_ID
|8 | **PALLET_ID**              | DNPALLET.PALLET_ID
|9 | **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
|10| **COMPANY_CODE**           |
|11| **COMPANY_NAME**           |
|12| **SHIP_TICKET_NO**         |
|13| **SHIP_LINE_NO**           |
|14| **SHIP_BRANCH_NO**         |
|15| **BATCH_NO**               | DNWORKINFO.BATCH_NO
|16| **ORDER_NO**               |
|17| **PLAN_AREA_NO**           |
|18| **PLAN_LOCATION_NO**       |
|19| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
|20| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
|21| **JAN**                    |
|22| **CASE_ITF**               |
|23| **BUNDLE_ITF**             |
|24| **ENTERING_QTY**           |
|25| **BUNDLE_ENTERING_QTY**    |
|26| **PLAN_LOT_NO**            |
|27| **STORAGE_TYPE**           | DNWORKINFO.STORAGE_TYPE
|28| **STORAGE_DATE**           | DNSTORAGEPLAN.PLAN_DAY
|29| **NOTE**                   |
|30| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
|31| **STOCK_QTY**              | 0
|32| **ALLOCATION_QTY**         | 0
|33| **WORK_NOTE**              |
|34| **PRIORITY**               |
|35| **RETRIEVAL_STATION_NO**   |
|36| **RETRIEVAL_DETAIL**       |
|37| **WORK_NO**                |
|38| **SOURCE_STATION_NO**      |
|39| **DEST_STATION_NO**        |
|40| **SCHEDULE_NO**            |
|41| **SCHEDULE_NO**            |
|42| **USER_ID**                | DNWORKINFO.USER_ID
|43| **USER_NAME**              | DCUSER.USER_NAME
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

### DNHostSend (INSERT)
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **WORK_DAY**               | 
|2 | **JOB_NO**                 | DNWORKINFO.JOB_NO
|3 | **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
|4 | **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
|5 | **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
|6 | **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
|7 | **HARDWARE_TYPE**          |
|8 | **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
|9 | **STOCK_ID**               | DNWORKINFO.STOCK_ID
|10| **SYSTEM_CONN_KEY**        | 
|11| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
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
|22| **BATCH_NO**               | DNWORKINFO.BATCH_NO
|23| **ORDER_NO**               |
|24| **PLAN_AREA_NO**           |
|25| **PLAN_LOCATION_NO**       |
|26| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
|27| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
|28| **JAN**                    |
|29| **CASE_ITF**               |
|30| **BUNDLE_ITF**             |
|31| **ENTERING_QTY**           |
|32| **BUNDLE_ENTERING_QTY**    |
|33| **PLAN_LOT_NO**            |
|34| **NOTE**                   |
|35| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
|36| **RESULT_QTY**             |
|37| **SHORTAGE_QTY**           |
|38| **RESULT_AREA_NO**         |
|39| **RESULT_LOCATION_NO**     |
|40| **RESULT_LOT_NO**          |
|41| **WORK_NOTE**              |
|42| **REPORT_FLAG**            | DNSTORAGEPLAN.REPORT_FLAG
|43| **USER_ID**                | DNWORKINFO.USER_ID
|44| **USER_NAME**              | DCUSER.USER_NAME
|45| **TERMINAL_NO**            |
|46| **WORK_SECOND**            |
|47| **MSG_ID**                 |
|48| **MSG_TYPE**               |
|49| **MSG_ID_ORI**             |
|50| **ERROR_INDICATION**       |
|51| **TYPE**                   |
|52| **MESSAGE_DESC**           |
|53| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|54| **REGIST_PNAME**           | ClassName
|55| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|56| **LAST_UPDATE_PNAME**      | ClassName

# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)
