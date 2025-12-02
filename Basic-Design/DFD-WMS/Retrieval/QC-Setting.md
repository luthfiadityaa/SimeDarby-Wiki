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
    P1 ->> System: Input StartDate, TemperingPeriod=72H
    P2 ->> System: Calculate DateEnd = StartDate + 72H
    P2 ->> StockDB: Save StockStatus=UU,\nTemperingFlag=Not Reached,\nQCFlag=Not Done,\nDateStart, DateEnd

    P3 ->> System: Now() >= DateEnd ?
    P2 -->> System: If true → ready for QC Start
    P2 ->> User: Show in Retrieval for QC Start

    %% --- 2. QC Work from Retrieval for QC Start ---
    P1 ->> System: Retrieve Stock (Filter: Status=UU)
    P2 ->> StockDB: Update\nStockStatus=QI\nTemperingFlag=Reached\nQCFlag=Not Done\nQC Duration Start=Now()
    P2 ->> StockDB: Stock Qty: 60 → 58
    P2 ->> QCDB: Record QC Start

    P1 -->> System: If TemperingFlag == Reached
    P1 ->> User: Move to Retrieval for Return Stock

    %% --- 3. Retrieval for Return Stock ---
    P1 ->> System: Retrieve Stock (Filter: Status=QI)
    P2 ->> StockDB: Update Stock Qty: 58 → 60

    %% --- 4. QC Work from Retrieval for Return Stock ---
    P1 ->> System: Update QC Status (Filter: Status=QI)
    P2 ->> StockDB: Update\nStockStatus=UU\nQC Check Flag=Done\nQC Duration Stop=Now()
    P2 ->> QCDB: Save QC Completion

    %% --- 5. Update QC Status ---
    P1 ->> System: Extend Tempering Period (Filter: Status=QI & QC Not Done)
    P2 ->> StockDB: Update\nTemperingPeriod=72H + ExtendValue\nTemperingFlag=Not Reached

    %% --- 6. Extend Tempering Period ---
    P1 ->> System: Extend Tempering Period (Filter: Status=QI & QC Not Done)
    P2 ->> StockDB: Update\nTemperingPeriod=72H + ExtendValue\nTemperingFlag=Not Reached

:::