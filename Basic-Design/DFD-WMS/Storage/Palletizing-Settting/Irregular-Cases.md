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
| **CONTROLINFO**            | <span style="color:red; font-weight:bold">Control Information = "110"</span>
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

##<span style="color:skyblue; font-weight:bold">No-Read (Go to Reject station)</span>

##<span style="color:skyblue; font-weight:bold">Wait for Batch Start. Treat as Data Error</span>

##<span style="color:skyblue; font-weight:bold">Reverse – Batch End Not Executed Despite Force Completion. Treat as Data Error</span>



