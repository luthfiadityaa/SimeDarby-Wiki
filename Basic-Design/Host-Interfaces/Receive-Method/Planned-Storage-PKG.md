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
    Cond1 --> |FALSE| E[(DNReceivingPlan)]
    Cond1 --> |TRUE| G[(DNLoadErrorInfo)]
    E --> |Save Communication Data|F[(DNExchangeHistory)]
    G --> |Save Communication Data|F[(DNExchangeHistory)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2
    end
:::

![image.png](/.attachments/image-409d5c38-b7f4-4af5-99c4-2be34851bf7f.png)


# XML Format

`Name File`: PL_Stor_YYYYMMMDDhhmmss.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PurchaseOrder>
    <PurchaseOrder>1111111111</PurchaseOrder>
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

## Validation
- When the OrderUnit is not same to existing UOM in DMItem, the plan data should be rejected with error message.


## Planned Storage from Host

<span style="background-color:yellow; color:black; font-weight:bold"> `jp.co.daifuku.wms.web.display.storage.plannedstoragepkg.PlannedStoragePkgSCH` </span>

::: mermaid
flowchart LR

subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanPkgData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2 --> insert[("DNRETRIEVALPLAN")]
end

HostCommExecutor
:::

# StoragePlanPkgDataLoader
- Document Number
- Company Code
- Vendor
- Vendor Name
- Document Date
- Item No / Line No
- Plant
- Material Code
- Planned Quantity
- Uom
- Delivery Date

Upon receiving new Plan Storage from Host system, WareNavi will insert related planned information to DNRETRIEVALPLAN database table.

## DNRECEIVINGPLAN                                                                                                           
- STATUS_FLAG          = 0:Not Started                                                       
- CANCEL_FLAG          = 0:Normal Data                                                      
- PLAN_DAY             = Value from SAP (**Delivery Date**)                                                       
- VENDOR_CODE          = Value from SAP (**Vendor Code**)
- VENDOR_NAME          = Value from SAP (**Vendor Name**)                                                     
- COMPANY_CODE         = Value from SAP (**Company Code**)                                                      
- RECEIVE_TICKET_NO    = Value from SAP (**Purchase No**)                                                      
- RECEIVE_LINE_NO      = Value from SAP (**ItemNumber**)                                                      
- DOCUMENT_DATE        = Value from SAP (**Document Date**)                                                                                                              
- PLANT                = CONSTANT.SAP_PLANT (9908)                                                      
- ITEM_CODE            = Value from SAP (**MaterialNumber**)                                                                                                              
- PLAN_QTY             = Value from SAP (**OrderQuantity**) 
- SAP_TO_LOCATION      = CONSTANT.SLOC_PACK 
- REPORT_FLAG          = 0:Not Reported                                                                                                           
- REGIST_KIND          = 0:File Loading                                                      
- REGIST_DATE          = SYSTIMESTAMP                                                  
- REGIST_PNAME         = ClassName
- LAST_UPDATE_DATE     = SYSTIMESTAMP
- LAST_UPDATE_PNAME    = ClassName

### <span style="color:skyblue; font-weight:bold">DNRECEIVINGPLAN</span>

The data will be paired as an input: **SAP** ⇄ **Warenavi**

|       SAP      |      Warenavi     |  Primary Key | Required |            Remarks            |
|:--------------:|:-----------------:|:------------:|:--------:|:-----------------------------:|
|  PurchaseOrder | RECEIVE_TICKET_NO |      P1      |          |                               |
|     Vendor     |   SUPPLIER_CODE   |              |          |                               |
|   VendorName   |   SUPPLIER_NAME   |              |          |                               |
|  DocumentDate  |       --NA--      |              |          |                               |
|    ---------   |   --------------  | ------------ |  ------- |  ---------------------------- |
|   ItemNumber   |  RECEIVE_LINE_NO  |      P2      |          |                               |
|      Plant     |       9908        |              |          |                               |
| MaterialNumber |     ITEM_CODE     |      P3      |          |                               |
|  OrderQuantity |      PLAN_QTY     |              |          |                               |
|    OrderUnit   |       --NA--      |              |          |       Follow DMITEM.UOM       |
|  DeliveryDate  |     PLAN_DAY      |              |          |                               |

# User Story

- #5117

# Related DFD

- [Storage Plan Maintenance (PKG) - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/917/Storage-Plan-Maintenance-(PKG))
