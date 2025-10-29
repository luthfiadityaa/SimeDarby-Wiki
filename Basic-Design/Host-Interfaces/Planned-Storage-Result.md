[[_TOC_]]

# StorageReportData
This is the module to send planned storage data result from WareNavi.
SAP will send back the planned storage data result on the response.

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
    E[(DNStock)] <--> |Update| A
    F[(DNHostSend)] <--> |Update| A
    G[(DNStoragePlan)] <--> |Update| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["sendStorageReportData()<br>→ StorageReportData"]
        C1 --> C2
    end
:::

#Result Data from WareNavi
##TXT/CSV Format
###GR
```csv
20250623,20250620,101,4500012345,00010,MAT123456789000001,SG01,0001,50.000,EA,PALLET-001,BATCH00123
20250623,20250620,101,4500012345,00020,MAT123456789000002,SG01,0002,75.000,EA,PALLET-002,BATCH00456
```
##XML Format
###GR
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MaterialDocument>
    <PostingDate>20250623</PostingDate>
    <DocumentDate>20250620</DocumentDate>
    <Items>
        <Item>
            <MovementType>101</MovementType>
            <PurchaseOrder>4500012345</PurchaseOrder>
            <PurchaseOrderItem>00010</PurchaseOrderItem>
            <MaterialNumber>MAT123456789000001</MaterialNumber>
            <Plant>SG01</Plant>
            <StorageLocation>0001</StorageLocation>
            <Quantity>50.000</Quantity>
            <UnitOfEntry>EA</UnitOfEntry>
            <UnloadingPoint>PALLET-001</UnloadingPoint>
            <Batch>BATCH00123</Batch>
        </Item>
        <Item>
            <MovementType>101</MovementType>
            <PurchaseOrder>4500012345</PurchaseOrder>
            <PurchaseOrderItem>00020</PurchaseOrderItem>
            <MaterialNumber>MAT123456789000002</MaterialNumber>
            <Plant>SG01</Plant>
            <StorageLocation>0002</StorageLocation>
            <Quantity>75.000</Quantity>
            <UnitOfEntry>EA</UnitOfEntry>
            <UnloadingPoint>PALLET-002</UnloadingPoint>
            <Batch>BATCH00456</Batch>
        </Item>
    </Items>
</MaterialDocument>
```

##TXT/CSV Format
###CANCEL GR
```csv
20250623,20250620,102,4500012345,00010,MAT123456789000001,SG01,0001,50.000,EA,PALLET-001,BATCH00123
20250623,20250620,101,4500012345,00020,MAT123456789000002,SG01,0002,75.000,EA,PALLET-002,BATCH0045656
```
##XML Format
###CANCEL GR
```xml
<?xml version="1.0" encoding="UTF-8"?>
<MaterialDocument>
    <PostingDate>20250623</PostingDate>
    <DocumentDate>20250620</DocumentDate>
    <Items>
        <Item>
            <MovementType>102</MovementType>
            <PurchaseOrder>4500012345</PurchaseOrder>
            <PurchaseOrderItem>00010</PurchaseOrderItem>
            <MaterialNumber>MAT123456789000001</MaterialNumber>
            <Plant>SG01</Plant>
            <StorageLocation>0001</StorageLocation>
            <Quantity>50.000</Quantity>
            <UnitOfEntry>EA</UnitOfEntry>
            <UnloadingPoint>PALLET-001</UnloadingPoint>
            <Batch>BATCH00123</Batch>
        </Item>
        <Item>
            <MovementType>101</MovementType>
            <PurchaseOrder>4500012345</PurchaseOrder>
            <PurchaseOrderItem>00020</PurchaseOrderItem>
            <MaterialNumber>MAT123456789000002</MaterialNumber>
            <Plant>SG01</Plant>
            <StorageLocation>0002</StorageLocation>
            <Quantity>75.000</Quantity>
            <UnitOfEntry>EA</UnitOfEntry>
            <UnloadingPoint>PALLET-002</UnloadingPoint>
            <Batch>BATCH00456</Batch>
        </Item>
    </Items>
</MaterialDocument>
```

#Response Data from SAP
##XML Format
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <MsgID>5F678C281A9F11F0A3D4000011FE13AB</MsgID>
    <MessageType>DELIVERY_ORDER</MessageType>
    <OriginalMessageID>17875422</OriginalMessageID>
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
5F678C281A9F11F0A3D4000011FE13AB,PRODUCTION_STORAGE,17875422,100001301,0,I,"Processing SPOT invoice 2747/VCH/2025/0193"
5F678C281A9F11F0A3D4000011FE13AB,PRODUCTION_STORAGE,17875422,100001301,0,S,"Incoming invoice 5105698830 2025 is created"
```

# User Story
  - #5756

# Related DFD
  - {}
