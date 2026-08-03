# Building `dim_date` in MySQL

## Purpose

The `dim_date` table is a **date dimension** used in a star schema data warehouse. Instead of storing date information repeatedly in fact tables, a single dimension contains useful calendar attributes that can be joined to any fact table.

Typical uses include:

- Filtering by year, quarter, month, or weekday
- Grouping reports by time periods
- Comparing trends over time
- Supporting Power BI, Tableau, Excel Power Pivot, and SQL reporting

---

# Table Structure

```sql
CREATE TABLE dim_date (
    DateKey INT PRIMARY KEY,
    FullDate DATE NOT NULL,
    DayName VARCHAR(10),
    MonthName VARCHAR(10),
    CalendarQuarter VARCHAR(2),
    CalendarYear INT,
    WeekEndingDate DATE,
    WeekdayFlag VARCHAR(10)
);
```

---

# Column Breakdown

| Column | Data Type | Purpose |
|---------|-----------|---------|
| DateKey | INT | Surrogate key in YYYYMMDD format (e.g. 20240115) |
| FullDate | DATE | Actual calendar date |
| DayName | VARCHAR(10) | Monday, Tuesday, etc. |
| MonthName | VARCHAR(10) | January, February, etc. |
| CalendarQuarter | VARCHAR(2) | Q1, Q2, Q3, Q4 |
| CalendarYear | INT | Four-digit year |
| WeekEndingDate | DATE | End of payroll/reporting week |
| WeekdayFlag | VARCHAR(10) | Weekday or Weekend |

---

# Why is DateKey an INT?

Instead of using the DATE as the primary key, many data warehouses use an integer.

Example:

| Date | DateKey |
|------|---------|
| 2022-01-01 | 20220101 |
| 2022-01-02 | 20220102 |

Advantages:

- Smaller indexes
- Faster joins
- Standard practice in dimensional modeling
- Easy to recognize by humans

---

# Increasing the Recursive Limit

```sql
SET SESSION cte_max_recursion_depth = 2000;
```

Recursive Common Table Expressions (CTEs) have a default recursion limit.

Since this script generates nearly 1,900 dates, the recursion limit must be increased.

---

# Generating Every Date

```sql
WITH RECURSIVE dates AS (
    SELECT DATE('2022-01-01') AS dt

    UNION ALL

    SELECT DATE_ADD(dt, INTERVAL 1 DAY)
    FROM dates
    WHERE dt < '2026-12-31'
)
```

How it works:

1. Start with January 1, 2022.
2. Add one day.
3. Repeat.
4. Stop when December 31, 2026 is reached.

The result is one row for every calendar date.

---

# Populating the Table

```sql
INSERT INTO dim_date (...)
SELECT ...
FROM dates;
```

The `SELECT` transforms each generated date into useful reporting attributes before inserting them into the dimension.

---

# Creating the DateKey

```sql
DATE_FORMAT(dt,'%Y%m%d')
```

Produces:

```
20220101
20220102
20220103
```

Although `DATE_FORMAT()` returns text, MySQL automatically converts it into an integer when inserting into the `INT` column.

Alternative approach:

```sql
YEAR(dt)*10000
+ MONTH(dt)*100
+ DAY(dt)
```

Both produce the same result, but `DATE_FORMAT()` is easier to read.

---

# Day Name

```sql
DAYNAME(dt)
```

Examples:

```
Monday
Tuesday
Wednesday
```

Useful for reporting and filtering.

---

# Month Name

```sql
MONTHNAME(dt)
```

Examples:

```
January
February
March
```

Makes reports easier to read than numeric month values.

---

# Quarter

```sql
CONCAT('Q', QUARTER(dt))
```

Produces:

```
Q1
Q2
Q3
Q4
```

This is commonly used in dashboards and executive reporting.

---

# Calendar Year

```sql
YEAR(dt)
```

Produces:

```
2022
2023
2024
```

Used for yearly summaries and filtering.

---

# Week Ending Date

Current calculation:

```sql
DATE_ADD(
    dt,
    INTERVAL (7 - DAYOFWEEK(dt)) DAY
)
```

### Important

This calculation assumes **weeks end on Saturday**.

If your payroll or business defines weeks differently (for example, ending on Sunday), this calculation should be adjusted.

Always ensure the `WeekEndingDate` matches the business definition used in your source systems.

---

# Weekday Flag

```sql
CASE
    WHEN DAYOFWEEK(dt) IN (1,7)
    THEN 'Weekend'
    ELSE 'Weekday'
END
```

Produces:

| Day | Result |
|------|--------|
| Monday | Weekday |
| Tuesday | Weekday |
| Saturday | Weekend |
| Sunday | Weekend |

Useful for filtering business days from weekends.

---

# Good Design Choices

This implementation demonstrates several data warehousing best practices:

- Uses a surrogate integer key.
- Uses a recursive CTE instead of manually entering dates.
- Generates all dates automatically.
- Stores useful reporting attributes.
- Separates calendar logic from fact tables.
- Produces a reusable date dimension for all reports.

---

# Possible Future Enhancements (Version 2)

Additional attributes often found in enterprise data warehouses include:

| Column | Example |
|---------|----------|
| DayOfMonth | 15 |
| DayOfYear | 196 |
| MonthNumber | 7 |
| WeekOfYear | 29 |
| QuarterNumber | 3 |
| MonthStartDate | 2022-07-01 |
| MonthEndDate | 2022-07-31 |
| IsLeapYear | Yes |
| FiscalYear | 2024 |
| FiscalQuarter | Q3 |

These are useful for more advanced reporting but are not necessary for a first version of the project.
