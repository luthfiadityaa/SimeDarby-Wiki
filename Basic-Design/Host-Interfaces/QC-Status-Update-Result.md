[[_TOC_]]

# QCStatusUpdateReportData
This is the module to send production storage data result from WareNavi.
SAP will send back the production storage data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[HostCommExecutor] -->|Convert TXT/CSV -> XML| B[FileExchangeConverter]
    B -->|GET XML| C[(FTP Folder)]
    C -->|Send XML via SFTP| D[SAP]
    D -->|Send Back Response| C
    C -->|Convert XML → TXT/CSV| B
    B -->|Return TXT/CSV Response via SFTP| A
    E[(DNStock)] <--> |Update| A
    F[(DNHostSend)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendQCStatusUpdateReportData()<br>→ QCStatusUpdateReportData"]
        C1 --> C2
    end
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **MSG_ID**: Sequence Object
* **MSG_TYPE**: 2:Pallet Update 
* **ERROR_INDICATION**: 0:Successful
* **TYPE**: S: Success
* **MESSAGE_DESC**: Free Text -> **The QC Status has been Updated.**    

The data will be selected as a result:

* **ITEM_CODE** ⇄ **MATERIAL CODE**
* **RESULT_AREA_NO** ⇄ **PLANT** 
* **BATCH_NO** ⇄ **BATCH**
* **RESULT_QTY** ⇄ **QUANTITY**
* **UOM** ⇄ **UOM**
* **LAST_UPDATE_DATE** ⇄ **POSTING DATE**
* **STOCK_STATUS_TO** ⇄ **STATUSTO**
* **STORAGE_LOCATION_FROM** ⇄ **STORAGE LOCATION FROM**
* **STORAGE_LOCATION_TO** ⇄ **STORAGE LOCATION TO**

#Result Data from WareNavi
##TXT/CSV Format
```csv
1469,0,3100006023,9908,PS023128,64,CTN,2025-04-10,QI,UU,FGW2
```

##XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<PalletUpdate>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <MaterialCode>3100006023</MaterialCode>
    <Plant>9908</Plant>
    <Batch>PS023128</Batch>
    <Quantity>64</Quantity>
    <UoM>CTN</UoM>
    <PostingDate>2025-04-10</PostingDate>
    <StatusFrom>QI</StatusFrom>
    <StatusTo>UU</StatusTo>
    <StorageLocationTo>FGW2</StorageLocationTo>
</PalletUpdate>
```

# Related DFD
  - [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)
