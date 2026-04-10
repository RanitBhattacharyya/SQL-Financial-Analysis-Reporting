# 💰 SQL Financial Analysis & Reporting
### Corporate P&L, Time Intelligence, Joins, Views & Financial Reporting in PostgreSQL

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Financial%20Analysis-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-green?style=for-the-badge)

---

## 📌 Project Overview

This project covers end-to-end SQL financial analysis and reporting using PostgreSQL — simulating the kind of analytical work performed by analysts at major financial institutions like Barclays, Goldman Sachs, and Morgan Stanley.

The project progresses from foundational SQL concepts through to building a complete **Profit & Loss Statement**, covering **22 structured parts** across data setup, querying, time intelligence, aggregations, joins, views, and financial reporting.

---

## 📂 Datasets Used

| Table | Description |
|---|---|
| `gl` | General Ledger — all financial transactions with date, account, territory and amount |
| `coa` | Chart of Accounts — account classifications including P&L and Balance Sheet |
| `territory` | Territory — country and region mapping for geographic analysis |
| `calendar` | Calendar — full date dimension with year, quarter, month, day |

---

## 🔗 How Tables Connect

| Relationship | Join Key |
|---|---|
| gl → coa | gl.account_key = coa.account_key |
| gl → territory | gl.territory_key = territory.territory_key |
| gl → calendar | gl.date = calendar.date |

---

## 📋 Project Parts — Full Breakdown

---

### Part 4 — Understanding the Data
Explored all 4 tables. Understood column structures, data types, and how tables relate to each other through shared keys.

```sql
SELECT * FROM gl LIMIT 10;
SELECT * FROM coa LIMIT 10;
SELECT * FROM territory LIMIT 10;
SELECT * FROM calendar LIMIT 10;
```

---

### Part 5 — Our First SQL Query
Wrote basic SELECT queries to retrieve financial data. Learned column selection, aliasing, and result ordering.

```sql
SELECT entryno, date, amount
FROM gl
ORDER BY date ASC;
```

---

### Part 6 — Filtering to Specific Columns & Rows
Used WHERE clause to filter specific account keys and ranges.

```sql
SELECT * FROM gl
WHERE account_key BETWEEN 1 AND 30;
```

---

### Part 7 — Filtering Data with WHERE
Applied multiple filter conditions using AND, OR operators.

```sql
SELECT * FROM gl
WHERE account_key = 210
AND EXTRACT(YEAR FROM date) = 2020;
```

---

### Part 8 — Time Intelligence
Learned PostgreSQL date functions — EXTRACT for pulling year, month, quarter, day from date columns.

```sql
SELECT date,
EXTRACT(YEAR FROM date) AS year,
EXTRACT(MONTH FROM date) AS month,
EXTRACT(QUARTER FROM date) AS quarter,
EXTRACT(DAY FROM date) AS day
FROM calendar;
```

---

### Part 9 — Time Intelligence 2
Extended date analysis using TO_CHAR for formatting dates and extracting day names.

```sql
SELECT date,
EXTRACT(DOY FROM date) AS day_of_year,
TO_CHAR(date, 'Day') AS weekday,
TO_CHAR(date, 'Month') AS month_name
FROM calendar;
```

---

### Part 10 — Time Intelligence 3
Applied time filters to the GL table to analyse financial performance within specific time periods.

```sql
SELECT * FROM gl
WHERE EXTRACT(YEAR FROM date) = 2020
ORDER BY date ASC;
```

---

### Part 11 — SUM & GROUP BY
Aggregated financial data using SUM and GROUP BY to calculate total amounts by year.

```sql
SELECT
EXTRACT(YEAR FROM date) AS year,
SUM(amount) AS total_amount
FROM gl
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY year;
```

---

### Part 12 — Subquery
Used subqueries to filter and summarise financial data in multiple layers.

```sql
SELECT account_key, total_amount
FROM (
  SELECT account_key, SUM(amount) AS total_amount
  FROM gl
  GROUP BY account_key
) AS summary
WHERE total_amount > 10000;
```

---

### Part 13 — PIVOT
Built pivot style reports using CASE statements — PostgreSQL equivalent of SQL Server PIVOT.

```sql
SELECT
  account_key,
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2018 THEN amount ELSE 0 END) AS "2018",
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2019 THEN amount ELSE 0 END) AS "2019",
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2020 THEN amount ELSE 0 END) AS "2020"
FROM gl
GROUP BY account_key
ORDER BY account_key;
```

---

### Part 14 — Practice Subquery & PIVOT
Combined subqueries with CASE-based pivoting to produce multi-dimensional financial summaries.

---

### Part 15 — Creating Views in SQL
Created reusable SQL Views to store complex queries.

```sql
CREATE VIEW vw_annual_summary AS
SELECT
  EXTRACT(YEAR FROM date) AS year,
  SUM(amount) AS total_amount
FROM gl
GROUP BY EXTRACT(YEAR FROM date);

-- Query the view
SELECT * FROM vw_annual_summary;
```

---

### Part 16 — Joining Tables in SQL
Joined GL with COA using INNER JOIN on account_key.

```sql
SELECT gl.account_key, coa.report, coa.class, coa.account,
SUM(gl.amount) AS total_amount
FROM gl
JOIN coa ON gl.account_key = coa.account_key
GROUP BY gl.account_key, coa.report, coa.class, coa.account
ORDER BY gl.account_key;
```

