SQL Financial Analysis & Reporting
Corporate P&L, Time Intelligence, Joins, Views & Financial Reporting in PostgreSQL
Tool: PostgreSQL  |  Author: Ranit Bhattacharyya  |  GitHub: github.com/RanitBhattacharyya
Project Overview
This project covers end-to-end SQL financial analysis and reporting using PostgreSQL — simulating the kind of analytical work performed by analysts at major financial institutions. The project progresses from foundational SQL concepts through to building a complete Profit & Loss statement, covering 22 structured parts across data setup, querying, time intelligence, aggregations, joins, views, and financial reporting.

Datasets Used
Table
Description
gl
General Ledger — all financial transactions with date, account, territory and amount
coa
Chart of Accounts — account classifications including P&L and Balance Sheet
territory
Territory — country and region mapping for geographic analysis
calendar
Calendar — full date dimension with year, quarter, month, day


How Tables Connect
Relationship
Join Key
gl → coa
gl.account_key = coa.account_key
gl → territory
gl.territory_key = territory.territory_key
gl → calendar
gl.date = calendar.date


Project Parts — Full Breakdown
Part 4  —  Understanding the Data
Explored all 4 tables — GL, COA, Territory and Calendar. Understood column structures, data types, and how tables relate to each other through shared keys.
SELECT * FROM gl LIMIT 10;
SELECT * FROM coa LIMIT 10;
SELECT * FROM territory LIMIT 10;
SELECT * FROM calendar LIMIT 10;


Part 5  —  Our First SQL Query
Wrote basic SELECT queries to retrieve financial data. Learned column selection, aliasing, and result ordering.
SELECT entryno, date, amount
FROM gl
ORDER BY date ASC;


Part 6  —  Filtering to Specific Columns & Rows
Used WHERE clause to filter specific account keys and ranges. Learned the difference between text and numeric filtering.
SELECT * FROM gl
WHERE account_key BETWEEN 1 AND 30;


Part 7  —  Filtering Data with WHERE
Applied multiple filter conditions using AND, OR, and IN operators to narrow down financial transactions.
SELECT * FROM gl
WHERE account_key = 210
AND EXTRACT(YEAR FROM date) = 2020;


Part 8  —  Time Intelligence
Learned PostgreSQL date functions — EXTRACT for pulling year, month, quarter, day and day of year from date columns.
SELECT date,
EXTRACT(YEAR FROM date) AS year,
EXTRACT(MONTH FROM date) AS month,
EXTRACT(QUARTER FROM date) AS quarter,
EXTRACT(DAY FROM date) AS day
FROM calendar;


Part 9  —  Time Intelligence 2
Extended date analysis using TO_CHAR for formatting dates and extracting day names. Learned key PostgreSQL vs SQL Server syntax differences.
SELECT date,
EXTRACT(DOY FROM date) AS day_of_year,
TO_CHAR(date, 'Day') AS weekday,
TO_CHAR(date, 'Month') AS month_name
FROM calendar;


Part 10  —  Time Intelligence 3
Applied time filters to the GL table to analyse financial performance within specific time periods.
SELECT * FROM gl
WHERE EXTRACT(YEAR FROM date) = 2020
ORDER BY date ASC;


Part 11  —  SUM & GROUP BY
Aggregated financial data using SUM and GROUP BY to calculate total amounts by year. Core skill for financial reporting.
SELECT
EXTRACT(YEAR FROM date) AS year,
SUM(amount) AS total_amount
FROM gl
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY year;


Part 12  —  Subquery
Used subqueries to filter and summarise financial data in multiple layers — enabling more complex business questions to be answered.
SELECT account_key, total_amount
FROM (
  SELECT account_key, SUM(amount) AS total_amount
  FROM gl
  GROUP BY account_key
) AS summary
WHERE total_amount > 10000;


Part 13  —  PIVOT
Built pivot style reports using CASE statements — PostgreSQL equivalent of SQL Server PIVOT. Used to show financial amounts by year as separate columns.
SELECT
  account_key,
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2018 THEN amount ELSE 0 END) AS "2018",
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2019 THEN amount ELSE 0 END) AS "2019",
  SUM(CASE WHEN EXTRACT(YEAR FROM date) = 2020 THEN amount ELSE 0 END) AS "2020"
FROM gl
GROUP BY account_key
ORDER BY account_key;


