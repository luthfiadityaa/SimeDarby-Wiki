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
    B -->|Return TXT/CSV Response via SFTP| A
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
##GR
###TXT/CSV Format
```csv
20250623,20250620,101,4500012345,00010,MAT123456789000001,SG01,0001,50.000,EA,PALLET-001,BATCH00123
20250623,20250620,101,4500012345,00020,MAT123456789000002,SG01,0002,75.000,EA,PALLET-002,BATCH00456
```
###XML Format
`Name File`: GR_XMLxml  

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

##CANCEL GR
###TXT/CSV Format
```csv
20250623,20250620,102,4500012345,00010,MAT123456789000001,SG01,0001,50.000,EA,PALLET-001,BATCH00123
20250623,20250620,101,4500012345,00020,MAT123456789000002,SG01,0002,75.000,EA,PALLET-002,BATCH0045656
```
###XML Format
`Name File`: Cancel_GR_XML.xml

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

# Related DFD
  - [SAP Response - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)
