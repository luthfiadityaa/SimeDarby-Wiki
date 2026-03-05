[[_TOC_]]

# Conditions
## Initial condition
### The first day of operation
- No Empty palelt at 1101-1105 and 1111-1115.
- Batch is not execued.
- Cartons are not on production lane conveyor.

### Begining of one day after operation day.
- Empty palelt exists at 1101-1105 and may be also in front of Robot. Because AGC automatically supplies.
- Batch is not execued.
- Cartons are not on production lane conveyor.

## Condition during operation (Regardless of Batch Start/End)
### At 1101-1105
- Empty palelt exists at 1101-1105 and may be also in front of Robot. Because AGC automatically supplies.

#### In case a batch is not executed
- DNPallet and DNSotck exist but DNStoragePlan and DNCarryInfo don't exist.

#### In case a batch is executed
- DNPallet, DNSotck, DNStoragePlan and DNCarryInfo exist.

### At 1111-1115
#### In case a batch is not executed
- No palelt exist.

#### In case a batch is executed
- Palelt with palletized carton exist.
- DNPallet, DNSotck, DNStoragePlan and DNCarryInfo exist.


# Scenarios

## Normal
### Batch start setting
- Operator starts Batch on WareNavi at first.
- Operator starts Robot with corresponding setting.
- Cartons are supplied to production lane.

### Palletizing Work
#### If Empty pallet is already at 1101-1105. 
1. DNPallet and DNStock are already created when received ID26
2. WareNavi sends ID05.
   This must be triggered by Batch Start screen. Create DNCarryinfo and kick StorageSender
#### If Empty pallet just arrives at 1101-1105. 
1. WareNavi creates DNPallet and DNStock.
2. WareNavi sends ID05.
   This can be triggered from ID26 receiving process.
#### Common process
3. Empty pallet moves to front of Robot area.
4. Robot starts palletizing work.
5. Robot completes the work and release the palelt to 1111-1115.
6. Warenavi receives ID26 with Normal Completion.
7. Warenavi creates DNWorkInfo and connects it to active DNStoragePlan and DNCarryInfo.
8. WareNavi sends ID05 for going to Aisle station.

Continue to [If Empty pallet just arrives at 1101-1105].

### Palletizing Work for last pallet
1. Empty pallet arrives at 1101-1105. 
2. WareNavi creates DNPallet and DNStock.
3. WareNavi sends ID05.
   This can be triggered from ID26 receiving process.
4. Empty pallet moves to front of Robot area.
5. Robot starts palletizing work.
#### If the number of cartons is totlly planned number.
6. Robot completes the work and releases the pallet to 1111-1115.
7. Warenavi receives ID26 with Normal Completion.
8. Warenavi creates DNWorkInfo and connects it to active DNStoragePlan and DNCarryInfo.
9. WareNavi sends ID05 for going to Aisle station.
---
10. At the same time, the next pallet arrives at 1101-1105.
11. **This pallet moves to front of Robot because Batch is not Ended on WareNavi screen.**
---
12. Operator stops batch on Robot.
13. Operator executes Batch End on WareNavi screen.
14. The pallet in front of robot remains there.

#### If the number of cartons is not planned number.
6. Robot stops work because there are not cartons from production lane.
7. Operator executes Batch End on WareNavi screen. (Status --> Waiting for last pallet) Cf. Step11
8. Operator forcibly completes the pallet.
9. Robot releases the pallet to 1111-1115.
10. Warenavi receives ID26 with Force Completion.
11. Warenavi creates DNWorkInfo and connects it to DNStoragePlan(Waiting for last pallet) and DNCarryInfo.
12. WareNavi sends ID05 for going to Aisle station.
---
13. At the same time, the next pallet arrives at 1101-1105.
14. **This pallet won't move to front of Robot because Batch is Ended on WareNavi screen.**
---


#Palletize Start - Set(F2)
![image.png](/.attachments/image-cf980be6-3128-4022-828d-b2a0fa92b1a6.png =600x)
`jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH`

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 15,
    "rankSpacing": 15
  }
}}%%

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

