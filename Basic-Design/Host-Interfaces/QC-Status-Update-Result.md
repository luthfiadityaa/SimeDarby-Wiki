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

#Response Data from SAP
##XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <MsgID>5F678C281A9F11F0A3D4000011FE13AB</MsgID>
    <MessageType>PALLET_UPDATE</MessageType>
    <OriginalMessageID>1469</OriginalMessageID>
    <SAPDocNo>100001301</SAPDocNo>
    <ErrorIndicator>0</ErrorIndicator>
    <Messages>
        <Type>I</Type>
        <MessageDesc>Processing SPOT invoice 2747/VCH/2025/0193
         </MessageDesc>
    </Messages>
    <Messages>
        <Type>S</Type>
        <MessageDesc>Incoming invoice 5105698830 2025 is created
         </MessageDesc>
    </Messages>
</Response>
```

##TXT/CSV Format
```csv
5F678C281A9F11F0A3D4000011FE13AB,PRODUCTION_STORAGE,1469,100001301,0,I,"Processing SPOT invoice 2747/VCH/2025/0193"
5F678C281A9F11F0A3D4000011FE13AB,PRODUCTION_STORAGE,1469,100001301,0,S,"Incoming invoice 5105698830 2025 is created"
```

# User Story
  - https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5759)

# Related DFD
  - {}
