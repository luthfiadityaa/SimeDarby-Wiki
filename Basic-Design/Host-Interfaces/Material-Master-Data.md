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
periodicItemMaster["getItemMaster()\n>ItemMasterRequestSender"]
end
:::

# User Story
- #5737

# Related DFD
- {}

