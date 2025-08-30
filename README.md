# Odoo Inventory ETL Pipeline

This repository contains automated ETL pipelines that extract inventory data from Odoo and load it into Google BigQuery for analytics and reporting.

## Overview

The repository includes two complementary ETL scripts:

### 1. Historical Inventory Tracking (`inventory_etl.py`)
- Runs daily at 00:15 UTC (3:15 AM Riyadh time)
- Maintains historical snapshots with location-level detail
- Creates detailed inventory history with date-based tracking

### 2. Current Stock Snapshot (`stock_etl.py`)
- Runs daily at 18:00 UTC (9:00 PM Riyadh time) - End of business day
- Provides current stock levels by product
- Simplified view focused on product-level inventory

Both pipelines perform the following operations:
1. **Extract**: Connects to Odoo API to fetch current inventory levels
2. **Transform**: Processes the data and calculates available quantities  
3. **Load**: Uploads the data to BigQuery with appropriate schema

## Features

- **Automated daily runs** via GitHub Actions with timezone awareness
- **Manual trigger** capability for on-demand execution
- **Two complementary views**: Historical tracking and current snapshots
- **Secure credential management** using GitHub Secrets
- **Comprehensive logging** for monitoring and debugging

## Data Schemas

### Historical Inventory Table (`inventory_levels_history`)
Detailed location-level inventory tracking with historical snapshots.

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

### Current Stock Table (`stock_data`)
Simplified product-level current stock snapshot updated at end of business day.

| Column | Type | Description |
|--------|------|-------------|
| Product Name | STRING | Product display name |
| Barcode | STRING | Product barcode |
| Category | STRING | Product category |
| Qty On Hand | FLOAT | Total quantity on hand |
| Reserved Qty | FLOAT | Reserved quantity |
| Available Qty | FLOAT | Available quantity (on_hand - reserved) |
| Unit Cost | FLOAT | Standard cost per unit |
| Total Cost | FLOAT | Total inventory value (qty × unit cost) |

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
  - `inventory_levels_history` (detailed historical table with locations)
  - `inventory_levels_staging` (temporary staging table)
  - `stock_data` (current stock snapshot table)

### 4. Customize Configuration

Update the following variables in both `inventory_etl.py` and `stock_etl.py` if needed:

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
- **Historical Inventory**: Runs automatically every day at 00:15 UTC (3:15 AM Riyadh time)
- **Stock Snapshot**: Runs automatically every day at 18:00 UTC (9:00 PM Riyadh time)

### Manual Execution
1. Go to the "Actions" tab in your GitHub repository
2. Select "Odoo Inventory ETL" workflow
3. Click "Run workflow"
4. Choose which script to run:
   - `both`: Run both historical and stock snapshot scripts
   - `inventory_history`: Run only the historical tracking script
   - `stock_snapshot`: Run only the current stock snapshot script

## Monitoring

Check the Actions tab for execution logs and status. Both pipelines include comprehensive logging for:
- Authentication status
- Data extraction progress
- Upload confirmation
- Error handling

## Architecture

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────────────────┐
│   Odoo API  │───▶│  GitHub      │───▶│         BigQuery            │
│             │    │  Actions     │    │                             │
│ Inventory   │    │              │    │ • inventory_levels_history  │
│ Data        │    │ • Python ETL │    │ • inventory_levels_staging  │
│             │    │ • Scheduler  │    │ • stock_data               │
└─────────────┘    └──────────────┘    └─────────────────────────────┘
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
