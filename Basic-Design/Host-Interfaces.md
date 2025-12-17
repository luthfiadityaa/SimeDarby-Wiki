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

# Detail Flow Process
The trigger to start the process is this file.
`C:\daifuku\wms\tomcat\webapps\wms\serviceHostComm.prj`

**Stage 1**

::: mermaid
flowchart LR
 C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]  
 C2["HostCommExecutor.java"]
 C13["AbstractXmlDataLoader.java"]
 C1 --> C2
 C2 --> Receiving --> C13
 C2 --> Sending --> C13

subgraph Receiving
 C3["recvMaterialMasterData()"]
 C4["recvStoragePlanPkgData()"]
 C5["recvRetrievalPlanData()"]
 C6["recvResponseData()"]
end

subgraph Sending
 C7["sendProductionStorageData()"]
 C8["sendQCStatusUpdateData()"]
 C9["sendInternalLocTransferData()"]
 C10["sendStorageData()"]
 C11["sendRetrievalData()"]
 C12["sendStorageRetrievalData()"]
end
:::

**Stage 2**
::: mermaid
flowchart LR
 C1["AbstractXmlDataLoader.java"]
 C1 --> execute

subgraph execute
 C2 --> C3 --> Cond1

 C2["connect()"]
 C3["WarenaviSystemController"]
 Cond1{"!getLock(sysCon) ?"} 
 Cond1 --> |FALSE| execute
 Cond1 --> |TRUE| Rollback1
end
:::
