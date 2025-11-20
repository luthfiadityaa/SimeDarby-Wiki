[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9001, 
9002 , 9003, 9004, 9005, 
9006, 9007, 9008, 9009, 
9010, 9011, 9012, 9013, 
9014]-->P21[ID12]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1303] 
:::

#<span style="color:skyblue; font-weight:bold">Retrieval for QC Start database flow</span>
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
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|----|


#<span style="color:skyblue; font-weight:bold">Mode Change Station</span>
##ID63
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id63Process&nbsp;</span>

If the station mode is **Storage Mode**, change the mode of the station to **Retrieval mode**.

::: mermaid
flowchart LR
    input[
        Operators pressed the retrieval mode button on the operation box.      
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

# Retrieval for QC Start - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.retrieval.QCSettingSCH &nbsp;</span>
 ![==image_0==.jpg](/.attachments/==image_0==-385c044c-0af1-4f9f-b8bd-8c195cfba22a.jpg) 

This screen is mainly used when the pallet needs to be inspected.

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Retrieval for QC Start
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- The station mode must be <span style="color:green; font-weight:bold">Retrieval mode</span>.
- The Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- The Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Target pallet of Stock Status is <span style="color:green; font-weight:bold">UU</span>.
- Only the pallets with tempering flag <span style="color:green; font-weight:bold">Reached</span> are applicable for QC settings.
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

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

    tableList-insert[("
        DNWORKINFO
        DNWORKLIST
    ")]

     tableList-update[("
        DNSTOCK
    ")]

     tableList-select[("
        DNSTOCK
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]

    className[QCSettingSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className
    className --> |UPDATE| tableList-update

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">QI</span>
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **JOB_NO**: Sequence Object    
*   **SETTING_UNIT_KEY**: Sequence Object    
*   **COLLECT_JOB_NO**: Sequence Object    
*   **JOB_TYPE**: 23:Unplanned Retrieval    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNSTOCK.LOCATION_NO
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **BATCH_NO**: DNSTOCK.BATCH_NO    
*   **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE 
*   **PLAN_QTY**: Value from screen (Retrieval Qty / Stock Qty)    
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNPALLET.BCR_DATA  
*   **STORAGE_LOCATION_TO**: Value from Screen (To Location)
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName
####<span style="color:skyblue; font-weight:bold">DNWorkList</span>