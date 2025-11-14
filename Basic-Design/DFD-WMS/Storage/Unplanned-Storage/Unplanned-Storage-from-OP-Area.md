[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">QC Station from Manual Palletizing</span>
![==image_0==.png](/.attachments/==image_0==-479f50f3-14b1-4640-8413-8645cc9ed164.png) 
If a downtime occurs on one of the lines, manual palletizing will be carried out in the blue box. Once palletizing is complete, the items will be transferred to the inbound station, which is the same station used for QC.

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR
  
P1[FROM OP STATION - 1303]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR
  
Cond1{Flow?} 
Cond1 --> |Flow 1: Without going 
through crane 7~10 - STV| P8[ID64]
Cond1 --> |Flow 2: Go through 
crane 7~10 -STV| P9[ID64]
P8[ID64]--> |7101-7110| P10[ID26]-->P11[StorageSender]-->P12[ID25]--> |SRM| P18[ID64]
P9[ID64]--> |7107-7110| P15[ID26]-->P16[StorageSender]-->P17[ID25]--> |SRM| P18[ID64]
:::

**Stage 3**
::: mermaid
flowchart LR 
P18[ID64]--> |STV| P19[ID64]--> |7211-7214| P20[ID26]-->P21[StorageSender]-->P22[ID25]--> |SRM| P23[ID64]

P18[ID64]--> |9001-9010| P14[ID33]
P23[ID64]--> |9011-9014|P14[ID33]
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
**Abbreviation:**
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

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                                                      |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN| 
|------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|
| ID63 [(1)](#ID63)                                                |    |    |    |    |    |    |     |    |    |    |    | U  |   
| Unplanned Storage - Set(F2) [(2)](#Unplanned-Storage---Set(F2)) | I  |    |    |    |    |    |     |    |    |    | S  |    |  
| ID26[(3)](#ID26-at-1301-&-1302)                                  |    |    | I  | I  | I  |    | I   |    |    |    |    | S  |  
| StorageSender[(4)](#Storage-Sender-at-1301-&-1302)               | U  |    | U  | U  | U  |    | U   | U  | U  |    |    |    |    
| ID25[(5)](#ID25-at-1301-&-1302)                                  |    |    |    | U  |    |    | D   |    |    |    |    |    |     
| ID64[(6)](#ID64-at-STV-From-1301-&-1302)                         |    |    |    | U  |    |    |     |    |    |    |    |    |     
| **Flow 1**                                                       |    |    |    |    |    |    |     |    |    |    |    |    | 
| ID26[(7)](#ID64-at-7207-7214)                                    |    |    | U  | U  |    |    | I   |    |    |    |    |    |
| StorageSender[(10)](#Storage-Sender-at-7207-7214)                |    |    |    | U  |    |    | U   |    |    |    |    |    | 
| ID25[(8)](#ID25-at-7207-7214)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    | 
| ID64[(9)](#ID64-at-SRM-From-7207-7214)                          |    |    |    | U  |    |    |     |    |    |    |    |    | 
| **Flow 2**                                                       |    |    |    |    |    |    |     |    |    |    |    |    | 
| ID26[(7)](#ID64-at-7207-7210)                                    |    |    | U  | U  |    |    | I   |    |    |    |    |    |
| StorageSender[(8)](#Storage-Sender-at-7207-7210)                |    |    |    | U  |    |    | U   |    |    |    |    |    |        
| ID25[(9)](#ID25-at-7207-7210)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64[(10)](#ID64-at-STV-From-7207-7210)                          |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID64[(11)](#ID64-at-SRM-From-7207-7210)                          |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID26[(12)](#ID26-at-7101-7106)                                   |    |    | U  | U  |    |    | I   |    |    |    |    |    |       
| StorageSender[(15)](#Storage-Sender-at-7101-7106)                |    |    |    | U  |    |    | U   |    |    |    |    |    |
| ID25[(13)](#ID25-at-7101-7106)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64[(14)]((#ID64-at-SRM-From-7101-7106)                         |    |    |    | U  |    |    |     |    |    |    |    |    |
| **Last Process**                                                 |    |    |    |    |    |    |     |    |    |    |    |    |    
| ID33[(15)](#)                                                    | U  | I  | U  | D  | U  | I  |     |    | U  | I  |    |    |
| **Host Communication**                                           |    |    |    |    |    |    |     |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |    |    |    |     |    |    |    |    |    | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##<span style="color:skyblue; font-weight:bold">ID63</span>
If the station mode is **Retrieval Mode**, change the mode of the station to **Storage mode**.
::: mermaid
flowchart LR
    input[
        Operators pressed the storage mode button on the operation box.      
    ]

    id61msg("
     ID63
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id61msg-->id63process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DMSTATION</span>
**CURRENT_MODE**: 1:Storage Mode

#<span style="color:skyblue; font-weight:bold">Unplanned Storage - Set(F2)</span>
![image.png](/.attachments/image-6293c4f4-5fd4-4247-8cbc-fbcc6d9cf3a5.png)
 
The Unplanned Storage Setting screen uses for manually set the storage work .
This screen mainly uses when there are some troubles with Host System Linkage(irregular cases) or when manual operation is necessary.. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wms.web.display.storage.unplannedstorage.UnplannedStorageSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Pallet ID
        Material Code
        Material Name
        Material Type
        Qty kg/crtn
        Qty crtn/PL
        Batch #
        Storage Qty
        UOM
        Tempering Period
        Expiry Days
        Storage Location From
        Storage Location To 
    ]

    tableList-insert[("
        DNWORKINFO
    ")]

    tableList-select[("
        DMITEM
        DMTOSTATION
    ")]


    className[UnplannedStorageSCH]

    input --> className --> |INSERT| tableList-insert
    className --> |SELECT| tableList-select

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.
- Material Code exists in <span style="color:green; font-weight:bold">DMITEM</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 22: Unplanned Storage    
*   **STATUS_FLAG**: 0: Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: Sequence Object    
*   **PLAN_AREA_NO**: Value from screen (Storage Location To) ⟶ <span style="color:green; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>    
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: Value from screen (Batch No)    
*   **MATERIAL_CODE**: Value from screen (Material Code)    
*   **MATERIAL_NAME**: Value from screen (Material Name)    
*   **PLAN_QTY**: Value from screen (Storage Qty)    
*   **QTY_KG_CRTN**: Value from screen (Qty kg/crtn)    
*   **QTY_CRTN_PL**: Value from screen (Qty crtn/PL)    
*   **UOM**: Value from screen (UOM)    
*   **TEMPERING_PERIOD**: Value from screen (Tempering Period)    
*   **EXPIRY_DAYS**: Value from screen (Expiry Days)    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **USER_ID**: Login Info    
*   **STORAGE_LOCATION_FROM**: Value from screen (Storage Location From) ⟶ <span style="color:green; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

<br>
<hr style="border: 2px solid red;">

#Irregular Cases
Continue this flow if occurs.
- [Wait for available location if full](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/953/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Ewait-for-available-location-if-full%3C/span%3E)
- [No-Read (Reverse Back to Previous Station)](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/953/Irregular-Cases?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eno-read%3C/span%3E)
<hr style="border: 2px solid red;">
 
#<span style="color:skyblue; font-weight:bold">ID26</span>

::: mermaid
flowchart LR

releaseCommand["
Operator puts the pallet onto Conveyor
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
DNPALLET
DNSTOCK
DNCARRYINFO
")]

inoutstationoperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->inoutstationoperator
inoutstationoperator--> |INSERT| id26-insert
:::

After Completion, Conveyor receives the signal and starts transferring the pallet. AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **ARRIVAL_DATE**: SYSTIMESTAMP    
*   **STATION_NO**: Arrival Station Number from ID26    
*   **CARRY_KEY**: 99999999    
*   **BCR_DATA**: Barcode information from ID26    
*   **CONTROLINFO**: Control information from ID26    
*   **SEND_FLAG**: 0: Not sent    
*   **HEIGHT**: Dimension Information from ID26    
*   **WIDTH**: Dimension Information From ID26    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **PALLET_ID**: Sequence Object    
*   **CURRENT_STATION_NO**: DNARRIVAL.STATION_NO -> <span style="color:green; font-weight:bold">1303</span>    
*   **WH_STATION_NO**: Value from screen (Storage Location To) ⟶ <span style="color:green; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>    
*   **STATUS_FLAG**: 1: Reserved for Storage    
*   **EMPTY_FLAG**: 0: Normal Pallet    
*   **BCR_DATA**: Value from screen (Pallet ID)    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **STOCK_ID**: Sequence Object    
*   **AREA_NO**: Value from screen (Storage Location To) ⟶ <span style="color:green; font-weight:bold">FGW1 (Ambient/9002) or FGW2 (Tempering/9001)</span>    
*   **LOCATION_NO**: 22222222    
*   **STORAGE_TYPE**: 2: New    
*   **QTY_CRTN_PL**: Value from screen (Qty crtn/PL)    
*   **QTY_KG_CRTN**: Value from screen (Qty kg/crtn)    
*   **ALLOCATION_QTY**: 0    
*   **PLAN_QTY**: Value from screen (Storage Qty)    
*   **PALLET_ID**: Sequence Object    
*   **BATCH_NO**: Value from screen (Batch No)    
*   **TEMPERING_PERIOD**: Value from screen (Tempering Period)    
*   **STORING_PAIR_KEY**: <span style="color:green; font-weight:bold">DNWORKINFO.MATERIAL_CODE + DNWORKINFO.BATCH_NO</span>    
*   **EXPIRY_DATE**: Value from screen (Expiry Days)    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **WORK_TYPE**: 26: Direct Transfer    
*   **CMD_STATUS**: 1: Started    
*   **PRIORITY**: 2: Normal    
*   **RESTORING_FLAG**: 0: Not Restore to Original Location    
*   **CARRY_FLAG**: 3: Direct Transfer    
*   **WORK_NO**: Sequence Object    
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO    
*   **DEST_STATION_NO**: <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7101, 7102, 7103, 7104, 7105, 7206, 7207, 7208, 7109, 7110)</span>    
*   **CANCEL_REQUEST**: 0: Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **END_STATION_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

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

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DMWarehouse</span>
**LAST_USED_STATION_NO**: Aisle Number where a reserved location belongs to

####<span style="color:skyblue; font-weight:bold">DMShelf</span>
*   **STATUS_FLAG**: 2:Reserved Location    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **AISLE_STATION_NO**: Aisle Number where a reserved location belongs to    
*   **CMD_STATUS**: 2:Waiting for response    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **PLAN_LOCATION_NO**: Reserved Location Number    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO**: Reserved Location Number    
*   **WH_STATION_NO**: DNCARRYINFO.END_STATION_NO    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
*   **AREA_NO** — DNCARRYINFO.END_STATION_NO    
*   **LOCATION_NO** — Reserved Location Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** — DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** — 1:Sent    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

#<span style="color:skyblue; font-weight:bold">ID25</span>

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
*   **CMD_STATUS** — 3:Commanded    
*   **ERROR_CODE** — 0    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ###[Flow 1](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/880/Unplanned-Storage-from-OP-Area?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eflow-1-(refer-to-agc-linkage-specs)%3C/span%3E)
- ###[Flow 2](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/880/Unplanned-Storage-from-OP-Area?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Eflow-2-(refer-to-agc-linkage-specs)%3C/span%3E)

<hr>

#<span style="color:skyblue; font-weight:bold">Flow 1 (Refer to AGC Linkage Specs)</span>
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)
##<span style="color:skyblue; font-weight:bold">ID64</span>
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
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>

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


storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CURRENT_STATION_NO** — Reserved Location Number    
*   **WH_STATION_NO** — Reserved Area Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** — Reserved Location Number    
*   **WH_STATION_NO** — Reserved Area Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** — 2:Storage    
*   **CMD_STATUS** — 1:Started    
*   **CARRY_FLAG** — 1:Storage    
*   **AISLE_STATION_NO** — Aisle Number where a reserved location belongs to    
*   **SOURCE_STATION_NO** — DNARRIVAL.STATION_NO    
*   **DEST_STATION_NO** — Reserved Location Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">AutomaticChangeSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutomaticChangeSender &nbsp;</span>

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

automaticmodechangesender-input-->AutomaticChangeSender-->id05msg
AutomaticChangeSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS** — 2: Waiting for response    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** — DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** — 1:Sent    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">&nbsp;ID25&nbsp;</span>

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
*   **CMD_STATUS** — 3:Commanded    
*   **ERROR_CODE** — 0    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
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
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Flow 2 (Refer to AGC Linkage Specs)</span>
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
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
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>

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

storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and Storage Station Operator will execute the receive task based on information in received ID26. While Storage Station Operator processes ID26, it will create an Arrival record..

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **ARRIVAL_DATE** — SYSTIMESTAMP    
*   **STATION_NO** — Arrival Station Number from ID26    
*   **CARRY_KEY** — 99999999    
*   **BCR_DATA** — Barcode information from ID26    
*   **CONTROLINFO** — Control information from ID26    
*   **SEND_FLAG** — 0:Not sent    
*   **HEIGHT** — Dimension Information from ID26    
*   **WIDTH** — Dimension Information from ID26    
*   **REGIST_DATE** — SYSTIMESTAMP    
*   **REGIST_PNAME** — ClassName    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP
    
*   **LAST_UPDATE_PNAME** — ClassName
###<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** — DNARRIVAL.STATION_NO    
*   **REGIST_DATE** — SYSTIMESTAMP    
*   **REGIST_PNAME** — ClassName    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** — 26:Direct Transfer    
*   **CMD_STATUS** — 1:Started    
*   **CARRY_FLAG** — 3: Direct Transfer    
*   **SOURCE_STATION_NO** — DNARRIVAL.STATION_NO ⟶ <span style="color:green; font-weight:bold">(7107, 7108, 7109, 7110)</span>    
*   **DEST_STATION_NO** — <span style="color:green; font-weight:bold">Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (7211, 7212, 7213, 7214)</span>    
*   **REGIST_DATE** — SYSTIMESTAMP    
*   **REGIST_PNAME** — ClassName    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — ClassName


##<span style="color:skyblue; font-weight:bold">AutomaticChangeSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutomaticChangeSender &nbsp;</span>

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

automaticmodechangesender-input-->AutomaticChangeSender-->id05msg
AutomaticChangeSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1: Started to 2: Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS** — 2: Waiting for response    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** — DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** — 1:Sent    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID25&nbsp;</span>

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
*   **CMD_STATUS** — 3:Commanded    
*   **ERROR_CODE** — 0    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID64</span>
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
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID64</span>
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
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID26&nbsp;&nbsp;</span>

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

storageStationOperator[InOutStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process <span style="color:green; font-weight:bold">Direct Transfer</span>, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.recv.As21Id26 &nbsp;</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **ARRIVAL_DATE** — SYSTIMESTAMP    
*   **STATION_NO** — Arrival Station Number from ID26    
*   **CARRY_KEY** — 99999999    
*   **BCR_DATA** — Barcode information from ID26    
*   **CONTROLINFO** — Control information from ID26    
*   **SEND_FLAG** — 0:Not sent    
*   **HEIGHT** — Dimension Information from ID26    
*   **WIDTH** — Dimension Information from ID26    
*   **REGIST_DATE** — SYSTIMESTAMP    
*   **REGIST_PNAME** — ClassName    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — ClassName

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** — Reserved Location Number    
*   **WH_STATION_NO** — Reserved Area Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **WORK_TYPE** — 2:Storage    
*   **CMD_STATUS** — 1:Started    
*   **CARRY_FLAG** — 1:Storage    
*   **AISLE_STATION_NO** — Aisle Number where a reserved location belongs to    
*   **SOURCE_STATION_NO** — DNARRIVAL.STATION_NO    
*   **DEST_STATION_NO** — Reserved Location Number    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">AutomaticChangeSender&nbsp;&nbsp;</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.AutomaticChangeSender &nbsp;</span>

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

automaticmodechangesender-input-->AutomaticChangeSender-->id05msg
AutomaticChangeSender--> |UPDATE| automaticmodechangesender-update
:::

After successful creation of arrival record in <span style="color:green; font-weight:bold">ID26process</span>, Automatic Change Sender is the following process where it will send <span style="color:green; font-weight:bold">ID05 to AGC</span>. To indicate <span style="color:green; font-weight:bold">ID05</span> is sent to AGC, <span style="color:green; font-weight:bold">DNCARRYINFO.CMD_STATUS</span> will be updated from <span style="color:green; font-weight:bold">1:Started to 2:Waiting for Response.</span>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CMD_STATUS** — 2: Waiting for response    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNArrival</span>
*   **CARRY_KEY** — DNCARRYINFO.CARRY_KEY    
*   **SEND_FLAG** — 1:Sent    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">&nbsp;ID25&nbsp;</span>

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
*   **CMD_STATUS** — 3:Commanded    
*   **ERROR_CODE** — 0    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

##<span style="color:skyblue; font-weight:bold">ID64&nbsp;&nbsp;</span>
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
*   **CMD_STATUS** — 4:Pickup completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-31228a41-0618-46d0-acf0-93f1edd79e4e.png)

<hr>

##<span style="color:skyblue; font-weight:bold">ID33</span>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process&nbsp;</span>

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
*   **STATUS_FLAG** — 1: Occupied    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
*   **CURRENT_STATION_NO** — Location Number information from ID33    
*   **STATUS_FLAG** — 2:Occupied    
*   **ALLOCATION_FLAG** — 0:Not allocated    
*   **LAST_STORED_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **RESULT_QTY** — DNWORKINFO.PLAN_QTY    
*   **RESULT_AREA_NO** — DNWORKINFO.PLAN_AREA_NO    
*   **RESULT_LOCATION_NO** — DNWORKINFO.PLAN_LOCATION_NO    
*   **WORK_DAY** — DMWARENAVISYSTEM.WORK_DAY    
*   **STATUS_FLAG** — 4:Completed    
*   **LAST_UPDATE_DATE** — SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME** — Class name

####<span style="color:skyblue; font-weight:bold">DNStock</span>
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)

* **STORAGE_DAY** — DMWARENAVISYSTEM.WORK_DAY  
* **NEWEST_STORAGE_DATE** — SYSTIMESTAMP  
* **STOCK_QTY** — DNWORKINFO.RESULT_QTY  
* **ALLOCATION_QTY** — DNWORKINFO.RESULT_QTY  
* **PLAN_QTY** — 0  
* **STOCK_STATUS** — <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>  
* **TEMPERING_FLAG** — <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span>  
* **QC_FLAG** — <span style="color:green; font-weight:bold">0: Not Done</span>  
* **LAST_UPDATE_DATE** — SYSTIMESTAMP  
* **LAST_UPDATE_PNAME** — Class name  

####<span style="color:skyblue; font-weight:bold">DNStockHistory </span> 
[<span style="color:red; font-weight:bold; font-size:12px">*Tempering Flag is only valid for Storage location: Tempering</span>](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/874/Batch-End?anchor=%3Cspan-style%3D%22color%3Askyblue%3B-font-weight%3Abold%22%3Estorage-flag-condition%3C/span%3E)

* **WORK_DAY** — DMWARENAVISYSTEM.WORK_DAY  
* **INC_DEC_TYPE** — 1: Stock Increase  
* **JOB_TYPE** — 2: Storage  
* **UPDATE_STOCK_QTY** — DNSTOCK.STOCK_QTY  
* **INC_DEC_QTY** — DNSTOCK.STOCK_QTY  
* **STOCK_ID** — DNSTOCK.STOCK_ID  
* **AREA_NO** — DNSTOCK.AREA_NO  
* **LOCATION_NO** — DNSTOCK.LOCATION_NO  
* **STORAGE_DAY** — DNSTOCK.STORAGE_DAY  
* **STORAGE_DATE** — DNSTOCK.STORAGE_DATE  
* **NEWEST_STORAGE_DATE** — DNSTOCK.NEWEST_STORAGE_DATE  
* **MATERIAL_CODE** — DNSTOCK.MATERIAL_CODE  
* **PALLET_ID** — DNSTOCK.PALLET_ID  
* **BCR_DATA** — DNPALLET.BCR_DATA  
* **AREA_TYPE** — DMAREA_AREA_TYPE  
* **MATERIAL_NAME** — DMITEM.MATERIAL_NAME  
* **ENTERING_QTY** — DNSTORAGEPLAN.QTY_CRTN_PL  
* **USER_ID** — Login info  
* **USER_NAME** — Login info  
* **TERMINAL_NO** — Login info  
* **TERMINAL_NAME** — Login info  
* **IP_ADDRESS** — Login info  
* **STOCK_STATUS** — <span style="color:green; font-weight:bold">UU: Unrestricted Used</span>  
* **TEMPERING_FLAG** — <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span>  
* **QC_FLAG** — <span style="color:green; font-weight:bold">0: Not Done</span>  
* **EXPIRY_DATE** — DNWORKINFO.EXPIRY_DAYS  
* **REGIST_DATE** — SYSTIMESTAMP  
* **REGIST_PNAME** — Class name  

####<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **WORK_DAY** — DMWARENAVISYSTEM.WORK_DAY  
* **JOB_NO** — DNWORKINFO.JOB_NO  
* **COLLECT_JOB_NO** — DNWORKINFO.COLLECT_JOB_NO  
* **SETTING_UNIT_KEY** — DNWORKINFO.SETTING_UNIT_KEY  
* **JOB_TYPE** — DNWORKINFO.JOB_TYPE  
* **STATUS_FLAG** — DNWORKINFO.STATUS_FLAG  
* **HARDWARE_TYPE** — DNWORKINFO.HARDWARE_TYPE  
* **PLAN_UKEY** — DNWORKINFO.PLAN_UKEY  
* **STOCK_ID** — DNWORKINFO.STOCK_ID  
* **SYSTEM_CONN_KEY** — DNWORKINFO.SYSTEM_CONN_KEY  
* **PLAN_DAY** — DNWORKINFO.PLAN_DAY  
* **BATCH_NO** — DNWORKINFO.BATCH_NO  
* **PLAN_AREA_NO** — DNWORKINFO.PLAN_AREA_NO  
* **PLAN_LOCATION_NO** — DNWORKINFO.PLAN_LOCATION_NO  
* **MATERIAL_CODE** — DNWORKINFO.MATERIAL_CODE  
* **MATERIAL_NAME** — DMITEM.MATERIAL_NAME  
* **UOM** — MMATERIALMASTER.UOM  
* **ENTERING_QTY** — DNSTORAGEPLAN.QTY_CRTN_PL  
* **BUNDLE_ENTERING_QTY** — DNPALLETIZE.PLANNED_CARTON_QTY  
* **PLAN_QTY** — DNWORKINFO.PLAN_QTY  
* **RESULT_QTY** — DNWORKINFO.RESULT_QTY  
* **SHORTAGE_QTY** — DNWORKINFO.SHORTAGE_QTY  
* **RESULT_AREA_NO** — DNWORKINFO.RESULT_AREA_NO  
* **RESULT_LOCATION_NO** — DNWORKINFO.RESULT_LOCATION_NO  
* **RESULT_LOT_NO** — DNWORKINFO.RESULT_LOT_NO  
* **USER_ID** — DNWORKINFO.USER_ID  
* **TERMINAL_NO** — DNWORKINFO.TERMINAL_NO  
* **WORK_SECOND** — DNWORKINFO.WORK_SECOND  
* **USER_NAME** — DCUSER.USERNAME  
* **REPORT_FLAG** — 0: Not Reported  
* **REGIST_DATE** — SYSTIMESTAMP  
* **REGIST_PNAME** — Class name  
* **LAST_UPDATE_DATE** — SYSTIMESTAMP  
* **LAST_UPDATE_PNAME** — Class name  

####<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
* **RESULT_KIND** — 1:Storage(Stock+)  
* **STATION_NO** — DNCARRYINFO.SOURCE_STATION_NO  
* **LOCATION_NO** — DNCARRYINFO.DEST_STATION_NO  
* **WH_STATION_NO** — DNPALLET.WH_STATION_NO  
* **AISLE_STATION_NO** — DNCARRYINFO.AISLE_STATION_NO  
* **WORK_TYPE** — DNCARRYINFO.WORK_TYPE  
* **RETRIEVAL_DETAIL** — DNCARRYINFO.RETRIEVAL_DETAIL  
* **WORK_NO** — DNCARRYINFO.WORK_NO  
* **PALLET_ID** — DNPALLET.PALLET_ID  
* **CARRY_KEY** — DNCARRYINFO.CARRY_KEY  
* **RESTORING_FLAG** — DNCARRYINFO.RESTORING_FLAG  
* **WORK_DAY** — DMWARENAVISYSTEM.WORK_DAY  
* **REMOVE_FLAG** — 00:Normal  
* **REGIST_PNAME** — Class name  
* **LAST_UPDATE_DATE** — SYSTIMESTAMP  
* **LAST_UPDATE_PNAME** — Class name  

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
* **JOB_NO** — DNWORKINFO.JOB_NO  
* **CARRY_KEY** — DNCARRYINFO.CARRY_KEY  
* **SETTING_UNIT_KEY** — DNWORKINFO.SETTING_UNIT_KEY  
* **COLLECT_JOB_NO** — DNWORKINFO.COLLECT_JOB_NO  
* **JOB_TYPE** — DNWORKINFO.JOB_TYPE  
* **PLAN_UKEY** — DNWORKINFO.PLAN_UKEY  
* **STOCK_ID** — DNWORKINFO.STOCK_ID  
* **PALLET_ID** — DNPALLET.PALLET_ID  
* **PLAN_DAY** — DNWORKINFO.PLAN_DAY  
* **BATCH_NO** — DNWORKINFO.BATCH_NO  
* **PLAN_AREA_NO** — DNPALLETITZE.PLAN_AREA_NO  
* **MATERIAL_CODE** — DNWORKINFO.MATERIAL_CODE  
* **MATERIAL_NAME** — DMMASTERIALMASTER.MATERIAL_NAME  
* **STORAGE_TYPE** — DNWORKINFO.STORAGE_TYPE  
* **STORAGE_DATE** — DNSTORAGEPLAN.PLAN_DAY  
* **PLAN_QTY** — DNWORKINFO.PLAN_QTY  
* **STOCK_QTY** — DNWORKINFO.STOCK_QTY  
* **USER_ID** — DNWORKINFO.USER_ID  
* **USER_NAME** — DCUSER.USER_NAME  
* **REGIST_DATE** — SYSTIMESTAMP  
* **REGIST_PNAME** — ClassName  
* **LAST_UPDATE_DATE** — SYSTIMESTAMP  
* **LAST_UPDATE_PNAME** — ClassName  


#<span style="color:skyblue; font-weight:bold">[Send Report as Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)</span>

#User Story
- [DFD Manual Storage to Ambient](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5785)
- [DFD Manual Storage to Tempering](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5786)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)