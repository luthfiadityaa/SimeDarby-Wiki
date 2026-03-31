[[_TOC_]]
[[_TOSP_]]

# Transfer Setting Tempering Area to Ambient Area
![image.png](/.attachments/image-2fb067b7-aa3e-40d4-b523-ccf656bd68d6.png)

# <span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR

P1[TransferSettingSCH<br/>Set F2]-->P2[processTransferWarehouse<br/>in RetrievalSender]-->P3[ID12]-->P4[ID32]-->P5[ID33]-->P6[ID64]-->P7[ID26<br/>at 7207-7214]-->P8[StorageSender<br/>swap rear/front]-->P9[ID05]-->P10[ID25]-->P11[ID64]-->P12[ID33<br/>at 9007-9014]
:::

## Scope
- **Item type**: ZFNP only (Soft Zone 005, 002)
- **Source**: Tempering shelves in aisles 9001-9006 (WH 9100, single deep)
- **Destination**: Ambient shelves in aisles 9007-9014 (WH 9200, double deep)
- **Route**: Shelf (9001-9006) -> ID12 -> BCR (7207-7214) -> ID05 -> Aisle (9007-9014)

## Station Map
```
Source Aisles (WH 9100 Tempering, single deep):
  9001 -> BCR 7101 (OP inbound only)
  9002 -> BCR 7102
  9003 -> BCR 7103
  9004 -> BCR 7104
  9005 -> BCR 7105
  9006 -> BCR 7106

Destination Aisles (WH 9200 Ambient, double deep):
  9007 -> BCR 7107 (OP) / 7207 (HP)   <- transfer uses 7207
  9008 -> BCR 7108 (OP) / 7208 (HP)   <- transfer uses 7208
  9009 -> BCR 7109 (OP) / 7209 (HP)   <- transfer uses 7209
  9010 -> BCR 7110 (OP) / 7210 (HP)   <- transfer uses 7210
  9011 -> BCR 7211 (HP)               <- transfer uses 7211
  9012 -> BCR 7212 (HP)               <- transfer uses 7212
  9013 -> BCR 7213 (HP)               <- transfer uses 7213
  9014 -> BCR 7214 (HP)               <- transfer uses 7214

Transfer Route:
  Shelf in 9001-9006 --ID12--> 7207-7214 (HP BCR) --ID05--> 9007-9014
  via STV 8102
```

---

# <span style="color:red; font-weight:bold">Shelf Booking Strategy (Double Deep)</span>

## Problem
40 pallets same item/batch booked in a fast loop. Aisles 9007-9014 are **double deep**.
If we only book rear shelves, 10 pairs used for 10 pallets (front wasted).
We should book **rear + front of same pair** before moving to next aisle.

## Booking Order (processTransferWarehouse)
```
Pallet  1 -> aisle 9007, pair A REAR   -> dest=7207
Pallet  2 -> aisle 9007, pair A FRONT  -> dest=7207
Pallet  3 -> aisle 9008, pair B REAR   -> dest=7208
Pallet  4 -> aisle 9008, pair B FRONT  -> dest=7208
Pallet  5 -> aisle 9009, pair C REAR   -> dest=7209
Pallet  6 -> aisle 9009, pair C FRONT  -> dest=7209
Pallet  7 -> aisle 9010, pair D REAR   -> dest=7210
Pallet  8 -> aisle 9010, pair D FRONT  -> dest=7210
Pallet  9 -> aisle 9011, pair E REAR   -> dest=7211
Pallet 10 -> aisle 9011, pair E FRONT  -> dest=7211
Pallet 11 -> aisle 9012, pair F REAR   -> dest=7212
Pallet 12 -> aisle 9012, pair F FRONT  -> dest=7212
Pallet 13 -> aisle 9013, pair G REAR   -> dest=7213
Pallet 14 -> aisle 9013, pair G FRONT  -> dest=7213
Pallet 15 -> aisle 9014, pair H REAR   -> dest=7214
Pallet 16 -> aisle 9014, pair H FRONT  -> dest=7214
Pallet 17 -> aisle 9007, pair I REAR   -> dest=7207  (second round)
... round-robin across 8 aisles, 2 per pair
```

