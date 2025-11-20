**Stage 1**
::: mermaid
flowchart LR
  
P1[From Station 1303]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR
  
Cond1{Flow?} 
Cond1 --> |Flow 1: Without going 
through crane 7~10 - STV| P8[ID64]
Cond1 --> |Flow 2: Go through 
crane 7~10 -STV| P9[ID64]
P8[ID64]--> |7101-7110| P10[ID26]-->P11[StorageSender]-->P12[ID25]--> |SRM| P18[ID64]
P9[ID64]--> |7107-7110| P15[ID26]-->P16[StorageSender]-->P17[ID25]--> |SRM| P18[ID64]
:::

**Stage 3**
::: mermaid
flowchart LR 
P18[ID64]--> |STV| P19[ID64]--> |7211-7214| P20[ID26]-->P21[StorageSender]-->P22[ID25]--> |SRM| P23[ID64]

P18[ID64]--> |9001-9010| P14[ID33]
P23[ID64]--> |9011-9014| P14[ID33]

P14[ID33]-->P1[TO AISLE STATION - 9001, 
9002 , 9003, 9004, 9005, 
9006, 9007, 9008, 9009, 
9010, 9011, 9012, 9013, 
9014]
:::