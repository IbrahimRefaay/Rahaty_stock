# Odoo Inventory ETL Pipeline

This repository contains an automated ETL pipeline that extracts inventory data from Odoo and loads it into Google BigQuery for analytics and reporting.

## Overview

The pipeline runs daily at 00:15 UTC (3:15 AM Riyadh time) and performs the following operations:

1. **Extract**: Connects to Odoo API to fetch current inventory levels
2. **Transform**: Processes the data and calculates available quantities
3. **Load**: Uploads the data to BigQuery while maintaining historical records

## Features

- **Automated daily runs** via GitHub Actions
- **Manual trigger** capability for on-demand execution
- **Historical data preservation** with daily snapshots
- **Secure credential management** using GitHub Secrets
- **Comprehensive logging** for monitoring and debugging

## Data Schema

The pipeline creates a table with the following structure:

| Column | Type | Description |
|--------|------|-------------|
| snapshot_date | DATE | Date of the inventory snapshot |
| product_id | STRING | Odoo product ID |
| product_name | STRING | Product display name |
| product_barcode | STRING | Product barcode |
| location_id | STRING | Odoo location ID |
| location_name | STRING | Complete location name |
| on_hand_quantity | FLOAT | Total quantity on hand |
| reserved_quantity | FLOAT | Reserved quantity |
| available_quantity | FLOAT | Available quantity (on_hand - reserved) |

## Setup Instructions

### 1. Prerequisites

- Odoo instance with API access
- Google Cloud Project with BigQuery enabled
- GitHub repository with Actions enabled

### 2. Configure Secrets

Add the following secrets to your GitHub repository:

#### `ODOO_PASSWORD`
The password for your Odoo user account.

#### `GCP_SA_KEY` 
Your Google Cloud Service Account key in JSON format. The service account needs:
- BigQuery Data Editor role
- BigQuery Job User role

### 3. Configure BigQuery

Ensure your BigQuery project has:
- Dataset: `Orders`
- Tables will be created automatically:
  - `inventory_levels_history` (main historical table)
  - `inventory_levels_staging` (temporary staging table)

### 4. Customize Configuration

Update the following variables in `inventory_etl.py` if needed:

```python
# Odoo Settings
ODOO_URL = "https://your-odoo-instance.odoo.com"
ODOO_DB = "your-database-name"
ODOO_USERNAME = "your-username@domain.com"

# BigQuery Settings
PROJECT_ID = "your-gcp-project-id"
DATASET_ID = "your-dataset-name"
```

## Usage

### Automatic Execution
The pipeline runs automatically every day at 00:15 UTC.

### Manual Execution
1. Go to the "Actions" tab in your GitHub repository
2. Select "Odoo Inventory ETL" workflow
3. Click "Run workflow"

## Monitoring

Check the Actions tab for execution logs and status. The pipeline includes comprehensive logging for:
- Authentication status
- Data extraction progress
- Upload confirmation
- Error handling

## Architecture

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Odoo API  │───▶│  GitHub      │───▶│   BigQuery      │
│             │    │  Actions     │    │                 │
│ Inventory   │    │              │    │ • Staging Table │
│ Data        │    │ • Python ETL │    │ • History Table │
│             │    │ • Scheduler  │    │                 │
└─────────────┘    └──────────────┘    └─────────────────┘
```

## Troubleshooting

### Common Issues

1. **Authentication Failed**: Check ODOO_PASSWORD secret
2. **BigQuery Permission Denied**: Verify GCP_SA_KEY has proper roles
3. **Schedule Not Running**: Ensure the repository has Actions enabled

### Logs
All execution logs are available in the GitHub Actions interface with detailed information about each step.

## Security

- Credentials are stored securely as GitHub Secrets
- Service account follows principle of least privilege
- No sensitive data is logged or exposed

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License.
