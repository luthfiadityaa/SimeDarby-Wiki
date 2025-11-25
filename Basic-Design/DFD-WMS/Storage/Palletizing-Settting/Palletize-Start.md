[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR

P1[Palletizing Start]--> |1101-1105| P2[ID26]-->P3[StorageSender]-->P4[ID25]
P4[ID25]--> |1111-1115| P5[ID26]-->P6[StorageSender]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR

Cond2{Flow?}

Cond2 --> |Flow 1: Without going 
through crane 7~10 - STV| P8[ID64]
Cond2 --> |Flow 2: Go through 
crane 7~10 - STV| P9[ID64]

P8[ID64]--> |7101-7110| P10[ID26]-->P11[StorageSender]-->P12[ID25]--> |SRM| P18[ID64]
P9[ID64]--> |7107-7110| P15[ID26]-->P16[StorageSender]-->P17[ID25]--> |SRM|P18[ID64]
:::

**Stage 3**
::: mermaid
flowchart LR

P18[ID64]--> |STV| P19[ID64]--> |7211-7212| P20[ID26]-->P21[StorageSender]-->P22[ID25]--> |SRM| P23[ID64]

P18[ID64]--> |9001-9010| P14[ID33]
P23[ID64]--> |9011-9014| P14[ID33]
:::

#<span style="color:skyblue; font-weight:bold">Palletize Start database flow</span>
**Abbreviation:**
* **Table Name**
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
  - **ITEM** : DMITEM
  - **STSN** : DMSTATION  
  - **STCH** : DNSTOCKHISTORY 
 
* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                 |STRP|WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN| 
|-------------------------------------------------------------|----|----|----|----|----|----|----|-----|----|----|----|----|----|
| Palletize Start - SetF2 [(1)](#Palletize-Start---Set(F2))   | I  |    |    |    |    |    |    |     |    |    |    | S  | S  |    
| ID26 [(2)](#ID26-at-1101-1105)                              |    | I  |    | I  | I  | I  |    | I   |    |    |    |    |    |    
| Storage Sender [(3)](#Storage-Sender-at-1101-1105)          |    |    |    |    | U  |    |    | U   |    |    |    |    |    |    
| ID25 [(4)](#ID25-at-1101-1105)                              |    |    |    |    | U  |    |    | D   |    |    |    |    |    |    
| ID26 [(5)](#ID26-at-1111-1115)                              |    |    |    | U  | U  |    |    | I   |    |    |    |    |    |    
| Storage Sender [(6)](#Storage-Sender-at-1111-1115)          |    | U  |    |    | U  | U  |    | U   | U  | U  |    |    |    | 
| ID25 [(7)](#ID25-at-1111-1115)                              |    |    |    |    | U  |    |    | D   |    |    |    |    |    |    
| ID64 [(8)](#ID64-at-STV-from-1111-1115)                     |    |    |    |    | U  |    |    |     |    |    |    |    |    |
| **Flow 1**                                                  |    |    |    |    |    |    |    |     |    |    |    |    |    |
| ID26 [(9)](#ID26-at-7101-7110)                              |    |    |    | U  | U  |    |    | I   |    |    |    |    |    |  
| Storage Sender [(10)](#Storage-Sender-at-7101-7110)         |    |    |    |    | U  |    |    | U   |    |    |    |    |    |
| ID25 [(11)](#ID25-at-7101-7110)                             |    |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64 [(12)](#ID64-at-SRM-to-9001-or-9002)                   |    |    |    |    | U  |    |    |     |    |    |    |    |    |
| **Flow 2**                                                  |    |    |    |    |    |    |    |     |    |    |    |    |    |    
| ID26 [(9)](#ID26-at-7107-7110)                              |    |    |    | U  | U  |    |    | I   |    |    |    |    |    |
| Storage Sender [(10)](#Storage-Sender-at-7107-7110)         |    |    |    |    | U  |    |    | U   |    |    |    |    |    | 
| ID25 [(11)](#ID25-at-7107-7110)                             |    |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64 [(12)](#ID64-at-SRM-from-7107-7110)                    |    |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID64 [(13)](#ID64-at-STV-from-9007-9010)                    |    |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID26 [(14)](#ID26-at-7211-7214)                             |    |    |    | U  | U  |    |    | I   |    |    |    |    |    |       
| Storage Sender [(15)](#Storage-Sender-at-7211-7214)         |    |    |    |    | U  |    |    | U   |    |    |    |    |    |
| ID25 [(16)](#ID25-at-7211-7214)                             |    |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64 [(17)](#ID64-at-SRM-to-9002)                           |    |    |    |    | U  |    |    |     |    |    |    |    |    |
| **Last Process**                                            |    |    |    |    |    |    |    |     |    |    |    |    |    |
| ID33 [(18)](#ID33)                                          | U  | U  | U  | U  | D  | U  | I  |     |    |  U | I  |    |    |  
| **Host Communication**                                      |    |    |    |    |    |    |    |     |    |    |    |    |    |
| Production Storage Result [(19)](#Production-Storage-Result)|    |    |    |    |    |    | U   |     |    |    |    |    |    | 

#Palletize Start - Set(F2)
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
        DNSTORAGEPLAN
    ")]

     tableList-select[("
        DMITEM
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
- Material Code exists in <span style="color:green; font-weight:bold">DMITEM</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
*   **PLAN_UKEY** : Sequence Object    
*   **STATUS_FLAG** : 1: Working    
*   **CANCEL_FLAG** : 0: Normal Data    
*   **DELETE_FLAG** : 0: No    
*   **PLAN_DAY** : Value from screen (**Storage Date/Time**)    
*   **PLAN_AREA_NO** : Value from screen (**Storage Location**) : **FGW1 (Ambient/9002)** or **FGW2 (Tempering/9001)**    
*   **MATERIAL_CODE** : Value from screen (**Material Code**) -> **<span style="color:green; font-weight:bold">DMITEM.ITEM_CODE </span>**   
*   **PLAN_QTY** : Value from screen (**Qty Crtn/PL**)    
*   **REPORT_FLAG** : 0: Not Reported    
*   **STORING_PAIR_KEY** : Value from screen **(Material Code + Batch_No)**    
*   **STATION_NO** : Value from screen (**Station No**)    
*   **CURRENT_STATUS** : Value from screen (**Current Status**)    
*   **MATERIAL_NAME** : Value from screen (**Material Name**) -> **<span style="color:green; font-weight:bold">DMITEM.ITEM_NAME </span>**    
*   **MATERIAL_TYPE** : Value from screen (**Material Type**) -> **<span style="color:green; font-weight:bold">DMITEM.ITEM_TYPE </span>**    
*   **BATCH_NO** : Value from screen (**Batch_No**)    
*   **PLAN_QTY** : Value from screen (**Planned Carton Qty**)    
*   **QTY_KG_CRTN** : Value from screen (**Qty Kg/Crtn**) -> **<span style="color:green; font-weight:bold">DMITEM.ENTERING_QTY </span>**    
*   **UOM** : Value from screen (**UOM**) -> **<span style="color:green; font-weight:bold">DMITEM.UOM </span>**    
*   **TEMPERING_PERIOD** : Value from screen (**Tempering Period**)    
*   **EXPIRY_DAYS** : Value from screen (**Expiry Days**)    
*   **STORAGE_QTY** : **PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY**    
*   **TOTAL_ACTUAL_CARTON_QTY** : **TOTAL_ACTUAL_CARTON_QTY + QTY_KG_CRTN**    
*   **MAX_PALLET_BATCH_END** : **PLANNED_CARTON_QTY / QTY_CRTN_PL**    
*   **BATCH_PALLET_START** : **Value from screen (Storage Date/Time) + System Timestamp**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

#Release Command from Palletize Robot - Dummy Arrival
After palletizing is completed, the palletizing robot sends <span style="color:green; font-weight:bold">“Release Command”</span> signal via conveyor interlock.

<hr style="border: 2px solid red;">

#Irregular Cases
Continue this flow if occurs.
- [Wait for available location if full](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=wait-for-available-location-if-full)
- [No-Read (Go to Reject station)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=no-read-(go-to-reject-station))
- [Wait for Batch Start. Treat as Data Error](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=wait-for-batch-start.-treat-as-data-error)
<hr style="border: 2px solid red;">

##ID26 at 1101-1105

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
*   **ARRIVAL_DATE** : SYSTIMESTAMP    
*   **STATION_NO** : Arrival Station Number from **ID26**    
*   **CARRY_KEY** : 99999999    
*   **BCR_DATA** : Barcode information from **ID26**    
*   **CONTROLINFO** : Control information from **ID26**    
*   **SEND_FLAG** : 0: Not sent    
*   **HEIGHT** : Dimension information from **ID26**    
*   **WIDTH** : Dimension information from **ID26**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **PALLET_ID** : Sequence Object    
*   **CURRENT_STATION_NO** : DNSTORAGEPLAN.STATION_NO    
*   **WH_STATION_NO** : DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STATUS_FLAG** : 1: Reserved for Storage    
*   **EMPTY_FLAG** : 0: Normal Pallet    
*   **ALLOCATION_FLAG** : 1: Allocated    
*   **SOFT_ZONE_ID** : DMITEM.SOFT_ZONE_ID    
*   **BCR_DATA** : DNARRIVAL.BCR_DATA    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO** : Sequence Object    
*   **SETTING_UNIT_KEY** : Sequence Object    
*   **COLLECT_JOB_NO** : Sequence Object    
*   **JOB_TYPE** : 02: Storage    
*   **STATUS_FLAG** : 0: Not Started    
*   **PLAN_UKEY** : DNSTORAGEPLAN.PLAN_UKEY    
*   **STOCK_ID** : Sequence Object    
*   **PLAN_DAY** : DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO** : DNSTORAGEPLAN.BATCH_NO    
*   **PLAN_AREA_NO** : DNSTORAGEPLAN.PLAN_AREA_NO_NO    
*   **MATERIAL_CODE** : DNSTORAGEPLAN.MATERIAL_CODE    
*   **PLAN_QTY** : DNSTORAGEPLAN.PLAN_QTY    
*   **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY    
*   **USER_ID** : Login Info    
*   **STORAGE_LOCATION_FROM** : DNSTORAGEPLAN.STATION_NO    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY** : Sequence Object    
*   **PALLET_ID** : DNPALLET.PALLET_ID    
*   **WORK_TYPE** : 26: Direct Transfer    
*   **CMD_STATUS** : 1: Started    
*   **PRIORITY** : 2: Normal    
*   **RESTORING_FLAG** : 0: Not Restore to Original Location    
*   **CARRY_FLAG** : 3: Direct Transfer    
*   **WORK_NO** : Sequence Object    
*   **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO : **(1101 / 1102 / 1103 / 1104 / 1105)**    
*   **DEST_STATION_NO** : **Based on SOURCE_STATION_NO where a reserved location belongs to : (1111 / 1112 / 1113 / 1114 / 1115)**    
*   **CANCEL_REQUEST** : 0: Not Requested    
*   **SCHEDULE_NO** : Sequence Object    
*   **END_STATION_NO** : DNWORKINFO.PLAN_AREA_NO    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **STOCK_ID** : Sequence Object    
*   **AREA_NO** : DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STORAGE_TYPE** : 2: New    
*   **STOCK_QTY** : 0    
*   **ALLOCATION_QTY** : 0    
*   **PLAN_QTY** : DNSTORAGEPLAN.PLAN_QTY    
*   **PALLET_ID** : Sequence Object    
*   **BATCH_NO** : DNSTORAGEPLAN.BATCH_NO    
*   **TEMPERING_PERIOD** : DNSTORAGEPLAN.TEMPERING_PERIOD    
*   **STORING_PAIR_KEY** : DNSTORAGEPLAN.STORING_PAIR_KEY    
*   **EXPIRY_DATE** : DNSTORAGEPLAN.EXPIRY_DAYS    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##Storage Sender at 1101-1105
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
*   **CMD_STATUS** : 2: Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1: Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##ID25 at 1101-1105
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
*   **CMD_STATUS** : 3: Commanded   
*   **ERROR_CODE** : 0    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##ID26 at 1111-1115

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
*   **ARRIVAL_DATE** : SYSTIMESTAMP    
*   **STATION_NO** : Arrival Station Number from **ID26**    
*   **CARRY_KEY** : 99999999    
*   **BCR_DATA** : Barcode information from **ID26**    
*   **CONTROLINFO** : Control information from **ID26**    
*   **SEND_FLAG** : 0: Not sent    
*   **HEIGHT** : Dimension information from **ID26**    
*   **WIDTH** : Dimension information from **ID26**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** : DNARRIVAL.STATION_NO    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName


####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** : 26: Direct Transfer    
*   **CMD_STATUS** : 1: Started    
*   **CARRY_FLAG** : 3: Direct Transfer    
*   **SOURCE_STATION_NO** : DNARRIVAL.STATION_NO : **(1111, 1112, 1113, 1114, 1115)**    
*   **DEST_STATION_NO** : **Based on SOURCE_STATION_NO where a reserved location belongs to : (7101, 7102, 7103, 7104, 7105, 7106, 7107, 7108, 7109, 7110)**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##Storage Sender at 1111-1115
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
*   **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DMShelf</span>
*   **STATUS_FLAG** : 2:Reserved Location    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **CMD_STATUS** : 2:Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **LOCATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **PLAN_LOCATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1:Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

###ID25 at 1111-1115

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
*   **CMD_STATUS** : 3:Commanded    
*   **ERROR_CODE** : 0    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID64 at STV from 1111-1115
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process&nbsp;</span>

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
*   **CMD_STATUS** : 4:Pickup completed    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ###[Flow 1 : not through crane 7-10](#Flow-1-:-not-through-crane-7-10)
- ###[Flow 2 : through crane 7 10](#Flow-2--:-through-crane-7-10)

<hr>

#Flow 1 : not through crane 7-10 
Refer to AGC Linkage Specs : [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##ID26 at 7101-7110

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
*   **ARRIVAL_DATE** : SYSTIMESTAMP    
*   **STATION_NO** : Arrival Station Number from ID26    
*   **CARRY_KEY** : 99999999    
*   **BCR_DATA** : Barcode information from ID26    
*   **CONTROLINFO** : Control information from ID26    
*   **SEND_FLAG** : 0:Not sent    
*   **HEIGHT** : Dimension Information from ID26    
*   **WIDTH** : Dimension Information from ID26    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** : Reserved Location Number    
*   **WH_STATION_NO** : Reserved Area Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** : 2:Storage    
*   **CMD_STATUS** : 1:Started    
*   **CARRY_FLAG** : 1:Storage    
*   **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **SOURCE_STATION_NO** : DNARRIVAL.STATION_NO    
*   **DEST_STATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##Storage Sender at 7101-7110
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
*   **CMD_STATUS** : 2: Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1: Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID25 at 7101-7110
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
*   **CMD_STATUS** : 3: Commanded    
*   **ERROR_CODE** : 0    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID64 at SRM to 9001 or 9002
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process</span>

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
*   **CMD_STATUS** : 4: Pickup completed    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

<hr>

#Flow 2 : through crane 7-10 
Refer to AGC Linkage Specs : [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##ID26 at 7107-7110

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
*   **ARRIVAL_DATE** : SYSTIMESTAMP    
*   **STATION_NO** : Arrival Station Number from ID26    
*   **CARRY_KEY** : 99999999    
*   **BCR_DATA** : Barcode information from ID26    
*   **CONTROLINFO** : Control information from ID26    
*   **SEND_FLAG** : 0: Not sent    
*   **HEIGHT** : Dimension Information from ID26    
*   **WIDTH** : Dimension Information from ID26    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** : DNARRIVAL.STATION_NO    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** : 26:Direct Transfer    
*   **CMD_STATUS** : 1:Started    
*   **CARRY_FLAG** : 3:Direct Transfer    
*   **SOURCE_STATION_NO** : DNARRIVAL.STATION_NO ⟶ **(7107, 7108, 7109, 7110)**    
*   **DEST_STATION_NO** : **Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7211, 7212, 7213, 7214)**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##Storage Sender at 7107-7110
<span style="background-color:yellow; color: black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.StorageSender &nbsp;</span>

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
*   **CMD_STATUS** : 2:Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1:Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID25 at 7107-7110
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
*   **CMD_STATUS** : 3:Commanded    
*   **ERROR_CODE** : 0    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID64 at SRM from 7107-7110
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process&nbsp;</span>

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
*   **CMD_STATUS** : 4: Pickup completed    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID64 at STV from 9007-9010
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process&nbsp;</span>

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
*   **CMD_STATUS** : 4: Pickup completed    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID26 at 7211-7214

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
*   **ARRIVAL_DATE** : SYSTIMESTAMP    
*   **STATION_NO** : Arrival Station Number from ID26    
*   **CARRY_KEY** : 99999999    
*   **BCR_DATA** : Barcode information from ID26    
*   **CONTROLINFO** : Control information from ID26    
*   **SEND_FLAG** : 0: Not sent    
*   **HEIGHT** : Dimension Information from ID26    
*   **WIDTH** : Dimension Information from ID26    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** : Reserved Location Number    
*   **WH_STATION_NO** : Reserved Area Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** : 2:Storage    
*   **CMD_STATUS** : 1:Started    
*   **CARRY_FLAG** : 1:Storage    
*   **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **SOURCE_STATION_NO** : DNARRIVAL.STATION_NO    
*   **DEST_STATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##Storage Sender at 7211-7214
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
*   **CMD_STATUS** : 2: Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1:Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID25 at 7211-7214
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
*   **CMD_STATUS** : 3: Commanded    
*   **ERROR_CODE** : 0    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##ID64 at SRM to 9002
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process&nbsp;</span>

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
*   **CMD_STATUS** : 4: Pickup completed    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-0f17bef0-54b4-4a5a-9227-3908b6fe5b7f.png)

Table Impact:
- **DNStock**
- **DNStockHistory**
- **DNSTORAGEPLAN**

<hr>

##<span style="color:skyblue; font-weight:bold">[Click this to Continue flow, If comming from Batch End Process](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid33%3C/span%3E)</span><br>
<hr>

##ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process&nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY

DNWORKLIST
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
*   **STATUS_FLAG** : 1: Occupied    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** : Location Number information from ID33    
*   **STATUS_FLAG** : 2: Occupied    
*   **ALLOCATION_FLAG** : 0: Not allocated    
*   **LAST_STORED_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
* **RESULT_QTY**: DNWORKINFO.PLAN_QTY
* **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
* **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **STATUS_FLAG**: 4: Completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)

* **STORAGE_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **NEWEST_STORAGE_DATE**: SYSTIMESTAMP
* **STOCK_QTY**: DNWORKINFO.RESULT_QTY
* **ALLOCATION_QTY**: DNWORKINFO.RESULT_QTY
* **PLAN_QTY**: 0
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
* **TEMPERING_FLAG**: <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span>
* **QC_FLAG**: <span style="color:green; font-weight:bold">0: Not Done</span>
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold"> DNStoragePlan </span>
Storage Date will be update by the system in this section represent by <span style="color:green; font-weight:bold">LAST_UPDATE_DATE</span>
<span style="color:red; font-weight:bold; font-size:12px">*If DNSTORAGEPLAN.RESULT_QTY = DNSTORAGEPLAN.PLAN_QTY then update Status to completed</span>

**STATUS_FLAG**: 4: Completed
* **RESULT_QTY**: DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
* **SHORTAGE_QTY**: DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
* **TEMPERING_FLAG**: <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span>
* **QC_FLAG**: <span style="color:green; font-weight:bold">0: Not Done</span>
* **LAST_UPDATE_DATE**: <span style="color:green; font-weight:bold">DNSTORAGEPLAN.PLAN_DAY + Time value (HH:MM:SS.sss) : (System Generated)</span>
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNStockHistory </span> 
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)

* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **INC_DEC_TYPE**: 1: Stock Increase
* **JOB_TYPE**: 2: Storage
* **UPDATE_STOCK_QTY**: DNSTOCK.STOCK_QTY
* **INC_DEC_QTY**: DNSTOCK.STOCK_QTY
* **STOCK_ID**: DNSTOCK.STOCK_ID
* **AREA_NO**: DNSTOCK.AREA_NO
* **LOCATION_NO**: DNSTOCK.LOCATION_NO
* **STORAGE_DAY**: DNSTOCK.STORAGE_DAY
* **STORAGE_DATE**: DNSTOCK.STORAGE_DATE
* **NEWEST_STORAGE_DATE**: DNSTOCK.NEWEST_STORAGE_DATE
* **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE
* **PALLET_ID**: DNSTOCK.PALLET_ID
* **BCR_DATA**: DNPALLET.BCR_DATA
* **AREA_TYPE**: DMAREA_AREA_TYPE
* **MATERIAL_NAME**: DMMITEM.MATERIAL_NAME
* **ENTERING_QTY**: DNSTORAGEPLAN.QTY_CRTN_PL
* **USER_ID**: Login info
* **USER_NAME**: Login info
* **TERMINAL_NO**: Login info
* **TERMINAL_NAME**: Login info
* **IP_ADDRESS**: Login info
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>
* **TEMPERING_FLAG**: <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span>
* **QC_FLAG**: <span style="color:green; font-weight:bold">0: Not Done</span>
* **EXPIRY_DATE**: DNWORKINFO.EXPIRY_DAYS
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **JOB_NO**: DNWORKINFO.JOB_NO
* **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO
* **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY
* **JOB_TYPE**: DNWORKINFO.JOB_TYPE
* **STATUS_FLAG**: DNWORKINFO.STATUS_FLAG
* **HARDWARE_TYPE**: DNWORKINFO.HARDWARE_TYPE
* **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY
* **STOCK_ID**: DNWORKINFO.STOCK_ID
* **SYSTEM_CONN_KEY**: DNWORKINFO.SYSTEM_CONN_KEY
* **PLAN_DAY**: DNWORKINFO.PLAN_DAY
* **BATCH_NO**: DNWORKINFO.BATCH_NO
* **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
* **PLAN_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
* **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE
* **MATERIAL_NAME**: DMITEM.MATERIAL_NAME
* **UOM**: DMITEM.UOM
* **ENTERING_QTY**: DNSTORAGEPLAN.QTY_CRTN_PL
* **BUNDLE_ENTERING_QTY**: DNSTORAGEPLAN.PLANNED_CARTON_QTY
* **PLAN_QTY**: DNWORKINFO.PLAN_QTY
* **RESULT_QTY**: DNWORKINFO.RESULT_QTY
* **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY
* **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO
* **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO
* **RESULT_LOT_NO**: DNWORKINFO.RESULT_LOT_NO
* **USER_ID**: DNWORKINFO.USER_ID
* **TERMINAL_NO**: DNWORKINFO.TERMINAL_NO
* **WORK_SECOND**: DNWORKINFO.WORK_SECOND
* **USER_NAME**: DCUSER.USERNAME
* **REPORT_FLAG**: 0: Not Reported
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

##Production Storage Result
- After the process complete WareNavi will send result to SAP as [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)

<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD Storage from Palletizing Area](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5783)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Batch End - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End)
- [Production Storage Result - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)