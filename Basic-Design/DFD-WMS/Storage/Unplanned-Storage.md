[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM OP STATION - QC STATION]-->P2[ID26]-->P3[AutomaticChangeSender]-->P4[ID25]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 
Cond1 --> |Flow 1| P8[ID64]
Cond1 --> |Flow 2| P9[ID64]
P8[ID64]-->P10[ID26]-->P11[AutomaticChangeSender]-->P12[ID25]-->P13[ID64]
P9[ID64]-->P15[ID26]-->P16[AutomaticChangeSender]-->P17[ID25]-->P18[ID64]-->P19[ID64]-->P20[ID26]-->P21[AutomaticChangeSender]-->P22[ID25]-->P23[ID64]

P24[FROM HP STATION - 1301 & 
1302]-->P2[ID26]

P13[ID64]-->P14[ID33]
P23[ID64]-->P14[ID33]
:::