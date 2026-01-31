[[_TOC_]]

# ProductionStorageReportData
This is the module to send production storage data result from WareNavi.
SAP will send back the production storage data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[HostCommExecutor] --> |SEND XML| C[(FTP Folder)]
    C -->|GET XML via SFTP| D[SAP]
    D -->|Send Back Response| C
    C -->|Return XML Response via SFTP| A
    E[(DNStoragePlan)] <--> |Update| A
    F[(DNHostSend)] <--> |Update| A
    G[(DNWorkInfo)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendProductionStorageReportData()<br>→ ProductionStorageReportData"]
        C1 --> C2
    end
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **MSG_ID**: Sequence Object
* **MSG_TYPE**: 2:Pallet Update 
* **ERROR_INDICATION**: 0:Successful
* **TYPE**: S: Success
* **MESSAGE_DESC**: Free Text -> **The production process has been finished.**    

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
* **BCR_DATA** ⇄ **PALLET ID**

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
    <StorageLocationFrom>VT01</StorageLocationFrom>
    <StorageLocationTo>FGW2</StorageLocationTo>
    <PalletID>PLY0001</PalletID>
    <StartDateTime>2025-04-10-08:00:00</StartDateTime> 
    <EndDateTime>2025-04-13-08:00:00<EndDateTime>
    <Remarks>process complete</Remarks>
</PalletUpdate>
```

###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
* **REPORT_FLAG**: 1:Reported
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Classname

# Related DFD
- [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)