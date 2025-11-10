[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Empty Pallet Storage database flow</span>
##<span style="color:Green; font-weight:bold">Abbreviation</span>
| **CODE** | TABLE NAME       |
|----------|------------------|
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
|Action Name| PLLT | WRKI | WRKL | CRYI | STCK | HSTS | ARVL | WRHS | SHLF | STCH | MTST | STSN |
|-----------|--|--|--|--|--|--|--|--|--|--|--|--|
| [Empty Pallet - Set (F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&pageId=884&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eempty-pallet---set-(f2)%3C/span%3E) | INSERT | | | | | | | | | | SELECT | SELECT |

#<span style="color:skyblue; font-weight:bold">Empty Pallet - Set (F2)</span>
![image.png](/.attachments/image-ae0da33c-5f36-45e7-ad8d-527f6a9e9325.png)
The Empty Pallet Setting Screen uses for storage the empty pallet to ASRS.

::: mermaid
flowchart LR
    input[
        Pallet #      
    ]

    tableList-insert[("
        DNPALLET
    ")]

    tableList-select[("
        DMSTATION
        DMMASTERMATERIAL
    ")]


    className[EmptyPalletSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

# <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Storage Packaging Material
- AGC is online.  <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Material Code exists in **DMMaterialMaster**
- Material Code filtered with <span style="color:green; font-weight:bold">MATERIALCODE.EMP_PB</span> 
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.
- Station <span style="color:green; font-weight:bold">(ST1301)</span> is not suspended <span style="color:green; font-weight:bold">(DMSTATION.SUSPEND.OFF)</span>
- Station <span style="color:green; font-weight:bold">(ST1302)</span> is not disconnected <span style="color:green; font-weight:bold">(DMSTATION.STATUS_FLAG.ACTIVE)</span>

## <span style="color:skyblue; font-weight:bold">Table Value</span>

### <span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**            | **Description / Notes**                           |
|----------------------------|---------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                                                                         
| **EMPTY_FLAG**             | 1: EMPTY PALLET                                                                                                               
| **BCR_DATA**               | Value from Screen (Pallet #)                                                   
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

#<span style="color:skyblue; font-weight:bold">Storage Flow Process</span>
This storage process flow is refer to AGC linkage Specification
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

#<span style="color:skyblue; font-weight:bold">ID26</span>

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
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO ⟶ **1301/1302**
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(7207/7208/7209/7210)**
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

#User Story
- [DFD Storage Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5787)

# Related DFD
  - {}
