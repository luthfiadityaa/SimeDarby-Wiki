[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Planned Storage Setting (PKG) database flow</span>

|Action Name| DNSTORAGEPLAN | DNPALLET | DNWORKINFO | DNWORKLIST | DNCARRYINFO | DNSTOCK | DNHOSTSEND |
|-----------|--|--|--|--|--|--|--|
| [Planned Storage from Host](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20to%20Ambient%20Area%20(PKG)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage-from-host%3C/span%3E) | INSERT| | | | | |
| [Planned Storage - Set (F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/Planned-Storage-to-Ambient-Area-(PKG)?_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage---set-(f2)%3C/span%3E) | UPDATE| INSERT | INSERT | INSERT | INSERT | INSERT | INSERT |


#<span style="color:skyblue; font-weight:bold">Planned Storage from Host</span>

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

##<span style="color:skyblue; font-weight:bold">StoragePlanPkgDataLoader</span>
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

### <span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN (INSERT)</span>

| **Column Name**            | **Description / Notes**                               |
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

#<span style="color:skyblue; font-weight:bold">Planned Storage - Set (F2)</span>
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

# <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Storage Packaging Material
- AGC is online.  <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Material Code exists in **DMMaterialMaster**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.

# <span style="color:skyblue; font-weight:bold">Storage Process flow</span>
This section explains the Storage flow after the operation of Planned Storage Packaging Material is Start


# <span style="color:skyblue; font-weight:bold">Table Value</span>

### <span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN (UPDATE)</span>
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

### <span style="color:skyblue; font-weight:bold">DNPallet (INSERT)</span>
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

### <span style="color:skyblue; font-weight:bold">DNWorkInfo (INSERT)</span>
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
| **SYSTEM_CONN_KEY**        | Sequence Object
| **PLAN_DAY**               | DNSTORAGE.PLAN_DAY
| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
| **VENDOR_NAME**            | DNSTORAGEPLAN.VENDOR_NAME
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

### <span style="color:skyblue; font-weight:bold">DNWorkList (INSERT)</span>
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
| **WORK_NO**                | DNWORKINFO.JOB_NO
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

### <span style="color:skyblue; font-weight:bold">DNCarryInfo (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **CARRY_KEY**              | DNWORKINFO.SYSTEM_CONN_KEY
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **WORK_TYPE**              | 02:Storage
| **GROUP_NO**               |
| **GROUP_SEQNO**            |
| **CMD_STATUS**             | 1:Started 
| **NEXT_RESERVE_FLAG**      |
| **PRIORITY**               | 2:Normal
| **RESTORING_FLAG**         | 0:Not Restore to Original Location
| **CARRY_FLAG**             | 1:Storage
| **RETRIEVAL_STATION_NO**   | 
| **RETRIEVAL_DETAIL**       |
| **WORK_NO**                | DNWORKINFO.JOB_NO
| **SOURCE_STATION_NO**      | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**        | DNWORKINFO.PLAN_AREA_NO
| **ARRIVAL_DATE**           |
| **CONTROLINFO**            |
| **CANCEL_REQUEST**         | 0:Not Requested
| **CANCEL_REQUEST_DATE**    |
| **SCHEDULE_NO**            | Sequence Object
| **AISLE_STATION_NO**       |
| **END_STATION_NO**         | DNWORKINFO.PLAN_AREA_NO
| **RESERVE_SHELF_NO**       |
| **ERROR_CODE**             |
| **MAINTENANCE_TERMINAL**   |
| **REJECT_FACTOR**          |
| **AGC_DATE**               |
| **WAIT_REASON**            |
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNStock (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
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

### <span style="color:skyblue; font-weight:bold">DNHostSend (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | DNWORKINFO.WORK_DAY
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
| **HARDWARE_TYPE**          |
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **SYSTEM_CONN_KEY**        | 
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **VENDOR_CODE**            | DNWORKINFO.VENDOR_CODE
| **VENDOR_NAME**            | DNWORKINFO.VENDOR_NAME
| **RECEIVE_TICKET_NO**      | DNWORKINFO.RECEIVE_TICKET_NO
| **RECEIVE_LINE_NO**        | DNWORKINFO.RECEIVE_LINE_NO
| **RECEIVE_BRANCH_NO**      |
| **COMPANY_CODE**           | RECEIVE_LINE_NO
| **COMPANY_NAME**           |
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **ORDER_NO**               |
| **PLAN_AREA_NO**           | DNWORKINFO.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNWORKINFO.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
| **JAN**                    |
| **CASE_ITF**               |
| **BUNDLE_ITF**             |
| **ENTERING_QTY**           |
| **BUNDLE_ENTERING_QTY**    |
| **PLAN_LOT_NO**            |
| **NOTE**                   |
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **RESULT_QTY**             | DNWORKINFO.RESULT_QTY
| **SHORTAGE_QTY**           |
| **RESULT_AREA_NO**         | DNWORKINFO.PLAN_AREA_NO
| **RESULT_LOCATION_NO**     | DNWORKINFO.PLAN_LOCATION_NO
| **RESULT_LOT_NO**          |
| **WORK_NOTE**              |
| **REPORT_FLAG**            | DNSTORAGEPLAN.REPORT_FLAG
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            | DNWORKINFO.TERMINAL_NO
| **WORK_SECOND**            |
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


# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)
