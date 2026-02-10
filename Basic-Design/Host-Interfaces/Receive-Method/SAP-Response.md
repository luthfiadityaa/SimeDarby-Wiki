[[_TOC_]]

#<b>ResponseDataLoader</b>
This is the module When warenavi send the Result to SAP, SAP will send a response to indicate the result of the communication.

#<b>DFD</b>
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
        C2["recvResponseDataLoader()<br>→ ResponseReportData"]
        C1 --> C2
    end
:::

#<b>Response Data from SAP</b>
##<b>Description</b>
- <b>MsgID</b>  
  Unique Id for communication
- <b>OriginalMessageID</b>  
  Original Message ID originationg from the file sent by Warenavi to SAP  
- <b>MessageType</b>
![image.png](/.attachments/image-0953a9ac-e601-4ae7-bb27-06b4cb667d71.png)
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
 - <b>SAPDocNo</b>  
 SAP document number.
 - <b>ErrorIndicator</b> 
 1 = Error
 0 = Successful
 - <b>Type</b> 
 S – Success
 E – Error
 I – Information
 W - Warning
 - <b>MessageDesc</b>    
 SAP base UOM

##<b>XML Format</b>
`Name File`: Response_<MessageType>_YYYYMMMDD_hhmmss-xxx.xml

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
        <MessageDesc>Processing SPOT invoice 2747/VCH/2025/0193</MessageDesc>
    </Messages>
    <Messages>
        <Type>S</Type>
        <MessageDesc>Incoming invoice 5105698830 2025 is created</MessageDesc>
    </Messages>
</Response>
```