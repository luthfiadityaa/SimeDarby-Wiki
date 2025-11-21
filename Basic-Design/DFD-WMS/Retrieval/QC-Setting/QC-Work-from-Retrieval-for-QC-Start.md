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

| Action Name                                                      |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN| 
|------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|
| ID63 [(1)](#ID63)                                                |    |    |    |    |    |    |     |    |    |    |    | U  |   
| QC Work - Retrieval for QC Start - Set(F2) [(2)](#QC-Work---Retrieval-for-QC-Start---Set(F2)) | I  |    |    |    |    |    |     |    |    |    | S  |    |  
| ID26[(3)](#ID26-at-1303)                                  |    |    | I  | I  | I  |    | I   |    |    |    |    | S  |  
| StorageSender[(4)](#Storage-Sender-at-1301-&-1302)               | U  |    | U  | U  | U  |    | U   | U  | U  |    |    |    |    
| ID25[(5)](#ID25-at-1303)                                  |    |    |    | U  |    |    | D   |    |    |    |    |    |     
| ID64[(6)](#ID64-at-STV-From-1303)                         |    |    |    | U  |    |    |     |    |    |    |    |    |     
| **Flow 1**                                                       |    |    |    |    |    |    |     |    |    |    |    |    | 
| ID26[(7)](#ID64-at-7101-7110)                                    |    |    | U  | U  |    |    | I   |    |    |    |    |    |
| StorageSender[(10)](#Storage-Sender-at-7101-7110)                |    |    |    | U  |    |    | U   |    |    |    |    |    | 
| ID25[(8)](#ID25-at-7101-7110)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    | 
| ID64[(9)](#ID64-at-SRM-From-7101-7110)                          |    |    |    | U  |    |    |     |    |    |    |    |    | 
| **Flow 2**                                                       |    |    |    |    |    |    |     |    |    |    |    |    | 
| ID26[(7)](#ID64-at-7107-7110)                                    |    |    | U  | U  |    |    | I   |    |    |    |    |    |
| StorageSender[(8)](#Storage-Sender-at-7107-7110)                |    |    |    | U  |    |    | U   |    |    |    |    |    |        
| ID25[(9)](#ID25-at-7107-7110)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64[(10)](#ID64-at-STV-From-7107-7110)                          |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID64[(11)](#ID64-at-SRM-From-7107-7110)                          |    |    |    | U  |    |    |     |    |    |    |    |    |
| ID26[(12)](#ID26-at-7211-7214)                                   |    |    | U  | U  |    |    | I   |    |    |    |    |    |       
| StorageSender[(15)](#Storage-Sender-at-7211-7214)                |    |    |    | U  |    |    | U   |    |    |    |    |    |
| ID25[(13)](#ID25-at-7211-7214)                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |
| ID64[(14)](#ID64-at-SRM-From-7211-7214)                         |    |    |    | U  |    |    |     |    |    |    |    |    |
| **Last Process**                                                 |    |    |    |    |    |    |     |    |    |    |    |    |    
| ID33[(15)](#)                                                    | U  | I  | U  | D  | U  | I  |     |    | U  | I  |    |    |
| **Host Communication**                                           |    |    |    |    |    |    |     |    |    |    |    |    |
| Internal Location Transfer Result[(9)](#Internal-Location-Transfer-Result)      |    |    |    |    |    | U  |     |    |    |    |    |    |    |    |     |

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
If the station mode is **Retrieval Mode**, change the mode of the station to **Storage mode**.
::: mermaid
flowchart LR
    input[
        Operators pressed the storage mode button on the operation box.      
    ]

    id61msg("
     ID63
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id61msg-->id63process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>

####<span style="color:skyblue; font-weight:bold">DMStation</span>
**CURRENT_MODE**: 1:Storage Mode

# QC Work - Retrieval for QC Start - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCWorkSettingSCH &nbsp;</span>

-

This screen is mainly used when the pallet needs to be inspected.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess QC Work - Retrieval for QC Start


