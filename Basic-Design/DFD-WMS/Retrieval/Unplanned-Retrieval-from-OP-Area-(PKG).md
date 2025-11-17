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
| **OPRR** | DNOPERATIONRESULT|
| **ITEM** | DMITEM           |
| **STSN** | DMSTATION        |
| **TTSN** | DMTOSTATION      |

| **CODE** | OPERATION NAME   |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |

## Inbound Table Data Flow
| Action Name                                                    | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | HTSD | OPRR | ITEM | STSN | TTSN |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Inquiry Retrieval - Set (F2) [(1)](#inquiry-retrieval---set-(f2))|   S  |   I  |      |   I  |      |      |   S  |   S  |      |      |      |      |   S  |   S  |   S  | 
| Retrieval Sender [(2)](#retrieval-sender)                      |   U  |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID32 [(3)](#id32)                                              |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID33 [(4)](#id33)                                              |      |      |      |   U  |      |      |      |  U   |      |      |      |      |      |      |      |
| ID68 [(5)](#id68)                                              |      |      |      |      |      |      |      |      |      |      |      |   I  |      |      |      |
| ID26 [(6)](#id26)                                              |   D  |   U  |   I  |   D  |   D  |      |      |  U   |      |   I  |   I  |      |      |      |      |
| **Host Communication** [(7)](#host-communication)              |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |

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
- VENDOR_CODE = DNSTOCK.VENDOR_CODE
- VENDOR_NAME = DNSTOCK.VENDOR_NAME
- COMPANY_CODE = DNSTOCK.COMPANY_CODE    
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
- RESTORING_FLAG = 0 :Not Restore to Original Location  
- WORK_NO = Sequence Object    
- RETRIEVAL_STATION_NO = DNSTOCK.LOCATION_NO
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO    
- DEST_STATION_NO = Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(1201, 1202, 1203, 1204**)
- PRIORITY = Value from screen **(Dropdown ⟶ 1:Urgent / 2: Normal)**
- CANCEL_REQUEST = 0: Not Requested    
- SCHEDULE_NO = Sequence Object    
- CARRY_FLAG = 2: Retrieval
- CANCEL_REQUEST = 0: Not requested
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
DNSTOCK
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

## DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID33
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

## DMSHELF
- STATUS_FLAG = 0:Empty
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNCARRYINFO
- CMD_STATUS = 5:Retrieval completed
- RETRIEVAL_STATION_NO = DMSHELF.STATION_NO
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id68Process` &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-->id68process
id68process--> |INSERT| id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

## DNOPERATIONDISPLAY
- CARRY_KEY = MC Key information from ID68
- STATION_NO = Station information from ID68
- MATERIAL_CODE = DNWORKINFO.MATERIAL_CODE
- RETRIEVAL_QTY = DNWORKINFO.STOCK_QTY
- ARRIAL_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##LED DISPLAY
[Display information]
- Job Type
- Material Code
- Qty

![image.png](/.attachments/image-1e56dc6e-657e-47ff-9a47-575b04705583.png)

#ID26

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id26Process` &nbsp;</span>

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
DMSHELF
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

After the completion button flashes, the operator removes the pallet and presses the completion button to clear the operation indication. At the same time, sending ID 26 to the id26process, then delete related records from DNPALLET, DNCARRYINFO, and DNSTOCK.

## DNWORKINFO
- STATUS_FLAG = 4:Completed
- RESULT_AREA_NO = DNWORKINFO.PLAN_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- RESULT_LOT_NO = DNWORKINFO.PLAN_LOT_NO
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- USER_ID = Login info
- TERMINAL_NO = Login info
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DMSHELF
- STATUS_FLAG        = 0:Empty Location
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = Class name

## DNWORKLIST
- JOB_NO             = DNWORKINFO.JOB_NO
- CARRY_KEY          = DNWORKINFO.SYSTEM_CONN_KEY
- SETTING_UNIT_KEY   = DNWORKINFO.SETTING_UNIT_KEY
- COLLECT_JOB_NO     = DNWORKINFO.COLLECT_JOB_NO
- JOB_TYPE           = DNWORKINFO.JOB_TYPE
- PLAN_UKEY          = DNWORKINFO.PLAN_UKEY
- STOCK_ID           = DNWORKINFO.STOCK_ID
- PALLET_ID          = DNPALLET.PALLET_ID
- PLAN_DAY           = DNWORKINFO.PLAN_DAY
- COMPANY_CODE       = DNWORKINFO.COMPANY_CODE
- BATCH_NO           = DNWORKINDO.BATCH_NO
- PLAN_AREA_NO       = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO   = DNWORKINFO.PLAN_LOCATION_NO
- MATERIAL_CODE      = DNWORKINFO.MATERIAL_CODE
- MATERIAL_NAME      = DMITEM.MATERIAL_NAME
- ALLOCATION_QTY     = 1
- RETRIEVAL_STATION_NO = Arrival Station Number from **ID26** 
- RETRIEVAL_DETAIL   = 1:Unit Retrieval
- WORK_NO            = DNWORKINFO.JOB_NO
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

## DNHOSTSEND
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY    
- JOB_NO = DNWORKINFO.JOB_NO    
- COLLECT_JOB_NO = DNWORKINFO.COLLECT_JOB_NO    
- SETTING_UNIT_KEY = DNWORKINFO.SETTING_UNIT_KEY    
- JOB_TYPE = DNWORKINFO.JOB_TYPE    
- STATUS_FLAG = DNWORKINFO.STATUS_FLAG    
- HARDWARE_TYPE = DNWORKINFO.HARDWARE_TYPE    
- PLAN_UKEY = DNWORKINFO.PLAN_UKEY    
- STOCK_ID = DNWORKINFO.STOCK_ID    
- SYSTEM_CONN_KEY = DNWORKINFO.SYSTEM_CONN_KEY    
- PLAN_DAY = DNWORKINFO.PLAN_DAY    
- BATCH_NO = DNWORKINFO.BATCH_NO    
- PLAN_AREA_NO = DNWORKINFO.PLAN_AREA_NO    
- PLAN_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO    
- MATERIAL_CODE = DNWORKINFO.MATERIAL_CODE    
- MATERIAL_NAME = DMITEM.MATERIAL_NAME    
- RESULT_QTY = 1
- RESULT_AREA_NO = DNWORKINFO.RESULT_AREA_NO    
- RESULT_LOCATION_NO = DNWORKINFO.RESULT_LOCATION_NO    
- RESULT_LOT_NO = DNWORKINFO.RESULT_LOT_NO
- USER_ID = DNWORKINFO.USER_ID    
- TERMINAL_NO = DNWORKINFO.TERMINAL_NO    
- WORK_SECOND = DNWORKINFO.WORK_SECOND    
- USER_NAME = DCUSER.USERNAME    
- REPORT_FLAG = 0:Not Reported    
- REGIST_DATE = SYSTIMESTAMP    
- REGIST_PNAME = Class name    
- LAST_UPDATE_DATE = SYSTIMESTAMP    
- LAST_UPDATE_PNAME = Class name

## DNINOUTRESULT
- RESULT_KIND = 2:Retrieval(Stock-)
- STATION_NO = DNCARRYINFO.DEST_STATION_NO
- LOCATION_NO = DNPALLET.CURRENT_STATION_NO
- WH_STATION_NO = DNPALLET.WH_STATION_NO
- AISLE_STATION_NO = DNCARRYINFO.AISLE_STATION_NO
- WORK_TYPE = DNCARRYINFO.WORK_TYPE
- RETRIEVAL_DETAIL = DNCARRYINFO.RETRIEVAL_DETAIL
- WORK_NO = DNCARRYINFO.WORK_NO
- PALLET_ID = DNPALLET.PALLET_ID
- CARRY_KEY = DNCARRYINFO.CARRY_KEY
- RESTORING_FLAG = DNCARRYINFO.RESTORING_FLAG
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- REMOVE_FLAG = 00: Normal
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# Host Communication

##<span style="color:skyblue; font-weight:bold">[Unplanned Storage and Retrieval Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)</span>

# User Story
  - #5793

# Related DFD
  - [Unplanned Storage and Retrieval Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)
