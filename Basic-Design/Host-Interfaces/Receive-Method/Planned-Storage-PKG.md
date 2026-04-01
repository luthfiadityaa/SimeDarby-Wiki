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
- **Material number** should be exist in **DMItem**
- The **item type** associated with the **material number** must be **ZPCK**.
- The **order unit** associated with the material number must be same with **UOM** in **DMItem**.
- The **plant** should be existed in **DMToStation**.
- The **irregular pallet** associated with the **material number** for abnormal shelves and cannot be used. 
- The **direct shipping** associated with the **material number** cannot be used. 
- The **delivery date** should be using format **yyyyMMdd**.
- The **order quantity** must be greater than 0 and less than or equal to **MAX_STOCK_QTY (999,999)**.


## Planned Storage from Host

<span style="background-color:yellow; color:black; font-weight:bold"> `jp.co.daifuku.wms.web.display.storage.plannedstoragepkg.PlannedStoragePkgSCH` </span>

::: mermaid
flowchart LR

subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvStoragePlanPkgData()<br>→ StoragePlanPkgDataLoader"]
        C1 --> C2 --> insert[("DNRECEIVINGPLAN")]
end

HostCommExecutor
:::

# StoragePlanPkgDataLoader
- Purchase Order
- Company Code
- Vendor
- Vendor Name
- Document Date
- Item Number
- Plant
- Material Number
- Order Quantity
- Order Unit
- Delivery Date

Upon receiving new Plan Storage from Host system, WareNavi will insert related planned information to DNRECEIVINGPLAN database table.

## DNRECEIVINGPLAN 
- PLAN_UKEY            = WMS Sequence Handler   
- LOAD_UNIT_KEY        = System Date with format **yyyyMMddHHmmss**
- RECEIVE_TICKET_NO    = Value from SAP (**Purchase No**) 
- CUSTOMER_CODE        = Value from SAP (**Company Code**)          
- SUPPLIER_CODE        = Value from SAP (**Vendor**)
- SUPPLIER_NAME        = Value from SAP (**Vendor Name**)
- DOCUMENT_DATE        = Value from SAP (**DOCUMENT_DATE**)                                                                                                       
- STATUS_FLAG          = 0:Not Started                                                       
- CANCEL_FLAG          = 0:Normal Data                                                      
- PLAN_DAY             = Value from SAP (**Delivery Date**)
- FILE_LINE_NO         = Indicating Line Number of location XML Tag (**System Decided**), as default is null.                                     
- RECEIVE_LINE_NO      = Value from SAP (**ItemNumber**)                                                      
- PLAN_AREA_NO         = 9200
- ITEM_CODE            = Value from SAP (**MaterialNumber**)
- SAP_TO_LOCATION      = Value from SAP (**PLANT**)                                                                                                              
- PLAN_QTY             = Value from SAP (**OrderQuantity**)
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
|  DocumentDate  |   DOCUMENT_DATE   |              |          |                               |
|  CompanyCode   |   CUSTOMER_CODE   |              |          |                               |
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
