[[_TOC_]]
[[_TOSP_]]

# <span style="color:skyblue; font-weight:bold">TC-SEP-C001 — Storage Empty Pallet — UI Verification</span>

| **Field**           | **Value**                                                                                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Case ID**    | TC-SEP-C001-storage-empty-pallet-UI-Verification                                                                                                                                   |
| **Test Type**       | Combined Test                                                                                                                                                                      |
| **Feature**         | Empty Pallet Storage Setting                                                                                                                                                       |
| **DFD Reference**   | [[SCREEN] Empty Pallet Storage](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/884/-SCREEN-Empty-Pallet-Storage)                  |
| **Screen Class**    | `jp.co.daifuku.wms.web.display.storage.emptypalletsetting.EmptyPalletSettingSCH`                                                                                                   |
| **Job Type**        | Planned Storage — `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.STORAGE` (value: `02`)                                                                                                        |
| **Item Code**       | Fixed: `DMITEM.EMP_PB` (Empty Pallet — Material Type 99)                                                                                                                           |
| **Related Stories** | [US-5464](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/5464) · [Bug-6751](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_workitems/edit/6751) |
| **Tester**          | _____________                                                                                                                                                                      |
| **Test Date**       | _____________                                                                                                                                                                      |
| **Overall Status**  | [ ] Pass &nbsp;&nbsp;[ ] Fail                                                                                                                                                      |

---

## <span style="color:skyblue; font-weight:bold">Objective</span>

Verify that the **Empty Pallet Setting** screen (`EmptyPalletSettingSCH`) correctly enforces all input validation rules, creates a valid `DNSTORAGEPLAN` record upon pressing **Set (F2)**.

Key assertions per design spec:
- `ITEM_CODE` is fixed to `DMITEM.EMP_PB` and cannot be changed by the user.
- `PLAN_QTY` is always hardcoded to `1`.
- `PLAN_AREA_NO` is always fixed to `Constant.AMBIENT_AREA` (= `9200`).
- `JOB_TYPE` = `DNSTORAGEPLAN.JOB_TYPE.STORAGE` (planned storage, **not** `22` unplanned).
- Duplicate pallet detection covers both `DNSTOCK` (via `BarcodeCheckOperator`) and active `DNSTORAGEPLAN` status `0`/`1` (via `checkBCR()`).

---

## <span style="color:skyblue; font-weight:bold">Pre-Conditions</span>

| No  | Pre-Condition                                                                                                  | How to Verify                                                                                                 |
| --- | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| 1   | System (MC/WMS) is **ONLINE** and all background services are active                                           | Check system status screen                                                                                    |
| 2   | `DMITEM` contains a valid `EMP_PB` item code                                                                   | `SELECT item_code, item_type FROM dmitem WHERE item_code = (SELECT emp_pb FROM wms_constants)`                |
| 3   | No existing `DNSTORAGEPLAN` with `BCR_DATA = 'EMP0000001'` in status `0` (**UNSTART**) or `1` (**NOWWORKING**) | `SELECT COUNT(*) FROM dnstorageplan WHERE bcr_data = 'EMP0000001' AND status_flag IN ('0','1')` → Expected: 0 |
| 4   | No existing `DNSTOCK` / `BarcodeCheckOperator` record with `BCR_DATA = 'EMP0000001'`                           | `SELECT COUNT(*) FROM dnstock WHERE bcr_data = 'EMP0000001'` → Expected: 0                                    |

---

## <span style="color:skyblue; font-weight:bold">Test Data</span>

