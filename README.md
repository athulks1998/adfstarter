# adfstarter

Azure Data Factory pipeline definitions for a nightly sales ETL pipeline using the Medallion architecture.

## Tech Stack
- Azure Data Factory
- Azure Blob Storage (bronze layer)
- Azure Data Lake Storage Gen2 (silver/gold layers)
- Azure SQL Database (gold layer — reporting)
- Azure Key Vault (secrets management)

## Use Case
Ingests daily sales CSV files, cleanses and type-casts them to Parquet (silver), then aggregates by region and product into a SQL summary table (gold) for reporting.

## Structure
```
adfstarter/
├── factory/            # ARM template for ADF factory deployment
├── linkedService/      # Linked service definitions (Blob, ADLS, SQL, Key Vault)
├── dataset/            # Source and sink dataset definitions
├── pipeline/           # Pipeline JSON (ingestion, transformation, master)
├── trigger/            # Schedule trigger (nightly at 01:00 UTC)
└── docs/               # Architecture documentation
```

## Full documentation
See [docs/architecture.md](docs/architecture.md).
