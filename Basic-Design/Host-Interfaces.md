[[_TOC_]]
[[_TOSP_]]

# Overview

# DNExchangeEnvironment
![==image_0==.png](/.attachments/==image_0==-2a2df44c-c65d-4ed7-817a-18cee6b21c67.png) 

#Validation
![image.png](/.attachments/image-c7afb3c6-c8fe-4a35-b607-423d4299a214.png)

| No.  | Item (Data Name)                                         | Name on Prodction Spec.       | Class name                                                                      | When          | 
|------|----------------------------------------------------------|-------------------------------|---------------------------------------------------------------------------------|---------------|
| 1    | Material Master Data                                     | Material Master Data          | [MaterialMasterDataLoader](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/835/Material-Master-Data)                                 | Every 10 min  |
| 2    | Planned Storage PKG                                          | Planned Storage               | [StoragePlanPkgDataLoader](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/836/Planned-Storage-PKG)                                 | Every 5 min   |
| 3    | Planned Retrieval                                        | Planned Retrieval             | [RetrievalPlanDataLoader](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/838/Planned-Retrieval)                             | Every 5 min   |
| 4    | Response                                                 | SAP Response                  | [ResponseDataLoader](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/866/SAP-Response)                                                                                | Every Result  | 
| 5    | Production Storage Result                                | Production Storage Result     | [ProductionStorageReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/840/Production-Storage-Result)                                 | Once has Data |
| 6    | QC Status Update Result                                  | QC Status Update              | [QCStatusUpdateReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/842/QC-Status-Update-Result)                                      | Once has Data |
| 7    | Internal Location Transfer Result                        | Internal Location Transfer    | [InternalLocTransferReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/844/Internal-Location-Transfer-Result)                  | Once has Data |
| 8    | Planned Storage Result                                   | Planned Storage Result        | [StorageReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/846/Planned-Storage-Result)                                       | Once has Data |
| 9    | Planned Retrieval Result                                 | Planned Retrieval Result      | [RetrievalReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/848/Planned-Retrieval-Result)                                   | Once has Data |
| 10   | Unplanned Storage Result                                 | Unllanned Storage Result      | [StorageRetrievalReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result?anchor=storageretievalreportdata)                                   | Once has Data |
| 11   | Unplanned Retrieval Result                               | Unplanned Retrieval Result    | [StorageRetrievalReportData](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result?anchor=storageretievalreportdata)                               | Once has Data |

# Summary Flow Process
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
