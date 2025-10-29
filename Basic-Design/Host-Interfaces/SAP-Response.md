[[_TOC_]]

# ResponseDataLoader
This is the module When warenavi send the Result to SAP, sap will send a response to indicate the result of the communication.

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
    G[(DNRetrievalPlan)] <--> |Update| A
    H[(DNHostSend)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvResponse()<br>→ ResponseReportData"]
        C1 --> C2
    end
:::

#Response Data from SAP

##Description
  
- MessageType
  Mentioning of the interfaces Processed below:  
  1. Production Storage
     - [Production Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)
  2. Pallet Update
     - [QC Status Update Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/842/QC-Status-Update-Result)
     - [Internal Location Transfer Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)
  3. Delivery Order
     - [Planned Storage Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/846/Planned-Storage-Result)
     - [Planned Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/848/Planned-Retrieval-Result)
     - [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result) 

##XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <MsgID>5F678C281A9F11F0A3D4000011FE13AB</MsgID>
    <MessageType>PRODUCTION_STORAGE</MessageType>
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
  - #5760

# Related DFD
  - {}
