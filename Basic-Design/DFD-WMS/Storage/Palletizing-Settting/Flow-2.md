[[_TOC_]]
[[_TOSP_]]

#Flow 2 (Refer to AGC Linkage Specs)
[AGCⅦA Linkage Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx - Storage Section - 05 Sheets](https://daifuku.sharepoint.com/:x:/r/sites/jp0211039/Shared%20Documents/PTDI/Projects/PT.%20Guthrie%20Indonesia%20Sei%20Mangkei%20Refinery(Sime%20Darby)/Garuda%20Project/F.%20Basic%20Design/A.%20AGC%20Linkage%20Specs/AGC%E2%85%A6A%20Linkage%20Specs_1.2.7_AF954201_SimeDarby_v1.1.xlsx?d=w31ccf4d7958e4ea989446a6ac5a1a566&csf=1&web=1&e=J2FP9Y)
##ID64
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

##ID26
##ID25
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.U.->id25-update
id25process-.D.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
#### DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##ID64
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

##ID64
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

##ID26
##ID25
<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.wcs.mc.as21.communication.control.Id25Process &nbsp;</span>

::: mermaid
flowchart LR

id25("
ID 25
")

id25-update[("
DNCARRYINFO
")]
id25-delete[("
DNARRIVAL
")]

id25-->id25process
id25process-.U.->id25-update
id25process-.D.->id25-delete
:::

ID25 sent from AGC to WareNavi indicate AGC responded the job by WareNavi.
###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
#### DNCARRYINFO
- CMD_STATUS: 3:Commanded
- ERROR_CODE: 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

##ID64
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

##ID33
jp.co.daifuku.wcs.mc.as21.communication.control.Id33Process

::: mermaid
flowchart LR

id33("
ID 33
")

id33-insert[("
DNHOSTSEND
DNSTOCKHISTORY
DNINOUTRESULT
")]
id33-update[("
DNPALLET
DMSHELF
DNSTOCK
DNWORKINFO
DNSTORAGEPLAN
")]
id33-delete[("
DNCARRYINFO
")]

id33-->id33process
id33process--I-->id33-insert
id33process-.U.->id33-update
id33process-.D.->id33-delete
:::

ID33 for Storage operation which is sent by AGC to WareNavi to indicate Storage operation of the pallet is completed by SRM.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####DMSHELF
- STATUS_FLAG = 1:Occupied
- LAST_UPDATE_DATE = SYSTIMESTAMP

####DNPALLET
- CURRENT_STATION_NO = Location Number information from ID33
- STATUS_FLAG = 2:Occupied
- ALLOCATION_FLAG = 0:Not allocated
- LAST_STORED_DATE = SYSTIMESTAMP
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

####DNWORKINFO
- RESULT_QTY = DNWORKINFO.PLAN_QTY
- RESULT_AREA_NO = DNWORKINFO.PLAN_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- RESULT_LOT_NO = DNWORKINFO.PLAN_LOT_NO
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- STATUS_FLAG = 4:Completed
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

####DNSTOCK
- STORAGE_DAY = DMWARENAVISYSTEM.WORK_DAY
- NEWEST_STORAGE_DATE = SYSTIMESTAMP
- STOCK_QTY = DNSTOCK.PLAN_QTY
- ALLOCATION_QTY = DNWORKINFO.RESULT_QTY
- PLAN_QTY = 0
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

####DNSTORAGEPLAN (If DNSTORAGEPLAN.RESULT_QTY = DNSTORAGEPLAN.PLAN_QTY then update Status to completed)
- STATUS_FLAG = 4:Completed
- RESULT_QTY = DNSTORAGEPLAN.RESULT_QTY + DNWORKINFO.RESULT_QTY
- SHORTAGE_QTY = DNSTORAGEPLAN.SHORTAGE_QTY + DNWORKINFO.SHORTAGE_QTY
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

####DNSTOCKHISTORY
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- INC_DEC_TYPE = 1:Stock Increase
- JOB_TYPE = 22:Unplanned Storage
- UPDATE_STOCK_QTY = DNSTOCK.STOCK_QTY
- INC_DEC_QTY = DNSTOCK.STOCK_QTY
- STOCK_ID = DNSTOCK.STOCK_ID
- AREA_NO = DNSTOCK.AREA_NO
- LOCATION_NO = DNSTOCK.LOCATION_NO
- LOT_NO = DNSTOCK.LOT_NO
- STORAGE_DAY = DNSTOCK.STORAGE_DAY
- STORAGE_DATE = DNSTOCK.STORAGE_DATE
- NEWEST_STORAGE_DATE = DNSTOCK.NEWEST_STORAGE_DATE
- ITEM_CODE = DNSTOCK.ITEM_CODE
- SECOND_ITEM_CODE = DMITEM.SECOND_ITEM_CODE
- PALLET_ID = DNSTOCK.PALLET_ID
- BCR_DATA = DNPALLET.BCR_DATA
- AREA_TYPE = DMAREA_AREA_TYPE
- ITEM_NAME = DMITEM.ITEM_NAME
- ENTERING_QTY = DMITEM.ENTERING_QTY
- BUNDLE_ENTERING_QTY = DMITEM.BUNDLE_ENTERING_QTY
- USER_ID = Login info
- USER_NAME = Login info
- TERMINAL_NO = Login info
- TERMINAL_NAME = Login info
- IP_ADDRESS = Login info
- MANUFACTURING_DATE = DNSTOCK.MANUFACTURING_DATE
- EXPIRY_DATE = DNSTOCK.EXPIRY_DATE
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name

####DNINOUTRESULT
- RESULT_KIND = 1:Storage(Stock+)
- STATION_NO = DNCARRYINFO.SOURCE_STATION_NO
- LOCATION_NO = DNCARRYINFO.DEST_STATION_NO
- WH_STATION_NO = DNPALLET.WH_STATION_NO
- AISLE_STATION_NO = DNCARRYINFO.AISLE_STATION_NO
- WORK_TYPE = DNCARRYINFO.WORK_TYPE
- RETRIEVAL_DETAIL = DNCARRYINFO.RETRIEVAL_DETAIL
- WORK_NO = DNCARRYINFO.WORK_NO
- PALLET_ID = DNPALLET.PALLET_ID
- CARRY_KEY = DNCARRYINFO.CARRY_KEY
- RESTORING_FLAG = DNCARRYINFO.RESTORING_FLAG
- WORK_DAY = DMWARENAVISYSTEM.WORK_DAY
- REMOVE_FLAG = 00:Normal
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name

####DNHOSTSEND
- WORK_DAY = DNWORKINFO.WORK_DAY
- JOB_NO = DNWORKINFO.JOB_NO
- COLLECT_JOB_NO = DNWORKINFO.COLLECT_JOB_NO
- SETTING_UNIT_KEY = DNWORKINFO.SETTING_UNIT_KEY
- JOB_TYPE = DNWORKINFO.JOB_TYPE
- STATUS_FLAG = DNWORKINFO.STATUS_FLAG
- HARDWARE_TYPE = DNWORKINFO.HARDWARE_TYPE
- PLAN_UKEY = DNWORKINFO.PLAN_UKEY
- STOCK_ID = DNWORKINFO.STOCK_ID
- SYSTEM_CONN_KEY = DNWORKINFO.SYSTEM_CONN_KEY
- PLAN_DAY = DNWORKINFO.PLAN_DAY
- SUPPLIER_CODE = DNWORKINFO.SUPPLIER_CODE
- SUPPLIER_NAME = DMSUPPLIER.SUPPLIER_NAME
- RECEIVE_TICKET_NO = DNWORKINFO.RECEIVE_TICKET_NO
- RECEIVE_LINE_NO = DNWORKINFO.RECEIVE_LINE_NO
- RECEIVE_BRANCH_NO = DNWORKINFO.RECEIVE_BRANCH_NO
- CUSTOMER_CODE = DNWORKINFO.CUSTOMER_CODE
- CUSTOMER_NAME = DMCUSTOMER.CUSTOMER_NAME
- SHIP_TICKET_NO = DNWORKINFO.SHIP_TICKET_NO
- SHIP_LINE_NO = DNWORKINFO.SHIP_LINE_NO
- SHIP_BRANCH_NO = DNWORKINFO.SHIP_BRANCH_NO
- BATCH_NO = DNWORKINFO.BATCH_NO
- ORDER_NO = DNWORKINFO.ORDER_NO
- PLAN_AREA_NO = DNWORKINFO.PLAN_AREA_NO
- PLAN_LOCATION_NO = DNWORKINFO.PLAN_LOCATION_NO
- NOTE = DNWORKINFO.NOTE
- WORK_NOTE = DNWORKINFO.WORK_NOTE
- ITEM_CODE = DNWORKINFO.ITEM_CODE
- ITEM_NAME = DMITEM.ITEM_NAME
- UOM = DMITEM.UOM2
- JAN = DMITEM.JAN
- CASE_ITF = DMITEM.CASE_ITF
- BUNDLE_ITF = DMITEM.BUNDLE_ITF
- ENTERING_QTY = DMITEM.ENTERING_QTY
- BUNDLE_ENTERING_QTY = DMITEM.BUNDLE_ENTERING_QTY
- PLAN_LOT_NO = DNWORKINFO.PLAN_LOT_NO
- PLAN_QTY = DNWORKINFO.PLAN_QTY
- RESULT_QTY = DNWORKINFO.RESULT_QTY
- SHORTAGE_QTY = DNWORKINFO.SHORTAGE_QTY
- RESULT_AREA_NO = DNWORKINFO.RESULT_AREA_NO
- RESULT_LOCATION_NO = DNWORKINFO.RESULT_LOCATION_NO
- RESULT_LOT_NO = DNWORKINFO.RESULT_LOT_NO
- USER_ID = DNWORKINFO.USER_ID
- TERMINAL_NO = DNWORKINFO.TERMINAL_NO
- WORK_SECOND = DNWORKINFO.WORK_SECOND
- USER_NAME = DCUSER.USERNAME
- REPORT_FLAG = 9:Report not required
- REGIST_DATE = SYSTIMESTAMP
- REGIST_PNAME = Class name
- LAST_UPDATE_DATE = SYSTIMESTAMP
- LAST_UPDATE_PNAME = Class name