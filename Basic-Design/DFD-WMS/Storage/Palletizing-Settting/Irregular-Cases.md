[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart TD  
ArrivalAt1101[ID26] --> Cond1{Is Location Full?}  
Cond1 -->|Yes| Await[Await] --> Cond1  
Cond1 -->|No| Cond2{Is No Read?}  
Cond2 -->|Yes| Move1[ID05 ControlInfo:NoRead]  
Cond2 -->|No| Cond3{Is Batch Started?}  
Cond3 -->|Yes| Cond4{Total Actual Carton Qty = 
Planned Carton Qty ?}  
Cond3 -->|No| DataError1[Data Error]  
DataError1[Data Error] --> BatchStart[Batch Start] --> Cond4  
Cond4 -->|Yes| Move2[ID05]  
Cond4 -->|No| DataError2[Data Error] --> BatchEnd[Batch End] --> Move2[ID05]
:::

##<span style="color:skyblue; font-weight:bold">No-Read (Go to Reject station)</span>

##<span style="color:skyblue; font-weight:bold">Wait for Batch Start. Treat as Data Error</span>

##<span style="color:skyblue; font-weight:bold">Reverse – Batch End Not Executed Despite Force Completion. Treat as Data Error</span>

