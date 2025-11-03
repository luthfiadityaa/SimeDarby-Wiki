[[_TOC_]]
[[_TOSP_]]

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
| **LAST_UPDATE_PNAME** 	| Class name

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
| **AISLE_STATION_NO**           | Aisle Number where a reserved location belongs to
| **SOURCE_STATION_NO**          | DNARRIVAL.STATION_NO
| **DEST_STATION_NO**            | Reserved Location Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **STATUS_FALG**       | 1: Working 
| **SYSTEM_CONN_KEY**   | Sequence object
| **HARDWARE_TYPE**     | 3: ASRS
| **PLAN_LOCATION_NO**  | Reserved Location Number
| **LAST_UPDATE_DATE**  | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**   | Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **CURRENT_STATION_NO**         | Reserved Location Number
| **WH_STATION_NO**              | Reserved Area Number
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Fiekd Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **AREA_NO**                    | Reserved Area Number
| **LOCATION_NO**                | Reserved Location Number
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
| **Field Name**            | **Insert Value**                                      |
|---------------------------|-------------------------------------------------------|
| **CMD_STATUS**        	| 3:Commanded
| **ERROR_CODE**        	| 0
| **LAST_UPDATE_DATE**  	| SYSTIMESTAMP
| **LAST_UPDATE_PNAME** 	|Class name


###<span style="color:skyblue; font-weight:bold">ID64</span>
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
| **LAST_UPDATE_PNAME** 	| Class name

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
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME        | Class name

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
<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>
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
| **QC_DURATION**            | <span style="color:yellow; font-weight:bold">0: Not Reached</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold">0: Not Reached</span>
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