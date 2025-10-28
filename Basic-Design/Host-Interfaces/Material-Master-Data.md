[[_TOC_]]

# MaterialMasterDataLoader
This is the module to receive material master data from SAP.
SAP will send the material master data on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Convert XML → TXT/CSV| D[FileExchangeConverter]
    D -->|Insert Data| E[(DMMaterialMaster)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvMaterialMasterData()<br>→ MaterialMasterDataLoader"]
        C1 --> C2
    end
:::

# User Story
- #5737

# Related DFD
- [Material Information Maintenance - Overview](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/914/Material-Information-Maintenance)

