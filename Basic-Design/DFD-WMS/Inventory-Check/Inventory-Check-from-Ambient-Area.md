[[_TOC_]]
[[_TOSP_]]

# Inventory Check from Ambient Area
![image.png](/.attachments/image-da2c69da-502e-434b-8de2-eafe8b266f4f.png)

# Summary Flow
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1301, 1302]
:::

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
| **INVC** | DNINVENTORYCHECK |

| **CODE** | OPERATION NAME   |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |

## Inbound Table Data Flow
| Action Name                                                    | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | HTSD | OPRR | ITEM | STSN | INVC |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Inquiry Retrieval - Set(F2) [(1)](#inventory-check---set(f2))  |   U  |   I  |   I  |   I  |      |      |   S  |   S  |      |      |      |      |   S  |   S  |   I  |
| Retrieval Sender [(2)](#retrieval-sender)                      |   U  |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
# Inventory Check - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.inventorycheck.InventoryCheckSCH` &nbsp;</span>
![image.png](/.attachments/image-30ccad7a-c7a3-44cb-aeba-8d19d917c4ea.png)
  
The Inventory Check Setting screen uses for manually set the inventory check work (**always Partial Pick, and retrieval qty = 0**).

::: mermaid
flowchart LR
    input[
        Stock Qty
        Material Code
        Material Name
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
    ]

    tableList-insert[("
        DNINVENTORYCHECK
        DNCARRYINFO
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
    tableList-update[("
        DNPALLET
    ")]

    className[InventoryCheckSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |UPDATE| tableList-update
    tableList-select --> |SELECT| className
:::

## Validations
This section explains the validations for the whole proccess Unplanned Retrieval process (PKG)
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

## DNPALLET
- STATUS_FLAG = 3:Reserved for retrieval
- ALLOCATION_FLAG = 1:Allocated
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNINVENTORYCHECK
- SCHEDULE_NO = Sequence object
- FROM_LOCATION = Value from screen (Start Location No.)
- TO_LOCATION = Value from screen (End Location No.)
- WH_STATION_NO = Value from screen (Area No.)
- STATION_NO = Value from screen (Station No.)
- INVENTORY_ORDER = 1: By Location Number
- INVENTORY_DAY = DMWARENAVISYSTEM.WORK_DAY
- STATUS_FLAG = 1:Inventory Check in Progress
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNCARRYINFO
- CARRY_KEY = Sequence Object    
- PALLET_ID = DNSTOCK.PALLET_ID    
- WORK_TYPE = 40: Inventory Check    
- CMD_STATUS = 1: Started    
- PRIORITY = 2: Normal    
- RESTORING_FLAG = 1: Return to Same Location  
- WORK_NO = Sequence Object
- CARRY_FLAG = 2: Retrieval    
- RETRIEVAL_STATION_NO = DNSTOCK.LOCATION_NO
- RETRIEVAL_DETAIL = 0: Inventory Check
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO    
- DEST_STATION_NO = Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(1301, 1302)**
- PRIORITY = 2: Normal
- CANCEL_REQUEST = 0: Not Requested    
- SCHEDULE_NO = Sequence Object  
- CANCEL_REQUEST = 0: Not requested
- AISLE_STATION_NO = DMSHELF.PARENT_STATION_NO
- END_STATION_NO = DNCARRYINFO.DEST_STATION_NO  
- REGIST_DATE = SYSTIMESTAMP    
- REGIST_PNAME = ClassName    
- LAST_UPDATE_DATE = SYSTIMESTAMP    
- LAST_UPDATE_PNAME = ClassName

## DNWORKINFO
- JOB_NO =  Sequence Object    
- SETTING_UNIT_KEY =  Sequence Object    
- COLLECT_JOB_NO =  Sequence Object    
- JOB_TYPE =  40: Inventory Check      
- STATUS_FLAG =  1: Working
- HARDWARE_TYPE = 3: ASRS    
- PLAN_UKEY =  Sequence Object    
- STOCK_ID =  DNSTOCK.STOCK_ID    
- PLAN_AREA_NO =  DNSTOCK.AREA_NO   
- PLAN_LOCATION_NO =  DNSTOCK.LOCATION_NO
- PLAN_DAY =  DMWARENAVISYSTEM.WORK_DAY
- VENDOR_CODE = DNSTOCK.VENDOR_CODE
- VENDOR_NAME = DNSTOCK.VENDOR_NAME
- COMPANY_CODE = DNSTOCK.COMPANY_CODE     
- BATCH_NO =  Value from screen (Batch #)    
- MATERIAL_CODE =  Value from screen (Material Code) 
- PLAN_LOT_NO = DNSTOCK.LOT_NO 
- PLAN_QTY =  0   
- QTY_KG_CRTN =  Value from screen (Qty kg/crtn)    
- QTY_CRTN_PL =  Value from screen (Qty crtn/PL)    
- UOM =  Value from screen (UOM)    
- TEMPERING_PERIOD =  Value from screen (Tempering Period)    
- EXPIRY_DAYS =  DNSTOCK.EXPIRYDATE    
- WORK_DAY =  DMWARENAVISYSTEM.WORK_DAY  
- BCR_DATA =  DNPALLET.BCR_DATA  
- DOCK_NO =  DNWORKINFO.DOCK_NO
- TRUCK_PLATE_NO =  DNWORKINFO.TRUCK_PLATE_NO
- USER_ID =  Login Info     
- TERMINAL_NO =  Login info
- REGIST_DATE =  SYSTIMESTAMP    
- REGIST_PNAME =  ClassName    
- LAST_UPDATE_DATE =  SYSTIMESTAMP    
- LAST_UPDATE_PNAME =  ClassName

## DNWORKLIST
- JOB_NO             = DNWORKINFO.JOB_NO
- CARRY_KEY          = DNWORKINFO.SYSTEM_CONN_KEY
- SETTING_UNIT_KEY   = DNWORKINFO.SETTING_UNIT_KEY
- COLLECT_JOB_NO     = DNWORKINFO.COLLECT_JOB_NO
- JOB_TYPE           = DNWORKINFO.JOB_TYPE
- PLAN_UKEY          = DNWORKINFO.PLAN_UKEY
- STOCK_ID           = DNWORKINFO.STOCK_ID
- ITEM_CODE          = DNWORKINFO.ITEM_CODE
- PLAN_LOT_NO        = DNWORKINFO.PLAN_LOT_NO
- PLAN_QTY           = DNWORKINFO.PLAN_QTY
- PLAN_DAY           = DNWORKINFO.PLAN_DAY
- PALLET_ID          = DNCARRYINFO.PALLET_ID
- PLAN_DAY           = DNWORKINFO.PLAN_DAY
- COMPANY_CODE       = DNWORKINFO.COMPANY_CODE
- BATCH_NO           = DNWORKINDO.BATCH_NO
- PLAN_AREA_NO       = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO   = DNWORKINFO.PLAN_LOCATION_NO
- MATERIAL_CODE      = DNWORKINFO.MATERIAL_CODE
- MATERIAL_NAME      = DMITEM.MATERIAL_NAME
- ALLOCATION_QTY     = 1
- RETRIEVAL_STATION_NO = DNCARRYINFO.RETRIEVAL_STATION_NO
- RETRIEVAL_DETAIL   = DNCARRYINFO.RETRIEVAL_DETAIL
- WORK_NO            = DNCARRYINFO.WORK_NO
- SOURCE_STATION_NO  = DNCARRYINFO.SOURCE_STATION_NO
- DEST_STATION_NO    = DNCARRYINFO.DEST_STATION_NO
- SCHEDULE_NO        = DNCARRYINFO.SCHEDULE_NO, DNINVENTORYCHECK.SCHEDULE_NO
- END_STATION_NO     = DNCARRYINFO.END_STATION_NO
- USER_ID            = Login Info
- USER_NAME          = Login Info
- TERMINAL_NO        = Login Terminal
- REGIST_DATE        = SYSTIMESTAMP                                                    
- REGIST_PNAME       = ClassName
- LAST_UPDATE_DATE   = SYSTIMESTAMP
- LAST_UPDATE_PNAME  = ClassName

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

For Inventory Check performed in ASRS where there is creation of DNCARRYINFO, DNCARRYNFO data will be processed in Retrieval Sender. ID 12 will be sent after related tables are updated successfully.

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


# User Story
  - #5795

# Related DFD
  - {}
