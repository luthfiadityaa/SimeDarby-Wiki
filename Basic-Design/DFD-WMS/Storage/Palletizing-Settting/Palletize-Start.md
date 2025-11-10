 [[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletize Start database flow</span>
**Abbreviation:**
- **PLTZ** : DNPALLETIZE  
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
- **PLTZH** : DNPALLETIZEHISTORY  

| Action Name                 | STRP   | WRKI   | WRKL   | PLLT   | CRYI   | STCK   | HSTS   | ARRVL  | WRHS   | SHLF   | STCH   | MTMS   | STSN   | 
|-----------------------------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|
| Palletize Start - SetF2(1)  | INSERT |        |        |        |        |        |        |        |        |        |        | SELECT | SELECT |    
| ID26(2)                     |        | INSERT |        | INSERT | INSERT | INSERT |        | INSERT |        |        |        |        |        |    
| Storage Sender(3)           |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |    
| ID25(4)                     |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |    
| ID26(5)                     |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |    
| Storage Sender(6)           |        | UPDATE |        |        | UPDATE | UPDATE |        | UPDATE | UPDATE | UPDATE |        |        |        |    
| ID25(7)                     |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |    
| **Flow 1**                  |        |        |        |        |        |        |        |        |        |        |        |        |        |
| ID64(8)                     |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID26(9)                     |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |  
| Storage Sender(10)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25(11)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64(12)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| **Flow 2**                  |        |        |        |        |        |        |        |        |        |        |        |        |        |   
| ID64(8)                     |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |   
| ID26(9)                     |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |
| Storage Sender(10)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        | 
| ID25(11)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64(12)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID64(13)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| ID26(14)                    |        |        |        | UPDATE | UPDATE |        |        | INSERT |        |        |        |        |        |       
| Storage Sender(15)          |        |        |        |        | UPDATE |        |        | UPDATE |        |        |        |        |        |
| ID25(16)                    |        |        |        |        | UPDATE |        |        | DELETE |        |        |        |        |        |
| ID64(17)                    |        |        |        |        | UPDATE |        |        |        |        |        |        |        |        |
| **Last Process**            |        |        |        |        |        |        |        |        |        |        |        |        |        |   
| ID33(18)                    | UPDATE | UPDATE | UPDATE | INSERT | UPDATE | DELETE | INSERT |        |        |        | INSERT |        |        |  


#<span style="color:skyblue; font-weight:bold">Palletize Start - Set(F2)</span>

Palletize Start is used to set the information of stock which will be palletized by robot automatically. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)

![image.png](/.attachments/image-cf980be6-3128-4022-828d-b2a0fa92b1a6.png)

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
        Planned Carton Qty
        Storage Date/Time
        Qty kg/crtn
        Qty crtn/PL
        UOM
        Tempering Period
        Storage Location
        Expiry Days
    ]

    tableList-insert[("
        DNPALLETIZE
        DNSTORAGEPLAN
    ")]

     tableList-select[("
        DMMATERIALMASTER
        DMSTATION
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Selected Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- Selected Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.
- <span style="color:green; font-weight:bold">Planned Carton</span> must be greater than <span style="color:green; font-weight:bold">0</span>.
- <span style="color:green; font-weight:bold">Quantity (Carton per Pallet)</span> must be greater than <span style="color:green; font-weight:bold">0</span>.
- <span style="color:green; font-weight:bold">Planned Carton</span> must be greater than or equal to <span style="color:green; font-weight:bold">Quantity (Carton per Pallet)</span>.
- Material Code exists in <span style="color:green; font-weight:bold">DMMaterialMaster</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-----------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                      
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | DNPALLETIZE.BATCH_PALLET_START                                                       
| **PLAN_AREA_NO**           | DNPALLETIZE.STORAGE_LOCATION                             
| **MATERIAL_CODE**          | DNPALLETIZE.MATERIAL_CODE                                                  
| **PLAN_QTY**               | DNPALLETIZE.QTY_CRTN_PL                                                 
| **REPORT_FLAG**            | 0:Not Reported                                                   
| **STORING_PAIR_KEY**       | <span style="color:green; font-weight:bold">DNPALLETIZE.MATERIAL_CODE + DNPALLETIZE.BATCH_NO</span>
| **STATION_NO**                 | Value from screen (Station No)
| **CURRENT_STATUS**             | Value from screen (Current Status)
| **MATERIAL_CODE**              | Value from screen (Material Code)
| **MATERIAL_NAME**              | Value from screen (Material Name)
| **MATERIAL_TYPE**              | Value from screen (Material Type)
| **BATCH_NO**                   | Value from screen (BATCH_NO)
| **PLANNED_CARTON_QTY**         | Value from screen (PLANNED CARTON QTY)
| **STORAGE_DATE_TIME**          | Value from screen (Storage Date/Time) 
| **QTY_KG_CRTN**                | Value from screen (Qty Kg/Crtn)
| **QTY_CRTN_PL**                | Value from screen (Qty Crtn/PL)
| **UOM**                        | Value from screen (UOM)
| **TEMPERING_PERIOD**           | Value from screen (Tempering Period)
| **STORAGE_LOCATION**           | Value from screen (Storage Location) ⟶ <span style="color:green; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>
| **EXPIRY_DAYS**                | Value from screen (Expiry Days)
| **STORAGE_QTY**                | <span style="color:green; font-weight:bold">PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY</span>
| **TOTAL_ACTUAL_CARTON_QTY**    | <span style="color:green; font-weight:bold">TOTAL_ACTUAL_CARTON_QTY + QTY_KG_CRTN</span>
| **MAX_PALLET_BATCH_END**       | <span style="color:green; font-weight:bold">PLANNED_CARTON_QTY / QTY_CRTN_PL</span> 
| **BATCH_PALLET_START**         | <span style="color:green; font-weight:bold">Value from screen (Storage Date/Time) + TimeStamp (System Generated)</span>
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

#<span style="color:skyblue; font-weight:bold">Release Command from Palletize Robot - Dummy Arrival</span>
  After palletizing is completed, the palletizing robot sends <span style="color:green; font-weight:bold">“Release Command”</span> signal via conveyor interlock.

<br>
<hr style="border: 2px solid red;">

#Irregular Cases
Continue this flow if occurs.
- [Wait for available location if full](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ewait-for-available-location-if-full%3C/span%3E)
- [No-Read (Go to Reject station)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eno-read-(go-to-reject-station)%3C/span%3E)
- [Wait for Batch Start. Treat as Data Error](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ewait-for-batch-start.-treat-as-data-error%3C/span%3E)
<hr style="border: 2px solid red;">

##<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

releaseCommand["
Release Command from Palletize Robot
After Completion 
"]

id26msg("
ID 26
")

id26-insert[("
   DNARRIVAL
   DNWORKINFO
   DNPALLET
   DNCARRYINFO
   DNSTOCK		
")]

storageStationOperator[StorageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and Storage Station Operator will execute the receive task based on information in received ID26. While Storage Station Operator processes ID26, it will create an Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-----------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**            | **Insert Vlaue**                               |
|----------------------------|-----------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | DNPALLETIZE.STATION_NO                                                       
| **WH_STATION_NO**          | DNPALLETIZE.STORAGE_LOCATION                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                     
| **EMPTY_FLAG**             | 0:Normal Pallet 
| **ALLOCATION_FLAG**        | 1:Allocated
| **SOFT_ZONE_ID**           | DMMATERIALMASTER.SOFT_ZONE_ID                                                       
| **BCR_DATA**               | DNARRIVAL.BCR_DATA                                                 
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Filed Name**            | **Insert Value**                               |
| ----------------------------|----------------------------------------------|
| **JOB_NO**                 | Sequence Object
| **SETTING_UNIT_KEY**       | Sequence Object
| **COLLECT_JOB_NO**         | Sequence Object
| **JOB_TYPE**               | 02:Storage
| **STATUS_FLAG**            | 0:Not Started
| **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
| **STOCK_ID**               | Sequence Object
| **PLAN_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **BATCH_NO**               | DNPALLETIZE.BATCH_NO
| **PLAN_AREA_NO**           | DNPALLETIZE.STORAGE_LOCATION 
| **MATERIAL_CODE**          | DNPALLETIZE.MATERIAL_CODE 
| **PLAN_QTY**               | DNPALLETIZE.QTY_CRTN_PL 
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **USER_ID**                | Login Info
| **STORAGE_LOCATION_FROM**  | DNPALLETIZE.STATION_NO 
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 26:Direct Transfer
| **CMD_STATUS**                 | 1:Started 
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 3: Direct Transfer
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO ⟶ <span style="color:green; font-weight:bold">(1101/1102/1103/1104/1105)</span>
| **DEST_STATION_NO**            | <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (1111/1112/1113/1114/1115)</span>
| **CANCEL_REQUEST**             | 0:Not Requested
| **SCHEDULE_NO**                | Sequence Object
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **field Name**            | **Insert Value**                               |
|----------------------------|-----------------------------------------------|
| **STOCK_ID**               | Sequence Object   
| **AREA_NO**                | DNPALLETIZE.STORAGE_LOCATION
| **STORAGE_TYPE**           | 2:New 
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **PLAN_QTY**               | DNPALLETIZE.QTY_CRTN_PL 
| **PALLET_ID**              | Sequence Object
| **BATCH_NO**               | DNPALLETIZE.BATCH_NO 
| **TEMPERING_PERIOD**       | DNPALLETIZE.TEMPERING_PERIOD
| **STORING_PAIR_KEY**       | <span style="color:green; font-weight:bold">DNSTORAGEPLAN.STORING_PAIR_KEY</span>
| **EXPIRY_DATE**            | DNPALLETIZE.EXPIRY_DAYS
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

##<span style="color:skyblue; font-weight:bold">Storage Sender</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

::: mermaid
flowchart LR
storageSender-update[("
DNCARRYINFO
DNARRIVAL
")]
storageSender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

storageSender-input-->storageSender-->id05msg
storageSender--> |UPDATE| storageSender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 2:Waiting for response
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**         | Class name

##<span style="color:skyblue; font-weight:bold">ID25</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**         | **Insert Value**                               |
|-----------------------|-------------------------------------------------|
| **CMD_STATUS**        | 3:Commanded
| **ERROR_CODE**        | 0
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** |Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process Direct 
Transfer
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNPALLET
DNCARRYINFO
")];

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and Storage Station Operator will execute the receive task based on information in received ID26. While Storage Station Operator processes ID26, it will create an Arrival record..

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999      
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**            | **Insert Vlaue**                               |
|----------------------------|-----------------------------------------------|                                                
| **CURRENT_STATION_NO**     | DNARRIVAL.STATION_NO                                          
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName


####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **WORK_TYPE**                  | 26:Direct Transfer
| **CMD_STATUS**                 | 1:Started 
| **CARRY_FLAG**                 | 3: Direct Transfer
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO ⟶ <span style="color:green; font-weight:bold">(1111, 1112, 1113, 1114, 1115)</span>
| **DEST_STATION_NO**            | <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7101, 7102, 7103, 7104, 7105, 7106, 7107, 7108, 7109, 7110)</span>
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

##<span style="color:skyblue; font-weight:bold">Storage Sender&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DMWAREHOUSE
DMSHELF
DNSTOCK
DNARRIVAL
DNCARRYINFO
DNWORKINFO
DNPALLETIZE
DNSTORAGEPLAN
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->storageSender-->id05msg
storageSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DMWarehouse</span>
| **Field Name**             | **Insert Value**                              |
|----------------------------|-----------------------------------------------|
| **LAST_USED_STATION_NO**   | Aisle Number where a reserved location belongs to 
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DMShelf</span>
| **Field Name**             | **Insert Value**                              |
|----------------------------|-----------------------------------------------|
| **STATUS_FLAG**            | 2:Reserved Location
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **AISLE_STATION_NO**      | Aisle Number where a reserved location belongs to
| **CMD_STATUS**            | 2:Waiting for response
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **PLAN_LOCATION_NO**      | Reserved Location Number
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **CARRY_KEY**             | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**             | 1:Sent
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

<span style="color:green; font-weight:bold">Storage Date will be update by the system in this section.</span>
####<span style="color:skyblue; font-weight:bold">DNPALLETIZE</span>
| **Field Name**             | **Insert Value**                              |
|----------------------------|-----------------------------------------------|
| **STORAGE_DATE_TIME**      | <span style="color:green; font-weight:bold">DNPALLETIZE.STORAGE_DATE_TIME + Time value (HH:MM:SS.sss)->(System Generated)</span>

####<span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN</span>
| **Field Name**             | **Insert Value**                              |
|----------------------------|-----------------------------------------------|
| **LAST_UPDATE_DATE**        | <span style="color:green; font-weight:bold">DNPALLETIZE.STORAGE_DATE_TIME</span>

##<span style="color:skyblue; font-weight:bold">ID25&nbsp;</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**        | **Insert Value**                                      |
|---------------------- |-------------------------------------------------------|
| **CMD_STATUS**        | 3:Commanded
| **ERROR_CODE**        | 0
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** |Class name

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ###[Flow 1](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eflow-1-(refer-to-agc-linkage-specs)%3C/span%3E)
- ###[Flow 2](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Settting/Palletize%20Start&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eflow-2-(refer-to-agc-linkage-specs)%3C/span%3E)

<hr>

#<span style="color:skyblue; font-weight:bold">Flow 1 (Refer to AGC Linkage Specs)</span>
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)
##<span style="color:skyblue; font-weight:bold">ID64</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(STV)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process from Direct Transfer
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNPALLET
DNCARRYINFO
")]


storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**             | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | Reserved Area Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **WORK_TYPE**                  | 2:Storage
| **CMD_STATUS**                 | 1:Started 
| **CARRY_FLAG**                 | 1:Storage
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO
| **DEST_STATION_NO**            | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

##<span style="color:skyblue; font-weight:bold">StorageSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNARRIVAL
DNCARRYINFO
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->storageSender-->id05msg
storageSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **CMD_STATUS**            | 2: Waiting for response
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **CARRY_KEY**             | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**             | 1:Sent
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name


##<span style="color:skyblue; font-weight:bold">&nbsp;ID25&nbsp;</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                                      |
|---------------------------|-------------------------------------------------------|
| **CMD_STATUS**            | 3:Commanded
| **ERROR_CODE**            | 0
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     |Class name

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">SRM</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Flow 2 (Refer to AGC Linkage Specs)</span>
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(STV)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>


::: mermaid
flowchart LR

releaseCommand["
Continue the Process Direct 
Transfer
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNPALLET
DNCARRYINFO
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and Storage Station Operator will execute the receive task based on information in received ID26. While Storage Station Operator processes ID26, it will create an Arrival record..

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**            | **Insert Vlaue**                               |
|----------------------------|-----------------------------------------------|                                                
| **CURRENT_STATION_NO**     | DNARRIVAL.STATION_NO                                          
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **WORK_TYPE**                  | 26:Direct Transfer
| **CMD_STATUS**                 | 1:Started 
| **CARRY_FLAG**                 | 3: Direct Transfer
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO ⟶ <span style="color:green; font-weight:bold">(7107, 7108, 7109, 7110)</span>
| **DEST_STATION_NO**            | <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7211, 7212, 7213, 7214)</span>
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName


##<span style="color:skyblue; font-weight:bold">StorageSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNARRIVAL
DNCARRYINFO
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->storageSender-->id05msg
storageSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Storage Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1: Started to 2: Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**                 | 2: Waiting for response
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

##<span style="color:skyblue; font-weight:bold">ID25&nbsp;</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**        | 3:Commanded
| **ERROR_CODE**        | 0
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** |Class name

##<span style="color:skyblue; font-weight:bold">ID64</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">SRM</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** 	| Class name

##<span style="color:skyblue; font-weight:bold">ID64</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">STV</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                              |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** 	| Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process from 
Direct Transfer
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNPALLET
DNCARRYINFO
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | 99999999       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**             | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | Reserved Area Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **WORK_TYPE**                  | 2:Storage
| **CMD_STATUS**                 | 1:Started 
| **CARRY_FLAG**                 | 1:Storage
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO
| **DEST_STATION_NO**            | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

##<span style="color:skyblue; font-weight:bold">StorageSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNARRIVAL
DNCARRYINFO
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->storageSender-->id05msg
storageSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **CMD_STATUS**            | 2: Waiting for response
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|------------------------------------------------|
| **CARRY_KEY**             | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**             | 1:Sent
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name


##<span style="color:skyblue; font-weight:bold">&nbsp;ID25&nbsp;</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                                      |
|---------------------------|-------------------------------------------------------|
| **CMD_STATUS**            | 3:Commanded
| **ERROR_CODE**            | 0
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     |Class name

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">SRM</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
| **Field Name**            | **Insert Value**                               |
|---------------------------|-----------------------------------------------|
| **CMD_STATUS**            | 4:Pickup completed
| **LAST_UPDATE_DATE**      | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**     | Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-0f17bef0-54b4-4a5a-9227-3908b6fe5b7f.png)


Table Impact:
- DNStock
- DNStockHistory
- DNSTORAGEPLAN

##<span style="color:skyblue; font-weight:bold">[Click this to Continue flow, If comming from Batch End Process](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid33%3C/span%3E)</span><br>
<hr>

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
")]
id33-update[("
DNPALLET
DMSHELF
DNSTOCK
DNWORKINFO
DNWORKLIST
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
| **STOCK_QTY**              | DNWORKINFO.RESULT_QTY
| **ALLOCATION_QTY**         | DNWORKINFO.RESULT_QTY
| **PLAN_QTY**               | 0
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
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
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
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
| **STOCK_STATUS**           | <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached </span>
| **QC_FLAG**                | <span style="color:green; font-weight:bold">0: Not Done</span>
| **EXPIRY_DATE**            | DNWORKINFO.EXPIRY_DAYS
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | Class name

####<span style="color:skyblue; font-weight:bold">DNHostSend</span>
| **Field Name**             | **Insert Value**                                      |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | DMWARENAVISYSTEM.WORK_DAY
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
| **REPORT_FLAG**            | 0: Not Reported
| **REGIST_DATE**            | SYSTIMESTAMP
| **REGIST_PNAME**           | Class name
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | Class name

<hr>

##<span style="color:skyblue; font-weight:bold">[Send Report as Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)</span><br>

<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Batch End - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)