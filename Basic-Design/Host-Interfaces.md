[[_TOC_]]

# Overview

| No.  | Item (Data Name)                                         | Name on Prodction Spec.       | Class name                                                                      | When          | 
|------|----------------------------------------------------------|-------------------------------|---------------------------------------------------------------------------------|---------------|
| 1    | Material Master Data                                     | Material Master Data          | [MaterialMasterDataLoader](#5737)                                 | Every 10 min  |
| 2    | Planned Storage                                          | Planned Storage               | [PlannedStorageReceive](#5117)                                 | Every 5 min   |
| 3    | Planned Retrieval                                        | Planned Retrieval             | [PlannedRetrievalReceive](#5749)                             | Every 5 min   |
| 4    | Response                                                 | SAP Response                  | [ResponseReceive](#ResponseReceive)                                                                                | Every Result  | 
| 5    | Production Storage Result                                | Production Storage Result     | [ProductionStorageSend](#productionstoragesend)                                 | Once has Data |
| 6    | QC Status Update Result                                  | QC Status Update              | [QCStatusUpdateSend](#qcsatatusupdatesend)                                      | Once has Data |
| 7    | Internal Location Transfer Result                        | Internal Location Transfer    | [InternalLocationTransferSend](#internallocationtransferesend)                  | Once has Data |
| 8    | Planned Storage Result                                   | Planned Storage Result        | [PlannedStorageSend](#plannedstoragesend)                                       | Once has Data |
| 9    | Planned Retrieval Result                                 | Planned Retrieval Result      | [PlannedRetrievalSend](#plannedretrievalsend)                                   | Once has Data |
| 10   | Unplanned Storage Result                                 | Unllanned Storage Result      | [UnplannedStorageRetrievalSend](#5752)                                   | Once has Data |
| 11   | Unplanned Retrieval Result                               | Unplanned Retrieval Result    | [UnplannedStorageRetrievalSend](#5752)                               | Once has Data |