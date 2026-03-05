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

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

    input[
        Station
        Current Status
        Material Code, Name, Type
        Batch #
        Planned Carton Qty
        Storage Date/Time
        Qty kg/crtn, Qty crtn/PL
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

#ID26 at 1101-1105

![image.png](/.attachments/image-3a70d81c-6bad-4966-a3d1-0a312b5f9e58.png =700x)

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

id26msg("
ID 26
")

id26-insert[("
   DNARRIVAL
   DNPALLET(for EMP_PB)
   DNCARRYINFO(for EMP_PB)
   DNSTOCK(for EMP_PB)
")]

prePalletizeStationOperator[PrePalletizeStationOperator]

id26msg-->id26process-->prePalletizeStationOperator
prePalletizeStationOperator--> |INSERT| id26-insert
:::

#Storage Sender at 1101-1105

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

storageSender-update[("
DNCARRYINFO(for EMP_PB)
DNARRIVAL
")]
storageSender-input[("
DNARRIVAL
DNCARRYINFO(for EMP_PB)
")]

id05msg("
ID 05
")

storageSender-input-->storageSender-->id05msg
storageSender--> |UPDATE| storageSender-update
:::


#ID25 at 1101-1105

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

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

#ID26 at 1111-1115
- DNStoragePlan.status_flag
  - 0:Not Started = This is not used for palletizing operation.
  - 1:Working = When Batch Started on Screen.
  - 2:Waiting for Bath End = After receive ID26 at 1111-1115 with ControlInfo Force Completion.
  - 3:Batch Ended = When Batch Ended on Screen.
  - 4:Completed = When receive ID33 for last pallet.
  - 9:Deleted = Due to maintenance.

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
DNWORKINFO
")]

id26-update[("
DNSTORAGEPLAN
DNPALLET
DNCARRYINFO
")];

prePalletizeStationOperator[PrePalletizeStationOperator]

id26msg-->id26process-->prePalletizeStationOperator
prePalletizeStationOperator--> |INSERT| id26-insert
prePalletizeStationOperator--> |UPDATE| id26-update
:::

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

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

#Storage Sender at 1111-1115

::: mermaid
flowchart LR

%%{init:{
  "themeVariables": { "fontSize": "10px" },
  "flowchart": {
    "useMaxWidth": false,
    "diagramPadding": 5,
    "nodeSpacing": 10,
    "rankSpacing": 30
  }
}}%%

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