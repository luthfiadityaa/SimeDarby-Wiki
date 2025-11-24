[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR

P1[FROM AISLE STATION - 9001, 
9002 , 9003, 9004, 9005, 
9006, 9007, 9008, 9009, 
9010, 9011, 9012, 9013, 
9014]-->P2[RetrievalSender]-->P21[ID12]-->P3[ID32]-->P4[ID33]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR

Cond1{Flow?} 

Cond1{Flow?}--> |Flow 1: Thorugh crane 7-10.
9001, 9002, 9003, 9004, 9005, 9006, 9011, 9012, 9013, 9014.| P5[ID64]--> |7207-7210| P8[ID26]-->P9[Retrieval Sender]-->P10[ID25]--> |SRM| P11[ID64]--> |STV| P12[ID64]--> P7[ID68]
Cond1{Flow?}--> |Flow 2: Not through crane 7-10.
9007, 9008, 9009, 9010| P6[ID64]--> P7[ID68]-->P81[ID26]-->P13[To Station 1303]
:::

#<span style="color:skyblue; font-weight:bold">Retrieval for QC Start database flow</span>
**Abbreviation:**
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKINFOLIST  
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

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD|INOUT| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|-----|
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |     |
| Retrieval for QC Start - Set(F2) [(1)](#Retrieval-for-QC-Start---Set(F2))       | I  | I  | S  | I  | U  |    |     | S  | S  |    | S  | S  | S  |    |     |
| RetrievalSender[(3)](#Retrieval-Sender)                                         | U  |    | U  | U  |    |    |     |    |    |    |    |    |    |    |     |  
| ID12[(2)](#ID12)                                                                |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |     | 
| ID32[(4)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |    
| ID33[(5)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     |     
| **Flow 1: Thorugh crane 7-10**                                                  |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID64[(6)](#ID64-at-STV-from-(9001-9006-&-9011-9014))                            |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| ID26[(7)](#ID26-at-7207-7210)                                                   |    |    | U  | U  |    |    | I   |    |    |    |    |    |    |    |     |
| Retrieval Sender[(8)](#Retrieval-Sender-at-7207-7210)                           |    |    |    | U  |    |    | U   |    |    |    |    |    |    |    |     |
| ID25[(9)](#ID25-at-7207-7210)                                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |    |    |     |
| ID64[(10)](#ID64-at-SRM-from-7207-7210)                                         |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| ID64[(11)](#ID64-at-STV-from-9007-910)                                          |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| **Flow 2: Not through crane 7-10**                                              |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID64[(6)](#ID64-at-STV-from-9001-9014)                                          |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| **Last Process:**                                                               |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID68[(7)](#ID68)                                                                | S  |    |    | S  |    |    |     |    |    |    |    |    |    | I  |     |
| ID26[(8)](#ID26)                                                                | U  | U  | U  | U  |    | I  |  I  |    | U  | I  |    |    |    |    | I   | 
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| Internal Location Transfer Result[(9)](#Internal-Location-Transfer-Result)      |    |    |    |    |    | U  |     |    |    |    |    |    |    |    |     |
 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id63Process&nbsp;</span>

If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.

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

# Retrieval for QC Start - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCSettingSCH &nbsp;</span>
 ![==image_0==.jpg](/.attachments/==image_0==-385c044c-0af1-4f9f-b8bd-8c195cfba22a.jpg) 

This screen is mainly used when the pallet needs to be inspected.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Retrieval for QC Start
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- The station mode must be <span style="color:green; font-weight:bold">Retrieval mode</span>.
- The Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- The Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Target pallet of Stock Status is <span style="color:green; font-weight:bold">UU</span>.
- Only the pallets with tempering flag <span style="color:green; font-weight:bold">Reached</span> are applicable for QC settings.
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

::: mermaid
flowchart LR
    input[
        Area
        QC Station
        Pallet #
        Material Code
        Batch #
        Stock Status
        Tempering Flag
        QC Check Flag
        Loc #
        Material Name
        Stock Qty
        Storage Date/Time
        Tempering Period
        QC Duration
    ]

    tableList-insert[("
        DNWORKINFO
        DNWORKLIST
    ")]

     tableList-update[("
        DNSTOCK
    ")]

     tableList-select[("
        DNSTOCK
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]

    className[QCSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className
    className --> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">QI</span>
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>  
*   **JOB_TYPE**: 40:Inventory Check (AS/RS)    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **PLAN_QTY**: DNSTOCK.STOCK_QTY    
*   **BATCH_NO**: DNSTOCK.BATCH_NO    
*   **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE   
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNPALLET.BCR_DATA  
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME      
*   **RETRIEVAL_DETAIL**: 0:Inventory Check   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY  
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG        
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: QCSettingSCH

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
DNWORKINFO
")]

id12msg("
ID 12
")

retrievalsender-input-->retrievalsender--> |UPDATE| retrievalsender-update
retrievalsender--> |SendText| id12msg
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1303 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 3:Reserved for Retrieval
* **ALLOCATION_FLAG**: 1:Allocated
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: RetrievalSender

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNSTOCK.PALLET_ID    
*   **WORK_TYPE**: 40:Inventory Check (AS/RS)    
*   **CMD_STATUS**: 1:Started    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location  
*   **WORK_NO**: Sequence Object    
*   **RETRIEVAL_STATION_NO**: DNSTOCK.LOCATION_NO
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO    
*   **DEST_STATION_NO**: Value from screen (Station) **<span style="color:green;">1303</span>**     
*   **PRIORITY**: Value from screen ⟶ **<span style="color:green;">(1:Urgent, 2:Normal)</span>**
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **CARRY_FLAG**: 2:Retrieval
*   **CANCEL_REQUEST**: 0:Not requested
*   **AISLE_STATION_NO**: DMSHELF.PARENT_STATION_NO
*   **END_STATION_NO**: DNCARRYINFO.DEST_STATION_NO  
*   **RETRIEVAL_DETAIL**: 0:Inventory Check
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: RetrievalSender

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>   
* **STATUS_FLAG**: 1:Working  
* **SYSTEM_CONN_KEY**: DNCARRYINFO.CARRY_KEY

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

inoutstationoperator[InOutStaionOperator]

id26msg-->inoutstationoperator
inoutstationoperator--> |INSERT| id26-insert
inoutstationoperator--> |UPDATE| id26-update
:::

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

<hr>

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ##[Flow 1 : through crane 7-10 from 9001-9006 & 9011-9014](#Flow-1)
- ##[Flow 2 : not through crane 7-10 from 9007-9010](#Flow-2)

<hr>

#Flow 1 

**<h2>Through crane 7-10</h2>**

Refer to AGC Linkage Specs: [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Retrieval Section - 14 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##ID64 at STV from (9001-9006 & 9011-9014)
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

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID26 at 7207-7210
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet Arrived at Conveyor Station
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

id26process[id26process]
retrievaloperator[RetrievalStationOperator]


buttonclicked --> id26msg
id26msg -->id26process
id26process-->retrievaloperator
retrievaloperator--> |INSERT| id26-insert
retrievaloperator--> |UPDATE| id26-update
:::

Continue the process Direct Transfer, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNArrival</span>
* **ARRIVAL_DATE**: SYSTIMESTAMP
* **STATION_NO**: Arrival Station Number from ID26
* **CARRY_KEY**: 99999999
* **BCR_DATA**: Barcode information from ID26
* **CONTROLINFO**: Control information from ID26
* **SEND_FLAG**: 0:Not sent
* **HEIGHT**: Dimension Information from ID26
* **WIDTH**: Dimension Information From ID26
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>

*   **CURRENT_STATION_NO**: DNARRIVAL.STATION_NO
*   **REGIST_DATE**: SYSTIMESTAMP
*   **REGIST_PNAME**: ClassName
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>

*   **WORK_TYPE**: 26:Direct Transfer
*   **CMD_STATUS**: 1:Started
*   **CARRY_FLAG**: 3: Direct Transfer
*   **SOURCE_STATION_NO**: DNARRIVAL.STATION_NO ⟶ (7207, 7208, 7209, 7210)
*   **DEST_STATION_NO**: **<span style="color:green">1303</span>**
*   **REGIST_DATE**: SYSTIMESTAMP
*   **REGIST_PNAME**: ClassName
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: ClassName

##Retrieval Sender at 7207-7210
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

retrievalsender-input-->retrievalsender--> |UPDATE| retrievalsender-update
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1303 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 4:Being retrieved
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID25 at 7207-7210
jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process

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
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 3:Commanded
* **ERROR_CODE**: 0
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID64 at SRM from 7207-7210
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

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID64 at at STV from 9007-9010 
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
id64process--> |DELETE| id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

#Flow 2
**<h2>not through crane 7-10</h2>** 
Refer to AGC Linkage Specs: [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Retrieval Section - 14 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##ID64 at STV from 9007-9010
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

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

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
* **ARRIAL_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet arrived at station
"]

buttonclicked2["
Internal Location Transfer Result 
"]

buttonclicked3["
Go to the next process -> QC WORK 
"]

id26msg("
ID 26
")

id26-update[("
DNWORKINFO
DNSHELF
DNCARRYINFO
DNPALLET
DNSTOCK
")]

id26-insert[("
DNARRIVAL
DNWORKLIST
DNHOSTSEND
DNINOUTRESULT
")]

id26process[id26process]
retrievaloperator[RetrievalStationOperator]

buttonclicked-->buttonclicked3
buttonclicked --> id26msg
id26msg -->id26process
id26process-->retrievaloperator

retrievaloperator--> |INSERT| id26-insert
retrievaloperator--> |UPDATE| id26-update
id26-insert-->buttonclicked2 

click buttonclicked2 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result" "Go Internal Location Transfer Result"
click buttonclicked3 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/961/QC-Work-from-Retrieval-for-QC-Start" "Go to QC Work"
style buttonclicked2 fill:#00cc66,stroke:#006633,color:#ffffff
style buttonclicked3 fill:#00cc66,stroke:#006633,color:#ffffff
:::

When the retrieved pallet reaches the QC station, Warenavi sends an ‘Internal Location Transfer’ message to the host system and initiates `QC Work`. 

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
* **CARRY_KEY**: DNCARRYINFO.CARRY_KEY
* **SEND_FLAG**: 0:Not Sent
* **BCR_DATA**: Arrival Information from ID26 

###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 6:Arrival
* **RESTORING_FLAG**: 0:Not Re-store to Original Location
* **CARRY_FLAG**: 1:Storage

###<span style="color:skyblue; font-weight:bold">DNPALLET</span>
* **STATUS_FLAG**: 1:Reserved for Storage
* **EMPTY_FLAG**: 0:Normal Pallet

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **STATUS_FLAG**: 4:Completed
*   **SHORTAGE_QTY**: DNWORKINFO.PLAN_QTY - DNWORKINFO.RESULT_QTY
*   **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
*   **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **STORAGE_LOCATION_FROM**: <span style="color:green; font-weight:bold">FGW1 or FGW2</span>
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: Login info
*   **TERMINAL_NO**: Login info
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
* **REGIST_DATE**
* **WORK_DAY**:
* **INC_DEC_TYPE**: 2:Stock Decrease (Retrieval)
* **JOB_TYPE**: 40:Inventory Check (AS/RS)
* **STOCK_ID**: DNSTOCK.STOCK_ID
* **PLAN_AREA_NO**: DNSTOCK.AREANO
* **PLAN_LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
* **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE
* **MATERIAL_NAME**: DMITEM.MATERIAL_NAME
* **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE
* **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
* **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG
* **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
* **QC_DURATION**: DNSTOCK.QC_DURATION
* **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG
* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **BCR_DATA**: DNPALLET.BCR_DATA
* **STORAGE_LOCATION_FROM**: <span style="color:green; font-weight:bold">FGW1 or FGW2</span>
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
* **TERMINAL_NO**: Login info
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: ClassName
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: ClassName

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
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME    
*   **RETRIEVAL_STATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **RETRIEVAL_DETAIL**: 1:Unit Retrieval   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY      
*   **STORAGE_LOCATION_FROM**: <span style="color:green; font-weight:bold">FGW1 or FGW2</span>
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
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
*   **ENTERING_QTY**: DNSTOCK.STOCK_QTY      
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **RESULT_QTY**: DNWORKINFO.RESULT_QTY    
*   **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY    
*   **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO    
*   **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **STORAGE_LOCATION_FROM**: <span style="color:green; font-weight:bold">FGW1 or FGW2</span>
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
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
  
#Internal Location Transfer Result
- [Internal Location Transfer Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)

#QC Work from Retrieval for QC Start
- [QC Work from Retrieval for QC Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/961/QC-Work-from-Retrieval-for-QC-Start)

<hr>

#User Story
- [DFD - QC Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5788)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Retrieval for QC Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/900/Retrieval-for-QC-Start)
