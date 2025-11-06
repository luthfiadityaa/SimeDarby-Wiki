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

### <span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                                                                             
| **STATUS_FLAG**            | 0:Not Started                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | Value from SAP (Delivery Date)                                                       
| **VENDOR_CODE**            | Value from SAP (Vendor Code)
| **VENDOR_NAME**            | Value from SAP (Vendor Name)                                                     
| **COMPANY_CODE**           | Value from SAP (Company Code)                                                      
| **RECEIVE_TICKET_NO**      | Value from SAP (Document Number)                                                      
| **RECEIVE_LINE_NO**        | Value from SAP (Item No / Line No)                                                      
| **RECEIVE_TICKET_DATE**    | Value from SAP (Document Date)                                                                                                              
| **PLAN_AREA_NO**           | Value from SAP (Plant)                                                      
| **PLAN_LOCATION_NO**       | DMWAREHOUSE.Warehose_no                                                      
| **MATERIAL_CODE**          | Value from SAP (Material Code)     
| **UOM**                    | Value from SAP (UOM)                                                                                                                         
| **PLAN_QTY**               | Planned Quantity                                                                                                            
| **REPORT_FLAG**            | 0:Not Reported                                                                                                           
| **REGIST_KIND**            | 0:File Loading                                                      
| **ERROR_INDICATION**       | 0:Successfull
| **TYPE**                   | S: Success
| **REGIST_DATE**            | SYSTIMESTAMP                                                  
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

#<span style="color:skyblue; font-weight:bold">Planned Storage - Set (F2)</span>
Planned Storage Setting (PKG) is used to set the information of stock which will be entered into ASRS. After **Set(F2)** all item in input text will be process and The result will be posted back to SAP as [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)

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
        DNCARRYINFO<br>
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
- Station <span style="color:green; font-weight:bold">(ST1106)</span> is not suspended <span style="color:green; font-weight:bold">(DMSTATION.SUSPEND.OFF)</span>
- Station <span style="color:green; font-weight:bold">(ST1106)</span> is not disconnected <span style="color:green; font-weight:bold">(DMSTATION.STATUS_FLAG.ACTIVE)</span>

# <span style="color:skyblue; font-weight:bold">Storage Process flow</span>
This section explains the Storage flow after the operation of Planned Storage Packaging Material is started.
##[Storage Flow](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/Storage-Flow-(Packaging-Material))


## <span style="color:skyblue; font-weight:bold">Table Value</span>

