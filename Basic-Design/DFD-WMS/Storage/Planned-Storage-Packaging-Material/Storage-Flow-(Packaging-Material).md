[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Storage Flow</span>
|Action Name| DNSTORAGEPLAN | DNPALLET | DNWORKINFO | DNWORKLIST | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | DMWAREHOUSE | DMSHELF | DNSTOCKHISTORY |
|-----------|--|--|--|--|--|--|--|--|--|--|--|
| [ID26 - Dummy Arrival](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26---dummy-arrival%3C/span%3E) | | | | |INSERT | | | INSERT | | | |
| [Automatic Mode Change Sender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/942/?wikiVersion=GBwikiMaster&_a=edit&pagePath=/Basic%20Design/DFD%20WMS/Storage/Planned%20Storage%20Packaging%20Material/Storage%20Flow%20(Packaging%20Material)&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eautomatic-mode-change-sender%3C/span%3E) | | UPDATE | UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25]() | | | | | UPDATE | | | UPDATE | | | |
| [ID64]() | | | | | UPDATE | | | | | | |
| [ID26]() | | | | |INSERT | | | INSERT | | | |
| [Storage Sender]() | | UPDATE| UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | |
| [ID25]() | | | | | UPDATE | | | UPDATE | | | |
| [ID64]() | | | | | UPDATE | | | | | | |
| [ID33]() | UPDATE | UPDATE | UPDATE | | DELETE | UPDATE | INSERT | | | UPDATE | INSERT |

This storage process flow is refer to AGC linkage Specification
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

#<span style="color:skyblue; font-weight:bold">ID26 - Dummy Arrival</span>

::: mermaid
flowchart LR

