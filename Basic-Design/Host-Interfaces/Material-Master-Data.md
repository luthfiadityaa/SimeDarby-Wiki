[[_TOC_]]

# MaterialMasterDataLoader
This is the module to receive material master data from SAP.
SAP will send the material master data on the SFTP.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Convert XML → TXT/CSV| D[FileExchangeConverter]
    D -->|Insert Data| E[(DMMaterialMaster)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvMaterialMasterData()<br>→ MaterialMasterDataLoader"]
        C1 --> C2
    end
:::

#XML Format
##Finish Goods
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

##Packaging Material
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MaterialMaster>
     <MsgID>00000000025230999</MsgID>
     <MaterialCode>4900000002</MaterialCode>
     <MaterialName>PACKAGING 001</MaterialName>
     <UoM>KG</UoM>
     <MaterialType>ZPCK</MaterialType>
     <DeletionInd>0</DeletionInd>
</MaterialMaster>
```

#TXT/CSV Format
##Finish Goods
```csv
0000000002523009,2900000002,"CRUDE PALM OIL (CPO) v1",12,64,CTN,ZFNP,0
```  

##Packaging Material
```csv
00000000025230999,4900000002,"PACKAGING 001",KG,ZPCK,0
```

# User Story
- #5737

# Related DFD
- [Material Information Maintenance - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/914/Material-Information-Maintenance)

