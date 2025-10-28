[[_TOC_]]

# ProductionStorageReportData
This is the module to send production storage data result from WareNavi.
SAP will send back the production storage data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP System] -->|Send XML via SFTP| B[FileExchangeConverter]
    B -->|Convert XML → TXT/CSV| C[HostCommExecutor]
    C -->|Insert Data| D[(DMMaterialMaster)]
    D -->|Send Data Back| C
    C -->|Convert TXT/CSV → XML| B
    B -->|Return XML Response via SFTP| A

     subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvProductionStorageReportData()<br>→ ProductionStorageReportData"]
        C1 --> C2
    end
:::

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Convert XML → TXT/CSV| D[FileExchangeConverter]
    D -->|Insert| E[(DNStoragePlan)]
    E -->|Response| F[(DNHostSend)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvProductionStorageReportData()<br>→ ProductionStorageReportData"]
        C1 --> C2
    end
:::

#XML Format
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
            <BatchNo>P133/D30P</BatchNo>
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

#TXT/CSV Format
```csv
1469,0,230001792,20250601,10,3100005421,10.000,MT,P132/D29P,10.000,MT
1469,0,230001792,20250601,10,3100005421,10.000,MT,P133/D30P,10.000,MT
1469,0,230001792,20250601,20,3100005422,10.000,MT,P133/D31P,10.000,MT
```

# User Story
  - #5758

# Related DFD
- {}