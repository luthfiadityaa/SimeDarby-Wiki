[[_TOC_]]

# QcStatusUpdateSend
This is the module to send QC status update data result from WareNavi.
SAP will send back the QC status update data result on the response.

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
