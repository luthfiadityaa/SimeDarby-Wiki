[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">TC-USS-C001 — Unplanned Storage Setting — UI Verification</span>

| **Field**           | **Value**                                                                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Test Case ID**    | TC-USS-C001-unplanned-storage-UI-Verification                                                                                                                                                                                                                |
| **Test Type**       | Combined Test (UI Validation + DNSTORAGEPLAN INSERT Verification)                                                                                                                                                                                            |
| **Feature**         | Unplanned Storage Setting                                                                                                                                                                                                                                    |
| **DFD Reference**   | [[SCREEN] Unplanned Storage](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/946/Unplanned-Storage)                                                                                                          |
| **Screen Class**    | `jp.co.daifuku.wms.web.display.storage.unplannedstorage.UnplannedStorageSCH`                                                                                                                                                                                 |
| **Job Type**        | Unplanned Storage — `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.NOPLAN_STORAGE` (value: `22`)                                                                                                                                                                        |
| **Related Stories** | [US-5140](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5140) · [US-6184](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6184) · [Bug-6675](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6675) · [Bug-6673](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6673) · [Bug-6672](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6672) |
| **Tester**          | _____________                                                                                                                                                                                                                                                |
| **Test Date**       | _____________                                                                                                                                                                                                                                                |
| **Overall Status**  | [ ] Pass &nbsp;&nbsp;[ ] Fail                                                                                                                                                                                                                                |

---

## <span style="color:skyblue; font-weight:bold">Objective</span>

Verify that the **Unplanned Storage Setting** screen (`UnplannedStorageSCH`) correctly:
1. Enforces all mandatory field validations (`validateBeforeStart` and `validateAfterSelectItemCode`).
2. Applies correct dynamic field behaviour (To Location, Tempering Period, Expiry Days, Stock Status) based on selected Material Code's `SOFT_ZONE_ID`.
3. Rejects invalid inputs (duplicate pallet, invalid qty, tempering/expiry rule violations, excluded material types).
4. Creates a valid `DNSTORAGEPLAN` record upon pressing **Set (F2)** with `JOB_TYPE = 22 (NOPLAN_STORAGE)`.

Key assertions per design spec:
- `JOB_TYPE` = `DNSTORAGEPLAN.JOB_TYPE.NOPLAN_STORAGE` (value `22`) — **NOT** `02` (planned).
- `PLAN_AREA_NO` is derived from the **To Location** field via `mappingAreaNo()` — `FGW1` → `9002`, `FGW2` → `9001`, `PACK` → `9200`.
- `STOCK_STATUS` is always forced to `'UU'` (Unrestricted Use) and is **disabled** on screen (read-only).
- Material Types `99` (**EMP_PB**), `98` (**IRREGULAR_PB**), `97` (**DIRECT_PB**) are **excluded** from selection.
- `STORING_PAIR_KEY` = `ITEM_CODE + PLAN_LOT_NO`.
- Daily cleanup check: if daily update is running, the screen rejects all submissions.

---

## <span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

| No  | Pre-Condition                                                                                                            | How to Verify                                                                                                                          |
| --- | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | System (MC/WMS) is **ONLINE** and all background services are active                                                     | Check system status screen                                                                                                             |
| 2   | Daily database cleanup is **NOT** actively running                                                                       | `SELECT * FROM system_status WHERE daily_update_flag = '1'` → Expected: no rows (or flag = 0)                                          |
| 3   | `DMITEM` contains valid ZFNP material codes mapped to Soft Zone `005`, `001`, `002` and a ZPCK code mapped to zone `003` | `SELECT item_code, soft_zone_id FROM dmitem WHERE item_code IN ('ZFNPFG005','ZFNPFG001','ZFNPFG002','ZPCKPM003')`                      |
| 4   | No existing `DNSTORAGEPLAN` with `BCR_DATA = 'PLTTSS0001'` in status `0`, `1`, or `2`                                    | `SELECT COUNT(*) FROM dnstorageplan WHERE bcr_data = 'PLTTSS0001' AND status_flag IN ('0','1','2')` → Expected: 0                      |
| 5   | No existing `DNSTOCK` / `DNPALLET` record with `BCR_DATA = 'PLTTSS0001'`                                                 | `SELECT COUNT(*) FROM dnpallet WHERE bcr_data = 'PLTTSS0001'` → Expected: 0                                                            |
| 6   | A duplicate pallet record `PLTDUPL001` exists in `DNSTORAGEPLAN` with `STATUS_FLAG IN ('0','1','2')` for negative tests  | Insert or confirm: `SELECT COUNT(*) FROM dnstorageplan WHERE bcr_data = 'PLTDUPL001' AND status_flag IN ('0','1','2')` → Expected: ≥ 1 |

---

## <span style="color:skyblue; font-weight:bold">Test Data</span>

| **Field**                           | **Value**     | **Notes**                                                                       |
| ----------------------------------- | ------------- | ------------------------------------------------------------------------------- |
| **Pallet No (Primary)**             | `PLTTSS0001`  | Must be exactly **10** characters                                               |
| **Pallet No (Duplicate — Plan)**    | `PLTDUPL001`  | Pre-existing in `DNSTORAGEPLAN` status 0/1/2 — for negative test                |
| **Pallet No (Duplicate — Stock)**   | `PLTSTCK001`  | Pre-existing in `DNSTOCK` / `DNPALLET` — for negative test                      |
| **Material Code (ZFNP zone 005)**   | `ZFNPFG005`   | Soft Zone **005** → To Location: **FGW1**, **FGW2**                             |
| **Material Code (ZFNP zone 001)**   | `ZFNPFG001`   | Soft Zone **001** → To Location: **FGW2** only                                  |
| **Material Code (ZFNP zone 002)**   | `ZFNPFG002`   | Soft Zone **002** → To Location: **FGW1** only                                  |
| **Material Code (ZPCK zone 003)**   | `ZPCKPM003`   | Soft Zone **003** → To Location: **PACK**; Tempering + Expiry disabled          |
| **Material Code (Excluded EMP_PB)** | `EMP_PB` item | Material Type 99 — must be rejected from selection                              |
| **Batch No**                        | `BATUSS0001`  | 10 characters                                                                   |
| **Storage Qty (Valid)**             | `100`         | Greater than 0                                                                  |
| **Storage Qty (Invalid)**           | `0` or `-1`   | Negative test — must be rejected                                                |
| **Tempering Period (Valid)**        | `7` (days)    | Must be > 1 (code checks `<= 1` → error)                                        |
| **Tempering Period (Invalid)**      | `1` (or `0`)  | Must trigger error: "Tempering Period must be greater than 1"                   |
| **Expiry Days (Valid)**             | `30` (days)   | `30 * 24 = 720 hours` > `7` hours Tempering Period — passes                     |
| **Expiry Days (Invalid)**           | `0` (days)    | `0 * 24 = 0 hours` < `7` — triggers "Expiry Days cannot be less than Tempering" |
| **Storage Location From**           | `VT01`        | Temporary / staging location                                                    |
| **Storage Location To (ZFNP/Amb)**  | `FGW1`        | Maps to `PLAN_AREA_NO = 9002` via `mappingAreaNo()`                             |
| **Storage Location To (ZFNP/Temp)** | `FGW2`        | Maps to `PLAN_AREA_NO = 9001` via `mappingAreaNo()`                             |
| **Storage Location To (ZPCK)**      | `PACK`        | Maps to `PLAN_AREA_NO = 9200` (PM Ambient) via `mappingAreaNo()`                |
| **Stock Status**                    | `UU`          | Always Unrestricted — field is disabled (read-only) on screen                   |

---

## <span style="color:skyblue; font-weight:bold">Test Steps</span>

---

### Case 1: Screen Load & UI Verification

**Purpose:** Confirm the screen renders correctly with required fields, placeholders, and button layout per design spec.

| **No** | **Action**                                                                            | **Expected Result**                                                                                                                                                                                                                       | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 1.1    | Navigate to the Unplanned Storage Setting screen (`UnplannedStorageSCH`).             | Screen loads successfully. Title shows **Unplanned Storage Setting** (from screens 1301, 1302, and 1106).                                                                                                                                 |                   | [ ] P <br>[ ] F |
| 1.2    | Observe all mandatory fields on load (red asterisk <span style="color:red">*</span>). | Fields marked (<span style="color:red">*</span>): **Pallet No**, **Material Code**, **Storage Qty**, **Stock Status**. Other fields such as **Batch No**, **Tempering Period**, **Expiry Days** are conditionally required per soft zone. |                   | [ ] P <br>[ ] F |
| 1.3    | Observe **Stock Status** field on initial load.                                       | **Stock Status** is pre-populated with `UU` (Unrestricted Use) and is **disabled** (greyed out, not editable by user), per spec: `STOCK_STATUS will be disable with value is 'UU'`.                                                       |                   | [ ] P <br>[ ] F |
| 1.4    | Confirm button layout.                                                                | **Set (F2)** and **Clear (F9)** buttons are visible and enabled.                                                                                                                                                                          |                   | [ ] P <br>[ ] F |
| 1.5    | Confirm **Storage Location (From)** dropdown.                                         | Dropdown displays only `SAP_LOCATION_STORAGE` list entries (per spec: "Storage Location (Master) only display SAP_LOCATION_STORAGE List").                                                                                                |                   | [ ] P <br>[ ] F |
| 1.6    | Press **Clear (F9)** after entering data in multiple fields.                          | All editable input fields are cleared. **Stock Status** remains `UU` (disabled field does not clear). Screen returns to initial ready state.                                                                                              |                   | [ ] P <br>[ ] F |

---

### Case 2: Dynamic Field Behaviour Based on Material Code Soft Zone

**Purpose:** Confirm that selecting a Material Code dynamically controls To Location options, Tempering Period, and Expiry Days fields based on `DMITEM.SOFT_ZONE_ID`.

#### Case 2.1: ZFNP — Soft Zone 005 (🟡 Dynamic Behaviour)

| **No** | **Action**                                                          | **Expected Result**                                                                                                                                                         | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.1.1  | Select Material Code `ZFNPFG005` (Soft Zone 005) from the dropdown. | **To Location** dropdown refreshes and shows **both** `FGW1` and `FGW2` as available options. **Tempering Period** and **Expiry Days** fields are **enabled** and editable. |                   | [ ] P <br>[ ] F |
| 2.1.2  | Confirm **Stock Status** field after material selection.            | **Stock Status** remains `UU` and stays **disabled** (read-only). Value is not affected by material selection.                                                              |                   | [ ] P <br>[ ] F |

#### Case 2.2: ZFNP — Soft Zone 001 (🟡 Dynamic Behaviour)

| **No** | **Action**                                        | **Expected Result**                                                                                                 | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.2.1  | Select Material Code `ZFNPFG001` (Soft Zone 001). | **To Location** dropdown shows **FGW2 only**. `FGW1` must NOT be available. Tempering/Expiry fields remain enabled. |                   | [ ] P <br>[ ] F |

#### Case 2.3: ZFNP — Soft Zone 002 (🟡 Dynamic Behaviour)

| **No** | **Action**                                        | **Expected Result**                                                                                                 | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.3.1  | Select Material Code `ZFNPFG002` (Soft Zone 002). | **To Location** dropdown shows **FGW1 only**. `FGW2` must NOT be available. Tempering/Expiry fields remain enabled. |                   | [ ] P <br>[ ] F |

#### Case 2.4: ZPCK — Soft Zone 003 (🟡 Dynamic Behaviour)

| **No** | **Action**                                             | **Expected Result**                                                                                                                                                                                | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.4.1  | Select Material Code `ZPCKPM003` (Soft Zone 003 / PM). | **To Location** dropdown shows **PACK only**. **Tempering Period** field is **disabled** (greyed out). **Expiry Days** field is **disabled** (greyed out). **Stock Status** remains `UU` disabled. |                   | [ ] P <br>[ ] F |

---

### Case 3: Input Validation — Negative Scenarios (🔴 Negative)

**Purpose:** Confirm all `validateBeforeStart()` guards reject invalid input before any DB write. No `DNSTORAGEPLAN` record must be created in any sub-case below.

#### Case 3.1: Mandatory Field — Pallet No Empty

| **No** | **Action**                                                                  | **Expected Result**                                                                          | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.1.1  | Leave **Pallet No** field empty. Fill all other fields. Click **Set (F2)**. | UI halts. Focus moves to `txt_PalletNo`. Error displayed. No `DNSTORAGEPLAN` record created. |                   | [ ] P <br>[ ] F |

#### Case 3.2: Pallet No Length ≠ 10 Characters

| **No** | **Action**                                                          | **Expected Result**                                                                           | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.2.1  | Enter **Pallet No** = `PLT0001` (7 chars). Click **Set (F2)**.      | Validation error: Pallet No must be exactly 10 characters. No `DNSTORAGEPLAN` record created. |                   | [ ] P <br>[ ] F |
| 3.2.2  | Enter **Pallet No** = `PLTTSS00012` (11 chars). Click **Set (F2)**. | Validation error: Pallet No must be exactly 10 characters. No `DNSTORAGEPLAN` record created. |                   | [ ] P <br>[ ] F |

#### Case 3.3: Duplicate Pallet — Active DNSTORAGEPLAN

| **No** | **Action**                                                                                              | **Expected Result**                                                                                                                                               | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.3.1  | Enter **Pallet No** = `PLTDUPL001` (pre-existing in `DNSTORAGEPLAN` status 0 or 1). Click **Set (F2)**. | Error `6463226`: "Pallet ID PLTDUPL001 is already used." Focus returns to `txt_PalletNo`. `isBCRExist()` detects status `0`/`1`/`2` match. No new record created. |                   | [ ] P <br>[ ] F |
| 3.3.2  | Update the pre-existing `PLTDUPL001` record to `STATUS_FLAG = '4'` (COMPLETE). Retry **Set (F2)**.      | `isBCRExist()` does NOT check status `4`. Validation passes this guard. If all other checks pass, record is created.                                              |                   | [ ] P <br>[ ] F |

#### Case 3.4: Duplicate Pallet — Existing DNSTOCK / DNPALLET

| **No** | **Action**                                                                                     | **Expected Result**                                                                                                                      | **Actual Result** | **Status**      |
| ------ | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.4.1  | Enter **Pallet No** = `PLTSTCK001` (pre-existing in `DNPALLET`/`DNSTOCK`). Click **Set (F2)**. | Error `6463198`: "Pallet ID PLTSTCK001 found in existing pallet in stock." `isStockExist()` or `BarcodeCheckOperator.exists()` triggers. |                   | [ ] P <br>[ ] F |

#### Case 3.5: Storage Qty ≤ 0 (🔴 Negative)

| **No** | **Action**                                         | **Expected Result**                                                                                 | **Actual Result** | **Status**      |
| ------ | -------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.5.1  | Enter **Storage Qty** = `0`. Click **Set (F2)**.   | Error `6463225`: "Storage Qty must be greater than 0." Focus moves to `txtN_StorageQty`. No INSERT. |                   | [ ] P <br>[ ] F |
| 3.5.2  | Enter **Storage Qty** = `-10`. Click **Set (F2)**. | Same error as 3.5.1. Negative value rejected.                                                       |                   | [ ] P <br>[ ] F |

#### Case 3.6: Tempering Period ≤ 1 (ZFNP only) (🔴 Negative)

| **No** | **Action**                                                                  | **Expected Result**                                                                                        | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.6.1  | Select ZFNP material. Enter **Tempering Period** = `1`. Click **Set (F2)**. | Error `6463034`: "Tempering Period must greater than 1." Focus moves to `txtN_TemperingPeriod`. No INSERT. |                   | [ ] P <br>[ ] F |
| 3.6.2  | Select ZFNP material. Enter **Tempering Period** = `0`. Click **Set (F2)**. | Same error as 3.6.1.                                                                                       |                   | [ ] P <br>[ ] F |

#### Case 3.7: Expiry Days < Tempering Period (ZFNP only) (🔴 Negative)

| **No** | **Action**                                                                                                   | **Expected Result**                                                                                                                                                                          | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.7.1  | Select ZFNP material. Set **Tempering Period** = `10` hours, **Expiry Days** = `0` days. Click **Set (F2)**. | Error `6463220`: "Tempering Period (Hours) cannot greater than Expiry Days (Days)." Code checks: `TempPeriod (10 hrs) > expiryHours (0 * 24 = 0)` → fails. Focus moves to `txtN_ExpiryDays`. |                   | [ ] P <br>[ ] F |
| 3.7.2  | Set **Tempering Period** = `7` hours, **Expiry Days** = `0` days.                                            | Error `6463220` triggered. `7 > 0 * 24 = 0`.                                                                                                                                                 |                   | [ ] P <br>[ ] F |
| 3.7.3  | Set **Tempering Period** = `7` hours, **Expiry Days** = `30` days (= 720 hours).                             | Validation passes: `7 ≤ 720`. No error.                                                                                                                                                      |                   | [ ] P <br>[ ] F |

---

### Case 4: Successful Set (F2) — DNSTORAGEPLAN INSERT Verification (🟢 Positive)

**Purpose:** Confirm that a valid ZFNP submission with all correct data creates a `DNSTORAGEPLAN` record with the exact field values defined in the design spec.

**Test Input (ZFNP / Zone 005 → To Location: FGW1):**

| Field                  | Value          |
| ---------------------- | -------------- |
| Pallet No              | `PLTTSS0001`   |
| Material Code          | `ZFNPFG005`    |
| Batch No               | `BATUSS0001`   |
| Storage Qty            | `100`          |
| Tempering Period       | `7`            |
| Expiry Days            | `30`           |
| Storage Location From  | `VT01`         |
| To Location            | `FGW1`         |
| Stock Status           | `UU` (fixed)   |

| **No** | **Action**                                                                           | **Expected Result**                                                                                         | **Actual Result** | **Status**      |
| ------ | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 4.1    | Ensure all pre-conditions are met. Enter all test data above and click **Set (F2)**. | Success message `6461009` ("Setting done.") is displayed. Input fields clear after successful registration. |                   | [ ] P <br>[ ] F |
| 4.2    | Query `DNSTORAGEPLAN` immediately after Set (F2).                                    | One new record exists. Run **SQL-01** (Section: Verification SQL). Verify all fields match checklist below. |                   | [ ] P <br>[ ] F |
| 4.3    | Verify `DNSTORAGEPLAN.STOCK_STATUS` is `'UU'`.                                       | Value = `UU`. Confirmed from screen's disabled field mapped directly to the DB column.                      |                   | [ ] P <br>[ ] F |
| 4.4    | Verify `DNSTORAGEPLAN.JOB_TYPE` is `'22'` (NOPLAN_STORAGE), NOT `'02'` (planned).    | `JOB_TYPE = '22'`. Confirm via SQL-01.                                                                      |                   | [ ] P <br>[ ] F |

**DNSTORAGEPLAN Field Verification Checklist (Post Set F2):**

| **Column**                   | **Expected Value**                                        | **Source**                                                                             | **Pass / Fail** |
| ---------------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------- |
| `PLAN_UKEY`                  | Sequence-generated key                                    | `WMSSequenceHandler.nextStoragePlanukeySeq()`                                          | [ ] P [ ] F     |
| `LOAD_UNIT_KEY`              | Timestamp at time of insert                               | `DbDateUtil.getSystemDateTime()`                                                       | [ ] P [ ] F     |
| `STATUS_FLAG`                | `0` (UNSTART)                                             | `DBValueDefine.DNStoragePlan.STATUS_FLAG.UNSTART`                                      | [ ] P [ ] F     |
| `CANCEL_FLAG`                | `0` (NORMAL)                                             | `DBValueDefine.DNStoragePlan.CANCEL_FLAG.NORMAL`                                       | [ ] P [ ] F     |
| `JOB_TYPE`                   | `22` — Unplanned / No Plan Storage                        | `DBValueDefine.DNStoragePlan.JOB_TYPE.NOPLAN_STORAGE` ⚠️ NOT `02` (planned)            | [ ] P [ ] F     |
| `PLAN_DAY`                   | Today's date in `YYYYMMDD`                                | `WmsFormatUtil.toStringDate(dsUnplannedStorage.getStorageDateTime())`                  | [ ] P [ ] F     |
| `BCR_DATA`                   | `PLTTSS0001`                                              | Value from screen **Pallet No** field                                                  | [ ] P [ ] F     |
| `ITEM_CODE`                  | `ZFNPFG005`                                               | `dsItemMasterCond.getItemCode()`                                                       | [ ] P [ ] F     |
| `PLAN_QTY`                   | `100`                                                     | `dsUnplannedStorage.getPlanQty().intValue()`                                           | [ ] P [ ] F     |
| `PLAN_LOT_NO`                | `BATUSS0001`                                              | `dsUnplannedStorage.getPlanLotNo()`                                                    | [ ] P [ ] F     |
| `PLAN_AREA_NO`               | `9002` (mapped from `FGW1`)                               | `mappingAreaNo(dsUnplannedStorage.getAreaTo())` — FGW1 → `9002`                        | [ ] P [ ] F     |
| `STORAGE_LOCATION_FROM`      | `VT01`                                                    | `dsUnplannedStorage.getAreaFrom()`                                                     | [ ] P [ ] F     |
| `STORAGE_LOCATION_TO`        | `FGW1`                                                    | `dsUnplannedStorage.getAreaTo()`                                                       | [ ] P [ ] F     |
| `STORING_PAIR_KEY`           | `ZFNPFG005BATUSS0001` (ITEM_CODE + PLAN_LOT_NO)           | `dsItemMasterCond.getItemCode() + dsUnplannedStorage.getPlanLotNo()`                   | [ ] P [ ] F     |
| `BATCH_TEMPERING_PERIOD`     | `7`                                                       | `dsUnplannedStorage.getTemperingPeriod().intValue()`                                   | [ ] P [ ] F     |
| `BATCH_EXPIRY_DAYS`          | `30`                                                      | `dsUnplannedStorage.getExpiryDays().intValue()`                                        | [ ] P [ ] F     |
| `STOCK_STATUS`               | `UU`                                                      | `dsUnplannedStorage.getStockStatus()` — always `UU`, field disabled on screen          | [ ] P [ ] F     |
| `REGIST_DATE`                | Current timestamp                                         | `new Date()` at insert time                                                            | [ ] P [ ] F     |
| `LAST_UPDATE_DATE`           | Current timestamp                                         | `new Date()` at insert time                                                            | [ ] P [ ] F     |
| `REGIST_PNAME`               | `UnplannedStorageSCH`                                     | `this.getClass().getSimpleName()`                                                      | [ ] P [ ] F     |
| `LAST_UPDATE_PNAME`          | `UnplannedStorageSCH`                                     | `this.getClass().getSimpleName()`                                                      | [ ] P [ ] F     |

---

### Case 5: ZPCK Submission Verification (🟢 Positive)

**Purpose:** Confirm ZPCK material correctly omits Tempering Period and Expiry Days from DB record, and maps To Location `PACK` to `PLAN_AREA_NO`.

**Test Input (ZPCK / Zone 003 → To Location: PACK):**

| Field                  | Value          |
| ---------------------- | -------------- |
| Pallet No              | `PLTTSS0001`   |
| Material Code          | `ZPCKPM003`    |
| Batch No               | (leave empty — optional for ZPCK per Bug-6675) |
| Storage Qty            | `50`           |
| Tempering Period       | (disabled)     |
| Expiry Days            | (disabled)     |
| Storage Location From  | `VT01`         |
| To Location            | `PACK`         |
| Stock Status           | `UU` (fixed)   |

| **No** | **Action**                                                                                                                        | **Expected Result**                                                                                                         | **Actual Result** | **Status**      |
| ------ | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 5.1    | Select `ZPCKPM003`. Confirm **Tempering Period** and **Expiry Days** are disabled. Enter remaining test data. Click **Set (F2)**. | Success message `6461009`. Record created in `DNSTORAGEPLAN`.                                                               |                   | [ ] P <br>[ ] F |
| 5.2    | Query `DNSTORAGEPLAN` for `BCR_DATA = 'PLTTSS0001'`.                                                                              | `PLAN_AREA_NO` = `9200` (mapped from `PACK`). `BATCH_TEMPERING_PERIOD` = `0`. `BATCH_EXPIRY_DAYS` = `0`. `JOB_TYPE` = `22`. |                   | [ ] P <br>[ ] F |
| 5.3    | Verify `STORAGE_LOCATION_TO` = `PACK`.                                                                                            | Column value is `PACK` (direct screen value stored). `PLAN_AREA_NO` is the mapped numeric value separately.                 |                   | [ ] P <br>[ ] F |

---

## <span style="color:skyblue; font-weight:bold">Verification SQL</span>

### SQL-01 — DNSTORAGEPLAN Record (After Set F2)

```sql
SELECT
    PLAN_UKEY,
    LOAD_UNIT_KEY,
    STATUS_FLAG,
    CANCEL_FLAG,
    JOB_TYPE,
    PLAN_DAY,
    BCR_DATA,
    ITEM_CODE,
    PLAN_QTY,
    PLAN_LOT_NO,
    PLAN_AREA_NO,
    STORAGE_LOCATION_FROM,
    STORAGE_LOCATION_TO,
    STORING_PAIR_KEY,
    BATCH_TEMPERING_PERIOD,
    BATCH_EXPIRY_DAYS,
    STOCK_STATUS,
    REGIST_DATE,
    REGIST_PNAME,
    LAST_UPDATE_DATE,
    LAST_UPDATE_PNAME
FROM dnstorageplan
WHERE bcr_data = 'PLTTSS0001'
ORDER BY regist_date DESC;
-- Expected:
-- STATUS_FLAG           = '0'                          (UNSTART)
-- CANCEL_FLAG           = '0'
-- JOB_TYPE              = '22'                         (NOPLAN_STORAGE — NOT '02' planned)
-- PLAN_DAY              = YYYYMMDD of today
-- ITEM_CODE             = 'ZFNPFG005'
-- PLAN_QTY              = 100
-- PLAN_LOT_NO           = 'BATUSS0001'
-- PLAN_AREA_NO          = '9002'                       (mapped from FGW1)
-- STORAGE_LOCATION_FROM = 'VT01'
-- STORAGE_LOCATION_TO   = 'FGW1'
-- STORING_PAIR_KEY      = 'ZFNPFG005BATUSS0001'
-- BATCH_TEMPERING_PERIOD= 7
-- BATCH_EXPIRY_DAYS     = 30
-- STOCK_STATUS          = 'UU'
-- REGIST_PNAME          = 'UnplannedStorageSCH'
-- LAST_UPDATE_PNAME     = 'UnplannedStorageSCH'
```

### SQL-02 — Verify No Record Created on Negative Test

```sql
-- Run after any negative test step above to confirm no spurious INSERT occurred
SELECT COUNT(*) AS unexpected_records
FROM dnstorageplan
WHERE bcr_data = 'PLTTSS0001'
  AND status_flag IN ('0','1','2');
-- Expected: 0
```

### SQL-03 — Duplicate Detection Verification

```sql
-- Confirm pre-existing duplicate pallet record for Case 3.3
SELECT bcr_data, status_flag, job_type, plan_day
FROM dnstorageplan
WHERE bcr_data = 'PLTDUPL001'
  AND status_flag IN ('0','1','2');
-- Expected: ≥ 1 row
```

---

## <span style="color:skyblue; font-weight:bold">Notes / Defects</span>

> ⚠️ **JOB_TYPE is 22 (NOPLAN_STORAGE):** Unlike Empty Pallet (`02`), Unplanned Storage always sets `JOB_TYPE = '22'`. This propagates through the entire flow to `DNWORKINFO` and ultimately `DNHOSTSEND`. Any hardcoded `02` in the downstream flow is a defect.

> ⚠️ **PLAN_AREA_NO is derived from To Location via `mappingAreaNo()`:** FGW1 → `9002`, FGW2 → `9001`, PACK → `9200`. The screen does not let the user type an area number directly.

> ⚠️ **Tempering/Expiry validation uses hours internally:** `TempPeriod` is stored as hours; `expiryHours = expiryDays * 24`. E.g. Tempering Period = 7 (hours), Expiry Days = 0 → `7 > 0` → error. Tempering Period = 7, Expiry Days = 1 → `7 > 24` → no error.

> ⚠️ **Batch No is optional for ZPCK (Bug-6675):** For Soft Zone 003 (PM/ZPCK), `validateBeforeStart()` skips the Batch No mandatory check (the `!Constant.SOFT_ZONE.PM.equals(softZoneId)` guard). ZPCK submissions with empty Batch No are accepted.

> ⚠️ **STOCK_STATUS is always `UU`:** The field is disabled on screen and the value is passed directly from screen state. No user override is possible.

```
[                                                                               ]
[                                                                               ]
[                                                                               ]
```
