[[_TOC_]]
[[_TOSP_]]

# Transfer Setting Ambient Area to Tempering Area
![image.png](/.attachments/image-c5758bc4-cf12-4686-ada3-9bdb5b9b65aa.png)
![image.png](/.attachments/image-73207ff0-617e-4e69-88df-40df91dd8d4d.png)

# <span style="color:skyblue; font-weight:bold">Summary Flow</span>
## Stage 1 - Normal Retrieval
::: mermaid
flowchart LR

P1[FROM STATION 9007 - 9014]-->P2[RetrievalSender]-->P3[ID12]-->P4[ID32]-->|SRM|P5[ID33]-->Cond1{Flow ?} 
:::

## Stage 2 - Deposit from station to SRM
::: mermaid
flowchart LR 
Cond1{Flow ?}-->|STV HP|P1[ID64]-->|7207-7210|P2[ID26]-->P3[StorageSender]-->P4[ID05]-->P5[ID25]-->|SRM 9007-9010|P6[ID64]
Cond1{Flow ?}-->|STV OP|P7[ID64]-->|7101-7106|P8[ID26]-->P9[StorageSender]-->P10[ID05]-->P11[ID25]-->|SRM 9001-9006|P6[ID64]

P6[ID64]-->Cond2{Continue Deposit to ?}

:::

## Stage 3
::: mermaid
flowchart LR 
Cond1{Continue Deposit to ?}-->|Through SRM 9007-9010|P1[Continue Go to Stage 2 - 
STV OP]
Cond1{Continue Deposit to ?}-->|SRM 9001-9006|P2[ID33]

:::

#<span style="color:skyblue; font-weight:bold">Transfer Setting from Ambient to Tempering</span>
## Abbreviation
- **PLLT** : DNPALLET          
- **WRKI** : DNWORKINFO       
- **WRKL** : DNWORKLIST        
- **CRYI** : DNCARRYINFO       
- **STCK** : DNSTOCK           
- **ARVL** : DNARRIVAL        
- **WRHS** : DMWAREHOUSE      
- **SHLF** : DMSHELF          
- **STCH** : DNSTOCKHISTORY   
- **INOT** : DNINOUTRESULT    
- **HTST** : DNHOSTSEND       
- **OPRR** : DNOPERATIONRESULT
- **ITEM** : DMITEM           
- **STSN** : DMSTATION    
    
