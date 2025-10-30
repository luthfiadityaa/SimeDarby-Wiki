[[_TOC_]]
[[_TOSP_]]

[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Batch End database flow</span>

| Action Name                        | DNSTOCK | DNPALLET | DNWORKINFO | DNWORKLIST | DNHOSTSEND | DNSTORAGEPLAN | DNPALLETIZE | 
|------------------------------------|---------|----------|------------|------------|------------|---------------|---------------|
| [Batch End - Set(F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Setting&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Epalletizing-setting-set(f2)%3C/span%3E)      |   |    |      |      |      |         |     UPDATE    |

#<span style="color:skyblue; font-weight:bold">Batch End - Set(F2)</span>
  
Batch End is used to set qty of stock on the last pallet of current Batch and set the palletizing work to stop. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as Production Storage Result.

![image.png](/.attachments/image-afd50f57-e0f4-49c3-bea7-73a1dad10869.png)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Station<br>
        Current Status<br>
        Material Code<br>
        Material Name<br>
        Material Type<br>
        Batch #<br>
        Storage_Qty #<br>
        Planned Carton Qty<br>
        Total Actual Carton Qty<br>
        Storage Date/Time<br>
        Qty kg/crtn<br>
        Qty crtn/PL<br>
        UOM<br>
        Tempering Period<br>
        Storage Location<br>
        Expiry Days<br>
    ]

    tableList-insert-update[("
        DNPALLETIZE<br>
    ")]

    tableList-insert[("
        DNSTORAGEPLAN
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |Insert/Update| tableList-insert-update

    classDef leftAlign text-align:left;
    class input leftAlign;

:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start
- Material Code exists in **DMMaterialMaster**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNPALLETIZE</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **STATION_NO**                 | Value from screen (Station No)
| **CURRENT_STATUS**             | Value from screen (Current Status)
| **MATERIAL_CODE**              | Value from screen (Material Code)
| **MATERIAL_NAME**              | Value from screen (Material Name)
| **MATERIAL_TYPE**              | Value from screen (Material Type)
| **BATCH_NO**                   | Value from screen (BATCH_NO)
| **PLANNED_CARTON_QTY**         | Value from screen (PLANNED CARTON QTY)
| **QTY_KG_CRTN**                | Value from screen (Qty Kg/Crtn)
| **QTY_CRTN_PL**                | Value from screen (Qty Crtn/PL)
| **UOM**                        | Value from screen (UOM)
| **TEMPERING_PERIOD**           | Value from screen (Tempering Period)
| **STORAGE_LOCATION**           | Value from screen (Storage Location)
| **EXPIRY_DAYS**                | Value from screen (Expiry Days)
| **STORAGE_QTY**                | 0:Not-Started
| **TOTAL_ACTUAL_CARTON_QTY**    | 
| **MAX_PALLET_BATCH_END**       | 
| **BATCH_PALLET_START**         | 
| **BATCH_PALLET_END**           | SYSTIMESTAMP
| **REGIST_DATE**                |                                                     
| **REGIST_PNAME**               | 
| **LAST_UPDATE_DATE**           | 
| **LAST_UPDATE_PNAME**          | 

#User Story
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)