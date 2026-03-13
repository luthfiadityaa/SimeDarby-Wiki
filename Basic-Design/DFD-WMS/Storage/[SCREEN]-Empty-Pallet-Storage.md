[[_TOC_]]
[[_TOSP_]]

# [SCREEN] Empty Pallet Storage DATABASE FLOW
## Abbreviation
| **CODE** | TABLE NAME       |
|----------|------------------|
| **STPL** | DNSTORAGEPLAN    | 
| **ITEM** | DNPALLET         | 


| **CODE** | OPERATION NAME   |
|----------|------------------|
| **S**    | SELECT           |
| **I**    | INSERT           |
| **U**    | UPDATE           |
| **D**    | DELETE           |


## Inbound Table Data Flow
| Action Name                                                    | STPL | ITEM |
|----------------------------------------------------------------|------|------|
| Empty Pallet - Set (F2) [(1)](#empty-pallet---set-(f2))        |   I  |   S  |


# Empty Pallet - Set (F2)

![image.png](/.attachments/image-ae0da33c-5f36-45e7-ad8d-527f6a9e9325.png)
The Empty Pallet Setting Screen uses for storage the empty pallet to ASRS.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.storage.emptypalletsetting.EmptyPalletSettingSCH` &nbsp;</span>


::: mermaid
flowchart LR
input[
PALLET No
EMP_PB   
]

tableList-insert[("
DNSTORAGEPLAN
")]

className[EmptyPalletSettingSCH]


input --> className --> |"INSERT"| tableList-insert

:::

## Validations
This section explains the validations for the whole proccess Storage Packaging Material
- ITEM_CODE FIXED **DMITEM.EMP_PB**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty 

## DNSTORAGEPLAN
- PLAN_UKEY = Sequence Object
- LOAD_UNIT_KEY = Sequence Object
- CANCEL_FLAG = 0
- STATUS_FLAG = 0
- JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.STORAGE
- PLAN_DAY = TODAY (YYYYMMDD)
- BCR_DATA = Value from Screen (**Pallet #**)
- ITEM_CODE = DMITEM.EMP_PB
- PLAN_QTY = 1
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = ClassName
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


## Control FLOW

** TO BE CONTINUE ** 

#User Story
- [DFD Storage Stacked Empty Pallet](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5787)
- [5464 Empty Pallet Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5464)

# Related DFD
  - {}
