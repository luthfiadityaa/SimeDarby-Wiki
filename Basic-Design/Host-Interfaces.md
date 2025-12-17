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

This stage describes how the **Host Communication subsystem** handles **incoming and outgoing XML-based data** between an external host system and the internal WMS logic.
It is divided into **Receiving** and **Sending** processes, coordinated by `HostCommExecutor.java`.
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
flowchart TD
 C1["AbstractXmlDataLoader.java"]
 C1 --> execute

subgraph execute
 C2 --> C3 --> Cond1
 C2["connect()"]
 C3["WarenaviSystemController"]
 Cond1{"!getLock(sysCon) ?"} 
 Cond1 --> |FALSE| C2
 Cond1 --> |TRUE| C4["setConfig()"]
end
:::

**Stage 3**
::: mermaid
flowchart LR
 E1["setModel()"]
 C1["setConfig()"]  
 C1 --> Receiving -->E1
 C1 --> Sending -->E1

subgraph Receiving
 C21["MaterialMasterDataLoader"]
 C22["StoragePlanPkgDataLoader"] 
 C23["RetrievalPlanDataLoader"] 
 C24["ResponseDataLoader"]
end

subgraph Sending
 C25["ProductionStorageReportData"]
 C26["QCStatusUpdateReportData"]
 C27["InternalLocTransferReportData"] 
 C28["StorageReportData"]
 C29["RetrievalReportData"] 
 C30["StorageRetrievalReportData"] 
end 
:::

**Stage 4**
::: mermaid
flowchart LR
 E1["setSchema()"]
 C1["setModel()"]  
 C1 --> Receiving --> E1
 C1 --> Sending --> E1

subgraph Receiving
 C21["MaterialMaster.class"]
 C22["StoragePlanPkg.class"] 
 C23["RetrievalPlan.class"] 
 C24["Response.class"]
end

subgraph Sending
 C25["ProductionStorage.class"]
 C26["QCStatusUpdate.class"]
 C27["InternalLocTransfer.class"] 
 C28["StorageResult.class"]
 C29["RetrievalResult.class"] 
 C30["StorageRetrievalResult.class"] 
end 
:::

**Stage 5**
::: mermaid
flowchart LR
 E1["setExEnv()"]
 C1["setSchema()"]  
 C1 --> Receiving --> E1
 C1 --> Sending --> E1

subgraph Receiving
 C21["MaterialMaster.xsd"]
 C22["StoragePlanPkg.xsd"] 
 C23["RetrievalPlan.xsd"] 
 C24["Response.xsd"]
end

subgraph Sending
 C25["ProductionStorage.xsd"]
 C26["QCStatusUpdate.xsd"]
 C27["InternalLocTransfer.xsd"] 
 C28["StorageResult.xsd"]
 C29["RetrievalResult.xsd"] 
 C30["StorageRetrievalResult.xsd"] 
end 
:::
