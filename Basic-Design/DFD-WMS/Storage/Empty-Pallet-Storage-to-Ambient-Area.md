[[_TOC_]]
[[_TOSP_]]

# Empty Pallet Storage database flow
## Abbreviation
| **CODE** | TABLE NAME       |
|----------|------------------|
| **PLLT** | DNPALLET         | 
| **WRKI** | DNWORKINFO       | 
| **WRKL** | DNWORKLIST       | 
| **CRYI** | DNCARRYINFO      | 
| **STCK** | DNSTOCK          | 
| **ARVL** | DNARRIVAL        |
| **WRHS** | DMWAREHOUSE      |
| **SHLF** | DMSHELF          |
| **STCH** | DNSTOCKHISTORY   |
| **INOT** | DNINOUTRESULT    |
| **ITEM** | DMITEM           |
| **STSN** | DMSTATION        |

| **CODE** | TABLE NAME       |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |


##<span style="color:Green; font-weight:bold">Inbound Table Data Flow </span>
| Action Name                 | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | ITEM | STSN |
|-----------------------------|------|------|------|------|------|------|------|------|------|------|------|------|
| [Empty Pallet - Set (F2)]() |   I  |      |      |      |      |      |      |      |      |      |   S  |   S  |
| [ID26]()                    |   U  |   I  |   I  |   I  |   I  |   I  |      |      |      |      |      |      |
| [Storage Sender]()          |   U  |      |      |   U  |      |   U  |   U  |   U  |      |      |      |      |
| [ID25]()                    |   U  |   U  |      |   U  |      |   D  |      |      |   I  |   I  |      |   S  |
| [ID64]()                    |      |      |      |   U  |      |      |      |      |      |      |      |      |
| [ID26]()                    |   U  |   U  |   U  |   U  |   U  |   I  |      |      |      |      |      |      |
| [Storage Sender]()          |   U  |   U  |      |   U  |   U  |   U  |   U  |   U  |      |      |      |   S  |
| [ID25]()                    |   U  |   U  |      |   U  |      |   D  |      |      |   U  |   U  |      |      |
| [ID64]()                    |      |      |      |   U  |      |      |      |      |      |      |      |      |
| [ID33]()                    |   U  |      |      |   D  |      |      |      |   U  |      |      |      |      |

# Empty Pallet - Set (F2)

