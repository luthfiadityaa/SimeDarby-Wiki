[[_TOC_]]
[[_TOSP_]]

::: mermaid
sequenceDiagram
    autonumber

    participant Retrieval for QC Start
    participant QC Work from Retrieval for QC Start
    participant Retrieval for Return Stock
    participant QC Work from Retrieval for Return Stock
    participant Update QC Status
    participant Extend Tempering Period

    %% --- 1. Retrieval for QC Start ---
    Retrieval for QC Start ->> QC Work from Retrieval for QC Start: Input StartDate, TemperingPeriod=72H
    Retrieval for QC Start ->> QC Work from Retrieval for QC Start: Input StartDate, TemperingPeriod=72H

:::