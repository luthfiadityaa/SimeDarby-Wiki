[[_TOC_]]

# StoragePlanPkgDataLoader
This is the module to receive planned storage PKG data from SAP.
SAP will send the planned storage data on the SFTP.

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
<PurchaseOrder>
    <CompanyCode>1000</CompanyCode>
    <Vendor>1234567890</Vendor>
    <VendorName>ABC Supplies Ltd.</VendorName>
    <DocumentDate>20250623</DocumentDate>
    <Items>
        <Item>
            <ItemNumber>00010</ItemNumber>
            <Plant>SG01</Plant>
            <MaterialNumber>MAT123456789000001</MaterialNumber>
            <OrderQuantity>100.000</OrderQuantity>
            <OrderUnit>EA</OrderUnit>
            <DeliveryDate>20250701</DeliveryDate>
        </Item>
        <Item>
            <ItemNumber>00020</ItemNumber>
            <Plant>SG01</Plant>
            <MaterialNumber>MAT123456789000002</MaterialNumber>
            <OrderQuantity>200.000</OrderQuantity>
            <OrderUnit>BOX</OrderUnit>
            <DeliveryDate>20250710</DeliveryDate>
        </Item>
    </Items>
</PurchaseOrder>
```

#TXT/CSV Format
```csv
1000,1234567890,"ABC Supplies Ltd.",20250623,00010,SG01,MAT123456789000001,100.000,EA,20250701
1000,1234567890,"ABC Supplies Ltd.",20250623,00020,SG01,MAT123456789000002,200.000,BOX,20250710
```  

# User Story
  - #5117

# Related DFD
- [Storage Plan Maintenance (PKG) - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/917/Storage-Plan-Maintenance-(PKG))
