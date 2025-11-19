[[_TOC_]]
[[_TOSP_]]

# Transfer Setting Tempering Area to Ambient Area
![image.png](/.attachments/image-2fb067b7-aa3e-40d4-b523-ccf656bd68d6.png)

# Summary
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 9002, 9003, 9004, 9005, 9006]-->P2[ID12]-->P3[RetrievalSender]-->P4[ID32]-->P5[ID33]-->P6[ID64]-->P7[ID26]-->P8[StorageSender]-->P9[ID05]-->P10[ID25]-->P11[ID64]-->P12[ID33]
:::

#<span style="color:skyblue; font-weight:bold">Transfer Setting from Tempering to Ambient</span>
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
| Transfer Setting - Set(F2) [(1)](#transfer-setting ---set(f2)) |   S  |   I  |   I  |      |      |      |   S  |   S  |      |      |      |      |   S  |   S  |
| ID12 [(2)](#id12)                                              |   U  |      |      |   I  |      |      |      |      |      |      |      |      |      |      |


# Transfer Setting - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.transfersetting.TransferSettingSCH` &nbsp;</span>

![image.png](/.attachments/image-e54f05df-ae09-4220-9c06-4587ada90f7d.png)
This screen is mainly used to move pallets from tempering to ambient.

## <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Unplanned Retrieval process (PKG)
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- Stock Status must be **UU**
- Tempering Flag must be **REACHED**
- QC Check Flag must be **DONE**

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
- **JOB_TYPE** : 45: Location-to-location   
- **STATUS_FLAG** : 1: Working
- **HARDWARE_TYPE** : 3: ASRS    
- **PLAN_UKEY** : Sequence Object    
- **STOCK_ID** : DNSTOCK.STOCKID    
- **PLAN_AREA_NO** : Value from screen (Dropdown ⟶ Area: **9001: Tempering**)    
- **PLAN_LOCATION_NO** : Value from screen (Dropdown ⟶ To_Location: **9002: Ambient**)  
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

#ID12

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.send.As21Id12` &nbsp;</span>

::: mermaid
flowchart LR

id12msg("
ID12
")

id12-insert[("
DNCARRYINFO
")]

id12-update[("
DNPALLET
")]

retrievalstationoperator[RetrievalStationOperator]

id12msg-->id12process
id12process-->retrievalstationoperator--> |INSERT| id12-insert
retrievalstationoperator--> |UPDATE| id12-update
:::

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 3:Reserved for Retrieval
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CARRY_KEY** : Sequence Object    
- **PALLET_ID** : DNSTOCK.PALLET_ID    
- **WORK_TYPE** : 23:Unplanned Retrieval    
- **CMD_STATUS** : 1:Started    
- **PRIORITY** : 2:Normal    
- **RESTORING_FLAG** : 0 :Not Restore to Original Location  
- **WORK_NO** : Sequence Object    
- **RETRIEVAL_STATION_NO** : DNSTOCK.LOCATION_NO
- **RETRIEVAL_DETAIL** : 1: Unit Retrieval
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO    
- **DEST_STATION_NO** : Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(1201, 1202, 1203, 1204)**
- **PRIORITY** : Value from screen **(Dropdown ⟶ 1:Urgent / 2: Normal)**
- **CANCEL_REQUEST** : 0: Not Requested    
- **SCHEDULE_NO** : Sequence Object    
- **CARRY_FLAG** : 2: Retrieval
- **CANCEL_REQUEST** : 0: Not requested
- **AISLE_STATION_NO** : DMSHELF.PARENT_STATION_NO
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO  
- **REGIST_DATE** : SYSTIMESTAMP    
- **REGIST_PNAME** : ClassName    
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    






# User Story
  - #5789

# Related DFD
  - {}
