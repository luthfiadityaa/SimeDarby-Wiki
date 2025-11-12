# Hyper link

|                                     | Pros                       | Cons                  |
|-------------------------------------|----------------------------|-----------------------|
| Sample 1 [(2)](#sample-1)           | Easy to edit for everyone. | A bit worse to read.  |
| Sample 2 [(3)](#sample-2)           | Easy to read for everyone. | Worse to maintenance. |

---

# List vs Table

## Sample 1
👌Easy to edit for everyone.
👎A bit worse to read.

### Column and value
- CARRY_KEY = Sequence object
- PALLET_ID = DNSTOCK.PALLET_ID
- WORK_TYPE = 40:Inventory Check
- WORK_NO = Sequence object
- RETRIEVAL_STATION_NO = DNSTOCK.LOCATION_NO
- SOURCE_STATION_NO = DNPALLET.CURRENT_STATION_NO
- DEST_STATION_NO = **1251-1254**
- SCHEDULE_NO = Sequence object


## Sample 2
👌Easy to read for everyone.
👎Worse to maintenance.

| Column               | Value                                           |
|----------------------|-------------------------------------------------|
| CARRY_KEY            | Sequence object                                 |
| PALLET_ID            | DNSTOCK.PALLET_ID                               |
| WORK_TYPE            | 40:Inventory Check                              |
| WORK_NO              | Sequence object                                 |
| RETRIEVAL_STATION_NO | DNSTOCK.LOCATION_NO                             |
| SOURCE_STATION_NO    | DNPALLET.CURRENT_STATION_NO                     |
| DEST_STATION_NO      | <span style="font-weight:bold">1251-1254</span> |
| SCHEDULE_NO          | Sequence object                                 |

---

# Word decoration
**Detail Flow**
👌Easy to edit for everyone.
👎A bit worse to read.

<span style="color:skyblue; font-weight:bold">Detail Flow</span>
👌Easy to read for everyone.
👎Worse to maintenance.

---

# Diagram

## All in one
::: mermaid
flowchart LR
  
P1[Palletizing Start]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]
P4[ID25]-->P5[ID26]-->P6[StorageSender]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 
Cond1 --> |Flow 1: Without going 
through crane 7~10| P8[ID64]
Cond1 --> |Flow 2: Go through 
crane 7~10| P9[ID64]
P8[ID64]-->P10[ID26]-->P11[StorageSender]-->P12[ID25]-->P13[ID64]
P9[ID64]-->P15[ID26]-->P16[StorageSender]-->P17[ID25]-->P18[ID64]-->P19[ID64]-->P20[ID26]-->P21[StorageSender]-->P22[ID25]-->P23[ID64]

P24[Batch End]-->P2[ID26]

P13[ID64]-->P14[ID33]
P23[ID64]-->P14[ID33]
:::

## Separate
### Stage 1
::: mermaid
flowchart LR
  
P1[Palletizing Start]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]
P4[ID25]-->P5[ID26]-->P6[StorageSender]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 

P24[Batch End]-->P2[ID26]
:::

### Stage 2
::: mermaid
flowchart LR

Cond1{Flow?}  --> |Flow 1: Without going 
through crane 7~10| P8[ID64]
Cond1 --> |Flow 2: Go through 
crane 7~10| P9[ID64]
P8[ID64]-->P10[ID26]-->P11[StorageSender]-->P12[ID25]-->P13[ID64]
P9[ID64]-->P15[ID26]-->P16[StorageSender]-->P17[ID25]-->P18[ID64]-->P19[ID64]-->P20[ID26]-->P21[StorageSender]-->P22[ID25]-->P23[ID64]

P13[ID64]-->P14[ID33]
P23[ID64]-->P14[ID33]
:::