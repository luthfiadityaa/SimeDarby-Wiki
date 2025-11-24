[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR

P1[Select the List 
from Screen]-->P2[Update To QC Status]-->P21[QC Status Update Result]
:::

#<span style="color:skyblue; font-weight:bold">Update QC Status database flow</span>
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
| Update QC Status - Set(F2) [(1)](#Update-QC-Status--Set(F2))                    |    |    |    |    | U  |    |     |    |    | I  |    |    |    |    |     |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| QC Status Update Result[(2)](#QC-Status-Update-Result)                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |

# Update QC Status - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCSettingSCH &nbsp;</span>
![Picture1.png](/.attachments/Picture1-d36a622b-7f90-4076-8fb0-a10e85d67eb4.png)
  
This screen is mainly used when it is decided to update QC Status of stock after QC operation.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole process QC Status Update

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

#<span style="color:skyblue; font-weight:bold">Storage Flag Condition</span>
![image.png](/.attachments/image-ce857ad3-8f9d-4967-b554-217332bb058c.png)
  
*) If QC result requires more tempering, refer Re-QC.

<hr>

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **STOCK_STATUS**: Value from Screen -> **<span style="color:green; font-weight:bold">QI: Quality Inspection or BS:Block Stock</span>**
* **TEMPERING_FLAG** : <span style="color:yellow; font-weight:bold; background-color:grey">1: Reached</span> 
* **QC_CHECK_FLAG**: **<span style="color:green; font-weight:bold">Done</span>**
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

###<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
* **JOB_TYPE**: <span style="color:green; font-weight:bold">91:QC Status Update</span>
* **STOCK_STATUS_FROM**: DNSTOCK.STOCK_STATUS
* **STOCK_STATUS_TO**: Value from Screen -> **<span style="color:green; font-weight:bold">QI: Quality Inspection or BS:Block Stock</span>**
* **QC_CHECK_FLAG**: **<span style="color:green; font-weight:bold">Done</span>**
* **TERMINAL_NO**: Login info
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: ClassName
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: ClassName
  
#QC Status Update Result
- [QC Status Update Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)

<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD - QC Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5788)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Extend Tempering Period](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/906/Extend-Tempering-Period)