[[_TOC_]]
[[_TOSP_]]

#Receive Method (Data Loader)
**Overall End-to-End Flow Summary (Stage 1 → Stage 8)**
-------------------------------------------------------

| Stage | Responsibility |
| --- | --- |
| [Stage 1](#Stage-1) | Host communication & routing |
| [Stage 2](#Stage-2) | System connection & locking |
| [Stage 3](#Stage-3) | DataLoader selection |
| [Stage 4](#Stage-4) | Java model binding |
| [Stage 5](#Stage-5) | XSD schema validation |
| [Stage 6](#Stage-6) | Environment binding & file resolution |
| [Stage 7](#Stage-7) | File validation, duplication check & processing control |
| [Stage 8](#Stage-8) | XML validation, unmarshalling, and entity conversion |
| [Stage 9](#Stage-9) | Processing finalization, history recording, and flow rollback control |

##Stage 1 - Start Process
Stage 1 describes how the **Host Communication subsystem** handles **incoming and outgoing XML-based data** between an external host system and the internal WMS logic.
::: mermaid
flowchart LR
 C1["serviceHostComm.prj<br>(ConsoleApplicationExecutor)"]  
 C2["HostCommExecutor.java"]
 C13["AbstractXmlDataLoader.java"]
 C1 --> C2
 C2 --> Receiving --> C13

subgraph Receiving
 C3["recvMaterialMasterData()"]
 C4["recvStoragePlanPkgData()"]
 C5["recvRetrievalPlanData()"]
 C6["recvResponseData()"]
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

#####**What happens:**
1.  `HostCommExecutor` triggers sending based on system events    
2.  Each `sendXXX()` method:
    *   Collects data from DB        
    *   Generates XML        
    *   Sends XML to the host        
3.  Uses shared logic from `AbstractXmlDataLoader.java` where applicable
<br>
<hr>

##Stage 2
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

##Stage 3

Stage 3 is responsible for **executing the actual business data loaders** after the system configuration is completed in Stage 2.  

It is **Sending** data processes and finally binds the processed data into a unified model.

::: mermaid
flowchart LR
 E1["setModel()"]
 C1["setConfig()"]  
 C1 --> Receiving -->E1

subgraph Receiving
 C21["MaterialMasterDataLoader"]
 C22["StoragePlanPkgDataLoader"] 
 C23["RetrievalPlanHostDataLoader"] 
 C24["ResponseDataLoader"]
end
:::

### **1. Entry Point: `setConfig()`**
*   The flow starts after **Stage 2 successfully acquires the system lock** and applies configuration.    
*   `setConfig()` determines:
    *   Data type        
    *   Direction (Receiving / Sending)        
    *   Target loader classes

### **2. Receiving Flow (Inbound Data)**
**Objective:**  
Process incoming XML data from the host system and convert it into internal domain objects.

#### **2.1 Receiving Loaders**
Each loader handles **one specific inbound data type**:
*   `MaterialMasterDataLoader`    
*   `StoragePlanPkgDataLoader`    
*   `RetrievalPlanHostDataLoader`    
*   `ResponseDataLoader`
    
#### **2.2 Process**
For each incoming XML file:
1.  Select the corresponding **DataLoader**    
2.  Parse XML    
3.  Validate data    
4.  Convert to domain objects    
5.  Store into internal structures
    
All receiving loaders follow the **same abstract execution pattern** inherited from `AbstractXmlDataLoader`.   

###**4. Model Binding: `setModel()`**
After **Receiving or Sending processing completes**:
*   `setModel()` is invoked    
*   All processed data is:
    *   Consolidated        
    *   Normalized        
    *   Stored into a unified execution model
        
This model is later used for:
*   Database persistence    
*   Transaction handling    
*   Response generation    
*   Logging and auditing

<br>
<hr>

##Stage 4
Stage 4 is responsible for **binding concrete Java model classes** and **preparing XML schema definitions (XSD)** so the system can **validate and process XML data structurally**.

This stage bridges:
> **Data Loader logic (Stage 3)** → **Concrete domain models + schema validation**

::: mermaid
flowchart LR
 E1["setSchema()"]
 C1["setModel()"]  
 C1 --> Receiving --> E1

subgraph Receiving
 C21["MaterialMaster.class"]
 C22["StoragePlanPkg.class"] 
 C23["RetrievalPlan.class"] 
 C24["Response.class"]
end
:::

### **1. Input from Stage 3:** `setModel()`
*   After Stage 3 selects the correct **DataLoader** (Receiving or Sending),    
*   `setModel()` is invoked to determine **which Java classes represent the XML data**.
   
###**2. Receiving Flow (Inbound Data)**

When the system is **receiving data from Host**:

####**2.1 Bound Model Classes**
These classes represent **incoming XML payloads**:
*   `MaterialMaster.class`    
*   `StoragePlanPkg.class`    
*   `RetrievalPlan.class`    
*   `Response.class`    

####**2.2 What Happens Here**

*   Each XML file is mapped to its **corresponding JAXB model**    
*   These classes define:
    *   XML structure        
    *   Field mapping        
    *   Data constraints (length, type, required fields)
        
At this point:
*   XML → Java Object mapping is **fully defined**    
*   But **not yet validated against schema** 

###**4. Schema Preparation: `setSchema()`**
After model binding is complete: `setSchema()` is executed

This step:
*   Associates each model with its **XSD definition**    
*   Prepares **XML schema validation** for the next stage
    
Examples:
*   `MaterialMaster.class` → `MaterialMaster.xsd`    
*   `RetrievalPlan.class` → `RetrievalPlan.xsd`

<br>
<hr>

##Stage 5
Stage 5 finalizes **XML structure validation** and prepares the **execution environment** before actual data processing (parse, validate, DB insert, response generation).

::: mermaid
flowchart LR
 E1["setExEnv()"]
 C1["setSchema()"]  
 C1 --> Receiving --> E1

subgraph Receiving
 C21["MaterialMaster.xsd"]
 C22["StoragePlanPkg.xsd"] 
 C23["RetrievalPlan.xsd"] 
 C24["Response.xsd"]
end
:::

####**1. Input from Previous Stage**

From **Stage 4**, you already have:
*   Java **model classes** (POJO)    
*   Logical mapping between **data types** (Receiving / Sending)
    
Now Stage 5 maps these models to **XML schemas (XSD)**.

####**2. Receiving (Inbound XML Validation)**

The system assigns XSDs used to validate **incoming XML files**:

| XML Type | XSD |
| --- | --- |
| Material Master | `MaterialMaster.xsd` |
| Storage Plan Package | `StoragePlanPkg.xsd` |
| Retrieval Plan | `RetrievalPlan.xsd` |
| Response | `Response.xsd` |

These schemas ensure:
*   Mandatory fields exist    
*   Data types are correct    
*   Length / format rules are enforced
    
This is typically used by:
*   JAXB validation    
*   SAX pre-validation

<br>
<hr>

##Stage 6
At this stage, the system has already:
*   Loaded models    
*   Applied schemas (XSD)    
*   Set execution environment (`setExEnv()`)
    
Now it **binds runtime environment IDs** and **resolves the actual files to be processed or generated**.

::: mermaid
flowchart LR
 E1["getFile()"]
 C1["setExEnv()"]  
 C1 --> Receiving --> E1

subgraph Receiving
 C21["Material (MaterialMaster ID env)"]
 C22["Pl_Stor (StoragePLanPkg ID env)"] 
 C23["ShippingProcess (RetrievalPLan ID env)"] 
 C24["Response (Response ID env)"]
end
:::

### **1. Receiving Flow (Inbound Data Processing)**
Each inbound message is now **fully contextualized** with:
*   Model    
*   Schema    
*   Environment ID
    
**Receiving objects:**
1.  **Material**
    *   Bound with **MaterialMaster ID env**        
    *   Identifies inbound _material master data_
        
2.  **PL_Stor**
    *   Bound with **StoragePlanPkg ID env**        
    *   Represents storage planning packages
        
3.  **ShippingProcess**
    *   Bound with **RetrievalPlan ID env**        
    *   Represents retrieval/shipping instructions
        
4.  **Response**
    *   Bound with **Response ID env**        
    *   Handles acknowledgments and results
        
At this point:
*   Each object is **environment-aware**    
*   Routing and validation rules are now deterministic

### **3. Final Transition → `getFile()`**
Once all objects are:
*   Bound to environment    
*   Fully validated    
*   Properly classified (Receiving / Sending)
    
The flow reaches: `getFile()`

This step:
*   Resolves **physical file names**    
*   Determines **target directories**    
*   Prepares files for:
    *   Writing        
    *   Sending        
    *   Archiving        
    *   Error handling

<br>
<hr>

##Stage 7

Stage 7 is responsible for **file-level control and validation before XML processing**.  
It ensures that only **valid, non-duplicate, correctly named XML files** are passed to JAXB/XML validation.  
Any failure causes a **rollback to Stage 1** (re-waiting for host communication).

::: mermaid
flowchart LR
 C1["getFile()"] --> Cond1{"checkDataHostDirectory() ?"}
 Cond1 --> |FALSE| C3[RollBack to Stage 1]
 Cond1 --> |TRUE| Cond2{"listFiles() ?"}  
 Cond2 --> |FALSE| C3
 Cond2 --> |TRUE| Cond3{"checkFileName() ?"} 
 Cond3 --> |FALSE| C1
 Cond3 --> |TRUE| Cond4{"checkDuplicatedFile() ?"} 
 Cond4 --> |TRUE| C1
 Cond4 --> |FALSE| C4["convertToEntity() -> validateXml()"]
:::

**Detailed Flow Description**

### **1. getFile()**
*   The process starts by requesting a file candidate from the host directory.    
*   This function does **not assume the file exists yet**.    

### **2. checkDataHostDirectory()**
*   Verifies whether the **host data directory exists and is accessible**.    
**Decision:**
*   **FALSE** → Roll back to **Stage 1**  
    (System waits or reinitializes host communication)    
*   **TRUE** → Continue to `listFiles()`

### **3. listFiles()**
*   Retrieves the list of files from the host directory.
**Decision:**
*   **FALSE (empty or error)** → Roll back to **Stage 1**    
*   **TRUE** → Continue to `checkFileName()`

### **4. checkFileName()**
*   Validates file naming rules, such as:
    *   Prefix (e.g., `Material_`)        
    *   ID format        
    *   Timestamp        
    *   Sequence number        
    *   Extension (`.xml`)
        
**Decision:**
*   **FALSE** → Return to `getFile()`  
    (Skip invalid file and continue scanning)    
*   **TRUE** → Continue to `checkDuplicatedFile()`

### **5. checkDuplicatedFile()**

*   Determines whether the file:
    *   Has already been processed        
    *   Exists in history / backup / error directories        
    *   Has the same unique key (ID + timestamp)
        
**Decision:**
*   **TRUE (duplicate)** → Return to `getFile()`  
    (Ignore duplicate and scan next file)    
*   **FALSE (new file)** → Continue to processing

### **6. convertToEntity()**
*   Converts XML file into Java objects (JAXB unmarshalling).    
*   Maps XML elements to domain models.

### **7. validateXml()**

*   Performs validation such as:
    *   XSD validation        
    *   Mandatory field checks        
    *   Length / type validation        
    *   Business rule validation
        
<br>
<hr>

##Stage 8

Stage 8 is to validate incoming XML data against its XSD schema and convert it into Java objects with strict validation control.

::: mermaid
flowchart LR
 C1["AbstractXmlDataLoader.java --> ValidationUtil.validateXml()"]
 C1 --> C2
 C9 --> Cond1{"isValidationOK ?"}

subgraph ValidationUtil 
 C2 --> C3 --> C4 --> C5 --> C6 --> C7 --> C8 --> C9
 C2["Load the XML Schema --> newSchema()"]
 C3["Create a JAXB Context and Unmashaller --> createUnmarshaller()"]
 C4["setTargetRootElement()"]
 C5["Apply Schema for Validation --> setSchema()"]
 C6["Set Custom Handler --> setEventHandler()"]
 C7["Setup SAX Reader --> newSAXParser.getXMLReader()"]
 C8["Inject the Custom Filter --> new ValidationUtil()"]
 C9["Perform Unmarshalling and Validation --> unmarshal()"]
end

click C2 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/833/Host-Interfaces?anchor=stage-5"
click C4 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/833/Host-Interfaces?anchor=stage-4"
style C2 fill:#00cc66,stroke:#006633,color:#ffffff
style C4 fill:#00cc66,stroke:#006633,color:#ffffff
:::

**Detailed Flow Description**

### **1. Entry Point**
- `AbstractXmlDataLoader.java`        
- Calls `ValidationUtil.validateXml()`
        
### **2. Load XML Schema**
- Load the corresponding XSD schema (`newSchema`)
- Schema is selected based on data type (Material, StoragePlan, etc.)
        
### **3. Create JAXB Context**
- Initialize `JAXBContext`
- Create `Unmarshaller`
        
### **4. Set Target Root Element**
- Define the expected root XML element (Selected Model Class)
- Prevents incorrect document structure
        
### **5. Apply Schema Validation**
- Attach XSD schema to the unmarshaller
- Enables schema-based validation during parsing
        
### **6. Set Custom Validation Handler**
- Register custom `ValidationEventHandler`
- Captures:
  - Validation error message
  - XML line number
  - Column number
            
### **7. Setup SAX Reader**
- Initialize `SAXParser`
- Create `XMLReader`
        
### **8. Inject Custom Validation Filter**
- Attach `ValidationUtil` as a filter
- Enables enhanced error tracking and filtering logic
        
### **9. Unmarshal XML**
- Parse XML
- Convert XML → Java model
- Validate structure and data at the same time
        
### **10. Validation Result Check**
- Decision: `isValidationOK?`
  - ❌ **No** → validation exception raised            
  - ✅ **Yes** → proceed to next stage

##Stage 9 - End Process

Stage 9 is to **finalize processing**, record execution results, manage database transactions, and control rollback or continuation based on validation and processing outcomes.

::: mermaid
flowchart LR
Cond1{"isValidationOK ?"}
Cond1 --> |TRUE| B2
Cond1 --> |FALSE| C2

subgraph Error
 C2 --> C3 
 C2 --> C4
 C4[(DNExchangeHistory)]
 C3[(DNLoadErrorInfo)]
 C2["Set Exchange History as Error Data"] 
end

subgraph Process
 B2 --> B3 --> Cond2
 Cond2{"isProcess ?"}
 Cond3{"isException ?"}
 B2["Load the Selected Data Loader Class --> process()"]
 B3["Validation Process --> validationBeforeInsert()"]
 B4["Set Exchange History as Error Data or Skip Data"]
 B5["Set Exchange History as Normal"]
 B6[(DNLoadErrorInfo)]
 B7[(DNExchangeHistory)]
 Cond2 --> |TRUE| B5 --> B7 --> D2
 B4 --> B7
 Cond2 --> |FALSE| Cond3
 Cond3 --> |FALSE| B4 --> |Insert Data| B6 --> D2
 Cond3 --> |TRUE| D2
end 

subgraph Exception
 D2 --> D3 
 D2["Rollback"]
 D3["Go to the Stage 2"] 
end

click B2 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/833/Host-Interfaces?anchor=stage-3"
style B2 fill:#00cc66,stroke:#006633,color:#ffffff
click D3 "https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/833/Host-Interfaces?anchor=stage-2"
style D3 fill:#00cc66,stroke:#006633,color:#ffffff
:::

**Detailed Flow Description**

### 1. Entry Condition
- Input decision: **`isValidationOK?`**
- Result from **Stage 8 (XML validation & unmarshalling)**

### 2. Validation Failed (`FALSE`)

If validation fails:
1.  **Set Exchange History as Error Data**    
2.  **Insert error records**
    - `DNLoadErrorInfo`
    - `DNExchangeHistory`        
3.  **End current file processing**
    
➡ Flow continues to the **next execution cycle**

### 3. Validation Passed (`TRUE`)
If validation passes:
1.  **Load Selected Data Loader**
    - Execute `process()`        
2.  **Business Validation**
    - `validationBeforeInsert()`
       
### 4. Process Decision (`isProcess?`)
#### a. Processing Successful (`TRUE`)
1.  **Set Exchange History as Normal**    
2.  **Insert into `DNExchangeHistory`**
3.  **Processing ends normally**

#### b. Processing Failed (`FALSE`)
Continue to exception evaluation

### 5. Exception Check (`isException?`)
#### a. No Exception (`FALSE`)
*   Data is skipped or business rule failed    
*   **Set Exchange History as Error / Skip Data**    
*   Insert into:
    - `DNLoadErrorInfo`
    - `DNExchangeHistory`
        
➡ End current file processing

#### b. Exception Occurred (`TRUE`)
1.  **Insert error information**
    - `DNLoadErrorInfo`
        
2.  **Control-flow rollback**
    *   Processing flow returns to **Stage 2**        
    *   System reconnects and reinitializes
        
➡ Database state remains unchanged