##Flow 1 (Refer to AGC Linkage Specs)
###ID64
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.U.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

###ID26
###ID25
###ID64
jp.co.daifuku.wcs.mc.as21.communication.control.Id64Process

::: mermaid
flowchart LR

id64("
ID 64
")

id64-update[("
DNCARRYINFO
")]

id64-->id64process
id64process-.U.->id64-update
:::

Upon equipment have picked up the Pallet successfully, ID64 will be sent from AGC to WareNavi to indicate pick up of Pallet is completed.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####DNCARRYINFO
- CMD_STATUS = 4:Pickup completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

###ID33