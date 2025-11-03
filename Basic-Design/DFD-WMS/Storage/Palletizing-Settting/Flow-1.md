##Flow 1 (Refer to AGC Linkage Specs)
###ID64
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
id64process-.U.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##<span style="color:skyblue; font-weight:bold">ID26</span>

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

inoutstationoperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->inoutstationoperator
inoutstationoperator--> |INSERT| id26-insert
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
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

##<span style="color:skyblue; font-weight:bold">Automatic Mode Change Sender</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutomaticModeChangeSender &nbsp;</span>

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

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">3: Commanded to 1: Started. </span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **WORK_TYPE**                  | 2: Storage
| **CMD_STATUS**                 | 1: Started
| **CARRY_FLAG**                 | 1: Storage
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO
| **DEST_STATION_NO**            | Aisle Number where a reserved location belongs to
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **STATUS_FALG**       | 1: Working 
| **SYSTEM_CONN_KEY**   | Sequence object
| **HARDWARE_TYPE**     | 3: ASRS
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME   | Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Fiekd Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | DNCARRYINFO.CARRY_KEY
| **SEND_FLAG**                  | 1:Sent
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

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
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CMD_STATUS**        | 3:Commanded
| **ERROR_CODE**        | 0
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME** |Class name


###ID64
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
id64process-.U.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

###ID33