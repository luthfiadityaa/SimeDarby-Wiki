[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Planned Storage Setting (PKG) database flow</span>
##<span style="color:Green; font-weight:bold">Abbreviation</span>
| CODE | TABLE NAME       |
|------|------------------|
| STRP | DNSTORAGEPLAN    | 
| PLLT | DNPALLET         | 
| WRKI | DNWORKINFO       | 
| WRKL | DNWORKLIST       | 
| CRYI | DNCARRYINFO      | 
| STCK | DNSTOCK          | 
| STCH | DNSTOCKHISTORY   |
| HSTS | DNHOSTSEND       |
| MTMS | DMMASTERMATERIAL |
| STSN | DMSTATION        |


##<span style="color:Green; font-weight:bold">Inbound Table Data Flow </span>
|Action Name| STRP | PLLT | WRKI | WRKL | CRYI | STCK | HSTS | MTMS | STSN |
|-----------|--|--|--|--|--|--|--|--|--|
| [Planned Storage from Host](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20to%20Ambient%20Area%20(PKG)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage-from-host%3C/span%3E) | INSERT| | | | | |
| [Planned Storage - Set (F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/Planned-Storage-to-Ambient-Area-(PKG)?_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eplanned-storage---set-(f2)%3C/span%3E) | UPDATE| | | | | | | SELECT | SELECT |


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

    tableList-insert[("
        DNPALLET
    ")]

    className[PlannedStoragePkgSCH]

    input --> className--> |INSERT| tableList-insert
    tableList-select-->|SELECT |className --> |UPDATE| tableList-update

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
| **BATCH_NO**               | Value from screen (Batch #)                                                   
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)
