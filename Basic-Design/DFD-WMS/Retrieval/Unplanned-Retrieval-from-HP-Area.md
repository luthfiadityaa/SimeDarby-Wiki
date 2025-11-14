#<span style="color:skyblue; font-weight:bold">Retrieval from 1301, 1302, 1205, 1206, 1207, 1208, 1209</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR
  
P1[FROM HP STATION - 1301, 1302, 1205, 1206, 1207, 1208, 1209]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR
  
Cond1{Flow?} 
Cond1 --> |Flow 1: Without going 
through crane 7~10 - STV| P8[ID64]
Cond1 --> |Flow 2: Go through 
crane 7~10 -STV| P9[ID64]
P8[ID64]--> |7207-7214| P10[ID26]-->P11[StorageSender]-->P12[ID25]--> |SRM| P18[ID64]
P9[ID64]--> |7207-7210| P15[ID26]-->P16[StorageSender]-->P17[ID25]--> |SRM| P18[ID64]
:::

**Stage 3**
::: mermaid
flowchart LR 
P18[ID64]--> |STV| P19[ID64]--> |7101-7106| P20[ID26]-->P21[StorageSender]-->P22[ID25]--> |SRM| P23[ID64]

P18[ID64]--> |9007-9014| P14[ID33]
P23[ID64]--> |9001-9006|P14[ID33]
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
**Abbreviation:**
- **STRP** : DNSTORAGEPLAN  
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKINFOLIST  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
- **HSTS** : DNHOSTSEND  
- **ARVL** : DNARRIVAL  
- **WRHS** : DMWAREHOUSE  
- **SHLF** : DMSHELF  
- **ITEM** : DMITEM
- **STSN** : DMSTATION  
- **STCH** : DNSTOCKHISTORY  

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 