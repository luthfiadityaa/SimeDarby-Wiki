[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletizing Setting Set(F2)**</p>
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

Palletizing Start is used to set the information of stock which will be palletized by robot automatically. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as Production Storage Result.

![image.png](/.attachments/image-0617abe5-5a77-4ffa-a06f-55d39356473b.png)

::: mermaid
flowchart LR
    input[
        Station<br>
        Current Status<br>
        Material Code<br>
        Material Name<br>
        Material Type<br>
        Batch #<br>
        Planned Craton Qty<br>
        Qty kg/crtn<br>
        Qty crtn/PL<br>
        UOM<br>
        Tempering Period<br>
        Storage Location<br>
        Expiry Days<br>
    ]

    tableList-insert[("
        DNSTORAGEPLAN<br>
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
    ")]

    className[PalletizingSettingSCH]

    input --> className --> tableList-insert

    classDef leftAlign text-align:left;
    class input leftAlign;

:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the `DNSTORAGEPLAN` data
- Material Code exists in **DMMaterialMaster**
- Input text with red asterisk <span style="color:red">(*)</span> is not empty

##<span style="color:skyblue; font-weight:bold">Related Table</span>
###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>
| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              | Sequence object (Unique identifier for the plan)     |
| **LOAD_UNIT_KEY**          | `1 = Started` (Status: whether the plan has started)  |
| **FILE_LINE_NO**           | `0 = Normal data` (Flag for cancellation)            |
| **STATUS_FLAG**            | Value from the screen (Supplier Code)                 |
| **CANCEL_FLAG**            | Value from the screen (Supplier Name)                 |
| **PLAN_DAY**               | Value from the screen (Storer Code)                   |
| **VENDOR_CODE**            | Value from the screen (Unplanned Purchase Order)     |
| **COMPANY_CODE**           | Value from the screen (Short Item ID)                 |
| **RECEIVE_TICKET_NO**      | Value from the screen (Primary Unit of Measure Quantity) |
| **RECEIVE_LINE_NO**        | `1 = Unplanned` (Storage type code)                   |
| **RECEIVE_TICKET_DATE**    | Login info (User who registered the plan)            |
| **BRANCH_NO**              | `SYSTIMESTAMP` (Timestamp of registration)            |
| **PLAN_AREA_NO**           | Class name (Class that registered the plan)          |
| **PLAN_LOCATION_NO**       | `SYSTIMESTAMP` (Timestamp of last update)             |
| **MATERIAL_CODE**          | Class name (Class that last updated the plan)        |

