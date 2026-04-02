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
        Stock Status
        Storage Location From
        Storage Location To        
    ]

    tableList-insert[("
        DNSTORAGEPLAN
    ")]

    tableList-select[("
        DMITEM
    ")]

    className[UnplannedStorageSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Unplanned Storage Start

- Input text with red asterisk <span style="color:red">(*)</span> is not empty
- **Barcode Data** not exist : 
  *   PalletNo is progress does not exist in **DNStoragePlan**.
  *   PalletID is not found in existing pallet in **DNStock**.
- **Daily cleanup** not processing.
- **Material Code** only select exclude :
  * if select Material_Type = **99** (EMP_PB) OR **98** (IRREGULAR_PB) OR **97** (DIRECT_PB), Then pass <span style="color:red">**ERROR**</span> message.
- **Storage Location (Master)** only display <span style="color:green; font-weight:bold">SAP_LOCATION_STORAGE</span> List.
- When **Material Code** has been selected, it will be filtered based on <span style="color:green; font-weight:bold">DMITEM.SOFT_ZONE_ID</span>:
  * **SOFT_ZONE_ID** = <span style="color:green; font-weight:bold">005</span>
    --> **To Location** is display value (**FGW1, FGW2**).
    --> **Stock Status** will appear with Value is **'UU'**.

  * **SOFT_ZONE_ID** = <span style="color:green; font-weight:bold">001</span> 
    --> **To Location** is display value (**FGW2**).
    --> **Stock Status** will appear with Value is '**UU**'.

  * **SOFT_ZONE_ID** = <span style="color:green; font-weight:bold">002</span> 
    --> **To Location** is display value (**FGW1**).
    --> **Stock Status** will appear with Value is '**UU**'.

  * **SOFT_ZONE_ID** = <span style="color:green; font-weight:bold">003</span> 
    --> **To Location** is display value (**ZPCK**)
    --> **Tempering Period** will be <span style="color:red">disable</span> .
    --> **Expiry Days** will be <span style="color:red">disable</span> .

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
- PLAN_AREA_NO      = **FGW1:9002 or FGW2:9001**
- STORAGE_LOCATION_FROM = Value from screen (**Storage Location**)
- STORAGE_LOCATION_TO   = Value from screen (**To Location**)
- STORING_PAIR_KEY  = **ITEM_CODE + PLAN_LOT_NO**
- BATCH_TEMPEREING_PERIOD = Value from screen (**Tempering Period**)
- BATCH_EXPIRY_DAYS = Value from screen (**Expiry Days**)
- STOCK_STATUS      = Value from screen (**Stock Status**)
- REGIST_DATE       = SYSTIMESTAMP
- REGIST_PNAME      = ClassName
- LAST_UPDATE_DATE  = SYSTIMESTAMP
- LAST_UPDATE_PNAME = ClassName

CONTROL FLOW[](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/882/-SCREEN-ONLY-Planned-Storage-Packaging-Material?anchor=control-flow)
=======================================================================================================================================================================================

*   [PM / FG Inbound Storage - Same Warehouse (1106/1301/1302 -> 720x -> SRM)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1040/Storage-Process?anchor=2.-pm-/-fg-inbound-storage---same-warehouse-(1106/1301/1302--%3E-720x--%3E-srm))
*   [9200 — WNCollectAisleSelector (Pattern 4, Double Deep)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1040/Storage-Process?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3E9200-%E2%80%94-wncollectaisleselector-(pattern-4%2C-double-deep)%3C/span%3E)
*   [Storage Completion (ID33)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/1040/Storage-Process?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-completion-(id33)%3C/span%3E)

#Related User Story

- [#5139 Unplanned Storage Setting​​](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5140)
- [#6524 Control for ALL Storage operation](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6524)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)
- [DFD Manual Storage to Ambient](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5785)
- [DFD Manual Storage to Tempering](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5786)