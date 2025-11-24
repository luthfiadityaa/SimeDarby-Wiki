[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR

P1[Select the Pallet Number 
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
| Update QC Status - Set(F2) [(1)](#Update-QC-Status--Set(F2))                    | I  | I  | S  | I  | U  |    |     | S  | S  |    | S  | S  | S  |    |     |
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| QC Status Update Result[(2)](#QC-Status-Update-Result)      |    |    |    |    |    | U  |     |    |    |    |    |    |    |    |     |

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

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

###<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
* **REGIST_DATE**
* **WORK_DAY**:
* **INC_DEC_TYPE**: 2:Stock Decrease (Retrieval)
* **JOB_TYPE**: 40:Inventory Check (AS/RS)
* **STOCK_ID**: DNSTOCK.STOCK_ID
* **PLAN_AREA_NO**: DNSTOCK.AREANO
* **PLAN_LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
* **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE
* **MATERIAL_NAME**: DMITEM.MATERIAL_NAME
* **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE
* **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
* **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG
* **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
* **QC_DURATION**: DNSTOCK.QC_DURATION
* **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG
* **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
* **BCR_DATA**: DNPALLET.BCR_DATA
* **STORAGE_LOCATION_FROM**: <span style="color:green; font-weight:bold">FGW1 or FGW2</span>
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
* **TERMINAL_NO**: Login info
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: ClassName
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: ClassName
  
#QC Status Update Result
- [Internal Location Transfer Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)

<hr>

#<span style="color:skyblue; font-weight:bold">User Story</span>
- [DFD - QC Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5788)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Extend Tempering Period](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/906/Extend-Tempering-Period)