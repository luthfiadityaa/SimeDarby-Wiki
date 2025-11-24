[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR

P1[Select the List 
from Screen]-->P2[Extend Tempering Period]
:::

#<span style="color:skyblue; font-weight:bold">Extend Tempering Period database flow</span>
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

| Action Name                                                                     |WRKI|WRKL|PLLT|CRYI|STCK|HSTS|ARRVL|WRHS|SHLF|STCH|ITEM|STSN|TTSN|OPRD|INOUT| 
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|-----|
| Extend Tempering Period - Set(F2) [(1)](#Update-QC-Status--Set(F2))                    |    |    |    |    | U  |    |     |    |    | I  |    |    |    |    |     |

# Extend Tempering Period - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCSettingSCH &nbsp;</span>
![Picture1.png](/.attachments/Picture1-d36a622b-7f90-4076-8fb0-a10e85d67eb4.png)
  
This screen is mainly used when it is decided to extend the tempering period of stock after QC operation.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole process QC Status Update
- Once the QC status is updated from **<span style="color:green; font-weight:bold">QI→UU/BS</span>**, it cannot turn back to **<span style="color:green; font-weight:bold">QI</span>**. **<span style="color:green; font-weight:bold">UU</span>** can be turned to **<span style="color:green; font-weight:bold">BS</span>** and `vice-versa`, but not to **<span style="color:green; font-weight:bold">QI</span>**.

::: mermaid
flowchart LR
    input[
        Area
        QC Station
        Pallet #
        Material Code
        Batch #
        Stock Status
        Tempering Flag
        QC Check Flag
        Loc #
        Material Name
        Stock Qty
        Storage Date/Time
        Tempering Period
        QC Duration
    ]

     tableList-update[("
        DNSTOCK
    ")]

     tableList-insert[("
        DNSTOCKHISTORY
    ")]

    input-->className[QCSettingSCH]

    className --> |UPDATE| tableList-update
    className --> |INSERT| tableList-insert

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

<hr>

##<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-db552ec6-fdf6-4fac-a11d-c741bbe387b8.png)
  
*) After setting, **<span style="color:green; font-weight:bold">Tempering Flag</span>** will be changed from `Reached` to `Not Reached`.

<hr>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **QC_DURATION**: DNSTOCK.QC_DURATION + Value from screen
* **TEMPERING_FLAG** : <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span> 
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
* **TEMPERING_FLAG**: <span style="color:yellow; font-weight:bold; background-color:grey">0: Not Reached</span> 
* **QC_DURATION**: DNSTOCK.QC_DURATION
* **TERMINAL_NO**: Login info
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: ClassName
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: ClassName
  
<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD - QC Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5788)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Extend Tempering Period](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/906/Extend-Tempering-Period)