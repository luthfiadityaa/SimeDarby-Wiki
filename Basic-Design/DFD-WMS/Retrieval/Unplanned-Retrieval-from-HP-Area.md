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
- **TTSN** : DMToSTATION 

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|
| **Only For 1301 & 1302**                                                        |    |    |    |    |    |    |     |    |    |    |    |    |
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |  
| **Directly For 1205,1206,1207,1208,1209**                                       |    |    |    |    |    |    |     |    |    |    |    |    | 
| Inquiry Retrieval Setting - Set(F2) [(1)](#Inquiry-Retrieval-Setting---Set(F2)) | I  |    | S  | I  | S  |    |     | S  | S  |    | S  |    |  
| RetrievalSender[(2)](#Retrieval-Sender)                                         |    |    | I  | I  | I  |    | I   |    |    |    |    | S  |  
| ID32[(3)](#ID32)                                                                | U  |    | U  | U  | U  |    | U   | U  | U  |    |    |    |    
| ID33[(4)](#ID33)                                                                |    |    |    | U  |    |    | D   |    |    |    |    |    |     
| ID64[(5)](#ID64)                                                                | U  | I  | U  | D  | U  | I  |     |    | U  | I  |    |    |
| ID68[(6)](#ID68)                                                                | U  | I  | U  | D  | U  | I  |     |    | U  | I  |    |    |
| ID26[(7)](#ID26)                                                                | U  | I  | U  | D  | U  | I  |     |    | U  | I  |    |    |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |    |    |    |     |    |    |    |    |    | 

#Inquiry Retrieval Setting - Set(F2)
 ![==image_0==.jpg](/.attachments/==image_0==-58f99576-be7b-41a4-bc02-6cef92ce5b43.jpg) 