releaseCommand["
Release Command from Storage Packaging Material Screen
After Completion 
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
DNCARRYINFO
")]

InOutStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->InOutStationOperator
InOutStationOperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let InOutStationOperator picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (INSERT)</span>
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

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|-----------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 26:Direct Transfer
| **CMD_STATUS**                 | 1:Started 
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 3: Direct Transfer
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(7211/7212/7213/7214)**
| **CANCEL_REQUEST**             | 0:Not Requested
| **SCHEDULE_NO**                | Sequence Object
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

##<span style="color:skyblue; font-weight:bold">Automatic Mode Change Sender</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.AutomaticModeChangeSender` &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNARRIVAL
DNCARRYINFO
DMWAREHOUSE
DMSHELF
DNPALLET
DNSTOCK
DNWORKINFO
")]
automaticmodechangesender-input[("
DNARRIVAL
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->automaticmodechangesender-->id05msg
automaticmodechangesender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DMWAREHOUSE (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_USED_STATION_NO**   | Aisle Number where a reserved location belongs to 
| **LAST_USED_STATION_NO_PM**| Aisle Number where a reserved location belongs to 

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Column Name**                | **Description / Notes**                                  |
|--------------------------------|-------------------------------------------------------|
| **STATUS_FLAG**                | 2:Reserved Location
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Field Name**                     | **Description / Notes**                                  |
|------------------------------------|-----------------------------------------------|
| **AISLE_STATION_NO**               | Aisle Number where a reserved location belongs to
| **CMD_STATUS**                     | 2:Waiting for response
| **LAST_UPDATE_DATE**               | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**              | Class name


####<span style="color:skyblue; font-weight:bold">DNWORKINFO (UPDATE)</span>
| **Field Name**                 | **Description / Notes**                                  |
|--------------------------------|-------------------------------------------------------|
| **STATUS_FLAG**                | 1: Working 
| **PLAN_LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | DNCARRYINFO.END_STATION_NO
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK (UPDATE)</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **AREA_NO**                    | DNCARRYINFO.END_STATION_NO
| **LOCATION_NO**                | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (UPDATE)</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID25</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

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
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**                 | 3:Commanded
| **ERROR_CODE**                 | 0
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID64</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(STV)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**                 | 4:Pickup completed
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name


#<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process Storage
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
DNCARRYINFO
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
:::

Continue the process  <span style="color:green; font-weight:bold">storage</span> , AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (INSERT)</span>
| **Column Name**              | **Description / Notes**                                  |
|----------------------------|---------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 
| **CARRY_KEY**              | DNCARRYINFO.CARRY_KEY       
| **BCR_DATA**               | Barcode information from ID26
| **CONTROLINFO**            | Control information from ID26
| **SEND_FLAG**              | 0:Not sent
| **HEIGHT**                 | Dimension Information from ID26
| **WIDTH**                  | Dimension Information From ID26
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (INSERT)</span>
| **Column Name**                | **Description / Notes**                                |
|--------------------------------|-----------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 2: Storage
| **CMD_STATUS**                 | 1:Started 
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 1: Storage
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**            | Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9011/9012/9013/9014)**
| **CANCEL_REQUEST**             | 0:Not Requested
| **SCHEDULE_NO**                | Sequence Object
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

##<span style="color:skyblue; font-weight:bold">Storage Sender</span>

::: mermaid
flowchart LR
storageSender-update[("
DNCARRYINFO
DNWORKINFO
DNPALLET
DNSTOCK
DMSHELF
DNARRIVAL
DMWAREHOUSE
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

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender ` &nbsp;</span>

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DMWAREHOUSE (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **LAST_USED_STATION_NO**       |  Aisle Number where a reserved location belongs to 
| **LAST_USED_STATION_NO_PM**    |  Aisle Number where a reserved location belongs to 

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Field Name**                 | **Insert Value**                              |
|--------------------------------|-----------------------------------------------|
| **STATUS_FLAG**                | 2:Reserved Location
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **CMD_STATUS**                 | 2:Waiting for response
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO (UPDATE)</span>
| **Field Name**                 | **Insert Value**                              |
|--------------------------------|-----------------------------------------------|
| **PLAN_LOCATION_NO**           | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | DNCARRYINFO.END_STATION_NO
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **AREA_NO**                    | DNCARRYINFO.END_STATION_NO
| **LOCATION_NO**                | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (UPDATE)</span>
| **Field Name**                 | **Insert Value**                               |
|--------------------------------|------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

#<span style="color:skyblue; font-weight:bold">ID25</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

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
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CMD_STATUS                 | 3:Commanded
| ERROR_CODE                 | 0
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#<span style="color:skyblue; font-weight:bold">ID64</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment <span style="color:green; font-weight:bold">(SRM)</span> have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
| **Column Name**            | **Description / Notes**                              |
|----------------------------|-------------------------------------------------------|
| CMD_STATUS                 | 4:Pickup completed
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#<span style="color:skyblue; font-weight:bold">ID33</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id33` &nbsp;</span>

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

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 1: Occupied
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CURRENT_STATION_NO         | Location Number information from ID33
| STATUS_FLAG                | 2:Occupied
| ALLOCATION_FLAG            | 0:Not allocated
| LAST_STORED_DATE           | SYSTIMESTAMP
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| RESULT_QTY                 | DNWORKINFO.PLAN_QTY
| RESULT_AREA_NO             | DNWORKINFO.PLAN_AREA_NO
| RESULT_LOCATION_NO         | DNWORKINFO.PLAN_LOCATION_NO
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| STATUS_FLAG                | 4:Completed
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK (UPDATE)</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STORAGE_DAY                | DMWARENAVISYSTEM.WORK_DAY
| NEWEST_STORAGE_DATE        | SYSTIMESTAMP
| STOCK_QTY                  | DNSTOCK.PLAN_QTY
| ALLOCATION_QTY             | DNWORKINFO.RESULT_QTY
| PLAN_QTY                   | 0
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold"> DNSTORAGEPLAN (UPDATE) </span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 4: Completed
| RESULT_QTY                 | DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
| SHORTAGE_QTY               | DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY (INSERT) </span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| INC_DEC_TYPE               | 1: Stock Increase
| JOB_TYPE                   | 2: Storage
| UPDATE_STOCK_QTY           | DNSTOCK.STOCK_QTY
| INC_DEC_QTY                | DNSTOCK.STOCK_QTY
| STOCK_ID                   | DNSTOCK.STOCK_ID
| AREA_NO                    | DNSTOCK.AREA_NO
| LOCATION_NO                | DNSTOCK.LOCATION_NO
| STORAGE_DAY                | DNSTOCK.STORAGE_DAY
| STORAGE_DATE               | DNSTOCK.STORAGE_DATE
| NEWEST_STORAGE_DATE        | DNSTOCK.NEWEST_STORAGE_DATE
| MATERIAL_CODE              | DNSTOCK.MATERIAL_CODE
| PALLET_ID                  | DNSTOCK.PALLET_ID
| BCR_DATA                   | DNPALLET.BCR_DATA
| AREA_TYPE                  | DMAREA_AREA_TYPE
| MATERIAL_NAME              | DMMATERIALMASTER.MATERIAL_NAME
| ENTERING_QTY               | DMMATERIALMASTER.QTY_CRTN
| USER_ID                    | Login info
| USER_NAME                  | Login info
| TERMINAL_NO                | Login info
| TERMINAL_NAME              | Login info
| IP_ADDRESS                 | Login info
| EXPIRY_DATE                | DNWORKINFO.EXPIRY_DAYS
| REGIST_DATE                | SYSTIMESTAMP
| REGIST_PNAME               | Class name

####<span style="color:skyblue; font-weight:bold">DNINOUTRESULT (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| RESULT_KIND                | 1:Storage(Stock+)
| STATION_NO                 | DNCARRYINFO.SOURCE_STATION_NO
| LOCATION_NO                | DNCARRYINFO.DEST_STATION_NO
| WH_STATION_NO              | DNPALLET.WH_STATION_NO
| AISLE_STATION_NO           | DNCARRYINFO.AISLE_STATION_NO
| WORK_TYPE                  | DNCARRYINFO.WORK_TYPE
| RETRIEVAL_DETAIL           | DNCARRYINFO.RETRIEVAL_DETAIL
| WORK_NO                    | DNCARRYINFO.WORK_NO
| PALLET_ID                  | DNPALLET.PALLET_ID
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| RESTORING_FLAG             | DNCARRYINFO.RESTORING_FLAG
| WORK_DAY                   | DMWARENAVISYSTEM.WORK_DAY
| REMOVE_FLAG                | 00:Normal
| REGIST_PNAME               | Class name
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNHOSTSEND (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| WORK_DAY                   | DNWORKINFO.WORK_DAY
| JOB_NO                     | DNWORKINFO.JOB_NO
| COLLECT_JOB_NO             | DNWORKINFO.COLLECT_JOB_NO
| SETTING_UNIT_KEY           | DNWORKINFO.SETTING_UNIT_KEY
| JOB_TYPE                   | DNWORKINFO.JOB_TYPE
| STATUS_FLAG                | DNWORKINFO.STATUS_FLAG
| HARDWARE_TYPE              | DNWORKINFO.HARDWARE_TYPE
| PLAN_UKEY                  | DNWORKINFO.PLAN_UKEY
| STOCK_ID                   | DNWORKINFO.STOCK_ID
| SYSTEM_CONN_KEY            | DNWORKINFO.SYSTEM_CONN_KEY
| PLAN_DAY                   | DNWORKINFO.PLAN_DAY
| BATCH_NO                   | DNWORKINFO.BATCH_NO
| PLAN_AREA_NO               | DNWORKINFO.PLAN_AREA_NO
| PLAN_LOCATION_NO           | DNWORKINFO.PLAN_LOCATION_NO
| MATERIAL_CODE              | DNWORKINFO.MATERIAL_CODE
| MATERIAL_NAME              | DMMATERIALMASTER.MATERIAL_NAME
| UOM                        | MMATERIALMASTER.UOM
| ENTERING_QTY               | DMMATERIALMASTER.QTY_CRTN
| BUNDLE_ENTERING_QTY        | DNPALLETIZE.PLANNED_CARTON_QTY
| PLAN_QTY                   | DNWORKINFO.PLAN_QTY
| RESULT_QTY                 | DNWORKINFO.RESULT_QTY
| SHORTAGE_QTY               | DNWORKINFO.SHORTAGE_QTY
| RESULT_AREA_NO             | DNWORKINFO.RESULT_AREA_NO
| RESULT_LOCATION_NO         | DNWORKINFO.RESULT_LOCATION_NO
| RESULT_LOT_NO              | DNWORKINFO.RESULT_LOT_NO
| USER_ID                    | DNWORKINFO.USER_ID
| TERMINAL_NO                | DNWORKINFO.TERMINAL_NO
| WORK_SECOND                | DNWORKINFO.WORK_SECOND
| USER_NAME                  | DCUSER.USERNAME
| REPORT_FLAG                | 1:Reported
| REGIST_DATE                | SYSTIMESTAMP
| REGIST_PNAME               | Class name
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