##Validations
This section explains the validations for the whole proccess Palletize Start
- AGC is online. **(DMGroupController.STATUS_FLAG.ONLINE)**
- Selected Station Number is NOT under suspend. **(DMStation.SUSPEND.OFF)**
- Selected Station Number is available. **(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)**
- Pallet Information does not exist in
  To check for Pallet Information:  
  **JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number>**
  So if result > 0, Palletize Start cannot proceed.
- **Planned Carton** must be greater than **0**.
- **Quantity (Carton per Pallet)** must be greater than **0**.
- **Planned Carton** must be greater than or equal to **Quantity (Carton per Pallet)**.
- Material Code exists in **DMITEM**
- Input text with red asterisk <span style="color:red">(*)** is not empty
- If the actual Carton Qty exceeds the planned Qty, continue the storage operation.
  Ex.)
  Planned Carton Qty: 1000
  Actual: 1010
  The purpose to show the Planned Carton Qty is to have the history purpose.
  With this history, user can detect the result is abit exceeded from plan.
  So the Planned Carton Qty doesn't directly affect to the number of Pallet or Carton in Storage Operation.
- If DNCarryInfo found from 1101-1105 to 1111-1115 when Batch start
  - The pallet is for next up coming Batch
  - Update Batch No. on the associated DNStoragePlan

    Ex.)
    ![image.png](/.attachments/image-0fb60da3-6bc2-4fd7-bce4-bce1e8df8f6e.png =400x)
    ![image.png](/.attachments/image-22f2b51e-d1c8-4df5-b6e5-89081d456122.png =400x)

##Table Operation DML**

###DNStoragePlan**
*   **PLAN_UKEY** : Sequence Object    
*   **STATUS_FLAG** : 0: Not Started    
*   **CANCEL_FLAG** : 0: Normal Data    
*   **DELETE_FLAG** : 0: No    
*   **PLAN_DAY** : Value from screen (**Storage Date/Time**)    
*   **PLAN_AREA_NO** : Value from screen (**Storage Location**) : **FGW1 (Ambient/9002)** or **FGW2 (Tempering/9001)**    
*   **MATERIAL_CODE** : Value from screen (**Material Code**) -> ****DMITEM.ITEM_CODE ****   
*   **PLAN_QTY** : Value from screen (**Qty Crtn/PL**)    
*   **REPORT_FLAG** : 0: Not Reported    
*   **STORING_PAIR_KEY** : Value from screen **(Material Code + Batch_No)**    
*   **STATION_NO** : Value from screen (**Station No**)    
*   **CURRENT_STATUS** : 1:Palletizing Started    
*   **MATERIAL_NAME** : Value from screen (**Material Name**) -> ****DMITEM.ITEM_NAME ****    
*   **MATERIAL_TYPE** : Value from screen (**Material Type**) -> ****DMITEM.ITEM_TYPE ****    
*   **BATCH_NO** : Value from screen (**Batch_No**)    
*   **PLANNED_CARTON_QTY** : Value from screen (**Planned Carton Qty**)    
*   **QTY_KG_CRTN** : Value from screen (**Qty Kg/Crtn**) -> ****DMITEM.ENTERING_QTY ****    
*   **UOM** : Value from screen (**UOM**) -> ****DMITEM.UOM ****    
*   **TEMPERING_PERIOD** : Value from screen (**Tempering Period**)    
*   **EXPIRY_DAYS** : Value from screen (**Expiry Days**)    
*   **STORAGE_QTY** : **PLANNED_CARTON_QTY - TOTAL_ACTUAL_CARTON_QTY**    
*   **TOTAL_ACTUAL_CARTON_QTY** : **TOTAL_ACTUAL_CARTON_QTY + PLAN_QTY**    
*   **MAX_PALLET_BATCH_END** : **PLANNED_CARTON_QTY / PLAN_QTY**    
*   **BATCH_PALLET_START** : **PLAN_DAY + System Timestamp (Time Value)**    
*   **REGIST_DATE** : SYSTIMESTAMP    
*   **REGIST_PNAME** : ClassName    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

