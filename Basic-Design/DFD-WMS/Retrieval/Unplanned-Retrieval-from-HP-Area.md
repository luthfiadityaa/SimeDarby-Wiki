#<span style="color:skyblue; font-weight:bold">Retrieval from 1301, 1302, 1205, 1206, 1207, 1208, 1209</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 9002 , 9003, 9004, 9005, 9006, 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1205, 1206, 1207, 1208, 1209, 1301, 1302] 
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
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

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|
| **Only For 1301 & 1302**                                                        |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |
| **Directly For 1205,1206,1207,1208,1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Inquiry Retrieval Setting - Set(F2) [(1)](#Inquiry-Retrieval-Setting---Set(F2)) | I  |    | S  | I  | S  |    |     | S  | S  |    | S  |    | S  |    |
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |  
| ID32[(3)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    
| ID33[(4)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     
| ID64[(5)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |
| ID68[(6)](#ID68)                                                                |    |    |    |    |    |    |     |    |    |    |    |    |    | I  |
| ID26[(7)](#ID26)                                                                | U  | I  | D  | D  | D  | I  |  I  |    | U  | I  |    |    |    |    |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |   |    | U  |    |    |    |    |    |    | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
**<span style="color:green">Only For 1301 & 1302.</span>** If not, start from [Inquiry Retrieval Setting - Set(F2)](#Inquiry-Retrieval-Setting---Set(F2))

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

#Inquiry Retrieval Setting - Set(F2)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.InquiryRetrievalSettingSCH &nbsp;</span>

 ![==image_0==.jpg](/.attachments/==image_0==-58f99576-be7b-41a4-bc02-6cef92ce5b43.jpg) 

The Inquiry Retrieval Setting screen uses for manually set the retrieval work(UnitRetrieval).​ This screen uses when there are some troubles with Host System Linkage(Irregular Cases) or when manual operation is necessary.

#Retrieval-Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.transmission.RetrievalSender&nbsp;</span>

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

retrievalsender-input-->retrievalsender-.U.->retrievalsender-update
retrievalsender-->id12msg
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1301, 1302, 1205, 1206, 1207, 1208, 1209 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

## DNCARRYINFO
- CMD_STATUS = 2:Waiting for response
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNPALLET
- STATUS_FLAG = 4:Being retrieved
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID32
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id32Process&nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process-.U.->id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

## DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process&nbsp;</span>

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
id33process-.U.->id33-update
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

## DMSHELF
- STATUS_FLAG = 0:Empty
- LAST_UPDATE_DATE = SYSTIMESTAMP

## DNCARRYINFO
- CMD_STATUS = 5:Retrieval completed
- RETRIEVAL_STATION_NO = DMSHELF.STATION_NO
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID64
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process &nbsp;</span>

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

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id68Process &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-->id68process
id68process--I-->id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

## DNOPERATIONDISPLAY
- CARRY_KEY = MC Key information from ID68
- STATION_NO = Station information from ID68
- ARRIAL_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.wcs.mc.as21.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet is arrived at station
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNCARRYINFO
")]

id26process[id26process]
retrievaloperator[RetrievalStationOperator]


buttonclicked --> id26msg
id26msg -->id26process
id26process-->retrievaloperator
retrievaloperator--I-->id26-insert
retrievaloperator-.U.->id26-update
:::

After user clicked on Completion button at Station, AGC will send ID26 to WareNavi and WareNavi will execute the receive task based on information in received ID26. While WareNavi processes ID26, WareNavi will create a Arrival record and let Storage Sender picks up the data.

## DNARRIVAL
- ARRIVAL_DATE = SYSTIMESTAMP
- STATION_NO = Arrival Station Number from ID26
- CARRY_KEY = 99999999
- BCR_DATA = Barcode information from ID26
- CONTROLINFO = Control information from ID26
- SEND_FLAG = 0:Not sent
- HEIGHT = Dimension Information from ID26
- WIDTH = Dimension Information From ID26
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

## DNCARRYINFO
- CMD_STATUS = 1:Started
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name