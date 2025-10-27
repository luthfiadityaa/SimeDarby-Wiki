[[_TOC_]]

# UnplannedRetrievalSend
This is the module to send unplanned retrieval data result from WareNavi.
SAP will send back the unplanned retrieval data result on the response.

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
  - #5752
