# Project 1 — Lakehouse Foundation

**Platform:** Local PySpark environment → Microsoft Fabric (next phase)
**Notebook:** `01_lakehouse_foundation.ipynb`
**Status:** ✅ Phase 1 complete (local environment) — Fabric implementation planned Week 5
**Domain coverage:** DP-700 Domain 1 (Lakehouse design), Domain 2 (ingestion, PySpark transformation, Spark SQL)

---

## Business scenario

A fictional UK retail analytics team needs a structured data store for regional sales data.
Raw transactional records arrive as flat data (product, region, category, quantity, price)
and need to be cleaned, transformed, and made available for SQL-based reporting.

**Why this scenario:** Directly relevant to retail operations background. Demonstrates
the full Bronze → Silver layer of a medallion architecture — raw ingestion, type casting,
column derivation, standardisation, and SQL serving — all core DP-700 Domain 2 skills.

---

## Architecture

```
Raw sales data (in-memory / CSV source)
        │
        ▼
PySpark — schema definition + data load
        │
        ▼
PySpark transformations (Cell 4)
  • Cast sale_date: string → date
  • Derive total_revenue = quantity × unit_price
  • Extract year, month columns
  • Standardise region to UPPERCASE
  • Add processed_at audit timestamp
        │
        ▼
Spark SQL temp view (equivalent to Fabric Lakehouse table)
        │
        ▼
SQL aggregation queries (revenue by region, category, product)
        │
        ▼
Output: sales_clean.csv (local)
        │
        ▼ (Phase 2 — Fabric)
Delta table on OneLake → SQL analytics endpoint → Power BI
```

---

## What was built

### Dataset
10 retail sales transactions across 3 UK regions (London, Manchester, Birmingham),
3 product categories (Electronics, Clothing, Food), and 7 distinct products.

### Transformations applied
| Column | Before | After |
|--------|--------|-------|
| `sale_date` | String `"2026-01-05"` | Date type `2026-01-05` |
| `region` | Mixed case `"London"` | Uppercase `"LONDON"` |
| `total_revenue` | Not present | Derived: `quantity × unit_price` |
| `year` | Not present | Extracted from `sale_date` |
| `month` | Not present | Extracted from `sale_date` |
| `processed_at` | Not present | Audit timestamp added |

### Results

**Revenue by region:**
| Region | Total Revenue | Units Sold | Distinct Products |
|--------|--------------|-----------|-------------------|
| LONDON | £2,319.75 | 25 | 4 |
| BIRMINGHAM | £1,549.88 | 12 | 3 |
| MANCHESTER | £549.95 | 5 | 2 |

**Revenue by category:**
| Category | Total Revenue |
|----------|--------------|
| Electronics | £3,949.94 |
| Clothing | £359.94 |
| Food | £109.70 |

**Top products:**
| Product | Revenue | Units Sold |
|---------|---------|-----------|
| Laptop | £1,799.98 | 2 |
| Phone | £1,799.97 | 3 |
| Tablet | £349.99 | 1 |
| Jacket | £199.96 | 4 |
| Shoes | £159.98 | 2 |

---

## Key design decisions

### 1. Explicit schema definition vs inferSchema
**What I chose:** Explicit `StructType` schema definition.

**Reasoning:** `inferSchema=True` reads the data twice (one pass to infer, one to load)
and can miscast types — for example reading a date as a string, or an integer as a long.
Defining the schema explicitly is faster, deterministic, and fails loudly if the source
data doesn't match expectations. In production pipelines, silent type mismatches cause
downstream failures that are hard to trace back to the ingestion layer.

### 2. String → Date casting at transformation, not ingestion
**What I chose:** Load `sale_date` as `StringType` in the raw schema, then cast to
`DateType` in the transformation step using `F.to_date()` with an explicit format string.

**Reasoning:** Separates raw ingestion (Bronze layer — preserve the source exactly)
from transformation (Silver layer — apply business rules). If the source date format
changes, the fix is in one place. This pattern maps directly to the medallion
architecture used in Fabric Lakehouses.

### 3. Spark SQL for aggregations vs DataFrame API
**What I chose:** Spark SQL (`spark.sql("""...""")`) for all aggregation queries.

