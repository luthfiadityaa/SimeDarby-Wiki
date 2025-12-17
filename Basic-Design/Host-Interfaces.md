[[_TOC_]]
[[_TOSP_]]

# **Overview**

## **DNExchangeEnvironment Definition**
<br>

![==image_0==.png](/.attachments/==image_0==-2a2df44c-c65d-4ed7-817a-18cee6b21c67.png) 

## **Overall Validation**
<br>

![image.png](/.attachments/image-c7afb3c6-c8fe-4a35-b607-423d4299a214.png)

## **Exchange Table**

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

##**Stage 1**

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

###**1. serviceHostComm.prj**
*   Acts as the **application entry point**    
*   Launched via `ConsoleApplicationExecutor`    
*   Initializes the Host Communication process

###**2. HostCommExecutor.java**
*   Central **orchestrator**    
*   Determines **what type of data** to process    
*   Routes execution to:
    *   **Receiving flow** (incoming data from host)        
    *   **Sending flow** (outgoing data to host)

####**2.1 Receiving Flow (Inbound Data)**
This section handles **data received from the external host system**.

**Receiving functions:**
*   `recvMaterialMasterData()`    
*   `recvStoragePlanPkgData()`    
*   `recvRetrievalPlanData()`    
*   `recvResponseData()`    

#####**What happens:**
1.  Host sends XML data    
2.  `HostCommExecutor` detects the data type    
3.  Corresponding `recvXXX()` method is called    
4.  Each method:
    *   Reads XML files        
    *   Performs validation        
    *   Passes control to `AbstractXmlDataLoader.java`

####**2.2 Sending Flow (Outbound Data)**
This section handles **data sent from WMS to the host system**.

**Sending functions:**
*   `sendProductionStorageData()`    
*   `sendQCStatusUpdateData()`    
*   `sendInternalQCTransferData()`    
*   `sendStorageData()`    
*   `sendRetrievalData()`    
*   `sendStorageRetrievalData()`    

#####**What happens:**
1.  `HostCommExecutor` triggers sending based on system events    
2.  Each `sendXXX()` method:
    *   Collects data from DB        
    *   Generates XML        
    *   Sends XML to the host        
3.  Uses shared logic from `AbstractXmlDataLoader.java` where applicable
<br>
<hr>

##**Stage 2**
Stage 2 describes how incoming XML data is processed safely by the system, ensuring **exclusive access (locking)** before any configuration or business processing continues.

This stage focuses on **execution control, system connection, and lock validation**.

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

### **1. Entry Point**
**`AbstractXmlDataLoader.java`**
*   Acts as the **central controller** for XML data processing.    
*   Responsible for managing execution flow and system safety checks.

### **2. Execute Process**
**`execute()`**
*   This method initiates the processing lifecycle.    
*   It ensures all preconditions are met before data handling continues.

### **3. System Connection**
**`connect()`**
*   Attempts to establish a connection with the **Warenavi system**.    
*   This step is required before any lock or configuration actions.

### **4. System Controller Interaction**
**`WarenaviSystemController`**
*   Manages system-level operations.    
*   Provides lock control to prevent concurrent processing conflicts.
    
### **5. Lock Validation Decision**
**Decision: `!getLock(sysCon)?`**
*   The system checks whether it can acquire an **exclusive lock**.

#### **If FALSE (Lock acquired)**
*   The process:
    *   Returns back to `connect()`        
    *   Retries connection and lock acquisition        
*   This loop prevents unsafe concurrent execution.    

#### **If TRUE (Lock not acquired)**
*   The system proceeds safely to the next step.    

### **6. Configuration Setup**
**`setConfig()`**
*   System configuration is initialized.    
*   Environment and execution parameters are prepared.    
*   At this point, the system is:
    *   Connected        
    *   Unlocked        
    *   Safe to process data

<br>
<hr>

**Stage 3**

Stage 3 is responsible for **executing the actual business data loaders** after the system configuration is completed in Stage 2.  

It separates **Receiving** and **Sending** data processes and finally binds the processed data into a unified model.

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

<br>
<hr>

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
