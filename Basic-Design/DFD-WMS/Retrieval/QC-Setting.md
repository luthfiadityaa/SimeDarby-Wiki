[[_TOC_]]
[[_TOSP_]]

::: mermaid
sequenceDiagram
    autonumber

    participant Retrieval for Return Stock as P1
    participant QC Work from Retrieval for QC Start as P2
    participant Retrieval for Return Stock as P3
    participant QC Work from Retrieval for Return Stock as P4
    participant Update QC Status as P5
    participant Extend Tempering Period as P6

    %% --- 1. Retrieval for Return Stock ---
    User ->> System: Input StartDate, TemperingPeriod=72H
    System ->> System: Calculate DateEnd = StartDate + 72H
    System ->> StockDB: Save StockStatus=UU,\nTemperingFlag=Not Reached,\nQCFlag=Not Done,\nDateStart, DateEnd

    Timer ->> System: Now() >= DateEnd ?
    System -->> System: If true → ready for QC Start
    System ->> User: Show in Retrieval for QC Start

    %% --- 2. QC Work from Retrieval for QC Start ---
    User ->> System: Retrieve Stock (Filter: Status=UU)
    System ->> StockDB: Update\nStockStatus=QI\nTemperingFlag=Reached\nQCFlag=Not Done\nQC Duration Start=Now()
    System ->> StockDB: Stock Qty: 60 → 58
    System ->> QCDB: Record QC Start

    System -->> System: If TemperingFlag == Reached
    System ->> User: Move to Retrieval for Return Stock

    %% --- 3. Retrieval for Return Stock ---
    User ->> System: Retrieve Stock (Filter: Status=QI)
    System ->> StockDB: Update Stock Qty: 58 → 60

    %% --- 4. QC Work from Retrieval for Return Stock ---
    User ->> System: Update QC Status (Filter: Status=QI)
    System ->> StockDB: Update\nStockStatus=UU\nQC Check Flag=Done\nQC Duration Stop=Now()
    System ->> QCDB: Save QC Completion

    %% --- 5. Update QC Status ---
    User ->> System: Extend Tempering Period (Filter: Status=QI & QC Not Done)
    System ->> StockDB: Update\nTemperingPeriod=72H + ExtendValue\nTemperingFlag=Not Reached

    %% --- 6. Extend Tempering Period ---
    User ->> System: Extend Tempering Period (Filter: Status=QI & QC Not Done)
    System ->> StockDB: Update\nTemperingPeriod=72H + ExtendValue\nTemperingFlag=Not Reached

:::