**Reasoning:** The SQL syntax is identical to what runs against the Fabric Lakehouse
SQL analytics endpoint and Fabric Warehouse. Practising SQL here means the same queries
work without modification when the data moves to OneLake. The DataFrame API
(`.groupBy().agg()`) is more composable in code but less transferable to the Fabric
SQL endpoint.

### 4. Pandas for local file output vs Spark writer
**What I chose:** Convert to pandas DataFrame and use `df.to_csv()` for local output.

**Reasoning:** Spark's native file writer (`.write.parquet()` / `.write.csv()`)
requires `winutils.exe` on Windows — a Hadoop compatibility shim that doesn't exist
for Hadoop 3.5.0 (used by PySpark 4.2) at the time of writing. Using pandas for
local output is a pragmatic workaround. **This limitation does not exist in Microsoft
Fabric** — OneLake handles all storage natively and `.write.format("delta").saveAsTable()`
works without any additional configuration.

---

## Environment

| Component | Version |
|-----------|---------|
| PySpark | 4.2.0 |
| Python | 3.11.9 |
| Java | OpenJDK 25.0.4 (Temurin) |
| pyarrow | 25.0.0 |
| pandas | 3.x |
| OS | Windows 11 |
| IDE | Jupyter Lab (local) |

---

## How to reproduce locally

### Prerequisites
- Python 3.11
- Java 21+ (Temurin recommended)
- `pip install pyspark pandas pyarrow`

### Run the notebook
1. Open Jupyter Lab: `jupyter lab`
2. Open `01_lakehouse_foundation.ipynb`
3. Run Cell 1 first (sets environment variables)
4. Run cells 2–6 in order

**Note for Windows users:** Set these environment variables in Cell 1:
```python
import os
os.environ["PYSPARK_PYTHON"] = r"path\to\python3.11\python.exe"
os.environ["PYSPARK_DRIVER_PYTHON"] = r"path\to\python3.11\python.exe"
os.environ["HADOOP_HOME"] = r"C:\hadoop"
```

---

## What I learned

- ✅ Explicit schema definition prevents silent type mismatches at ingestion
- ✅ Separating Bronze (raw) and Silver (transformed) layers is a design decision, not just a naming convention — it determines where type logic lives and how failures are traced
- ✅ Spark SQL and the Fabric SQL analytics endpoint use the same syntax — practising locally transfers directly
- ✅ `winutils.exe` is a Windows-only Hadoop compatibility issue that does not exist in Fabric — OneLake handles storage natively
- ✅ PySpark 4.2 requires Python 3.11 — Python 3.14 is not yet supported

---

## Phase 2 — Fabric implementation (planned Week 5)

The same notebook will be adapted for Microsoft Fabric:

- Raw CSV uploaded to Lakehouse **Files zone** (Bronze layer)
- PySpark notebook reads from Files, applies same transformations
- Output written as managed **Delta table** to Lakehouse **Tables zone** (Silver layer)
- Data served via **SQL analytics endpoint** — no additional configuration needed
- OPTIMIZE + VACUUM maintenance notebook added for Delta table health

The transformation code (Cells 3–5) transfers without modification.
Only Cell 6 changes: `df.to_csv()` → `.write.format("delta").saveAsTable("sales_clean")`

---

## DP-700 skills demonstrated

**Domain 1 — Implement and manage an analytics solution**
- ✅ Understand Lakehouse architecture (Files zone vs Tables zone, Bronze/Silver layers)
- ✅ Understand managed Delta tables and SQL analytics endpoint
- ⏳ Create and configure a Fabric Lakehouse item (Phase 2)

**Domain 2 — Ingest and transform data**
- ✅ Define explicit schema for structured data ingestion
- ✅ Apply PySpark transformations: type casting, column derivation, standardisation
- ✅ Use Spark SQL for aggregation queries
- ✅ Understand batch ingestion pattern
- ✅ Understand Bronze → Silver medallion layer separation
- ⏳ Write managed Delta table with partitioning and V-Order (Phase 2)
- ⏳ Implement OPTIMIZE and VACUUM maintenance (Phase 2)
