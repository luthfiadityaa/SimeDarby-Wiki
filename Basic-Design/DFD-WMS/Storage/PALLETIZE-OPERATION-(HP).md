# **Palletize LAYOUT**

![image.png](/.attachments/image-32e1e76d-8510-472e-9299-aaa8b33d063f.png)

**Processes:**

1. No Operations (Empty Pallet Supply)
2. Batch Start
3. Batch End
4. Update To Host

Those processes are related to DNStoragePlan status_flag.

|STATUS_FLAG | Meaning | Explain |
| --- | --- | --- |
|0 | No Started | Plan just register and do not have any Pallet |
|1 | Working | When the 1st Pallet store in update to 1 |
|2 | Waiting for last pallet | Batch end submitted (last_pallet_qty > 0) and waiting for ID25.force_completion flag |
|4 | Completed | Process Completed |


#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
**Stage 1**
::: mermaid
flowchart LR

P1[Palletizing Start]--> |1101-1105| P2[ID26]-->P3[StorageSender]-->P4[ID25]
P4[ID25]--> |1111-1115| P5[ID26]-->P6[StorageSender]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 
:::

**Stage 2**
::: mermaid
flowchart LR

Cond2{Flow?}

Cond2 --> |Flow 1: Without going 
through crane 7~10 - STV| P8[ID64]
Cond2 --> |Flow 2: Go through 
crane 7~10 - STV| P9[ID64]

P8[ID64]--> |7101-7110| P10[ID26]-->P11[StorageSender]-->P12[ID25]--> |SRM| P18[ID64]
P9[ID64]--> |7107-7110| P15[ID26]-->P16[StorageSender]-->P17[ID25]--> |SRM|P18[ID64]
:::

**Stage 3**
::: mermaid
flowchart LR

P18[ID64]--> |STV| P19[ID64]--> |7211-7212| P20[ID26]-->P21[StorageSender]-->P22[ID25]--> |SRM| P23[ID64]

P18[ID64]--> |9001-9010| P14[ID33]
P23[ID64]--> |9011-9014| P14[ID33]
:::

