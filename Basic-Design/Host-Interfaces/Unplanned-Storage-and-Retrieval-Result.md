[[_TOC_]]

# StorageRetievalReportData
This is the module to send unplanned Storage/Retrieval data result from WareNavi.
SAP will send back the unplanned Storage/Retrieval data result on the response.

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
    G[(DNRetrievalPlan)] <--> |Update| A
    H[(DNStoragePlan)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendStorageRetrievalReportData()<br>→ StorageRetievalReportData"]
        C1 --> C2
    end
:::

#Result Data from WareNavi
##Unplanned Storage
###Finish Goods
####TXT/CSV Format
```csv
1469,0,3100006023,PS023128,64,CTN,2025-04-10,UU,9908,VT01,FGW1
```
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
####TXT/CSV Format
```csv
1469,0,3100006023,PS023128,64,CTN,2025-04-10,UU,9908,VT01,ZPCK
```
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
    <StorageLocationTo>ZPCK</StorageLocationTo>
</PalletUpdate>
```

##Unplanned Retrieval
###Finish Goods
####TXT/CSV Format
```csv
1469,0,3100006023,PS023128,64,CTN,2025-04-10,UU,9908,FGW1,VT01
```
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
    <StorageLocationFrom>FGW1</StorageLocationFrom>
    <StorageLocationTo>VT01</StorageLocationTo>
</PalletUpdate>
```

###Packaging Material
####TXT/CSV Format
```csv
1469,0,3100006023,PS023128,64,CTN,2025-04-10,UU,9908,ZPCK,VT01
```
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

# User Story
  - #5752

# Related DFD
  - {}
