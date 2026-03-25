[[_TOC_]]

#<span style="color:skyblue; font-weight:bold">Summary Flow</span>
::: mermaid
flowchart LR

P1[MC Thread Polling
Every N seconds]-->P2[Find Batches with
Tempering Reached]-->P3[Bulk Update
Tempering Flag to Reached]
:::

#<span style="color:skyblue; font-weight:bold">Tempering Checker Overview</span>

The **TemperingChecker** is a background MC thread (`As21Thread`) that automatically monitors and updates the tempering status of stocks in the ASRS.

It runs on a polling interval configured by `McParam.TEMPERING_CHECKER_SEC` and checks whether the tempering period of each stock has elapsed based on its `STORAGE_DATE` and `TEMPERING_PERIOD`.

<span style="background-color:yellow; color:black; font-weight:bold">&nbsp; jp.co.daifuku.asrs.transmission.TemperingChecker &nbsp;</span>

**Abbreviation:**
- **STCK** : DNSTOCK

* **Operation Name**
  - **U** : UPDATE
  - **S** : SELECT

| Action Name                                                                     |STCK|
|---------------------------------------------------------------------------------|----|
| Find batches with tempering reached [(1)](#Step-1--Find-Batches)                | S  |
| Find stocks per batch [(2)](#Step-2--Find-Stocks-Per-Batch)                     | S  |
| Update tempering flag to Reached [(3)](#Step-3--Bulk-Update-Tempering-Flag)     | U  |

<hr>

#<span style="color:skyblue; font-weight:bold">Tempering Checker Process Flow</span>

::: mermaid
flowchart TD
    start([TemperingChecker Thread Wakes Up])-->init[Initialize StockHandler]
    init-->query1[Step 1: Query distinct LOT_NO
    from DNSTOCK grouped by batch]
    query1-->hasNext{Has next batch?}
    hasNext-->|YES| query2[Step 2: Query STOCK_IDs for batch
    where tempering period reached]
    query2-->collect[Collect all STOCK_IDs
    into list]
    collect-->bulkUpdate[Step 3: Bulk UPDATE
    TEMPERING_FLAG = 1 Reached
    WHERE STOCK_ID IN list]
    bulkUpdate-->hasNext
    hasNext-->|NO| sleep([Sleep for N seconds])
    sleep-->start

    classDef leftAlign text-align:left;
:::

<hr>

# Step 1 — Find Batches

**Purpose:** Find all distinct `LOT_NO` (batch numbers) that have at least one stock where the tempering period has been reached.

###<span style="color:skyblue; font-weight:bold">SQL Query</span>

```sql
SELECT DNSTOCK.LOT_NO
FROM DNSTOCK
WHERE SYSDATE >= (DNSTOCK.STORAGE_DATE + NUMTODSINTERVAL(DNSTOCK.TEMPERING_PERIOD, 'HOUR'))
  AND DNSTOCK.STOCK_STATUS = 'UU'       -- Unrestricted Used
  AND DNSTOCK.TEMPERING_FLAG = 0         -- Not Reached
GROUP BY DNSTOCK.LOT_NO
```

###<span style="color:skyblue; font-weight:bold">Conditions</span>
| Column | Condition | Description |
|--------|-----------|-------------|
| STORAGE_DATE + TEMPERING_PERIOD | `SYSDATE >=` | Current time has passed the tempering period (in hours) since storage |
| STOCK_STATUS | `= 'UU'` | Only **Unrestricted Used** stocks (target for normal retrieval) |
| TEMPERING_FLAG | `= 0` | Only stocks that have **Not Reached** tempering yet |

<hr>

# Step 2 — Find Stocks Per Batch

**Purpose:** For each `LOT_NO` found in Step 1, find all individual `STOCK_ID`s that meet the tempering condition. Rows are locked with `SELECT FOR UPDATE` to prevent concurrent modification.

###<span style="color:skyblue; font-weight:bold">SQL Query</span>

```sql
SELECT DNSTOCK.STOCK_ID
FROM DNSTOCK
WHERE DNSTOCK.LOT_NO = :lotNo
  AND SYSDATE >= (DNSTOCK.STORAGE_DATE + NUMTODSINTERVAL(DNSTOCK.TEMPERING_PERIOD, 'HOUR'))
  AND DNSTOCK.STOCK_STATUS = 'UU'       -- Unrestricted Used
  AND DNSTOCK.TEMPERING_FLAG = 0         -- Not Reached
ORDER BY DNSTOCK.STOCK_ID
```

> **Note:** The tempering condition is re-checked per stock because individual stocks within the same batch may have different `STORAGE_DATE` values.

<hr>

# Step 3 — Bulk Update Tempering Flag

**Purpose:** Update all collected `STOCK_ID`s in a single bulk UPDATE using `StockAlterKey.setStockId(String[])`.

###<span style="color:skyblue; font-weight:bold">Table Operation DML</span>
####<span style="color:skyblue; font-weight:bold">DNStock</span>
* **TEMPERING_FLAG** : <span style="color:yellow; font-weight:bold; background-color:grey">1: Reached</span> (tempering period has elapsed)
* **LAST_UPDATE_PNAME**: TemperingChecker

**Effective SQL:**
```sql
UPDATE DNSTOCK
SET TEMPERING_FLAG = 1,
    LAST_UPDATE_PNAME = 'TemperingChecker'
WHERE STOCK_ID IN (:stockId1, :stockId2, ... :stockIdN)
```

<hr>

#<span style="color:skyblue; font-weight:bold">Tempering Flag Lifecycle</span>

::: mermaid
stateDiagram-v2
    [*] --> NotReached : Stock stored in ASRS
    NotReached --> Reached : TemperingChecker detects SYSDATE >= STORAGE_DATE + TEMPERING_PERIOD
    Reached --> NotReached : Extend Tempering Period (QC Screen)
    Reached --> QCRetrieved : Retrieval for QC Start
:::

| Flag Value | Description | Set By |
|------------|-------------|--------|
| **0: Not Reached** | Tempering period has NOT elapsed yet | Initial storage / Extend Tempering Period screen |
| **1: Reached** | Tempering period has elapsed, stock is ready for QC | TemperingChecker (automatic) |

<hr>

#<span style="color:skyblue; font-weight:bold">Configuration</span>

| Parameter | Source | Description |
|-----------|--------|-------------|
| `TEMPERING_CHECKER_SEC` | `McParam` / `AS21Param.properties` | Polling interval in seconds between each check cycle |
| `FINDER_READ_DEFAULT_SIZE` | `AS21Param.properties` | Number of records fetched per batch from DirectDBFinder (default: 100) |

<hr>

#<span style="color:skyblue; font-weight:bold">Related DFD</span>
- [Extend Tempering Period](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/906/Extend-Tempering-Period)
- [Update QC Status](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/904/Update-QC-Status)
- [Retrieval for QC Start](https://dev.azure.com/Daifuku-SW/ID_SimeDarbyPlantation/_wiki/wikis/ID_SimeDarbyPlantation.wiki/900/Retrieval-for-QC-Start)
