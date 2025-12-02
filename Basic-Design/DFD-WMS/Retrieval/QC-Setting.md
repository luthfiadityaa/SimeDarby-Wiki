[[_TOC_]]
[[_TOSP_]]

**Tempering Period Checker**

- **Tempering Period** will use the **Storage Date & Time** for calculate
  - `Storage Date & Time`: 2/12/2025 02:00:00
  - `Tempering Period`: 72 Hours
  - `Tempering Period Date End`: (Storage Date & Time) + (Tempering Period) → **5/12/2025 02:00:00** 
   
::: mermaid
sequenceDiagram
    autonumber

    participant DNSTOCK
    participant Retrieval for QC Start
    participant QC Work from Retrieval for QC Start
    participant Retrieval for Return Stock
    participant QC Work from Retrieval for Return Stock
    participant Update QC Status
    participant Extend Tempering Period

    %% --- 1. DNSTOCK ---
    DNSTOCK ->> Retrieval for QC Start: Now >= Tempering Period Date End
    DNSTOCK ->> Retrieval for QC Start: Stock Status: UU  
    DNSTOCK ->> Retrieval for QC Start: Tempering Flag: Not Reached 
    DNSTOCK ->> Retrieval for QC Start: QC Check Flag: Not Done
    DNSTOCK ->> Retrieval for QC Start: Stock Qty: 60

    %% --- 2. Retrieval for QC Start ---
    Retrieval for QC Start ->> QC Work from Retrieval for QC Start: Qty To Pick?  2

    %% --- 3. QC Work from Retrieval for QC Start ---
    QC Work from Retrieval for QC Start ->> DNSTOCK: Stock Qty: (60-2) → 58
    QC Work from Retrieval for QC Start ->> DNSTOCK: QC Duration: (Last Update Date + Now)
    QC Work from Retrieval for QC Start ->> DNSTOCK: Stock Status: QI
    QC Work from Retrieval for QC Start ->> DNSTOCK: Tempering Flag: Reached

    %% --- 4. Retrieval for Return Stock ---
    DNSTOCK ->> Retrieval for Return Stock: Now >= Tempering Period Date End
    DNSTOCK ->> Retrieval for Return Stock: Stock Status: QI  
    DNSTOCK ->> Retrieval for Return Stock: Tempering Flag: Reached 
    DNSTOCK ->> Retrieval for Return Stock: QC Check Flag: Not Done
    DNSTOCK ->> Retrieval for Return Stock: Stock Qty: 58

    Retrieval for Return Stock ->> QC Work from Retrieval for Return Stock: Qty To Add?  2

    %% --- 5. QC Work from Retrieval for Return Stock ---
    QC Work from Retrieval for Return Stock ->> DNSTOCK: Stock Qty: (58+2) → 60

    %% --- 6. Update QC Status ---
:::