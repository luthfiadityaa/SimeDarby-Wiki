[[_TOC_]]
[[_TOSP_]]

::: mermaid
sequenceDiagram
    autonumber

    participant Retrieval for Return Stock
    participant QC Work from Retrieval for QC Start
    participant Retrieval for Return Stock
    participant QC Work from Retrieval for Return Stock
    participant Update QC Status
    participant Extend Tempering Period

    %% --- 1. Palletize Storage to Tempering ---
    User ->> System: Input StartDate, TemperingPeriod=72H
    System ->> System: Calculate DateEnd = StartDate + 72H
    System ->> StockDB: Save StockStatus=UU,\nTemperingFlag=Not Reached,\nQCFlag=Not Done,\nDateStart, DateEnd

    Timer ->> System: Now() >= DateEnd ?
    System -->> System: If true → ready for QC Start
    System ->> User: Show in Retrieval for QC Start

    %% --- 2. Retrieval for QC Start ---
    User ->> System: Retrieve Stock (Filter: Status=UU)
    System ->> StockDB: Update\nStockStatus=QI\nTemperingFlag=Reached\nQCFlag=Not Done\nQC Duration Start=Now()
    System ->> StockDB: Stock Qty: 60 → 58
    System ->> QCDB: Record QC Start

    System -->> System: If TemperingFlag == Reached
    System ->> User: Move to Retrieval for Return Stock

    %% --- 3. Retrieval for Return Stock ---
    User ->> System: Retrieve Stock (Filter: Status=QI)
    System ->> StockDB: Update Stock Qty: 58 → 60

    %% --- 4. Update QC Status ---
    User ->> System: Update QC Status (Filter: Status=QI)
    System ->> StockDB: Update\nStockStatus=UU\nQC Check Flag=Done\nQC Duration Stop=Now()
    System ->> QCDB: Save QC Completion

    %% --- 5. Extend Tempering Period ---
    User ->> System: Extend Tempering Period (Filter: Status=QI & QC Not Done)
    System ->> StockDB: Update\nTemperingPeriod=72H + ExtendValue\nTemperingFlag=Not Reached

:::