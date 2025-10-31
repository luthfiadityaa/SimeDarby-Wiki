[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletize Start database flow</span>

| Action Name                        | DNPALLETIZE | DNSTORAGEPLAN | DNWORKINFO | DNWORKLIST | DNPALLET | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | 
|-|-|-|-|-|-|-|-|-|-|
| [Palletize Start - Set(F2)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki?wikiVersion=GBwikiMaster&pagePath=/Basic%20Design/DFD%20WMS/Storage/Palletizing%20Setting&pageId=886&_a=edit&anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Epalletizing-setting-set(f2)%3C/span%3E) | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | |
| ID26 | | | | | | | | | INSERT |
| ID05 | | | | | | | | | |
| ID64 | | | | | | | | | |
| ID33 | | | | | | | | | |

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
| **STORAGE_LOCATION**           | Value from screen (Storage Location)
| **EXPIRY_DAYS**                | Value from screen (Expiry Days)
| **STORAGE_QTY**                | 1:In-Progress
| **TOTAL_ACTUAL_CARTON_QTY**    | QTY_KG_CRTN++
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
| **PLAN_AREA_NO**           | Value from screen (Storage Location)                                                       
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
| **CURRENT_STATION_NO**     | Selected Value from screen (1101/1102/1103/1104/1105)                                                       
| **WH_STATION_NO**          | 9001                                                      
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
| **STOCK_STATUS**           | UU: Unrestricted Used (Target for normal retrieval)
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
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **JOB_NO**                 | Sequence Object
|2 | **SETTING_UNIT_KEY**       | Sequence Object
|3 | **COLLECT_JOB_NO**         | Sequence Object
|4 | **JOB_TYPE**               | 02:Storage
|5 | **STATUS_FLAG**            | 0:Not Started
|6 | **RFT_STATUS_FLAG**        |
|7 | **HARDWARE_TYPE**          |
|8 | **PLAN_UKEY**              | DNSTORAGEPLAN.PLAN_UKEY
|9 | **STOCK_ID**               | Sequence Object
|10| **SYSTEM_CONN_KEY**        |
|11| **PLAN_DAY**               | DMWARENAVISYSTEM.WORK_DAY
|12| **VENDOR_CODE**            |
|13| **RECEIVE_TICKET_NO**      |
|14| **RECEIVE_LINE_NO**        |
|15| **RECEIVE_BRANCH_NO**      |
|16| **COMPANY_CODE**           |
|17| **SHIP_TICKET_NO**         |
|18| **SHIP_LINE_NO**           |
|19| **SHIP_BRANCH_NO**         |
|20| **BATCH_NO**               | Value from screen (Batch #)
|21| **ORDER_NO**               |
|22| **ORDER_SERIAL_NO**        |
|23| **PLAN_AREA_NO**           | Value from screen (Storage Location)
|24| **PLAN_LOCATION_NO**       |
|25| **MATERIAL_CODE**          | Value from screen (Material Code)
|26| **PLAN_LOT_NO**            |
|27| **NOTE**                   |
|28| **PLAN_QTY**               | Value from screen (Planned Carton Qty)
|29| **RESULT_QTY**             | 
|30| **SHORTAGE_QTY**           |
|31| **RESULT_AREA_NO**         |
|32| **RESULT_LOCATION_NO**     |
|33| **RESULT_NO**              |
|34| **WORK_NOTE**              |
|35| **SKIP_CNT**               |
|36| **WORK_DAY**               |
|37| **USER_ID**                | Login Info
|38| **TERMINAL_NO**            |
|39| **WORK_SECOND**            |
|40| **STORAGE_LOCATION_TO**    |
|41| **STORAGE_LOCATION_FROM**  | Selected Value from screen (1101/1102/1103/1104/1105)
|42| **DOCK_NO**                |
|43| **TRUCK_PLATE_NO**         |
|44| **STORING_PAIR_KEY**       |
|45| **NEW_STORING_PAIR_KEY**   |
|46| **PRIORITY_TYPE**          |
|47| **MSG_ID**                 |
|48| **MSG_TYPE**               |
|49| **MSG_ID_ORI**             |
|50| **ERROR_INDICATION**       |
|51| **TYPE**                   |
|52| **MESSAGE_DESC**           |   
|53| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|54| **REGIST_PNAME**           | ClassName
|55| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|56| **LAST_UPDATE_PNAME**      | ClassName

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
| **Column Name**                | **Description / Notes**                               |
|--------------------------------|-------------------------------------------------------|
| **CARRY_KEY**                  | Sequence Object  
| **PALLET_ID**                  | DNPALLET.PALLET_ID
| **WORK_TYPE**                  | 02:Storage
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
| **DEST_STATION_NO**            | DNWORKINFO.PLAN_AREA_NO
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
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **JOB_NO**                 | DNWORKINFO.JOB_NO
|2 | **CARRY_KEY**              |
|3 | **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
|4 | **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
|5 | **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
|6 | **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
|7 | **STOCK_ID**               | DNWORKINFO.STOCK_ID
|8 | **PALLET_ID**              | DNPALLET.PALLET_ID
|9 | **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
|10| **COMPANY_CODE**           |
|11| **COMPANY_NAME**           |
|12| **SHIP_TICKET_NO**         |
|13| **SHIP_LINE_NO**           |
|14| **SHIP_BRANCH_NO**         |
|15| **BATCH_NO**               | DNWORKINFO.BATCH_NO
|16| **ORDER_NO**               |
|17| **PLAN_AREA_NO**           |
|18| **PLAN_LOCATION_NO**       |
|19| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
|20| **MATERIAL_NAME**          | DMMASTERIALMASTER.MATERIAL_NAME
|21| **JAN**                    |
|22| **CASE_ITF**               |
|23| **BUNDLE_ITF**             |
|24| **ENTERING_QTY**           |
|25| **BUNDLE_ENTERING_QTY**    |
|26| **PLAN_LOT_NO**            |
|27| **STORAGE_TYPE**           | DNWORKINFO.STORAGE_TYPE
|28| **STORAGE_DATE**           | DNSTORAGEPLAN.PLAN_DAY
|29| **NOTE**                   |
|30| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
|31| **STOCK_QTY**              | 0
|32| **ALLOCATION_QTY**         | 0
|33| **WORK_NOTE**              |
|34| **PRIORITY**               |
|35| **RETRIEVAL_STATION_NO**   |
|36| **RETRIEVAL_DETAIL**       |
|37| **WORK_NO**                |
|38| **SOURCE_STATION_NO**      |
|39| **DEST_STATION_NO**        |
|40| **SCHEDULE_NO**            |
|41| **SCHEDULE_NO**            |
|42| **USER_ID**                | DNWORKINFO.USER_ID
|43| **USER_NAME**              | DCUSER.USER_NAME
|44| **TERMINAL_NO**            |
|45| **DOCK_NO**                |
|46| **TRUCK_PLATE_NO**         |
|47| **STORING_PAIR_KEY**       |
|48| **NEW_STORING_PAIR_KEY**   |
|49| **MSG_ID**                 |
|50| **MSG_TYPE**               |
|51| **MSG_ID_ORI**             |
|52| **ERROR_INDICATION**       |
|53| **TYPE**                   |
|54| **MESSAGE_DESC**           |   
|55| **REGIST_DATE**            |                                                    
|56| **REGIST_PNAME**           |
|57| **LAST_UPDATE_DATE**       |
|58| **LAST_UPDATE_PNAME**      |

###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **WORK_DAY**               | 
|2 | **JOB_NO**                 | DNWORKINFO.JOB_NO
|3 | **SETTING_UNIT_KEY**       | DNWORKINFO.SETTING_UNIT_KEY
|4 | **COLLECT_JOB_NO**         | DNWORKINFO.COLLECT_JOB_NO
|5 | **JOB_TYPE**               | DNWORKINFO.JOB_TYPE
|6 | **STATUS_FLAG**            | DNWORKINFO.STATUS_FLAG
|7 | **HARDWARE_TYPE**          |
|8 | **PLAN_UKEY**              | DNWORKINFO.PLAN_UKEY
|9 | **STOCK_ID**               | DNWORKINFO.STOCK_ID
|10| **SYSTEM_CONN_KEY**        | 
|11| **PLAN_DAY**               | DNWORKINFO.PLAN_DAY
|12| **VENDOR_CODE**            |
|13| **VENDOR_NAME**            |
|14| **RECEIVE_TICKET_NO**      |
|15| **RECEIVE_LINE_NO**        |
|16| **RECEIVE_BRANCH_NO**      |
|17| **COMPANY_CODE**           |
|18| **COMPANY_NAME**           |
|19| **SHIP_TICKET_NO**         |
|20| **SHIP_LINE_NO**           |
|21| **SHIP_BRANCH_NO**         |
|22| **BATCH_NO**               | DNWORKINFO.BATCH_NO
|23| **ORDER_NO**               |
|24| **PLAN_AREA_NO**           |
|25| **PLAN_LOCATION_NO**       |
|26| **MATERIAL_CODE**          | DNWORKINFO.MATERIAL_CODE
|27| **MATERIAL_NAME**          | DMMATERIALMASTER.MATERIAL_NAME
|28| **JAN**                    |
|29| **CASE_ITF**               |
|30| **BUNDLE_ITF**             |
|31| **ENTERING_QTY**           |
|32| **BUNDLE_ENTERING_QTY**    |
|33| **PLAN_LOT_NO**            |
|34| **NOTE**                   |
|35| **PLAN_QTY**               | DNWORKINFO.PLAN_QTY
|36| **RESULT_QTY**             |
|37| **SHORTAGE_QTY**           |
|38| **RESULT_AREA_NO**         |
|39| **RESULT_LOCATION_NO**     |
|40| **RESULT_LOT_NO**          |
|41| **WORK_NOTE**              |
|42| **REPORT_FLAG**            | DNSTORAGEPLAN.REPORT_FLAG
|43| **USER_ID**                | DNWORKINFO.USER_ID
|44| **USER_NAME**              | DCUSER.USER_NAME
|45| **TERMINAL_NO**            |
|46| **WORK_SECOND**            |
|47| **MSG_ID**                 |
|48| **MSG_TYPE**               |
|49| **MSG_ID_ORI**             |
|50| **ERROR_INDICATION**       |
|51| **TYPE**                   |
|52| **MESSAGE_DESC**           |
|53| **REGIST_DATE**            | SYSTIMESTAMP                                                    
|54| **REGIST_PNAME**           | ClassName
|55| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|56| **LAST_UPDATE_PNAME**      | ClassName

#<span style="color:skyblue; font-weight:bold">Release Command from Palletize Robot</span
  After palletizing is completed, the palletizing robot sends <span style="color:green; font-weight:bold">“Release Command”</span> signal via conveyor interlock.

::: mermaid
flowchart LR

releaseCommand["
Release Command from Palletize Robot
"]

buttonClicked["
Station Completion 
button clicked
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

inoutstationoperator[InOutStationOperator]

releaseCommand-->buttonClicked-->id26msg-->id26process-->inoutstationoperator
inoutstationoperator--I-->id26-insert
:::

After user clicked on Completion button at Station, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">DNArrival</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **ARRIVAL_DATE**           | SYSTIMESTAMP 
|2 | **STATION_NO**             | Arrival Station Number from ID26 
|3 | **CARRY_KEY**              | 99999999       
|4 | **BCR_DATA**               | Barcode information from ID26
|5 | **CONTROLINFO**            | Control information from ID26
|6 | **SEND_FLAG**              | 0:Not sent
|7 | **HEIGHT**                 | Dimension Information from ID26
|8 | **WIDTH**                  | Dimension Information From ID26
|9 | **REGIST_DATE**            | SYSTIMESTAMP                                                    
|10| **REGIST_PNAME**           | ClassName
|11| **LAST_UPDATE_DATE**       | SYSTIMESTAMP
|12| **LAST_UPDATE_PNAME**      | ClassName

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


###<span style="color:skyblue; font-weight:bold">DMWarehouse</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | **LAST_USED_STATION_NO**   | Aisle Number where a reserved location belongs to 

###<span style="color:skyblue; font-weight:bold">DMShelf</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | STATUS_FLAG                | 2:Reserved Location
|2 | LAST_UPDATE_DATE           | SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | DEST_STATION_NO            | Reserved Location Number
|2 | AISLE_STATION_NO           | Aisle Number where a reserved location belongs to
|3 | CMD_STATUS                 | 2:Waiting for response
|4 | LAST_UPDATE_DATE           | SYSTIMESTAMP
|5 | LAST_UPDATE_PNAME          | Class name

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
|NO| **Column Name**            | **Description / Notes**                               |
|--|----------------------------|-------------------------------------------------------|
|1 | PLAN_LOCATION_NO           | Reserved Location Number
|2 | LAST_UPDATE_DATE           | SYSTIMESTAMP

## DNPALLET
- CURRENT_STATION_NO = Reserved Location Number
- WH_STATION_NO = 9000
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNSTOCK
- AREA_NO = 9000
- LOCATION_NO = Reserved Location Number
- LAST_UPDATE_DATE = SYSTIMESTAMP
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