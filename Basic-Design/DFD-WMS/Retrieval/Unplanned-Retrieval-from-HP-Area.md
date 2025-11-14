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

* **Operation Name**
  - **I** : INSERT  
  - **U** : UPDATE  
  - **D** : DELETE
  - **S** : SELECT 

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                                                             |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN| 
|-------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|
| **For 1301 & 1302**                                                     |    |    |    |    |    |    |     |    |    |    |    | U  |
| ID63 [(1)](#ID63)                                                       |    |    |    |    |    |    |     |    |    |    |    | U  |  
| For 1301 & 1302                                                         |    |    |    |    |    |    |     |    |    |    |    | U  | 
| Inquiry Retrieval Setting - Set(F2) [(1)](#Inquiry-Retrieval-Setting---Set(F2)) | I  |    |    |    |    |    |     |    |    |    | S  |    |  
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
| Unplanned Storage and Retrieval Result[(19)](#Unplanned-Storage-and-Retrieval-Result)     |    |    |    |    |    |    |     |    |    |    |    |    | 

#Inquiry Retrieval Setting - Set(F2)
 ![==image_0==.jpg](/.attachments/==image_0==-58f99576-be7b-41a4-bc02-6cef92ce5b43.jpg) 