[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletize Start database flow</span>
| Action Name                        | DNPALLETIZE | DNSTORAGEPLAN | DNWORKINFO | DNWORKLIST | DNPALLET | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | 
|-|-|-|-|-|-|-|-|-|-|
| Palletize Start - Set(F2)[(1)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Setting&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Epalletizing-setting-set(f2)%3C/span%3E) | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | |
| ID26(2) | | | | | | | | | INSERT |
| Automatic Mode Change Sender(3) | | | | | | | | | |
| ID25(4) | | | | | | | | | |
| ID26(5) | | | | | | | | | INSERT |
| Automatic Mode Change Sender(6) | | | | | | | | | |
| ID25(7) | | | | | | | | | |

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

| **Field Name**                | **Insert Value**                               |
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
| **STORAGE_LOCATION**           | Value from screen (Storage Location) ⟶ <span style="color:yellow; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>
| **EXPIRY_DAYS**                | Value from screen (Expiry Days)
| **STORAGE_QTY**                | <span style="color:yellow; font-weight:bold">PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY</span>
| **TOTAL_ACTUAL_CARTON_QTY**    | <span style="color:yellow; font-weight:bold">TOTAL_ACTUAL_CARTON_QTY + QTY_KG_CRTN</span>
| **MAX_PALLET_BATCH_END**       | <span style="color:yellow; font-weight:bold">PLANNED_CARTON_QTY / QTY_CRTN_PL</span> 
| **BATCH_PALLET_START**         | <span style="color:yellow; font-weight:bold">Value from screen (Storage Date/Time) + TimeStamp (System Generated)</span>
| **BATCH_PALLET_END**           | 
| **REGIST_DATE**                | SYSTIMESTAMP                                                    
| **REGIST_PNAME**               | ClassName
| **LAST_UPDATE_DATE**           | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**          | ClassName

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              | Sequence Object                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            | 1:Working                                                       
| **CANCEL_FLAG**            | 0:Normal Data                                                      
| **PLAN_DAY**               | DNPALLETIZE.BATCH_PALLET_START                                                      
| **VENDOR_CODE**            |                                                       
| **COMPANY_CODE**           |                                                       
| **RECEIVE_TICKET_NO**      |                                                       
| **RECEIVE_LINE_NO**        |                                                       
| **RECEIVE_TICKET_DATE**    |                                                       
| **BRANCH_NO**              |                                                         
| **PLAN_AREA_NO**           | DNPALLETIZE.STORAGE_LOCATION                                                       
| **PLAN_LOCATION_NO**       |                                                       
| **MATERIAL_CODE**          | DNPALLETIZE.MATERIAL_CODE                                                      
| **PLAN_LOT_NO**            |                                                       
| **NOTE**                   |                                                       
| **PLAN_QTY**               | DNPALLETIZE.PLANNED_CARTON_QTY                                                       
| **PROCESS_QTY**            | DNPALLETIZE.QTY_CRTN_PL                                         
| **RESULT_QTY**             |                                                       
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            | 0:Not Reported                                                      
| **WORK_DAY**               |                                                       
| **REGIST_KIND**            |                                                       
| **BCR_DATA**               |                                                        
| **STORING_PAIR_KEY**       | <span style="color:yellow; font-weight:bold">DNPALLETIZE.MATERIAL_CODE + DNPALLETIZE.BATCH_NO</span>  
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
| **Field Name**            | **Insert Vlaue**                               |
|----------------------------|-------------------------------------------------------|
| **PALLET_ID**              | Sequence Object                                                       
| **CURRENT_STATION_NO**     | DNPALLETIZE.STATION_NO                                                       
| **WH_STATION_NO**          | DNPALLETIZE.STORAGE_LOCATION                                                      
| **STATUS_FLAG**            | 1:Reserved for Storage                                                      
| **ALLOCATION_FLAG**        |                                                       
| **EMPTY_FLAG**             | 0:Normal Pallet                                                       
| **HEIGHT**                 |                                                       
| **WIDTH**                  |                                                       
| **SOFTZONE_ID**            |                                                       
| **BCR_DATA**               | System generated (YYXXXXXXXX, XXXXXXXX represents sequence number)                                                      
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
| **field Name**            | **Insert Value**                               |
|----------------------------|-------------------------------------------------------|
| **STOCK_ID**               | Sequence Object   
| **AREA_NO**                | DNPALLETIZE.STORAGE_LOCATION
| **LOCATION_NO**            |
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
| **PLAN_QTY**               | DNPALLETIZE.QTY_CRTN_PL 
| **PALLET_ID**              | Sequence Object
| **BATCH_NO**               | DNPALLETIZE.BATCH_NO 
| **STOCK_STATUS**           | <span style="color:yellow; font-weight:bold">UU: Unrestricted Used</span>
| **QC_DURATION**            | <span style="color:yellow; font-weight:bold">0: Not Reached</span>
| **TEMPERING_FLAG**         | <span style="color:yellow; font-weight:bold">0: Not Reached</span>
| **QC_FLAG**                | <span style="color:yellow; font-weight:bold">Not Done</span>
| **TEMPERING_PERIOD**       | DNPALLETIZE.TEMPERING_PERIOD
| **STORING_PAIR_KEY**       | <span style="color:yellow; font-weight:bold">DNPALLETIZE.MATERIAL_CODE + DNPALLETIZE.BATCH_NO </span>
| **EXPIRY_DATE**            | DNPALLETIZE.EXPIRY_DAYS
| **REGIST_DATE**            | SYSTIMESTAMP                                                    
| **REGIST_PNAME**           | ClassName
| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Filed Name**            | **Insert Value**                               |
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
| **BATCH_NO**               | DNPALLETIZE.BATCH_NO
| **ORDER_NO**               |
| **ORDER_SERIAL_NO**        |
| **PLAN_AREA_NO**           | DNPALLETIZE.STORAGE_LOCATION 
| **PLAN_LOCATION_NO**       |
| **MATERIAL_CODE**          | DNPALLETIZE.MATERIAL_CODE 
| **PLAN_LOT_NO**            |
| **NOTE**                   |
| **PLAN_QTY**               | DNPALLETIZE.QTY_CRTN_PL 
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
| **STORAGE_LOCATION_FROM**  | DNPALLETIZE.STATION_NO 
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
")]

inoutstationoperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->inoutstationoperator
inoutstationoperator--I-->id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
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

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DMWarehouse</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **LAST_USED_STATION_NO**   |  Aisle Number where a reserved location belongs to 

####<span style="color:skyblue; font-weight:bold">DMShelf</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| STATUS_FLAG                | 2:Reserved Location
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| AISLE_STATION_NO           | Aisle Number where a reserved location belongs to
| CMD_STATUS                 | 2:Waiting for response
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| PLAN_LOCATION_NO           | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CURRENT_STATION_NO         | Reserved Location Number
| WH_STATION_NO              | DNCARRYINFO.END_STATION_NO
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| AREA_NO                    | DNCARRYINFO.END_STATION_NO
| LOCATION_NO                | Reserved Location Number
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| SEND_FLAG                  | 1:Sent
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

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
id25process-.U.->id25-update
id25process-.D.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
#### DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

releaseCommand["
Continue the Process
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

Continue the process, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **ARRIVAL_DATE**           | SYSTIMESTAMP 
| **STATION_NO**             | Arrival Station Number from ID26 

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

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Mode Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| LAST_UPDATE_DATE           | SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| CARRY_KEY                  | DNCARRYINFO.CARRY_KEY
| SEND_FLAG                  | 1:Sent
| LAST_UPDATE_DATE           | SYSTIMESTAMP
| LAST_UPDATE_PNAME          | Class name

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
id25process-.U.->id25-update
id25process-.D.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
#### DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##Continue Process with
###[Flow 1 - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/938/Flow-1)
###[Flow 2 - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/940/Flow-2)

#User Story
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)