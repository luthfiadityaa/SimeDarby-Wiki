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
    C --> Cond1{"isDataError ?"}
    Cond1 --> |FALSE| E[(DNStoragePlan)]
    Cond1 --> |TRUE| G[(DNLoadErrorInfo)]
    E --> |Save Communication Data|F[(DNExchangeHistory)]
    G --> |Save Communication Data|F[(DNExchangeHistory)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2
    end
:::

# XML Format

`Name File`: PL_Stor_YYYYMMMDDhhmmss.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PurchaseOrder>
    <CompanyCode>1000</CompanyCode>
    <Vendor>1234567890</Vendor>
    <VendorName>ABC Supplies Ltd.</VendorName>
    <DocumentDate>20250623</DocumentDate>
    <Items>
        <Item>
            <PurchaseOrder>1111111111<PurchaseOrder>
            <ItemNumber>00010</ItemNumber>
            <Plant>SG01</Plant>
            <MaterialNumber>MAT123456789000001</MaterialNumber>
            <OrderQuantity>100.000</OrderQuantity>
            <OrderUnit>EA</OrderUnit>
            <DeliveryDate>20250701</DeliveryDate>
        </Item>
        <Item>
            <PurchaseOrder>1111111111<PurchaseOrder>
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

### <span style="color:skyblue; font-weight:bold">DNSTORAGEPLAN</span>

The data will be paired as an input: **SAP** ⇄ **Warenavi**

|       SAP      |      Warenavi     |  Primary Key | Required |            Remarks            |
|:--------------:|:-----------------:|:------------:|:--------:|:-----------------------------:|
|     Vendor     |   SUPPLIER_CODE   |              |          |                               |
|   VendorName   |   SUPPLIER_NAME   |              |          |                               |
|  DocumentDate  |       --NA--      |              |          |                               |
|    ---------   |   --------------  | ------------ |  ------- |  ---------------------------- |
|  PurchaseOrder | RECEIVE_TICKET_NO |      P1      |          |                               |
|   ItemNumber   |  RECEIVE_LINE_NO  |      P2      |          |                               |
|      Plant     |  SAP_TO_LOCATION  |              |          | Refer: DMTOSTATION.STATION_NO |
| MaterialNumber |     ITEM_CODE     |              |          |                               |
|  OrderQuantity |      PLAN_QTY     |              |          |                               |
|    OrderUnit   |       --NA--      |              |          |       Follow DMITEM.UOM       |
|  DeliveryDate  |      PLAN_DAY     |              |          |                               |

# User Story

- #5117

# Related DFD

- [Storage Plan Maintenance (PKG) - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/917/Storage-Plan-Maintenance-(PKG))
