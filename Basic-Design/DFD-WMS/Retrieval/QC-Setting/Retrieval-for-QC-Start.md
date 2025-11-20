[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR

P1[FROM AISLE STATION - 9001, 
9002 , 9003, 9004, 9005, 
9006, 9007, 9008, 9009, 
9010, 9011, 9012, 9013, 
9014]-->P2[RetrievalSender]-->P21[ID12]-->P3[ID32]-->P4[ID33]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR

Cond1{Flow?} 

Cond1{Flow?}--> |Flow 1: Thorugh crane 7-10.
9001, 9002, 9003, 9004, 9005, 9006, 9011, 9012, 9013, 9014.| P5[ID64]--> |7207-7210| P8[ID26]-->P9[Retrieval Sender]-->P10[ID25]--> |SRM| P11[ID64]--> |STV| P12[ID64]--> P7[ID68]
Cond1{Flow?}--> |Flow 2: Not through crane 7-10.
9007, 9008, 9009, 9010| P6[ID64]--> P7[ID68]-->P81[ID26]-->P13[To Station 1303]
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
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>  
*   **JOB_TYPE**: 23:Unplanned Retrieval    
*   **STATUS_FLAG**: 0:Not Started    
*   **PLAN_UKEY**: Sequence Object    
*   **STOCK_ID**: DNSTOCK.STOCKID    
*   **PLAN_AREA_NO**: DNSTOCK.AREANO   
*   **PLAN_LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
*   **PLAN_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **PLAN_QTY**: DNSTOCK.STOCK_QTY    
*   **BATCH_NO**: DNSTOCK.BATCH_NO    
*   **MATERIAL_CODE**: DNSTOCK.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **MATERIAL_TYPE**: DMITEM.MATERIAL_TYPE   
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG    
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY  
*   **BCR_DATA**: DNPALLET.BCR_DATA  
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: Login Info     
*   **TERMINAL_NO**: Login info
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME      
*   **RETRIEVAL_DETAIL**: 1:Unit Retrieval   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY  
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG        
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: QCSettingSCH

#Retrieval Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalSender&nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
")]

retrievalsender-update[("
DNCARRYINFO
DNPALLET
")]

id12msg("
ID 12
")

retrievalsender-input-->retrievalsender--> |UPDATE| retrievalsender-update
retrievalsender--> |SendText| id12msg
:::

All Carton Retrieval operation at Ambient or Tempering will be retrieved to Station 1303 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 3:Reserved for Retrieval
* **ALLOCATION_FLAG**: 1:Allocated
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: RetrievalSender

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNSTOCK.PALLET_ID    
*   **WORK_TYPE**: 23:Unplanned Retrieval    
*   **CMD_STATUS**: 1:Started    
*   **RESTORING_FLAG**: 0:Not Restore to Original Location  
*   **WORK_NO**: Sequence Object    
*   **RETRIEVAL_STATION_NO**: DNSTOCK.LOCATION_NO
*   **SOURCE_STATION_NO**: DNPALLET.CURRENT_STATION_NO    
*   **DEST_STATION_NO**: Value from screen (Station) **<span style="color:green;">1303</span>**     
*   **PRIORITY**: Value from screen ⟶ **<span style="color:green;">(1:Urgent, 2:Normal)</span>**
*   **CANCEL_REQUEST**: 0:Not Requested    
*   **SCHEDULE_NO**: Sequence Object    
*   **CARRY_FLAG**: 2:Retrieval
*   **CANCEL_REQUEST**: 0:Not requested
*   **AISLE_STATION_NO**: DMSHELF.PARENT_STATION_NO
*   **END_STATION_NO**: DNCARRYINFO.DEST_STATION_NO  
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: RetrievalSender

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>   
* **STATUS_FLAG**: 1:Working  
* **SYSTEM_CONN_KEY**: DNCARRYINFO.CARRY_KEY

#ID12
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.id.send.As21Id12&nbsp;</span>

::: mermaid
flowchart LR

id26msg("
ID12
")

id26-insert[("
DNCARRYINFO
")]

id26-update[("
DNPALLET
")]

inoutstationoperator[InOutStaionOperator]

id26msg-->inoutstationoperator
inoutstationoperator--> |INSERT| id26-insert
inoutstationoperator--> |UPDATE| id26-update
:::

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 4:Being retrieved
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name