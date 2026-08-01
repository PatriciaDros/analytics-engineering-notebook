# BUILD_LOG.md

**Project:** Environmental Operational BI Warehouse (Portfolio Release 1.0)

**Started:** August 1, 2026

---

# Session Log

---

## August 1, 2026

### Milestone Reached

Today marks the official restart of the warehouse using the curated portfolio dataset.

The previous database has been retired. Going forward, all development will use the anonymized dataset and the finalized staging schemas.

This build represents **Release 1.0** of the portfolio warehouse.

---

### Decisions Made

#### Confidentiality

* Removed all employee names.
* Removed all legacy employee keys.
* Retained `emp_alias` for development and validation.
* Removed `primary_hygienist`.
* Public school names, addresses, and building IDs will remain.
* Job IDs will remain.
* Monetary values will be anonymized by multiplying every monetary field by the same constant factor.

---

#### Naming Standards

Updated naming conventions:

* `emp_id`
* `wa_number`
* `rfp_number`

Removed:

* `job_title` from `stg_payroll`

---

#### Staging Philosophy

The staging layer should closely represent the source systems.

Allowed changes:

* Data cleaning
* Standardized column names
* Standardized data types
* Confidentiality changes
* `load_date`

The staging layer should **not** contain business logic or calculated fields.

---

#### Build Strategy

Instead of creating every table first, each staging table will be completed individually.

For every table:

1. Create table
2. Load CSV
3. Verify row count
4. Inspect sample data
5. Verify data types
6. Confirm business meaning
7. Mark complete

Only then move to the next table.

Reason:

This project is simultaneously:

* Learning SQL
* Building a professional portfolio
* Validating business logic
* Being developed over many work sessions

Smaller completed units reduce confusion and make it much easier to resume after time away.

---

### SQL Notes

Remember:

`DESCRIBE` output is **not** `CREATE TABLE` syntax.

Do **not** copy:

* YES
* NULL
* DEFAULT_GENERATED

into table definitions.

Use readable SQL formatting:

* One column per line
* Consistent indentation
* Monetary values as `DECIMAL`
* `load_date DATETIME DEFAULT CURRENT_TIMESTAMP`

---

# Current Status

## Overall Phase

☑ Curated dataset finalized

☑ Staging schemas finalized

☑ Anonymization complete

☐ Export CSV files

☐ Drop existing database

☐ Create new database

☐ Build staging tables

☐ Build dimensions

☐ Build fact tables

☐ Validate warehouse

☐ Build KPIs

---

# Current Task

Export every staging worksheet from Google Sheets as individual CSV files.

After exporting:

1. Drop existing database.
2. Create new database.
3. Begin building staging tables.

---

# Current Table

**None**

The database has not yet been rebuilt.

The first table will be:

`stg_contract_values`

---

# Definition of Done (Per Table)

A table is complete only when all boxes are checked.

Schema

☐ Created successfully

Data

☐ CSV imported

Validation

☐ Row count verified

☐ Sample records reviewed

☐ Data types verified

☐ Business meaning confirmed

Result

☐ Table complete

---

# Parking Lot

Ideas for later.

* Documentation
* Repository structure
* README
* ER Diagram
* Portfolio screenshots

These items are intentionally deferred until after the warehouse is functioning correctly.

---

# End-of-Session Checklist

Before stopping work each day, answer these questions.

### Today's Win

What did I complete?

---

### Current State

Exactly where did I stop?

---

### Next Action

What is the **very next thing** I should do when I return?

---

### Questions / Blockers

Anything I need to remember before continuing?

---

### Notes

Anything learned today that Future Me should know?