| **Field**                | **Value**              | **Notes**                                             |
| ------------------------ | ---------------------- | ----------------------------------------------------- |
| **Pallet No**            | `EMP0000001`           | Must be ≥ 10 characters (PALLET_NO_LENGTH constraint) |
| **Item Code**            | `EMP_PB`               | Fixed; read-only on screen; from `DMITEM.EMP_PB`      |
| **Item Name**            | `Empty Pallet`         | Display only; auto-populated                          |
| **Storage Qty**          | `1`                    | Fixed; hardcoded; cannot be changed                   |
| **Soft Zone**            | `004`                  | Empty Pallet zone; assigned from DMITEM               |
| **Plan Area No**         | `9200`                 | `Constant.AMBIENT_AREA` — hardcoded in SCH            |
| **Job Type**             | `STORAGE` (02)         | `DNSTORAGEPLAN.JOB_TYPE.STORAGE` — not unplanned (22) |
| **Target Warehouse**     | `9200`                 | FGW1 Ambient                                          |

---

## <span style="color:skyblue; font-weight:bold">Test Steps</span>

---

### Case 1: Screen Load & UI Verification

**Purpose:** Confirm the screen renders correctly with the proper default values and read-only constraints per the design spec.

| **#** | **Action**                                                             | **Expected Result**                                                                                                                                                                                                                                               | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 1.1   | Navigate to the Empty Pallet Setting screen (`EmptyPalletSettingSCH`). | Screen loads successfully. Title shows **Empty Pallet Setting**.                                                                                                                                                                                                  |                   | [ ] P <br>[ ] F |
| 1.2   | Observe all screen fields on load.                                     | **Pallet No** field is empty with a mandatory red asterisk `(*)`. **Material Code** is pre-filled with `EMP_PB` and is **read-only** (not editable). **Material Name** shows `Empty Pallet` and is **read-only**. **Storage Qty** shows `1` and is **read-only**. |                   | [ ] P <br>[ ] F |
| 1.3   | Confirm button layout.                                                 | **Set (F2)** button and **Clear (F9)** button are visible and enabled.                                                                                                                                                                                            |                   | [ ] P <br>[ ] F |
| 1.4   | Press **Clear (F9)** after typing any input in Pallet No.              | The **Pallet No** field is cleared. All other read-only fields remain unchanged (`EMP_PB`, `Empty Pallet`, `1`).                                                                                                                                                  |                   | [ ] P <br>[ ] F |

---

### Case 2: Input Validation: Negative Scenarios (🔴 Negative)

**Purpose:** Confirm all validation guards in `EmptyPalletSettingSCH.addnewStart()` reject invalid input before any DB write occurs. No `DNSTORAGEPLAN` records should be created in any sub-case below.

#### Case 2.1: Pallet No is empty (Mandatory Field Check) (🔴 Negative)

| **#** | **Action**                                        | **Expected Result**                                                                                                                                                       | **Actual Result** | **Status**      |
| ----- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.1.1 | Leave **Pallet No** blank and click **Set (F2)**. | System rejects the input. Pallet No field is highlighted/focused. No record inserted into `DNSTORAGEPLAN`.                                                                |                   | [ ] P <br>[ ] F |
| 2.1.2 | Verify DB state.                                  | `SELECT COUNT(*) FROM dnstorageplan WHERE plan_day = TO_CHAR(SYSDATE,'YYYYMMDD') AND item_code = (SELECT emp_pb FROM wms_constants)` → Expected: 0 (no new rows created). |                   | [ ] P <br>[ ] F |

#### Case 2.2: Pallet No is less than 10 characters (Length Validation) (🔴 Negative)

| **#** | **Action**                                                             | **Expected Result**                                                                                         | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 2.2.1 | Enter `EMP001` (7 characters) in **Pallet No** and click **Set (F2)**. | System rejects with error message `6467999`: *"The minimum length of Pallet No is 10 digit"*. No DB insert. |                   | [ ] P <br>[ ] F |
| 2.2.2 | Enter exactly `EMP0000001` (10 characters).                            | The 10-character input passes the length check (boundary test). Proceed to other validations.               |                   | [ ] P <br>[ ] F |

#### Case 2.3: Pallet No already exists in DNSTOCK (BarcodeCheckOperator duplicate check) (🔴 Negative)

