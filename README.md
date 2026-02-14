# Industrial EAV Database Architecture for Rotogravure Print

## Project Objective
Industrial print production is a dynamic process where new sensors, chemical components, and measurement parameters are frequently introduced. Traditional database schemas handle this variability poorly, requiring constant structural modifications like adding new columns.

This project aims to build a flexible and normalized EAV database architecture designed to achieve:
* **Dynamic Scalability:** Allowing new production metrics to be seamlessly integrated as simple row inserts rather than disruptive schema changes (`ALTER TABLE`).
* **Data Integrity:** Maintaining strict relational rules and eliminating the sparse, null-heavy columns that are common in traditional wide tables.

---

## Architectural Approach: The EAV Model
To achieve maximum flexibility and mitigate sparse data issues (null-heavy columns), the database departs from the conventional column-per-metric design. Instead, the dataset is decomposed into highly cohesive, interrelated sub-tables:



* **Entity Table (`runs`):**
  * Acts as the central anchor of the system, recording each unique production cycle with a distinct `run_id` and execution `timestamp`.
* **Attribute / Dictionary Tables (`numericcols`, `stringcols`):**
  * Serve as centralized lookup directories that dynamically define the production parameters (e.g., *press_speed, viscosity, paper_type, blade_mfg*).
* **Value Tables (`runid_numericvalues`, `runid_stringvalues`):**
  * Store the actual measurement values, mapped to their respective entities and attributes via strict Foreign Keys (`ON DELETE CASCADE`), ensuring referential integrity and optimized querying.

---

## ETL Pipeline & Data Preprocessing
During the Extract, Transform, Load (ETL) phase, extensive data wrangling was performed using **Python** and **Pandas** to eliminate noise and ensure high data quality prior to database insertion:

1. **Zero-Variance Feature Elimination:** Variables containing identical values across the entire dataset (`ink_color` and `cylinder_division`) provided no analytical value and were completely dropped to reduce dimensionality.
2. **Anomaly & Duplicate Handling:** Duplicate production logs that would cause data redundancy and compromise database integrity were identified and purged.
3. **Data Standardization & Type Casting:**
   * Concatenated raw date strings (e.g., `19910108`) were parsed and strictly cast to the SQL `DATE` format (`1991-01-08`).
   * Categorical text values were normalized to lowercase to prevent case-sensitivity inconsistencies during aggregations.
   * Boolean-like string states (`YES`/`NO`) were standardized into programmatic `True`/`False` logic.

---

## Technologies & Tools
* **Database Management System (RDBMS):** MySQL
* **Data Modeling:** EAV (Entity-Attribute-Value) Design Pattern, Relational Data Modeling, Foreign Key Constraints
* **Data Cleaning & Pipeline:** Python, Pandas
