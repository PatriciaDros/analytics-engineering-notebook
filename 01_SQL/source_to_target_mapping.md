# Source-to-Target Mapping (STM) Notes

## Purpose

The **Source-to-Target Mapping (STM)** document is the single source of truth for how data moves from source files into the data warehouse.

It eliminates the need to remember:

* CSV column names
* SQL column names
* Data types
* Transformation logic
* Lookup logic
* Which tables are loaded manually vs. through ETL

---

# Why Use a Source-to-Target Mapping?

During development it is common to ask questions like:

* What was the CSV column called?
* What did I rename it in MySQL?
* Which table does this field belong to?
* Is this imported or generated?
* Is this field looked up from another table?

Instead of searching SQL scripts, Word documents, or CSV files, the STM provides one place to answer these questions.

---

# Recommended Project Structure

```text
docs/
├── 01_Project_Overview.md
├── 02_Data_Dictionary.md
├── 03_Grain_Definitions.md
├── 04_Source_to_Target_Mapping.md
├── 05_ETL_Process.md
```

---

# Mapping Template

| Source File  | Source Column | Target Table | Target Column | Data Type | Transformation | Notes       |
| ------------ | ------------- | ------------ | ------------- | --------- | -------------- | ----------- |
| filename.csv | column_name   | table_name   | column_name   | VARCHAR   | Direct Import  | Description |

---

# Example – dim_technician

| Source File        | Source Column         | Target Table   | Target Column         | Data Type   | Transformation    | Notes                    |
| ------------------ | --------------------- | -------------- | --------------------- | ----------- | ----------------- | ------------------------ |
| dim_technician.csv | tech_key              | dim_technician | tech_key              | INT         | Direct Import     | Technician surrogate key |
| dim_technician.csv | emp_id_key            | dim_technician | emp_id_key            | VARCHAR(10) | Direct Import     | Employee identifier      |
| dim_technician.csv | job_title             | dim_technician | job_title             | VARCHAR(40) | Direct Import     | Technician job title     |
| dim_technician.csv | yrs_exp               | dim_technician | yrs_exp               | INT         | Direct Import     | Years of experience      |
| dim_technician.csv | job_title_yrs_exp_key | dim_technician | job_title_yrs_exp_key | VARCHAR(50) | Direct Import     | Business key             |
| dim_technician.csv | emp_alias             | dim_technician | emp_alias             | VARCHAR(10) | Direct Import     | Employee alias           |
| *(Generated)*      | —                     | dim_technician | load_date             | DATETIME    | CURRENT_TIMESTAMP | Load timestamp           |

---

# Example – ETL Lookup

Some columns are **not imported**.

Instead, they are looked up during ETL.

| Source Table | Source Column | Target Column | Transformation          |
| ------------ | ------------- | ------------- | ----------------------- |
| stg_payroll  | Employee_Name | emp_id_key    | Lookup in stg_employees |
| stg_payroll  | Date_Worked   | date_key      | Lookup in dim_date      |
| stg_payroll  | JobID         | job_key       | Lookup in dim_job       |

---

# Column Categories

Every column should fall into one of three categories.

## 1. Direct Import

Imported exactly as it appears in the source.

Examples:

* Employee Name
* Job Title
* Contract Number
* Sample Type

---

## 2. Derived

Calculated from other columns.

Examples:

* Profit
* Margin
* Revenue
* Cost
* job_title_yrs_exp_key

---

## 3. Lookup

Retrieved from another table.

Examples:

* Employee Key
* Job Key
* Contract Key
* Date Key

---

# Benefits

Using a Source-to-Target Mapping document:

* Eliminates guesswork during imports.
* Documents every field in the warehouse.
* Makes ETL development easier.
* Simplifies debugging.
* Speeds up future enhancements.
* Serves as technical documentation for the project.

---

# Best Practice

Maintain the STM throughout development.

Whenever a new table or column is added:

1. Update the SQL schema.
2. Update the Data Dictionary.
3. Update the Source-to-Target Mapping.

Keeping these documents synchronized ensures the warehouse remains easy to understand, maintain, and extend.
