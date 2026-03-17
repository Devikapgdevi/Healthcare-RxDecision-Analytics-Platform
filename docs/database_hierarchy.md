# Snowflake Project — Database Hierarchy

## Overview

| Metric | Count |
|--------|-------|
| Custom Databases | 9 |
| Schemas (non-default) | 16 |
| Base Tables | 42 |
| Views | 17 |
| Total Objects | 59 |

---

## Data Flow Architecture

```
External Sources (S3, CMS)
        │
        ▼
     RAW_DB  ──────►  TRANSFORM_DB  ──────►  ANALYTICS_DB  ──────►  AI_READY_DB
    (Bronze)            (Silver)                (Gold)               (ML/AI)
        │
        └───────────►  AUDIT_DB / MONITORING_DB / SECURITY_DB / GOVERNANCE_DB / DEVOPS_DB
                              (Operational & Governance Layer)
```

---

## 1. RAW_DB (Raw Data Landing — Bronze Layer)

**Owner:** SYSADMIN | **Purpose:** Ingests raw data from source systems

```
RAW_DB
├── RAW_SCHEMA
│   ├── PATIENT_RAW .......................... 2,000 rows | PK: PATIENT_ID
│   │     Master patient demographics (name, DOB, gender, address, insurance)
│   │
│   ├── APPOINTMENTS ......................... 25,000 rows | PK: APPOINTMENT_ID → FK: PATIENT_ID
│   │     Patient appointment schedules and visit records
│   │
│   ├── LAB_RESULTS .......................... 40,000 rows | PK: LAB_ID → FK: PATIENT_ID
│   │     Laboratory test results and values
│   │
│   ├── MEDICATIONS .......................... 30,000 rows | PK: MEDICATION_ID → FK: PATIENT_ID
│   │     Medication orders and prescriptions
│   │
│   ├── MEDICATION_RECORDS ................... 3,000 rows | PK: RECORD_ID → FK: PATIENT_ID
│   │     Medication administration records
│   │
│   ├── ICU_EVENTS ........................... 4,000 rows | PK: EVENT_ID → FK: PATIENT_ID
│   │     ICU monitoring events (vitals, alerts, interventions)
│   │
│   ├── BILLING_DATA ......................... 3,000 rows | PK: BILL_ID → FK: PATIENT_ID
│   │     Patient billing and charges
│   │
│   ├── DEVICE_ALERTS ........................ 1,500 rows | PK: ALERT_ID → FK: PATIENT_ID, DEVICE_ID
│   │     Medical device alerts and notifications
│   │
│   ├── HOSPITALS ............................ 100 rows | PK: FACILITY_NPI → FK: DEFINITIVE_ID
│   │     Hospital master list (name, type, beds, revenue)
│   │
│   ├── HCO_LOCATIONS ........................ 100 rows | PK: DEFINITIVE_ID, LOCATION_ID
│   │     Healthcare organization site locations
│   │
│   ├── ICD10_REFERENCE ...................... 30 rows | PK: ICD_CODE
│   │     ICD-10 diagnosis code lookup table
│   │
│   ├── PRESCRIPTION_DATA .................... 100 rows | PK: CLAIM_YEAR
│   │     Prescription claims data
│   │
│   └── PROVIDER_AFFILIATIONS ................ 100 rows | PK: HCP_NPI
│         Provider-to-hospital affiliation mappings
│
└── CMS_INGEST
    ├── CMS_RAW_LANDING ...................... 0 rows | VARIANT column
    │     Raw JSON landing from S3 stage (s3://cms-hhs-1115-demo-downloads/)
    │
    ├── CMS_HOSPITAL_GENERAL_INFO ............ 0 rows | PK: FACILITY_ID
    │     Parsed CMS hospital general information
    │
    └── PIPELINE_AUDIT_LOG ................... 0 rows | PK: RUN_ID
          Pipeline execution tracking
```

**Stream:** `CMS_LANDING_STREAM` tracks new rows in `CMS_RAW_LANDING` (APPEND_ONLY)

---

## 2. TRANSFORM_DB (Cleaned Data — Silver Layer)

**Owner:** SYSADMIN | **Purpose:** Cleaned and standardized data from RAW_DB

```
TRANSFORM_DB
└── TRANSFORM_SCHEMA
    ├── CLEAN_PATIENT ........................ 2,000 rows | PK: PATIENT_ID
    │     Source: RAW_DB.RAW_SCHEMA.PATIENT_RAW
    │     Cleaned patient records with masked ID (PATIENT_ID_MASKED)
    │
    └── CLEAN_ICU_EVENTS ..................... 4,000 rows | PK: EVENT_ID → FK: PATIENT_ID
          Source: RAW_DB.RAW_SCHEMA.ICU_EVENTS
          Cleaned and validated ICU event records
```

---

## 3. ANALYTICS_DB (Business Analytics — Gold Layer)

