[[_TOC_]]
[[_TOC_]]
[[_TOSP_]]

# Planned Storage Setting (PKG) database flow

## Abbreviation
| **CODE** | TABLE NAME       |
|----------|------------------|
| **RECP** | DNRECEIVEPLAN  | 
| **STRP** | DNSTORAGEPLAN    | 


| **CODE** | OPERATION NAME   |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |

## Inbound Table Data Flow
| Action Name                                                    | RECP | STRP |
|----------------------------------------------------------------|------|------|
| Planned Storage from Host [(1)](#planned-storage-from-host)    |   S  |      |
| Planned Storage - Set (F2) [(2)](#planned-storage---set-(f2))  |   U  |      |


# Planned Storage - Set (F2)
![image.png](/.attachments/image-d46acf7f-3bd7-437c-b1d2-19798aacae2d.png)

Planned Storage Setting (PKG) is used to set the information of stock which will be entered into ASRS. After **Set(F2)** all item in input text will be process and The result will be posted back to SAP as [Planned Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/846/Planned-Storage-Result).

<span style="background-color:yellow; color:black; font-weight:bold">jp.co.daifuku.wms.web.display.storage.plannedstoragepkg.PlannedStoragePkgSCH</span>

::: mermaid
flowchart LR
    input[
        Document #
        Company Code
        Vendor Code / Vendor Name
        Material Code
        Material Name
        UOM
        Pallet No        
        Batch #
        Storage Qty / Planned Qty / Stored Qty        
        Plant
        Line #
        Document Date
        Delivery Date
    ]

    tableList-select[("
        DMITEM
    ")]

    tableList-select2[("
        DNRECEIVINGPLAN
    ")]

    className[PlannedStoragePkgSCH]

    input --> className--> |UPDATE| tableList-select2
    tableList-select--> |SELECT| className 
    tableList-select2--> |SELECT| className 

    style input text-align:left
:::

## Relantionship between Storage Plan and Pallet

One ReceivingPlan can have many StoragePlan.


::: mermaid
erDiagram
    DNRECEIVEPLAN ||--o{ DNSTORAGEPLAN : "identifies"

    DNRECEIVEPLAN {
        string RECEIVE_TICKET_NO PK
        string RECEIVE_LINE_NO PK
    }

    DNSTORAGEPLAN {
        string RECEIVE_TICKET_NO FK
        string RECEIVE_LINE_NO FK
        string bcr_data PK
    }	
:::

## Validations
This section explains the validations for the whole proccess Storage Packaging Material

- Material Code exists in **DMITEM**
- The Planned Storage PKG list is displayed after being filtered by **DMITEM.ITEM_TYPE.ZPCK**.
- Input text with red asterisk <span style="color:red">(*)</span> is required to filled.
- **Storage Qty** must be greater than **"0"**
- The **Storage Qty + Stored Qty** should be less than Planned Qty.
- **Daily cleanup** not processing.
- **Pallet No** is progress does not exist in **DNStoragePlan**.

<br>

**Note:** 
- All IN-stations can be used.
- Stored Qty = DNRECEIVINGPLAN.PROCESS_QTY
- Planned Qty = DNRECEVINGPLAN.PLAN_QTY

## DNSTORAGEPLAN

- PLAN_UKEY             = Sequence Object
- LOAD_UNIT_KEY         = Sequence Object
- CANCEL_FLAG           = 0
- STATUS_FLAG           = 0
- JOB_TYPE              = DNSTORAGEPLAN.JOB_TYPE.STORAGE
- PLAN_DAY              = TODAY (YYYYMMDD)
- BCR_DATA              = Value from Screen (**Pallet #**)
- ITEM_CODE             = Value from screen (**ITEM CODE**)
- PLAN_QTY              = Value from screen (**Storage Qty**)
- STORAGE_LOCATION_FROM = Constant.SAP_STORAGE_LOCATION.TEMPORARY_LOCATION
- STORAGE_LOCATION_TO   = Constant.SAP_STORAGE_LOCATION.PACKAGING_MATERIAL
- RECEIVE_TICKET_NO     = DNSTORAGEPLAN.RECEIVE_TICKET_NO
- RECEIVE_LINE_NO       = DNSTORAGEPLAN.RECEIVE_LINE_NO
- **STORING_PAIR_KEY    = DNSTORAGEPLAN.ITEM_CODE + DNSTORAGEPLAN.PLAT_LOT_NO**
- REGIST_DATE           = SYSTIMESTAMP
- REGIST_PNAME          = ClassName
- LAST_UPDATE_DATE      = SYSTIMESTAMP
- LAST_UPDATE_PNAME     = ClassName

## DNRECEIVINGPLAN

- STATUS_FLAG       = 1:Working    **Very first pallet will update**    
- PROCESS_QTY       = DNSTORAGEPLAN + DNWORKINGO.PLAN_QTY   
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


## CONTROL Flow

**TO BE CONTINUE**


##<span style="color:skyblue; font-weight:bold">[Planned Storage PKG - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/836/Planned-Storage-PKG)</span>


# User Story
  - [DFD Storage Packaging Material](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5784)

  - [5472: Storage Plan Information Maintenance​ (PKG)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5472/)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>

- [Planned Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/846/Planned-Storage-Result)
