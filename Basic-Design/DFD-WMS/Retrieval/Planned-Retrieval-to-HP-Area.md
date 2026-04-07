[[_TOC_]]
[[_TOSP_]]

[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Retrieval from 1301, 1302, 1205, 1206, 1207, 1208, 1209</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 
9002 , 9003, 9004, 9005, 
9006, 9007, 9008, 9009, 
9010, 9011, 9012, 9013, 
9014]-->P2[RetrievalSender]-->P21[ID12]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1205, 1206, 
1207, 1208, 1209, 1301, 
1302] 
:::

## Station Groups
User can select either a group station or an individual station as retrieval destination.
When group is selected, the framework resolves to the least-busy member via `checkChildren` round-robin.

| Group | Members | Area |
|-------|---------|------|
| 1211 | 1205, 1301 | HP Area 1 |
| 1212 | 1206, 1207, 1208 | HP Area 2 |
| 1213 | 1209, 1302 | HP Area 3 |

`RetrievalRouteControllerImpl.retrievalDetermin()` detects group (WORKPLACE_TYPE != FLOOR),
calls `checkChildren()` which iterates members starting after `last_used_station_no` (round-robin).
Suspended members are skipped (via `RetrievalStationCheckerAllocate`).

## Operation Display Panel
Stations with display panel (OPERATION_DISPLAY = 1: display only):
**1205, 1206, 1207, 1208, 1209, 1301, 1302**

When pallet arrives, ID68 writes work data (material, qty, batch) to the display panel.

