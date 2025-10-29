[[_TOC_]]

# ProductionStorageReportData
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
    B -->|Return XML Response via SFTP| A
    E[(DNStoragePlan)] <--> |Update| A
    F[(DNHostSend)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendProductionStorageReportData()<br>→ ProductionStorageReportData"]
        C1 --> C2
    end
:::

#Result Data from WareNavi
##TXT/CSV Format
```csv
1469,0,3100006023,9908,PS023128,64,CTN,2025-04-10,UU,VT01,FGW2,PLY0001
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
    <StatusTo>UU</StatusTo>
    <StorageLocationFrom>VT01</StorageLocationFrom>
    <StorageLocationTo>FGW2</StorageLocationTo>
    <PalletID>PLY0001</PalletID>
</PalletUpdate>
```

# User Story
  - #5758

# Related DFD
- {}