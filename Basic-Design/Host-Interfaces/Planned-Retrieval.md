[[_TOC_]]

# RetrievalPlanDataLoader
This is the module to receive planned retrieval data from SAP.
SAP will send the planned retrieval data on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR
    A[SAP] -->|Send XML via SFTP| B[(FTP Folder)]
    B -->|GET XML| C[HostCommExecutor]
    C -->|Convert XML → TXT/CSV| D[FileExchangeConverter]
    D -->|Insert| E[(DNRetrievalPlan)]

    subgraph HostCommExecutor
        C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]
        C2["recvRetrievalPlanData()<br>→ RetrievalPlanDataLoader"]
        C1 --> C2
    end
:::

# User Story
- #5749

# Related DFD
- {}
