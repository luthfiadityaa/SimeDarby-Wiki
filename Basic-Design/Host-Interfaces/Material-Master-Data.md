[[_TOC_]]

# MaterialMasterReceive
This is the module to receive material master data from SAP.
SAP will send the material master data on the response.

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
- https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5737

