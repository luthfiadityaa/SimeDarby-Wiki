[[_TOC_]]

# ResponseSend
This is the module When warenavi send the Result to SAP, sap will send a response to indicate the result of the communication.

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
  - #5760

# Related DFD
  - {}
