# Building `dim_job`

Yes! This is where the fun starts. 😄

We're going to build `dim_job` the same way we built our Excel model—one small, intentional step at a time.

---

# Step 1 - Open a New SQL Script

Create a new query tab in MySQL Workbench and save it as:

```text
01_build_dim_job.sql
```

---

# Step 2 - Examine the Source Data

Before writing any ETL, always inspect the source data.

Run:

```sql
SELECT *
FROM stg_job_list
LIMIT 10;
```

> **Why?**
>
> Never assume you remember what the source looks like. Reviewing the data first helps identify column names, data types, missing values, and anything unexpected before you begin writing SQL.

---

# Step 3 - Examine the Destination Table

Next, look at the destination table.

Run:

```sql
SELECT *
FROM dim_job;
```

Since this is a newly created dimension table, it should return **0 rows**.

The goal isn't to see data yet—it's to verify:

- The column names
- The column order
- That the table exists as expected

---

# Step 4 - Review the ETL Pattern

Most dimension loads follow the same basic pattern:

```sql
INSERT INTO dim_job (
    column1,
    column2,
    column3
)
SELECT
    column1,
    column2,
    column3
FROM stg_job_list;
```

We are **not** ready to write the actual `INSERT` statement yet.

Before loading data, we need to carefully map each source column from `stg_job_list` to its corresponding destination column in `dim_job`.

Just like we did in Excel, every column should be placed intentionally—not simply copied over.

---

# Your Turn

Please run the following commands and paste the results.

### Source Table

```sql
DESCRIBE stg_job_list;
```

### Destination Table

```sql
DESCRIBE dim_job;
```

Once I see both table schemas, we'll write the **first `INSERT ... SELECT` statement together**.

We'll:

- Map every column intentionally.
- Discuss **why** each field belongs in `dim_job`.
- Build a proper ETL statement instead of blindly copying data.
- Execute the load.
- Watch your first dimension table come to life.
