[[_TOC_]]
[[_TOSP_]]

[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-31228a41-0618-46d0-acf0-93f1edd79e4e.png)

#<span style="color:skyblue; font-weight:bold">Batch End database flow</span>

| Action Name                        | DNPALLETIZE | DNSTORAGEPLAN | DNWORKINFO | DNWORKLIST | DNPALLET | DNSTOCK | DNHOSTSEND   | 
|------------------------------------|-------------|---------------|------------|------------|----------|---------|--------------|
| [Batch End - Set(F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ebatch-end-database-flow%3C/span%3E)      |  UPDATE |  INSERT  |   INSERT   |   INSERT   |   INSERT   |     INSERT    |     INSERT    |

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
        Storage Qty<br>
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
        DNSTORAGEPLAN<br>
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |UPDATE| tableList-insert-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNPalletize</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **STATION_NO**                 | Data Last Pallet
| **CURRENT_STATUS**             | Data Last Pallet
| **MATERIAL_CODE**              | Data Last Pallet
| **MATERIAL_NAME**              | Data Last Pallet
| **MATERIAL_TYPE**              | Data Last Pallet
| **BATCH_NO**                   | Data Last Pallet
| **PLANNED_CARTON_QTY**         | Data Last Pallet
| **QTY_KG_CRTN**                | Data Last Pallet
| **QTY_CRTN_PL**                | Data Last Pallet
| **UOM**                        | Data Last Pallet
| **TEMPERING_PERIOD**           | Data Last Pallet 
| **STORAGE_LOCATION**           | Data Last Pallet
| **EXPIRY_DAYS**                | Data Last Pallet
| **STORAGE_QTY**                | Value set by system or Input at screen 
| **TOTAL_ACTUAL_CARTON_QTY**    | Data Last Pallet
| **MAX_PALLET_BATCH_END**       | Data Last Pallet
| **BATCH_PALLET_START**         | Data Last Pallet
| **BATCH_PALLET_END**           | SYSTIMESTAMP
| **REGIST_DATE**                | Data Last Pallet                                                     
| **REGIST_PNAME**               | Data Last Pallet
| **LAST_UPDATE_DATE**           | Data Last Pallet
| **LAST_UPDATE_PNAME**          | Data Last Pallet 

##<span style="color:skyblue; font-weight:bold">[Continue flow Release Command from Palletize Robot](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Erelease-command-from-palletize-robot%3C/span%3E)</span>

##<span style="color:skyblue; font-weight:bold">ID33</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY
DNINOUTRESULT
")]
id33-update[("
DNPALLET
DMSHELF
DNSTOCK
DNWORKINFO
DNSTORAGEPLAN
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--> |INSERT| id33-insert
id33process--> |UPDATE| id33-update
id33process--> |DELETE| id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

####<span style="color:skyblue; font-weight:bold">DMShelf</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **STATUS_FLAG**            | 1: Occupied
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**     | Location Number information from ID33
| **STATUS_FLAG**            | 2:Occupied
| **ALLOCATION_FLAG**        | 0:Not allocated
| **LAST_STORED_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **RESULT_QTY**             | DNWORKINFO.PLAN_QTY
| **RESULT_AREA_NO**         | DNWORKINFO.PLAN_AREA_NO
| **RESULT_LOCATION_NO**     | DNWORKINFO.PLAN_LOCATION_NO
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **STATUS_FLAG**            | 4:Completed
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **STORAGE_DAY**            | DMWARENAVISYSTEM.WORK_DAY
| **NEWEST_STORAGE_DATE**    | SYSTIMESTAMP
| **STOCK_QTY**              | DNSTOCK.PLAN_QTY
| **ALLOCATION_QTY**         | DNWORKINFO.RESULT_QTY
| **PLAN_QTY**               | 0
| **STOCK_STATUS**           | <span style="color:yellow; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:yellow; font-weight:bold">0: Not Reached</span>
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold"> DNStoragePlan </span>
<span style="color:red; font-weight:bold; font-size:12px">*If DNSTORAGEPLAN.RESULT_QTY = DNSTORAGEPLAN.PLAN_QTY then update Status to completed</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **STATUS_FLAG**            | 4: Completed
| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
| **SHORTAGE_QTY**           | DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DNStockHistory </span> 
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)


| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **INC_DEC_TYPE**           | 1: Stock Increase
| **JOB_TYPE**               | 2: Storage
| **UPDATE_STOCK_QTY**       | DNSTOCK.STOCK_QTY
| **INC_DEC_QTY**            | DNSTOCK.STOCK_QTY
| **STOCK_ID**               | DNSTOCK.STOCK_ID
| **AREA_NO**                | DNSTOCK.AREA_NO
| **LOCATION_NO**            | DNSTOCK.LOCATION_NO
| **STORAGE_DAY**            | DNSTOCK.STORAGE_DAY
| **STORAGE_DATE**           | DNSTOCK.STORAGE_DATE
| **NEWEST_STORAGE_DATE**    | DNSTOCK.NEWEST_STORAGE_DATE
| **MATERIAL_CODE            | DNSTOCK.MATERIAL_CODE
| **PALLET_ID**              | DNSTOCK.PALLET_ID
| **BCR_DATA**               | DNPALLET.BCR_DATA
| **AREA_TYPE**              | DMAREA_AREA_TYPE
| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
| **ENTERING_QTY**           | DMMATERIALMASTER.QTY_CRTN
| **USER_ID**                | Login info
| **USER_NAME**              | Login info
| **TERMINAL_NO**            | Login info
| **TERMINAL_NAME**          | Login info
| **IP_ADDRESS**             | Login info
| **STOCK_STATUS**           | <span style="color:yellow; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:yellow; font-weight:bold">Not Done</span>
| **EXPIRY_DATE**            | DNWORKINFO.EXPIRY_DAYS
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | Class name

####<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **RESULT_KIND**            | 1:Storage(Stock+)
| **STATION_NO**             | DNCARRYINFO.SOURCE_STATION_NO
| **LOCATION_NO**            | DNCARRYINFO.DEST_STATION_NO
| **WH_STATION_NO**          | DNPALLET.WH_STATION_NO
| **AISLE_STATION_NO**       | DNCARRYINFO.AISLE_STATION_NO
| **WORK_TYPE**              | DNCARRYINFO.WORK_TYPE
| **RETRIEVAL_DETAIL**       | DNCARRYINFO.RETRIEVAL_DETAIL
| **WORK_NO**                | DNCARRYINFO.WORK_NO
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **CARRY_KEY**              | DNCARRYINFO.CARRY_KEY
| **RESTORING_FLAG**         | DNCARRYINFO.RESTORING_FLAG
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **REMOVE_FLAG**            | 00:Normal
| **REGIST_PNAME**           | Class name
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DNHostSend</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | DNWORKINFO.WORK_DAY
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
| **HARDWARE_TYPE**          | DNWORKINFO.HARDWARE_TYPE
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **SYSTEM_CONN_KEY**        | DNWORKINFO.SYSTEM_CONN_KEY
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **PLAN_AREA_NO**           | DNWORKINFO.PLAN_AREA_NO
| **PLAN_LOCATION_NO**       | DNWORKINFO.PLAN_LOCATION_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
| **UOM**                    | MMATERIALMASTER.UOM
| **ENTERING_QTY**           | DMMATERIALMASTER.QTY_CRTN
| **BUNDLE_ENTERING_QTY**    | DNPALLETIZE.PLANNED_CARTON_QTY
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **RESULT_QTY**             | DNWORKINFO.RESULT_QTY
| **SHORTAGE_QTY**           | DNWORKINFO.SHORTAGE_QTY
| **RESULT_AREA_NO**         | DNWORKINFO.RESULT_AREA_NO
| **RESULT_LOCATION_NO**     | DNWORKINFO.RESULT_LOCATION_NO
| **RESULT_LOT_NO**          | DNWORKINFO.RESULT_LOT_NO
| **USER_ID**                | DNWORKINFO.USER_ID
| **TERMINAL_NO**            | DNWORKINFO.TERMINAL_NO
| **WORK_SECOND**            | DNWORKINFO.WORK_SECOND
| **USER_NAME**              | DCUSER.USERNAME
| **REPORT_FLAG**            | 1:Reported
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | Class name
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

![image.png](/.attachments/image-1de466bf-fb93-4371-8c45-00cb7a5a5f29.png)

##<span style="color:skyblue; font-weight:bold">[Send Report as Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)</span>

#User Story
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)