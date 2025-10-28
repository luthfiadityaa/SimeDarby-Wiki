[[_TOC_]]

# StoragePlanPkgDataLoader
This is the module to receive planned storage PKG data from SAP.
SAP will send the planned storage data on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Convert XML → TXT/CSV| D[FileExchangeConverter]
    D -->|Insert| E[(DNStoragePlan)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanPkgData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2
    end
:::

#XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MaterialMaster>
    <MsgID>0000000002523009</MsgID>
    <MaterialCode>2900000002</MaterialCode>
    <MaterialName>CRUDE PALM OIL (CPO) v1</MaterialName>
    <QuantityKGCtn>12</QuantityKGCtn>
    <QuantityCtnPal>64</QuantityCtnPal>
    <UoM>CTN</UoM>
    <MaterialType>ZFNP</MaterialType>
    <DeletionInd>0</DeletionInd>
</MaterialMaster>
```

#TXT/CSV Format
```csv
0000000002523009,2900000002,"CRUDE PALM OIL (CPO) v1",12,64,CTN,ZFNP,0
```  

# User Story
  - #5117

# Related DFD
- {}
