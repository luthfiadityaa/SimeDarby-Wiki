[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Storage Flow</span>
|Action Name| DNSTORAGEPLAN | DNPALLET | DNWORKINFO | DNWORKLIST | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | DMWAREHOUSE | DMSHELF |
|-----------|--|--|--|--|--|--|--|--|--|--|
| [ID26]() | | | | | | | | INSERT | | |
| [StorageSender]() | | UPDATE | UPDATE | | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE |
| [ID25]() | | | | | UPDATE | | | UPDATE | | |
| [ID64]() | | | | | UPDATE | | | | | |

This Storage Flow is refer to AGC linkage Specification.

#<span style="color:skyblue; font-weight:bold">ID26</span>

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
StationOperator.registArrival()
")]

StorageStationOperator[StorageStationOperator]

releaseCommand-->id26msg-->id26process-->StorageStationOperator
StorageStationOperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let StorageStationOperator picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
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

#<span style="color:skyblue; font-weight:bold">StorageSender</span>

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender` &nbsp;</span>

::: mermaid
flowchart LR
StorageSender-update[("
DNARRIVAL
DNCARRYINFO
DMWAREHOUSE
DMSHELF
DNPALLET
DNSTOCK
DNWORKINFO
")]
StorageSender-input[("
DNARRIVAL
DMSTATION
")]

id05msg("
ID 05
")

StorageSender-input-->StorageSender-->id05msg
StorageSender--> |UPDATE| StorageSender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response</span> and DNARRIVAL will be updated from  <span style="color:green; font-weight:bold">0:Not send to 1:Sent.</span>

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DMWAREHOUSE (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_USED_STATION_NO**   |  Aisle Number where a reserved location belongs to 

####<span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 2:Reserved Location
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCARRYINFO (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| AISLE_STATION_NO           | Aisle Number where a reserved location belongs to
| CMD_STATUS                 | 2:Waiting for response
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNWORKINFO (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| PLAN_LOCATION_NO           | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPALLET (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CURRENT_STATION_NO         | Reserved Location Number
| WH_STATION_NO              | DNCARRYINFO.END_STATION_NO
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNSTOCK (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| AREA_NO                    | DNCARRYINFO.END_STATION_NO
| LOCATION_NO                | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNARRIVAL (INSERT)</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| SEND_FLAG                  | 1:Sent
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

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
id25process-.DELETE.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
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

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

##<span style="color:skyblue; font-weight:bold">Table Value</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CMD_STATUS                 | 4:Pickup completed
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

#<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
StationOperator.registArrival()
")]

StorageStationOperator[StorageStationOperator]

id26msg-->id26process-->StorageStationOperator
StorageStationOperator--> |INSERT| id26-insert
:::

