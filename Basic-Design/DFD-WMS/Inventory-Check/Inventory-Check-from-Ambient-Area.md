[[_TOC_]]
[[_TOSP_]]

# Inventory Check from Ambient Area
![image.png](/.attachments/image-0a629e0c-d766-4b61-aba2-f968f908bd60.png)

# Summary Flow
::: mermaid
flowchart LR
  
P1[FROM AISLE STATION - 9007, 9008, 9009, 9010, 9011, 9012, 9013, 9014]-->P2[RetrievalSender]-->P3[ID32]-->P4[ID33]-->P5[ID64]-->P6[ID68]-->P7[ID26]-->P8[To STATION - 1301, 1302]
:::

# Inventory Check - Set(F2)
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp;
`jp.co.daifuku.wms.web.display.retrieval.inventorycheck.InventoryCheckSCH` &nbsp;</span>
![image.png](/.attachments/image-30ccad7a-c7a3-44cb-aeba-8d19d917c4ea.png)
  
The Inventory Check Setting screen uses for manually set the inventory check work (**always Partial Pick, and retrieval qty = 0**).

## Validations
This section explains the validations for the whole proccess Unplanned Retrieval process (PKG)
- AGC is online. (**DMGroupController.STATUS_FLAG.ONLINE**)
- Selected Station Number is NOT under suspend. (**DMStation.SUSPEND.OFF**)
- Selected Station Number is available. (**DMStation.STATUS.NORMAL** and **DMMachine.STATUS_FLAG.ACTIVE**)
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

::: mermaid
flowchart LR
    input[
        Stock Qty
        Material Code
        Material Name
        Batch #
        Area
        Loc #
        Stock Status
        Storage Date/Time
        Qty kg/crtn
        Qty crtn/PL
        UOM
        Tempering Flag
        Tempering Period
        QC Duration
        QC Check Flag
    ]

    tableList-insert[("
        DNINVENTORYCHECK
        DNCARRYINFO
        DNWORKINFO
    ")]

     tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]
tableList-select[("
        DMWAREHOUSE
        DMSHELF
        DNPALLET
        DMITEM
        DMSTATION
    ")]

    className[InventoryCheckSCH]

    input --> className --> |INSERT| tableList-insert
    tableList-select --> |SELECT| className

    classDef leftAlign text-align:left;
    class input leftAlign;
:::


# User Story
  - #5795

# Related DFD
  - {}