## Arrival Swap Logic (StorageSender at 7207-7214)
Pallets may arrive out of order. If front-booked pallet arrives before rear-booked:

```
Case 1: Booked REAR -> rear shelf empty -> store at rear. OK.

Case 2: Booked FRONT -> check rear shelf status:
  2a: rear = OCCUPIED   -> store at front. OK.
  2b: rear = RESERVED   -> rear pallet not stored yet.
      SWAP: this pallet takes rear, update the other carry to front.
      Store at rear. OK.
  2c: rear = EMPTY       -> rear reservation cancelled/failed.
      Take rear instead. Store at rear. OK.
```

**Rule: rear must always be filled before front.** The swap ensures physical correctness regardless of arrival order.

---

# <span style="color:red; font-weight:bold">Concurrent Operation Analysis</span>

Aisles 9007-9014 handle THREE types of operations simultaneously:

| Operation | carry_flag | Source | Shelf Search |
|-----------|-----------|--------|-------------|
| Normal inbound storage | 1 (STORAGE) | 1301/1302 via 7207-7214 | StorageSender -> LocationManager (new search) |
| Transfer from Tempering | 5 (RACK_TO_RACK) | 9001-9006 via 7207-7214 | processTransferWarehouse (pre-booked) |
| DoubleDeep rack-to-rack | 5 (RACK_TO_RACK) | within 9200 | DoubleDeepChecker (dest=shelf, no BCR) |
| Retrieval outbound | 2 (RETRIEVAL) | 9007-9010 to 1301/1302, 1205-1209 | Releases shelves |
| Retrieval outbound | 2 (RETRIEVAL) | 9007-9014 to 1301/1302, 1205-1209, 1201-1204 | Releases shelves |

## Conflict Analysis

### Transfer booking vs Normal inbound storage
- **Transfer**: books shelves with `DMSHELF.status = RESERVED` via `FOR UPDATE` lock
- **Normal storage**: StorageSender -> LocationManager -> `DoubleDeepShelfSelector` also uses `FOR UPDATE`
- **Result**: `FOR UPDATE WAIT` serializes access. Normal storage skips RESERVED shelves (queries `status = EMPTY`). **No conflict.**

### Transfer booking vs DoubleDeep rack-to-rack (within 9200)
- **DoubleDeepChecker** creates carry_flag=5 with `dest_station_no = target shelf` (set)
- **Transfer** creates carry_flag=5 with `dest_station_no = BCR 7207-7210` (set) or blank initially
- **Distinction**: DoubleDeep dest is a shelf (12-digit), Transfer dest is a station (4-digit)
- **Shelf locking**: both use `FOR UPDATE`. **No conflict.**

### Transfer booking vs Retrieval from 9007-9010
- Retrieval **releases** shelves (OCCUPIED -> EMPTY at ID33)
- Transfer booking **reserves** shelves (EMPTY -> RESERVED)
- These are opposite operations, no collision
- **Benefit**: recently freed shelves become available for transfer booking

### Risk: Transfer fills all pairs, normal storage has nowhere to go
- If 40 transfers book all empty pairs in 9007, normal inbound to 7207 gets `wait_reason = FULL`
- **Mitigation**: `processTransferWarehouse` should check remaining capacity and leave a buffer
- **Suggestion**: leave at least `MIN_EMPTY_PAIR_DIFFERENT_ZONE (2)` unreserved pairs per aisle

---

#<span style="color:skyblue; font-weight:bold">Transfer Setting from Tempering to Ambient — Detailed Flow</span>

## Abbreviation
- **PLLT** : DNPALLET
- **WRKI** : DNWORKINFO
- **WRKL** : DNWORKLIST
- **CRYI** : DNCARRYINFO
- **STCK** : DNSTOCK
- **ARVL** : DNARRIVAL
- **WRHS** : DMWAREHOUSE
- **SHLF** : DMSHELF
- **STCH** : DNSTOCKHISTORY
- **INOT** : DNINOUTRESULT
- **HTST** : DNHOSTSEND
- **OPRR** : DNOPERATIONRESULT
- **ITEM** : DMITEM
- **STSN** : DMSTATION
- **AISL** : DMAISLE

