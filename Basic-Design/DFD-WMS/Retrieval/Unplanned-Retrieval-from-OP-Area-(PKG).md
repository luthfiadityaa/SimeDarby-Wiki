[[_TOC_]]
[[_TOSP_]]

#Retrieval process station layout of Packaging Material
![image.png](/.attachments/image-548d8561-0d1f-408d-8c12-683cbc1b3927.png)

# Summary Flow
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1201, 1202, 1203, 1204] 
:::

# Unplanned Retrieval from OP Area (PKG)
## Abbreviation
| **CODE** | TABLE NAME       |
|----------|------------------|
| **RTPN** | DNRETRIEVALPLAN  | 
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
| Action Name                                                    | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT  | HTSD | ITEM | STSN |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Inquiry Retrieval - Set (F2) [(1)](#inquiry-retrieval---set-(f2))|      |   I  |      |   I  |      |      |      |      |      |      |      |      |      |
| Retrieval Sender [(2)](#retrieval-sender)                      |   U  |      |      |   U  |      |      |      |      |      |      |      |      |      |

# Inquiry Retrieval - Set (F2)

Inquiry Retrieval Setting (PKG) is used for manually pickup Packaging Material item in ASRS. 
![image.png](/.attachments/image-bbf9a08f-5425-40ef-bb6b-8a9991a6e82a.png)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.storage.palletizingsetting.InquiryRetrievalSettingSCH` &nbsp;</span>

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


className[EmptyPalletSettingSCH]

input --> className --> |INSERT| tableList-insert
tableList-select --> |SELECT| className
:::

## Validations
This section explains the validations for the whole proccess Unplanned Retrieval process (PKG)
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##  DNWORKINFO
- JOB_NO = Sequence Object    
- SETTING_UNIT_KEY = Sequence Object    
- COLLECT_JOB_NO = Sequence Object    
- JOB_TYPE = 23: Unplanned Retrieval    
- STATUS_FLAG = 0: Not Started    
- PLAN_UKEY = Sequence Object    
- STOCK_ID = DNSTOCK.STOCKID    
- PLAN_AREA_NO = DNSTOCK.AREA_NO   
- PLAN_LOCATION_NO = DNSTOCK.LOCATION_NO
- PLAN_DAY = DMWARENAVISYSTEM.WORK_DAY    
- BATCH_NO = Value from screen (**Batch #**)    
- MATERIAL_CODE = Value from screen (**Material Code**)    
- MATERIAL_NAME = Value from screen (**Material Name**)    
- PLAN_QTY = Value from screen (**Retrieval Qty**)    
- QTY_KG_CRTN = Value from screen (**Qty kg/crtn**)    
- QTY_CRTN_PL = Value from screen (**Qty crtn/PL**)    
- UOM = Value from screen (**UOM**)    
- TEMPERING_PERIOD = Value from screen (**Tempering Period**)    
- EXPIRY_DAYS = DNSTOCK.EXPIRY_DATE    
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY  
- BCR_DATA = DNPALLET.BCR_DATA  
- USER_ID = Login Info     
- TERMINAL_NO = Login info
- REGIST_DATE = SYSTIMESTAMP    
- REGIST_PNAME = ClassName    
- LAST_UPDATE_DATE = SYSTIMESTAMP    
- LAST_UPDATE_PNAME = ClassName

## DNCARRYINFO
- CARRY_KEY = Sequence Object    
- PALLET_ID = DNSTOCK.PALLET_ID    
- WORK_TYPE = 23:Unplanned Retrieval    
- CMD_STATUS = 1:Started    
- PRIORITY = 2:Normal    
- RESTORING_FLAG = 0 : Not Restore to Original Location  
- WORK_NO = Sequence Object    
- RETRIEVAL_STATION_NO = DNSTOCK.LOCATION_NO
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO    
- DEST_STATION_NO = Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(1201, 1202, 1203, 1204**)
- PRIORITY = Value from screen **(Dropdown ⟶ 1:Urgent / 2: Normal)**
- CANCEL_REQUEST = 0 =Not Requested    
- SCHEDULE_NO = Sequence Object    
- CARRY_FLAG = 2 =Retrieval
- CANCEL_REQUEST = 0 =Not requested
- AISLE_STATION_NO = DMSHELF.PARENT_STATION_NO
- END_STATION_NO = DNCARRYINFO.DEST_STATION_NO  
- REGIST_DATE = SYSTIMESTAMP    
- REGIST_PNAME = ClassName    
- LAST_UPDATE_DATE = SYSTIMESTAMP    
- LAST_UPDATE_PNAME = ClassName

# Retrieval Sender

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.transmission.RetrievalSender` &nbsp;</span>

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

retrievalsender--SEND-->id12msg
retrievalsender-input-->retrievalsender-.UPDATE.->retrievalsender-update
:::

The Retrieval operation at **Packaging Material zone (9002: Ambient)** will be retrieved to Station 1201, 1202, 1203, 1204 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

## DNCARRYINFO
- CMD_STATUS = 2:Waiting for response
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNPALLET
- STATUS_FLAG = 4:Being retrieved
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name




# User Story
  - #5776

# Related DFD
  - {}
