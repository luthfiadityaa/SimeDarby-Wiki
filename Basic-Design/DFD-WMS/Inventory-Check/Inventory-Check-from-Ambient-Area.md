[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">Inventory Check from Ambient Area</span>
![image.png](/.attachments/image-da2c69da-502e-434b-8de2-eafe8b266f4f.png)

# <span style="color:skyblue; font-weight:bold">Summary Flow</span>
### Stage 1 - Retrieval Station
::: mermaid
flowchart LR
  
PST[To Station 1301,1302]-->|Mode Change| P2[ID63]
:::


### Stage 1 - Retrieval Operation
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[ID12]-->P3[RetrievalSender]-->P4[ID32]-->P5[ID33]-->P6[ID64]-->P7[ID68]-->P8[To STATION - 1301, 1302]
:::

### Stage 2 - Restorage Operation

::: mermaid
flowchart LR
  
P1[Work Display - Click Complete Button]-->P2[ID45]-->P3[ID26]-->P4[StorageSender]-->P5[ID25]-->P6[ID64]-->P7[ID26]-->P8[StorageSender]-->P9[ID25]-->P10[ID64]-->P11[ID33]
:::

## Abbreviation
- **PLLT** : DNPALLET          
- **WRKI** : DNWORKINFO        
- **WRKL** : DNWORKLIST        
- **CRYI** : DNCARRYINFO       
- **STCK** : DNSTOCK           
- **ARVL** : DNARRIVAL        
- **WRHS** : DMWAREHOUSE      
- **SHLF** : DMSHELF          
- **STCH** : DNSTOCKHISTORY   
- **INOT** : DNINOUTRESULT    
- **HSTS** : DNHOSTSEND       
- **OPRR** : DNOPERATIONRESULT
- **ITEM** : DMITEM           
- **STSN** : DMSTATION        
- **INVC** : DNINVENTORYCHECK
- **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT

| Action Name                                                    | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | HSTS | OPRR | ITEM | STSN | INVC |
|----------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| **Mode Change For 1301-1302**                                  |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID63 [(1)](#id33)                                              |      |      |      |      |      |      |      |      |      |      |      |      |      |   U  |      |
| Inquiry Retrieval - Set(F2) [(2)](#inventory-check---set(f2))  |   S  |   I  |   I  |      |      |      |   S  |   S  |      |      |      |      |   S  |   S  |   I  |
| **Retrieval Flow**                                             |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| Retrieval Sender [(3)](#retrieval-sender)                      |   U  |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID12 [(4)](#id12)                                              |   U  |      |      |   I  |      |      |      |      |      |      |      |      |      |      |      |
| ID32 [(5)](#id32)                                              |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID33 [(6)](#id33)                                              |      |      |      |   U  |      |      |      |  U   |      |      |      |      |      |      |      |
| ID64 [(7)](#id64)                                              |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID68 [(8)](#id68)                                              |      |      |      |      |      |      |      |      |      |      |      |   I  |      |      |      |
| **Re-Storage Flow**                                            |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID45 [(9)](#id45)                                              |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID26 at 1301-1302 [(10)](#id26-at-1301-1302)                   |   U  |   U  |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| StorageSender at 1301-1302 [(11)](#StorageSender-at-1301-1302) |      |   U  |      |   U  |      |   U  |   U  |   U  |      |      |      |      |      |      |      |
| ID25 at 1301-1302 [(12)](#ID25-at-1301-1302)                   |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |      |      |
| ID64 at STV [(13)](#id64-at-STV)                               |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID26 at 7207-7214 [(14)](#id26-at-7207-7214)                   |   U  |   U  |      |   U  |      |   I  |      |      |      |      |      |      |      |      |      |
| StorageSender at 7207-7214 [(15)](#StorageSender-at-7207-7214) |      |   U  |      |   U  |      |   U  |   U  |   U  |      |      |      |      |      |      |      |
| ID25 at 7207-7214 [(16)](#ID25-at-7207-7214)                   |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |      |      |
| ID64 at SRM [(17)](#id64-at-SRM)                               |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |
| ID33 at 9007-9014 [(18)](#id33-at-9007-9014)                   |   U  |   U  |      |   D  |   U  |      |      |  U   |   I  |      |      |      |      |      |   U  |

# <span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id63Process` &nbsp;</span>

If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
**Only For 1301, 1302**.

::: mermaid
flowchart LR
    input[
        Operators pressed the retrieval mode button on the operation box.      
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

## <span style="color:skyblue; font-weight:bold">DMSTATION</span>
- **CURRENT_MODE** : 2: Retrieval Mode

# Inventory Check - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.inventorycheck.InventoryCheckSCH` &nbsp;</span>

![image.png](/.attachments/image-1d7c83f5-0bc5-41ad-ab76-6b162f697426.png)
  
The Inventory Check Setting screen uses for manually set the inventory check work (**always Partial Pick, and retrieval qty = 0**).

::: mermaid
flowchart LR
    input[
        Stock Qty
        Material Code
        Material Name
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
    ]

    tableList-insert[("
        DNINVENTORYCHECK
        DNWORKINFO
        DNWORKLIST
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]

    className[InventoryCheckSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className
:::

## <span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole process Inventory Check
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

## <span style="color:skyblue; font-weight:bold">DNINVENTORYCHECK</span>
- **SCHEDULE_NO** : Sequence object
- **FROM_LOCATION** : Value from screen (Start Location No.)
- **TO_LOCATION** : Value from screen (End Location No.)
- **WH_STATION_NO** : Value from screen (Area No.)
- **STATION_NO** : Value from screen (Station No.)
- **INVENTORY_ORDER** : 1: By Location Number
- **INVENTORY_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **STATUS_FLAG** : 1:Inventory Check in Progress
- **REGIST_DATE** : SYSTIMESTAMP
- **REGIST_PNAME** : Class name
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>  
- **JOB_TYPE** :  40: Inventory Check      
- **STATUS_FLAG** :  1: Working
- **HARDWARE_TYPE** : 3: ASRS    
- **PLAN_UKEY** :  Sequence Object    
- **STOCK_ID** :  DNSTOCK.STOCK_ID    
- **PLAN_AREA_NO** :  DNSTOCK.AREA_NO   
- **PLAN_LOCATION_NO** :  DNSTOCK.LOCATION_NO
- **PLAN_DAY** :  DMWARENAVISYSTEM.WORK_DAY
- **VENDOR_CODE** : DNSTOCK.VENDOR_CODE
- **VENDOR_NAME** : DNSTOCK.VENDOR_NAME
- **COMPANY_CODE** : DNSTOCK.COMPANY_CODE     
- **BATCH_NO** :  Value from screen (Batch #)    
- **MATERIAL_CODE** :  Value from screen (Material Code) 
- **PLAN_LOT_NO** : DNSTOCK.LOT_NO 
- **PLAN_QTY** :  0   
- **QTY_KG_CRTN** :  Value from screen (Qty kg/crtn)    
- **QTY_CRTN_PL** :  Value from screen (Qty crtn/PL)    
- **UOM** :  Value from screen (UOM)    
- **TEMPERING_PERIOD** :  Value from screen (Tempering Period)    
- **EXPIRY_DAYS** :  DNSTOCK.EXPIRYDATE    
- **WORK_DAY** :  DMWARENAVISYSTEM.WORK_DAY  
- **BCR_DATA** :  DNPALLET.BCR_DATA  
- **DOCK_NO** :  DNWORKINFO.DOCK_NO
- **TRUCK_PLATE_NO** :  DNWORKINFO.TRUCK_PLATE_NO
- **USER_ID** :  Login Info     
- **TERMINAL_NO** :  Login info
- **REGIST_DATE** :  SYSTIMESTAMP    
- **REGIST_PNAME** :  ClassName    
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** :  ClassName

## <span style="color:skyblue; font-weight:bold">DNWORKLIST</span>
- **JOB_TYPE** : DNWORKINFO.JOB_TYPE
- **PLAN_UKEY** : DNWORKINFO.PLAN_UKEY
- **STOCK_ID** : DNWORKINFO.STOCK_ID
- **ITEM_CODE** : DNWORKINFO.ITEM_CODE
- **PLAN_LOT_NO** : DNWORKINFO.PLAN_LOT_NO
- **PLAN_QTY** : DNWORKINFO.PLAN_QTY
- **PLAN_DAY** : DNWORKINFO.PLAN_DAY
- **PALLET_ID** : DNCARRYINFO.PALLET_ID
- **PLAN_DAY** : DNWORKINFO.PLAN_DAY
- **COMPANY_CODE** : DNWORKINFO.COMPANY_CODE
- **BATCH_NO** : DNWORKINDO.BATCH_NO
- **PLAN_AREA_NO** : DNWORKINFO.PLAN_AREA_NO
- **PLAN_LOCATION_NO** : DNWORKINFO.PLAN_LOCATION_NO
- **MATERIAL_CODE** : DNWORKINFO.MATERIAL_CODE
- **MATERIAL_NAME** : DMITEM.MATERIAL_NAME
- **ALLOCATION_QTY** : 1
- **RETRIEVAL_STATION_NO** : DNCARRYINFO.RETRIEVAL_STATION_NO
- **RETRIEVAL_DETAIL** : DNCARRYINFO.RETRIEVAL_DETAIL
- **WORK_NO** : DNCARRYINFO.WORK_NO
- **SOURCE_STATION_NO** : DNCARRYINFO.SOURCE_STATION_NO
- **DEST_STATION_NO** : DNCARRYINFO.DEST_STATION_NO
- **SCHEDULE_NO** : DNCARRYINFO.SCHEDULE_NO, DNINVENTORYCHECK.SCHEDULE_NO
- **END_STATION_NO** : DNCARRYINFO.END_STATION_NO
- **USER_ID** : Login Info
- **USER_NAME** : Login Info
- **TERMINAL_NO** : Login Terminal
- **REGIST_DATE** : SYSTIMESTAMP                                                    
- **REGIST_PNAME** : ClassName
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

# Retrieval Sender

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.transmission.RetrievalSender` &nbsp;</span>

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

retrievalsender--SEND-->id12msg
retrievalsender-input-->retrievalsender-.UPDATE.->retrievalsender-update
:::

For Inventory Check performed in ASRS where there is creation of DNCARRYINFO, DNCARRYNFO data will be processed in Retrieval Sender. ID 12 will be sent after related tables are updated successfully.

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 3:Reserved for retrieval
- **ALLOCATION_FLAG** : 1:Allocated
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CARRY_KEY** : Sequence Object    
- **PALLET_ID** : DNSTOCK.PALLET_ID    
- **WORK_TYPE** : 40: Inventory Check    
- **CMD_STATUS** : 1: Started    
- **PRIORITY** : 2: Normal    
- **RESTORING_FLAG** : 1: Return to Same Location  
- **WORK_NO** : Sequence Object
- **CARRY_FLAG** : 2: Retrieval    
- **RETRIEVAL_STATION_NO** : DNSTOCK.LOCATION_NO
- **RETRIEVAL_DETAIL** : 0: Inventory Check
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO    
- **DEST_STATION_NO** : Based on **SOURCE_STATION_NO** where a reserved location belongs to ⟶ **(1301, 1302)**
- **PRIORITY** : 2: Normal
- **CANCEL_REQUEST** : 0: Not Requested    
- **SCHEDULE_NO** : Sequence Object  
- **CANCEL_REQUEST** : 0: Not requested
- **AISLE_STATION_NO** : DMSHELF.PARENT_STATION_NO
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO  
- **REGIST_DATE** : SYSTIMESTAMP    
- **REGIST_PNAME** : ClassName    
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : ClassName

#ID12

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.send.As21Id12` &nbsp;</span>

::: mermaid
flowchart LR

id26msg("
ID12
")

id26-insert[("
DNCARRYINFO
")]

id26-update[("
DNPALLET
")]

InOutStationOperation[InOutStationOperation]

id26msg-->id12process
id12process-->InOutStationOperation--> |INSERT| id26-insert
InOutStationOperation--> |UPDATE| id26-update
:::

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 2:Waiting for response
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 4:Being retrieved
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID32

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id32Proces` &nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process-.UPDATE.->id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 3:Commanded
- **ERROR_CODE** : 0
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process` &nbsp;</span>

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

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet is out of rack and is being transferred to related Station.

## <span style="color:skyblue; font-weight:bold">DMSHELF</span>
- **STATUS_FLAG**: 2:Reserved Location
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 5:Retrieval completed
- **RETRIEVAL_STATION_NO** : DMSHELF.STATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID64

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id68Process` &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-->id68process
id68process--> |INSERT| id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived at related Station. Upon receiving of ID68, insertion of data will be executed for Stations 1301 or 1302.

## <span style="color:skyblue; font-weight:bold">DNOPERATIONDISPLAY</span>
- **CARRY_KEY** : MC Key information from ID68
- **STATION_NO** : Station information from ID68
- **MATERIAL_CODE** : DNWORKINFO.MATERIAL_CODE
- **RETRIEVAL_QTY** : DNWORKINFO.STOCK_QTY
- **DOCK_NO** : DNWORKINFO.DOCK_NO
- **TRUCK_PLATE_NO** : DNWORKINFO.TRUCK_PLATE_NO
- **ARRIVAL_DATE** : SYSTIMESTAMP
- **REGIST_PNAME** : Class name
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

##LED DISPLAY
[Display information]
- **Job Type**
- **Material Code**
- **Qty**
- **Dock Type**
- **Truck_Plate_no**

![image.png](/.attachments/image-1e56dc6e-657e-47ff-9a47-575b04705583.png)

# Work Display

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.inquiry.workdisplay.WorkDisplaySCH` &nbsp;</span>

![image.png](/.attachments/image-c2e79f90-311f-492c-a016-10421c8fe162.png)

::: mermaid
flowchart LR

input("
Complete button
")

id45msg("
ID 45
")

workdisplay-update[("
DNSTOCK
DNPALLET
DNCARRYINFO
DNWORKINFO
")]

workdisplay-delete[("
DNOPERATIONDISPLAY
")]

workdisplay[WorkDisplaySCH]

input-->workdisplay
workdisplay-.UPDATE.->workdisplay-update
workdisplay-.DELETE.->workdisplay-delete
workdisplay--SEND-->id45msg
:::

Upon Pallet arrival at Station 1301-1302, user can use Work Display Screen to verify the Material, Stock Qty and Others detail information then can complete the Inventory Check process also re-storage the pallet.

After user successful click  **Complete** button, ID45 will be sent along from WareNavi where Completion button at related Stations in Unit Load Control Box will start to blink. After pressing the Completion button, pallet will return back to ASRS.

## <span style="color:skyblue; font-weight:bold">DNSTOCK</span>
- **STOCK_QTY** : Value from screen (Primary UOM Count Qty)
- **ALLOCATION_QTY** : Value from screen (Primary UOM Count Qty)
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 1:Reserved for Storage
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 6:Arrival
- **CARRY_FLAG** : 1:Storage
- **SOURCE_STATION_NO** : 2111-2113
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
- **STATUS_FLAG** : 4:Completed
- **RESULT_AREA_NO** : DNWORKINFO.PLAN_AREA_NO
- **RESULT_LOCATION_NO** : DNWORKINFO.PLAN_LOCATION_NO
- **RESULT_LOT_NO** : DNWORKINFO.PLAN_LOT_NO
- **RESULT_QTY** : 0
- **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **USER_ID** : Login info
- **TERMINAL_NO** : Login info
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID45
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.id.send.As21Id45` &nbsp;</span>
::: mermaid
flowchart LR

id45msg("
ID 45
")

buttonlight["
Station Completion button
Light Up
"]

id45msg--> As21Id45
As21Id45 --> buttonlight
:::

Sending of ID45 is sent to AGC when user clicked on **Complete** at Work Display where the Completion button at Control Box will start blinking. If user confirmed re-storing of pallet is safe to proceed, user can click on the blinking Completion button to proceed with transporting of pallet to ASRS.

# ID26 at 1301-1302

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

::: mermaid
flowchart LR

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

buttonclicked["
Station Completion 
button clicked
"]

id26process--> |INSERT| id26-insert
buttonclicked-->id26msg-->id26process--> |UPDATE| id26-update
:::

After user clicked on Completion button at Station in Unit Load, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26.

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
- **ARRIVAL_DATE** :  SYSTIMESTAMP    
- **STATION_NO** :  Arrival Station Number from ID26    
- **CARRY_KEY** :  DNCARRYINFO.CARRY_KEY    
- **BCR_DATA** :  Barcode information from ID26    
- **CONTROLINFO** :  Control information from ID26    
- **SEND_FLAG** :  0: Not sent    
- **HEIGHT** :  Dimension Information from ID26    
- **WIDTH** :  Dimension Information From ID26    
- **REGIST_DATE** :  SYSTIMESTAMP    
- **REGIST_PNAME** :  ClassName    
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** :  ClassName

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **CURRENT_STATION_NO** : Station Number from **ID26**
- **STATUS_FLAG** : 1: Reserved for Storage     
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **WORK_TYPE** : 2: Storage    
- **CMD_STATUS** : 6: Arrival  
- **RESTORING_FLAG** : 1: Return to Same Location  
- **CARRY_FLAG** : 1: Storage 
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO   
- **DEST_STATION_NO** : Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ (**7207/7208/7209/7210/7211/7212/7213/7214**)
- **ARRIVAL_DATE** : SYSTIMESTAMP  
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : ClassName

#StorageSender at 1301-1302

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender` &nbsp;</span>

::: mermaid
flowchart LR
automaticmodechangesender-update[("
DNCARRYINFO
DMWAREHOUSE
DMSHELF
DNWORKINFO
DNARRIVAL
")]
automaticmodechangesender-input[("
DNPALLET
DNCARRYINFO
")]

id05msg("
ID 05
")

automaticmodechangesender-input-->StorageSender-->id05msg
StorageSender--> |UPDATE| automaticmodechangesender-update
:::

## <span style="color:skyblue; font-weight:bold">DMWAREHOUSE</span>
- **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to
- **LAST_USED_STATION_NO_PM** : Aisle Number where a reserved location belongs to
- **LAST_USED_STATION_NO_EP** : Aisle Number where a reserved location belongs to

## <span style="color:skyblue; font-weight:bold">DMSHELF</span>
- **STATUS_FLAG** :  2:Reserved Location    
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to    
- **CMD_STATUS** :  2:Waiting for response    
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
- **JOB_TYPE** : 2:Storage
- **STATUS_FLAG** : 4: Completed
- **RESULT_AREA_NO** : DNWORKINFO.PLAN_AREA_NO
- **RESULT_LOCATION_NO** : DNWORKINFO.PLAN_LOCATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span>  
- **SEND_FLAG** :  1: Sent    
- **LAST_UPDATE_DATE** :  SYSTIMESTAMP    
- **LAST_UPDATE_PNAME** :  Class name

# ID25 at 1301-1302

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

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
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete

:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 3:Commanded
- **ERROR_CODE** :  0 
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID64 at STV

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment **(STV)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID26 at 7207-7214

::: mermaid
flowchart LR

releaseCommand["
Continue the Process Storage
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNCARRYINFO
DNPALLET
DNWORKINFO
")]

storageStationOperator[storageStationOperator]

releaseCommand-->id26msg-->id26process-->storageStationOperator
storageStationOperator--> |INSERT| id26-insert
storageStationOperator--> |UPDATE| id26-update
:::

Continue the process **storage**, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Automatic Mode Change Sender picks up the data.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id26` &nbsp;</span>

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
- **ARRIVAL_DATE** : SYSTIMESTAMP 
- **STATION_NO** : Arrival Station Number from ID26 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY       
- **BCR_DATA** : Barcode information from ID26
- **CONTROLINFO** : Control information from ID26
- **SEND_FLAG** : 0:Not sent
- **HEIGHT** : Dimension Information from ID26
- **WIDTH** : Dimension Information From ID26
- **REGIST_DATE** : SYSTIMESTAMP                                                    
- **REGIST_PNAME** : ClassName
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **PALLET_ID** : DNPALLET.PALLET_ID
- **WORK_TYPE** : 2: Storage
- **CMD_STATUS** : 1:Started 
- **PRIORITY** : 2:Normal
- **CARRY_FLAG** : 1: Storage
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO ⟶ **(7207/7208/7209/7210/7211/7212/7213/7214)**
- **DEST_STATION_NO** : Based on SOURCE_STATION_NO where a reserved location belongs to ⟶ **(9007/9008/9009/9010/9011/9012/9013/9014)**
- **END_STATION_NO** : DNCARRYINFO.DEST_STATION_NO
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>                                                     
- **CURRENT_STATION_NO** : DNARRIVAL.STATION_NO
- **WH_STATION_NO** : DNCARRYINFO.END_STATION_NO                                                                                                                                                
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

# StorageSender at 7207-7214

::: mermaid
flowchart LR
storageSender-update[("
DMWAREHOUSE
DMSHELF
DNCARRYINFO
DNWORKINFO
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

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender ` &nbsp;</span>

After successful creation of arrival record in **ID26process**, Automatic Mode Change Sender is the following process where it will send **ID05** to **AGC**. To indicate **ID05** is sent to AGC, **DNCARRYINFO.CMD_STATUS** will be updated from **1:Started to 2:Waiting for Response**.

## <span style="color:skyblue; font-weight:bold">DMWAREHOUSE</span> 
- **LAST_USED_STATION_NO** : Aisle Number where a reserved location belongs to
- **LAST_USED_STATION_NO_PM** : Aisle Number where a reserved location belongs to
- **LAST_USED_STATION_NO_EP** : Aisle Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name 

## <span style="color:skyblue; font-weight:bold">DMSHELF</span> 
- **STATUS_FLAG** : 2:Reserved Location
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **AISLE_STATION_NO** : Aisle Number where a reserved location belongs to
- **CMD_STATUS** : 2:Waiting for response
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span> 
- **JOB_TYPE** : 02:Storage
- **STATUS_FLAG** : 1: Working
- **PLAN_AREA_NO** : Area Number where a reserved location belongs to
- **PLAN_LOCATION_NO** : Location Number where a reserved location belongs to
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : ClassName

## <span style="color:skyblue; font-weight:bold">DNARRIVAL</span> 
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY
- **SEND_FLAG** : 1:Sent
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID25 at 7207-7214

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process` &nbsp;</span>

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
id25process-.UPDATE.->id25-update
id25process-.DELETE.-xid25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 3:Commanded
- **ERROR_CODE** : 0
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID64 at SRM

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process` &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.UPDATE.->id64-update
:::

Upon equipment **(SRM)** have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span> 
- **CMD_STATUS** : 4:Pickup completed
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

# ID33 at 9007-9014

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.id.recv.As21Id33` &nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNSTOCKHISTORY
")]

id33-update[("
DNSTOCK
DMSHELF
DNPALLET
DNWORKINFO
DNINVENTORYCHECK
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--INSERT-->id33-insert
id33process-.UPDATE.->id33-update
id33process--DELETE--xid33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

## <span style="color:skyblue; font-weight:bold">DNSTOCK</span>
- **NEWEST_STORAGE_DATE**: SYSTIMESTAMP
- **LOCATION_NO** : DNPALLET.CURRENT_STATION_NO
- **LAST_UPDATE_DATE**: SYSTIMESTAMP
- **LAST_UPDATE_PNAME**: Class name 

## <span style="color:skyblue; font-weight:bold">DMSHELF</span> 
- **STATUS_FLAG** : 1: Occupied
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNPALLET</span> 
- **CURRENT_STATION_NO** : Location Number information from ID33
- **STATUS_FLAG** : 2:Occupied
- **ALLOCATION_FLAG** : 0:Not allocated
- **LAST_STORED_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span> 
- **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
- **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
- **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
- **STATUS_FLAG**: 4: Completed
- **LAST_UPDATE_DATE**: SYSTIMESTAMP
- **LAST_UPDATE_PNAME**: Class name

## <span style="color:skyblue; font-weight:bold">DNINVENTORYCHECK</span> 
- **STATUS_FLAG** : 0:Inventory Check Undone
- **LAST_UPDATE_DATE** : SYSTIMESTAMP
- **LAST_UPDATE_PNAME** : Class name

## <span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY</span> 
- **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **INC_DEC_TYPE** : 1: Stock Increase
- **JOB_TYPE** : 2: Storage
- **STOCK_ID** : DNSTOCK.STOCK_ID
- **AREA_NO** : DNSTOCK.AREA_NO
- **LOCATION_NO** : DNSTOCK.LOCATION_NO
- **MATERIAL_CODE** : DNSTOCK.MATERIAL_CODE
- **MATERIAL_NAME** : DMITEM.MATERIAL_NAME
- **STORAGE_DAY** : DNSTOCK.STORAGE_DAY
- **STORAGE_DATE** : DNSTOCK.STORAGE_DATE
- **NEWEST_STORAGE_DATE** : DNSTOCK.NEWEST_STORAGE_DATE
- **UPDATE_STOCK_QTY**: DNSTOCK.STOCK_QTY
- **INC_DEC_QTY** : DNSTOCK.STOCK_QTY
- **PALLET_ID** : DNSTOCK.PALLET_ID
- **AREA_TYPE** : DMAREA_AREA_TYPE
- **USER_ID** : Login info
- **USER_NAME** : Login info
- **TERMINAL_NO** : Login info
- **TERMINAL_NAME** : Login info
- **IP_ADDRESS** : Login info
- **STOCK_STATUS** : DNSTOCK.STOCK_STATUS
- **TEMPERING_FLAG** : DNSTOCK.TEMPERING_FLAG
- **QC_FLAG** : DNSTOCK.QC_FLAG
- **EXPIRY_DATE** : DNSTOCK..EXPIRY_DAYS
- **REGIST_DATE** : SYSTIMESTAMP
- **REGIST_PNAME** : Class name


# User Story
  - #5795

# Related DFD
  - {}
