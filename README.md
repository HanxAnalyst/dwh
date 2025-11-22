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
