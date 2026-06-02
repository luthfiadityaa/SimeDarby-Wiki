_TOC_
_TOSP_

# #TC-PSEP-C001 — Empty Pallet from HP Area

> [!NOTE]
> **Design Document Reference:** `Storage-Process / Data-Flow` & `[SCREEN]-Empty-Pallet-Storage`
> This Combine Test scenario is designated to validate the End-to-End (E2E) integration of the Empty Pallet Putaway operation initiated from the Home Position / Hand Pallet (HP) Induction Area into the AS/RS High-Bay Warehouse. The testing scope ensures strict compliance with **Softzone Allocation** constraints and seamless Material Flow Control (MFC/AGC) tracking from the source induction points up to the final rack storage.

---

## ## 1. Test Metadata & Intralogistics Specifications

| System Component | Specification Details |
| :--- | :--- |
| **Test Type** | 🟢 Combine Test (UI, Business Logic & MFC/AGC Integration) |
| **DFD / Architecture Ref** | `Storage-Process` / `Data-Flow` |
| **Screen Class** | `jp.co.daifuku.wms.web.display.storage.EmptyPalletStorageSCH` |
| **Job Type** | Empty Pallet Storage (`JOB_TYPE` = `02` / Putaway) |
| **Operator Class** | `StorageOperator` → `AbstractStorageOperator` |
| **Source Induction Point** | 🏭 HP Stations: `1106` / `1302` / `1303` (Hand Pallet Feeders) |
| **Gateway Station** | 🛠️ `ID26` (Main Supply Conveyor Entry Hub) |
| **Softzone Constraint** | 🛑 Dedicated Empty Pallet Zone (`SOFT_ZONE` = `99` / `SZ-EMPTY`) |
| **Destination Area** | AS/RS High-Bay Rack Location (Filtered via Softzone Matrix) |
| **Execution Priority** | ⚡ High |
| **Overall Status** | `[ ] Pass`  `[ ] Fail` |

---

## 3. Pre-conditions & Master Data Configuration
3.1 Material Handling Equipment (MHE) Status
1. The conveyor subsystems bridging the HP stations (1106/1302/1303) to gateway ID26 are fully operational and set to Online / Automatic Mode.

2. The WMS-AGC Communication Gateway (Telegram Logger) is active and ready to intercept network traffic.

3.2 Inventory & Location Master Setup
- Ensure that specific coordinates within the MMLOCATION table matching the empty pallet profile are set to open status.


## ## 2. Material Flow & Routing Architecture (Conveyor Tracking Diagram)

The following diagram illustrates the physical and systematic route of an empty pallet shifting from the HP induction lines, passing the gateway check, and entering the storage rack via the Stacker Crane (SRM):

```mermaid
graph TD
    subgraph HP_Induction_Zone [HP Feeders Area]
        ST1106[Induction Station 1106] --> ID26
        ST1302[Induction Station 1302] --> ID26
        ST1303[Induction Station 1303] --> ID26
    end

    subgraph MHE_Conveyor_Routing [Material Handling Equipment Route]
        ID26[Gateway Hub: ID26] --> ID68[Tracking Spur: ID68]
        ID68 --> ID64[BCR & Softzone Verification: ID64]
        ID64 --> ID33[Accumulation Buffer Section: ID33]
        ID33 --> ID32[SRM Shuttle Station: ID32]
        ID32 --> ID12[AS/RS Stacker Crane Putaway: ID12]
    end

    style ID26 fill:#f96,stroke:#333,stroke-width:2px
    style ID64 fill:#bbf,stroke:#333,stroke-width:2px
    style ID12 fill:#bfb,stroke:#333,stroke-width:2px