| **#** | **Action**                                                                                | **Expected Result**                                                                                                                        | **Actual Result** | **Status**      |
| ----- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 2.3.1 | Pre-condition: Ensure `DNSTOCK` contains an active record with `bcr_data = 'EMP0000001'`. | Setup confirmed via: `SELECT COUNT(*) FROM dnstock WHERE bcr_data = 'EMP0000001'` → Expected: ≥ 1.                                         |                   | [ ] P <br>[ ] F |
| 2.3.2 | Enter `EMP0000001` in **Pallet No** and click **Set (F2)**.                               | `BarcodeCheckOperator.exists()` returns true. System rejects with error `6463198`. Focus set to `txt_PalletNo`. No `DNSTORAGEPLAN` insert. |                   | [ ] P <br>[ ] F |

#### Case 2.4 Pallet No already exists in DNSTORAGEPLAN with active status (checkBCR duplicate check) (🔴 Negative)

| **#** | **Action**                                                                                                       | **Expected Result**                                                                                                                                          | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------- |
| 2.4.1 | Pre-condition: Insert a `DNSTORAGEPLAN` record with `BCR_DATA = 'EMP0000001'` and `STATUS_FLAG = '0'` (UNSTART). | Setup confirmed via: `SELECT status_flag FROM dnstorageplan WHERE bcr_data = 'EMP0000001'` → Expected: `0`.                                                  |                   | [ ] P <br>[ ] F |
| 2.4.2 | Enter `EMP0000001` in **Pallet No** and click **Set (F2)**.                                                      | `checkBCR()` detects existing plan in status `0`. System rejects with error `6463198`. Focus set to `txt_PalletNo`. No duplicate insert.                     |                   | [ ] P <br>[ ] F |
| 2.4.3 | Update the pre-existing record to `STATUS_FLAG = '1'` (NOWWORKING). Re-attempt **Set (F2)** with `EMP0000001`.   | `checkBCR()` also covers status `1`. System still rejects. No insert.                                                                                        |                   | [ ] P <br>[ ] F |
| 2.4.4 | Update the pre-existing record to `STATUS_FLAG = '4'` (COMPLETE). Re-attempt **Set (F2)** with `EMP0000001`.     | `checkBCR()` does **not** include status `4`. If `BarcodeCheckOperator` also clears, the submission is accepted and a new `DNSTORAGEPLAN` record is created. |                   | [ ] P <br>[ ] F |

---

### Case 3 : Successful Set (F2) - DNSTORAGEPLAN INSERT Verification (🟢 Positive)

**Purpose:** Confirm that a valid submission creates the `DNSTORAGEPLAN` record with exactly the field values defined in the design spec, including the correct `JOB_TYPE`, `PLAN_AREA_NO`, and `STORING_PAIR_KEY`.

| **#** | **Action**                                                                                     | **Expected Result**                                                                                                    | **Actual Result** | **Status**      |
| ----- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------- | --------------- |
| 3.1   | Ensure all pre-conditions are met. Enter `EMP0000001` in **Pallet No** and click **Set (F2)**. | Success message `6461009` displayed. Screen input field clears after successful registration.                          |                   | [ ] P <br>[ ] F |
| 3.2   | Query `DNSTORAGEPLAN` immediately after Set (F2).                                              | One new record exists. Run SQL in **Section: Verification SQL → SQL-01**. Verify all fields match the checklist below. |                   | [ ] P <br>[ ] F |

**DNSTORAGEPLAN Field Verification Checklist (Post Set F2):**

