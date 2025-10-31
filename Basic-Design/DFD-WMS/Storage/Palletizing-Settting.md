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

#<span style="color:skyblue; font-weight:bold">Palletize Start</span>
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