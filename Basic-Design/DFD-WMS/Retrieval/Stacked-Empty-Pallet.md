[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Retrieval from 1210</span>
![image.png](/.attachments/image-b5fcb612-8067-43e8-a98a-ac23b8402498.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010]-->P2[ID66]-->P3[RetrievalTriggerAllocator]-->P4[RetrievalSender]-->P5[ID32]-->P6[ID33]-->P7[ID64]-->P9[ID26]-->P10[To STATION - 1210] 
:::

#<span style="color:skyblue; font-weight:bold">Empty Stacked database flow</span>
**Abbreviation:**
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKINFOLIST  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **ARVL** : DNARRIVAL  
- **WRHS** : DMWAREHOUSE  
- **SHLF** : DMSHELF  
- **ITEM** : DMITEM
- **STSN** : DMSTATION  
- **STCH** : DNSTOCKHISTORY  

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|OPRD|INOUT| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|
| ID66 [(1)](#I66)                                                                |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |
| Retrieval Trigger Allocator [(2)](#Retrieval-Trigger-Allocator)                 | I  | I  | S  | I  | S  |    |     | S  | S  |    | S  | S  |    |    |
| ID12[(2)](#ID12)                                                                |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |    | 
| Retrieval Sender[(3)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |  
| ID32[(4)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    
| ID33[(5)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     
| ID64[(6)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |
| ID26[(8)](#ID26)                                                                | U  | U  | D  | D  | D  | I  |     |    | U  | I  |    |    |    | I  |

#ID66
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id66Process&nbsp;</span>

::: mermaid
flowchart LR
    input[
        Automated Retrieval Trigger.      
    ]

    id66msg("
     ID66
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id66msg-->id66process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

AGC will send ID66 when the MC contains data for any stock items that need to be retrieved. 

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DMStation</span>
**STATION_NO**: 1210
**RETRIEVAL_TRIGGER_REQUEST**: 2:Requested by AGC
**LAST_UPDATE_PNAME**: Id66Process

#Retrieval Trigger Allocator
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalTriggerAllocator&nbsp;</span>

::: mermaid
flowchart LR    

    tableList-insert[("
        DNWORKINFO
        DNWORKLIST
        DNCARRYINFO
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]

    className[RetrievalTriggerAllocator]

    className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

Next, the Retrieval Trigger Allocator will assign stock items that are eligible for retrieval.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 23:Unplanned Retrieval    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNSTOCK.LOCATION_NO
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY       
*   **MATERIAL_CODE**: **<span style="color:green">EMP_PB</span>**   
*   **MATERIAL_NAME**: **<span style="color:green">Empty Pallet</span>** 
*   **PLAN_QTY**: DNSTOCK.STOCK_QTY     
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNPALLET.BCR_DATA  
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#ID12

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.id.send.As21Id12&nbsp;</span>

::: mermaid
flowchart LR

id26msg("
ID12
")

id26-insert[("
DNCARRYINFO
")]

id26-update[("
DNPALLET
")]

retrievalstationoperator[RetrievalStationOperator]

id26msg-->retrievalstationoperator
retrievalstationoperator--> |INSERT| id26-insert
retrievalstationoperator--> |UPDATE| id26-update
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 3:Reserved for Retrieval
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNSTOCK.PALLET_ID    
*   **WORK_TYPE**: 23:Unplanned Retrieval    
*   **CMD_STATUS**: 1:Started    
*   **PRIORITY**: 2:Normal    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location  
*   **WORK_NO**: Sequence Object    
*   **RETRIEVAL_STATION_NO**: DNSTOCK.LOCATION_NO
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO → **<span style="color:green;">(9007,9008,9009,910)</span>**   
*   **DEST_STATION_NO**: **<span style="color:green;">1210**    
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **CARRY_FLAG**: 2:Retrieval
*   **CANCEL_REQUEST**: 0:Not requested
*   **AISLE_STATION_NO**: DMSHELF.PARENT_STATION_NO
*   **END_STATION_NO**: DNCARRYINFO.DEST_STATION_NO  
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#Retrieval Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalSender&nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
")]

retrievalsender-update[("
DNCARRYINFO
DNPALLET
")]

id12msg("
ID 12
")

retrievalsender-input-->retrievalsender--> |UPDATE| retrievalsender-update
retrievalsender-->id12msg
:::

All Empty Pallet Retrieval operation at Ambient will be retrieved to Station 1210 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 4:Being retrieved
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID32
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id32Process&nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process--> |UPDATE| id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 3:Commanded
* **ERROR_CODE**: 0
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id33Process&nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-update[("
DMSHELF
DNCARRYINFO
")]

id33-->id33process
id33process--> |UPDATE| id33-update
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DMShelf</span>
* **STATUS_FLAG**: 0:Empty
* **LAST_UPDATE_DATE**: SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 5:Retrieval completed
* **RETRIEVAL_STATION_NO**: DMSHELF.STATION_NO
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID64
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.U.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id68Process &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-select[("
DNCARRYINFO
DNWORKINFO
")]

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-select-->id68
id68-->id68process
id68process--> |INSERT| id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNOperationDisplay</span>
* **CARRY_KEY**: MC Key information from ID68
* **STATION_NO**: Station information from ID68
* **ARRIAL_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet is arrived at station
"]

id26msg("
ID 26
")

id26-update[("
DNWORKINFO
DNWORKLIST
DNSHELF
")]

id26-delete[("
DNCARRYINFO
DNPALLET
DNSTOCK
")]

id26-insert[("
DNHOSTSEND
DNINOUTRESULT
")]

id26process[id26process]
retrievaloperator[RetrievalStationOperator]


buttonclicked --> id26msg
id26msg -->id26process
id26process-->retrievaloperator

retrievaloperator--> |INSERT| id26-insert
retrievaloperator--> |UPDATE| id26-update
retrievaloperator--> |DELETE| id26-delete
:::

<span style="color:black; font-weight:bold; color:red">*The operator performs the operation according to the work display on the work terminal.</span>

After the completion button flashes, the operator removes the pallet and presses the completion button to clear the operation indication. At the same time, sending ID 26 to the id26process, then delete related records from DNPALLET, DNCARRYINFO, and DNSTOCK.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **STATUS_FLAG**: 4:Completed
*   **SHORTAGE_QTY**: DNWORKINFO.PLAN_QTY - DNWORKINFO.RESULT_QTY
*   **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
*   **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **USER_ID**: Login info
*   **TERMINAL_NO**: Login info
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWOKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMMASTERIALMASTER.MATERIAL_NAME    
*   **RETIREVAL_DETAIL**: 1:Unit Retrieval    
*   **RETRIEVAL_STATION_NO**: **<span style="color:green">1210</span>**    
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **STOCK_QTY**: DNWORKINFO.STOCK_QTY   
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **STATUS_FLAG**: DNWORKINFO.STATUS_FLAG    
*   **HARDWARE_TYPE**: DNWORKINFO.HARDWARE_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **SYSTEM_CONN_KEY**: DNWORKINFO.SYSTEM_CONN_KEY    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **PLAN_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **UOM**: MMATERIALMASTER.UOM        
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **RESULT_QTY**: DNWORKINFO.RESULT_QTY    
*   **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY    
*   **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO    
*   **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO    
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **TERMINAL_NO**: DNWORKINFO.TERMINAL_NO    
*   **WORK_SECOND**: DNWORKINFO.WORK_SECOND    
*   **USER_NAME**: DCUSER.USERNAME    
*   **REPORT_FLAG**: 0: Not Reported    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: Class name    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
*   **RESULT_KIND**: 2:Retrieval(Stock-)
*   **STATION_NO**: = DNCARRYINFO.DEST_STATION_NO
*   **LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
*   **WH_STATION_NO**: DNPALLET.WH_STATION_NO
*   **AISLE_STATION_NO**: DNCARRYINFO.AISLE_STATION_NO
*   **WORK_TYPE**: DNCARRYINFO.WORK_TYPE
*   **RETRIEVAL_DETAIL**: DNCARRYINFO.RETRIEVAL_DETAIL
*   **WORK_NO**: DNCARRYINFO.WORK_NO
*   **PALLET_ID**: DNPALLET.PALLET_ID
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY
*   **RESTORING_FLAG**: DNCARRYINFO.RESTORING_FLAG
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **REMOVE_FLAG**: 00:Normal
*   **REGIST_PNAME**: Class name
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

<hr>

#User Story
- [DFD Retrieval for Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5794)

#Related DFD
- [Empty Pallet Storage to Ambient Area - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/Empty-Pallet-Storage-to-Ambient-Area)