![image.png](/.attachments/image-ae0da33c-5f36-45e7-ad8d-527f6a9e9325.png)
The Empty Pallet Setting Screen uses for storage the empty pallet to ASRS.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.storage.emptypalletsetting.EmptyPalletSettingSCH` &nbsp;</span>


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

## Validations
This section explains the validations for the whole proccess Storage Packaging Material
- AGC is online. **DMGroupController.STATUS_FLAG.ONLINE**
- Material Code exists in **DMMaterialMaster**
- Material Code filtered with **MATERIALCODE.EMP_PB** 
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Pallet Information does not exist in **DNCARRYINFO**  
  To check for Pallet Information:  
  **JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID**  
  **CONDITION DNPALLET.BCR_DATA = <Pallet Number>**  
  So if result > 0, Palletize Start cannot proceed.
- Station **(ST1301)** is not suspended **(DMSTATION.SUSPEND.OFF)**
- Station **(ST1302)** is not disconnected **(DMSTATION.STATUS_FLAG.ACTIVE)**

## DNPALLET
- PALLET_ID = Sequence Object
- BCR_DATA = Value from Screen (Pallet #)
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = ClassName
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

# Storage Flow Process

This storage process flow is refer to AGC linkage Specification

[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

# ID26

::: mermaid
flowchart LR

releaseCommand["
Release Command from Storage Packaging Material Screen
After Completion 
"]

id26msg("
ID 26
")

id26-update[("
DNPALLET
")]

id26-insert[("
DNARRIVAL
DNCARRYINFO
DNSTOCK
DNWORKINFO
DNWORKLIST
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let InOutStationOperator picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## DNPALLET
- CURRENT_STATION_NO = DNARRIVAL.STATION_NO ⟶ **1301/1302**  
- WH_STATION_NO      = 9002
- STATUS_FLAG        = 1:Reserved for Storage 
- ALLOCATION_FLAG    = 1:Allocated
- EMPTY_FLAG         = 0:Normal Pallet 
- LAST_STORED_DATE   = SYSTIMESTAMP
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNARRIVAL
- ARRIVAL_DATE       = SYSTIMESTAMP 
- STATION_NO         = Arrival Station Number from ID26
- CARRY_KEY          = 99999999
- BCR_DATA           = Barcode information from ID26
- CONTROLINFO        = Control information from ID26
- SEND_FLAG          = 0:Not sent
- HEIGHT             = Dimension Information from ID26
- WIDTH              = Dimension Information From ID26
- REGIST_DATE        = SYSTIMESTAMP
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNCARRYINFO
- CARRY_KEY          = Sequence Object
- PALLET_ID          = Sequence Object
- WORK_TYPE          = 26:Direct Transfer
- CMD_STATUS         = 1:Started
- PRIORITY           = 1:Urgent
- RESTORING_FLAG     = 0:Not Restore to Original Location
- CARRY_FLAG         = 1:Storage
- WORK_NO            = Sequence Object
- SOURCE_STATION_NO  = DNPALLET.CURRENT_STATION_NO ⟶ **1301/1302**
- DEST_STATION_NO    = Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(7207/7208/7209/7210)**
- CANCEL_REQUEST     = 0:Not Requested
- SCHEDULE_NO        = Sequence Object
- END_STATION_NO     = DNCARRYINFO.DEST_STATION_NO
- REGIST_DATE        = SYSTIMESTAMP
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNSTOCK
- STOCK_ID           = Sequence Object
- AREA_NO            = DNWORKINFO.PLAN_AREA_NO
- LOCATION_NO        = DNWORKINFO.PLAN_LOCATION_NO
- STORAGE_TYPE       = 2:NEW
- STORAGE_DAY        = DMWARENAVISYSTEM.WORK_DAY
- STORAGE_DATE       = SYSTIMESTAMP
- PLAN_QTY           = DNWORKINFO.PLAN_QTY
- PALLET_ID          = DNPALLET.PALLET_ID
- REGIST_DATE        = SYSTIMESTAMP
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNWORKINFO
- JOB_NO             = Sequence Object
- SETTING_UNIT_KEY   = Sequence Object
- COLLECT_JOB_NO     = Sequence Object
- JOB_TYPE           = 26:Direct Transfer
- STATUS_FLAG        = 1:WORKING
- HARDWARE_TYPE      = 3:ASRS
- PLAN_UKEY          = Sequence Object
- STOCK_ID           = DNSTOCK.STOCK_ID
- SYSTEM_CONN_KEY    = Sequence Object
- PLAN_DAY           = DMWARENAVISYSTEM.WORK_DAY
- PLAN_AREA_NO       = Area Number where a reserved location belongs to
- PLAN_LOCATION_NO   = Location Number where a reserved location belongs to
- PLAN_QTY           = 1
- WORK_DAY           = DMWARENAVISYSTEM.WORK_DAY
- USER_ID            = Login Info
- TERMINAL_NO        = Login Terminal
- REGIST_DATE        = SYSTIMESTAMP
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNWORKLIST
- JOB_NO             = DNWORKINFO.JOB_NO
- CARRY_KEY          = DNWORKINFO.SYSTEM_CONN_KEY
- SETTING_UNIT_KEY   = DNWORKINFO.SETTING_UNIT_KEY
- COLLECT_JOB_NO     = DNWORKINFO.COLLECT_JOB_NO
- JOB_TYPE           = DNWORKINFO.JOB_TYPE
- PLAN_UKEY          = DNWORKINFO.PLAN_UKEY
- STOCK_ID           = DNWORKINFO.STOCK_ID
- PALLET_ID          = DNCARRYINFO.PALLET_ID
- PLAN_DAY           = DNWORKINFO.PLAN_DAY
- PLAN_AREA_NO       = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO   = DNWORKINFO.PLAN_LOCATION_NO
- STORAGE_TYPE       = 2:New
- STORAGE_DATE       = SYSTIMESTAMP
- PLAN_QTY           = DNWORKINFO.PLAN_QTY
- PRIORITY           = DNCARRYINFO.PRIORITY
- WORK_NO            = DNCARRYINFO.WORK_NO
- PRIORITY           = DNCARRYINFO.PRIORITY
- SOURCE_STATION_NO  = DNCARRYINFO.SOURCE_STATION_NO
- DEST_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- SCHEDULE_NO        = DNCARRYINFO.SCHEDULE_NO
- END_STATION_NO     = DNCARRYINFO.END_STATION_NO
- USER_ID            = Login Info
- USER_NAME          = Login Info
- TERMINAL_NO        = Login Terminal
- REGIST_DATE        = SYSTIMESTAMP                                                    
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName


# Storage Sender

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender` &nbsp;</span>

