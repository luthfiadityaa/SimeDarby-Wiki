[[_TOC_]]

# RetrievalPlanDataLoader
This is the module to receive planned retrieval data from SAP.
SAP will send the planned retrieval data on the SFTP.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Insert Data| E[(DNRetrievalPlan)]
    C -->|Insert Data| F[(DNExchangeHistory)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvRetrievalPlanData()<br>→ RetrievalPlanHostDataLoader"]
        C1 --> C2
    end
:::

#XML Format

`Name File`: ShippingProcess_<MaterialNum>_YYYYMMMDD_hhmmss-xxx.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ShippingProcess>
    <MsgID>1469</MsgID>
    <Action>0</Action>
    <DocNo>230001792</DocNo>
    <ShippingDate>20250601</ShippingDate>
    <!-- YYYYMMDD -->
    <Item>
        <!-- Can be repeated -->
        <ItemNo>10</ItemNo>
        <MaterialCode>3100005421</MaterialCode>
        <Quantity>10.000</Quantity>
        <UoM>MT</UoM>
        <Batch>
            <!-- Can be repeated -->
            <BatchNo>P132/D29P</BatchNo>
            <BatchQuantity>10.000</BatchQuantity>
            <BatchUoM>MT</BatchUoM>
        </Batch>
        <ItemNo>20</ItemNo>
        <MaterialCode>3100005422</MaterialCode>
        <Quantity>10.000</Quantity>
        <UoM>MT</UoM>
        <Batch>
            <!-- Can be repeated -->
            <BatchNo>P133/D31P</BatchNo>
            <BatchQuantity>10.000</BatchQuantity>
            <BatchUoM>MT</BatchUoM>
        </Batch>
    </Item>
</ShippingProcess>
```

# User Story
- #5749

# Related DFD
- [Retrieval Plan Maintenance - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/915/Retrieval-Plan-Maintenance)