#<span style="color:skyblue; font-weight:bold">Planned Retrieval database flow</span>
**Abbreviation:**
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKLIST  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **HSTS** : DNHOSTSEND  
- **ARVL** : DNARRIVAL  
- **WRHS** : DMWAREHOUSE  
- **SHLF** : DMSHELF  
- **ITEM** : DMITEM
- **STSN** : DMSTATION  
- **STCH** : DNSTOCKHISTORY  
- **TTSN** : DMTOSTATION 
- **INOUT**: DNINOUTSTATION
- **RPLAN**: DNRETRIEVALPLAN

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD|INOUT|RPLAN| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|-----|-----|
| **Only For 1301 & 1302**                                                        |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |     |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |     |     |
| **Directly For 1205,1206,1207,1208,1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |     |
| Planned Retrieval Setting - Set(F2) [(1)](#Planned-Retrieval-Setting---Set(F2)) | I  | I  | S  | I  | U  |    |     | S  | S  |    | S  | S  |    |    |     | U   |
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |     |     |
| ID12[(3)](#ID12)                                                                |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |     |  
| ID32[(4)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |     |    
| ID33[(5)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     |     |     
| ID64[(6)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |     |
| ID68[(7)](#ID68)                                                                | S  |    |    | S  |    |    |     |    |    |    |    |    |    | I  |     |     |
| ID26[(8)](#ID26)                                                                | U  |    | D  | D  | D  | I  |     |    | U  | I  |    |    |    |    | I   |     |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |     |
| Planned Retrieval Result[(9)](#Planned-Retrieval-Result)                        |    |    |    |    |    | U  |     |    |    |    |    |    |    |    |     |     | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id63Process&nbsp;</span>

If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
**<span style="color:green">Only For 1301 & 1302.</span>** If not, start from [Inquiry Retrieval Setting - Set(F2)](#Inquiry-Retrieval-Setting---Set(F2))

::: mermaid
flowchart LR
    input[
        Operators pressed the retrieval mode button on the operation box.      
    ]

    id61msg("
     ID63
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id61msg-->id63process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DMStation</span>
**CURRENT_MODE**: 2: Retrieval Mode

#Planned Retrieval
All data in the Planned Retrieval Setting List originates from process [Planned Retrieval - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/838/Planned-Retrieval). Please refer to the details for further information.

#Planned Retrieval Setting - Set(F2)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.PlannedRetrievalSettingSCH &nbsp;</span>

![==image_0==.jpg](/.attachments/==image_0==-da2d8aa4-ecd6-4ef5-88b4-5a1178eb48bb.jpg) 

The Planned Retrieval Setting screen is used to start multiple retrieval work grouped byTicket # (Document Number in SAP).​

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Planned Retrieval
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>.
- The selected station mode must be <span style="color:green; font-weight:bold">Retrieval mode</span> if it is Bi-Direction station.
- Selected Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>.
- Selected Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>.
- **<span style="color:green; font-weight:bold">Daily Update</span>** is not running.
- **<span style="color:green; font-weight:bold">Retrieval Allocate Flag </span>** is in progress.
- **<span style="color:green; font-weight:bold">Inventory to retrieval</span>** is allocated.
- Shelf condition is not **<span style="color:red; font-weight:bold">NG (Not Goods), Prohibitied, Reserved for Storage and Empty</span>**.
- Input text with red asterisk <span style="color:red">(*)</span> is not empty.

::: mermaid
flowchart LR
    input[
        Pallet #
        Shipping Date
        Ticket #
        Status
        Ticket List
        Detail Record Qty
        Ticket Line #
        Material Code
        Material Name
        Batch #
        Batch Qty
        Batch UOM
        Dock #
        Truck Plate #    
    ]

    tableList-insert[("
        DNWORKINFO
        DNWORKLIST
        DNCARRYINFO
    ")]

    tableList-update[("       
        DNSTOCK
        DNPALLET
    ")]

    tableList-update2[("
        DNRETRIEVALPLAN
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
        DNSTOCK
    ")]

    className[PlanRetrievalSCH]
    className--> |Calling| methodName[RetrievalAllocateOperator→allocate] 
    methodName--> |Calling| methodName2[AbstractAllocateOperator→allocateStock]--> |INSERT| tableList-insert
    input --> className
    tableList-select --> |SELECT| className
    methodName--> |Calling| P1[RetrievalSender]
    methodName2--> |UPDATE| tableList-update
    methodName--> |UPDATE| tableList-update2

    click P1 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/956/Stacked-Empty-Pallet?anchor=retrieval-sender" "Go to Retrieval Retrieval Sender"
    style P1 fill:#00cc66,stroke:#006633,color:#ffffff

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 03:Retrieval    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: DNRETRIEVAL.PLAN_UKEY    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNSTOCK.LOCATION_NO
*   **PLAN_DAY**: DNRETRIEVALPLAN.PLAN_DAY   
*   **SHIP_TICKET_NO**: DNRETRIEVALPLAN.SHIP_TICKET_NO
*   **SHIP_LINE_NO**: DNRETRIEVALPLAN.SHIP_LINE_NO
*   **PLAN_LOT_NO**: DNSTOCK.LOT_NO
*   **PLAN_QTY**: DNRETRIEVALPLAN.PLAN_QTY
*   **UOM**: DNRETRIEVALPLAN.UOM
*   **ITEM_CODE**: DNRETRIEVALPLAN.ITEM_CODE    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNSTOCK.DNPALLET.BCR_DATA  
*   **DOCK_NO**: Value from Screen (Dock No)
*   **TRUCK_PLATE_NO**: Value from Screen (Truck Plate No)
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 3:Reserved for Retrieval
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **ALLOCATION_QTY**: DNSTOCK.STOCK_QTY - DNWORKINFO.PLAN_QTY
* **LAST_UPDATE_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNWORKINFO.PALLET_ID    
*   **WORK_TYPE**: 03:Retrieval    
*   **CMD_STATUS**: 0:Allocated    
*   **GROUP_NO**: DNWORKINFO.COLLECT_JOB_NO 
*   **GROUP_SEQNO**: To set the sequence of picking base on **GROUP_NO** 
*   **RESTORING_FLAG**: 0:Return to Different Location  
*   **WORK_NO**: Sequence Object    
*   **RETRIEVAL_STATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **SOURCE_STATION_NO**: DMSHELF.PARENT_STATION_NO    
*   **DEST_STATION_NO**: Value from screen (Station) ⟶ **<span style="color:green;">1205, 1206, 1207, 1208, 1209, 1301, 1302</span>**     
*   **PRIORITY**: Value from screen ⟶ **<span style="color:green;">(1:Urgent, 2:Normal)</span>**
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **CARRY_FLAG**: 2:Retrieval
*   **CANCEL_REQUEST**: 0:Not requested
*   **CONTROL_INFO**: DNWORKINFO.BCR_DATA
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

####<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY   
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO   
*   **WORK_NO**: DNCARRYINFO.WORK_NO    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNWORKINFO.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY
*   **SHIP_TICKET_NO**: DNWORKINFO.SHIP_TICKET_NO
*   **SHIP_LINE_NO**: DNWORKINFO.SHIP_LINE_NO    
*   **PLAN_LOT_NO**: DNWORKINFO.PLAN_LOT_NO    
*   **ENTERING_QTY**: DMITEM.ENTERING_QTY
*   **BUNDLE_ENTERING_QTY**: DMITEM.BUNDLE_ENTERING_QTY
*   **STOCK_QTY**: DNSTOCK.STOCK_QTY
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY
*   **UOM**: DNWORKINFO.UOM
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **ITEM_CODE**: DNWORKINFO.ITEM_CODE    
*   **RETRIEVAL_DETAIL**: 1:Unit Retrieval or 2:Partial Pick
*   **DOCK_NO**: DNWORKINFO.DOCK_NO
*   **PRIORITY**: Value from screen ⟶ **<span style="color:green;">(1:Urgent, 2:Normal)</span>**
*   **TRUCK_PLATE_NO**: DNWORKINFO.TRUCK_PLATE_NO 
*   **SOURCE_STATION_NO**: Source Station No
*   **DEST_STATION_NO**: Destination Station No
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **TERMINAL_NO**: Login Info    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

####<span style="color:skyblue; font-weight:bold">DNRetrievalPlan</span>
* **STATUS_FLAG**: 1:Working
*  **LAST_UPDATE_PNAME**: ClassName


#Retrieval Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalSender&nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
DMSTATION
")]

retrievalsender-update[("
DNCARRYINFO
DNPALLET
")]

id12msg("
ID 12
")

retrievalsender-input--> P1[retrievalsender→process]-->P2[SendCarry→getSendCarryArray]--> |UPDATE| retrievalsender-update
P2--> |SendText| id12msg

 click id12msg "#" "Go to ID12"
 style id12msg fill:#00cc66,stroke:#006633,color:#ffffff
:::

All Pallet Retrieval operation at Ambient or Tempering will be retrieved to Station 1301, 1302, 1205, 1206, 1207, 1208, 1209 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
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
")]

id33-update2[("
DNCARRYINFO
")]

id33-->P1[id33process→retrievalCompletion→normalRetrievalCompletion]
P1--> |Calling| P2[RetrievalCompleter→completeOperation]
P4[releaseOrReserveSourceShelf]
P5[RetrievalCompleteManager→updateCarryForCompRetrieval]
P2--> |Calling| P4--> |Calling| P3[ShelfController→releaseShelf]--> |UPDATE| id33-update
P2--> |Calling| P5--> |UPDATE| id33-update2
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DMShelf</span>
* **STATUS_FLAG**: 0:Empty
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **CURRENT_STATION_NO**: DMSHELF.PARENT_STATION_NO
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
id64process--> |UPDATE| id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id68Process &nbsp;</span>

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
* **ARRIVAL_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##<span style="color:skyblue; font-weight:bold">LED sign displays the work</span>  
[Display information]

Information get from DNCarryInfo.CARRY_KEY

* **Job Type**
* **Material Code**
* **Qty**
* **Dock No**
* **Truck Plate No**

![image.png](/.attachments/image-1e56dc6e-657e-47ff-9a47-575b04705583.png)

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Operators Press the 
completion button.
"]

id26msg("
ID 26
")

id26-update[("
DNWORKINFO
DNSHELF
")]

id26-delete[("
DNCARRYINFO
DNPALLET
DNSTOCK
")]

id26-insert[("
DNWORKLIST
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

After the completion button flashes, the operator removes the pallet and presses the completion button to clear the operation indication. After that, AGC will sending ID 26 to the id26process, then delete related records from DNPALLET, DNCARRYINFO, and DNSTOCK.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **STATUS_FLAG**: 4:Completed
*   **SHORTAGE_QTY**: **FILLED ONLY WHEN ANY TRANSACTION THAT DO NOT COMPLETE** always 0
*   **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
*   **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **RESULT_LOT_NO**: DNWORKINFO.LOT_NO
*   **RESULT_QTY**: DNWORKINFO.PLAN_QTY
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

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
*   **PLAN_LOT_NO**: DNWORKINFO.PLAN_LOT_NO    
*   **RESULT_LOT_NO**: DNWORKINFO.RESULT_LOT_NO
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO    
*   **PLAN_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO    
*   **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO    
*   **ITEM_CODE**: DNWORKINFO.ITEM_CODE    
*   **ITEM_NAME**: DMITEM.MATERIAL_NAME    
*   **UOM**: DNWORKINFO.UOM    
*   **ENTERING_QTY**: DMITEM.ENTERING_QTY      
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **RESULT_QTY**: DNWORKINFO.RESULT_QTY    
*   **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY     
*   **DOCK_NO**: **DNWORKINFO.DOCK_NO**
*   **TRUCK_PLATE_NO**: **DNWORKINFO.TRUCK_PLATE_NO**   
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **TERMINAL_NO**: DNWORKINFO.TERMINAL_NO    
*   **WORK_SECOND**: DNWORKINFO.WORK_SECOND    
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USERNAME (FROM DNWORKINFO.USER_ID) 
*   **MSG_ID**: UNIQUE COMMUNICATION ID
*   **MSG_TYPE**: 3: Delivery Order
*   **MSG_ID_ORI**: DNRETRIEVALPLAN.MSG_ID
*   **ERROR_INDICATION**: 0: Successful
*   **TYPE**: S: Success
*   **MESSAGE_DESC**:  -_EMPTY--
*   **POSTING_DATE**: DNWORKINFO.WORK_DAY
*   **START_DATE_TIME**: DNWORKINFO.REGIST_DATE
*   **END_DATE_TIME**: SYSTIMESTAMP
*   **REPORT_FLAG**: 0: Not Reported    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: Class name    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
*Only for 1301 & 1302*
*   **RESULT_KIND**: 2:Retrieval(Stock-)
*   **STATION_NO**: = DNCARRYINFO.DEST_STATION_NO
*   **LOCATION_NO**: DNSTOCK.LOCATION_NO
*   **WH_STATION_NO**: DNSTOCK.LOCATION_NO (DMSHELF.WH_STATION_NO)
*   **AISLE_STATION_NO**: DNSTOCK.LOCATION_NO (DMSHELF.PARENT_STATION_NO)
*   **WORK_TYPE**: DNWORKINFO.WORK_TYPE
*   **RETRIEVAL_DETAIL**: DNCARRYINFO.RETRIEVAL_DETAIL
*   **WORK_NO**: DNCARRYINFO.WORK_NO
*   **PALLET_ID**: DNSTOCK.PALLET_ID
*   **CARRY_KEY**: DNWORKINFO.CARRY_KEY
*   **RESTORING_FLAG**: DNCARRYINFO.RESTORING_FLAG
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **REMOVE_FLAG**: 00:Normal
*   **REGIST_PNAME**: Class name
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

#Planned Retrieval Result
- [Planned Retrieval Results](https://dev.azure.com/DaifukuSW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/848/Planned-Retrieval-Result)

<hr>

#User Story
- [DFD Retrieval for Shipping](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5790)

#Related DFD
- [Planned Retrieval from OP Area (PKG)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/890/Planned-Retrieval-from-OP-Area-(PKG))