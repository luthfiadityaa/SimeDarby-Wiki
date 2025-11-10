[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Planned Storage Setting (PKG) database flow</span>
##<span style="color:Green; font-weight:bold">Abbreviation</span>
| **CODE** | TABLE NAME       |
|----------|------------------|
| **STRP** | DNSTORAGEPLAN    | 
| **PLLT** | DNPALLET         | 
| **WRKI** | DNWORKINFO       | 
| **WRKL** | DNWORKLIST       | 
| **CRYI** | DNCARRYINFO      | 
| **STCK** | DNSTOCK          | 
| **HSTS** | DNHOSTSEND       |
| **ARVL** | DNARRIVAL        |
| **WRHS** | DMWAREHOUSE      |
| **SHLF** | DMSHELF          |
| **STCH** | DNSTOCKHISTORY   |
| **MTMS** | DMMASTERMATERIAL |
| **STSN** | DMSTATION        |

##<span style="color:Green; font-weight:bold">Inbound Table Data Flow </span>
|Action Name| SRTP | PLLT | WRKI | WRKL | CRYI | STCK | HSTS | ARVL | WRHS | SHLF | STCH | MTST | STSN |
|-----------|--|--|--|--|--|--|--|--|--|--|--|--|--|
| [Planned Storage from Host](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage-from-host%3C/span%3E) | INSERT| | | | | |
| [Planned Storage - Set (F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/Planned-Storage-to-Ambient-Area-(PKG)?_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage---set-(f2)%3C/span%3E) | UPDATE | | | | | | | | | | | SELECT | SELECT |
| [ID26 - Dummy Arrival](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26---dummy-arrival%3C/span%3E) | | INSERT | INSERT | |INSERT | | | INSERT | | | |
| [Automatic Mode Change Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eautomatic-mode-change-sender%3C/span%3E) | | UPDATE | UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%3C/span%3E) | | | | | UPDATE | | | UPDATE | | | |
| [ID64](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&pageId=942&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%3C/span%3E) | | | | | UPDATE | | | | | | |
| [ID26](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E) | | | | |INSERT | | | INSERT | | | |
| [Storage Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-sender%3C/span%3E) | | UPDATE| UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25-%3C/span%3E) | | | | | UPDATE | | | UPDATE | | | |
| [ID64](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64-%3C/span%3E) | | | | | UPDATE | | | | | | |
| [ID33](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid33%3C/span%3E) | UPDATE | UPDATE | UPDATE | | DELETE | UPDATE | INSERT | | | UPDATE | INSERT |


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
        Pallet #
        Document #
        Company Code
        Vendor Code / Vendor Name
        Material Code
        Material Name
        Batch #
        Storage Qty / Planned Qty / Stored Qty
        UOM
        Plant
        Line #
        Document Date
        Delivery Date
    ]

    tableList-select[("
        DMMASTERMATERIAL
        DMSTATION
    ")]

    tableList-update[("
        DNSTORAGEPLAN
    ")]

    className[PlannedStoragePkgSCH]

    input --> className--> |UPDATE| tableList-update
    tableList-select--> |SELECT| className 

:::

# <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Storage Packaging Material
- AGC is online.  <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Material Code exists in **DMMaterialMaster**
- Material Code filtered with <span style="color:green; font-weight:bold">MaterialType.ZPCK</span> 
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.
- Storage Qty, Planned Qty and Stored Qty must be greater than **"0"**
- Station <span style="color:green; font-weight:bold">(ST1106)</span> is not suspended <span style="color:green; font-weight:bold">(DMSTATION.SUSPEND.OFF)</span>
- Station <span style="color:green; font-weight:bold">(ST1106)</span> is not disconnected <span style="color:green; font-weight:bold">(DMSTATION.STATUS_FLAG.ACTIVE)</span>

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
| **BATCH_NO**               | Value from screen (Batch #)                                                   
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName


#<span style="color:skyblue; font-weight:bold">Storage Flow Process</span>
This storage process flow is refer to AGC linkage Specification
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

#<span style="color:skyblue; font-weight:bold">ID26 - Dummy Arrival</span>

::: mermaid
flowchart LR

releaseCommand["
Release Command from Storage Packaging Material Screen
After Completion 
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
DNCARRYINFO
DNPALLET
DNWORKINFO
")]

InOutStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->InOutStationOperator
InOutStationOperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let InOutStationOperator picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
| **Column Name**            | **Description / Notes**                       |
|----------------------------|-----------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**                | **Description / Notes**                       |
|--------------------------------|-----------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 26:Direct Transfer
| **CMD_STATUS**                 | 1:Started 
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 3: Direct Transfer
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO ⟶ **1106**
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(7211/7212/7213/7214)**
| **CANCEL_REQUEST**             | 0:Not Requested
| **SCHEDULE_NO**                | Sequence Object
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

### <span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**            | **Description / Notes**                           |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | DNARRIVAL.STATION_NO ⟶ **1106**                                                        
| **WH_STATION_NO**          | 9002                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                      
| **ALLOCATION_FLAG**        | 1:Allocated                                                      
| **EMPTY_FLAG**             | 0:Normal Pallet                                                                                                              
| **BCR_DATA**               | Barcode information from ID26                                                     
| **LAST_STORED_DATE**       | SYSTIMESTAMP                                                                                                           
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

### <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
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
| **BATCH_NO**               | DNSTORAGEPLAN.BATCH_NO
| **PLAN_AREA_NO**           | DNSTORAGEPLAN.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNSTORAGEPLAN.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNSTORAGEPLAN.MATERIAL_CODE
| **PLAN_QTY**               | DNSTORAGEPLAN.PLAN_QTY
| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY
| **RESULT_AREA_NO**         | DMWAREHOUSE.AREA_NO ⟶ **9002 (Packaging Material)**
| **RESULT_LOCATION_NO**     | DMWAREHOUSE.WAREHOUSE_NO
| **WORK_DAY**               | DNSTORAGEPLAN.WORK_DAY
| **USER_ID**                | Login Info
| **TERMINAL_NO**            | Login Terminal
| **STORING_PAIR_KEY**       | DNSTORAGEPLAN.STORING_PAIR_KEY
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

##<span style="color:skyblue; font-weight:bold">Automatic Mode Change Sender</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.AutomaticModeChangeSender` &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNARRIVAL
DNCARRYINFO
DMWAREHOUSE
DMSHELF
DNPALLET
DNSTOCK
DNWORKINFO
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->automaticmodechangesender-->id05msg
automaticmodechangesender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DMWAREHOUSE</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_USED_STATION_NO_PM**| Aisle Number where a reserved location belongs to
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name 

####<span style="color:skyblue; font-weight:bold">DMSHELF</span>
| **Column Name**                | **Description / Notes**                                  |
|--------------------------------|-------------------------------------------------------|
| **STATUS_FLAG**                | 2:Reserved Location
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**                    | **Description / Notes**                                  |
|------------------------------------|-----------------------------------------------|
| **AISLE_STATION_NO**               | Aisle Number where a reserved location belongs to
| **CMD_STATUS**                     | 2:Waiting for response
| **LAST_UPDATE_DATE**               | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**              | Class name


####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **STATUS_FLAG**                | 1: Working 
| **PLAN_LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | DNCARRYINFO.END_STATION_NO
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **AREA_NO**                    | DNCARRYINFO.END_STATION_NO
| **LOCATION_NO**                | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID25</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CMD_STATUS**                 | 3:Commanded
| **ERROR_CODE**                 | 0
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID64</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(STV)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**                 | 4:Pickup completed
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name


#<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process Storage
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNCARRYINFO
DNPALLET
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |update| id26-update
:::

Continue the process  <span style="color:green; font-weight:bold">storage</span> , AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
| **Column Name**              | **Description / Notes**                                  |
|----------------------------|---------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | DNCARRYINFO.CARRY_KEY       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**                | **Description / Notes**                                |
|--------------------------------|-----------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 2: Storage
| **CMD_STATUS**                 | 1:Started 
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 1: Storage
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9011/9012/9013/9014)**
| **CANCEL_REQUEST**             | 0:Not Requested
| **SCHEDULE_NO**                | Sequence Object
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

### <span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|                                                       
| **CURRENT_STATION_NO**     | DNARRIVAL.STATION_NO                                                       
| **WH_STATION_NO**          | 9002                                                                                                                                                         
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

##<span style="color:skyblue; font-weight:bold">Storage Sender</span>

::: mermaid
flowchart LR
storageSender-update[("
DNCARRYINFO
DNWORKINFO
DNPALLET
DNSTOCK
DMSHELF
DNARRIVAL
DMWAREHOUSE
")]
storageSender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

storageSender-input-->storageSender-->id05msg
storageSender--> |UPDATE| storageSender-update
:::

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender ` &nbsp;</span>

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DMWAREHOUSE (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **LAST_USED_STATION_NO_PM**    |  Aisle Number where a reserved location belongs to 

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Field Name**                 | **Insert Value**                              |
|--------------------------------|-----------------------------------------------|
| **STATUS_FLAG**                | 2:Reserved Location
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **CMD_STATUS**                 | 2:Waiting for response
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
| **Field Name**                 | **Insert Value**                              |
|--------------------------------|-----------------------------------------------|
| **PLAN_LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | DNCARRYINFO.END_STATION_NO
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **AREA_NO**                    | DNCARRYINFO.END_STATION_NO
| **LOCATION_NO**                | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID25 </span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CMD_STATUS                 | 3:Commanded
| ERROR_CODE                 | 0
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#<span style="color:skyblue; font-weight:bold">ID64 </span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(SRM)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**            | **Description / Notes**                              |
|----------------------------|-------------------------------------------------------|
| CMD_STATUS                 | 4:Pickup completed
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#<span style="color:skyblue; font-weight:bold">ID33</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id33` &nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY
DNINOUTRESULT
")]
id33-update[("
DNPALLET
DMSHELF
DNSTOCK
DNWORKINFO
DNSTORAGEPLAN
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--> |INSERT| id33-insert
id33process--> |UPDATE| id33-update
id33process--> |DELETE| id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

####<span style="color:skyblue; font-weight:bold">DMSHELF</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 1: Occupied
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CURRENT_STATION_NO         | Location Number information from ID33
| STATUS_FLAG                | 2:Occupied
| ALLOCATION_FLAG            | 0:Not allocated
| LAST_STORED_DATE           | SYSTIMESTAMP
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| RESULT_QTY                 | DNWORKINFO.PLAN_QTY
| RESULT_AREA_NO             | DNWORKINFO.PLAN_AREA_NO
| RESULT_LOCATION_NO         | DNWORKINFO.PLAN_LOCATION_NO
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| STATUS_FLAG                | 4:Completed
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STORAGE_DAY                | DMWARENAVISYSTEM.WORK_DAY
| NEWEST_STORAGE_DATE        | SYSTIMESTAMP
| STOCK_QTY                  | DNSTOCK.PLAN_QTY
| ALLOCATION_QTY             | DNWORKINFO.RESULT_QTY
| PLAN_QTY                   | 0
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold"> DNSTORAGEPLAN</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 4: Completed
| RESULT_QTY                 | DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| INC_DEC_TYPE               | 1: Stock Increase
| JOB_TYPE                   | 2: Storage
| UPDATE_STOCK_QTY           | DNSTOCK.STOCK_QTY
| INC_DEC_QTY                | DNSTOCK.STOCK_QTY
| STOCK_ID                   | DNSTOCK.STOCK_ID
| AREA_NO                    | DNSTOCK.AREA_NO
| LOCATION_NO                | DNSTOCK.LOCATION_NO
| STORAGE_DAY                | DNSTOCK.STORAGE_DAY
| STORAGE_DATE               | DNSTOCK.STORAGE_DATE
| NEWEST_STORAGE_DATE        | DNSTOCK.NEWEST_STORAGE_DATE
| MATERIAL_CODE              | DNSTOCK.MATERIAL_CODE
| PALLET_ID                  | DNSTOCK.PALLET_ID
| BCR_DATA                   | DNPALLET.BCR_DATA
| AREA_TYPE                  | DMAREA_AREA_TYPE
| MATERIAL_NAME              | DMMATERIALMASTER.MATERIAL_NAME
| USER_ID                    | Login info
| USER_NAME                  | Login info
| TERMINAL_NO                | Login info
| TERMINAL_NAME              | Login info
| IP_ADDRESS                 | Login info
| REGIST_DATE                | SYSTIMESTAMP
| REGIST_PNAME               | Class name

####<span style="color:skyblue; font-weight:bold">DNINOUTRESULT</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| RESULT_KIND                | 1:Storage(Stock+)
| STATION_NO                 | DNCARRYINFO.SOURCE_STATION_NO
| LOCATION_NO                | DNCARRYINFO.DEST_STATION_NO
| WH_STATION_NO              | DNPALLET.WH_STATION_NO
| AISLE_STATION_NO           | DNCARRYINFO.AISLE_STATION_NO
| WORK_TYPE                  | DNCARRYINFO.WORK_TYPE
| RETRIEVAL_DETAIL           | DNCARRYINFO.RETRIEVAL_DETAIL
| WORK_NO                    | DNCARRYINFO.WORK_NO
| PALLET_ID                  | DNPALLET.PALLET_ID
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| RESTORING_FLAG             | DNCARRYINFO.RESTORING_FLAG
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| REMOVE_FLAG                | 00:Normal
| REGIST_PNAME               | Class name
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNHOSTSEND</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| WORK_DAY                   | DNWORKINFO.WORK_DAY
| JOB_NO                     | DNWORKINFO.JOB_NO
| COLLECT_JOB_NO             | DNWORKINFO.COLLECT_JOB_NO
| SETTING_UNIT_KEY           | DNWORKINFO.SETTING_UNIT_KEY
| JOB_TYPE                   | DNWORKINFO.JOB_TYPE
| STATUS_FLAG                | DNWORKINFO.STATUS_FLAG
| HARDWARE_TYPE              | DNWORKINFO.HARDWARE_TYPE
| PLAN_UKEY                  | DNWORKINFO.PLAN_UKEY
| STOCK_ID                   | DNWORKINFO.STOCK_ID
| SYSTEM_CONN_KEY            | DNWORKINFO.SYSTEM_CONN_KEY
| PLAN_DAY                   | DNWORKINFO.PLAN_DAY
| BATCH_NO                   | DNWORKINFO.BATCH_NO
| PLAN_AREA_NO               | DNWORKINFO.PLAN_AREA_NO
| PLAN_LOCATION_NO           | DNWORKINFO.PLAN_LOCATION_NO
| MATERIAL_CODE              | DNWORKINFO.MATERIAL_CODE
| MATERIAL_NAME              | DMMATERIALMASTER.MATERIAL_NAME
| UOM                        | MMATERIALMASTER.UOM
| ENTERING_QTY               | DMMATERIALMASTER.QTY_CRTN
| BUNDLE_ENTERING_QTY        | DNPALLETIZE.PLANNED_CARTON_QTY
| PLAN_QTY                   | DNWORKINFO.PLAN_QTY
| RESULT_QTY                 | DNWORKINFO.RESULT_QTY
| SHORTAGE_QTY               | DNWORKINFO.SHORTAGE_QTY
| RESULT_AREA_NO             | DNWORKINFO.RESULT_AREA_NO
| RESULT_LOCATION_NO         | DNWORKINFO.RESULT_LOCATION_NO
| RESULT_LOT_NO              | DNWORKINFO.RESULT_LOT_NO
| USER_ID                    | DNWORKINFO.USER_ID
| TERMINAL_NO                | DNWORKINFO.TERMINAL_NO
| WORK_SECOND                | DNWORKINFO.WORK_SECOND
| USER_NAME                  | DCUSER.USERNAME
| REPORT_FLAG                | 1:Reported
| REGIST_DATE                | SYSTIMESTAMP
| REGIST_PNAME               | Class name
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

##<span style="color:skyblue; font-weight:bold">[Planned Storage PKG - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/836/Planned-Storage-PKG)</span>


# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Storage Flow (Packaging Material) - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/Storage-Flow-(Packaging-Material))
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)
