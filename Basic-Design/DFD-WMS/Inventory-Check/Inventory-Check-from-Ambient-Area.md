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

## DNWORKINFO
- JOB_NO =  Sequence Object    
- SETTING_UNIT_KEY =  Sequence Object    
- COLLECT_JOB_NO =  Sequence Object- HARDWARE_TYPE = 2:RFT, 3:ASRS    
- JOB_TYPE =  23: Unplanned Retrieval    
- STATUS_FLAG =  0: Not Started    
- PLAN_UKEY =  Sequence Object    
- STOCK_ID =  DNSTOCK.STOCK_ID    
- PLAN_AREA_NO =  DNSTOCK.AREA_NO   
- PLAN_LOCATION_NO =  DNSTOCK.LOCATION_NO
- PLAN_DAY =  DMWARENAVISYSTEM.WORK_DAY    
- BATCH_NO =  DNSTOCK.BATCH_NO   
- MATERIAL_CODE =  Value from screen (Material Code)    
- MATERIAL_NAME =  Value from screen (Material Name)    
- MATERIAL_TYPE =  Value from screen (Material Type) 
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


# User Story
  - #5795

# Related DFD
  - {}
