# ETL Validation Playbook

> **Purpose:**
> A repeatable process for validating data before, during, and after every ETL step. This playbook helps prevent bad joins, incorrect calculations, duplicate records, and silent data quality issues.

---

# ETL Workflow

```text
Extract
    ↓
Load Staging
    ↓
Validate Staging
    ↓
Load Dimensions
    ↓
Validate Dimensions
    ↓
Load Facts
    ↓
Validate Facts
    ↓
Enrich Facts
    ↓
Validate Business Rules
    ↓
Publish
```

---

# Phase 1 — Source Validation

## Objective

Verify that the source file or source table is complete before loading.

### Checklist

- [ ] Source file exists
- [ ] Correct file name
- [ ] Correct file format (.csv, .xlsx, etc.)
- [ ] Expected number of rows
- [ ] Expected number of columns
- [ ] Header names are correct
- [ ] No missing required columns

### Example

```sql
SELECT COUNT(*)
FROM stg_payroll;
```

---

# Phase 2 — Load Validation

## Objective

Confirm the load completed successfully.

### Checklist

- [ ] No SQL errors
- [ ] No warnings
- [ ] Correct row count
- [ ] Auto-increment keys generated correctly
- [ ] Default values populated

Example:

```sql
SELECT COUNT(*)
FROM dim_labor_sca_map;
```

---

# Phase 3 — Data Quality Validation

## Objective

Ensure the data is clean enough to use.

## Check for NULL Join Keys

```sql
SELECT COUNT(*)
FROM fact_labor
WHERE contract_no IS NULL;
```

Expected:

```
0
```

---

## Check for Hidden Characters

Useful for finding carriage returns, tabs, or spaces.

```sql
SELECT DISTINCT
CONCAT('[',contract_no,']')
FROM dim_labor_sca_map;
```

If you see:

```
[C000015128\r]
```

Clean the data before joining.

---

## Check for Leading / Trailing Spaces

```sql
SELECT
contract_no,
LENGTH(contract_no),
LENGTH(TRIM(contract_no))
FROM dim_labor_sca_map;
```

---

## Verify Data Types

Examples

- INT joins INT
- DATE joins DATE
- VARCHAR joins VARCHAR

Avoid relying on implicit conversions.

---

# Phase 4 — Business Key Validation

## Objective

Confirm that the business key uniquely identifies one row.

### Ask

> What does the business consider to be unique?

Examples

| Table             | Business Key      |
| ----------------- | ----------------- |
| dim_job           | job_id            |
| dim_contract      | contract_no       |
| dim_date          | full_date         |
| dim_technician    | legacy_emp_key    |
| dim_labor_sca_map | map_sca_labor_key |

---

## Check for Duplicate Business Keys

```sql
SELECT
map_sca_labor_key,
COUNT(*) AS cnt
FROM dim_labor_sca_map
GROUP BY map_sca_labor_key
HAVING COUNT(*) > 1;
```

Expected:

```
0 rows
```

If duplicates exist:

1. Inspect them.
2. Determine whether they are identical.
3. Determine whether the business key is incomplete.
4. Correct the source data or redesign the key.

---

# Phase 5 — Domain Validation

## Objective

Confirm values match business expectations.

Examples

- Technician titles
- Contract numbers
- Experience levels
- Categories

Compare distinct values.

```sql
SELECT DISTINCT sca_job_title
FROM dim_technician;

SELECT DISTINCT title
FROM dim_labor_sca_map;
```

---

# Phase 6 — Referential Integrity Validation

## Objective

Ensure every lookup value exists.

Example:

```sql
SELECT
fl.contract_no
FROM fact_labor fl
LEFT JOIN dim_contract dc
ON fl.contract_no = dc.contract_no
WHERE dc.contract_no IS NULL;
```

Expected:

```
0 rows
```

---

# Phase 7 — Test the Join

Never write an UPDATE first.

Always test.

```sql
SELECT
*
FROM fact_labor fl
JOIN dim_labor_sca_map lm
ON fl.map_sca_labor_key = lm.map_sca_labor_key
LIMIT 20;
```

Verify:

- Correct matches
- No duplicate rows
- No unexpected NULL values

---

# Phase 8 — Perform the Update

Only after validation succeeds.

Example

```sql
UPDATE ...
JOIN ...
SET ...
```

---

# Phase 9 — Post-Update Validation

## Verify Row Counts

```sql
SELECT COUNT(*)
FROM fact_labor;
```

---

## Verify Updated Values

```sql
SELECT
actual_bill_rate,
revenue,
cost,
profit
FROM fact_labor
LIMIT 20;
```

Look for:

- NULL values
- Negative numbers
- Impossible values
- Unexpected zeros

---

# Common ETL Problems

| Symptom                               | Likely Cause            | How to Diagnose                    | Resolution                                                        |
| ------------------------------------- | ----------------------- | ---------------------------------- | ----------------------------------------------------------------- |
| 0 rows returned from JOIN             | Empty lookup table      | COUNT(\*)                          | Load lookup table                                                 |
| JOIN returns fewer rows than expected | Missing lookup values   | LEFT JOIN unmatched rows           | Fix lookup data                                                   |
| Duplicate rows after JOIN             | Duplicate business keys | GROUP BY ... HAVING COUNT(\*) > 1  | Remove duplicates or redesign key                                 |
| SQL Error 1175                        | Safe Update Mode        | Error message                      | Disable safe updates temporarily or use a key in the WHERE clause |
| SQL Error 3948                        | LOCAL INFILE disabled   | SHOW VARIABLES LIKE 'local_infile' | Enable LOCAL INFILE and reconnect                                 |
| NULL values after UPDATE              | Join failed             | LEFT JOIN unmatched rows           | Investigate join keys                                             |
| Unexpected duplicate facts            | Incorrect grain         | Review business process            | Redesign fact table grain                                         |
| Hidden characters (\r, tabs, spaces)  | CSV export              | CONCAT('[', column, ']')           | Use TRIM() / REPLACE() during load                                |

---

# ETL Decision Tree

```text
Load Data
    │
    ▼
Did the load succeed?
    │
 ┌──┴──┐
 │     │
No     Yes
 │      │
Fix     ▼
      Validate Row Count
            │
            ▼
Are required columns populated?
            │
      ┌─────┴─────┐
      │           │
     No          Yes
      │           │
 Fix Source       ▼
          Validate Business Keys
                  │
          ┌───────┴────────┐
          │                │
     Duplicates?          Unique?
          │                │
 Investigate         Test JOIN
          │                │
          ▼                ▼
     Clean Data      Any Unmatched Rows?
                           │
                     ┌─────┴─────┐
                     │           │
                    Yes         No
                     │           │
             Investigate     UPDATE
                                 │
                                 ▼
                      Validate Results
```

---

# Standard Questions Before Every Join

- [ ] What is the grain of each table?
- [ ] What is the business key?
- [ ] Is the business key unique?
- [ ] Are the join columns populated?
- [ ] Do the data types match?
- [ ] Are the values formatted consistently?
- [ ] Are there hidden characters or whitespace?
- [ ] Does a sample JOIN return the expected rows?
- [ ] Are there any unmatched records?
- [ ] Have I validated the results after the UPDATE?

---

# Guiding Principle

> **Never trust a successful SQL statement by itself.**
>
> A query that runs without errors can still produce incorrect results.
>
> Validate every assumption:
>
> - Validate the source.
> - Validate the keys.
> - Validate the join.
> - Validate the business rules.
> - Validate the output.
>
> ETL is not complete until the data is proven correct.
