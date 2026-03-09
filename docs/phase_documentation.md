# Healthcare RxDecision Analytics Platform - Phase Documentation

## Project Overview

| Attribute | Value |
|-----------|-------|
| **Project Name** | Healthcare RxDecision Analytics Platform |
| **Account** | tyb42779 |
| **Author** | DEVIKAPG |
| **Date** | March 2026 |
| **Data Source** | Snowflake Marketplace - Definitive Healthcare |
| **Total Records** | ~40,000 (cost-optimized, under 5K per table) |

## Phase Documentation

### Phase 1: Account Administration
- Network Policy (HC_NETWORK_POLICY)
- Password Policy (HC_PASSWORD_POLICY) - HIPAA compliant
- Session Policy (HC_SESSION_POLICY)

### Phase 2: RBAC Setup
6 Role Hierarchy:
```
ACCOUNTADMIN
    └── HC_ACCOUNT_ADMIN
            ├── HC_SECURITY_ADMIN
            ├── HC_DATA_SCIENTIST
            └── HC_DATA_ENGINEER
                    └── HC_ANALYST
                            └── HC_VIEWER
```

### Phase 3: Warehouse Management
| Warehouse | Size | Auto-Suspend | Purpose |
|-----------|------|--------------|---------|
| HC_ETL_WH | XSmall | 60s | ETL Processing |
| HC_TRANSFORM_WH | XSmall | 60s | Data Transformation |
| HC_ANALYTICS_WH | XSmall | 60s | Analytics Queries |
| HC_AI_WH | XSmall | 60s | AI/ML Workloads |

### Phase 4: Database Structure (10 databases)
RAW_DB, TRANSFORM_DB, ANALYTICS_DB, AI_READY_DB, MONITORING_DB, DEVOPS_DB, AUDIT_DB, SECURITY_DB, GOVERNANCE_DB, INTEGRATIONS_DB

### Phase 5-7: Resource Monitors, Monitoring Views, Alerts
5 monitors, 8 views, 3 alerts

### Phase 8: Data Governance
3 Tags, 3 Masking Policies, 1 Row Access Policy

### Phase 9: Audit Layer
6 audit/compliance views

### Phase 10: Verification
Deployment validation scripts

### Phase 11-13: Medallion Architecture + Healthcare + AI
16 tables across 4 layers, all under 5K records per table

## Total Objects Created

| Category | Count |
|----------|-------|
| Databases | 10 |
| Roles | 6 |
| Warehouses | 4 |
| Resource Monitors | 5 |
| Monitoring Views | 8 |
| Audit Views | 6 |
| Masking Policies | 3 |
| Tags | 3 |
| Data Tables | 16 |
| Streamlit Dashboard | 1 (7 tabs) |
