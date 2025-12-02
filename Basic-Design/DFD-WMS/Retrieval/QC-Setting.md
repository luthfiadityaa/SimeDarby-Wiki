[[_TOC_]]
[[_TOSP_]]

**Tempering Period Checker**

::: mermaid
sequenceDiagram
    autonumber

    participant Tempering Periods Passed
    participant Retrieval for QC Start
    participant QC Work from Retrieval for QC Start
    participant Retrieval for Return Stock
    participant QC Work from Retrieval for Return Stock
    participant Update QC Status
    participant Extend Tempering Period

    %% --- 1. Tempering Periods Passed ---
    Tempering Period Passed ->> Retrieval for QC Start: Input StartDate, TemperingPeriod=72H


:::