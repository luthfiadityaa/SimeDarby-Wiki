#<span style="color:skyblue; font-weight:bold">Summary Flow</span>

**Stage 1**
::: mermaid
flowchart LR

PST[If Retrieve To Station 1301-1303]-->PMode[ID63]  
:::

**Stage 2**
::: mermaid
flowchart LR

P1[FROM AISLE STATION - 9001, 9002 , 9003, 9004, 9005, 9006, 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->Cond1{Flow?} 
:::

**Stage 3**
::: mermaid
flowchart LR

Cond1{Flow?} 

Cond1{Flow?}--> |Flow 1: Thorugh crane 7-10 only for 1303| P5[ID64]--> |7207-7210| P8[ID26]-->P9[Retrieval Sender]-->P10[ID25]--> |SRM| P11[ID64]--> |STV| P12[ID64]--> |1303| P7[ID68]
Cond1{Flow?}--> |Flow 2: Not through crane 7-10| P6[ID64]--> |1205-1209 and 1301-1302| P7[ID68]
Cond1{Flow?}--> |Flow 3: for 1201-1204 directly to this process| P7[ID68]-->P13[ID26]
:::


#<span style="color:skyblue; font-weight:bold">Specific Location Retrieval Setting database flow</span>
**Abbreviation:**
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
- **TTSN** : DMTOSTATION 
- **INOUT**: DNINOUTSTATION

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD|INOUT| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|-----|
| **Only For 1301, 1302 & 1303**                                                  |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |     |
| **Directly For 1201-1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| Specific Location Retrieval Setting - Set(F2) [(1)](#Specific-Location-Retrieval-Setting---Set(F2)) | I  |    | S  | I  | S  |    |     | S  | S  |    | S  | S  | S   |    |    |
| Specific Location Retrieval Setting - Cancel Entries(F11) [(2)](#Specific-Location-Retrieval-Setting---Cancel-Entries(F11)) | D   |    |   | D   |   |    |     |    |    |    |    |    |    |    |    |
| **Normal Flow**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |    |  
| ID32[(3)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    |    
| ID33[(4)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |    |     
| **Flow 1: Thorugh crane 7-10 only for 1303**                                    |    |    |    |    |    |    |     |    |    |    |    |    |    |    |    |
| ID64[(5)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    |
| ID26[(6)]                                                                            |    |    | U   | U   |    |    | I    |    |    |    |    |    |    |    |    |
| Retrieval Sender[(7)]                                                                |    |    |    | U   |    |    | U    |    |    |    |    |    |    |    |    |
| ID25[(8)]                                                                            |    |    |    | U   |    |    | D    |    |    |    |    |    |    |    |    |
| ID64[(9)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    |
| ID64[(10)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    |
| **Flow 2: Not through crane 7-10 for 1205-1209 and 1301-1302**                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |    |
| ID64[(5)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    |
| **Flow 3 / Last Process: for 1201-1204 directly to this process**                                                  |    |    |    |    |    |    |     |    |    |    |    |    |    |    |    |
| ID68[(6)](#ID68)                                                                | S  |    |    | S  |    |    |     |    |    |    |    |    |    | I  |    |
| ID26[(7)](#ID26)                                                                | U  | I  | D  | D  | D  | I  |     |    | U  | I  |    |    |    |    | I  | 
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |   |    | U  |    |    |    |    |    |    | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id63Process&nbsp;</span>

If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
**<span style="color:green">Only For 1301, 1302, 1303.</span>** If not, start from [Specific Location Retrieval Setting - Set(F2)](#Specific-Location-Retrieval-Setting---Set(F2))

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
####<span style="color:skyblue; font-weight:bold">DMStation</span>
**CURRENT_MODE**: 2: Retrieval Mode

#Specific Location Retrieval Setting - Set(F2)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.SpecificLocationRetrievalSettingSCH &nbsp;</span>

 ![==image_0==.jpg](/.attachments/==image_0==-0f7ad1cc-fe5d-4f53-9b15-1c9fecf90cd0.jpg) 
  
The Specific Location Retrieval Setting screen uses for manually set the retrieval work through specific location(Unit Retrieval or Only Confirmation).
This screen mainly uses when there are some troubles with Host System Linkage(irregular cases) or when manual operation is necessary. Especially for retrieving Error Stock.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Unplanned Retrieval
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Selected Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- Selected Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

::: mermaid
flowchart LR
    input[
        Retrieval Material
        Station
        Pallet #
        SoftZone
        Priority
        To Location
        All Qty
        Retrieval Qty
        Stock Qty
        Material Code
        Material Name
        Material Type
        Batch #
        Area
        Loc #
        Stock Status
        Storage Date/Time
        Qty kg/crtn
        Qty crtn/PL
        UOM
        Tempering Flag
        Tempering Period
        QC Duration
        QC Check Flag
        Truck No
        Plate No
        Retrieval Type
    ]

    tableList-insert[("
        DNWORKINFO
        DNCARRYINFO
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
        DMTOSTATION
    ")]

    className[InquiryRetrievalSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 23:Unplanned Retrieval    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNSTOCK.LOCATION_NO
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: DNSTOCK.BATCH_NO    
*   **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE 
*   **PLAN_QTY**: Value from screen (Retrieval Qty / Stock Qty)    
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNPALLET.BCR_DATA  
*   **STORAGE_LOCATION_TO**: Value from Screen (To Location)
*   **DOCK_NO**: Value from Screen (Dock No)
*   **TRUCK_PLATE_NO**: Value from Screen (Truck Plate No)
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNSTOCK.PALLET_ID    
*   **WORK_TYPE**: 23:Unplanned Retrieval    
*   **CMD_STATUS**: 1:Started    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location  
*   **WORK_NO**: Sequence Object    
*   **RETRIEVAL_STATION_NO**: DNSTOCK.LOCATION_NO
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO    
*   **DEST_STATION_NO**: Value from screen (Station)     
*   **PRIORITY**: Value from screen ⟶ **<span style="color:green;">(1:Urgent, 2:Normal)</span>**
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **CARRY_FLAG**: 2:Retrieval
*   **CANCEL_REQUEST**: 0:Not requested
*   **AISLE_STATION_NO**: DMSHELF.PARENT_STATION_NO
*   **END_STATION_NO**: DNCARRYINFO.DEST_STATION_NO  
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

#Inquiry Retrieval Setting - Cancel Entries(F11)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.InquiryRetrievalSettingSCH &nbsp;</span>

::: mermaid
flowchart LR
     tableList-delete[("
        DNWORKINFO
        DNCARRYINFO
    ")]

     tableList-select[("
        DNWORKINFO
        DNCARRYINFO
    ")]

    className[InquiryRetrievalSettingSCH]

    tableList-select --> |SELECT| className --> |DELETE| tableList-delete

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

To cancel entries, the InquiryRetrievalSettingSCH component first reads data from the **<span style="color:green">DNWORKINFO and DNCARRYINFO</span>** tables, then deletes those records after processing. If not, continue process to [Retrieval-Sender](#Retrieval-Sender)

#Retrieval-Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalSender&nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
")]

retrievalsender-update[("
DNCARRYINFO
DNPALLET
")]

id12msg("
ID 12
")

retrievalsender-input-->retrievalsender--> |UPDATE| retrievalsender-update
retrievalsender-->id12msg
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1301, 1302, 1205, 1206, 1207, 1208, 1209 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNPALLET</span>
* **STATUS_FLAG**: 4:Being retrieved
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID32
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id32Process&nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process--> |UPDATE| id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 3:Commanded
* **ERROR_CODE**: 0
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id33Process&nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-update[("
DMSHELF
DNCARRYINFO
")]

id33-->id33process
id33process--> |UPDATE| id33-update
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DMSHELF</span>
* **STATUS_FLAG**: 0:Empty
* **LAST_UPDATE_DATE**: SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 5:Retrieval completed
* **RETRIEVAL_STATION_NO**: DMSHELF.STATION_NO
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ###[Flow 1 : through crane 7-10](#)
- ###[Flow 2 : not through crane 7-10](#)
- ###[Flow 3 / Last Process: Only for 1201-1204 Directly to this process](#)

<hr>

#Flow 1 : Through crane 7-10 
Refer to AGC Linkage Specs: [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Retrieval Section - 14 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

#ID64
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

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

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID26 at

#Retrieval Sender at

#ID25 at

#ID64 at
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

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

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID64 at 
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

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

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

#Flow 2 : not through crane 7-10 
Refer to AGC Linkage Specs: [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Retrieval Section - 14 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

#ID64
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

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

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name


<hr>

- ###[Flow 3 / Last Process: Only for 1201-1204 Directly to this process](#)

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id68Process &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-select[("
DNCARRYINFO
DNWORKINFO
")]

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-select-->id68
id68-->id68process
id68process--> |INSERT| id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNOPERATIONDISPLAY</span>
* **CARRY_KEY**: MC Key information from ID68
* **STATION_NO**: Station information from ID68
* **MATERIAL_CODE**: **<span style="color:green">DNWORKINFO.MATERIAL_CODE</span>**
* **RETRIEVAL_QTY**: **<span style="color:green">DNWORKINFO.RESULT_QTY</span>**
* **DOCK_NO**: **<span style="color:green">DNWORKINFO.DOCK_NO</span>**
* **TRUCK_PLATE_NO**: **<span style="color:green">DNWORKINFO.TRUCK_PLATE_NO</span>**
* **ARRIAL_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##<span style="color:skyblue; font-weight:bold">LED sign displays the work</span>  
[Display information]
* **Job Type**
* **Material Code**
* **Qty**
* **Dock No**
* **Truck Plate No**

![image.png](/.attachments/image-1e56dc6e-657e-47ff-9a47-575b04705583.png)

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet is arrived at station
"]

id26msg("
ID 26
")

id26-update[("
DNWORKINFO
DNSHELF
")]

id26-delete[("
DNCARRYINFO
DNPALLET
DNSTOCK
")]

id26-insert[("
DNWORKLIST
DNHOSTSEND
DNINOUTRESULT
")]

id26process[id26process]
retrievaloperator[RetrievalStationOperator]


buttonclicked --> id26msg
id26msg -->id26process
id26process-->retrievaloperator

retrievaloperator--> |INSERT| id26-insert
retrievaloperator--> |UPDATE| id26-update
retrievaloperator--> |DELETE| id26-delete
:::

<span style="color:black; font-weight:bold; color:red">*The operator performs the operation according to the work display on the work terminal.</span>

After the completion button flashes, the operator removes the pallet and presses the completion button to clear the operation indication. At the same time, sending ID 26 to the id26process, then delete related records from DNPALLET, DNCARRYINFO, and DNSTOCK.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **STATUS_FLAG**: 4:Completed
*   **SHORTAGE_QTY**: DNWORKINFO.PLAN_QTY - DNWORKINFO.RESULT_QTY
*   **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
*   **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **USER_ID**: Login info
*   **TERMINAL_NO**: Login info
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME    
*   **RETRIEVAL_STATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **RETRIEVAL_DETAIL**: 1:Unit Retrieval   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY      
*   **DOCK_NO**: DNWORKINFO.DOCK_NO
*   **TRUCK_PLATE_NO**: DNWORKINFO.TRUCK_PLATE_NO 
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName


###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **STATUS_FLAG**: DNWORKINFO.STATUS_FLAG    
*   **HARDWARE_TYPE**: DNWORKINFO.HARDWARE_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **SYSTEM_CONN_KEY**: DNWORKINFO.SYSTEM_CONN_KEY    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **PLAN_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **UOM**: MMATERIALMASTER.UOM    
*   **ENTERING_QTY**: DNSTOCK.STOCK_QTY      
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **RESULT_QTY**: DNWORKINFO.RESULT_QTY    
*   **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY    
*   **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO    
*   **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **DOCK_NO**: **DNWORKINFO.DOCK_NO**
*   **TRUCK_PLATE_NO**: **DNWORKINFO.TRUCK_PLATE_NO**   
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **TERMINAL_NO**: DNWORKINFO.TERMINAL_NO    
*   **WORK_SECOND**: DNWORKINFO.WORK_SECOND    
*   **USER_NAME**: DCUSER.USERNAME    
*   **REPORT_FLAG**: 0: Not Reported    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: Class name    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
*   **RESULT_KIND**: 2:Retrieval(Stock-)
*   **STATION_NO**: = DNCARRYINFO.DEST_STATION_NO
*   **LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
*   **WH_STATION_NO**: DNPALLET.WH_STATION_NO
*   **AISLE_STATION_NO**: DNCARRYINFO.AISLE_STATION_NO
*   **WORK_TYPE**: DNCARRYINFO.WORK_TYPE
*   **RETRIEVAL_DETAIL**: DNCARRYINFO.RETRIEVAL_DETAIL
*   **WORK_NO**: DNCARRYINFO.WORK_NO
*   **PALLET_ID**: DNPALLET.PALLET_ID
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY
*   **RESTORING_FLAG**: DNCARRYINFO.RESTORING_FLAG
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **REMOVE_FLAG**: 00:Normal
*   **REGIST_PNAME**: Class name
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

