[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Flow</span>
::: mermaid
flowchart LR
    input[Palletize Start]
    process[Background Process]
    output[Batch End]

    input --> process --> output

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

::: mermaid
flowchart TD  
P1[Palletizing Start]-->P2[ID26]-->P3[StorageSender]-->P4[ID25]
P4[ID25]-->P5[ID26]-->P6[StorageSender]-->P7[ID25]
P7[ID25]-->Cond1{Flow?} 
Cond1 --> |Flow 1| P8[ID64]
Cond1 --> |Flow 2| P9[ID64]
P8[ID64]-->P10[ID26]-->P11[StorageSender]-->P12[ID25]-->P13[ID64]
P9[ID64]-->P15[ID26]-->P16[StorageSender]-->P17[ID25]-->P18[ID64]-->P19[ID64]-->P20[ID26]-->P21[StorageSender]-->P22[ID25]-->P23[ID64]

P24[Batch End]-->P2[ID26]

P13[ID64]-->P14[ID33]
P23[ID64]-->P14[ID33]
:::

#<span style="color:skyblue; font-weight:bold">Palletize Start</span>
This is only one-time input (no per pallet entry).<br>
Example Condition:
- **Planned Carton Qty:** `200` (Input at screen)    
- **Qty crtn/PL:** `12` (Input at screen)

#<span style="color:skyblue; font-weight:bold">Background Process</span>
- System auto-calculates `200 ÷ 12 = 16 pallets (plus 1 partial)`.    
- Then it auto-creates **detail records** in background:
    
    | Pallet No | Qty crtn/PL | Total Actual Carton Qty |
    | --- | --- | --- |
    | 1 | 12 | 12 |
    | 2 | 12 | 24 |
    | ... | ... | ... |
    | 17 | 8 | 200 |

#<span style="color:skyblue; font-weight:bold">Batch End</span>
- **Storage Qty:** `8` (Value set by system or Input at screen) 