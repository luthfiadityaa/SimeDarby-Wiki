[[_TOC_]]
[[_TOSP_]]

# **QC Summary Flow**   

## **Tempering Period**
Tempering Period will use the **Storage Date & Time** for calculate.
   
  **DNSTOCK** data as example:
  - `Storage Date & Time`: 2/12/2025 02:00:00
  - `Tempering Period`: 72 Hours

  The tempering period is over:
  - `Tempering Period Date End`: (Storage Date & Time) + (Tempering Period) → **5/12/2025 02:00:00** 

  The tempering period will remain at `72 hours` even if the original end date has passed. Any changes will only be applied upon approval to `Extend the Tempering Period`. 


## **Tempering Flag Checker**
@<5841075A-3462-65B0-8512-76DD7AE397C1> 
`jp.co.daifuku.asrs.transmission.ShelfMonitor`
Please see this process and create the checker process.


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
    DNSTOCK ->> Retrieval for QC Start: Now >= Tempering Period Date End <br><br> Stock Status: UU <br> Tempering Flag: Not Reached <br> QC Check Flag: Not Done <br> Stock Qty: 60


    %% --- 2. Retrieval for QC Start ---
    Retrieval for QC Start ->> QC Work from Retrieval for QC Start: Qty To Pick?  2

    %% --- 3. QC Work from Retrieval for QC Start ---
    QC Work from Retrieval for QC Start ->> DNSTOCK: Stock Qty: (60-2) → 58 <br> QC Duration: (Newest Storage Date + Now) <br> Stock Status: QI <br> Tempering Flag: Reached

    %% --- 4. Retrieval for Return Stock ---
    DNSTOCK ->> Retrieval for Return Stock: Stock Status: QI <br> Tempering Flag: Reached <br> QC Check Flag: Not Done <br> Stock Qty: 58

    Retrieval for Return Stock ->> QC Work from Retrieval for Return Stock: Qty To Add?  2

    %% --- 5. QC Work from Retrieval for Return Stock ---
    QC Work from Retrieval for Return Stock ->> DNSTOCK: Stock Qty: (58+2) → 60

    %% --- 6. Update QC Status ---
    DNSTOCK ->> Update QC Status: Stock Status: QI <br> Tempering Flag: Reached <br> QC Check Flag: Not Done <br> QC Duration: (QC Duration + Now)
    
    Update QC Status ->> DNSTOCK: Stock Status: UU <br> QC Check Flag: Done <br> QC Duration: (QC Duration + Now)

    %% --- 7. Extend Tempering Period ---
    DNSTOCK ->> Extend Tempering Period: Stock Status: QI <br> Tempering Flag: Reached <br> QC Check Flag: Not Done <br> QC Duration: (QC Duration + Now)

    Extend Tempering Period ->> DNSTOCK: Tempering Period: 72 H + Extend Value (3) → 75 <br> Tempering Flag: Not Reached
:::

**Process 1 — Retrieval for QC Start**
--------------------------------------

**Filter:** 
`Now() >= (Storage Date + Tempering Period)`
`StockStatus = UU`
`TemperingFlag = 0` 

**Data will be showing following criteria:**
* Tempering Period: 72 H    
* QC Check Flag: Not Done
* Stock Status: UU
* Tempering Flag: Reached
* Stock Qty: 60
    
**Process 2 — QC Work from Retrieval for QC Start**
--------------------------------------
**Keep (Retain):**
*   Tempering Period: 72 H    
*   QC Check Flag: Not Done

**Input:**
* Take out Qty: 2
    
**Change:**
*   Stock Status → QI    
*   Tempering Flag → Reached    
*   QC Duration → Newest Storage Date + Now()    
*   Stock Qty → 58

**Process 3 — Retrieval for Return Stock**
------------------------------------------

**Filter:** 
`StockStatus = QI`

**Data will be showing following criteria:**
*   Tempering Period: 72 H    
*   QC Check: Not Done    
*   StockStatus: QI    
*   TemperingFlag: Reached
*   Stock Qty: 58
*   QC Duration → QC Duration + Now()

**Process 4 — QC Work from Retrieval for Return Stock**
--------------------------------------
**Keep (Retain):**
*   Tempering Period: 72 H    
*   QC Check: Not Done    
*   StockStatus: QI    
*   TemperingFlag: Reached
*   Stock Qty: 58

**Input:**
* Qty to Add: 2
    
**Change:** 
*   Stock Qty → 60
*   QC Duration → QC Duration + Now()

**Process 5 — Update QC Status**
--------------------------------

**Filter:** 
`StockStatus = QI`

**Keep:**
*   Tempering Period: 72 H    
*   Tempering Flag: Reached
*   Stock Qty: 60
    
**Change:**
*   Stock Status → UU    
*   QC Check → Done    
*   QC Duration Stop → QC Duration + Now()

**Process 6 — Extend Tempering Period**
---------------------------------------

**Filter:** 
`Status Status: QI`
`QC Check Flag: Not Done`

**Keep (Retain):** 
*   Stock Qty: 60

**Input:**
* Extended Value: 3

**Change:**
*   Tempering Period → 75 H      
*   Tempering Flag → Not Reached