### <span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN</span>
| **Column Name**            | **Description / Notes**                           |
|----------------------------|---------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                                                                             
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                                                                          
| **PLAN_QTY**               | Value from screen (Planned Qty)                                                     
| **PROCESS_QTY**            | Value from screen (Storage Qty)                                                       
| **RESULT_QTY**             | Value from screen (Stored Qty)                                                                                                            
| **REPORT_FLAG**            | 0:Not Reported                                                      
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY                                                                                                            
| **BCR_DATA**               | Value from screen (Pallet #)                                                      
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**            | **Description / Notes**                           |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | Only 1 route station (ST1106)                                                       
| **WH_STATION_NO**          | 9002                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                      
| **ALLOCATION_FLAG**        | 1:Allocated                                                      
| **EMPTY_FLAG**             | 0:Normal Pallet                                                                                                              
| **BCR_DATA**               | Value from screen (Pallet #)                                                     
| **LAST_STORED_DATE**       | SYSTIMESTAMP                                                                                                           
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
| **HARDWARE_TYPE**          | 3:ASRS
| **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
| **STOCK_ID**               | Sequence Object
| **SYSTEM_CONN_KEY**        | Sequence Object
| **PLAN_DAY**               | DNSTORAGE.PLAN_DAY
| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
| **VENDOR_NAME**            | DNSTORAGEPLAN.VENDOR_NAME
| **RECEIVE_TICKET_NO**      | DNSTORAGEPLAN.RECEIVE_TICKET_NO
| **RECEIVE_LINE_NO**        | DNSTORAGEPLAN.RECEIVE_LINE_NO
| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
| **BATCH_NO**               | Value from screen (Batch #)
| **PLAN_AREA_NO**           | DNSTORAGEPLAN.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNSTORAGEPLAN.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
| **PLAN_QTY**               | DNSTORAGEPLAN.PLAN_QTY
| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY
| **RESULT_AREA_NO**         | 9002 (Packaging Material)
| **RESULT_LOCATION_NO**     | DMWAREHOUSE.WAREHOUSE_NO
| **WORK_DAY**               | DNSTORAGEPLAN.WORK_DAY
| **USER_ID**                | Login Info
| **TERMINAL_NO**            | Login Terminal
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNWORKLIST (INSERT)</span>
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
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **PLAN_AREA_NO**           | DNWORKINFO.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNWORKINFO.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
| **STORAGE_TYPE**           | DNSTORAGEPLAN.STORAGE_TYPE
| **STORAGE_DATE**           | DNSTORAGEPLAN.STORAGE_DATE
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **WORK_NO**                | DNWORKINFO.JOB_NO
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            | DNWORKINFO.TERMINAL_NO
| **STORING_PAIR_KEY**       | DNWORKINFO.STORING_PAIR_KEY 
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP 
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNCARRYINFO (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **CARRY_KEY**              | DNWORKINFO.SYSTEM_CONN_KEY
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **WORK_TYPE**              | 02:Storage
| **CMD_STATUS**             | 1:Started 
| **PRIORITY**               | 2:Normal
| **RESTORING_FLAG**         | 0:Not Restore to Original Location
| **CARRY_FLAG**             | 1:Storage
| **WORK_NO**                | DNWORKINFO.JOB_NO
| **SOURCE_STATION_NO**      | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**        | DNWORKINFO.PLAN_AREA_NO
| **CANCEL_REQUEST**         | 0:Not Requested
| **SCHEDULE_NO**            | Sequence Object
| **END_STATION_NO**         | DNWORKINFO.PLAN_AREA_NO
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNSTOCK (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **STOCK_ID**               | DNWORKINFO.STOCK_ID  
| **AREA_NO**                | DNSTORAGEPLAN.AREA_NO
| **LOCATION_NO**            | 99999999999
| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
| **COMPANY_CODE**           | DNSTORAGEPLAN.COMPANY_CODE
| **VENDOR_CODE**            | DNSTORAGEPLAN.VENDOR_CODE
| **VENDOR_NAME**            | DNSTORAGEPLAN.VENDOR_NAME 
| **STORAGE_TYPE**           | 2:New 
| **STORAGE_DAY**            | DNSTORAGEPLAN.PLAN_DAY
| **STORAGE_DATE**           | DNSTORAGEPLAN.REGIST_DATE
| **ALLOCATION_QTY**         | 0
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **BATCH_NO**               | Value from screen (Batch #) 
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)| 
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNHOSTSEND (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | DNWORKINFO.WORK_DAY
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **VENDOR_CODE**            | DNWORKINFO.VENDOR_CODE
| **VENDOR_NAME**            | DNWORKINFO.VENDOR_NAME
| **RECEIVE_TICKET_NO**      | DNWORKINFO.RECEIVE_TICKET_NO
| **RECEIVE_LINE_NO**        | DNWORKINFO.RECEIVE_LINE_NO
| **COMPANY_CODE**           | RECEIVE_LINE_NO
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **PLAN_AREA_NO**           | DNWORKINFO.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNWORKINFO.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **RESULT_QTY**             | DNWORKINFO.RESULT_QTY
| **RESULT_AREA_NO**         | DNWORKINFO.PLAN_AREA_NO
| **RESULT_LOCATION_NO**     | DNWORKINFO.PLAN_LOCATION_NO
| **REPORT_FLAG**            | DNSTORAGEPLAN.REPORT_FLAG
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            | DNWORKINFO.TERMINAL_NO
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName


# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)
