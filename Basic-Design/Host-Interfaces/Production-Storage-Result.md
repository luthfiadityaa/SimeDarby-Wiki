[[_TOC_]]

# ProductionStorageSend
This is the module to send production storage data result from WareNavi.
SAP will send back the production storage data result on the response.

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
  - {}
