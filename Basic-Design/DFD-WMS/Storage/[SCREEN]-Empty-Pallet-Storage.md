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
- STORING_PAIR_KEY = **ITEM_CODE + PLAT_LOT_NO**
- PLAN_QTY = 1
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = ClassName
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


## Control FLOW

- [PM / FG Inbound Storage - Same Warehouse (1106/1301/1302 -> 720x -> SRM)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Storage%20Process&pageId=1040&anchor=3.-fg-inbound-cross-warehouse-storage-(1301/1302--%3E-720x--%3E-710x--%3E-srm))
- [9200 — WNCollectAisleSelector (Pattern 4, Double Deep)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Storage%20Process&pageId=1040&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3E9200-%E2%80%94-wncollectaisleselector-(pattern-4%2C-double-deep)%3C/span%3E)
- [Storage Completion (ID33)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Storage%20Process&pageId=1040&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-completion-(id33)%3C/span%3E)

# Related User Story
- [User Story 5464 Empty Pallet Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5464)
- [Bug 6751 Empty Pallet Fix Error message when Pallet No exists](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6751)

# Related DFD
- [[SCREEN] Empty Pallet Storage - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/-SCREEN-Empty-Pallet-Storage)
- [Storage Process - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1040/Storage-Process)
