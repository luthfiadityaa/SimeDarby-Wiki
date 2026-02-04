[[_TOC_]]

# RetrievalReportData
This is the module to send planned retrieval data result from WareNavi.
SAP will send back the planned retrieval data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[HostCommExecutor] --> |SEND XML| C[(FTP Folder)]
    C -->|GET XML via SFTP| D[SAP]
    D -->|Send Back Response| C
    C -->|Return XML Response via SFTP| A
    E[(DNRetrievalPlan)] <--> |Update| A
    F[(DNHostSend)] <--> |Update| A
    G[(DNWorkInfo)] <--> |Update| A

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendRetrievalReportData()<br>→ RetrievalReportData"]
        C1 --> C2
    end
:::

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
    <StatusFrom>QI</StatusFrom>
    <StatusTo>UU</StatusTo>
    <StorageLocationTo>FGW2</StorageLocationTo>
    <DocNumber>3050343982</DocNumber>
    <PalletID>PLT00001</PalletID>
</PalletUpdate>
```

# Related DFD
  - [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)
