# Building `dim_rfp`

## Objective

Create the **RFP Dimension** by combining the monthly status staging table with the Job Dimension.

**Grain**

> **One row = One RFP for one JobID**

The source of truth for RFP information is `stg_monthly_status`, while `dim_job` provides the surrogate key (`job_key`) used throughout the warehouse.

---

# ETL Process

```
stg_monthly_status
        │
        │  (contains RFP information)
        ▼
      JOIN
        ▲
        │
dim_job (lookup job_key)
        │
        ▼
     dim_rfp
```

---

# Step 1 – Test the Join

Before inserting data, always test the join with a `SELECT`.

This confirms:

- the join condition is correct
- every JobID exists in `dim_job`
- the correct columns are returned
- no duplicate rows are created

```sql
SELECT
    j.job_key,
    j.job_id,
    m.rfp_number,
    m.amount_invoiced,
    m.job_completion_date,
    m.rfp_date_submitted
FROM stg_monthly_status m
JOIN dim_job j
    ON m.job_id = j.job_id;
```

### Why this step is valuable

Never write an `INSERT` first.

If the `SELECT` returns exactly what you expect, the same query can safely become the source of the `INSERT`.

Think of the `SELECT` as a "preview" of the data that will be loaded.

---

# Step 2 – Build the INSERT

Once the `SELECT` returns the expected results:

```sql
INSERT INTO dim_rfp (
    job_key,
    job_id,
    rfp_number,
    rfp_amount_invoiced,
    date_job_complete,
    rfp_date
)
SELECT
    j.job_key,
    j.job_id,
    m.rfp_number,
    m.amount_invoiced,
    m.job_completion_date,
    m.rfp_date_submitted
FROM stg_monthly_status m
JOIN dim_job j
    ON m.job_id = j.job_id;
```

Notice that the staging table column names are translated into the business-friendly dimension column names during the ETL.

| Staging | Dimension |
|----------|-----------|
| amount_invoiced | rfp_amount_invoiced |
| job_completion_date | date_job_complete |
| rfp_date_submitted | rfp_date |

---

# Validation Checks

## 1. Verify rows loaded

```sql
SELECT COUNT(*)
FROM dim_rfp;
```

Compare with:

```sql
SELECT COUNT(*)
FROM stg_monthly_status;
```

The counts should match unless rows were intentionally excluded.

---

## 2. Look at the loaded data

```sql
SELECT *
FROM dim_rfp
ORDER BY job_key;
```

Verify that:

- JobIDs are correct
- RFP numbers are correct
- Amounts are correct
- Dates are correct
- `load_date` populated automatically

---

## 3. Check for duplicate RFPs

```sql
SELECT
    job_id,
    rfp_number,
    COUNT(*) AS duplicates
FROM dim_rfp
GROUP BY
    job_id,
    rfp_number
HAVING COUNT(*) > 1;
```

Expected result:

```
Empty set
```

---

## 4. Check for missing JobIDs

Every RFP should have a matching Job.

```sql
SELECT m.job_id
FROM stg_monthly_status m
LEFT JOIN dim_job j
    ON m.job_id = j.job_id
WHERE j.job_key IS NULL;
```

Expected result:

```
Empty set
```

If rows are returned, those JobIDs exist in the staging table but not in `dim_job`.

---

## 5. Verify the Grain

The grain of `dim_rfp` is:

> **One row = One RFP for one JobID**

Confirm this by checking for duplicates:

```sql
SELECT
    job_id,
    rfp_number,
    COUNT(*)
FROM dim_rfp
GROUP BY
    job_id,
    rfp_number
HAVING COUNT(*) > 1;
```

An empty result confirms the grain is preserved.

---

# Lessons Learned

- Always test an ETL with a `SELECT` before using `INSERT`.
- Join staging tables to dimensions to obtain surrogate keys.
- Staging tables retain the original source column names.
- Dimension tables use standardized business-friendly names.
- Validation queries are just as important as the ETL itself.
- A successful ETL is one that can be proven correct through validation.
