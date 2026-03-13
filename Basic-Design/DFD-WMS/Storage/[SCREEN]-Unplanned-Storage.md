[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Storage from 1301 & 1302</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)


#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
**Abbreviation:**
- **STRP** : DNSTORAGEPLAN  

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                                               |STRP| 
|-------------------------------------------------------------------------------------------|----|
| Unplanned Storage - Set(F2) [(1)](#Unplanned-Storage---Set(F2))                           | I  |  
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    | 


#Unplanned Storage - Set(F2)
![image.png](/.attachments/image-6293c4f4-5fd4-4247-8cbc-fbcc6d9cf3a5.png)
 
The Unplanned Storage Setting screen uses for manually set the storage work .
This screen mainly uses when there are some troubles with Host System Linkage(irregular cases) or when manual operation is necessary.
After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wms.web.display.storage.unplannedstorage.UnplannedStorageSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Pallet ID
        Material Code
        Material Name
        Material Type
        Qty kg/crtn
        Qty crtn/PL
        Batch #
        Storage Qty
        UOM
        Tempering Period
        Expiry Days
        Storage Location From
        Storage Location To        
    ]

    tableList-insert[("
        DNSTORAGEPLAN
    ")]

    className[UnplannedStorageSCH]

    input --> className --> |INSERT| tableList-insert
	
    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start

- Pallet cannot exist in <span style="color:green; font-weight:bold">DNPallet.</span>  
  * if DNPallet.Bcr_data = pallet_no then failed
- Material Code exists in <span style="color:green; font-weight:bold">DMITEM</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
- PLAN_UKEY         = Sequence Object
- LOAD_UNIT_KEY     = Sequence Object
- CANCEL_FLAG       = 0
- STATUS_FLAG       = 0
- JOB_TYPE          = DNSTORAGEPLAN.JOB_TYPE.NOPLAN_STORAGE
- PLAN_DAY          = TODAY (YYYYMMDD)
- BCR_DATA          = Value from Screen (**Pallet #**)
- ITEM_CODE         = Value from screen (**ITEM CODE**)
- PLAN_QTY          = Value from screen (**Storage Qty**)
- PLAN_LOT_NO       = Value from screen (**Storage Qty**)
- PLAN_AREA_NO      = Value from screen (**Storage Location**)
- STORAGE_LOCATION_FROM = Constant.SAP_STORAGE_LOCATION.TEMPORARY_LOCATION
- STORAGE_LOCATION_TO   = Value from screen (**Storage Location**)
- REGIST_DATE       = SYSTIMESTAMP
- REGIST_PNAME      = ClassName
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName


#Unplanned Storage and Retrieval Result
- (https://dev.azure.com/DaifukuSW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)

<hr>

#User Story
- [DFD Manual Storage to Ambient](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5785)
- [DFD Manual Storage to Tempering](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5786)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)