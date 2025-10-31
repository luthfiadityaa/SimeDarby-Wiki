[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletize Start database flow</span>

| Action Name                        | DNPALLETIZE | DNSTORAGEPLAN | DNWORKINFO | DNWORKLIST | DNPALLET | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | 
|-|-|-|-|-|-|-|-|-|-|
| Palletize Start - Set(F2)[(1)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Setting&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Epalletizing-setting-set(f2)%3C/span%3E) | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | |
| ID26(2) | | | | | | | | | INSERT |
| Automatic Mode Change Sender(3) | | | | | | | | | |
| ID26(4) | | | | | | | | | INSERT |
| Automatic Mode Change Sender(5 | | | | | | | | | |
| ID25(6) | | | | | | | | | |
| ID64(7) | | | | | | | | | |
| ID33(8) | | | | | | | | | |

#<span style="color:skyblue; font-weight:bold">Palletize Start - Set(F2)</span>

Palletize Start is used to set the information of stock which will be palletized by robot automatically. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as Production Storage Result.

![image.png](/.attachments/image-cf980be6-3128-4022-828d-b2a0fa92b1a6.png)

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
        Planned Carton Qty<br>
        Qty kg/crtn<br>
        Qty crtn/PL<br>
        UOM<br>
        Tempering Period<br>
        Storage Location<br>
        Expiry Days<br>
    ]

    tableList-insert[("
        DNPALLETIZE<br>
        DNSTORAGEPLAN<br>
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
        DNCARRYINFO<br>
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert

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
- Material Code exists in <span style="color:green; font-weight:bold">DMMaterialMaster</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNPalletize</span>

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
| **STORAGE_LOCATION**           | Value from screen (Storage Location) ➡️ FGW1 (Ambient/9002) or FGW2 (Tempering/9001)
| **EXPIRY_DAYS**                | Value from screen (Expiry Days)
| **STORAGE_QTY**                | PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY
| **TOTAL_ACTUAL_CARTON_QTY**    | TOTAL_ACTUAL_CARTON_QTY + QTY_KG_CRTN
| **MAX_PALLET_BATCH_END**       | PLANNED_CARTON_QTY / QTY_CRTN_PL 
| **BATCH_PALLET_START**         | Value from screen (Storage Date/Time) + TimeStamp (System Generated)
| **BATCH_PALLET_END**           | 
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | Value from screen (Storage Date/Time)                                                      
| **VENDOR_CODE**            |                                                       
| **COMPANY_CODE**           |                                                       
| **RECEIVE_TICKET_NO**      |                                                       
| **RECEIVE_LINE_NO**        |                                                       
| **RECEIVE_TICKET_DATE**    |                                                       
| **BRANCH_NO**              |                                                         
| **PLAN_AREA_NO**           | Value from screen (Storage Location) ➡️ FGW1 (Ambient/9002) or FGW2 (Tempering/9001)                                                       
| **PLAN_LOCATION_NO**       |                                                       
| **MATERIAL_CODE**          | Value from screen (Material Code)                                                      
| **PLAN_LOT_NO**            |                                                       
| **NOTE**                   |                                                       
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)                                                       
| **PROCESS_QTY**            | Value from screen (Qty crtn/PL)                                         
| **RESULT_QTY**             |                                                       
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            | 0:Not Reported                                                      
| **WORK_DAY**               |                                                       
| **REGIST_KIND**            |                                                       
| **BCR_DATA**               | Value from Barcode Scanner                                                       
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)  
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | Selected Value from screen ➡️ (1101/1102/1103/1104/1105)                                                       
| **WH_STATION_NO**          | Value from screen (Storage Location) ➡️ FGW1 (Ambient/9002) or FGW2 (Tempering/9001)                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                      
| **ALLOCATION_FLAG**        |                                                       
| **EMPTY_FLAG**             | 0:Normal Pallet                                                       
| **HEIGHT**                 |                                                       
| **WIDTH**                  |                                                       
| **SOFTZONE_ID**            |                                                       
| **BCR_DATA**               | Value from Barcode Scanner                                                      
| **LAST_STORED_DATE**       |                                                       
| **AISLE_DISPERSE_KEY**     |                                                       
| **AISLE_COLLECT_KEY**      |                                                       
| **LEVEL_DISPERSE_KEY**     |                                                       
| **COLLECT_BATCH_NO**       |                                                       
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **STOCK_ID**               | Sequence Object   
| **AREA_NO**                | DNSTORAGEPLAN.PLAN_AREA_NO
| **LOCATION_NO**            | 222222222222
| **LOT_NO**                 | 
| **STORAGE_TYPE**           | 2:New 
| **NOTE**                   |
| **STORAGE_DAY**            |
| **STORAGE_DATE**           |
| **NEWEST_STORAGE_DATE**    |
| **RETRIEVAL_DAY**          |
| **INVENTORY_DAY**          |
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
| **PALLET_ID**              | Sequence Object
| **BATCH_NO**               | Value from screen (Batch)
| **STOCK_STATUS**           | UU: Unrestricted Used
| **QC_DURATION**            | 0: Not Reached
| **TEMPERING_FLAG**         | 0: Not Reached
| **QC_FLAG**                | Not Done
| **TEMPERING_PERIOD**       | Value from screen (Tempering Period)
| **STORING_PAIR_KEY**       | Value from screen (Material Code + Batch #)
| **EXPIRY_DATE**            | Value from Screen (Expiry Days)
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Column Name**            | **Description / Notes**                               |
| ----------------------------|-------------------------------------------------------|
| **JOB_NO**                 | Sequence Object
| **SETTING_UNIT_KEY**       | Sequence Object
| **COLLECT_JOB_NO**         | Sequence Object
| **JOB_TYPE**               | 02:Storage
| **STATUS_FLAG**            | 0:Not Started
| **RFT_STATUS_FLAG**        |
| **HARDWARE_TYPE**          |
| **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
| **STOCK_ID**               | Sequence Object
| **SYSTEM_CONN_KEY**        |
| **PLAN_DAY**               | DMWARENAVISYSTEM.WORK_DAY
| **VENDOR_CODE**            |
| **RECEIVE_TICKET_NO**      |
| **RECEIVE_LINE_NO**        |
| **RECEIVE_BRANCH_NO**      |
| **COMPANY_CODE**           |
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | Value from screen (Batch #)
| **ORDER_NO**               |
| **ORDER_SERIAL_NO**        |
| **PLAN_AREA_NO**           | Value from screen (Storage Location)
| **PLAN_LOCATION_NO**       |
| **MATERIAL_CODE**          | Value from screen (Material Code)
| **PLAN_LOT_NO**            |
| **NOTE**                   |
| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
| **RESULT_QTY**             | 
| **SHORTAGE_QTY**           |
| **RESULT_AREA_NO**         |
| **RESULT_LOCATION_NO**     |
| **RESULT_NO**              |
| **WORK_NOTE**              |
| **SKIP_CNT**               |
| **WORK_DAY**               |
| **USER_ID**                | Login Info
| **TERMINAL_NO**            |
| **WORK_SECOND**            |
| **STORAGE_LOCATION_TO**    | 
| **STORAGE_LOCATION_FROM**  | Selected Value from screen (1101/1102/1103/1104/1105)
| **DOCK_NO**                |
| **TRUCK_PLATE_NO**         |
| **STORING_PAIR_KEY**       |
| **NEW_STORING_PAIR_KEY**   |
| **PRIORITY_TYPE**          |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |   
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 26:Direct Transfer
| **GROUP_NO**                   |
| **GROUP_SEQNO**                |
| **CMD_STATUS**                 | 1:Started 
| **NEXT_RESERVE_FLAG**          |
| **PRIORITY**                   | 2:Normal
| **RESTORING_FLAG**             | 0:Not Restore to Original Location
| **CARRY_FLAG**                 | 1:Storage
| **RETRIEVAL_STATION_NO**       | 
| **RETRIEVAL_DETAIL**           |
| **WORK_NO**                    | Sequence Object
| **SOURCE_STATION_NO**          | DNPALLET.CURRENT_STATION_NO
| **DEST_STATION_NO**            | Decided Value based on from SOURCE_STATION_NO ➡️ (1111/1112/1113/1114/1115)
| **ARRIVAL_DATE**               |
| **CONTROLINFO**                |
| **CANCEL_REQUEST**             | 0:Not Requested
| **CANCEL_REQUEST_DATE**        |
| **SCHEDULE_NO**                | Sequence Object
| **AISLE_STATION_NO**           |
| **END_STATION_NO**             | DNWORKINFO.PLAN_AREA_NO
| **RESERVE_SHELF_NO**           |
| **ERROR_CODE**                 |
| **MAINTENANCE_TERMINAL**       |
| **REJECT_FACTOR**              |
| **AGC_DATE**                   |
| **WAIT_REASON**                |
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **CARRY_KEY**              |
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **PALLET_ID**              | DNPALLET.PALLET_ID
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **COMPANY_CODE**           |
| **COMPANY_NAME**           |
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **ORDER_NO**               |
| **PLAN_AREA_NO**           |
| **PLAN_LOCATION_NO**       |
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
| **JAN**                    |
| **CASE_ITF**               |
| **BUNDLE_ITF**             |
| **ENTERING_QTY**           |
| **BUNDLE_ENTERING_QTY**    |
| **PLAN_LOT_NO**            |
| **STORAGE_TYPE**           | DNWORKINFO.STORAGE_TYPE
| **STORAGE_DATE**           | DNSTORAGEPLAN.PLAN_DAY
| **NOTE**                   |
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **STOCK_QTY**              | 0
| **ALLOCATION_QTY**         | 0
| **WORK_NOTE**              |
| **PRIORITY**               |
| **RETRIEVAL_STATION_NO**   |
| **RETRIEVAL_DETAIL**       |
| **WORK_NO**                |
| **SOURCE_STATION_NO**      |
| **DEST_STATION_NO**        |
| **SCHEDULE_NO**            |
| **SCHEDULE_NO**            |
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            |
| **DOCK_NO**                |
| **TRUCK_PLATE_NO**         |
| **STORING_PAIR_KEY**       |
| **NEW_STORING_PAIR_KEY**   |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |   
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **WORK_DAY**               | 
| **JOB_NO**                 | DNWORKINFO.JOB_NO
| **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
| **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
| **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
| **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
| **HARDWARE_TYPE**          |
| **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
| **STOCK_ID**               | DNWORKINFO.STOCK_ID
| **SYSTEM_CONN_KEY**        | 
| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
| **VENDOR_CODE**            |
| **VENDOR_NAME**            |
| **RECEIVE_TICKET_NO**      |
| **RECEIVE_LINE_NO**        |
| **RECEIVE_BRANCH_NO**      |
| **COMPANY_CODE**           |
| **COMPANY_NAME**           |
| **SHIP_TICKET_NO**         |
| **SHIP_LINE_NO**           |
| **SHIP_BRANCH_NO**         |
| **BATCH_NO**               | DNWORKINFO.BATCH_NO
| **ORDER_NO**               |
| **PLAN_AREA_NO**           |
| **PLAN_LOCATION_NO**       |
| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
| **JAN**                    |
| **CASE_ITF**               |
| **BUNDLE_ITF**             |
| **ENTERING_QTY**           |
| **BUNDLE_ENTERING_QTY**    |
| **PLAN_LOT_NO**            |
| **NOTE**                   |
| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
| **RESULT_QTY**             |
| **SHORTAGE_QTY**           |
| **RESULT_AREA_NO**         |
| **RESULT_LOCATION_NO**     |
| **RESULT_LOT_NO**          |
| **WORK_NOTE**              |
| **REPORT_FLAG**            | DNSTORAGEPLAN.REPORT_FLAG
| **USER_ID**                | DNWORKINFO.USER_ID
| **USER_NAME**              | DCUSER.USER_NAME
| **TERMINAL_NO**            |
| **WORK_SECOND**            |
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

#<span style="color:skyblue; font-weight:bold">Release Command from Palletize Robot</span>
  After palletizing is completed, the palletizing robot sends <span style="color:green; font-weight:bold">“Release Command”</span> signal via conveyor interlock.

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
")]

inoutstationoperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->inoutstationoperator
inoutstationoperator--I-->id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Column Name**            | **Description / Notes**                               |
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

#<span style="color:skyblue; font-weight:bold">Automatic Mode Change Sender</span>
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

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DMWarehouse</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **STATION_NO**             | 
| **LAST_USED_STATION_NO**   | Aisle Number where a reserved location belongs to 

###<span style="color:skyblue; font-weight:bold">DMShelf</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 2:Reserved Location
| LAST_UPDATE_DATE           | SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| DEST_STATION_NO            | Reserved Location Number
| AISLE_STATION_NO           | Aisle Number where a reserved location belongs to
| CMD_STATUS                 | 2:Waiting for response
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| PLAN_LOCATION_NO           | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CURRENT_STATION_NO         | Reserved Location Number
| WH_STATION_NO              | 9000
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

###<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| AREA_NO                    | 9000
| LOCATION_NO                | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNARRIVAL
- CARRY_KEY = DNCARRYINFO.CARRY_KEY
- SEND_FLAG = 1:Sent
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID25

jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process

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
id25process-.U.->id25-update
id25process-.D.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

## DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

# ID64
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

## DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name


# ID33
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
id33process--I-->id33-insert
id33process-.U.->id33-update
id33process-.D.->id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

## DMSHELF
- STATUS_FLAG = 1:Occupied
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNPALLET
- CURRENT_STATION_NO = Location Number information from ID33
- STATUS_FLAG = 2:Occupied
- ALLOCATION_FLAG = 0:Not allocated
- LAST_STORED_DATE = SYSTIMESTAMP
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNWORKINFO
- RESULT_QTY = DNWORKINFO.PLAN_QTY
- RESULT_AREA_NO = DNWORKINFO.PLAN_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- RESULT_LOT_NO = DNWORKINFO.PLAN_LOT_NO
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- STATUS_FLAG = 4:Completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTOCK
- STORAGE_DAY = DMWARENAVISYSTEM.WORK_DAY
- NEWEST_STORAGE_DATE = SYSTIMESTAMP
- STOCK_QTY = DNSTOCK.PLAN_QTY
- ALLOCATION_QTY = DNWORKINFO.RESULT_QTY
- PLAN_QTY = 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTORAGEPLAN (If DNSTORAGEPLAN.RESULT_QTY = DNSTORAGEPLAN.PLAN_QTY then update Status to completed)
- STATUS_FLAG = 4:Completed
- RESULT_QTY = DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
- SHORTAGE_QTY = DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTOCKHISTORY
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- INC_DEC_TYPE = 1:Stock Increase
- JOB_TYPE = 22:Unplanned Storage
- UPDATE_STOCK_QTY = DNSTOCK.STOCK_QTY
- INC_DEC_QTY = DNSTOCK.STOCK_QTY
- STOCK_ID = DNSTOCK.STOCK_ID
- AREA_NO = DNSTOCK.AREA_NO
- LOCATION_NO = DNSTOCK.LOCATION_NO
- LOT_NO = DNSTOCK.LOT_NO
- STORAGE_DAY = DNSTOCK.STORAGE_DAY
- STORAGE_DATE = DNSTOCK.STORAGE_DATE
- NEWEST_STORAGE_DATE = DNSTOCK.NEWEST_STORAGE_DATE
- ITEM_CODE = DNSTOCK.ITEM_CODE
- SECOND_ITEM_CODE = DMITEM.SECOND_ITEM_CODE
- PALLET_ID = DNSTOCK.PALLET_ID
- BCR_DATA = DNPALLET.BCR_DATA
- AREA_TYPE = DMAREA_AREA_TYPE
- ITEM_NAME = DMITEM.ITEM_NAME
- ENTERING_QTY = DMITEM.ENTERING_QTY
- BUNDLE_ENTERING_QTY = DMITEM.BUNDLE_ENTERING_QTY
- USER_ID = Login info
- USER_NAME = Login info
- TERMINAL_NO = Login info
- TERMINAL_NAME = Login info
- IP_ADDRESS = Login info
- MANUFACTURING_DATE = DNSTOCK.MANUFACTURING_DATE
- EXPIRY_DATE = DNSTOCK.EXPIRY_DATE
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name

## DNINOUTRESULT
- RESULT_KIND = 1:Storage(Stock+)
- STATION_NO = DNCARRYINFO.SOURCE_STATION_NO
- LOCATION_NO = DNCARRYINFO.DEST_STATION_NO
- WH_STATION_NO = DNPALLET.WH_STATION_NO
- AISLE_STATION_NO = DNCARRYINFO.AISLE_STATION_NO
- WORK_TYPE = DNCARRYINFO.WORK_TYPE
- RETRIEVAL_DETAIL = DNCARRYINFO.RETRIEVAL_DETAIL
- WORK_NO = DNCARRYINFO.WORK_NO
- PALLET_ID = DNPALLET.PALLET_ID
- CARRY_KEY = DNCARRYINFO.CARRY_KEY
- RESTORING_FLAG = DNCARRYINFO.RESTORING_FLAG
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- REMOVE_FLAG = 00:Normal
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNHOSTSEND
- WORK_DAY = DNWORKINFO.WORK_DAY
- JOB_NO = DNWORKINFO.JOB_NO
- COLLECT_JOB_NO = DNWORKINFO.COLLECT_JOB_NO
- SETTING_UNIT_KEY = DNWORKINFO.SETTING_UNIT_KEY
- JOB_TYPE = DNWORKINFO.JOB_TYPE
- STATUS_FLAG = DNWORKINFO.STATUS_FLAG
- HARDWARE_TYPE = DNWORKINFO.HARDWARE_TYPE
- PLAN_UKEY = DNWORKINFO.PLAN_UKEY
- STOCK_ID = DNWORKINFO.STOCK_ID
- SYSTEM_CONN_KEY = DNWORKINFO.SYSTEM_CONN_KEY
- PLAN_DAY = DNWORKINFO.PLAN_DAY
- SUPPLIER_CODE = DNWORKINFO.SUPPLIER_CODE
- SUPPLIER_NAME = DMSUPPLIER.SUPPLIER_NAME
- RECEIVE_TICKET_NO = DNWORKINFO.RECEIVE_TICKET_NO
- RECEIVE_LINE_NO = DNWORKINFO.RECEIVE_LINE_NO
- RECEIVE_BRANCH_NO = DNWORKINFO.RECEIVE_BRANCH_NO
- CUSTOMER_CODE = DNWORKINFO.CUSTOMER_CODE
- CUSTOMER_NAME = DMCUSTOMER.CUSTOMER_NAME
- SHIP_TICKET_NO = DNWORKINFO.SHIP_TICKET_NO
- SHIP_LINE_NO = DNWORKINFO.SHIP_LINE_NO
- SHIP_BRANCH_NO = DNWORKINFO.SHIP_BRANCH_NO
- BATCH_NO = DNWORKINFO.BATCH_NO
- ORDER_NO = DNWORKINFO.ORDER_NO
- PLAN_AREA_NO = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- NOTE = DNWORKINFO.NOTE
- WORK_NOTE = DNWORKINFO.WORK_NOTE
- ITEM_CODE = DNWORKINFO.ITEM_CODE
- ITEM_NAME = DMITEM.ITEM_NAME
- UOM = DMITEM.UOM2
- JAN = DMITEM.JAN
- CASE_ITF = DMITEM.CASE_ITF
- BUNDLE_ITF = DMITEM.BUNDLE_ITF
- ENTERING_QTY = DMITEM.ENTERING_QTY
- BUNDLE_ENTERING_QTY = DMITEM.BUNDLE_ENTERING_QTY
- PLAN_LOT_NO = DNWORKINFO.PLAN_LOT_NO
- PLAN_QTY = DNWORKINFO.PLAN_QTY
- RESULT_QTY = DNWORKINFO.RESULT_QTY
- SHORTAGE_QTY = DNWORKINFO.SHORTAGE_QTY
- RESULT_AREA_NO = DNWORKINFO.RESULT_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.RESULT_LOCATION_NO
- RESULT_LOT_NO = DNWORKINFO.RESULT_LOT_NO
- USER_ID = DNWORKINFO.USER_ID
- TERMINAL_NO = DNWORKINFO.TERMINAL_NO
- WORK_SECOND = DNWORKINFO.WORK_SECOND
- USER_NAME = DCUSER.USERNAME
- REPORT_FLAG = 9:Report not required
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name


#User Story
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)