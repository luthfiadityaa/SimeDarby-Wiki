[[_TOC_]]
[[_TOSP_]]

#<span style="color:skyblue; font-weight:bold">Unplanned Storage database flow</span>
| Action Name                        | DNWORKINFO | DNWORKLIST | DNPALLET | DNCARRYINFO | DNSTOCK | DNHOSTSEND | DNARRIVAL | DMWAREHOUSE | DMSHELF |
|-|-|-|-|-|-|-|-|-|-|
| Palletize Start - Set(F2)[(1)]| INSERT | INSERT | INSERT | INSERT | INSERT | INSERT | | | |
| ID26[(2)] | | | | | | | INSERT | | |
| Automatic Mode Change Sender[(3)] | UPDATE | | UPDATE | UPDATE | UPDATE | | UPDATE | UPDATE | UPDATE | UPDATE
| ID25[(4)] | | | | UPDATE | | | DELETE | | |
| ID26[(5)] | | | | | | | INSERT | | |
| Automatic Mode Change Sender[(6)] | | | UPDATE | UPDATE | UPDATE | | UPDATE | | |
| ID25[(7)] | | | | UPDATE | | | | | |

#<span style="color:skyblue; font-weight:bold">Unplanned Storage - Set(F2)</span>
 
The Unplanned Storage Setting screen uses for manually set the storage work .
This screen mainly uses when there are some troubles with Host System Linkage(irregular cases) or when manual operation is necessary.. After **Set(F2)**, all items in input text will be added into a Pallet Number. The result will be posted back to SAP as [Unplanned Storage and Retrieval Result](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/850/Unplanned-Storage-and-Retrieval-Result)

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wms.web.display.storage.palletizingsetting.PalletizingSettingSCH &nbsp;</span>

::: mermaid
flowchart LR
    input[
        Pallet ID<br>
        Material Code<br>
        Material Name<br>
        Material Type<br>
        Qty kg/crtn<br>
        Qty crtn/PL<br>
        Batch #<br>
        Storage Qty<br>        
        UOM<br>
        Tempering Period<br>
        Expiry Days<br>
        Storage Location<br>        
    ]

    tableList-insert[("
        DNPALLET<br>
        DNSTOCK<br>
        DNWORKINFO<br>
        DNWORKLIST<br>
        DNHOSTSEND<br>
        DNCARRYINFO<br>
    ")]

    className[PalletizingSettingSCH]

    input --> className --> |INSERT| tableList-insert

    classDef leftAlign text-align:left;
    class input leftAlign;
:::

##<span style="color:skyblue; font-weight:bold">Validations</span>
This section explains the validations for the whole proccess Palletize Start
- AGC is online. <span style="color:green; font-weight:bold">(DMGroupController.STATUS_FLAG.ONLINE)</span>
- Selected Station Number is NOT under suspend. <span style="color:green; font-weight:bold">(DMStation.SUSPEND.OFF)</span>
- Selected Station Number is available. <span style="color:green; font-weight:bold">(DMStation.STATUS.NORMAL and DMMachine.STATUS_FLAG.ACTIVE)</span>
- Pallet Information does not exist in <span style="color:green; font-weight:bold">DNCARRYINFO.</span>  
  To check for Pallet Information:  
  <span style="color:green; font-weight:bold">JOIN DNCARRYINFO.PALLET_ID = DNPALLET.PALLET_ID  
  CONDITION DNPALLET.BCR_DATA = <Pallet Number> </span>  
  So if result > 0, Palletize Start cannot proceed.
- Material Code exists in <span style="color:green; font-weight:bold">DMMaterialMaster</span>
- Input text with red asterisk <span style="color:red">(*)</span> is not empty