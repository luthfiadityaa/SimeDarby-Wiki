#<span style="color:skyblue; font-weight:bold">Retrieval from 1210</span>
![image.png](/.attachments/image-b5fcb612-8067-43e8-a98a-ac23b8402498.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010]-->P2[RetrievalSender]-->P3[ID66]-->P4[ID12]-->P5[ID64]-->P7[ID26]-->P8[To STATION - 1210] 
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
        Operators pressed the storage mode button on the operation box.      
    ]

    id66msg("
     ID66
    ")
    tableList-update[("
        DMSTATION
    ")]

    input -->id66msg-->id63process--> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::


#ID12

#ID64

#ID26