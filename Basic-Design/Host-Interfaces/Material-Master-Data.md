[[_TOC_]]

# MaterialMasterReceive
This is the module to receive material master data from SAP.
SAP will send the material master data on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
item[("DMMaterialMaster")]
serviceHostComm-->periodicItemMaster--request-->SAP
SAP--response-->periodicItemMaster
periodicItemMaster--Insert-->item
subgraph HostCommExecutor
serviceHostComm["serviceHostComm.prj\n(ConsoleApplicationExecutor)"]
periodicItemMaster["recvItemMasterData()\n>MaterialMasterReceive"]
end
:::

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
        C2["recvItemMasterData()<br>→ MaterialMasterReceive"]
        C1 --> C2
    end
:::

# User Story
- #5737

# Related DFD
- {}