- **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                    | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | HTST | OPRR | ITEM | STSN |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Transfer Setting - Set(F2) [(1)](#transfer-setting---set(f2))  |   S  |   I  |   I  |      |      |      |   S  |   S  |      |      |      |      |   S  |   S  |
| **Normal Retrieval**                                           |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| Retrieval Sender [(2)](#retrieval-sender)                      |   U  |   U  |      |   I  |      |      |      |      |      |      |      |      |      |      |
| ID12 [(3)](#id12)                                              |   U  |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID32 [(4)](#id32)                                              |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID33 at SRM Retrieval [(5)](#id33-at-srm-retrieval)            |      |      |      |   U  |      |      |      |  U   |      |      |      |      |      |      |
| **Flow 1 - Storage from crane (9011-9014) through crane (9007-9010)**|      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID64 at STV HP [(6)](#id64-at-stv-hp)                          |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID26 at 7207-7210[(7)](#id26-at-7207-7210)                     |   U  |   U  |      |   U  |      |   I  |      |      |      |      |      |      |      |      |
| StorageSender at 7207-7210[(8)](#StorageSender-at-7207-7210)   |      |   U  |      |   U  |      |   U  |   U  |   U  |      |      |      |      |      |      |
| ID25 at 7207-7210[(9)](#ID25-at-7207-7210)                     |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |      |
| ID64 at SRM 9007-9010 [(10)](#id64-at-SRM-9007-9010)           |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| [Continue to Flow 2](#flow-2-%3A--direct-storage-from-crane-(9007-9010)-to-tempering-(9001-9006))|      |      |      |      |      |      |      |      |      |      |      |      |      |      |  
| **Flow 2 - Direct storage from crane (9007-9010) to Tempering (9001-9006)**|      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID64 at STV OP [(11)](#id64-at-stv-op)                          |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID26 at 7101-7106[(12)](#id26-at-7101-7106)                     |   U  |   U  |      |   U  |      |   I  |      |      |      |      |      |      |      |      |
| StorageSender at 7101-7106 [(13)](#StorageSender-at-7101-7106)  |      |   U  |      |   U  |      |   U  |   U  |   U  |      |      |      |      |      |      |
| ID25 at 7101-7106 [(14)](#ID25-at-7101-7106)                    |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |      |
| ID64 at SRM 9001-9006 [(15)](#id64-at-SRM-9001-9006)            |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |  
| ID33 at 9001-9006 [(16)](#id33-at-9001-9006)                    |   U  |   U  |      |   D  |   U  |      |      |  U   |   I  |      |      |      |      |      |                  



# Transfer Setting - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.transfersetting.TransferSettingSCH` &nbsp;</span>

![image.png](/.attachments/image-e54f05df-ae09-4220-9c06-4587ada90f7d.png)
This screen is mainly used to move pallets from Ambient to Tempering.

## <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole process Transfer Setting
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Stock Status must be **UU**
- Tempering Flag must be **REACHED**
- QC Check Flag must be **DONE**
- Storage Location filtered with **Ambient** value

::: mermaid
flowchart LR
input[
Area
Loc #
Pallet #
Material Code
Batch #
Material Name
Stock Qty
Stock Status
Tempering Flag
Storage Date/Time
Tempering Period
QC Duration
QC Check Flag
]

tableList-insert[("
DNWORKINFO
DNWORKLIST
")]

tableList-select[("
DMWAREHOUSE
DMSHELF
DNPALLET
DMITEM
DMSTATION
")]


className[TransferSettingSCH]

input --> className --> |INSERT| tableList-insert
tableList-select --> |SELECT| className
:::

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
- **JOB_NO** : Sequence Object    
- **SETTING_UNIT_KEY** : Sequence Object    
- **COLLECT_JOB_NO** : Sequence Object    
- **JOB_TYPE** : 03: Retrieval   
- **STATUS_FLAG** : 1: Working
- **HARDWARE_TYPE** : 3: ASRS    
- **PLAN_UKEY** : Sequence Object    
- **STOCK_ID** : DNSTOCK.STOCKID    
- **PLAN_AREA_NO** :  DNSTOCK.AREA_NO   
- **PLAN_LOCATION_NO** :  DNSTOCK.LOCATION_NO
- **PLAN_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **VENDOR_CODE** : DNSTOCK.VENDOR_CODE
- **VENDOR_NAME** : DNSTOCK.VENDOR_NAME
- **COMPANY_CODE** : DNSTOCK.COMPANY_CODE    
- **BATCH_NO** : Value from screen (**Batch #**)    
- **MATERIAL_CODE** : Value from screen (**Material Code**)    
- **MATERIAL_NAME** : Value from screen (**Material Name**)    
- **PLAN_QTY** : Value from screen (**Stock Qty**)     
- **UOM** : Value from screen (**UOM**)    
- **TEMPERING_PERIOD** : Value from screen (**Tempering Period**)    
- **EXPIRY_DAYS** : DNSTOCK.EXPIRY_DATE    
- **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY  
- **BCR_DATA** : DNPALLET.BCR_DATA  
- **USER_ID** : Login Info     
- **TERMINAL_NO** : Login info
- **REGIST_DATE** : SYSTIMESTAMP    
- **REGIST_PNAME** : ClassName    
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNWORKLIST</span>
- **JOB_NO**: DNWORKINFO.JOB_NO    
- **SETTING_UNIT_KEY**: Sequence Object   
- **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
- **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
- **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
- **STOCK_ID**: DNWORKINFO.STOCK_ID    
- **PALLET_ID**: DNPALLET.PALLET_ID    
- **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
- **BATCH_NO**: DNWORKINFO.BATCH_NO    
- **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
- **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
- **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME      
- **RETRIEVAL_DETAIL**: 1:Unit Retrieval   
- **PLAN_QTY**: DNWORKINFO.PLAN_QTY  
- **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
- **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
- **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
- **QC_DURATION**: DNSTOCK.QC_DURATION
- **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG        
- **DOCK_NO**: DNWORKINFO.DOCK_NO
- **TRUCK_PLATE_NO**: DNWORKINFO.TRUCK_PLATE_NO 
- **USER_ID**: DNWORKINFO.USER_ID    
- **USER_NAME**: DCUSER.USER_NAME    
- **REGIST_DATE**: SYSTIMESTAMP    
- **REGIST_PNAME**: ClassName    
- **LAST_UPDATE_DATE**: SYSTIMESTAMP    
- **LAST_UPDATE_PNAME**: ClassName

# Retrieval Sender

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.transmission.RetrievalSender` &nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
")]

retrievalsender-update[("
DNPALLET
DNWORKINFO
")]

retrievalsender-insert[("
DNCARRYINFO
")]

id12msg("
ID 12
")

retrievalsender-input-->RetrievalSender--> |UPDATE| retrievalsender-update
RetrievalSender--> |INSERT| retrievalsender-insert
RetrievalSender--> |SendText| id12msg
:::


The Retrieval operation at **Ambient Area (9002: Ambient)** will be retrieved to Station 7101, 7102, 7103, 7104, 7105, 7207, 7208, 7209, 7210 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 3:Reserved for Retrieval
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CARRY_KEY** : Sequence Object    
- **PALLET_ID** : DNSTOCK.PALLET_ID    
- **WORK_TYPE** : 03: Retrieval   
- **CMD_STATUS** : 1:Started    
- **PRIORITY** : 2:Normal    
- **RESTORING_FLAG** : 0 :Not Restore to Original Location  
- **WORK_NO** : Sequence Object    
- **RETRIEVAL_STATION_NO** : DNSTOCK.LOCATION_NO
- **RETRIEVAL_DETAIL** : 1: Unit Retrieval
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO ⟶ **(9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014)**    
- **DEST_STATION_NO** : Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(7101, 7102, 7103, 7104, 7105, 7106, 7207, 7208, 7209, 7210)**
- **CANCEL_REQUEST** : 0: Not Requested    
- **SCHEDULE_NO** : Sequence Object    
- **CARRY_FLAG** : 2: Retrieval
- **CANCEL_REQUEST** : 0: Not requested
- **AISLE_STATION_NO** : DMSHELF.PARENT_STATION_NO
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO  
- **REGIST_DATE** : SYSTIMESTAMP    
- **REGIST_PNAME** : ClassName    
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>   
- **STATUS_FLAG**: 1:Working  
- **SYSTEM_CONN_KEY**: DNCARRYINFO.CARRY_KEY

#ID12

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.send.As21Id12` &nbsp;</span>

::: mermaid
flowchart LR

id12msg("
ID12
")

id12-update[("
DNPALLET
DNCARRYINFO
")]

retrievalstationoperator[RetrievalStationOperator]

id12msg-->id12process
id12process-->retrievalstationoperator--> |UPDATE| id12-update
:::

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 2:Waiting for response
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 4:Being retrieved
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID32

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id32Proces` &nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process-.UPDATE.->id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 3: Commanded
- **ERROR_CODE** : 0
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID33 at SRM Retrieval
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process` &nbsp;</span>

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

## <span style="color:skyblue; font-weight:bold">DMSHELF</span>
- **STATUS_FLAG** : 0:Empty
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 5:Retrieval completed
- **CARRY_FLAG** : 5:Location-to-location Move
- **RETRIEVAL_STATION_NO** : DMSHELF.STATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ##[Flow 1 : Storage from crane (9011-9014) through crane (9007-9010)](#flow-1-%3A-storage-from-crane-(9011-9014)-through-crane-(9007-9010))
- ##[Flow 2 : Direct storage from crane (9007-9010) to Tempering (9001-9006)](#flow-2-%3A--direct-storage-from-crane-(9007-9010)-to-tempering-(9001-9006))

<hr>

#Flow 1 : Storage from crane (9011-9014) through crane (9007-9010)

# ID64 at STV HP

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

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID26 at 7207-7210

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
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process **storage**, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
- **ARRIVAL_DATE** : SYSTIMESTAMP 
- **STATION_NO** : Arrival Station Number from ID26 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY       
- **BCR_DATA** : Barcode information from ID26
- **CONTROLINFO** : Control information from ID26
- **SEND_FLAG** : 0:Not sent
- **HEIGHT** : Dimension Information from ID26
- **WIDTH** : Dimension Information From ID26
- **REGIST_DATE** : SYSTIMESTAMP                                                    
- **REGIST_PNAME** : ClassName
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **PALLET_ID** : DNPALLET.PALLET_ID
- **WORK_TYPE** : 2: Storage
- **CMD_STATUS** : 1:Started 
- **PRIORITY** : 2:Normal
- **CARRY_FLAG** : 1: Storage
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO ⟶ **(7207, 7208, 7209, 7210)**
- **DEST_STATION_NO** : Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(7101, 7102, 7103, 7104, 7105, 7106)**
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>                                                     
- **CURRENT_STATION_NO** : DNARRIVAL.STATION_NO
- **WH_STATION_NO** : DNCARRYINFO.END_STATION_NO                                                                                                                                                
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

# StorageSender at 7207-7210

::: mermaid
flowchart LR
storageSender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNWORKINFO
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

After successful creation of arrival record in **ID26 Process**, Storage Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

## <span style="color:skyblue; font-weight:bold">DMWAREHOUSE</span> 
- **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name 

## <span style="color:skyblue; font-weight:bold">DMSHELF</span> 
- **STATUS_FLAG** : 2:Reserved Location
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to
- **CMD_STATUS** : 2:Waiting for response
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span> 
- **JOB_TYPE** : 02:Storage
- **STATUS_FLAG** : 1: Working
- **PLAN_AREA_NO** : Area Number where a reserved location belongs to
- **PLAN_LOCATION_NO** : Location Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span> 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY
- **SEND_FLAG** : 1:Sent
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID25 at 7207-7210

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

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 3:Commanded
- **ERROR_CODE** : 0
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID64 at SRM 9007-9010

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

Upon equipment **(SRM 9007-9010)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

<br>
<hr style="width:50%; margin-left: 0; border: 2px solid green;">

**Continue Process with**: [Flow 2 : Direct storage from crane (9007-9010) to Tempering (9001-9006)](#flow-2-%3A--direct-storage-from-crane-(9007-9010)-to-tempering-(9001-9006))
<hr style="width:50%; margin-left: 0; border: 2px solid green;">


<hr>

#Flow 2 :  Direct storage from crane (9007-9010) to Tempering (9001-9006)

# ID64 at STV OP

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

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID26 at 7101-7106

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
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process **storage**, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
- **ARRIVAL_DATE** : SYSTIMESTAMP 
- **STATION_NO** : Arrival Station Number from ID26 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY       
- **BCR_DATA** : Barcode information from ID26
- **CONTROLINFO** : Control information from ID26
- **SEND_FLAG** : 0:Not sent
- **HEIGHT** : Dimension Information from ID26
- **WIDTH** : Dimension Information From ID26
- **REGIST_DATE** : SYSTIMESTAMP                                                    
- **REGIST_PNAME** : ClassName
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **PALLET_ID** : DNPALLET.PALLET_ID
- **WORK_TYPE** : 2: Storage
- **CMD_STATUS** : 1:Started 
- **PRIORITY** : 2:Normal
- **CARRY_FLAG** : 1: Storage
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO ⟶ **(7101, 7102, 7103, 7104, 7105, 7106)**
- **DEST_STATION_NO** : Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9001, 9002, 9003, 9004, 9005, 9006)**
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>                                                     
- **CURRENT_STATION_NO** : DNARRIVAL.STATION_NO
- **WH_STATION_NO** : DNCARRYINFO.END_STATION_NO                                                                                                                                                
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

# StorageSender at 7101-7106

::: mermaid
flowchart LR
storageSender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNWORKINFO
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

After successful creation of arrival record in **ID26 process**, Storage Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

## <span style="color:skyblue; font-weight:bold">DMWAREHOUSE</span> 
- **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name 

## <span style="color:skyblue; font-weight:bold">DMSHELF</span> 
- **STATUS_FLAG** : 2:Reserved Location
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to
- **CMD_STATUS** : 2:Waiting for response
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span> 
- **JOB_TYPE** : 02:Storage
- **STATUS_FLAG** : 1: Working
- **PLAN_AREA_NO** : Area Number where a reserved location belongs to
- **PLAN_LOCATION_NO** : Location Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span> 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY
- **SEND_FLAG** : 1:Sent
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID25 at 7101-7106

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

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 3:Commanded
- **ERROR_CODE** : 0
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID64 at SRM 9001-9006

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

Upon equipment **(SRM 9001-9006)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID33 at 9001-9006

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id33` &nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNSTOCKHISTORY
")]

id33-update[("
DNSTOCK
DNPALLET
DMSHELF
DNWORKINFO
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--INSERT-->id33-insert
id33process-.UPDATE.->id33-update
id33process--DELETE--xid33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

## <span style="color:skyblue; font-weight:bold">DNSTOCK</span>
- **NEWEST_STORAGE_DATE**: SYSTIMESTAMP
- **LOCATION_NO** : DNPALLET.CURRENT_STATION_NO
- **LAST_UPDATE_DATE**: SYSTIMESTAMP
- **LAST_UPDATE_PNAME**: Class name 

## <span style="color:skyblue; font-weight:bold">DMSHELF</span> 
- **STATUS_FLAG** : 1: Occupied
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNPALLET</span> 
- **CURRENT_STATION_NO** : Location Number information from ID33
- **STATUS_FLAG** : 2:Occupied
- **ALLOCATION_FLAG** : 0:Not allocated
- **LAST_STORED_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span> 
- **RESULT_QTY**: DNWORKINFO.PLAN_QTY
- **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
- **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
- **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
- **STATUS_FLAG**: 4: Completed
- **LAST_UPDATE_DATE**: SYSTIMESTAMP
- **LAST_UPDATE_PNAME**: Class name

## <span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY</span> 
- **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **INC_DEC_TYPE** : 1: Stock Increase
- **JOB_TYPE** : 2: Storage
- **STOCK_ID** : DNSTOCK.STOCK_ID
- **AREA_NO** : DNSTOCK.AREA_NO
- **LOCATION_NO** : DNSTOCK.LOCATION_NO
- **MATERIAL_CODE** : DNSTOCK.MATERIAL_CODE
- **MATERIAL_NAME** : DMITEM.MATERIAL_NAME
- **STORAGE_DAY** : DNSTOCK.STORAGE_DAY
- **STORAGE_DATE** : DNSTOCK.STORAGE_DATE
- **NEWEST_STORAGE_DATE** : DNSTOCK.NEWEST_STORAGE_DATE
- **UPDATE_STOCK_QTY**: DNSTOCK.STOCK_QTY
- **INC_DEC_QTY** : DNSTOCK.STOCK_QTY
- **PALLET_ID** : DNSTOCK.PALLET_ID
- **AREA_TYPE** : DMAREA_AREA_TYPE
- **USER_ID** : Login info
- **USER_NAME** : Login info
- **TERMINAL_NO** : Login info
- **TERMINAL_NAME** : Login info
- **IP_ADDRESS** : Login info
- **STOCK_STATUS** : DNSTOCK.STOCK_STATUS
- **TEMPERING_FLAG** : DNSTOCK.TEMPERING_FLAG
- **QC_FLAG** : DNSTOCK.QC_FLAG
- **EXPIRY_DATE** : DNSTOCK..EXPIRY_DAYS
- **REGIST_DATE** : SYSTIMESTAMP
- **REGIST_PNAME** : Class name

# User Story
  - #5863

# Related DFD
  - {}
