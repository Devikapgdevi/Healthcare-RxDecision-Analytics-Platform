# Healthcare RxDecision Analytics Platform

Enterprise healthcare analytics platform on Snowflake using Marketplace data (Definitive Healthcare), with Medallion Architecture, RBAC, governance, AI risk scoring, and a 7-tab Streamlit dashboard — optimized for minimal credit usage.

## Project Overview

| Component | Details |
|-----------|---------|
| **Project** | Healthcare RxDecision Analytics Platform |
| **Account** | tyb42779 |
| **Author** | DEVIKAPG |
| **Data Source** | Snowflake Marketplace - Definitive Healthcare RxDecision Insights |
| **Architecture** | Medallion (Bronze > Silver > Gold > Platinum) |
| **Total Records** | ~40,000 (cost-optimized, under 5K per table) |

## Marketplace Dataset

**RxDecision Insights Prescription Therapy Decisions (Sample)** by Definitive Healthcare

| Marketplace Table | Used As |
|---|---|
| `REF_HCP_BY_NPI` | Patient names, demographics, specialties |
| `REF_HCO_BY_NPI` | Hospital reference data |
| `REF_HCO_FINANCIAL_AND_CLINICAL_METRICS` | Hospital financials (beds, revenue) |
| `REF_HCO_LOCATIONS_BY_DEFINITIVE_ID` | Facility locations by region |
| `AFFILIATIONS_HCP_HCO_BY_NPI_ACTIVE` | Provider-hospital affiliations |
| `RX_PRODUCT_DECISIONS_YEARLY` | Prescription therapy decisions |
| `NRX_TRX_YEARLY` | New/total Rx counts |

## Architecture

```
+-----------------------------------------------------------------------+
|              HEALTHCARE RXDECISION ANALYTICS PLATFORM                  |
+-----------------------------------------------------------------------+
|   SNOWFLAKE MARKETPLACE (Definitive Healthcare)                       |
|   RXDECISION_INSIGHTS_PRESCRIPTION_THERAPY_DECISIONS_SAMPLE           |
+-----------------------------------------------------------------------+
          |
          v
+-------------+   +---------------+   +---------------+   +-------------+
|   BRONZE    |-->|    SILVER     |-->|     GOLD      |-->|  PLATINUM   |
|   RAW_DB    |   | TRANSFORM_DB  |   | ANALYTICS_DB  |   | AI_READY_DB |
| 9 tables    |   | 2 tables      |   | 2 tables      |   | 3 tables    |
+-------------+   +---------------+   +---------------+   +-------------+
```

## Databases

| Database | Purpose |
|----------|---------|
| RAW_DB | Bronze - Raw + Marketplace data |
| TRANSFORM_DB | Silver - Cleaned & transformed |
| ANALYTICS_DB | Gold - Analytics ready |
| AI_READY_DB | Platinum - ML features |
| MONITORING_DB | Observability |
| DEVOPS_DB | CI/CD |
| AUDIT_DB | Compliance |
| SECURITY_DB | Policies |
| GOVERNANCE_DB | Tags & masking |
| INTEGRATIONS_DB | GitHub integration |

## Data Tables (16 total)

| Layer | Table | Records | Source |
|-------|-------|---------|--------|
| Bronze | PATIENT_RAW | 2,000 | Marketplace HCP seeded |
| Bronze | ICU_EVENTS | 4,000 | Generated |
| Bronze | BILLING_DATA | 3,000 | Generated |
| Bronze | HOSPITALS | 100 | Marketplace HCO |
| Bronze | HCO_LOCATIONS | 100 | Marketplace |
| Bronze | PROVIDER_AFFILIATIONS | 100 | Marketplace |
| Bronze | PRESCRIPTION_DATA | 100 | Marketplace Rx |
| Bronze | DEVICE_ALERTS | 1,500 | Generated |
| Bronze | MEDICATION_RECORDS | 3,000 | Generated |
| Silver | CLEAN_PATIENT | 2,000 | Transformed |
| Silver | CLEAN_ICU_EVENTS | 4,000 | Transformed |
| Gold | PATIENT_ANALYTICS | 2,000 | Aggregated |
| Gold | BILLING_ANALYTICS | ~1,900 | Aggregated |
| Platinum | ICU_FEATURE_STORE | ~1,800 | ML Features |
| Platinum | PATIENT_NOTES | 2,000 | NLP Ready |
| Platinum | PATIENT_EMBEDDINGS | 2,000 | Vector Store |

## SQL Scripts (Execute in Order)

| # | Script | Purpose |
|---|--------|---------|
| 1 | `01_account_administration.sql` | Network, password, session policies |
| 2 | `02_rbac_setup.sql` | 6-role hierarchy |
| 3 | `03_warehouse_management.sql` | 4 XSMALL warehouses |
| 4 | `04_database_structure.sql` | 10 databases |
| 5 | `05_resource_monitors.sql` | 5 cost monitors |
| 6 | `06_monitoring_views.sql` | 8 monitoring views |
| 7 | `07_alerts.sql` | 3 automated alerts |
| 8 | `08_data_governance.sql` | Tags, masking policies |
| 9 | `09_audit_layer.sql` | 6 audit views |
| 10 | `10_verification.sql` | Deployment validation |
| 11 | `11_medallion_architecture.sql` | Marketplace-backed data pipeline |
| 12 | `12_healthcare_industry.sql` | Device alerts, medications |
| 13 | `13_ai_ready_layer.sql` | ML feature store |

## Quick Start

```sql
-- Step 1: Install marketplace dataset
-- Snowflake Marketplace > "RxDecision Insights" by Definitive Healthcare

-- Step 2: Run scripts in order (01 through 13)
```

## Streamlit Dashboard

7-tab interactive dashboard:
- **Overview** - KPIs, admission trends, diagnosis breakdown
- **Patient Explorer** - Filterable patient search
- **ICU Vitals** - Real-time vitals monitoring
- **Billing** - Revenue analytics, payment status
- **Rx Decisions** - Marketplace prescription data
- **Hospitals & Providers** - Marketplace HCO/HCP data
- **AI Risk Scoring** - ML feature store risk analysis

## CI/CD

- **GitHub Actions**: `.github/workflows/snowflake-deploy.yml`
- **Azure DevOps**: `azure-devops/azure-pipelines.yml`

## Required GitHub Secrets

- `SNOWFLAKE_ACCOUNT`
- `SNOWFLAKE_USER`
- `SNOWFLAKE_PASSWORD`