#Release Command from Palletize Robot - Dummy Arrival
After palletizing is completed, the palletizing robot sends **“Release Command”** signal via conveyor interlock.

<hr style="border: 2px solid red;">

#Irregular Cases
Continue this flow if occurs.
- [Wait for available location if full](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=wait-for-available-location-if-full)
- [No-Read (Go to Reject station)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=no-read-(go-to-reject-station))
- [Wait for Batch Start. Treat as Data Error](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/951/Irregular-Cases-Palletizing?anchor=wait-for-batch-start.-treat-as-data-error)
<hr style="border: 2px solid red;">

##ID26 at 1101-1105

![image.png](/.attachments/image-3a70d81c-6bad-4966-a3d1-0a312b5f9e58.png =700x)

::: mermaid
flowchart LR

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

prePalletizeStationOperator[PrePalletizeStationOperator]

id26msg-->id26process-->prePalletizeStationOperator
prePalletizeStationOperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and PrePalletizeStationOperator will execute the receive task based on information in received ID26. While PrePalletizeStationOperator processes ID26, it will create an Arrival record.

jp.co.daifuku.asrs.communication.id.recv.As21Id26

jp.co.daifuku.asrs.location.PrePalletizeStationOperator

###Table Operation DML**

####DNArrival**
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

####DNPallet**
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

####DNWorkInfo**
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

####DNCarryInfo**
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

####DNStock**
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

###DNStoragePlan** 
*  **STATUS_FLAG**: 1: Working  
*  **BCR_DATA**: DNPALLET.BCR_DATA

##Storage Sender at 1101-1105
jp.co.daifuku.asrs.transmission.StorageSender

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

After successful creation of arrival record in **ID26process**, StorageSender is the following process where it will send **ID05 to AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response.**

###Table Operation DML**

####DNCarryInfo**
*   **CMD_STATUS** : 2: Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####DNArrival**
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1: Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

##ID25 at 1101-1105
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
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

###Table Operation DML**
####DNCarryInfo**
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

palletizeLaneStationOperator[PalletizeLaneStationOperator]

releaseCommand-->id26msg-->id26process-->palletizeLaneStationOperator
palletizeLaneStationOperator--> |INSERT| id26-insert
palletizeLaneStationOperator--> |UPDATE| id26-update
:::

Continue the process **Direct Transfer**, AGC will send ID26 to WareNavi and PalletizeLaneStationOperator will execute the receive task based on information in received ID26. While PalletizeLaneStationOperator processes ID26, it will create an Arrival record..
At the same time, search for Empty location and decide an Aisle to storage(7101-7110).

jp.co.daifuku.asrs.communication.id.recv.As21Id26

jp.co.daifuku.asrs.location.PalletizeLaneStationOperator

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

After successful creation of arrival record in **ID26process**, StorageSender is the following process where it will send **ID05 to AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated **1:Started.**

###Table Operation DML**

####DMWarehouse**
*   **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####DMShelf**
*   **STATUS_FLAG** : 2:Reserved Location    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####DNCarryInfo**
*   **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to    
*   **CMD_STATUS** : 1:Started    
*   **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO : **(1111 / 1112 / 1113 / 1114 / 1115)**
*   **DEST_STATION_NO** : **Based on Empty location search: (7101 / 7102 / 7103 / 7104 / 7105 / 7106 / 7107 / 7108 / 7109 / 7110)**
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####DNStock**
*   **LOCATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####DNWorkInfo**
*   **PLAN_LOCATION_NO** : Reserved Location Number    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

####DNArrival**
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1:Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : Class name

##Storage Sender at 1111-1115
jp.co.daifuku.asrs.transmission.StorageSender

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

After successful creation of arrival record in **ID26process**, StorageSender is the following process where it will send **ID05 to AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response.**

###Table Operation DML**

####DNCarryInfo**
*   **CMD_STATUS** : 2: Waiting for response    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName

####DNArrival**
*   **CARRY_KEY** : DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** : 1: Sent    
*   **LAST_UPDATE_DATE** : SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** : ClassName