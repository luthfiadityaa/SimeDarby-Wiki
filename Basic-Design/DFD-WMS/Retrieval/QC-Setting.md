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
    P1 ->> P1: Input StartDate, TemperingPeriod=72H
    P2 ->> P2: Calculate DateEnd = StartDate + 72H
    P2 ->> StockDB: Save StockStatus=UU,\nTemperingFlag=Not Reached,\nQCFlag=Not Done,\nDateStart, DateEnd

:::