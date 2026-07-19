# SQL Notebook – Designing Queries Like an Analytics Engineer

## The Most Important Lesson

**Don't start by writing SQL.**

Start by asking:

> **"What am I trying to produce?"**

Every SQL query exists to answer a business question.

---

# Step 1 — Define the Final Grain

Before writing anything, answer:

> **What does one row represent in my final result?**

Examples:

- One row per customer
- One row per signup quarter
- One row per city
- One row per job
- One row per contract

The grain determines everything that follows.

---

# Step 2 — Identify the Business Metrics

Ask:

> **What numbers do I need on each row?**

Examples:

- Total customers
- Churned customers
- Churn rate
- Total revenue
- Average order value
- Total hours worked
- Profit

If the metric uses:

- COUNT()
- SUM()
- AVG()
- MIN()
- MAX()

then it is an **aggregate**.

---

# Step 3 — Compute First, Reason Second

This is the biggest mental shift.

Think in layers.

## Layer 1 — Compute the Facts

Calculate everything you need.

Examples:

- Total customers
- Revenue
- Churn rate
- Profit

This is usually where:

- GROUP BY
- COUNT()
- SUM()
- AVG()

live.

Think:

> **Create the facts.**

---

## Layer 2 — Reason About the Facts

Once the facts exist, ask questions about them.

Examples:

- Rank them
- Filter Top 5
- Compare to averages
- Find highest
- Find lowest

Think:

> **Analyze the facts.**

---

# The Layer Rule

If I need to **make a decision using a calculated metric**, I probably need another query layer.

Examples:

Calculate churn rate → then rank it.

Calculate revenue → then find the Top 10.

Calculate profit → then compare it to the company average.

---

# SQL Execution Order

SQL does **not** execute from top to bottom.

It executes in this order:

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. Window Functions
7. ORDER BY
8. LIMIT

Understanding this explains why subqueries and CTEs exist.

---

# Why Use a Subquery or CTE?

A subquery (or CTE) lets SQL finish one job before starting another.

Example:

Layer 1

- Calculate churn_rate

Layer 2

- Rank churn_rate

Instead of trying to do everything at once, split the work into logical steps.

---

# Ranking

## ORDER BY

Used when you simply want the highest or lowest values.

Example:

Highest churn rate

Lowest revenue

Top-selling products

This is the most common form of ranking.

---

## Window Ranking Functions

### ROW_NUMBER()

Every row gets a unique number.

Example:

1, 2, 3, 4

Use when every row must be unique.

---

### RANK()

Rows with equal values receive the same rank.

Example:

1, 2, 2, 4

Notice that rank 3 is skipped.

Think of finishing places in a race.

---

### DENSE_RANK()

Rows with equal values receive the same rank.

Example:

1, 2, 2, 3

No numbers are skipped.

Useful for "Top N" style reports.

---

# Choosing the Right Ranking Function

Ask one question:

**Should equal values receive the same rank?**

If no:

→ ROW_NUMBER()

If yes:

Ask:

**Should ranks skip numbers after a tie?**

Yes:

→ RANK()

No:

→ DENSE_RANK()

---

# The Analytics Engineer Mindset

Do not think:

> "What SQL function do I need?"

Instead ask:

1. What business question am I answering?
2. What should one row represent?
3. What metrics must exist?
4. What must be calculated first?
5. What decisions happen after the calculations?

The SQL usually writes itself after these questions are answered.

---

# Canonical Query Pattern

Almost every analytical SQL query follows this pattern:

1. Define the grain.
2. Compute the business metrics.
3. Wrap the results (subquery or CTE if needed).
4. Rank, filter, compare, or sort.
5. Present the answer.

Think:

**Compute → Then Reason**

instead of

**Write one giant SQL statement.**

---

# Connection to Project Observatory

This is the same mindset used to build a dimensional warehouse.

When designing fact tables, I ask:

> **"What business fact am I missing?"**

When designing SQL queries, I ask:

> **"What business answer am I trying to produce?"**

Both start with business thinking first and SQL second.

---

# Rules Worth Memorizing

- Define the grain before writing SQL.
- Aggregate before ranking.
- Compute first. Reason second.
- If a calculated metric drives a decision, use another query layer.
- SQL is easier when built in layers rather than written as one giant statement.

> **The goal is not to memorize SQL syntax. The goal is to think like an analytics engineer who uses SQL to answer business questions.**
