[[_TOC_]]
[[_TOSP_]]

# **QC Summary Flow**   
<br>

- **Tempering Period** will use the **Storage Date & Time** for calculate
   
  Data as example:
  - `Storage Date & Time`: 2/12/2025 02:00:00
  - `Tempering Period`: 72 Hours
  - `Tempering Period Date End`: (Storage Date & Time) + (Tempering Period) → **5/12/2025 02:00:00** 

  The tempering period will remain at `72 hours` even if the original end date has passed. Any changes will only be applied upon approval to `Extend the Tempering Period`. 

<br>

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
    DNSTOCK ->> Retrieval for QC Start: Now >= Tempering Period Date End <br> Stock Status: UU <br> Tempering Flag: Not Reached <br> QC Check Flag: Not Done <br> Stock Qty: 60


    %% --- 2. Retrieval for QC Start ---
    Retrieval for QC Start ->> QC Work from Retrieval for QC Start: Qty To Pick?  2

    %% --- 3. QC Work from Retrieval for QC Start ---
    QC Work from Retrieval for QC Start ->> DNSTOCK: Stock Qty: (60-2) → 58
    QC Work from Retrieval for QC Start ->> DNSTOCK: QC Duration: (Last Update Date + Now)
    QC Work from Retrieval for QC Start ->> DNSTOCK: Stock Status: QI
    QC Work from Retrieval for QC Start ->> DNSTOCK: Tempering Flag: Reached

    %% --- 4. Retrieval for Return Stock ---
    DNSTOCK ->> Retrieval for Return Stock: Stock Status: QI  
    DNSTOCK ->> Retrieval for Return Stock: Tempering Flag: Reached 
    DNSTOCK ->> Retrieval for Return Stock: QC Check Flag: Not Done
    DNSTOCK ->> Retrieval for Return Stock: Stock Qty: 58

    Retrieval for Return Stock ->> QC Work from Retrieval for Return Stock: Qty To Add?  2

    %% --- 5. QC Work from Retrieval for Return Stock ---
    QC Work from Retrieval for Return Stock ->> DNSTOCK: Stock Qty: (58+2) → 60

    %% --- 6. Update QC Status ---
    DNSTOCK ->> Update QC Status: Stock Status: QI  
    DNSTOCK ->> Update QC Status: Tempering Flag: Reached 
    DNSTOCK ->> Update QC Status: QC Check Flag: Not Done
    DNSTOCK ->> Update QC Status: QC Duration: (QC Duration + Now)
    
    Update QC Status ->> DNSTOCK: Stock Status: UU  
    Update QC Status ->> DNSTOCK: QC Check Flag: Done
    Update QC Status ->> DNSTOCK: QC Duration: (QC Duration + Now)

    %% --- 7. Extend Tempering Period ---
    DNSTOCK ->> Extend Tempering Period: Stock Status: QI  
    DNSTOCK ->> Extend Tempering Period: Tempering Flag: Reached 
    DNSTOCK ->> Extend Tempering Period: QC Check Flag: Not Done
    DNSTOCK ->> Extend Tempering Period: QC Duration: (QC Duration + Now)

    Extend Tempering Period ->> DNSTOCK: Tempering Period: 72 H + Extend Value
    Extend Tempering Period ->> DNSTOCK: Tempering Flag: Not Reached
:::