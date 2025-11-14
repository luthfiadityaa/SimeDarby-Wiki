#<span style="color:skyblue; font-weight:bold">Retrieval from 1301, 1302, 1205, 1206, 1207, 1208, 1209</span>
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 9002 , 9003, 9004, 9005, 9006, 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1205, 1206, 1207, 1208, 1209, 1301, 1302] 
:::

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
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

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|
| **Only For 1301 & 1302**                                                        |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |
| **Directly For 1205,1206,1207,1208,1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Inquiry Retrieval Setting - Set(F2) [(1)](#Inquiry-Retrieval-Setting---Set(F2)) | I  |    | S  | I  | S  |    |     | S  | S  |    | S  |    | S  |    |
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | U  | U  |    |    |     |    |    |    |    |    |    |    |  
| ID32[(3)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |    
| ID33[(4)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     
| ID64[(5)](#ID64)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |
| ID68[(6)](#ID68)                                                                |    |    |    |    |    |    |     |    |    |    |    |    |    | I  |
| ID26[(7)](#ID26)                                                                | U  | I  | U  | D  | U  | I  |  I  |    | U  | I  |    |    |    |    |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    | U  |    |    |     |    |    |    |    |    | 

#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.
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
**CURRENT_MODE**: 2: Retrieval Mode

#Inquiry Retrieval Setting - Set(F2)
 ![==image_0==.jpg](/.attachments/==image_0==-58f99576-be7b-41a4-bc02-6cef92ce5b43.jpg) 