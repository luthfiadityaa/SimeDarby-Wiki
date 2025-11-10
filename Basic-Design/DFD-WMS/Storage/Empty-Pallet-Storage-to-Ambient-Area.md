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
| [ID26](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E) | | INSERT | INSERT | |INSERT | | | INSERT | | | |
| [Automatic Mode Change Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eautomatic-mode-change-sender%3C/span%3E) | | UPDATE | UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%3C/span%3E) | | | | | UPDATE | | | UPDATE | | | |
| [ID64](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&pageId=884&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%3C/span%3E) | | | | | UPDATE | | | | | | |
| [ID26](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E) | | | | |INSERT | | | INSERT | | | |
| [Storage Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-sender%3C/span%3E) | | UPDATE| UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Empty%20Pallet%20Storage%20to%20Ambient%20Area&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%3C/span%3E) | | | | | UPDATE | | | UPDATE | | | |
| [ID64](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64-%3C/span%3E) | | | | | UPDATE | | | | | | |
| [ID33](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material&pageId=882&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid33%3C/span%3E) | UPDATE | UPDATE | UPDATE | | DELETE | UPDATE | INSERT | | | UPDATE | INSERT |

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
| **CURRENT_STATION_NO**     | DNARRIVAL.STATION_NO ⟶ **1301/1302**                                                        
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
| **RESULT_AREA_NO**         | DMWAREHOUSE.AREA_NO ⟶ **9002**
| **RESULT_LOCATION_NO**     | DMWAREHOUSE.WAREHOUSE_NO
| **WORK_DAY**               | DNSTORAGEPLAN.WORK_DAY
| **USER_ID**                | Login Info
| **TERMINAL_NO**            | Login Terminal
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
| **LAST_USED_STATION_NO_EP**| Aisle Number where a reserved location belongs to
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
| **LOCATION_NO**                | DN
| **STORAGE_DAY**                | DNSTOCK.STORAGE_DAY
| **STORAGE_DATE**               | DNSTOCK.STORAGE_DATE
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
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9007/9008/9009/9010)**
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
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_USED_STATION_NO_EP**| Aisle Number where a reserved location belongs to
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name 

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Column Name**                | **Description / Notes**                       |
|--------------------------------|-----------------------------------------------|
| **STATUS_FLAG**                | 2:Reserved Location
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Column Name**                | **Description / Notes**                        |
|--------------------------------|------------------------------------------------|
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **CMD_STATUS**                 | 2:Waiting for response
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
| **Column Name**                | **Description / Notes**                       |
|--------------------------------|-----------------------------------------------|
| **PLAN_LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET</span>
| **Column Name**                | **Description / Notes**                        |
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
DNSTOCKHISTORY
DNINOUTRESULT
")]
id33-update[("
DNPALLET
DMSHELF
DNSTOCK
DNWORKINFO
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
| STOCK_QTY                  | 1
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| INC_DEC_TYPE               | 1: Stock Increase
| JOB_TYPE                   | 2: Storage
| STOCK_ID                   | DNSTOCK.STOCK_ID
| AREA_NO                    | DNSTOCK.AREA_NO
| LOCATION_NO                | DNSTOCK.LOCATION_NO
| STORAGE_DAY                | DNSTOCK.STORAGE_DAY
| STORAGE_DATE               | DNSTOCK.STORAGE_DATE
| NEWEST_STORAGE_DATE        | DNSTOCK.NEWEST_STORAGE_DATE
| PALLET_ID                  | DNSTOCK.PALLET_ID
| BCR_DATA                   | DNPALLET.BCR_DATA
| AREA_TYPE                  | DMAREA_AREA_TYPE
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
| WORK_NO                    | DNCARRYINFO.WORK_NO
| PALLET_ID                  | DNPALLET.PALLET_ID
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| RESTORING_FLAG             | DNCARRYINFO.RESTORING_FLAG
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| REMOVE_FLAG                | 00:Normal
| REGIST_PNAME               | Class name
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#User Story
- [DFD Storage Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5787)

# Related DFD
  - {}
