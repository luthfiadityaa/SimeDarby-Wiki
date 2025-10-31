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
- Material Code
- Planned Quantity
- Uom
- Delivery Date

Upon receiving new Plan Storage from Host system, WareNavi will insert related planned information to DNSTORAGEPLAN database table.

### DNSTORAGEPLAN (INSERT)

| **Column Name**            | **Description / Notes**                           |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            | 0:Not Started                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | Value from SAP (Delivery Date)                                                       
| **VENDOR_CODE**            | Value from SAP (Vendor Code)
| **VENDOR_NAME**            | Value from SAP (Vendor Name)                                                     
| **COMPANY_CODE**           | Value from SAP (Company Code)                                                      
| **RECEIVE_TICKET_NO**      | Value from SAP (Document Number)                                                      
| **RECEIVE_LINE_NO**        | Value from SAP (Item No / Line No)                                                      
| **RECEIVE_TICKET_DATE**    | Value from SAP (Document Date)                                                     
| **BRANCH_NO**              |                                                         
| **PLAN_AREA_NO**           | Value from SAP (Plant)                                                      
| **PLAN_LOCATION_NO**       | DMWAREHOUSE.Warehose_no                                                      
| **MATERIAL_CODE**          | Value from SAP (Material Code)                                                        
| **PLAN_LOT_NO**            |                                                     
| **NOTE**                   |                                                       
| **PLAN_QTY**               | Planned Quantity                                                     
| **PROCESS_QTY**            |                                            
| **RESULT_QTY**             |                                                  
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            | 0:Not Reported                                                      
| **WORK_DAY**               |                                                       
| **REGIST_KIND**            | 0:File Loading                                                      
| **BCR_DATA**               |                                    
| **STORING_PAIR_KEY**       | 
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       | 0:Successfull
| **TYPE**                   | S: Success
| **MESSAGE_DESC**           |
| **REGIST_DATE**            | SYSTIMESTAMP                                                  
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

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
| **Column Name**            | **Description / Notes**                           |
|----------------------------|---------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               |                                                     
| **VENDOR_CODE**            |     
| **VENDOR_NAME**            |                                                 
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
| **PLAN_QTY**               | Value from screen (Planned Qty)                                                     
| **PROCESS_QTY**            | Value from screen (Storage Qty)                                                       
| **RESULT_QTY**             | Value from screen (Stored Qty)                                                      
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            | 0:Not Reported                                                      
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY                                                      
| **REGIST_KIND**            |                                                       
| **BCR_DATA**               | Value from screen (Pallet #)                                                      
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |
| **REGIST_DATE**            |                                                  
| **REGIST_PNAME**           | 
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### DNPallet (INSERT)
| **Column Name**            | **Description / Notes**                           |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | Only 1 route station (ST1106)                                                       
| **WH_STATION_NO**          | 9002                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                      
| **ALLOCATION_FLAG**        | 1:Allocated                                                      
| **EMPTY_FLAG**             | 0:Normal Pallet                                                       
| **HEIGHT**                 |                                                       
| **WIDTH**                  |                                                       
| **SOFTZONE_ID**            |                                                       
| **BCR_DATA**               | Value from screen (Pallet #)                                                     
| **LAST_STORED_DATE**       | SYSTIMESTAMP                                                       
| **AISLE_DISPERSE_KEY**     |                                                       
| **AISLE_COLLECT_KEY**      |                                                       
| **LEVEL_DISPERSE_KEY**     |                                                       
| **COLLECT_BATCH_NO**       |                                                       
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### DNStock (INSERT)
| **Column Name**            | **Description / Notes**                           |
|----------------------------|-------------------------------------------------------|
| **STOCK_ID**               | DNWORKINFO.STOCK_ID  
| **AREA_NO**                | DNSTORAGEPLAN.AREA_NO
| **LOCATION_NO**            | 99999999999
| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
| **VENDOR_NAME**            | DNSTORAGEPLAN.VENDOR_NAME
| **LOT_NO**                 | 
| **STORAGE_TYPE**           | 2:New 
| **NOTE**                   |
| **STORAGE_DAY**            | DNSTORAGEPLAN.PLAN_DAY
| **STORAGE_DATE**           | DNSTORAGEPLAN.REGIST_DATE
| **NEWEST_STORAGE_DATE**    |
| **RETRIEVAL_DAY**          |
| **INVENTORY_DAY**          |
| **STOCK_QTY**              | 
| **ALLOCATION_QTY**         | 0
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **QTY_CRTN**               | 
| **BATCH_NO**               | Value from screen (Batch #)
| **STOCK_STATUS**           | 
| **QC_DURATION**            | 
| **TEMPERING_FLAG**         | 
| **QC_FLAG**                | 
| **TEMPERING_PERIOD**       | 
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)
| **EXPIRY_DATE**            | 
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### DNWorkInfo (INSERT)
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **JOB_NO**                 | Sequence Object
| **SETTING_UNIT_KEY**       | Sequence Object
| **COLLECT_JOB_NO**         | Sequence Object
| **JOB_TYPE**               | 02:Storage
| **STATUS_FLAG**            | 0:Not Started
| **RFT_STATUS_FLAG**        |
| **HARDWARE_TYPE**          | 3:ASRS
| **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
| **STOCK_ID**               | Sequence Object
| **SYSTEM_CONN_KEY**        |
| **PLAN_DAY**               | DNSTORAGE.PLAN_DAY
| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
| **RECEIVE_TICKET_NO**      | DNSTORAGEPLAN.RECEIVE_TICKET_NO
| **RECEIVE_LINE_NO**        | DNSTORAGEPLAN.RECEIVE_LINE_NO
| **RECEIVE_BRANCH_NO**      |
| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | Value from screen (Batch #)
| **ORDER_NO**               |
| **ORDER_SERIAL_NO**        |
| **PLAN_AREA_NO**           | DNSTORAGEPLAN.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNSTORAGEPLAN.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
| **PLAN_LOT_NO**            | 
| **NOTE**                   |
| **PLAN_QTY**               | DNSTORAGEPLAN.PLAN_QTY
| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY
| **SHORTAGE_QTY**           |
| **RESULT_AREA_NO**         | 9002 (Packaging Material)
| **RESULT_LOCATION_NO**     | DMWAREHOUSE.WAREHOUSE_NO
| **RESULT_NO**              |
| **WORK_NOTE**              |
| **SKIP_CNT**               |
| **WORK_DAY**               | DNSTORAGEPLAN.WORK_DAY
| **USER_ID**                | Login Info
| **TERMINAL_NO**            | Login Terminal
| **WORK_SECOND**            |
| **STORAGE_LOCATION_TO**    |
| **STORAGE_LOCATION_FROM**  |
| **DOCK_NO**                |
| **TRUCK_PLATE_NO**         |
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)
| **NEW_STORING_PAIR_KEY**   |
| **PRIORITY_TYPE**          |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |   
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### DNWorkList (INSERT)
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **CARRY_KEY**              | Sequence Object
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
| **COMPANY_NAME**           | 
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **ORDER_NO**               |
| **PLAN_AREA_NO**           | DNWORKINFO.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNWORKINFO.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
| **JAN**                    |
| **CASE_ITF**               |
| **BUNDLE_ITF**             |
| **ENTERING_QTY**           |
| **BUNDLE_ENTERING_QTY**    |
| **PLAN_LOT_NO**            |
| **STORAGE_TYPE**           | DNSTORAGEPLAN.STORAGE_TYPE
| **STORAGE_DATE**           | DNSTORAGEPLAN.STORAGE_DATE
| **NOTE**                   |
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **WORK_NOTE**              |
| **PRIORITY**               |
| **RETRIEVAL_STATION_NO**   |
| **RETRIEVAL_DETAIL**       |
| **WORK_NO**                |
| **SOURCE_STATION_NO**      |
| **DEST_STATION_NO**        |
| **SCHEDULE_NO**            |
| **SCHEDULE_NO**            |
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            | DNWORKINFO.TERMINAL_NO
| **DOCK_NO**                |
| **TRUCK_PLATE_NO**         |
| **STORING_PAIR_KEY**       | DNWORKINFO.STORING_PAIR_KEY
| **NEW_STORING_PAIR_KEY**   |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |   
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP 
| **LAST_UPDATE_PNAME**      | ClassName

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
