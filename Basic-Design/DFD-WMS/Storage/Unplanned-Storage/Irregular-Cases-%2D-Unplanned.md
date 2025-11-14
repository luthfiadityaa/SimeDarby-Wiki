[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR  
ArrivalAt1101[ID26] --> Cond1{Is Location Full?}  
Cond1 -->|Yes| Await[Await] --> Cond1  
Cond1 -->|No| Cond2{Is No Read?}  
Cond2 -->|Yes| Move1[ID05 ControlInfo:NoRead]  
Cond2 -->|No| Move2[ID05]
:::

##Wait for available location if full

###<span style="color:skyblue; font-weight:bold">Database flow</span>
**Abbreviation:**
- **STRP** : DNSTORAGEPLAN  
- **WRKI** : DNWORKINFO  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **ARVL** : DNARRIVAL  
- **STSN** : DMSTATION 

| Action Name                                     |WRK |PLLT|CRYI|STCK|ARRVL|STSN| 
|-------------------------------------------------|----|----|----|----|-----|----|
| ID26[(1)](#ID26-at-1301-1303)                   | I  | I  | I  | I  | I   |    |
| StorageSender[(2)](#StorageSender-at-1301-1303) |    |    | U  |    | U   |    |
| ID25[(4)](#ID25-at-1301-1303)                   |    |    | U  |    | D   |    |
| ID26[(5)](#ID26-at-(1301-1303))                 |    | U  | U  |    | I   |    |

###ID26 at 1301-1303
::: mermaid
flowchart LR

releaseCommand["
Release Command from Conveyor After Completion 
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

storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
:::

After the process is completed, the conveyor receives a signal and begins transferring the pallet. The AGC then sends ID26 to WareNavi, prompting the Storage Station Operator to execute the receiving task based on the information contained in ID26. During this process, the Storage Station Operator creates an Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

#####<span style="color:skyblue; font-weight:bold">DNArrival</span>

- <span style="color:red; font-weight:bold">Control Information = "00␣"
Palletizing Skip("0": None),
No Read("0":None)</span>

*   **ARRIVAL_DATE**: SYSTIMESTAMP    
*   **STATION_NO**: Arrival Station Number from ID26    
*   **CARRY_KEY**: 99999999    
*   **BCR_DATA**: Barcode information from ID26    
*   **CONTROLINFO**: <span style="color:red; font-weight:bold">Control information from ID26</span>    
*   **SEND_FLAG**: 0:Not sent    
*   **HEIGHT**: Dimension Information from ID26    
*   **WIDTH**: Dimension Information From ID26    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **PALLET_ID**: Sequence Object    
*   **CURRENT_STATION_NO**: DNSTORAGEPLAN.STATION_NO    
*   **WH_STATION_NO**: DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STATUS_FLAG**: 1:Reserved for Storage    
*   **EMPTY_FLAG**: 0:Normal Pallet    
*   **ALLOCATION_FLAG**: 1:Allocated    
*   **SOFT_ZONE_ID**: DMITEM.SOFT_ZONE_ID    
*   **BCR_DATA**: DNARRIVAL.BCR_DATA    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 02:Storage    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: DNSTORAGEPLAN.PLAN_UKEY    
*   **STOCK_ID**: Sequence Object    
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: DNSTORAGEPLAN.BATCH_NO    
*   **PLAN_AREA_NO**: DNSTORAGEPLAN.STORAGE_LOCATION    
*   **MATERIAL_CODE**: DNSTORAGEPLAN.MATERIAL_CODE    
*   **PLAN_QTY**: DNSTORAGEPLAN.QTY_CRTN_PL    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **USER_ID**: Login Info    
*   **STORAGE_LOCATION_FROM**: DNSTORAGEPLAN.STATION_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **WORK_TYPE**: 26:Direct Transfer    
*   **CMD_STATUS**: 1:Started    
*   **PRIORITY**: 2:Normal    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location    
*   **CARRY_FLAG**: 3: Direct Transfer    
*   **WORK_NO**: Sequence Object    
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO ⟶ **P(1301, 1302)** or **T(1303)**    
*   **DEST_STATION_NO**: Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **P(7101, 7102, 7103, 7104, 7105, 7106, 7107, 7108, 7109, 7110)** or **T(7207, 7208, 7209, 7210, 7211, 7213, 7214)**    
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **END_STATION_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **STOCK_ID**: Sequence Object    
*   **AREA_NO**: DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STORAGE_TYPE**: 2:New    
*   **STOCK_QTY**: 0    
*   **ALLOCATION_QTY**: 0    
*   **PLAN_QTY**: DNSTORAGEPLAN.PLAN_QTY    
*   **PALLET_ID**: Sequence Object    
*   **BATCH_NO**: DNSTORAGEPLAN.BATCH_NO    
*   **TEMPERING_PERIOD**: DNSTORAGEPLAN.TEMPERING_PERIOD    
*   **STORING_PAIR_KEY**: **DNSTORAGEPLAN.STORING_PAIR_KEY**    
*   **EXPIRY_DATE**: DNSTORAGEPLAN.EXPIRY_DAYS    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###Storage Sender at 1301-1303
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

storageSender-input-->StorageSender-->id05msg
StorageSender--> |UPDATE| storageSender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, StorageSender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS**: 2:Waiting for response    
*   **CONTROL_INFO**: **DNARRIVAL.CONTROL_INFO**    
*   **REJECT_FACTOR**: **08:Location Full**    
*   **WAIT_REASON**: **07:No Available Storage Location**    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

#####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG**: 1:Sent    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###ID25 at 1301-1303

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
####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS**: 3:Commanded    
*   **ERROR_CODE**: 0    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###ID26 at (1301-1303)
(After the await operation completes and the empty locations are identified.)

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

releaseCommand2["
Wait until an empty location becomes available because all locations are full.
"]

releaseCommand4["
LocationManager
"]

releaseCommand3["
An empty location becomes available. 
"]

storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert-->releaseCommand4
storageStationOperator--> |UPDATE| id26-update-->releaseCommand4
releaseCommand4-->releaseCommand2-->releaseCommand3
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and Storage Station Operator will execute the receive task based on information in received ID26. While Storage Station Operator processes ID26, it will create an Arrival record..

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

#####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

#####<span style="color:skyblue; font-weight:bold">DNArrival</span>
- <span style="color:red; font-weight:bold">Control Information = "110"
Palletizing Skip("1": Skip),
No Read("1":Error),
Palletizing Completion("0":Normal Completion)</span>

*   **ARRIVAL_DATE**: SYSTIMESTAMP    
*   **STATION_NO**: Arrival Station Number from ID26    
*   **CARRY_KEY**: 99999999    
*   **BCR_DATA**: Barcode information from ID26    
*   **CONTROLINFO**:  <span style="color:red; font-weight:bold">Control information from ID26</span> 
*   **SEND_FLAG**: 0: Not sent    
*   **HEIGHT**: Dimension Information from ID26    
*   **WIDTH**: Dimension Information From ID26    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO**: DNARRIVAL.STATION_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE**: 26:Direct Transfer    
*   **CMD_STATUS**: 1:Started    
*   **CARRY_FLAG**: 3: Direct Transfer    
*   **SOURCE_STATION_NO**: DNARRIVAL.STATION_NO ⟶ <span style="color:green; font-weight:bold">P(1301, 1302) or T(1303)</span>    
*   **DEST_STATION_NO**: <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ P(7101, 7102, 7103, 7104, 7105, 7106, 7107, 7108, 7109, 7110) or T(7207, 7208, 7209, 7210, 7211, 7213, 7214)</span>    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

<br>

**IF FROM HP AREA**

- [Continue to Normal Flow FROM HP AREA -> StorageSender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/948/Unplanned-Storage-from-HP?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eautomatic-mode-change-sender%3C/span%3E)

**ELSE IF FROM OP AREA**

- [Continue to Normal Flow FROM OP AREA -> StorageSender](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/880/Unplanned-Storage-from-OP-Area?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eautomatic-mode-change-sender%3C/span%3E)

<hr>

##No-Read

###<span style="color:skyblue; font-weight:bold">Database flow</span>
**Abbreviation:**
- **STRP** : DNSTORAGEPLAN  
- **WRKI** : DNWORKINFO  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **ARVL** : DNARRIVAL  
- **STSN** : DMSTATION 

| Action Name                 | WRKI   | PLLT   | CRYI   | STCK   | ARRVL  | STSN   | 
|-----------------------------|--------|--------|--------|--------|--------|--------|
| ID26(1)                     | INSERT | INSERT | INSERT | INSERT | INSERT |        |
| ID54(2)                     |        |        |        |        |        |        |

###ID26 at [1301-1303]

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

storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
:::

After the process is completed, the conveyor receives a signal and begins transferring the pallet. The AGC then sends ID26 to WareNavi, prompting the Storage Station Operator to execute the receiving task based on the information contained in ID26. During this process, the Storage Station Operator creates an Arrival record. However, if the barcode data check results in a No-Read, the system will handle it accordingly.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

####<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

#####<span style="color:skyblue; font-weight:bold">DNArrival</span>

- <span style="color:red; font-weight:bold">Control Information = "11␣"
Palletizing Skip("1": Skip),
No Read("1":Error)</span>

*   **ARRIVAL_DATE**: SYSTIMESTAMP    
*   **STATION_NO**: Arrival Station Number from ID26    
*   **CARRY_KEY**: 99999999    
*   **BCR_DATA**: <span style="color:red; font-weight:bold">??</span>    
*   **CONTROLINFO**: <span style="color:red; font-weight:bold">Control information from ID26</span>    
*   **SEND_FLAG**: 0:Not sent    
*   **HEIGHT**: Dimension Information from ID26    
*   **WIDTH**: Dimension Information From ID26    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **PALLET_ID**: Sequence Object    
*   **CURRENT_STATION_NO**: DNSTORAGEPLAN.STATION_NO    
*   **WH_STATION_NO**: DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STATUS_FLAG**: 1:Reserved for Storage    
*   **EMPTY_FLAG**: 0:Normal Pallet    
*   **ALLOCATION_FLAG**: 1:Allocated    
*   **SOFT_ZONE_ID**: DMITEM.SOFT_ZONE_ID    
*   **BCR_DATA**: DNARRIVAL.BCR_DATA    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 02:Storage    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: DNSTORAGEPLAN.PLAN_UKEY    
*   **STOCK_ID**: Sequence Object    
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: DNSTORAGEPLAN.BATCH_NO    
*   **PLAN_AREA_NO**: DNSTORAGEPLAN.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNSTORAGEPLAN.MATERIAL_CODE    
*   **PLAN_QTY**: DNSTORAGEPLAN.PLAN_QTY    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **USER_ID**: Login Info    
*   **STORAGE_LOCATION_FROM**: DNSTORAGEPLAN.STATION    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **WORK_TYPE**: 26:Direct Transfer    
*   **CMD_STATUS**: 1:Started    
*   **PRIORITY**: 2:Normal    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location    
*   **CARRY_FLAG**: 3: Direct Transfer    
*   **WORK_NO**: Sequence Object    
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO ⟶ <span style="color:green; font-weight:bold">(1301, 1302)</span>    
*   **DEST_STATION_NO**: <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7207, 7208, 7209, 7210, 7211, 7213, 7214)</span>    
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **END_STATION_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#####<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **STOCK_ID**: Sequence Object    
*   **AREA_NO**: DNSTORAGEPLAN.PLAN_AREA_NO    
*   **STORAGE_TYPE**: 2:New    
*   **STOCK_QTY**: 0    
*   **ALLOCATION_QTY**: 0    
*   **PLAN_QTY**: DNSTORAGEPLAN.PLAN    
*   **PALLET_ID**: Sequence Object    
*   **BATCH_NO**: DNSTORAGEPLAN.BATCH_NO    
*   **TEMPERING_PERIOD**: DNSTORAGEPLAN.TEMPERING_PERIOD    
*   **STORING_PAIR_KEY**: <span style="color:green; font-weight:bold">DNSTORAGEPLAN.STORING_PAIR_KEY</span>    
*   **EXPIRY_DATE**: DNSTORAGEPLAN.EXPIRY_DAYS    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">ID54</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.id.sendAs21Id54&nbsp;</span>
::: mermaid
flowchart LR

id50msg("
ID 54
")

buttonlight["
The signal tower lights.
The buzzer sounds.
"]

buttonlight1["
Conveyor reverse back
"]

id50msg --> As21Id54
As21Id54 --> buttonlight --> buttonlight1
:::

After WareNavi receives information from **ID26** indicating a barcode NO-READ, it automatically sends **ID54** to the AGC. At the designated station, the signal tower lights up and the buzzer sounds, and initiate pallet transport.
Press the completion button.

###<span style="color:skyblue; font-weight:bold">Manual Operation</span>
::: mermaid
flowchart LR

buttonlight2["
The signal tower turns off.
The buzzer stops.
"]

buttonlight1["
Press the completion button.
"]

buttonlight3["
Storage plan is imported.
Correct BC label.
"]

buttonlight3-->buttonlight2-->buttonlight1
:::

<br>

**IF FROM HP AREA**

- [Continue to Normal Flow FROM HP AREA -> ID26](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/948/Unplanned-Storage-from-HP?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E)

**ELSE IF FROM OP AREA**

- [Continue to Normal Flow FROM OP AREA -> ID26](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/880/Unplanned-Storage-from-OP-Area?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eid26%3C/span%3E)

