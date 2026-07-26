# MySQL `LOAD DATA INFILE` Troubleshooting: CSV Line Ending Issues

## Scenario

Loading a CSV file into a MySQL table using:

```sql
LOAD DATA LOCAL INFILE ...
```

The CSV file was correctly formatted and the table schema matched the columns exactly.

---

## Symptoms

### Case 1 - Data shifted into the wrong columns

Instead of:

| employee_name | first_name | last_name | job_code |
| ------------- | ---------- | --------- | -------- |
| Cornel Albu   | Cornel     | Albu      | 01       |

The data looked like:

| employee_name | first_name  | last_name | job_code |
| ------------- | ----------- | --------- | -------- |
| 1             | Cornel Albu | Cornel    | Al       |

or other columns appeared shifted.

---

### Case 2 - Zero rows loaded

```text
Records: 0
Warnings: 0
```

Even though the file definitely contained data.

---

### Case 3 - Last column contained `\r`

Example:

```
cornelalbu01\r
```

instead of

```
cornelalbu01
```

---

# What was happening?

CSV files can use different line endings.

Windows:

```
\r\n
```

Linux / macOS:

```
\n
```

If MySQL expects the wrong line terminator, it may:

- fail to recognize records
- skip every row
- leave the carriage return (`\r`) attached to the final column

The data itself is fine.

The problem is simply how MySQL detects the end of each line.

---

# How to determine the line endings

In Terminal:

```bash
od -c filename.csv | head
```

Look for:

Windows:

```
...\r \n
```

Linux/macOS:

```
...\n
```

---

# Solution

Instead of

```sql
LINES TERMINATED BY '\r\n'
```

use

```sql
LINES TERMINATED BY '\n'
```

and clean the last field while importing.

Example:

```sql
LOAD DATA LOCAL INFILE '/path/to/file.csv'
INTO TABLE dim_technician
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(
    employee_name,
    first_name,
    last_name,
    job_code,
    sca_job_title,
    yrs_exp,
    @legacy_emp_key
)
SET
    legacy_emp_key = TRIM(TRAILING '\r' FROM @legacy_emp_key);
```

---

# Why this works

When using:

```sql
LINES TERMINATED BY '\n'
```

MySQL stops reading each record at the linefeed (`\n`).

If the file actually uses Windows line endings (`\r\n`), the carriage return (`\r`) remains attached to the final field.

Using

```sql
TRIM(TRAILING '\r' FROM @column)
```

removes that extra character before storing the value.

---

# How to recognize this problem in the future

Signs include:

- `LOAD DATA` reports **0 records loaded**
- Last column contains `\r`
- File appears perfectly formatted
- Manual `INSERT` works correctly
- Table schema matches the CSV exactly

Whenever these occur, check the CSV line endings before assuming the data or schema is incorrect.

---

# Lesson Learned

When debugging `LOAD DATA LOCAL INFILE`:

1. Verify the table schema.
2. Verify the CSV columns.
3. Test with a manual `INSERT`.
4. Inspect the file's line endings with `od -c`.
5. Match `LINES TERMINATED BY` to the file.
6. If using `LINES TERMINATED BY '\n'` with a CRLF file, trim the trailing `\r` from the last column during import.

This approach isolates the problem quickly and avoids chasing issues with the data or table definition when the real cause is simply the file's line endings.
