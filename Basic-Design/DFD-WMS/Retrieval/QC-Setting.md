[[_TOC_]]
[[_TOSP_]]

**Tempering Period Checker**

- **Tempering Period** will use the last pallet **Storage Date & Time** for calculate
  - `Storage Date & Time`: 2/12/2025 02:00:00
  - `Tempering Period`: 72 Hours

Formula
    Date End: 5/12/2025 02:00:00 
   
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

    %% --- 1. Tempering Periods Passed ---
    DNSTOCK ->> Retrieval for QC Start: Now >= (Storage Date & Time) + (Tempering Period)
    DNSTOCK ->> Retrieval for QC Start: Stock Status: UU  
    DNSTOCK ->> Retrieval for QC Start: Tempering Flag: Not Reached 
    DNSTOCK ->> Retrieval for QC Start: QC Check Flag: Not Done
 


:::