[[_TOC_]]
[[_TOSP_]]

[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Batch End database flow</span>
**Abbreviation:**
- **STRP** : DNSTORAGEPLAN  
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKINFOLIST  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **HSTS** : DNHOSTSEND  
- **ARVL** : DNARRIVAL  
- **WRHS** : DMWAREHOUSE  
- **SHLF** : DMSHELF  
- **MTMS** : DMATERIALMASTER  
- **STSN** : DMSTATION  
- **STCH** : DNSTOCKHISTORY  

| Action Name                 | STRP   | WRKI   | WRKL   | PLLT   | CRYI   | STCK   | HSTS   | ARRVL  | WRHS   | SHLF   | STCH   | MTMS   | STSN   |
|-----------------------------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| Batch End - SetF2[(1)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ebatch-end---set(f2)%3C/span%3E)  | SELECT |        |        |        |        |        |        |        |        |        |        | SELECT | SELECT |
| ID26[(2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E)                     |        | INSERT |        | INSERT | INSERT | INSERT |        | INSERT |        |        |        |        |        |
| Storage Sender[(3)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-sender%3C/span%3E)           |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25[(4)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%3C/span%3E)                     |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID26[(5)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%26nbsp%3B%3C/span%3E)                     |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |   
| Storage Sender[(6)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-sender%26nbsp%3B%3C/span%3E)           |        |        |        | UPDATE | UPDATE | UPDATE |        | UPDATE | UPDATE | UPDATE |        |        |        | 
| ID25[(7)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%26nbsp%3B%3C/span%3E)                     |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| **Flow 1**                  |        |        |        |        |        |        |        |        |        |        |        |        |        |  
| ID64[(8)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%3C/span%3E)                     |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        | 
| ID26[(9)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%26nbsp%3B%26nbsp%3B%3C/span%3E)                     |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |
| Storage Sender[(10)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estoragesender%26nbsp%3B%26nbsp%3B%3C/span%3E)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25[(11)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3E%26nbsp%3Bid25%26nbsp%3B%3C/span%3E)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64[(12)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%26nbsp%3B%26nbsp%3B%3C/span%3E)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| **Flow 2**                  |        |        |        |        |        |        |        |        |        |        |        |        |        |
| ID64[(8)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%26nbsp%3B%26nbsp%3B%3C/span%3E)                     |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID26[(9)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%26nbsp%3B%26nbsp%3B%3C/span%3E)                     |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |
| Storage Sender[(10)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estoragesender%26nbsp%3B%26nbsp%3B%3C/span%3E)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25[(11)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid25%26nbsp%3B%3C/span%3E)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64[(12)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%3C/span%3E)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID64[(13)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%3C/span%3E)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID26[(14)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%26nbsp%3B%26nbsp%3B%3C/span%3E)                    |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |
| Storage Sender[(15)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estoragesender%26nbsp%3B%26nbsp%3B%3C/span%3E)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25[(16)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3E%26nbsp%3Bid25%26nbsp%3B%3C/span%3E)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64[(17)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid64%26nbsp%3B%26nbsp%3B%3C/span%3E)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| **Last Process**            |        |        |        |        |        |        |        |        |        |        |        |        |        |  
| ID33[(18)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid33%3C/span%3E)                    | UPDATE | UPDATE | INSERT | UPDATE | UPDATE | DELETE | UPDATE | INSERT |        | UPDATE | UPDATE | INSERT |        |  

#<span style="color:skyblue; font-weight:bold">Batch End - Set(F2)</span>
  
Batch End is used to set qty of stock on the last pallet of current Batch and set the palletizing work to stop. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)

![image.png](/.attachments/image-afd50f57-e0f4-49c3-bea7-73a1dad10869.png)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Station
        Current Status
        Material Code
        Material Name
        Material Type
        Batch #
        Storage Qty
        Planned Carton Qty
        Total Actual Carton Qty
        Storage Date/Time
        Qty kg/crtn
        Qty crtn/PL
        UOM
        Tempering Period
        Storage Location
        Expiry Days
    ]

    tableList-insert-select[("
        DMSTATION
        DMMATERIALMASTER
    ")]

    tableList-insert[("
        DNSTORAGEPLAN
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |SELECT| tableList-insert-select

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Batch End

- **Total Actual Carton Qty** should be **less than or equal to** the **Planned Carton Qty**.  
  (This ensures that all planned cartons have been processed.)
- **Storage Qty** is calculated as:  
  **Storage Qty = Planned Carton Qty − Total Actual Carton Qty**  
  (This shows how many cartons are still remaining or unprocessed.)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-----------------------------------------------|
| **PLAN_UKEY**              | Sequence Object
| **STATION_NO**             | Data Last Pallet (STATION_NO)
| **CURRENT_STATUS**         | Data Last Pallet (CURRENT_STATUS)                                                     
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | Data Last Pallet (STORAGE_DATE_TIME)                                                       
| **PLAN_AREA_NO**           | Data Last Pallet (STORAGE_LOCATION)
| **PLANNED_CARTON_QTY**     | Data Last Pallet (PLANNED_CARTON_QTY)
| **QTY_KG_CRTN**            | Data Last Pallet (QTY_KG_CRTN)                             
| **MATERIAL_CODE**          | Data Last Pallet (MATERIAL_CODE)
| **MATERIAL_NAME**          | Data Last Pallet (MATERIAL_NAME)
| **MATERIAL_TYPE**          | Data Last Pallet (MATERIAL_TYPE)
| **BATCH_NO**               | Data Last Pallet (BATCH_NO)                                                    
| **PLAN_QTY**               | Data Last Pallet (QTY_CRTN_PL)                                               
| **REPORT_FLAG**            | 0:Not Reported                                                   
| **STORING_PAIR_KEY**       | Data Last Pallet (MATERIAL_CODE + BATCH_NO)
| **UOM**                    | Data Last Pallet (UOM)
| **TEMPERING_PERIOD**       | Data Last Pallet (TEMPERING PERIOD)
| **EXPIRY_DAYS**            | Data Last Pallet (EXPIRY_DAYS)
| **STORAGE_QTY**            | Data Last Pallet (PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY)
| **TOTAL_ACTUAL_CARTON_QTY**| Data Last Pallet (TOTAL_ACTUAL_CARTON_QTY - QTY_CRTN)
| **MAX_PALLET_BATCH_END**   | Data Last Pallet (PALLET_CARTON_QTY / QTY_CRTN_PL)
| **BATCH_PALLET_START**     | Data Last Pallet (STORAGE_DATE_TIME + Time value (System Generated))
| **BATCH_PALLET_END**       | SYSTIMESTAMP
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

<br>
<hr style="border: 2px solid red;">

#Irregular Cases
- [Reverse – Batch End Not Executed Despite Force Completion. Treat as Data Error](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ereverse-%E2%80%93-batch-end-not-executed-despite-force-completion.-treat-as-data-error%3C/span%3E)
<hr style="border: 2px solid red;">

<hr>

##<span style="color:skyblue; font-weight:bold">[Continue flow Release Command from Palletize Robot - Dummy Arrival](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Erelease-command-from-palletize-robot---dummy-arrival%3C/span%3E)</span><br>
<hr>

##<span style="color:skyblue; font-weight:bold">ID33</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process&nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY
DNWORKLIST
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
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)

| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **STORAGE_DAY**            | DMWARENAVISYSTEM.WORK_DAY
| **NEWEST_STORAGE_DATE**    | SYSTIMESTAMP
| **STOCK_QTY**              | DNSTOCK.PLAN_QTY
| **ALLOCATION_QTY**         | DNWORKINFO.RESULT_QTY
| **PLAN_QTY**               | 0
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold"> DNStoragePlan </span>
Storage Date will be update by the system in this section represent by <span style="color:green; font-weight:bold">LAST_UPDATE_DATE</span>
<span style="color:red; font-weight:bold; font-size:12px">*If DNSTORAGEPLAN.RESULT_QTY = DNSTORAGEPLAN.PLAN_QTY then update Status to completed</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **DELETE_FLAG**            | 1:Yes   
| **STATUS_FLAG**            | 4: Completed
| **RESULT_QTY**             | DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
| **SHORTAGE_QTY**           | DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
| **LAST_UPDATE_DATE**       | <span style="color:green; font-weight:bold">DNSTORAGEPLAN.PLAN_DAY + Time value (HH:MM:SS.sss)->(System Generated)</span>
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
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
| **EXPIRY_DATE**            | DNWORKINFO.EXPIRY_DAYS
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | Class name

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
| **BUNDLE_ENTERING_QTY**    | DNSTORAGEPLAN.PLANNED_CARTON_QTY
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

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-----------------------------------------------|
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **CARRY_KEY**              | DNCARRYINFO.CARRY_KEY
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **PLAN_AREA_NO**           | DNPALLETITZE.PLAN_AREA_NO
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
| **STORAGE_TYPE**           | DNWORKINFO.STORAGE_TYPE
| **STORAGE_DATE**           | DNSTORAGEPLAN.PLAN_DAY
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

<hr>

##<span style="color:skyblue; font-weight:bold">[Send Report as Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)</span><br>
<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Palletize Start - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)