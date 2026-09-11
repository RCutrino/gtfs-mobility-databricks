# gtfs-mobility-databricks

End-to-end ETL project on Databricks using open GTFS data from Italian public transport networks. Built to develop hands-on experience with Spark SQL, PySpark, Delta Lake, and the Databricks platform.

---

## Stack

- **Platform:** Databricks (Runtime 13.3 LTS+)
- **Languages:** PySpark (ingestion), Spark SQL (transformations)
- **Storage:** Delta Lake
- **Metastore:** Hive (v1), Unity Catalog (v3)
- **Data format:** GTFS (General Transit Feed Specification)

---

## Dataset

Open GTFS feeds from Italian public transport providers.

| Version | Cities |
|---------|--------|
| v1 | Roma (ATAC), Milano (ATM), Torino (GTT) |
| v2 | v1 + Napoli, Bologna, Firenze, Palermo, Venezia |
| v3 | v2 (production-grade) |

GTFS files used: `stops`, `routes`, `trips`, `stop_times`, `calendar`, `calendar_dates`, `shapes`

---

## Architecture

```
DBFS Raw (.txt files)
        │
        ▼
  bronze layer        ← 1:1 with raw files, added metadata (city, ingestion_ts)
        │
        ▼
  silver layer        ← cleaned, typed, normalized, partitioned by city
        │
        ▼
   gold layer         ← analytical marts, ready for reporting
```

### Gold Marts

| Mart | Description |
|------|-------------|
| `kpi_network` | Routes, stops, trips count + avg headway per city and transport type |
| `hourly_coverage` | Active trips and stops per hour of day and day type |
| `od_flows` | Origin-Destination pairs with trip count and avg travel time |
| `city_comparison` | All KPIs in long format for cross-city comparison |

---

## Notebooks

```
notebooks/
├── 00_setup.py              # database init, config, path setup
├── 01_ingest_bronze.py      # raw GTFS → Delta bronze
├── 02_transform_silver.sql  # cleaning, casting, normalization
├── 03_gold_kpi.sql          # mart: network KPIs
├── 03_gold_hourly.sql       # mart: hourly coverage
├── 03_gold_od.sql           # mart: OD flows
├── 03_gold_compare.sql      # mart: city comparison
└── 04_verify_e2e.sql        # end-to-end quality checks
```

All notebooks are **idempotent** — safe to re-run without side effects.

---

## Versions

### v1 — Foundation (current)
3 cities. Manual GTFS upload to DBFS. Linear pipeline, full overwrite on each run. Focus: SQL and Spark fundamentals, bronze/silver/gold pattern.

### v2 — Scale
8 cities. Automated ingestion via direct URL download. Incremental Delta Lake writes. Optimized partitioning and Z-ordering.

### v3 — Production (main)
Full platform setup: Databricks Workflows for orchestration, Unity Catalog for governance, Great Expectations for data quality, structured logging and alerting.

---

## Run Order

```
Task 0 → Download GTFS feeds manually, upload to DBFS
Task 1 → 00_setup.py
Task 2 → 01_ingest_bronze.py
Task 3 → 02_transform_silver.sql
Task 4 → 03_gold_kpi.sql
Task 5 → 03_gold_hourly.sql
Task 6 → 03_gold_od.sql
Task 7 → 03_gold_compare.sql
Task 8 → 04_verify_e2e.sql
```
