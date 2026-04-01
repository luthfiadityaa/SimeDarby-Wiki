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
    C --> Cond1{"isDataError ?"} 
    Cond1 --> |FALSE| E[(DMItem)]
    Cond1 --> |TRUE| G[(DNLoadErrorInfo)]
    E --> |Save Communication Data|F[(DNExchangeHistory)]
    G --> |Save Communication Data|F[(DNExchangeHistory)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvMaterialMasterData()<br>→ MaterialMasterDataLoader"]
        C1 --> C2
    end
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
* **MSG_ID**: Value from SAP
* **MSG_TYPE**: ? 
* **ERROR_INDICATION**: 0:Successful
* **DELETE_INDICATION**: 0:Normal
* **TYPE**: S: Success
* **MESSAGE_DESC**: Free Text -> **The master material data has been inserted.**    


#XML Format
`Name File`: Material_<MaterialNum>_YYYYMMMDD_hhmmss-xxx.xml​

##Finish Goods
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MaterialMaster>
    <MsgID>0000000002523009</MsgID>
    <MaterialCode>2900000002</MaterialCode>
    <MaterialName>CRUDE PALM OIL (CPO) v1</MaterialName>
    <QuantityKGCtn>12</QuantityKGCtn>
    <QuantityCtnPal>56</QuantityCtnPal>
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

###<span style="color:skyblue; font-weight:bold">DMItem</span>
The data will be paired as an input: **SAP** ⇄ **Warenavi**

|       SAP      |       Warenavi      | Primary Key | Required |
|:--------------:|:-------------------:|:-----------:|:--------:|
|  MaterialCode  |      ITEM_CODE      |      P1     |     Y    |
|  MaterialName  |      ITEM_NAME      |             |     Y    |
|  MaterialType  |      ITEM_TYPE      |             |     Y    |
|       UOM      |         UOM         |             |     Y    |
|  QuantityKgCtn |     ENTERING_QTY    |             |          |
| QuantityCtnPal | BUNDLE_ENTERING_QTY |             |          |

# User Story
- #5737

# Related DFD
- [Material Information Maintenance - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/914/Material-Information-Maintenance)

