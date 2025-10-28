[[_TOC_]]

# PlannedStorageSend
This is the module to send planned storage data result from WareNavi.
SAP will send back the planned storage data result on the response.

# DFD
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

::: mermaid
flowchart LR

subgraph HostCommExecutor
serviceHostComm["serviceHostComm.prj\n(ConsoleApplicationExecutor)"]
end
:::

# User Story
  - #5756

# Related DFD
  - {}