**Owner:** SYSADMIN | **Purpose:** Aggregated, business-ready datasets for BI/reporting

```
ANALYTICS_DB
└── ANALYTICS_SCHEMA
    ├── PATIENT_ANALYTICS .................... 2,000 rows | PK: PATIENT_ID
    │     Source: TRANSFORM_DB.TRANSFORM_SCHEMA.CLEAN_PATIENT
    │     Enriched patient analytics with derived metrics
    │
    └── BILLING_ANALYTICS .................... 1,556 rows | PK: PATIENT_ID
          Source: RAW_DB.RAW_SCHEMA.BILLING_DATA
          Aggregated billing summaries and financial analytics
```

---

## 4. AI_READY_DB (ML/AI Features — ML Layer)

**Owner:** SYSADMIN | **Purpose:** Feature-engineered data for machine learning

```
AI_READY_DB
├── FEATURE_STORE
│   ├── ICU_FEATURE_STORE .................... 1,747 rows | PK: PATIENT_ID
│   │     ML features derived from ICU events
│   │
│   ├── PATIENT_EMBEDDINGS ................... 2,000 rows | PK: PATIENT_ID
│   │     Vector embeddings for patient similarity/search
│   │
│   └── PATIENT_NOTES ........................ 2,000 rows | PK: NOTE_ID → FK: PATIENT_ID
│         Clinical notes for NLP processing
│
├── AI_SCHEMA
│   └── ICU_FEATURE_STORE .................... 10,000 rows | PK: PATIENT_ID
│         Extended ICU feature set (larger training dataset)
│
└── SEMANTIC_MODELS
    └── V_PATIENT_SEMANTIC ................... VIEW
          Semantic view over patient data for Cortex Analyst
```

---

## 5. MONITORING_DB (Operational Monitoring)

**Owner:** ACCOUNTADMIN | **Purpose:** Snowflake platform health and performance tracking

```
MONITORING_DB
└── MONITORING_SCHEMA
    ├── WAREHOUSE_BENCHMARK .................. 183 rows | TABLE
    │     Warehouse performance baseline metrics
    │
    ├── ALERT_LOG ............................ 1 row | TABLE
    │     Triggered alert history
    │
    ├── QUERY_HISTORY_VIEW ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── LONG_RUNNING_QUERIES_VIEW ............ VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── BLOCKED_QUERIES_VIEW ................. VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── CREDIT_USAGE_VIEW .................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── DAILY_CREDIT_USAGE ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── CREDIT_SAVINGS_TRACKER ............... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── STORAGE_USAGE_VIEW ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── LOGIN_HISTORY_VIEW ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── FAILED_LOGINS_VIEW ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── WAREHOUSE_LOAD_HISTORY_VIEW .......... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    └── WAREHOUSE_PERFORMANCE_SUMMARY ........ VIEW → SNOWFLAKE.ACCOUNT_USAGE
```

---

## 6. AUDIT_DB (Audit & Compliance)

**Owner:** ACCOUNTADMIN | **Purpose:** Audit trails, compliance, and data access tracking

```
AUDIT_DB
└── AUDIT_SCHEMA
    ├── ACCESS_HISTORY_VIEW .................. VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── DATA_CHANGES_AUDIT ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── GRANT_HISTORY_AUDIT .................. VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── LOGIN_HISTORY_AUDIT .................. VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── ROLE_CHANGES_AUDIT ................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── ROLE_GRANTS_AUDIT .................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── SENSITIVE_DATA_ACCESS ................ VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── USER_AUDIT ........................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── USER_LOGIN_AUDIT ..................... VIEW → SNOWFLAKE.ACCOUNT_USAGE
    ├── COMPLIANCE_SUMMARY ................... VIEW (aggregated compliance)
    ├── V_PATIENT_SEMANTIC ................... VIEW (copy)
    │
    │   (Snapshot copies of key tables for audit retention)
    ├── CLEAN_ICU_EVENTS ..................... 4,000 rows
    ├── CLEAN_PATIENT ........................ 2,000 rows
    ├── BILLING_ANALYTICS .................... 1,556 rows
    ├── PATIENT_ANALYTICS .................... 2,000 rows
    ├── ICU_FEATURE_STORE .................... 1,747 rows
    ├── PATIENT_EMBEDDINGS ................... 2,000 rows
    └── PATIENT_NOTES ........................ 2,000 rows
```

---

## 7. SECURITY_DB (Security Policies)

**Owner:** ACCOUNTADMIN | **Purpose:** Security configurations and access policies

```
SECURITY_DB
├── POLICIES
│   └── (masking policies, network rules)
│
└── SECURITY_SCHEMA
    └── (security configurations)
```

---

## 8. GOVERNANCE_DB (Data Governance)

**Owner:** ACCOUNTADMIN | **Purpose:** Data classification, tagging, and governance rules

```
GOVERNANCE_DB
├── POLICIES
│   └── (data governance policies)
│
└── TAGS
    └── (data classification tags)
```

