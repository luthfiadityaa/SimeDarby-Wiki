[[_TOC_]]
[[_TOSP_]]


#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
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

#<span style="color:skyblue; font-weight:bold">QC Work - Retrieval for QC Start database flow</span>
**Abbreviation:**
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
- **TTSN** : DMTOSTATION 
- **INOUT**: DNINOUTSTATION

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

# QC Work - Retrieval for QC Start - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCSettingSCH &nbsp;</span>
 ![==image_0==.jpg](/.attachments/==image_0==-385c044c-0af1-4f9f-b8bd-8c195cfba22a.jpg) 

This screen is mainly used when the pallet needs to be inspected.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess QC Work - Retrieval for QC Start