| **Column**          | **Expected Value**                         | **Source**                                                             | **Pass / Fail** |
| ------------------- | ------------------------------------------ | ---------------------------------------------------------------------- | --------------- |
| `PLAN_UKEY`         | Sequence-generated key (format `02` + seq) | `WMSSequenceHandler.nextStoragePlanukeySeq()`                          | [ ] P [ ] F     |
| `LOAD_UNIT_KEY`     | `SYSTIMESTAMP` at time of insert           | `DbDateUtil.getSystemDateTime()`                                       | [ ] P [ ] F     |
| `STATUS_FLAG`       | `0` (UNSTART)                              | `DBValueDefine.DNStoragePlan.STATUS_FLAG.UNSTART`                      | [ ] P [ ] F     |
| `CANCEL_FLAG`       | `0` (NORMAL)                               | `DBValueDefine.DNStoragePlan.CANCEL_FLAG.NORMAL`                       | [ ] P [ ] F     |
| `JOB_TYPE`          | `02` — Planned Storage                     | `DBValueDefine.DNStoragePlan.JOB_TYPE.STORAGE` ⚠️ NOT `22` (unplanned) | [ ] P [ ] F     |
| `PLAN_DAY`          | Today's date in `YYYYMMDD`                 | `WmsFormatUtil.toStringDate(new Date())`                               | [ ] P [ ] F     |
| `BCR_DATA`          | `EMP0000001`                               | Value from screen Pallet No field                                      | [ ] P [ ] F     |
| `ITEM_CODE`         | `EMP_PB` value from `DMITEM`               | Fixed: `dsCond.getItemCode()`                                          | [ ] P [ ] F     |
| `PLAN_QTY`          | `1`                                        | Fixed hardcoded value                                                  | [ ] P [ ] F     |
| `PLAN_AREA_NO`      | `9200`                                     | `Constant.AMBIENT_AREA` — hardcoded in SCH                             | [ ] P [ ] F     |
| `STORING_PAIR_KEY`  | `ITEM_CODE + PLAN_LOT_NO` (concatenation)  | `dsCond.getItemCode() + storagePlan.getPlanLotNo()`                    | [ ] P [ ] F     |
| `REGIST_DATE`       | Current timestamp                          | `new Date()` at insert time                                            | [ ] P [ ] F     |
| `LAST_UPDATE_DATE`  | Current timestamp                          | `new Date()` at insert time                                            | [ ] P [ ] F     |
| `REGIST_PNAME`      | `EmptyPalletSettingSCH`                    | `this.getClass().getSimpleName()`                                      | [ ] P [ ] F     |
| `LAST_UPDATE_PNAME` | `EmptyPalletSettingSCH`                    | `this.getClass().getSimpleName()`                                      | [ ] P [ ] F     |

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
    PLAN_AREA_NO,
    STORING_PAIR_KEY,
    REGIST_DATE,
    REGIST_PNAME,
    LAST_UPDATE_DATE,
    LAST_UPDATE_PNAME
FROM dnstorageplan
WHERE bcr_data = 'EMP0000001';
-- Expected:
-- STATUS_FLAG       = '0'                  (UNSTART)
-- CANCEL_FLAG       = '0'
-- JOB_TYPE          = '02'                 (STORAGE — NOT '22' unplanned)
-- PLAN_DAY          = YYYYMMDD of today
-- ITEM_CODE         = 'EMP_PB'
-- PLAN_QTY          = 1
-- PLAN_AREA_NO      = '9200'               (Constant.AMBIENT_AREA)
-- STORING_PAIR_KEY  = 'EMP_PB' || plan_lot_no  (concatenation)
-- REGIST_PNAME      = 'EmptyPalletSettingSCH'
-- LAST_UPDATE_PNAME = 'EmptyPalletSettingSCH'
```

---

## <span style="color:skyblue; font-weight:bold">Notes / Defects</span>

> ⚠️ **JOB_TYPE Clarification:** The `EmptyPalletSettingSCH` sets `JOB_TYPE = DNSTORAGEPLAN.JOB_TYPE.STORAGE` (value `02` — Planned Storage). This is **not** `22` (NOPLAN_STORAGE / Unplanned). The test case header in earlier drafts was incorrectly labelled "No Plan Storage" — this test covers **planned** storage with `job_type = 02`.

> ⚠️ **PLAN_AREA_NO is hardcoded:** The screen sets `PLAN_AREA_NO = Constant.AMBIENT_AREA` (= `9200`) regardless of user selection. There is no To Location field in the Empty Pallet screen (unlike Unplanned Storage). Empty pallets always go to FGW1 Ambient (Zone 004 aisles 9007–9010).

```
[                                                                               ]
[                                                                               ]
[                                                                               ]
```
