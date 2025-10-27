[[_TOC_]]

# PlannedRetrievalReceive
This is the module to receive planned retrieval data from SAP.
SAP will send the planned retrieval data on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR

subgraph HostCommExecutor
serviceHostComm["serviceHostComm.prj\n(ConsoleApplicationExecutor)"]
end
:::

# Related DFD
  - #5749