---

### Part 17 — Joining Tables in SQL 2
Extended joins to include Territory table for geographic financial analysis.

```sql
SELECT gl.account_key, t.country, t.region,
SUM(gl.amount) AS total_amount
FROM gl
JOIN territory t ON gl.territory_key = t.territory_key
GROUP BY gl.account_key, t.country, t.region
ORDER BY total_amount DESC;
```

---

### Part 18 — RIGHT & LEFT JOIN
Learned the difference between LEFT JOIN and RIGHT JOIN.

```sql
-- LEFT JOIN -- keep all GL records even if no COA match
SELECT gl.account_key, coa.account
FROM gl
LEFT JOIN coa ON gl.account_key = coa.account_key;

-- RIGHT JOIN -- keep all COA records even if no GL match
SELECT gl.account_key, coa.account
FROM gl
RIGHT JOIN coa ON gl.account_key = coa.account_key;
```

---

### Part 19 — Practice SQL JOINs
Combined multiple joins across all 4 tables.

```sql
SELECT
  c.year, t.country, t.region,
  coa.class, coa.account,
  SUM(gl.amount) AS total_amount
FROM gl
JOIN coa ON gl.account_key = coa.account_key
JOIN territory t ON gl.territory_key = t.territory_key
JOIN calendar c ON gl.date = c.date
GROUP BY c.year, t.country, t.region, coa.class, coa.account
ORDER BY c.year, total_amount DESC;
```

---

### Part 20 — Formatting Numbers in SQL
Applied TO_CHAR formatting to display financial amounts with comma separators.

```sql
SELECT
  date,
  TO_CHAR(amount, 'FM999,999,999') AS amount_formatted
FROM gl;
```

---

### Part 21 — Preparing P&L Statement 1
Built the first layer of the Profit & Loss statement.

```sql
SELECT
  gl.account_key,
  coa.report,
  coa.class,
  coa.account,
  SUM(gl.amount) AS total_amount
FROM gl
JOIN coa ON gl.account_key = coa.account_key
WHERE coa.report = 'Profit and Loss'
GROUP BY gl.account_key, coa.report, coa.class, coa.account
ORDER BY gl.account_key ASC;
```

---

### Part 22 — Preparing P&L Statement 2
Completed the full multi-year P&L statement using CASE-based pivoting.

```sql
SELECT
  gl.account_key,
  coa.report,
  coa.class,
  coa.account,
  SUM(CASE WHEN EXTRACT(YEAR FROM gl.date) = 2018 THEN gl.amount ELSE 0 END) AS "2018",
  SUM(CASE WHEN EXTRACT(YEAR FROM gl.date) = 2019 THEN gl.amount ELSE 0 END) AS "2019",
  SUM(CASE WHEN EXTRACT(YEAR FROM gl.date) = 2020 THEN gl.amount ELSE 0 END) AS "2020"
FROM gl
JOIN coa ON gl.account_key = coa.account_key
WHERE coa.report = 'Profit and Loss'
GROUP BY gl.account_key, coa.report, coa.class, coa.account
ORDER BY gl.account_key ASC;
```

---

## 📊 SQL Concepts Covered

| SQL Concept | Applied In |
|---|---|
| SELECT, WHERE, ORDER BY | Parts 5, 6, 7 |
| EXTRACT, TO_CHAR, Date Functions | Parts 8, 9, 10 |
| SUM, GROUP BY | Part 11 |
| Subqueries | Parts 12, 14 |
| CASE based PIVOT | Parts 13, 14, 22 |
| CREATE VIEW | Part 15 |
| INNER JOIN | Parts 16, 17 |
| LEFT JOIN, RIGHT JOIN | Part 18 |
| Multi table JOINs | Part 19 |
| Number Formatting | Part 20 |
| P&L Statement | Parts 21, 22 |

---

## 💡 Key Financial Concepts Learned

- **Chart of Accounts** — how financial transactions are classified into P&L and Balance Sheet categories
- **Profit & Loss Statement** — Revenue, Cost of Sales, Gross Profit, Operating Expenses, Net Income
- **General Ledger** — the master record of all financial transactions in an organisation
- **Multi-year financial comparison** — how analysts compare performance across 2018, 2019, 2020
- **Geographic financial analysis** — revenue and cost breakdown by country and region
- **Time Intelligence** — slicing financial data by year, quarter, month for trend analysis

---

## ⚠️ Key PostgreSQL vs SQL Server Differences

| Feature | SQL Server | PostgreSQL |
|---|---|---|
| Extract year | YEAR() | EXTRACT(YEAR FROM date) |
| Format numbers | FORMAT() | TO_CHAR() |
| Pivot | PIVOT | CASE statements |
| Identifiers | Square brackets [] | Double quotes "" |
| Limit rows | TOP | LIMIT |

---

## 👤 About the Author

**Ranit Bhattacharyya**
Senior Business Analyst | MSc Business Analytics — University of Limerick, Ireland

- 🔗 [LinkedIn](https://linkedin.com/in/ranitbhattacharyya)
- 💻 [GitHub](https://github.com/RanitBhattacharyya)
- 📧 ranitbhattacharyya07@gmail.com

---

*Next Project: Corporate Financial Analysis using Kaggle dataset — P&L reporting, Budget vs Actuals, Cost to Income Ratio, and Rolling 12-Month Trend Analysis*