::: mermaid
flowchart LR
StorageSender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNPALLET
DNARRIVAL
")]
StorageSender-input[("
DNARRIVAL
DNCARRYINFO
DMSTATION
")]

id05msg("
ID 05
")

StorageSender-input-->StorageSender-->id05msg
StorageSender--> |UPDATE| StorageSender-update
:::

After successful creation of arrival record in **ID26process**, Automatic Mode Change Sender is the following process where it will send **ID05** to AGC. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

##  DMWAREHOUSE
- LAST_USED_STATION_NO_EP = Aisle Number where a reserved location belongs to
- LAST_UPDATE_DATE        = SYSTIMESTAMP
- LAST_UPDATE_PNAME       = Class name 

## DMSHELF
- STATUS_FLAG        = 2:Reserved Location
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNCARRYINFO
- AISLE_STATION_NO   = Aisle Number where a reserved location belongs to
- CMD_STATUS         = 2:Waiting for response
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNPALLET
- CURRENT_STATION_NO = Reserved Location Number
- WH_STATION_NO      = DNCARRYINFO.END_STATION_NO
- BCR_DATA           = BCRData of station
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNARRIVAL
- CARRY_KEY          = DNCARRYINFO.CARRY_KEY
- SEND_FLAG          = 1:Sent
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name


# ID25

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-insert[("
DNINOUTRESULT
DNSTOCKHISTORY
")]

id25-update[("
DNCARRYINFO
DNPALLET
DNWORKINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.INSERT.->id25-insert
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete

:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

## DNCARRYINFO
- CMD_STATUS        = 3:Commanded
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNPALLET                                                                                               
- EMPTY_FLAG        = 1:Empty Pallet                                               
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNWORKINFO
- STATUS_FLAG         = 4: Completed
- RESULT_AREA_NO      = DNWORKINFO.PLAN_AREA_NO
- RESULT_LOCATION_NO  = DNWORKINFO.PLAN_LOCATION_NO
- RESULT_QTY          = DNWORKINFO.PLAN_QTY

## DNSTOCK
- STOCK_QTY           = DNWORKINFO.PLAN_QTY
- NEWEST_STORAGE_DATE = DNSTOCK.NEWEST_STORAGE_DATE

## DNINOUTRESULT
- RESULT_KIND       = 1:Storage(Stock+)
- STATION_NO        = DNCARRYINFO.SOURCE_STATION_NO
- LOCATION_NO       = DNCARRYINFO.DEST_STATION_NO
- WH_STATION_NO     = DNPALLET.WH_STATION_NO
- AISLE_STATION_NO  = DNCARRYINFO.AISLE_STATION_NO
- WORK_TYPE         = DNCARRYINFO.WORK_TYPE
- RETRIEVAL_DETAIL  = DNCARRYINFO.RETRIEVAL_DETAIL
- WORK_NO           = DNCARRYINFO.WORK_NO
- PALLET_ID         = DNPALLET.PALLET_ID
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY
- RESTORING_FLAG    = DNCARRYINFO.RESTORING_FLAG
- WORK_DAY          = DMWARENAVISYSTEM.WORK_DAY
- REMOVE_FLAG       = 00:Normal
- REGIST_DATE       = SYSTIMESTAMP
- REGIST_PNAME      = Class name
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTOCKHISTORY
- WORK_DAY              = DMWARENAVISYSTEM.WORK_DAY
- INC_DEC_TYPE          = 1: Stock Increase
- JOB_TYPE              = 2: Storage
- UPDATE_STOCK_QTY      = DNSTOCK.STOCK_QTY
- INC_DEC_QTY           = DNSTOCK.STOCK_QTY
- STOCK_ID              = DNSTOCK.STOCK_ID
- AREA_NO               = DNSTOCK.AREA_NO
- LOCATION_NO           = DNSTOCK.LOCATION_NO
- STORAGE_DAY           = DNSTOCK.STORAGE_DAY
- STORAGE_DATE          = DNSTOCK.STORAGE_DATE
- NEWEST_STORAGE_DATE   = DNSTOCK.NEWEST_STORAGE_DATE
- PALLET_ID             = DNSTOCK.PALLET_ID
- BCR_DATA              = DNPALLET.BCR_DATA
- AREA_TYPE             = DMAREA_AREA_TYPE
- USER_ID               = Login info
- USER_NAME             = Login info
- TERMINAL_NO           = Login info
- TERMINAL_NAME         = Login info
- IP_ADDRESS            = Login info
- REGIST_DATE           = SYSTIMESTAMP
- REGIST_PNAME          = Class name
- LAST_UPDATE_DATE      = SYSTIMESTAMP
- LAST_UPDATE_PNAME     = Class name

# ID64

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

Upon equipment **(STV)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## DNCARRYINFO
- CMD_STATUS         = 4:Pickup completed
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

# ID26

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
DNWORKINFO
DNSTOCK
DNWORKLIST
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process **storage**, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## DNARRIVAL
- ARRIVAL_DATE      = SYSTIMESTAMP 
- STATION_NO        = Arrival Station Number from ID26 
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY       
- BCR_DATA          = Barcode information from ID26
- CONTROLINFO       = Control information from ID26
- SEND_FLAG         = 0:Not sent
- HEIGHT            = Dimension Information from ID26
- WIDTH             = Dimension Information From ID26
- REGIST_DATE       = SYSTIMESTAMP                                                    
- REGIST_PNAME      = ClassName
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNCARRYINFO
- PALLET_ID         = DNPALLET.PALLET_ID
- WORK_TYPE         = 2: Storage
- CMD_STATUS        = 1:Started 
- PRIORITY          = 2:Normal
- CARRY_FLAG        = 1: Storage
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO ⟶ **(7207/7208/7209/7210)**
- DEST_STATION_NO   = Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9007/9008/9009/9010)**
- END_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNPALLET                                                     
- CURRENT_STATION_NO = DNARRIVAL.STATION_NO                                                                                                                                                
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNWORKINFO
- JOB_TYPE          = 02:Storage
- STATUS_FLAG       = 1: Working
- PLAN_AREA_NO      = Area Number where a reserved location belongs to
- PLAN_LOCATION_NO  = Location Number where a reserved location belongs to
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNSTOCK
- AREA_NO           = DNWORKINFO.PLAN_AREA_NO
- LOCATION_NO       = DNWORKINFO.PLAN_LOCATION_NO
- PLAN_QTY          = DNWORKINFO.PLAN_QTY
- PALLET_ID         = DNPALLET.PALLET_ID
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNWORKLIST
- JOB_TYPE          = DNWORKINFO.JOB_TYPE
- PLAN_AREA_NO      = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO  = DNWORKINFO.PLAN_LOCATION_NO
- SOURCE_STATION_NO = DNCARRYINFO.SOURCE_STATION_NO
- DEST_STATION_NO   = DNCARRYINFO.DEST_STATION_NO
- END_STATION_NO    = DNCARRYINFO.END_STATION_NO
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

# Storage Sender

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
DMSTATION
")]

id05msg("
ID 05
")

storageSender-input-->storageSender-->id05msg
storageSender--> |UPDATE| storageSender-update
:::

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender ` &nbsp;</span>

After successful creation of arrival record in **ID26process**, Automatic Mode Change Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**

## DMWAREHOUSE
- LAST_USED_STATION_NO_EP = Aisle Number where a reserved location belongs to
- LAST_UPDATE_DATE        = SYSTIMESTAMP
- LAST_UPDATE_PNAME       = Class name 

## DMSHELF
- STATUS_FLAG      = 2:Reserved Location
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNCARRYINFO
- AISLE_STATION_NO  = Aisle Number where a reserved location belongs to
- CMD_STATUS        = 2:Waiting for response
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNWORKINFO
- PLAN_LOCATION_NO = Reserved Location Number
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNPALLET
- CURRENT_STATION_NO = Reserved Location Number
- WH_STATION_NO      = DNCARRYINFO.END_STATION_NO
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNSTOCK
- AREA_NO           = DNCARRYINFO.END_STATION_NO
- LOCATION_NO       = Reserved Location Number
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNARRIVAL
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY
- SEND_FLAG         = 1:Sent
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID25

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
DNPALLET
DNWORKINFO
DNINOUTRESULT
DNSTOCKHISTORY
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

## DNCARRYINFO
- CMD_STATUS          = 3:Commanded
- ERROR_CODE          = 0
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

## DNPALLET                                                                                                    
- EMPTY_FLAG        = 1:Empty Pallet                                               
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNWORKINFO
- STATUS_FLAG        = 4: Completed
- RESULT_AREA_NO     = DNWORKINFO.PLAN_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- RESULT_QTY         = DNWORKINFO.PLAN_QTY

## DNSTOCK
- NEWEST_STORAGE_DATE = DNSTOCK.NEWEST_STORAGE_DATE
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

## DNINOUTRESULT
- STATION_NO        = DNCARRYINFO.SOURCE_STATION_NO
- LOCATION_NO       = DNCARRYINFO.DEST_STATION_NO
- AISLE_STATION_NO  = DNCARRYINFO.AISLE_STATION_NO
- WORK_TYPE         = DNCARRYINFO.WORK_TYPE
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTOCKHISTORY
- AREA_NO             = DNSTOCK.AREA_NO
- LOCATION_NO         = DNSTOCK.LOCATION_NO
- NEWEST_STORAGE_DATE = DNSTOCK.NEWEST_STORAGE_DATE
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

# ID64

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

Upon equipment **(SRM)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## DNCARRYINFO
- CMD_STATUS          = 4:Pickup completed
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

# ID33

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id33` &nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-update[("
DNPALLET
DMSHELF
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--> |UPDATE| id33-update
id33process--> |DELETE| id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

## DMSHELF
- STATUS_FLAG         = 1: Occupied
- LAST_UPDATE_DATE    = SYSTIMESTAMP

## DNPALLET
- CURRENT_STATION_NO  = Location Number information from ID33
- STATUS_FLAG         = 2:Occupied
- ALLOCATION_FLAG     = 0:Not allocated
- LAST_STORED_DATE    = SYSTIMESTAMP
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

#User Story
- [DFD Storage Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5787)

# Related DFD
  - {}
