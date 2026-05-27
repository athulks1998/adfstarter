# ADF Starter — Sales ETL Architecture

## Overview

This repository contains Azure Data Factory (ADF) pipeline definitions for a **nightly sales ETL** workflow. It follows the **Medallion architecture** (Bronze → Silver → Gold) using Azure Blob Storage, ADLS Gen2, and Azure SQL Database.

---

## Architecture Diagram

```
[Blob Storage: bronze/sales/]
         |
         v
[IngestRawSalesPipeline]   — validate file exists, copy CSV → Parquet
         |
         v
[ADLS Gen2: silver/sales/year=YYYY/month=MM/]
         |
         v
[TransformAndAggregateSalesPipeline]  — dataflow aggregation, upsert
         |
         v
[Azure SQL: gold.sales_summary]
```

All three steps are coordinated by `MasterOrchestrationPipeline`, which runs nightly via `NightlySalesTrigger`.

---

## Components

### Linked Services
| Name | Type | Purpose |
|------|------|---------|
| `AzureKeyVaultLS` | Azure Key Vault | Centralized secrets — no credentials in code |
| `AzureBlobStorageLS` | Azure Blob Storage | Raw CSV source (bronze) |
| `AzureDataLakeStorageLS` | ADLS Gen2 | Data lake (silver/gold Parquet) |
| `AzureSQLDatabaseLS` | Azure SQL Database | Final aggregated output (gold) |

### Datasets
| Name | Format | Layer |
|------|--------|-------|
| `RawSalesCSVDS` | CSV | Bronze |
| `SilverSalesParquetDS` | Parquet (Snappy) | Silver |
| `GoldSalesSummaryDS` | SQL Table | Gold |

### Pipelines
| Name | Purpose |
|------|---------|
| `IngestRawSalesPipeline` | Validate & copy raw CSV → silver Parquet |
| `TransformAndAggregateSalesPipeline` | Aggregate silver data → gold SQL |
| `MasterOrchestrationPipeline` | Orchestrates the full ETL end-to-end |

### Trigger
- **`NightlySalesTrigger`** — Schedule trigger, fires daily at 01:00 UTC.

---

## Security

- All credentials are stored in **Azure Key Vault**. No secrets are hardcoded.
- The ADF instance uses a **System-Assigned Managed Identity** — grant it `Key Vault Secrets User` role.
- Linked services reference Key Vault secrets by name.

---

## Getting Started

1. Deploy the factory ARM template: `factory/AdfStarterFactory.json`
2. Create the required Key Vault secrets (see below)
3. Import linked services, datasets, pipelines, and triggers via ADF Studio or CI/CD

### Required Key Vault Secrets
| Secret Name | Value |
|-------------|-------|
| `blob-storage-connection-string` | Connection string for Blob Storage |
| `adls-account-url` | ADLS Gen2 DFS endpoint URL |
| `adls-account-key` | ADLS Gen2 account key |
| `sql-db-connection-string` | Azure SQL Database connection string |

---

## Conventions

- Pipeline parameters use `camelCase`.
- All timestamps are UTC.
- Pipelines are **idempotent** — safe to re-run for the same date.
- Retry policies are set on all copy/dataflow activities.
