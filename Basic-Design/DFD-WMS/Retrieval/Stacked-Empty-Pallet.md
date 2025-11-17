#<span style="color:skyblue; font-weight:bold">Retrieval from 1210</span>
![image.png](/.attachments/image-b5fcb612-8067-43e8-a98a-ac23b8402498.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010]-->P2[ID66]-->P3[RetrievalTriggerAllocator]-->P4[RetrievalSender]-->P5[ID64]-->P7[ID26]-->P8[To STATION - 1210] 
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
**Abbreviation:**
- **WRKI** : DNWORKINFO  
- **WRKL** : DNWORKINFOLIST  
- **PLLT** : DNPALLET  
- **CRYI** : DNCARRYINFO  
- **STCK** : DNSTOCK  
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

#ID66

::: mermaid
flowchart LR
    input[
        Automated Retrieval Trigger.      
    ]

    id66msg("
     ID66
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id66msg-->id66process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DMStation</span>
**STATION_NO**: 1210
**RETRIEVAL_TRIGGER_REQUEST**: 2:Requested by AGC
**LAST_UPDATE_PNAME**: Id66Process

#ID12

#ID64

#ID26