Part 14  —  Practice — Subquery & PIVOT
Combined subqueries with CASE-based pivoting to produce multi-dimensional financial summaries across accounts and years.

Part 15  —  Creating Views in SQL
Created reusable SQL Views to store complex queries — enabling analysts to query the view like a table without rewriting logic every time.
CREATE VIEW vw_annual_summary AS
SELECT
  EXTRACT(YEAR FROM date) AS year,
  SUM(amount) AS total_amount
FROM gl
GROUP BY EXTRACT(YEAR FROM date);

-- Query the view
SELECT * FROM vw_annual_summary;


Part 16  —  Joining Tables in SQL
Joined GL with COA using INNER JOIN on account_key to enrich transaction data with account classifications like Report, Class and Account name.
SELECT gl.account_key, coa.report, coa.class, coa.account,
SUM(gl.amount) AS total_amount
FROM gl
JOIN coa ON gl.account_key = coa.account_key
GROUP BY gl.account_key, coa.report, coa.class, coa.account
ORDER BY gl.account_key;


Part 17  —  Joining Tables in SQL 2
Extended joins to include Territory table — enabling geographic financial analysis by country and region.
SELECT gl.account_key, t.country, t.region,
SUM(gl.amount) AS total_amount
FROM gl
JOIN territory t ON gl.territory_key = t.territory_key
GROUP BY gl.account_key, t.country, t.region
ORDER BY total_amount DESC;


Part 18  —  RIGHT & LEFT JOIN
Learned the difference between LEFT JOIN and RIGHT JOIN — how to retain all records from one table even when no matching record exists in the other.
-- LEFT JOIN -- keep all GL records even if no COA match
SELECT gl.account_key, coa.account
FROM gl
LEFT JOIN coa ON gl.account_key = coa.account_key;

-- RIGHT JOIN -- keep all COA records even if no GL match
SELECT gl.account_key, coa.account
FROM gl
RIGHT JOIN coa ON gl.account_key = coa.account_key;


Part 19  —  Practice SQL JOINs
Practice session combining multiple joins across GL, COA, Territory and Calendar to produce a full enriched financial dataset.
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


Part 20  —  Formatting Numbers in SQL
Applied TO_CHAR formatting to display financial amounts with comma separators for professional reporting output.
SELECT
  date,
  TO_CHAR(amount, 'FM999,999,999') AS amount_formatted
FROM gl;


Part 21  —  Preparing P&L Statement 1
Built the first layer of the Profit & Loss statement by joining GL with COA and filtering to Profit and Loss accounts only.
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


Part 22  —  Preparing P&L Statement 2
Completed the full multi-year P&L statement using CASE-based pivoting — showing Revenue, Cost of Sales, Gross Profit, Operating Expenses and Net Income across 2018, 2019 and 2020.
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


Key SQL Concepts Covered
SQL Concept
Applied In
SELECT, WHERE, ORDER BY
Parts 5, 6, 7
EXTRACT, TO_CHAR, Date Functions
Parts 8, 9, 10
SUM, GROUP BY, HAVING
Part 11
Subqueries
Parts 12, 14
CASE based PIVOT
Parts 13, 14, 22
CREATE VIEW
Part 15
INNER JOIN
Parts 16, 17
LEFT JOIN, RIGHT JOIN
Part 18
Multi table JOINs
Part 19
TO_CHAR number formatting
Part 20
P&L Statement build
Parts 21, 22


Key Financial Concepts Learned
Chart of Accounts — how financial transactions are classified into P&L and Balance Sheet categories
Profit & Loss Statement — Revenue, Cost of Sales, Gross Profit, Operating Expenses, Net Income
General Ledger — the master record of all financial transactions in an organisation
Multi-year financial comparison — how analysts compare performance across 2018, 2019, 2020
Geographic financial analysis — revenue and cost breakdown by country and region
Time Intelligence — how to slice financial data by year, quarter, month for trend analysis

About the Author
Ranit Bhattacharyya is a Senior Business Analyst with 7+ years of experience in data analytics and financial services, currently completing an MSc in Business Analytics at the University of Limerick, Ireland. Actively targeting Data Analyst, Financial Analyst, and FP&A roles in the Irish market.

LinkedIn: linkedin.com/in/ranitbhattacharyya  |  GitHub: github.com/RanitBhattacharyya
