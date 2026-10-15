# gtfs-mobility-databricks

End-to-end ETL project on Databricks using open GTFS data from Italian public transport networks. Built to develop hands-on experience with Spark SQL, PySpark, Delta Lake, and the Databricks platform.

---

## Stack

- **Platform:** Databricks (Runtime 13.3 LTS+)
- **Languages:** PySpark (ingestion), Spark SQL (transformations)
- **Storage:** Delta Lake
- **Metastore:** Hive (v1 → v5), Unity Catalog (v6)
- **Data format:** GTFS (General Transit Feed Specification)

---

## Dataset

Open GTFS feeds from Italian regional capital cities.

| Version | Cities |
|---------|--------|
| v1 | Roma (ATAC), Milano (ATM), Torino (GTT) |
| v2 | v1 + Napoli (ANM), Bologna (TPER), Firenze (Autolinee Toscane) |
| v3 | v2 + Palermo, Genova, Venezia |
| v4 | v3 + Bari, Cagliari, Trieste |
| v5 | v4 + Trento, Ancona, Catanzaro |
| v6 | v5 + Aosta, L'Aquila, Campobasso, Potenza |

GTFS files used: `stops`, `routes`, `trips`, `stop_times`, `calendar`, `calendar_dates`, `shapes`

---

## Architecture

```
Volumes (runtime_config.json + raw .txt files)
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
├── 00_config.py             # global config, generates runtime_config.json
├── 00_download.py           # automated GTFS feed download via URL
├── 00_setup.py              # database init, raw file verification
├── 01_ingest_bronze.py      # raw GTFS → Delta bronze
├── 02_transform_silver.sql  # cleaning, casting, normalization
├── 03_gold_kpi.sql          # mart: network KPIs
├── 03_gold_hourly.sql       # mart: hourly coverage
├── 03_gold_od.sql           # mart: OD flows
├── 03_gold_compare.sql      # mart: city comparison
└── 04_verify_e2e.sql        # end-to-end quality checks
```

All notebooks are **idempotent** — safe to re-run without side effects.

### Config system

`00_config.py` generates a `runtime_config.json` file on Volumes, used as the single source of truth across all notebooks. It tracks metadata (created_at, updated_at, updated_by) and is updated dynamically by `00_setup.py` after raw file verification.

---

## Versions

### v1 — Foundation
3 cities. Manual GTFS upload to Volumes. Linear pipeline, full overwrite on each run. Focus: SQL and Spark fundamentals, bronze/silver/gold pattern.

### v2 — Scale (current)
6 cities. Automated GTFS download via URL. Runtime config system via `runtime_config.json`. Dynamic detection of cities without `calendar.txt`.

### v3 → v5 — Expand
Progressive addition of 3 cities per version up to 15 Italian regional capitals.

### v6 — Complete + Production (main)
All 20 Italian regional capitals. Databricks Workflows for orchestration, Unity Catalog for governance, Great Expectations for data quality, structured logging and alerting.

---

## Run Order

```
Step 1 → 00_config.py        generate runtime_config.json
Step 2 → 00_download.py      download GTFS feeds from URLs
Step 3 → 00_setup.py         create databases, verify raw files, update config
Step 4 → 01_ingest_bronze.py raw GTFS → Delta bronze
Step 5 → 02_transform_silver.sql
Step 6 → 03_gold_kpi.sql
Step 7 → 03_gold_hourly.sql
Step 8 → 03_gold_od.sql
Step 9 → 03_gold_compare.sql
Step 10 → 04_verify_e2e.sql
```
