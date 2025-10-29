[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Palletizing Setting Set(F2)**</p>

Palletizing Start is used to set the information of stock which will be palletized by robot automatically. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as Production Storage Result.

![image.png](/.attachments/image-0617abe5-5a77-4ffa-a06f-55d39356473b.png)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

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

##<span style="color:skyblue; font-weight:bold">Table Process</span>
###<span style="color:skyblue; font-weight:bold">DNStoragePlan</span>

| **Column Name**            | **Description / Notes**                               |
|----------------------------|-------------------------------------------------------|
| **PLAN_UKEY**              |                                                       
| **LOAD_UNIT_KEY**          |                                                       
| **FILE_LINE_NO**           |                                                       
| **STATUS_FLAG**            |                                                       
| **CANCEL_FLAG**            |                                                       
| **PLAN_DAY**               |                                                       
| **VENDOR_CODE**            |                                                       
| **COMPANY_CODE**           |                                                       
| **RECEIVE_TICKET_NO**      |                                                       
| **RECEIVE_LINE_NO**        |                                                       
| **RECEIVE_TICKET_DATE**    |                                                       
| **BRANCH_NO**              |                                                       
| **PLAN_AREA_NO**           |                                                       
| **PLAN_LOCATION_NO**       |                                                       
| **MATERIAL_CODE**          |                                                       
| **PLAN_LOT_NO**            |                                                       
| **NOTE**                   |                                                       
| **PLAN_QTY**               |                                                       
| **PROCESS_QTY**            |                                                       
| **RESULT_QTY**             |                                                       
| **SHORTAGE_QTY**           |                                                       
| **REPORT_FLAG**            |                                                       
| **WORK_DAY**               |                                                       
| **REGIST_KIND**            |                                                       
| **BCR_DATA**               |                                                       
| **STORING_PAIR_KEY**       |   
| **MSG_ID**                 |
| **MSG_TYPE**               |
| **MSG_ID_ORI**             |
| **ERROR_INDICATION**       |
| **TYPE**                   |
| **MESSAGE_DESC**           |
| **REGIST_DATE**            |                                                    
| **REGIST_PNAME**           |
| **LAST_UPDATE_DATE**       |
| **LAST_UPDATE_PNAME**       |