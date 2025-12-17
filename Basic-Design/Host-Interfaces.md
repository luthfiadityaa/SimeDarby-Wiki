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

::: mermaid
flowchart TD
 C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]           
 C1 --> HostCommExecutor.java
 AbstractXmlDataLoader.java
 HostCommExecutor.java --> AbstractXmlDataLoader.java

subgraph HostCommExecutor.java
 C2["recvMaterialMasterData()"]
 C3["recvStoragePlanPkgData()"]
 C4["recvRetrievalPlanData()"]
 C5["recvResponseData()"]
 C6["sendProductionStorageData()"]
 C7["sendQCStatusUpdateData()"]
 C8["sendInternalLocTransferData()"]
 C9["sendStorageData()"]
 C10["sendRetrievalData()"]
 C11["sendStorageRetrievalData()"]
end

subgraph AbstractXmlDataLoader.java
 execute        
end

subgraph execute
 C13["connect()"]
 C14["WarenaviSystemController"]
 
 Cond1{"!getLock(sysCon)"} 

 C16["setConfig"]
 C15["finally"]

 C13 --> C14 --> Cond1
 Cond1 --> |False| C16
 Cond1 --> |True| C15   
end

subgraph setConfig
 C21["MaterialMasterDataLoader"]
 C22["StoragePlanPkgDataLoader"]
 C23["RetrievalPlanDataLoader"]
 C24["ResponseDataLoader"]
 C25["ProductionStorageReportData"]
 C26["QCStatusUpdateReportData"]
 C27["InternalLocTransferReportData"]
 C28["StorageReportData"]
 C29["RetrievalReportData"]
 C30["StorageRetrievalReportData"]        
end

:::