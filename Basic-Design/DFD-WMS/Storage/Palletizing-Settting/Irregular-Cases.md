[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart TD  
ArrivalAt1101[ID26] --> Cond1{Is Location Full?}  
Cond1 -->|Yes| Await[Await] --> Cond1  
Cond1 -->|No| Cond2{Is No Read?}  
Cond2 -->|Yes| Move1[ID05 ControlInfo:NoRead]  
Cond2 -->|No| Cond3{Is Batch Started?}  
Cond3 -->|Yes| Cond4{Total Actual Carton Qty = 
Planned Carton Qty ?}  
Cond3 -->|No| DataError1[Data Error]  
DataError1[Data Error] --> BatchStart[Batch Start] --> Cond4  
Cond4 -->|Yes| Move2[ID05]  
Cond4 -->|No| DataError2[Data Error] --> BatchEnd[Batch End] --> Move2[ID05]
:::

##<span style="color:skyblue; font-weight:bold">Wait for available location if full</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.as21.common.location.decide.LocationManager&nbsp;</span>

- [Continue Flow to Storage Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/886/Palletize-Start?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-sender%26nbsp%3B%3C/span%3E)

##<span style="color:skyblue; font-weight:bold">No-Read (Go to Reject station)</span>
###<span style="color:skyblue; font-weight:bold">ID26</span>

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

####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

#####<span style="color:skyblue; font-weight:bold">DNArrival</span>
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

#####<span style="color:skyblue; font-weight:bold">DNPallet</span>
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

#####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
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

#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
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

#####<span style="color:skyblue; font-weight:bold">DNStock</span>
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


##<span style="color:skyblue; font-weight:bold">Wait for Batch Start. Treat as Data Error</span>

##<span style="color:skyblue; font-weight:bold">Reverse – Batch End Not Executed Despite Force Completion. Treat as Data Error</span>



