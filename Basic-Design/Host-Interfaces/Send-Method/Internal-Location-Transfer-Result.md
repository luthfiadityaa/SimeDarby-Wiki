[[_TOC_]]

# InternalLocTransferReportData
This is the module to send internal location transfer data result from WareNavi.
SAP will send back the internal location transfer data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[HostCommExecutor] --> |SEND XML| C[(FTP Folder)]
    C -->|GET XML via SFTP| D[SAP]
    D -->|Send Back Response| C
    C -->|Return XML Response via SFTP| A
    F[(DNHostSend)] <--> |Update| A
    G[(DNWorkInfo)] <--> |Update| A
    H[(DNStock)] <--> |Update| A

   subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendInternalLocTransferReportData()<br>→ InternalLocTransferReportData"]
        C1 --> C2
   end
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **MSG_ID**: Sequence Object
* **MSG_TYPE**: 2:Pallet Update 
* **ERROR_INDICATION**: 0:Successful
* **TYPE**: S: Success
* **MESSAGE_DESC**: Free Text -> **The Retrieval QC process has been initiated.**    

The data will be selected as a result: **Warenavi** ⇄ **SAP**
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
##XML Format
`Name File`: YYYYMMDD_HHMMSS_PalletUpdate_<BatchID>.xml

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
    <StatusTo>UU</StatusTo>
    <StorageLocationFrom>FGW2</StorageLocationFrom>
    <StorageLocationTo>FGW1</StorageLocationTo>
    <StartDateTime>2025-04-10-08:00:00</StartDateTime>
    <EndDateTime>2025-04-13-08:00:00</EndDateTime>
    <Remark>process complete</Remark>
</PalletUpdate>
```

# Related DFD
  - [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)
