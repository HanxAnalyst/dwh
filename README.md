# Enterprise Data Warehouse (End-to-End)

## 1. Project Overview
**Goal:** Build a centralized Data Warehouse (DWH) using the **Medallion Architecture** to enable business intelligence reporting.
**Tech Stack:** Microsoft SQL Server, SSMS, Git.
**Source Systems:** ERP (CSV exports), CRM (CSV exports).

## 2. Architecture & Scope
* **Architecture:** Medallion (Bronze -> Silver -> Gold).
* **Data Flow:** CSV Flat Files -> SQL Server Staging -> SQL Server Core -> Star Schema.
* **Scope Strategy:** **Current State Only**.
    * *Decision:* No historical tracking (SCD Type 2).
    * *Load Pattern:* Truncate and Load (Full Refresh) or SCD Type 1 (Overwrite).
## 1. General Rules
Casing: Use snake_case (all lowercase with underscores). It is the most compatible across different tools (Python, SQL, BI).

Separators: Always use underscores _.
Pluralization: Use Plural for table names (they hold multiple records) and Singular for column names.
Good: crm_customers, dim_products
Bad: crm_customer, dim_product_list

2. Table Naming by Layer
Bronze Layer (Staging / Raw)
Format: {layer}_{source}_{entity}

Prefix: bronze_ or stg_
Source: crm, erp, web
Entity: customers, orders

Examples:
bronze_crm_customers
bronze_erp_invoices
Silver Layer (Cleaned / 3NF)

Format: {layer}_{entity}

Prefix: silver_ or norm_

Examples:
silver_customers
silver_order_details

Gold Layer (Dimensional / Presentation)
Format: {type}_{entity}
Dimensions: dim_{entity}
Facts: fact_{process}
Aggregates: agg_{process}_{period}

Examples:
dim_customers
fact_sales
agg_sales_monthly

3. Column Naming
Keys
Surrogate Key (PK): {table_name}_key (or sk suffix).
Example: customer_key, product_key
Business Key (Natural Key): {table_name}_bk or {table_name}_id.
Example: customer_id (The ID from the CRM), order_number.
Foreign Key: {referenced_table}_key.
Example: customer_key (inside the fact_sales table).

Technical / Audit Columns
Prefix these with dwh_ or meta_ so they group together alphabetically at the end of the column list.

dwh_load_date: Timestamp when the row was inserted.
dwh_update_date: Timestamp when the row was last changed.
dwh_batch_id: ID of the ETL execution (useful for debugging).
dwh_source_system: Where the data came from (e.g., 'SAP', 'Salesforce').

4. Stored Procedures (ETL)
Format: usp_{action}_{target_layer}_{entity}
Prefix: usp_ (User Stored Procedure - SQL Server standard).
Action: load, transform, export.
Layer: bronze, silver, gold.

Examples:
usp_load_bronze_crm_customers (Loading raw CSV to Bronze)
usp_transform_silver_customers (Cleaning Bronze to Silver)
usp_load_gold_dim_customers (Loading the Dimension)
---

## 3. Implementation Requirements

### Phase 1: Infrastructure & Bronze Layer (Raw)
**Objective:** Ingest raw CSV data into the database without transformation.
* [ ] **Setup:** Initialize `git` repo and creating database schemas (`SA_Bronze`, `BL_Silver`, `DWH_Gold`).
* [ ] **DDL:** Create "Raw" tables in `SA_Bronze` using loose data types (`VARCHAR(MAX)`) to prevent load failures.
* [ ] **ETL:** Develop Bulk Insert scripts to load ERP/CRM CSVs into `SA_Bronze`.
    * *Target Tables:* `Bronze_Customers`, `Bronze_Products`, `Bronze_Orders`.
* [ ] **Audit:** Add `LoadDate` timestamp column to all Bronze tables.

### Phase 2: Silver Layer (Cleansing & Standardization)
**Objective:** Clean raw data, handle NULLs, and enforce data types.
* [ ] **Data Quality:** Check for and handle orphaned records (e.g., Orders with no matching Customer).
* [ ] **Transformation:** Standardize text fields (e.g., `UPPER(City)`, Trim whitespace).
* [ ] **Integration:** Merge `Product` and `Category` data into a single logical product dataset.
* [ ] **DDL:** Create `BL_Silver` tables with strict data types (`INT`, `DECIMAL(10,2)`, `DATE`).

### Phase 3: Gold Layer (Dimensional Modeling)
**Objective:** Transform cleaned data into a Star Schema for reporting.
* [ ] **Modeling:** Design the Star Schema based on business processes.
    * **Fact Table:** `Fact_Sales` (Grain: One row per order line item).
    * **Dimension:** `Dim_Customer` (Attributes: Name, Address, Phone).
    * **Dimension:** `Dim_Product` (Attributes: Name, Category Name, Price).
    * **Dimension:** `Dim_Date` (Standard Date Dimension).
* [ ] **ETL Logic:** Implement **SCD Type 1** (Overwrite attributes if they change).
* [ ] **Surrogate Keys:** Generate custom `IDs` (e.g., `ProductKey`) to decouple DWH from source system IDs.

### Phase 4: Documentation & Delivery
**Objective:** Make the data usable for stakeholders.
* [ ] **Data Dictionary:** Document all Gold Layer columns, data types, and business definitions.
* [ ] **Lineage:** Diagram the flow from Source CSV -> Gold Fact Table.
* [ ] **Views:** Create SQL Views for top 3 key metrics (e.g., *Total Revenue by Category*).
