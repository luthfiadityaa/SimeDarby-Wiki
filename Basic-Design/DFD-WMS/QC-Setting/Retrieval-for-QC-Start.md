[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1 — Direct retrieval (9007-9010 to 1303)**
::: mermaid
flowchart LR

P1[FROM AISLE STATION - 9007, 
9008, 9009, 9010]-->P2[RetrievalSender]-->P21[ID12]-->P3[ID32]-->P4[ID33]-->P5[To Station 1303]
:::

**Stage 1a — Cross-warehouse retrieval (9001-9006, 9011-9014 to 1303 via 720x)**
::: mermaid
flowchart LR

P1[FROM AISLE STATION - 9001-9006, 
9011-9014]-->P2[RetrievalSender
destDetermine: no direct route
handleIntermediateRetrieval
selectLeastBusy 720x
check 1303 capacity]-->P21[ID12 to 720x]-->P3[ID32]-->P4[ID33]
:::

**Stage 2 — Through crane 7-10 (cross-warehouse only)**
::: mermaid
flowchart LR

P5[ID64]--> |7207-7210| P8[ID26]-->P9[AsrsInboundStationOperator
RETRIEVAL→DIRECT_TRAVEL
dest=1303]-->P10[StorageSender
ID05]-->P11[ID25]--> |SRM| P12[ID64]--> |STV| P13[ID64]--> P7[ID68]
P7[ID68]-->P71[ID45]-->P81[ID26]-->P14[To Station 1303]
:::

#<span style="color:skyblue; font-weight:bold">DMStation Settings — Station 1303 (QC Station)</span>

Station 1303 requires specific configuration for the QC retrieval and return-storage flow:

| Setting | Value | Description |
|---------|-------|-------------|
| **station_type** | 3 (INOUT) | Bi-directional station for retrieval and re-storage |
| **operation_display** | **3 (INSTRUCTION_AUTO_COMP)** | Enables ID68 → OperationDisplay → ID45 → ID26 flow. MC/WareNavi decides work completion timing based on operator input from QC Work screen. Without this, DIRECT_TRAVEL pallets are immediately removed by LoadRemover |
| **restoring_instruction** | **1 (WN_STORAGE_SEND)** | Required for StorageSender to accept return-storage carries with `cmd_status=ARRIVAL`. Without this (value 0), StorageSender only accepts `cmd_status=START` and return carries are stuck |
| **max_instruction** | **1** | StorageSender requires `max_instruction >= 1` to send carries from this station. With 0, StorageSender always skips (0 <= count is always true → OVER) |
| **max_pallet_qty** | **1** | Deadlock prevention — only 1 pallet heading to 1303 at a time. RetrievalSender checks `countCarriesHeadingToStation("1303")` before routing cross-warehouse carries |
| **wh_station_no** | 9200 | Station belongs to warehouse 9200. **Note:** return-storage uses `pallet.wh_station_no` (not station wh) for correct cross-warehouse routing |

###<span style="color:skyblue; font-weight:bold">Operation Display Modes</span>

| Value | Name | Behavior at ID26 (DIRECT_TRAVEL + end=1303) |
|-------|------|----------------------------------------------|
| 0 | NONE | `LoadRemover.remove()` — pallet data deleted immediately (reject flow) |
| 1 | DISP_ONLY | Same as NONE |
| **2** | **INSTRUCTION_HARD_SWITCH_COMP** | Operator presses physical button. `work_type=40` → ReturnStorageManager, `work_type=26` → LoadRemover |
| **3** | **INSTRUCTION_AUTO_COMP** | MC/WareNavi decides completion. Same routing logic as value 2. **Used for 1303** |

With `operation_display=3`, the flow at station 1303 is:
```
ID68 → OperationDisplay created → Work Display shows task (carry_key links to DNWorkInfoList)
  → Operator inspects pallet (QC Work screen — not yet implemented)
  → WareNavi sends ID45 (payout type decided by operator input)
  → AGC sends ID26
     → load=1 (pallet present): InOutStationOperator.arrival()
        → RETRIEVAL: updateArrival → ReturnStorageManager → return to ASRS
        → DIRECT_TRAVEL + work_type=40: same as RETRIEVAL (QC cross-warehouse)
        → DIRECT_TRAVEL + work_type=26: LoadRemover.remove() (reject)
     → load=0 (pallet taken out): LoadRemover.remove()
```

###<span style="color:skyblue; font-weight:bold">SQL</span>

```sql
-- QCRetrievalRoutes.sql
UPDATE DMStation SET max_pallet_qty = 1 WHERE station_no = '1303';
UPDATE DMStation SET operation_display = '3' WHERE station_no = '1303';
UPDATE DMStation SET restoring_instruction = '1' WHERE station_no = '1303';
UPDATE DMStation SET max_instruction = 1 WHERE station_no = '1303';
```

#<span style="color:skyblue; font-weight:bold">DMRouteId — QC Retrieval Routes</span>

Routes required for QC retrieval from ASRS aisles to station 1303.

###<span style="color:skyblue; font-weight:bold">Direct Retrieval: 9007-9010 → 1303</span>

Aisles 9007-9010 (FGW1 Ambient, wh=9200) have a **direct physical path** to station 1303 via STV.

```
9007 → SRM(SRA00701) → conveyor(1AB070) → STV(SLA00201/8101) → conveyor(1BB520_02) → 1303
9008 → SRM(SRA00801) → conveyor(1AB090) → STV(SLA00202/8101) → conveyor(1BB520_02) → 1303
9009 → SRM(SRA00901) → conveyor(1AB110) → STV(SLA00203/8101) → conveyor(1BB520_02) → 1303
9010 → SRM(SRA01001) → conveyor(1AB130) → STV(SLA00201/8101) → conveyor(1BB520_02) → 1303
```

RetrievalSender sends ID12 directly from aisle to 1303. No intermediate station needed.

###<span style="color:skyblue; font-weight:bold">Cross-warehouse: 9001-9006 → 7207-7210 (intermediate)</span>

Aisles 9001-9006 (FGW2 Tempering, wh=9100) have **no direct path** to 1303. Pallets must go through HP BCR stations 7207-7210 as intermediate, then forward to 1303.

**Leg 1 — Aisle to 720x** (24 routes, already existed):
```
9001 → SRM(SRA00101) → STV(SLA00101/8102) → 7207(1AA070)
9001 → SRM(SRA00101) → STV(SLA00101/8102) → 7208(1AA090)
9001 → SRM(SRA00101) → STV(SLA00101/8102) → 7209(1AA110)
9001 → SRM(SRA00101) → STV(SLA00101/8102) → 7210(1AA130)
... (same pattern for 9002-9006)
```

RetrievalSender selects the **least-busy** 720x station (counts active carries per station). The carry is updated: `dest=720x, end stays 1303`.

###<span style="color:skyblue; font-weight:bold">Cross-warehouse: 9011-9014 → 7207-7210 (intermediate)</span>

Aisles 9011-9014 (PM Ambient, wh=9200) also have **no direct path** to 1303. Same intermediate routing via 7207-7210.

**Leg 1 — Aisle to 720x** (16 routes, NEW):
```
9011 → SRM(SRA01101) → conveyor(1AA160) → conveyor(1BA220) → STV(SLA00111/8102) → 720x
9012 → SRM(SRA01201) → conveyor(1AA180) → conveyor(1BA240) → STV(SLA00112/8102) → 720x
9013 → SRM(SRA01301) → conveyor(1AA200) → conveyor(1BA260) → STV(SLA00113/8102) → 720x
9014 → SRM(SRA01401) → conveyor(1AA220) → conveyor(1BA280) → STV(SLA00101/8102) → 720x
```

###<span style="color:skyblue; font-weight:bold">Forwarding: 7207-7210 → 1303</span>

**Leg 2 — 720x to 1303** (4 routes, NEW):

After AsrsInboundStationOperator at 720x converts the carry (RETRIEVAL → DIRECT_TRAVEL, work_type stays 40), StorageSender sends ID05 from 720x to 1303 via these routes.

```
7207 → SRM(SRA00701/9007) → conveyor(1AB070) → STV(SLA00201/8101) → conveyor(1BB520_02) → 1303
7208 → SRM(SRA00801/9008) → conveyor(1AB090) → STV(SLA00202/8101) → conveyor(1BB520_02) → 1303
7209 → SRM(SRA00901/9009) → conveyor(1AB110) → STV(SLA00203/8101) → conveyor(1BB520_02) → 1303
7210 → SRM(SRA01001/9010) → conveyor(1AB130) → STV(SLA00201/8101) → conveyor(1BB520_02) → 1303
```

###<span style="color:skyblue; font-weight:bold">Virtual Routes: 9001-9006/9011-9014 → 1303 (scheduler validation)</span>

Virtual DMRouteId entries exist **only** for `WebUnplannedRetrievalScheduler` route validation. They have minimal DMRouteDetail (source station only) so `RouteDB.check()` returns ACTIVE. RetrievalSender **never uses** these routes — it intercepts before the route check via `isIntermediateRetrievalNeeded()` and redirects to 720x.

```
9001->1303v  9002->1303v  9003->1303v  9004->1303v  9005->1303v  9006->1303v   (6 routes)
9011->1303v  9012->1303v  9013->1303v  9014->1303v                              (4 routes)
```

Without these, the scheduler cannot create DNCarryInfo for cross-warehouse pallets (route check fails → carry never created → RetrievalSender never gets a chance to apply intermediate routing).

###<span style="color:skyblue; font-weight:bold">Route Summary</span>

| Route | Count | Purpose | Used by |
|-------|-------|---------|---------|
| 9007-9010 → 1303 | 4 | Direct QC retrieval | RetrievalSender (ID12) |
| 9001-9006 → 7207-7210 | 24 | Cross-WH leg 1 (already existed) | RetrievalSender (ID12 to 720x) |
| 9011-9014 → 7207-7210 | 16 | Cross-WH leg 1 (NEW) | RetrievalSender (ID12 to 720x) |
| 7207-7210 → 1303 | 4 | Cross-WH leg 2 (NEW) | StorageSender (ID05 from 720x) |
| 9001-9006 → 1303v | 6 | Virtual — scheduler only (NEW) | WebUnplannedRetrievalScheduler |
| 9011-9014 → 1303v | 4 | Virtual — scheduler only (NEW) | WebUnplannedRetrievalScheduler |
| **Total new** | **30** | | `QCRetrievalRoutes.sql` |

###<span style="color:skyblue; font-weight:bold">Return Storage Routes (1303 → ASRS)</span>

After QC work, ReturnStorageManager converts carry to STORAGE with `dest=pallet.wh_station_no`. StorageSender uses LocationManager to find an empty shelf and routes back to the aisle.

| Route | Count | Purpose |
|-------|-------|---------|
| 1303 → 9001-9006 | 6 | Return to WH 9100 (Tempering) |
| 1303 → 9007-9010 | 4 | Return to WH 9200 (Ambient/PM) |
| 1303 → 7101-7110 | 10 | Storage outbound via OP BCR |

Note: No routes 1303 → 9011-9014 (no physical path). PM pallets returning from QC go to aisles 9007-9010 via WH 9200.

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
|---------------------------------------------------------------------------------|----|----|----|----|----|----|-----|----|----|----|----|----|----|----|-----|
| ID63 [(1)](#ID63)                                                               |    |    |    |    |    |    |     |    |    |    |    | U  |    |    |     |
| Retrieval for QC Start - Set(F2) [(1)](#Retrieval-for-QC-Start---Set(F2))       | I  | I  | S  | I   | U  |    |     | S  | S  | I  | S  | S  | S  |    |     |
| RetrievalSender[(3)](#Retrieval-Sender)                                         | U  |    | U  | U  |    |    |     |    |    |    |    |    |    |    |     |  
| ID12[(2)](#ID12)                                                                |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     | 
| ID32[(4)](#ID32)                                                                |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |    
| ID33[(5)](#ID33)                                                                |    |    |    | U  |    |    |     |    | U  |    |    |    |    |    |     |     
| **Thorugh crane 7-10**                                                  |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID64[(6)](#ID64-at-STV-from-(9001-9006-&-9011-9014))                            |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| ID26[(7)](#ID26-at-7207-7210)                                                   |    |    | U  | U  |    |    | I   |    |    |    |    |    |    |    |     |
| Retrieval Sender[(8)](#Retrieval-Sender-at-7207-7210)                           |    |    |    | U  |    |    | U   |    |    |    |    |    |    |    |     |
| ID25[(9)](#ID25-at-7207-7210)                                                   |    |    |    | U  |    |    | D   |    |    |    |    |    |    |    |     |
| ID64[(10)](#ID64-at-SRM-from-7207-7210)                                         |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| ID64[(11)](#ID64-at-STV-from-9007-910)                                          |    |    |    | U  |    |    |     |    |    |    |    |    |    |    |     |
| **Last Process:**                                                               |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| ID68[(7)](#ID68)                                                                | S  |    |    | S  |    |    |     |    |    |    |    |    |    | I  |     |
| ID45[(8)](#ID45)                                                                |    |    |    |    |    |    |     |    |    |   |    |    |    |    |   | 
| ID26[(9)](#ID26)                                                                | U  | U  | U  | U  |    | I  |  I  |    | U  | U  |    |    |    |    | I   | 
| **Host Communication**                                                          |    |    |    |    |    |    |     |    |    |    |    |    |    |    |     |
| Internal Location Transfer Result[(10)](#Internal-Location-Transfer-Result)      |    |    |    |    |    | U  |     |    |    |    |    |    |    |    |     |
 

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
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.qcsetting.qcstartretrieval.QCStartSCH &nbsp;</span>

![image.png](/.attachments/image-83b7e25b-6301-40cc-82e2-9e8cc2d635a8.png)

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
        DNSTOCKHISTORY
        DNCARRYINFO
    ")]

     tableList-update[("
        DNSTOCK
        DNPALLET
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
####<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
* **STOCK_STATUS_FROM**: DNSTOCK.STOCK_STATUS
* **STORAGE_LOCATION_FROM**: DNSTOCK.STORAGE_LOCATION
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **STOCK_STATUS**: <span style="color:green; font-weight:bold">QI</span>
* **STORAGE_LOCATION**: <span style="color:green; font-weight:bold">QC01</span>
* **LAST_UPDATE_DATE**: SYSTIMESTAMP    
* **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>  
*   **JOB_TYPE**: 40:Inventory Check (AS/RS)    
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
*   **STORAGE_LOCATION**: <span style="color:green; font-weight:bold">QC01</span>
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
*   **RETRIEVAL_DETAIL**: 0:Inventory Check   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY  
*   **TEMPERING_PERIOD**: DNSTOCK.TEMPERING_PERIOD
*   **TEMPERING_FLAG**: DNSTOCK.TEMPERING_FLAG      
*   **EXPIRY_DAYS**: DNSTOCK.EXPIRYDATE
*   **QC_DURATION**: DNSTOCK.QC_DURATION
*   **QC_CHECK_FLAG**: DNSTOCK_QC_CHECK_FLAG        
*   **STORAGE_LOCATION**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: QCSettingSCH

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 3:Reserved for Retrieval
* **ALLOCATION_FLAG**: 1:Allocated
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: RetrievalSender

####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
*   **CARRY_KEY**: Sequence Object    
*   **PALLET_ID**: DNSTOCK.PALLET_ID    
*   **WORK_TYPE**: 40:Inventory Check (AS/RS)    
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
*   **RETRIEVAL_DETAIL**: 0:Inventory Check
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: RetrievalSender

#Retrieval Sender
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.RetrievalSender&nbsp;</span>

::: mermaid
flowchart LR

retrievalsender-input[("
DNCARRYINFO
DMSTATION
")]

retrievalsender-update[("
DNCARRYINFO
DNPALLET
")]

id12msg("
ID 12
")

retrievalsender-input--> P1[retrievalsender→process]-->P2[SendCarry→getSendCarryArray]--> |UPDATE| retrievalsender-update
P2--> |SendText| id12msg

 click id12msg "#" "Go to ID12"
 style id12msg fill:#00cc66,stroke:#006633,color:#ffffff
:::

All Pallet Retrieval operation at Ambient or Tempering will be retrieved to Station 1301, 1302, 1205, 1206, 1207, 1208, 1209 where the related DNCARRYNFO data will be processed in Retrieval Sender. ID12 will be sent after related tables are updated successfully.

**Cross-warehouse QC retrieval (9001-9006, 9011-9014 → 1303):**

When `destDetermine()` detects no direct route from the pallet's aisle to 1303, it calls `handleIntermediateRetrieval()`:

1. **Check 1303 capacity** — `countCarriesHeadingToStation("1303")` counts all active carries where `dest=1303 OR end=1303`. If count >= `max_pallet_qty` (1), blocks with `wait_reason=DEST_FULL(09)` to prevent deadlock.
2. **Select least-busy 720x** — `selectLeastBusyIntermediate(aisleNo)` finds routes from aisle to 7207-7210, filters by NORMAL/not suspended, counts active carries where `dest=720x OR source=720x`, picks lowest.
3. **Validate route** — confirms DMRouteId exists for aisle → selected 720x.
4. **Update carry** — `dest_station_no` updated to selected 720x, `end_station_no` stays 1303. ID12 sent from aisle to 720x.

| Aisle | Route | Intermediate |
|-------|-------|-------------|
| 9007-9010 | Direct to 1303 (9007->1303 .. 9010->1303) | None needed |
| 9001-9006 | Via 720x (9001->7207 .. 9006->7210) | 7207-7210 (least busy) |
| 9011-9014 | Via 720x (9011->7207 .. 9014->7210) | 7207-7210 (least busy) |

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>
* **STATUS_FLAG**: 4:Being retrieved
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID32
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id32Process&nbsp;</span>

::: mermaid
flowchart LR

id32("
ID 32
")

id32-update[("
DNCARRYINFO
")]

id32-->id32process
id32process--> |UPDATE| id32-update
:::

ID32 sent from AGC to WareNavi indicate AGC responded the retrieval job by WareNavi.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 3:Commanded
* **ERROR_CODE**: 0
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID33
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id33Process&nbsp;</span>

::: mermaid
flowchart LR

id33("
ID 33
")

id33-update[("
DMSHELF
DNCARRYINFO
")]

id33-->id33process
id33process--> |UPDATE| id33-update
:::

ID33 for Retrieval operation which is sent by AGC to WareNavi to notify WareNavi that the Pallet/Bin is out of rack and is being transferred to related Station.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DMShelf</span>
* **STATUS_FLAG**: 0:Empty
* **LAST_UPDATE_DATE**: SYSTIMESTAMP

###<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 5:Retrieval completed
* **RETRIEVAL_STATION_NO**: DMSHELF.STATION_NO
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

<hr>

#<span style="color:skyblue; font-weight:bold">Continue Process with</span>
- ##[Flow 1 : through crane 7-10 from 9001-9006 & 9011-9014 (cross-warehouse via 720x)](#Flow-1)
- ##[Flow 2 : not through crane 7-10 from 9007-9010 (direct to 1303)](#Flow-2)

<hr>

#Flow 1 

**<h2>Through crane 7-10</h2>**

Refer to AGC Linkage Specs: [AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Retrieval Section - 14 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)

##ID64 at STV from (9001-9006 & 9011-9014)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID26 at 7207-7210
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet Arrived at Conveyor Station
"]

id26msg("
ID 26
")

id26-insert[("
DNARRIVAL
")]

id26-update[("
DNCARRYINFO
DNPALLET
")]

id26process[id26process]
asrsoperator[AsrsInboundStationOperator]


buttonclicked --> id26msg
id26msg -->id26process
id26process-->asrsoperator
asrsoperator--> |INSERT| id26-insert
asrsoperator--> |UPDATE| id26-update
:::

When pallet arrives at 7207-7210, AGC sends ID26. `AsrsInboundStationOperator.arrival()` detects `carry_flag=RETRIEVAL` and calls `updateCarryInfoForRetrievalForwarding()` which converts the carry in-place (same carry_key). Then calls `registArrival()` + `carryRequest()` so StorageSender picks up the DIRECT_TRAVEL carry and sends ID05 to 1303.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNArrival</span>
* **ARRIVAL_DATE**: SYSTIMESTAMP
* **STATION_NO**: Arrival Station Number from ID26
* **CARRY_KEY**: 99999999
* **BCR_DATA**: Barcode information from ID26
* **CONTROLINFO**: Control information from ID26
* **SEND_FLAG**: 0:Not sent
* **HEIGHT**: Dimension Information from ID26
* **WIDTH**: Dimension Information From ID26
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

####<span style="color:skyblue; font-weight:bold">DNPallet</span>

*   **CURRENT_STATION_NO**: DNARRIVAL.STATION_NO
*   **REGIST_DATE**: SYSTIMESTAMP
*   **REGIST_PNAME**: ClassName
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: ClassName

####<span style="color:skyblue; font-weight:bold">DNCarryInfo (updated in-place by AsrsInboundStationOperator)</span>

*   **CARRY_FLAG**: <span style="color:green; font-weight:bold">3: Direct Travel</span> (was 2: Retrieval)
*   **WORK_TYPE**: <span style="color:green; font-weight:bold">26: Direct Travel</span> (was 40: Inventory Check)
*   **CMD_STATUS**: <span style="color:green; font-weight:bold">1: Started</span>
*   **SOURCE_STATION_NO**: DNARRIVAL.STATION_NO ⟶ (7207, 7208, 7209, 7210)
*   **DEST_STATION_NO**: **<span style="color:green">1303</span>** (from end_station_no)
*   **END_STATION_NO**: **<span style="color:green">1303</span>**
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: AsrsInboundStationOperator

##StorageSender at 7207-7210
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.transmission.StorageSender&nbsp;</span>

::: mermaid
flowchart LR

storagesender-input[("
DNCARRYINFO
carry_flag=DIRECT_TRAVEL
dest=1303
")]

storagesender-update[("
DNCARRYINFO
cmd_status→WAIT_RESPONSE
")]

id05msg("ID 05")

storagesender-input-->storagesender--> |UPDATE| storagesender-update
storagesender-->id05msg
:::

StorageSender picks up the DIRECT_TRAVEL carry (carry_flag=3, dest=1303) created by AsrsInboundStationOperator. Sends ID05 to transport pallet from 720x to 1303 via routes 7207->1303, 7208->1303, 7209->1303, 7210->1303.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 2:Waiting for response
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: StorageSender

##ID25 at 7207-7210
jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process--> |UPDATE| id25-update
id25process--> |DELETE| id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 3:Commanded
* **ERROR_CODE**: 0
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID64 at SRM from 7207-7210
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |UPDATE| id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID64 at at STV from 9007-9010 
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id64Process &nbsp;</span>

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process--> |DELETE| id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNCarryInfo</span>
* **CMD_STATUS**: 4:Pickup completed
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

#ID68
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.control.Id68Process &nbsp;</span>

::: mermaid
flowchart LR

id68("
ID 68
")

id68-select[("
DNCARRYINFO
DNWORKINFO
")]

id68-insert[("
DNOPERATIONDISPLAY
")]

id68-select-->id68
id68-->id68process
id68process--> |INSERT| id68-insert
:::

ID68 will be sent from AGC to WareNavi to indicate Pallet has arrived to related Station in ASRS. Upon receiving of ID68, insertion of data will be executed.

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNOperationDisplay</span>
* **CARRY_KEY**: MC Key information from ID68
* **STATION_NO**: Station information from ID68
* **ARRIAL_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: Class name
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: Class name

##ID45
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.communication.id.send.As21Id45&nbsp;</span>

::: mermaid
flowchart LR

id45msg("
ID 45
")

buttonlight["
Station Completion button
Light Up
"]

id45msg--> As21Id45
As21Id45 --> buttonlight
:::

Sending of ID45 is sent to AGC when user clicked on **Complete** at Work Display where the Completion button at Control Box will start blinking. If user confirmed re-storing of pallet is safe to proceed, user can click on the blinking Completion button to proceed with transporting of pallet to ASRS.

#ID26
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;jp.co.daifuku.asrs.communication.control.Id26Process&nbsp;</span>

::: mermaid
flowchart LR

buttonclicked["
Pallet arrived at station
"]

buttonclicked2["
Internal Location Transfer Result 
"]

buttonclicked3["
Go to the next process -> QC WORK 
"]

id26msg("
ID 26
")

id26-update[("
DNWORKINFO
DNSHELF
DNCARRYINFO
DNPALLET
DNSTOCK
")]

id26-insert[("
DNARRIVAL
DNWORKLIST
DNHOSTSEND
DNINOUTRESULT
")]

id26process[id26process]
inoutoperator[InOutStationOperator]

buttonclicked-->buttonclicked3
buttonclicked --> id26msg
id26msg -->id26process
id26process-->inoutoperator

inoutoperator--> |INSERT| id26-insert
inoutoperator--> |UPDATE| id26-update
id26-insert-->buttonclicked2 

click buttonclicked2 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result" "Go Internal Location Transfer Result"
click buttonclicked3 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/961/QC-Work-from-Retrieval-for-QC-Start" "Go to QC Work"
style buttonclicked2 fill:#00cc66,stroke:#006633,color:#ffffff
style buttonclicked3 fill:#00cc66,stroke:#006633,color:#ffffff
:::

When the retrieved pallet reaches the QC station, Warenavi sends an ‘Internal Location Transfer’ message to the host system and initiates `QC Work`. 

##<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
###<span style="color:skyblue; font-weight:bold">DNARRIVAL</span>
* **CARRY_KEY**: DNCARRYINFO.CARRY_KEY
* **SEND_FLAG**: 0:Not Sent
* **BCR_DATA**: Arrival Information from ID26 

###<span style="color:skyblue; font-weight:bold">DNCARRYINFO (updated by ReturnStorageManager)</span>
* **CMD_STATUS**: 6:Arrival
* **CARRY_FLAG**: 1:Storage
* **RESTORING_FLAG**: 0:Not Re-store to Original Location
* **SOURCE_STATION_NO**: 1303
* **DEST_STATION_NO**: <span style="color:green; font-weight:bold">DNPALLET.WH_STATION_NO</span> (9100 or 9200 — pallet's original warehouse)
* **END_STATION_NO**: same as DEST_STATION_NO

###<span style="color:skyblue; font-weight:bold">DNPALLET</span>
* **STATUS_FLAG**: 1:Reserved for Storage
* **EMPTY_FLAG**: 0:Normal Pallet

###<span style="color:skyblue; font-weight:bold">DNWorkInfo</span>
*   **STATUS_FLAG**: 4:Completed
*   **SHORTAGE_QTY**: DNWORKINFO.PLAN_QTY - DNWORKINFO.RESULT_QTY
*   **RESULT_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
*   **RESULT_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **STOCK_STATUS**: <span style="color:green; font-weight:bold">QI</span>   
*   **STORAGE_LOCATION**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: Login info
*   **TERMINAL_NO**: Login info
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNStockHistory</span>
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
* **STOCK_STATUS_FROM**: DNSTOCK.STOCK_STATUS
* **STOCK_STATUS_TO**: <span style="color:green; font-weight:bold">QI</span>
* **STORAGE_LOCATION_FROM**: DNSTOCK.STORAGE_LOCATION
* **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
* **TERMINAL_NO**: Login info
* **REGIST_DATE**: SYSTIMESTAMP
* **REGIST_PNAME**: ClassName
* **LAST_UPDATE_DATE**: SYSTIMESTAMP
* **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNWorkList</span>
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **PALLET_ID**: DNPALLET.PALLET_ID    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME    
*   **RETRIEVAL_STATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **RETRIEVAL_DETAIL**: 1:Unit Retrieval   
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY   
*   **STOCK_STATUS**: <span style="color:green; font-weight:bold">QI</span>   
*   **STORAGE_LOCATION**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **USER_NAME**: DCUSER.USER_NAME    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: ClassName    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: ClassName

###<span style="color:skyblue; font-weight:bold">DNHostSend</span>
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY    
*   **JOB_NO**: DNWORKINFO.JOB_NO    
*   **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO    
*   **SETTING_UNIT_KEY**: DNWORKINFO.SETTING_UNIT_KEY    
*   **JOB_TYPE**: DNWORKINFO.JOB_TYPE    
*   **STATUS_FLAG**: DNWORKINFO.STATUS_FLAG    
*   **HARDWARE_TYPE**: DNWORKINFO.HARDWARE_TYPE    
*   **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY    
*   **STOCK_ID**: DNWORKINFO.STOCK_ID    
*   **SYSTEM_CONN_KEY**: DNWORKINFO.SYSTEM_CONN_KEY    
*   **PLAN_DAY**: DNWORKINFO.PLAN_DAY    
*   **BATCH_NO**: DNWORKINFO.BATCH_NO    
*   **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO    
*   **PLAN_LOCATION_NO**: DNWORKINFO.PLAN_LOCATION_NO    
*   **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE    
*   **MATERIAL_NAME**: DMITEM.MATERIAL_NAME    
*   **UOM**: MMATERIALMASTER.UOM    
*   **ENTERING_QTY**: DNSTOCK.STOCK_QTY      
*   **PLAN_QTY**: DNWORKINFO.PLAN_QTY    
*   **RESULT_QTY**: DNWORKINFO.RESULT_QTY    
*   **SHORTAGE_QTY**: DNWORKINFO.SHORTAGE_QTY    
*   **RESULT_AREA_NO**: DNWORKINFO.RESULT_AREA_NO    
*   **RESULT_LOCATION_NO**: DNWORKINFO.RESULT_LOCATION_NO   
*   **STOCK_STATUS_FROM**: DNSTOCK.STOCK_STATUS
*   **STOCK_STATUS_TO**: <span style="color:green; font-weight:bold">QI</span>
*   **STORAGE_LOCATION_FROM**: DNSTOCK.STORAGE_LOCATION
*   **STORAGE_LOCATION_TO**: <span style="color:green; font-weight:bold">QC01</span>
*   **USER_ID**: DNWORKINFO.USER_ID    
*   **TERMINAL_NO**: DNWORKINFO.TERMINAL_NO    
*   **WORK_SECOND**: DNWORKINFO.WORK_SECOND    
*   **USER_NAME**: DCUSER.USERNAME    
*   **REPORT_FLAG**: 0: Not Reported    
*   **REGIST_DATE**: SYSTIMESTAMP    
*   **REGIST_PNAME**: Class name    
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP    
*   **LAST_UPDATE_PNAME**: Class name

###<span style="color:skyblue; font-weight:bold">DNInOutResult</span>
*   **RESULT_KIND**: 2:Retrieval(Stock-)
*   **STATION_NO**: = DNCARRYINFO.DEST_STATION_NO
*   **LOCATION_NO**: DNPALLET.CURRENT_STATION_NO
*   **WH_STATION_NO**: DNPALLET.WH_STATION_NO
*   **AISLE_STATION_NO**: DNCARRYINFO.AISLE_STATION_NO
*   **WORK_TYPE**: DNCARRYINFO.WORK_TYPE
*   **RETRIEVAL_DETAIL**: DNCARRYINFO.RETRIEVAL_DETAIL
*   **WORK_NO**: DNCARRYINFO.WORK_NO
*   **PALLET_ID**: DNPALLET.PALLET_ID
*   **CARRY_KEY**: DNCARRYINFO.CARRY_KEY
*   **RESTORING_FLAG**: DNCARRYINFO.RESTORING_FLAG
*   **WORK_DAY**: DMWARENAVISYSTEM.WORK_DAY
*   **REMOVE_FLAG**: 00:Normal
*   **REGIST_PNAME**: Class name
*   **LAST_UPDATE_DATE**: SYSTIMESTAMP
*   **LAST_UPDATE_PNAME**: Class name
  
#Internal Location Transfer Result
- [Internal Location Transfer Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)

#QC Work from Retrieval for QC Start
- [QC Work from Retrieval for QC Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/961/QC-Work-from-Retrieval-for-QC-Start)

<hr>

#User Story
- [DFD - QC Setting](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_boards/board/t/ID_SimeDarbyPlantation%20Team/Stories?workitem=5788)

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Retrieval for QC Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/900/Retrieval-for-QC-Start)
