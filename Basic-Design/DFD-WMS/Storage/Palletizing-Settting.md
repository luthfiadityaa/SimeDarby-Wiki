[[_TOC_]]
[[_TOSP_]]

**The Process Situation Example:**
*   **Planned Carton Qty:** `200` (Input at screen)    
*   **Qty crtn/PL:** `12` (Input at screen)
*   System auto-calculates `200 ÷ 12 = 16 pallets (plus 1 partial)`.    
*   Then it auto-creates **detail records** in background:
    
    | Pallet No | Qty crtn/PL | Total Actual Carton Qty |
    | --- | --- | --- |
    | 1 | 12 | 12 |
    | 2 | 12 | 24 |
    | ... | ... | ... |
    | 17 | 8 | 200 |