---

## 9. DEVOPS_DB (CI/CD & Deployment)

**Owner:** ACCOUNTADMIN | **Purpose:** Deployment tracking and script management

```
DEVOPS_DB
└── CI_CD
    ├── DEPLOYMENT_LOG ....................... 1 row
    │     Tracks deployment executions
    │
    ├── SCRIPT_REGISTRY ...................... 15 rows
    │     Registry of all SQL deployment scripts
    │
    └── TEST_TABLE ........................... 0 rows
          Testing sandbox
```

---

## Table Relationship Map

**PATIENT_ID** is the universal join key across the clinical data pipeline:

```
                    ┌─────────────────────────────────────────┐
                    │         RAW_DB.RAW_SCHEMA               │
                    │                                         │
  HOSPITALS ◄──DEFINITIVE_ID──► HCO_LOCATIONS                │
       │                                                      │
   FACILITY_NPI                                               │
       │                            ┌── APPOINTMENTS          │
       ▼                            ├── LAB_RESULTS           │
  PROVIDER_AFFILIATIONS             ├── MEDICATIONS           │
   (HCP_NPI)                        ├── MEDICATION_RECORDS    │
                    PATIENT_RAW ────┼── ICU_EVENTS            │
                    (PATIENT_ID)    ├── BILLING_DATA           │
                         │          ├── DEVICE_ALERTS          │
                         │          └── PRESCRIPTION_DATA      │
                         │                                     │
                    └────┼─────────────────────────────────────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       CLEAN_PATIENT         CLEAN_ICU_EVENTS
      (TRANSFORM_DB)         (TRANSFORM_DB)
              │                     │
     ┌────────┴────────┐            │
     ▼                 ▼            ▼
PATIENT_ANALYTICS  BILLING_    ICU_FEATURE_STORE
(ANALYTICS_DB)     ANALYTICS   PATIENT_EMBEDDINGS
                   (ANALYTICS) PATIENT_NOTES
                               (AI_READY_DB)
```

**Key Relationships (all logical joins — no formal FKs between databases):**

| From Table | Join Key | To Table |
|------------|----------|----------|
| PATIENT_RAW | PATIENT_ID | APPOINTMENTS, LAB_RESULTS, MEDICATIONS, ICU_EVENTS, BILLING_DATA, DEVICE_ALERTS, MEDICATION_RECORDS |
| HOSPITALS | DEFINITIVE_ID | HCO_LOCATIONS |
| HOSPITALS | FACILITY_NPI | PROVIDER_AFFILIATIONS (HCP_NPI) |
| PATIENT_RAW | PATIENT_ID | CLEAN_PATIENT (downstream) |
| ICU_EVENTS | PATIENT_ID | CLEAN_ICU_EVENTS (downstream) |
| CLEAN_PATIENT | PATIENT_ID | PATIENT_ANALYTICS, PATIENT_EMBEDDINGS |
| BILLING_DATA | PATIENT_ID | BILLING_ANALYTICS |
| CLEAN_ICU_EVENTS | PATIENT_ID | ICU_FEATURE_STORE |




MARKETPLACE DATA                    SYNTHETIC GENERATORS
(Definitive HC)                     (RANDOM + GENERATOR)
      │                                    │
      ▼                                    ▼
┌─────────────────── BRONZE (RAW_DB) ───────────────────┐
│ HOSPITALS, HCO_LOCATIONS,    PATIENT_RAW, ICU_EVENTS, │
│ PROVIDER_AFFILIATIONS,       BILLING_DATA,            │
│ PRESCRIPTION_DATA,           DEVICE_ALERTS,           │
│ ICD10_REFERENCE              MEDICATION_RECORDS       │
└──────────┬──────────────────────────┬─────────────────┘
           │                          │
           │   +PII masking           │   +IS_CRITICAL flag
           │   +ETL_TIMESTAMP         │   +ETL_TIMESTAMP
           ▼                          ▼
┌─────── SILVER (TRANSFORM_DB) ────────────────────────┐
│        CLEAN_PATIENT              CLEAN_ICU_EVENTS    │
└──────────┬──────────────────────────┬────────────────┘
           │                          │
           │   +ICU aggregations      │
           │   +Billing pivot         │
           ▼                          │
┌─────── GOLD (ANALYTICS_DB) ─────────────────────────┐
│   PATIENT_ANALYTICS         BILLING_ANALYTICS        │
└──────────┬──────────────────────────┬───────────────┘
           │                          │
           │   +Statistical features  │   +Risk scoring
           │   +Embeddings            │   +NLP notes
           ▼                          ▼
┌─────── PLATINUM (AI_READY_DB) ──────────────────────┐
│  ICU_FEATURE_STORE  PATIENT_NOTES  PATIENT_EMBEDDINGS│
│              V_PATIENT_SEMANTIC (view)                │
└──────────────────────────────────────────────────────┘