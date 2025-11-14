#<span style="color:skyblue; font-weight:bold">Retrieval from 1301, 1302, 1205, 1206, 1207, 1208, 1209</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 9002 , 9003, 9004, 9005, 9006, 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1205, 1206, 1207, 1208, 1209, 1301, 1302] 
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
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

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

#<span style="color:skyblue; font-weight:bold">Unplanned Retrieval database flow</span>
| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|
| **Only For 1301 & 1302**                                                        |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |
| **Directly For 1205,1206,1207,1208,1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Inquiry Retrieval Setting - Set(F2) [(1)](#Inquiry-Retrieval-Setting---Set(F2)) | I  |    | S  | I  | S  |    |     | S  | S  |    | S  | S  | S  |    |
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |  
| ID32[(3)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    
| ID33[(4)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     
| ID64[(5)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |
| ID68[(6)](#ID68)                                                                |    |    |    |    |    |    |     |    |    |    |    |    |    | I  |
| ID26[(7)](#ID26)                                                                | U  | I  | D  | D  | D  | I  |     |    | U  | I  |    |    |    |    |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |   |    | U  |    |    |    |    |    |    | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
**<span style="color:green">Only For 1301 & 1302.</span>** If not, start from [Inquiry Retrieval Setting - Set(F2)](#Inquiry-Retrieval-Setting---Set(F2))

::: mermaid
flowchart LR
    input[
        Operators pressed the storage mode button on the operation box.      
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

#Inquiry Retrieval Setting - Set(F2)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.InquiryRetrievalSettingSCH &nbsp;</span>

 ![==image_0==.jpg](/.attachments/==image_0==-58f99576-be7b-41a4-bc02-6cef92ce5b43.jpg) 

The Inquiry Retrieval Setting screen uses for manually set the retrieval work(UnitRetrieval).​ This screen uses when there are some troubles with Host System Linkage(Irregular Cases) or when manual operation is necessary.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Unplanned Retrieval
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Selected Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- Selected Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

::: mermaid
flowchart LR
    input[
        Station
        Pallet #
        SoftZone
        Priority
        To Location
        All Qty
        Retrieval Qty
        Stock Qty
        Material Code
        Material Name
        Material Type
        Batch #
        Area
        Loc #
        Stock Status
        Storage Date/Time
        Qty kg/crtn
        Qty crtn/PL
        UOM
        Tempering Flag
        Tempering Period
        QC Duration
        QC Check Flag
        Truck No
        Plate No
    ]

    tableList-insert[("
        DNWORKINFO
        DNCARRYINFO
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
        DMTOSTATION
    ")]

    className[InquiryRetrievalSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

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
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: Value from screen (Batch No)    
*   **MATERIAL_CODE**: Value from screen (Material Code)    
*   **MATERIAL_NAME**: Value from screen (Material Name)    
*   **PLAN_QTY**: Value from screen (Storage Qty)    
*   **QTY_KG_CRTN**: Value from screen (Qty kg/crtn)    
*   **QTY_CRTN_PL**: Value from screen (Qty crtn/PL)    
*   **UOM**: Value from screen (UOM)    
*   **TEMPERING_PERIOD**: Value from screen (Tempering Period)    
*   **EXPIRY_DAYS**: Value from screen (Expiry Days)    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **USER_ID**: Login Info    
*   **STORAGE_LOCATION_FROM**: Value from screen (Storage Location From) ⟶ **FGW1 (Ambient/9002) or FGW2 (Tempering/9001)**    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **WORK_TYPE**: 26:Direct Transfer    
*   **CMD_STATUS**: 1:Started    
*   **PRIORITY**: 2:Normal    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location    
*   **CARRY_FLAG**: 3: Direct Transfer    
*   **WORK_NO**: Sequence Object    
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO    
*   **DEST_STATION_NO**: **Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7207, 7208, 7209, 7210, 7211, 7212, 7213, 7214)**    
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **END_STATION_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#Retrieval-Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.transmission.RetrievalSender&nbsp;</span>

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

retrievalsender-input-->retrievalsender-.U.->retrievalsender-update
retrievalsender-->id12msg
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1301, 1302, 1205, 1206, 1207, 1208, 1209 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

## DNCARRYINFO
- CMD_STATUS = 2:Waiting for response
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNPALLET
- STATUS_FLAG = 4:Being retrieved
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID32
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id32Process&nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process-.U.->id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

## DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process&nbsp;</span>

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
id33process-.U.->id33-update
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

## DMSHELF
- STATUS_FLAG = 0:Empty
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNCARRYINFO
- CMD_STATUS = 5:Retrieval completed
- RETRIEVAL_STATION_NO = DMSHELF.STATION_NO
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID64
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process &nbsp;</span>

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

## DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id68Process &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-->id68process
id68process--I-->id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

## DNOPERATIONDISPLAY
- CARRY_KEY = MC Key information from ID68
- STATION_NO = Station information from ID68
- ARRIAL_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id26Process&nbsp;</span>

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

## DMSHELF
## DNWORKINFO
## DNWORKLIST
## DNHOSTSEND

