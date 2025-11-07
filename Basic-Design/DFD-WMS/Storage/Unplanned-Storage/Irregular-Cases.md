[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart TD  
ArrivalAt1101[ID26] --> Cond1{Is Location Full?}  
Cond1 -->|Yes| Await[Await] --> Cond1  
Cond1 -->|No| Cond2{Is No Read?}  
Cond2 -->|Yes| Move1[ID05 ControlInfo:NoRead]  
Cond2 -->|No| Move2[ID05]
:::

##<span style="color:skyblue; font-weight:bold">Wait for available location if full</span>

##<span style="color:skyblue; font-weight:bold">No-Read</span>



