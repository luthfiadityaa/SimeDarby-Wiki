[[_TOC_]]

# StorageRetievalReportData
This is the module to send unplanned Storage/Retrieval data result from WareNavi.
SAP will send back the unplanned Storage/Retrieval data result on the response.

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
    H[(DNStock)] <--> |Update| A
    I[(DNRetrievalPlan)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendStorageRetrievalReportData()<br>→ StorageRetievalReportData"]
        C1 --> C2
    end
:::

#Result Data from WareNavi
##Unplanned Storage
###Finish Goods
####XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<PalletUpdate>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <MaterialCode>3100006023</MaterialCode>
    <Batch>PS023128</Batch>
    <Quantity>64</Quantity>
    <UoM>CTN</UoM>
    <PostingDate>2025-04-10</PostingDate>
    <StatusTo>UU</StatusTo>
    <Plant>9908</Plant>
    <StorageLocationFrom>VT01</StorageLocationFrom>
    <StorageLocationTo>FGW1</StorageLocationTo>
</PalletUpdate>
```

###Packaging Material
####XML Format
`Name File`: YYYYMMDD_HHMMSS_PalletUpdate_<BatchID>.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PalletUpdate>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <MaterialCode>3100006023</MaterialCode>
    <Batch>PS023128</Batch>
    <Quantity>64</Quantity>
    <UoM>CTN</UoM>
    <PostingDate>2025-04-10</PostingDate>
    <StatusTo>UU</StatusTo>
    <Plant>9908</Plant>
    <StorageLocationFrom>VT01</StorageLocationFrom>
    <StorageLocationTo>ZPCK</StorageLocationTo>
</PalletUpdate>
```

##Unplanned Retrieval
###Finish Goods
####XML Format
`Name File`: YYYYMMDD_HHMMSS_PalletUpdate_<BatchID>.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PalletUpdate>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <MaterialCode>3100006023</MaterialCode>
    <Batch>PS023128</Batch>
    <Quantity>64</Quantity>
    <UoM>CTN</UoM>
    <PostingDate>2025-04-10</PostingDate>
    <StatusTo>UU</StatusTo>
    <Plant>9908</Plant>
    <StorageLocationFrom>FGW1</StorageLocationFrom>
    <StorageLocationTo>VT01</StorageLocationTo>
</PalletUpdate>
```

###Packaging Material
####XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<PalletUpdate>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <MaterialCode>3100006023</MaterialCode>
    <Batch>PS023128</Batch>
    <Quantity>64</Quantity>
    <UoM>CTN</UoM>
    <PostingDate>2025-04-10</PostingDate>
    <StatusTo>UU</StatusTo>
    <Plant>9908</Plant>
    <StorageLocationFrom>ZPCK</StorageLocationFrom>
    <StorageLocationTo>VT01</StorageLocationTo>
</PalletUpdate>
```

# Related DFD
  - [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)