- **Operation Name**
  - **I** : INSERT
  - **U** : UPDATE
  - **D** : DELETE
  - **S** : SELECT

| Action Name                                                       | PLLT | WRKI | WRKL | CRYI | STCK | ARVL | WRHS | SHLF | STCH | INOT | HTST | OPRR | ITEM | STSN | AISL |
|-------------------------------------------------------------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|
| Transfer Setting - Set(F2) [(1)](#1-transfer-setting---setf2)     |   S  |   I  |   I  |   I  |      |      |   S  |      |      |      |      |      |   S  |   S  |      |
| processTransferWarehouse [(2)](#2-processtransferwarehouse)       |   U  |      |      |   U  |      |      |      |  U   |      |      |      |      |      |      |   S  |
| **Retrieval Flow**                                                |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| RetrievalSender - ID12 [(3)](#3-retrievalsender---id12)           |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID32 [(4)](#4-id32)                                               |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID33 Retrieval [(5)](#5-id33-retrieval-complete)                  |      |      |      |   U  |      |      |      |  U   |      |      |      |      |      |      |      |
| ID64 STV [(6)](#6-id64-stv-pickup)                                |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| **Re-Storage Flow**                                               |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| ID26 at 7207-7214 [(7)](#7-id26-at-7207-7210)                    |   U  |   U  |      |   U  |      |   I  |      |      |      |      |      |      |      |      |      |
| StorageSender + swap [(8)](#8-storagesender-at-7207-7210)         |      |   U  |      |   U  |      |   U  |   U  |   U  |      |      |      |      |      |      |      |
| ID25 [(9)](#9-id25-at-7207-7210)                                  |      |      |      |   U  |      |   D  |      |      |      |      |      |      |      |      |      |
| ID64 SRM [(10)](#10-id64-at-srm)                                  |      |      |      |   U  |      |      |      |      |      |      |      |      |      |      |      |
| ID33 Storage [(11)](#11-id33-at-9007-9014)                        |   U  |   U  |      |   D  |   U  |      |      |  U   |   I  |      |   I  |      |      |      |      |
| **Host Reporting**                                                |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| DNHostSend [(12)](#12-dnhostsend---sap-reporting)                 |      |      |      |      |      |      |      |      |      |      |   U  |      |      |      |      |
| **Error Handling**                                                |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |
| Aisle Error [(13)](#13-aisle-disconnecterror-handling)             |      |      |      |      |      |      |      |      |      |      |      |      |      |      |      |

---

# (1) Transfer Setting - Set(F2)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.transfersetting.TransferSettingSCH` &nbsp;</span>

User selects pallets to transfer from Tempering (9100) to Ambient (9200).
**Aisle/shelf is NOT decided here.** Only creates work + carry records.

## <span style="color:skyblue; font-weight:bold">Validations</span>
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected source station is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected source station is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Stock Status must be **UU**
- Tempering Flag must be **REACHED**
- QC Check Flag must be **DONE**
- Item type must be **ZFNP** (Soft Zone 005 or 002 only)
- Target warehouse (9200) has at least **1 empty pair** in aisles 9007-9010

## <span style="color:skyblue; font-weight:bold">DNWORKINFO</span>
- **JOB_NO** : Sequence Object
- **SETTING_UNIT_KEY** : Sequence Object
- **COLLECT_JOB_NO** : Sequence Object
- **JOB_TYPE** : 45: ASRS Rack-to-Rack
- **STATUS_FLAG** : 1: Working
- **HARDWARE_TYPE** : 3: ASRS
- **PLAN_UKEY** : Sequence Object
- **STOCK_ID** : DNSTOCK.STOCK_ID
- **PLAN_AREA_NO** : DNSTOCK.AREA_NO (source area 9100)
- **PLAN_LOCATION_NO** : DNSTOCK.LOCATION_NO (source shelf)
- **PLAN_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **BATCH_NO** : Value from screen
- **MATERIAL_CODE** : Value from screen
- **MATERIAL_NAME** : Value from screen
- **PLAN_QTY** : Value from screen (Stock Qty)
- **UOM** : Value from screen
- **TEMPERING_PERIOD** : Value from screen
- **EXPIRY_DAYS** : DNSTOCK.EXPIRY_DATE
- **WORK_DAY** : DMWARENAVISYSTEM.WORK_DAY
- **BCR_DATA** : DNPALLET.BCR_DATA
- **USER_ID** : Login Info
- **TERMINAL_NO** : Login info

## <span style="color:skyblue; font-weight:bold">DNWORKLIST</span>
- **JOB_NO**: DNWORKINFO.JOB_NO
- **SETTING_UNIT_KEY**: Sequence Object
- **COLLECT_JOB_NO**: DNWORKINFO.COLLECT_JOB_NO
- **JOB_TYPE**: 45: ASRS Rack-to-Rack
- **PLAN_UKEY**: DNWORKINFO.PLAN_UKEY
- **STOCK_ID**: DNWORKINFO.STOCK_ID
- **PALLET_ID**: DNPALLET.PALLET_ID
- **PLAN_DAY**: DNWORKINFO.PLAN_DAY
- **BATCH_NO**: DNWORKINFO.BATCH_NO
- **PLAN_AREA_NO**: DNWORKINFO.PLAN_AREA_NO
- **MATERIAL_CODE**: DNWORKINFO.MATERIAL_CODE
- **MATERIAL_NAME**: DNWORKINFO.MATERIAL_NAME
- **RETRIEVAL_DETAIL**: 1: Unit Retrieval
- **PLAN_QTY**: DNWORKINFO.PLAN_QTY

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CARRY_KEY** : Sequence Object
- **PALLET_ID** : DNSTOCK.PALLET_ID
- **WORK_TYPE** : 60: Rack Move From
- **CMD_STATUS** : 1: Started
- **PRIORITY** : 2: Normal
- **RESTORING_FLAG** : 0: Not Restore to Original Location
- **CARRY_FLAG** : 5: Rack-to-Rack
- **WORK_NO** : Sequence Object
- **RETRIEVAL_STATION_NO** : DNSTOCK.LOCATION_NO (source shelf)
- **RETRIEVAL_DETAIL** : 1: Unit Retrieval
- **SOURCE_STATION_NO** : DNPALLET.CURRENT_STATION_NO (source shelf)
- **DEST_STATION_NO** : `blank` (decided by processTransferWarehouse)
- **END_STATION_NO** : 9200 (target warehouse)
- **AISLE_STATION_NO** : DMSHELF.PARENT_STATION_NO (source aisle 9001-9006)
- **RESERVED_SHELF_NO** : `blank` (decided by processTransferWarehouse)
- **CANCEL_REQUEST** : 0: Not requested
- **SCHEDULE_NO** : Sequence Object

## <span style="color:skyblue; font-weight:bold">DNPALLET</span>
- **STATUS_FLAG** : 3: Reserved for Retrieval
- **ALLOCATION_FLAG** : 1: Allocated

---

# (2) processTransferWarehouse

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.RetrievalSender.processTransferWarehouse()` &nbsp;</span>

Called from `getRackMoveInfoForUpdate()` when carry has `carry_flag=5` AND `dest_station_no IS BLANK`.

::: mermaid
flowchart TD
    A[Query DNCarryInfo<br/>carry_flag=5, cmd_status=1<br/>dest_station_no IS BLANK] --> B[Load Pallet + target WH 9200]
    B --> C[Decide aisle in 9200<br/>round-robin 9007-9014]
    C --> D{Empty pair<br/>available?}
    D -->|YES| E[Book rear shelf<br/>DMSHELF status->RESERVED<br/>FOR UPDATE lock]
    D -->|NO| F[wait_reason=FULL<br/>skip, retry next cycle]
    E --> G[Check: is front of<br/>same pair also empty?]
    G -->|YES next pallet| H[Book front shelf<br/>DMSHELF status->RESERVED]
    G -->|NO or last pallet| I[Get BCR from DMAisle<br/>aisle 9007->7207]
    H --> I
    I --> J[Update DNCarryInfo:<br/>dest=BCR, reserved_shelf_no=shelf<br/>aisle_station_no=target aisle]
    J --> K[Continue to ID12 send]
:::

## Aisle Decision Criteria
```
1. Query DMAisle WHERE wh_station_no = '9200' AND status = NORMAL
   -> aisles 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014
2. Round-robin using DMWareHouse.last_used_station_no
3. For each aisle: searchPairEmptyShelf (both rear+front EMPTY)
   -> filter: soft_zone matches pallet (005 or 002)
   -> filter: hard_zone matches
   -> FOR UPDATE WAIT (locks shelf rows)
4. Book rear first, then front of same pair, then next aisle
5. Leave buffer: stop booking if aisle has < 2 empty pairs remaining
   (for normal inbound storage to still work)
```

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
- **DEST_STATION_NO** : BCR station (7207..7214) from DMAisle.bcr_station_no
- **END_STATION_NO** : 9200 (stays unchanged)
- **AISLE_STATION_NO** : target aisle (9007..9014) — **changed from source to target**
- **RESERVED_SHELF_NO** : booked shelf location (rear or front)
- **LAST_UPDATE_PNAME** : RetrievalSender

## <span style="color:skyblue; font-weight:bold">DMSHELF (UPDATE)</span>
- **STATUS_FLAG** : 2: Reserved
- **LAST_UPDATE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
- **STATUS_FLAG** : 3: Reserved for Retrieval

---

# (3) RetrievalSender - ID12

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.RetrievalSender` &nbsp;</span>

After `processTransferWarehouse` sets dest, the carry continues through normal `sendCarry()` flow. ID12 sent with transport class = "5" (C_MOVE).

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
- **CMD_STATUS** : 2: Waiting for response

---

# (4) ID32

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id32Process` &nbsp;</span>

AGC acknowledged the retrieval command.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 3: Commanded
- **ERROR_CODE** : 0

---

# (5) ID33 Retrieval Complete

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id33Process` &nbsp;</span>

Pallet is out of the rack. Source shelf released.

## <span style="color:skyblue; font-weight:bold">DMSHELF (source shelf in 9100)</span>
- **STATUS_FLAG** : 0: Empty

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 5: Retrieval completed
- **CARRY_FLAG** : 5: Rack-to-Rack (stays unchanged)

---

# (6) ID64 STV Pickup

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id64Process` &nbsp;</span>

STV picked up the pallet from SRM and transporting to BCR.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4: Pickup completed

---

# (7) ID26 at 7207-7214

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.location.AsrsInboundStationOperator` &nbsp;</span>

Pallet arrives at HP BCR. Station operator detects carry_flag=5 (RACK_TO_RACK) with `end_station_no = 9200` matching `wh_station_no = 9200` -> **FINAL BCR**.

Converts carry from rack-to-rack retrieval to storage for re-storage into 9200.

## <span style="color:skyblue; font-weight:bold">DNARRIVAL (INSERT)</span>
- **ARRIVAL_DATE** : SYSTIMESTAMP
- **STATION_NO** : 7207..7214
- **CARRY_KEY** : DNCARRYINFO.CARRY_KEY
- **BCR_DATA** : from ID26
- **SEND_FLAG** : 0: Not sent

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
- **WORK_TYPE** : 2: Storage
- **CMD_STATUS** : 1: Started
- **CARRY_FLAG** : 1: Storage
- **SOURCE_STATION_NO** : 7207..7214
- **DEST_STATION_NO** : 9200 (WH station)
- **END_STATION_NO** : 9200

## <span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
- **CURRENT_STATION_NO** : 7207..7214
- **WH_STATION_NO** : 9200

## <span style="color:skyblue; font-weight:bold">DNWORKINFO (UPDATE)</span>
- **JOB_TYPE** : 02: Storage (changed from 45)
- **STATUS_FLAG** : 1: Working

---

# (8) StorageSender at 7207-7214

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.transmission.StorageSender` &nbsp;</span>

StorageSender picks up carry (now carry_flag=STORAGE, cmd_status=START). **Before calling LocationManager**, checks `reserved_shelf_no` and applies the **rear/front swap logic**.

::: mermaid
flowchart TD
    A[Carry has reserved_shelf_no<br/>from processTransferWarehouse] --> B{reserved shelf<br/>side?}
    B -->|REAR| C[Send ID05<br/>to rear shelf]
    B -->|FRONT| D{Check rear shelf<br/>of same pair}
    D -->|rear=OCCUPIED| E[Send ID05<br/>to front shelf]
    D -->|rear=RESERVED<br/>another carry| F[SWAP:<br/>this carry->rear<br/>other carry->front]
    D -->|rear=EMPTY| G[Take rear instead<br/>update reserved_shelf]
    F --> C
    G --> C
:::

## <span style="color:skyblue; font-weight:bold">DMSHELF (booked shelf)</span>
- **STATUS_FLAG** : 2: Reserved (already set, confirmed)

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO (UPDATE)</span>
- **AISLE_STATION_NO** : target aisle (9007..9014)
- **RESERVED_SHELF_NO** : final shelf (after swap if needed)
- **DEST_STATION_NO** : final shelf location
- **CMD_STATUS** : 2: Waiting for response

## <span style="color:skyblue; font-weight:bold">DMWAREHOUSE (UPDATE)</span>
- **LAST_USED_STATION_NO** : aisle used

## <span style="color:skyblue; font-weight:bold">DNARRIVAL (UPDATE)</span>
- **SEND_FLAG** : 1: Sent

---

# (9) ID25 at 7207-7214

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id25Process` &nbsp;</span>

AGC acknowledged the storage command.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 3: Commanded
- **ERROR_CODE** : 0

## <span style="color:skyblue; font-weight:bold">DNARRIVAL (DELETE)</span>
- Arrival record deleted after AGC response

---

# (10) ID64 at SRM

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id64Process` &nbsp;</span>

SRM picked up pallet for storage into shelf.

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO</span>
- **CMD_STATUS** : 4: Pickup completed

---

# (11) ID33 at 9007-9014

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.asrs.communication.control.Id33Process` &nbsp;</span>

Storage complete. Pallet is now on the shelf in 9200.

## <span style="color:skyblue; font-weight:bold">DNSTOCK (UPDATE)</span>
- **AREA_NO** : 9200 (new area)
- **LOCATION_NO** : actual shelf from ID33
- **NEWEST_STORAGE_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DMSHELF (target shelf)</span>
- **STATUS_FLAG** : 1: Occupied

## <span style="color:skyblue; font-weight:bold">DNPALLET (UPDATE)</span>
- **CURRENT_STATION_NO** : actual shelf from ID33
- **STATUS_FLAG** : 2: Occupied
- **ALLOCATION_FLAG** : 0: Not allocated
- **WH_STATION_NO** : 9200
- **LAST_STORED_DATE** : SYSTIMESTAMP

## <span style="color:skyblue; font-weight:bold">DNWORKINFO (UPDATE)</span>
- **RESULT_QTY** : PLAN_QTY
- **RESULT_AREA_NO** : 9200
- **RESULT_LOCATION_NO** : actual shelf
- **STATUS_FLAG** : 4: Completed

## <span style="color:skyblue; font-weight:bold">DNSTOCKHISTORY (INSERT)</span>
- **INC_DEC_TYPE** : 1: Stock Increase
- **JOB_TYPE** : 2: Storage
- **AREA_NO** : 9200
- **LOCATION_NO** : actual shelf

## <span style="color:skyblue; font-weight:bold">DNCARRYINFO (DELETE)</span>
- Carry record deleted after completion

---

# (12) DNHostSend - SAP Reporting

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.host.InternalLocTransferReportDataCreator` &nbsp;</span>

At ID33 storage completion, a DNHostSend record is created. The HostCommExecutor picks it up and sends an XML PalletUpdate to SAP via SFTP.

See: [Internal Location Transfer Result](../../Host-Interfaces/Send-Method/Internal-Location-Transfer-Result)

## <span style="color:skyblue; font-weight:bold">DNHOSTSEND (INSERT at ID33)</span>
- **JOB_TYPE** : 45: ASRS Rack-to-Rack
- **REPORT_FLAG** : 0: Not reported
- **PLAN_UKEY** : DNWORKINFO.PLAN_UKEY
- **ITEM_CODE** : DNSTOCK.ITEM_CODE
- **BATCH_NO** : DNSTOCK.BATCH_NO
- **RESULT_QTY** : DNWORKINFO.RESULT_QTY
- **UOM** : DNWORKINFO.UOM
- **RESULT_AREA_NO** : 9200 (destination area)
- **STOCK_STATUS** : DNSTOCK.STOCK_STATUS (UU)
- **TEMPERING_FLAG** : DNSTOCK.TEMPERING_FLAG (REACHED)

## <span style="color:skyblue; font-weight:bold">XML to SAP (PalletUpdate)</span>
```xml
<PalletUpdate>
    <MsgID>{sequence}</MsgID>
    <Action>0</Action>
    <MaterialCode>{ITEM_CODE}</MaterialCode>
    <Plant>{RESULT_AREA_NO mapped}</Plant>
    <Batch>{BATCH_NO}</Batch>
    <Quantity>{RESULT_QTY}</Quantity>
    <UoM>{UOM}</UoM>
    <PostingDate>{LAST_UPDATE_DATE}</PostingDate>
    <StatusTo>UU</StatusTo>
    <StorageLocationFrom>FGW2</StorageLocationFrom>
    <StorageLocationTo>FGW1</StorageLocationTo>
    <StartDateTime>{retrieval start}</StartDateTime>
    <EndDateTime>{storage complete}</EndDateTime>
    <Remark>process complete</Remark>
</PalletUpdate>
```

After SAP response:
- **DNHOSTSEND.REPORT_FLAG** : 1: Reported
- **DNHOSTSEND.MSG_ID** : assigned by SAP

---

# (13) Aisle Disconnect/Error Handling

## At processTransferWarehouse (booking time)
```
If aisle status != NORMAL (disconnect/error):
  -> skip this aisle, try next aisle in round-robin
  -> if ALL aisles in 9200 are down: wait_reason=NO_ROUTE, retry next cycle
```

## At StorageSender (7207-7214, after pallet arrives)
```
If target aisle is disconnect/error:
  -> StorageSender cannot send ID05
  -> wait_reason = NO_ROUTE or FULL
  -> Pallet sits at BCR station (7207-7214)
  -> StorageSender retries each cycle

If aisle does NOT recover:
  -> Operator sends ID35 (manual interference) to clear the pallet
  -> Id35Process cancels the carry
  -> Releases reserved shelf (DMSHELF.status -> EMPTY)
  -> Deletes DNCarryInfo
  -> Pallet physically removed by operator from BCR
```

## At ID33 Retrieval (pallet out of source shelf, in transit)
```
If target aisle goes down AFTER retrieval but BEFORE arrival at BCR:
  -> Pallet is in transit on STV, cannot be stopped
  -> Pallet arrives at BCR (7207-7214), ID26 received
  -> AsrsInboundStationOperator converts carry to STORAGE
  -> StorageSender finds aisle down -> wait_reason=NO_ROUTE
  -> Same as above: wait for recovery or ID35
```

## Reserved shelf cleanup on ID35
```
Id35Process:
  1. Read carry.reserved_shelf_no
  2. If shelf status = RESERVED: update to EMPTY
  3. Delete DNCarryInfo
  4. Update DNPallet status
  5. Update DNWorkInfo status -> cancelled
  NOTE: DNHostSend is NOT created for cancelled transfers
```

---

# Carry Lifecycle Summary

```
(1) TransferSettingSCH:
    carry_flag=5(R2R), cmd_status=1, dest=blank, end=9200
    reserved_shelf_no=blank, aisle_station_no=source(9001-9006)

(2) processTransferWarehouse:
    dest=7207-7210(BCR), reserved_shelf_no=booked shelf
    aisle_station_no=target(9007-9010), DMSHELF->RESERVED

(3) RetrievalSender ID12:
    cmd_status=2(WAIT_RESPONSE)

(4) ID32:
    cmd_status=3(COMMANDED)

(5) ID33 Retrieval:
    cmd_status=5(COMP_RETRIEVAL), source DMSHELF->EMPTY

(6) ID64:
    cmd_status=4(PICKUP)

(7) ID26 at 7207-7214:
    carry_flag=1(STORAGE), cmd_status=1(START)
    source=7207-7210, dest=9200, end=9200

(8) StorageSender (swap logic):
    dest=final shelf, cmd_status=2(WAIT_RESPONSE)

(9) ID25:
    cmd_status=3(COMMANDED)

(10) ID64:
    cmd_status=4(PICKUP)

(11) ID33 Storage:
    DNCARRYINFO deleted, DMSHELF->OCCUPIED, DNPALLET->shelf
```

---

# Database Prerequisites

## DMRouteId — Routes 9001-9006 -> 7207-7214
48 routes total, route_type='2' (retrieval). Already added to `DMRouteId.sql`:
- route_id 449-472: 9001-9006 -> 7207-7210 (aisles 9007-9010)
- route_id 473-496: 9001-9006 -> 7211-7214 (aisles 9011-9014)

## DMRouteDetail — Physical path via STV 8102
144 entries (3 per route): SRM -> STV 8102 -> BCR.
Already added to `DMRouteDetail.sql`.

## DMStation — Required status
```sql
-- Source aisles: must be NORMAL and online
SELECT station_no, status, suspend FROM DMStation
WHERE station_no IN ('9001','9002','9003','9004','9005','9006');
-- status='1'(NORMAL), suspend='0'(OFF)

-- Target BCR: must be NORMAL, sendable, max_instruction >= 1
SELECT station_no, status, sendable, max_instruction FROM DMStation
WHERE station_no IN ('7207','7208','7209','7210');
-- status='1', sendable='1', max_instruction=2

-- Target aisles: must be NORMAL
SELECT station_no, status FROM DMAisle
WHERE station_no IN ('9007','9008','9009','9010');
-- status='1'(NORMAL)
```

## DMAisle — BCR mapping
```
9007 -> bcr_station_no = 7107  (OP)  /  7207 (HP, used by transfer)
9008 -> bcr_station_no = 7108  (OP)  /  7208 (HP)
9009 -> bcr_station_no = 7109  (OP)  /  7209 (HP)
9010 -> bcr_station_no = 7110  (OP)  /  7210 (HP)
9011 -> bcr_station_no = 7211  (HP, used by transfer)
9012 -> bcr_station_no = 7212  (HP)
9013 -> bcr_station_no = 7213  (HP)
9014 -> bcr_station_no = 7214  (HP)
```
Note: DMAisle.bcr_station_no holds only ONE BCR per aisle.
Transfer must use `getReachableBcrForAisle()` or hardcode 720x for HP route.

## DMWareHouse — Aisle decision pattern
```sql
SELECT station_no, aisle_decision_pattern, zone_manage_type FROM DMWareHouse
WHERE station_no = '9200';
-- aisle_decision_pattern = '4' (WNCollectAisleSelector)
-- zone_manage_type = soft_priority_combine
```

## DMSoftZonePriority — Zone fallback
```
Zone 005 (ZFNP) -> fallback to Zone 002 (Ambient) in WH 9200
```

## DMGroupController — AGC online
```sql
SELECT controller_no, status_flag, connection_flag FROM DMGroupController;
-- status_flag='1'(ONLINE), connection_flag='1'(CONNECTED)
```

---

# Implementation Checklist

| # | Item | Class/File | Status |
|---|------|-----------|--------|
| 1 | TransferSettingSCH.addnew() | TransferSettingSCH.java | TODO |
| 2 | processTransferWarehouse() in RetrievalSender | RetrievalSender.java | TODO |
| 3 | Shelf booking (rear+front pair, round-robin) | New method in RetrievalSender | TODO |
| 4 | getRackMoveInfoForUpdate() — add dest=blank check | RetrievalSender.java | TODO |
| 5 | AsrsInboundStationOperator — handle RACK_TO_RACK arrival | AsrsInboundStationOperator.java | TODO |
| 6 | StorageSender — rear/front swap logic | StorageSender.java | TODO |
| 7 | DMRouteId 449-496 | DMRouteId.sql | DONE |
| 8 | DMRouteDetail 449-496 | DMRouteDetail.sql | DONE |
| 9 | ID35 cancel — release reserved shelf in 9200 | Verify existing | TODO |
| 10 | DNHostSend insert at ID33 (job_type=45) | CarryCompleteOperator / HostSendController | TODO |
| 11 | InternalLocTransferReportDataCreator | Host reporting (existing, verify) | TODO |
| 12 | Aisle error handling at StorageSender | StorageSender.java | TODO |

# User Story
  - #6614

# Related DFD
  - [Ambient-to-Tempering](Ambient-to-Tempering)
