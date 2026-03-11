[[_TOC_]]
[[_TOSP_]]

# Planned Storage Setting (PKG) database flow

## Abbreviation
| **CODE** | TABLE NAME       |
|----------|------------------|
| **STRP** | DNSTORAGEPLAN    | 
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
| **HTSD** | DNHOSTSEND       |
| **ITEM** | DMITEM           |
| **STSN** | DMSTATION        |

| **CODE** | OPERATION NAME   |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |

## Inbound Table Data Flow
| Action Name                                                    | STRP | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT  | HTSD | ITEM | STSN |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Planned Storage from Host [(1)](#planned-storage-from-host)    |   I  |      |      |      |      |      |      |      |      |      |      |      |      |      |
| Planned Storage - Set (F2) [(2)](#planned-storage---set-(f2))  |   U  |      |      |      |      |      |      |      |      |      |      |      |   S  |   ~~S~~  |
| ID26 - Dummy Arrival [(3)](#id26---dummy-arrival)              |      |   I  |      |      |   I  |   I  |   I  |      |      |      |      |      |      |      |
| Storage Sender at 1106 [(4)](#storage-sender-at-1106)          |      |   U  |   U  |      |   U  |   U  |   U  |   U  |   U  |      |      |      |      |      |
| ID25 at 1106 [(5)](#id25-at-1106)                              |      |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |
| ID64 at STV [(5)](#id64-at-stv)                                |      |      |      |      |   U  |      |      |      |      |      |      |      |      |      |
| ID26 at 7211-7214 [(6)](#id26-at-7211-7214)                    |      |   U  |   U  |      |   U  |   U  |   I  |      |      |      |      |      |      |      |
| Storage Sender at 7211-7214 [(7)](#storage-sender-at-7211-7214)|      |   U  |   U  |      |   U  |   U  |   U  |   U  |   U  |      |      |      |      |      |
| ID25 at 7211-7214 [(8)](#id25-at-7211-7214)                    |      |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |
| ID64 at SRM [(9)](#id64-at-srm)                                |      |      |      |      |   U  |      |      |      |      |      |      |      |      |      |
| ID33 [(10)](#id33)                                             |   U  |   U  |   U  |      |   D  |   U  |      |      |   U  |   I  |   I  |   I  |      |      |



# Planned Storage from Host

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

# StoragePlanPkgDataLoader
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

## DNSTORAGEPLAN                                                                                                           
- STATUS_FLAG          = 0:Not Started                                                       
- CANCEL_FLAG          = 0:Normal Data                                                      
- PLAN_DAY             = Value from SAP (**Delivery Date**)                                                       
- VENDOR_CODE          = Value from SAP (**Vendor Code**)
- VENDOR_NAME          = Value from SAP (**Vendor Name**)                                                     
- COMPANY_CODE         = Value from SAP (**Company Code**)                                                      
- RECEIVE_TICKET_NO    = Value from SAP (**Purchase No**)                                                      
- RECEIVE_LINE_NO      = Value from SAP (**Item No / Line No**)                                                      
- RECEIVE_TICKET_DATE  = Value from SAP (**Document Date**)                                                                                                              
- PLAN_AREA_NO         = Value from SAP (**Plant**)                                                      
- PLAN_LOCATION_NO     = DMWAREHOUSE.Warehose_no                                                      
- MATERIAL_CODE        = Value from SAP (**Material Code**)     
- UOM                  = Value from SAP (**UOM**)                                                                                                            
- REPORT_FLAG          = 0:Not Reported                                                                                                           
- REGIST_KIND          = 0:File Loading                                                      
- ERROR_INDICATION     = 0:Successfull
- TYPE                 = S: Success
- REGIST_DATE          = SYSTIMESTAMP                                                  
- REGIST_PNAME         = ClassName
- LAST_UPDATE_DATE     = SYSTIMESTAMP
- LAST_UPDATE_PNAME    = ClassName

# Planned Storage - Set (F2)
![image.png](/.attachments/image-731303e9-dc65-4f98-ba82-ba6d13a6c58b.png)
Planned Storage Setting (PKG) is used to set the information of stock which will be entered into ASRS. After **Set(F2)** all item in input text will be process and The result will be posted back to SAP as [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.storage.plannedstoragepkg.PlannedStoragePkgSCH` &nbsp;</span>

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
        DMITEM
        DMSTATION
    ")]

    tableList-update[("
        DNSTORAGEPLAN
    ")]

    className[PlannedStoragePkgSCH]

    input --> className--> |UPDATE| tableList-update
    tableList-select--> |SELECT| className 

:::

## Relantionship between Storage Plan and Pallet

One Storage Plan can have many Workinfo.
One workinfo is one pallet.

::: mermaid
erDiagram
    DNSTORAGEPLAN ||--o{ DNWORKINFO : "identifies"
    DNWORKINFO ||--|| DNPALLET : "mapped_by_bcr"

    STORAGEPLAN {
        string plan_ukey PK
    }

    WORKINFO {
        string plan_ukey FK
        string bcr_data PK
    }

    DNPALLET {
        string bcr_data PK
    }
	
:::

## Validations
This section explains the validations for the whole proccess Storage Packaging Material
- ~~AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)~~
- Material Code exists in **DMMaterialMaster**
- Material Code filtered with **MaterialType.ZPCK**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Pallet Information does not exist in **DNCARRYINFO** 
  To check for Pallet Information:  
  **JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number>**
  So if result > 0, Palletize Start cannot proceed.
- Storage Qty must be greater than **"0"**
- Stored Qty + Storage Qty cannot bigger than Planned Qty
- **Planned Qty and Stored Qty** are calculated fields (readonly).
- ~~Station (**ST1106**) is not suspended (**DMSTATION.SUSPEND.OFF**)~~
- ~~Station (**ST1106**) is not disconnected (**DMSTATION.STATUS_FLAG.ACTIVE**)~~

**Note:** All IN-stations can be used.

## DNWorkInfo
## DNPallet
Update 
## DNStock

## DNSTORAGEPLAN
- PLAN_UKEY         = Sequence Object                                                                                                             
- STATUS_FLAG       = 1:Working    **Very first pallet will update**                                                   
- CANCEL_FLAG       = 0:Normal Data                           
- PROCESS_QTY          = DNSTORAGEPLAN + DNWORKINGO.PLAN_QTY                                                                                
- ~~PLAN_QTY          = Value from screen (**Planned Qty**)~~
- ~~PROCESS_QTY       = Value from screen (**Storage Qty**)~~
- ~~RESULT_QTY        = Value from screen (**Stored Qty**)~~
- ~~REPORT_FLAG       = 0:Not Reported~~                                                      
- ~~WORK_DAY          = DMWARENAVISYSTEM.WORK_DAY~~                                                                                                            
- ~~BCR_DATA          = Value from screen (**Pallet #**)~~       
- ~~BATCH_NO          = Value from screen (**Batch #**)~~                                                   
- ~~STORING_PAIR_KEY  = Value from screen (**Material Code + Batch #**)~~  
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


# Storage Flow Process
This storage process flow is refer to AGC linkage Specification
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

# ID26 - Dummy Arrival

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
")]

carry-insert[("
DNCARRYINFO
DNPALLET
DNSTOCK
")]

plan-select[("
DNSTORAGEPLAN
")]

operator[StorageStationOperator]
scheduler[AutoStorageScheduler]

releaseCommand-->id26msg-->id26process-->operator--RMI-->scheduler
operator--> |INSERT| id26-insert
scheduler--> |SELECT| id26-insert
scheduler--> |SELECT| plan-select
scheduler--> |INSERT| carry-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let InOutStationOperator picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## DNARRIVAL
- ARRIVAL_DATE      = SYSTIMESTAMP 
- STATION_NO        = Arrival Station Number from **ID26** 
- CARRY_KEY         = 99999999       
- BCR_DATA          = Barcode information from **ID26**
- CONTROLINFO       = Control information from **ID26**
- SEND_FLAG         = 0:Not sent
- HEIGHT            = Dimension Information from **ID26**
- WIDTH             = Dimension Information From **ID26**
- REGIST_DATE       = SYSTIMESTAMP                                                    
- REGIST_PNAME      = ClassName
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNCARRYINFO
- CARRY_KEY         = Sequence Object  
- PALLET_ID         = Sequence Object
- WORK_TYPE         = 26:Direct Transfer
- CMD_STATUS        = 1:Started 
- PRIORITY          = 2:Normal
- RESTORING_FLAG    = 0:Not Restore to Original Location
- CARRY_FLAG        = 1:Storage
- WORK_NO           = Sequence Object
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO ⟶ ~~**1106**~~
- DEST_STATION_NO   = Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (**7211/7212/7213/7214**)
- CANCEL_REQUEST    = 0:Not Requested
- SCHEDULE_NO       = Sequence Object
- END_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- REGIST_DATE       = SYSTIMESTAMP                                                    
- REGIST_PNAME      = ClassName
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNPALLET
- PALLET_ID          = Sequence Object                                                       
- CURRENT_STATION_NO = DNARRIVAL.STATION_NO ⟶ **1106**                                                        
- WH_STATION_NO      = 9002                                                      
- STATUS_FLAG        = 1:Reserved for Storage                                                      
- ALLOCATION_FLAG    = 1:Allocated                                                      
- EMPTY_FLAG         = 0:Normal Pallet                                                                                                              
- BCR_DATA           = Barcode information from **ID26**                                                     
- LAST_STORED_DATE   = SYSTIMESTAMP                                                                                                           
- REGIST_DATE        = SYSTIMESTAMP                                                    
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNSTOCK
- STOCK_ID           = Sequence Object
- AREA_NO            = DNWORKINFO.PLAN_AREA_NO
- LOCATION_NO        = DNWORKINFO.PLAN_LOCATION_NO
- ITEM_CODE          = DNWORKINFO.ITEM_CODE
- COMPANY_CODE       = DNWORKINFO.COMPANY_CODE
- VENDOR_CODE        = DNWORKINFO.VENDOR_CODE
- VENDOR_NAME        = DNWORKINFO.VENDOR_NAME
- STORAGE_TYPE       = 2:NEW
- STORAGE_DAY        = DMWARENAVISYSTEM.WORK_DAY
- STORAGE_DATE       = SYSTIMESTAMP
- NEWEST_STORAGE_DATE= DNSTOCK.STORAGE_DATE
- STOCK_QTY          = ~~DNWORKINFO.PROCESS_QTY + DNWORKINFO.RESULT_QTY~~ Now still 0
- PLAN_QTY           = DNWORKINFO.PLAN_QTY
- PALLET_ID          = DNPALLET.PALLET_ID
- REGIST_DATE        = SYSTIMESTAMP
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName



# Storage Sender at 1106

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender` &nbsp;</span>

::: mermaid
flowchart LR
storagesender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNPALLET
DNARRIVAL
DNSTOCK
DNWORKINFO
")]
storagesender-input[("
DNARRIVAL
DNCARRYINFO
DMSTATION
")]

id05msg("
ID 05
")

storagesender-input-->storagesender-->id05msg
storagesender--> |UPDATE| storagesender-update
:::

After successful creation of arrival record in **ID26process**, Automatic Mode Change Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

##  DMWAREHOUSE
- LAST_USED_STATION_NO_PM = Aisle Number where a reserved location belongs to
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

## DNSTOCK
- AREA_NO            = DNWORKINFO.PLAN_AREA_NO
- LOCATION_NO        = DNWORKINFO.PLAN_LOCATION_NO
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNWORKINFO
- STATUS_FLAG        = 1:Working
- PLAN_AREA_NO       = Area Number where a reserved location belongs to
- PLAN_LOCATION_NO   = Location Number where a reserved location belongs to
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

# ID25 at 1106

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

## DNCARRYINFO
- CMD_STATUS        = 3:Commanded
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID64 at STV

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
- CMD_STATUS          = 4:Pickup completed
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name


# ID26 at 7211-7214

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
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO ⟶ **(7211/7212/7213/7214)**
- DEST_STATION_NO   = Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9011/9012/9013/9014)**
- END_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNPALLET                                                     
- CURRENT_STATION_NO = DNARRIVAL.STATION_NO                                                                                                                                                
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

# Storage Sender at 7211-7214

::: mermaid
flowchart LR
storageSender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNWORKINFO
DNPALLET
DNSTOCK
DNARRIVAL
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

After successful creation of arrival record in **ID26process**, Automatic Mode Change Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

## DMWAREHOUSE
- LAST_USED_STATION_NO_PM = Aisle Number where a reserved location belongs to
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
- JOB_TYPE          = 02:Storage
- STATUS_FLAG       = 1: Working
- PLAN_AREA_NO      = Area Number where a reserved location belongs to
- PLAN_LOCATION_NO  = Location Number where a reserved location belongs to
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNPALLET
- CURRENT_STATION_NO = Reserved Location Number
- WH_STATION_NO      = DNCARRYINFO.END_STATION_NO
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNSTOCK
- AREA_NO           = DNWORKINFO.PLAN_AREA_NO
- LOCATION_NO       = DNWORKINFO.PLAN_LOCATION_NO
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNARRIVAL
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY
- SEND_FLAG         = 1:Sent
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID25 at 7211-7214

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

## DNCARRYINFO
- CMD_STATUS          = 3:Commanded
- ERROR_CODE          = 0
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name

# ID64 at SRM

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

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY
DNINOUTRESULT
")]

id33-update[("
DMSHELF
DNPALLET
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
id33process--x |DELETE| id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.



## DNSTOCKHISTORY
- WORK_DAY              = DMWARENAVISYSTEM.WORK_DAY
- INC_DEC_TYPE          = 1: Stock Increase
- JOB_TYPE              = 2: Storage
- STOCK_ID              = DNSTOCK.STOCK_ID
- AREA_NO               = DNSTOCK.AREA_NO
- LOCATION_NO           = DNSTOCK.LOCATION_NO
- AREA_TYPE             = DMAREA_AREA_TYPE ⟶ **1:AS/RS**
- MATERIAL_CODE         = DNSTOCK.MATERIAL_CODE
- MATERIAL_NAME         = DMITEM.ITEM_NAME
- STORAGE_DAY           = DNSTOCK.STORAGE_DAY
- STORAGE_DATE          = DNSTOCK.STORAGE_DATE
- NEWEST_STORAGE_DATE   = DNSTOCK.NEWEST_STORAGE_DATE
- UPDATE_STOCK_QTY      = DNSTOCK.STOCK_QTY
- INC_DEC_QTY           = DNSTOCK.STOCK_QTY
- PALLET_ID             = DNSTOCK.PALLET_ID
- USER_ID               = Login info
- USER_NAME             = Login info
- TERMINAL_NO           = Login info
- TERMINAL_NAME         = Login info
- IP_ADDRESS            = Login info
- REGIST_DATE           = SYSTIMESTAMP
- REGIST_PNAME          = Class name
- LAST_UPDATE_DATE      = SYSTIMESTAMP
- LAST_UPDATE_PNAME     = Class name

## DNINOUTRESULT
- RESULT_KIND       = 1:Storage(Stock+)
- STATION_NO        = DNCARRYINFO.SOURCE_STATION_NO
- WH_STATION_NO     = DNPALLET.WH_STATION_NO
- REMOVE_FLAG       = 00:Normal
- LOCATION_NO       = DNCARRYINFO.DEST_STATION_NO
- WORK_DAY          = DMWARENAVISYSTEM.WORK_DAY
- CARRY_KEY         = DNCARRYINFO.CARRY_KEY
- PALLET_ID         = DNPALLET.PALLET_ID
- WORK_TYPE         = DNCARRYINFO.WORK_TYPE
- CMD_STATUS        = DNCARRYINFO.CMD_STATUS
- CARRY_FLAG        = DNCARRYINFO.CARRY_FLAG
- RESTORING_FLAG    = DNCARRYINFO.RESTORING_FLAG
- WORK_NO           = DNCARRYINFO.WORK_NO
- SOURCE_STATION_NO = DNCARRYINFO.SOURCE_STATION_NO
- DEST_STATION_NO   = DNCARRYINFO.DEST_STATION_NO
- BCR_DATA          = DNPALLET.BCR_DATA
- SCHEDULE_NO       = DNCARRYINFO.SCHEDULE_NO
- AISLE_STATION_NO  = DNCARRYINFO.AISLE_STATION_NO
- END_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- REGIST_DATE       = SYSTIMESTAMP
- REGIST_PNAME      = Class name
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name



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

## DNWORKINFO
- WORK_DAY          = DMWARENAVISYSTEM.WORK_DAY
- STATUS_FLAG       = 4:Completed
- RESULT_*          = ** UPDATE ALL THE RESULT FIELDS USING PLAN DATA **
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

## DNSTOCK
- STOCK_QTY          = DNWORKINFO.RESULT_QTY
- NEWEST_STORAGE_DATE= SYSTIMESTAMP
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

## DNHOSTSEND
- WORK_DAY            = DNWORKINFO.WORK_DAY
- JOB_NO              = DNWORKINFO.JOB_NO
- COLLECT_JOB_NO      = DNWORKINFO.COLLECT_JOB_NO
- SETTING_UNIT_KEY    = DNWORKINFO.SETTING_UNIT_KEY
- JOB_TYPE            = DNWORKINFO.JOB_TYPE
- STATUS_FLAG         = 4:Completed
- HARDWARE_TYPE       = DNWORKINFO.HARDWARE_TYPE
- PLAN_UKEY           = DNWORKINFO.PLAN_UKEY
- STOCK_ID            = DNWORKINFO.STOCK_ID
- SYSTEM_CONN_KEY     = DNWORKINFO.SYSTEM_CONN_KEY
- PLAN_DAY            = DNWORKINFO.PLAN_DAY
- VENDOR_CODE         = DNWORKINFO.VENDOR_CODE
- VENDOR_NAME         = DNWORKINFO.VENDOR_NAME
- COMPANY_CODE        = DNWORKINFO.COMPANY_CODE
- BCR_DATA            = DNWORKINFO.BCR_DATA
- LOT_NO              = DNWORKINFO.RESULT_LOT_NO
- PLAN_AREA_NO        = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO    = DNWORKINFO.PLAN_LOCATION_NO
- ITEM_CODE           = DNWORKINFO.ITEM_CODE
- ITEM_NAME           = DMITEM.ITEM_NAME
- PLAN_QTY            = DNWORKINFO.PLAN_QTY
- RESULT_QTY          = DNWORKINFO.RESULT_QTY
- RESULT_AREA_NO      = DNWORKINFO.RESULT_AREA_NO
- RESULT_LOCATION_NO  = DNWORKINFO.RESULT_LOCATION_NO
- REPORT_FLAG         = 0:Not Reported
- USER_ID             = DNWORKINFO.USER_ID
- TERMINAL_NO         = DNWORKINFO.TERMINAL_NO
- WORK_SECOND         = DNWORKINFO.WORK_SECOND
- USER_NAME           = DCUSER.USERNAME
- REPORT_FLAG         = 1:Reported
- REGIST_DATE         = SYSTIMESTAMP
- REGIST_PNAME        = Class name
- LAST_UPDATE_DATE    = SYSTIMESTAMP
- LAST_UPDATE_PNAME   = Class name



-------------------------------------------------------------------------------

!!! **NEED A NEW SCREEN/FUNCTION TO CLOSE DNSTORAGEPLAN**

## DNSTORAGEPLAN
- STATUS_FLAG       = 4:Completed
- WORK_DAY          = DMWARENAVISYSTEM.WORK_DAY
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


##<span style="color:skyblue; font-weight:bold">[Planned Storage PKG - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/836/Planned-Storage-PKG)</span>


# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>

- [Planned Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/846/Planned-Storage-Result)
