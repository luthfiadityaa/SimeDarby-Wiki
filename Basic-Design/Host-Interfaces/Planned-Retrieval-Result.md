[[_TOC_]]

# PlannedRetrievalSend
This is the module to send planned retrieval data result from WareNavi.
SAP will send back the planned retrieval data result on the response.

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
  - #5757
