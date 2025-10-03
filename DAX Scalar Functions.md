# DAX Scalar Functions: Comprehensive Course Notes with Coffee Retail Dataset Examples

Top takeaway: DAX scalar functions return a single value and are the core building blocks for measures and calculated columns. Mastering them across the categories below—Aggregation, Rounding, Information, Conversion, Logical, and Ranking—enables accurate business logic, robust data quality, and clear reporting.

## What Are Scalar Functions in DAX?

Scalar functions evaluate expressions row-by-row or in filter context and return a single value. They are different from table functions, which return entire tables. Common use cases include:

- Aggregating values into KPIs (e.g., total sales, average cost, max price)[^1]
- Converting data types and formatting values (e.g., text to dates, numbers to currency)[^1]
- Evaluating conditions and returning values based on TRUE/FALSE logic[^1]

The provided reference document introduces five major categories—Aggregation, Rounding, Information, Conversion, Logical—and also highlights iterator “X” patterns and important tips like SUM vs. SUMX evaluation (“syntax sugar”).[^1]

Below, each category is explained with definitions, syntaxes, gotchas, and hands‑on examples using the shared CSVs:

- Product Lookup.csv (product, category, current_cost, current_retail_price, etc.)
- Sales by Store.csv (transactions, quantity_sold, etc.)
- Customer Lookup.csv (customer attributes, useful for age/format examples)
- Store Lookup.csv (store characteristics)
- Calendar.csv (date dimension formatting)

Citations following statements refer to the attached PDF where definitions and function lists appear.

***

## 1) Aggregation Functions

Overview: Aggregate a column or expression to a single value within the current filter context. Key functions: SUM, AVERAGE, MAX, MIN, COUNT, COUNTA, DISTINCTCOUNT, COUNTROWS, plus iterator “X” versions (SUMX, AVERAGEX, etc.).[^1]

Important tip: SUM is internally evaluated like SUMX over a column expression—this is “syntax sugar” and applies to other aggregations as well.[^1]

Common syntaxes and uses:

- SUM(Column)
- AVERAGE(Column)
- MAX(Column) or MAX(Scalar1, Scalar2)
- MIN(Column) or MIN(Scalar1, Scalar2)
- COUNT(Column with numbers), COUNTA(Any non-blank), DISTINCTCOUNT(Column), COUNTROWS(Table)[^1]

Pro tip for cardinality: COUNTROWS(VALUES(Column)) may be more engine-friendly than DISTINCTCOUNT on very large datasets.[^1]

Practical examples with the coffee retail dataset:

- Total Units Sold:
Measure: Total Units = SUM('Sales by Store'[quantity_sold])[^1]
Use: KPI on transactions by store, product, date.
- Average Product Cost:
Measure: Avg Cost = AVERAGE('Product Lookup'[current_cost])[^1]
Use: Margin planning across product lines.
- Highest Retail Price:
Measure: Max Price = MAX('Product Lookup'[current_retail_price])[^1]
Use: Identify premium product price ceiling.
- Unique Customers:
Measure: Distinct Customers = DISTINCTCOUNT('Customer Lookup'[customer_id])[^1]
Use: Customer base growth tracking.
- Total Employees:
Measure: Total Employees = COUNTROWS('Employee Lookup')[^1]
Use: Workforce KPI by location or role.

Iterator example vs. “sugar”:

- Total Sales (units): SUM('Sales by Store'[quantity_sold]) is interpreted like SUMX over that column.[^1]
- When to prefer SUMX: When the aggregation requires row-wise expression (e.g., price × quantity) before summing.

Example:
Net Sales = SUMX('Sales by Store', 'Sales by Store'[quantity_sold] * RELATED('Product Lookup'[current_retail_price]))

***

## 2) Rounding Functions

Purpose: Control numeric precision for display and calculation. Functions include INT, ROUND, ROUNDUP, ROUNDDOWN, TRUNC, MROUND, CEILING, FLOOR, FIXED, ISO.CEILING.[^1]

Highlights:

- ROUND(Number, Digits), ROUNDUP(Number, Digits), ROUNDDOWN(Number, Digits)
- INT(Number) returns integer truncated toward negative infinity[^1]
- TRUNC(Number, [Digits]) truncates decimal without rounding[^1]
- MROUND(Number, Multiple), CEILING(Number, Significance), FLOOR(Number, Significance) for aligning to increments like 0.05 or 0.25[^1]
- FIXED(Number, [Decimals], [NoCommas]) returns text with set decimals[^1]

Time rounding examples (from doc):

- MROUND(9:34:14, “0:15”) → 9:30:00
- FLOOR(9:34:14, “0:15”) → 9:30:00
- CEILING(9:34:14, “0:15”) → 9:45:00[^1]

Practical examples:

- Price Rounded to 0.25:
Price Rounded = MROUND('Product Lookup'[current_retail_price], 0.25)[^1]
Use: Shelf pricing alignment.
- Margin Percent Rounded:
Margin % Rounded = ROUND( ( 'Product Lookup'[current_retail_price] - 'Product Lookup'[current_cost] ) / 'Product Lookup'[current_retail_price], 4 )[^1]
Use: Standardize KPI precision.
- Whole-Dollar Price:
Whole Price = INT('Product Lookup'[current_retail_price])[^1]
Use: Cash handling and display simplification.
- Currency as Text:
Price Text = FIXED('Product Lookup'[current_retail_price], 2, TRUE)[^1]
Use: Concatenations or label outputs requiring text.

Customer age assignment (from doc’s exercise):

- Compute age precisely from birthdate, then apply ROUND or INT to ensure whole-number ages for reporting.[^1]

***

## 3) Information Functions

Purpose: Test data types or states and return TRUE/FALSE. Key functions: ISBLANK, ISERROR, ISLOGICAL, ISNONTEXT, ISNUMBER, ISTEXT.[^1]

Uses:

- Data quality checks
- Defensive coding in measures
- Branching logic when combined with IF/COALESCE

Examples:

- Check for Missing Description:
Missing Desc? = ISBLANK('Product Lookup'[product_description])[^1]
- Validate Numeric ID:
Product ID Is Number = ISNUMBER('Product Lookup'[product_id])[^1]
- Guarding a Division:
Safe Markup = IF( ISERROR('Product Lookup'[current_retail_price] / 'Product Lookup'[current_cost]), BLANK(), 'Product Lookup'[current_retail_price] / 'Product Lookup'[current_cost] )[^1]

Tip: Replace IF(ISBLANK(x), y, x) with COALESCE(x, y) for clearer, optimized code (see Logical Functions).[^1]

***

## 4) Conversion Functions

Purpose: Convert or format data types. Functions include CURRENCY, INT, FORMAT, DATE, TIME, DATEVALUE, VALUE.[^1]

Key notes:

- CURRENCY(Value) evaluates and returns a currency data type, helpful for consistent financial aggregations and formatting in measures[^1]
- FORMAT(Value, Format) returns text; use cautiously when the result must remain numeric[^1]
- DATE(Year, Month, Day), TIME(Hour, Minute, Second), DATEVALUE(TextDate), VALUE(TextNumber)[^1]

Practical examples:

- Force Costs to Currency:
Cost (Currency) = CURRENCY(SUM('Sales by Store'[cost]))[^1]
Use: Enforce currency type without just changing visual formatting.
- ISO Date Column in Calendar:
Calendar[Date ISO] = FORMAT('Calendar'[Date], "yyyy-mm-dd")[^1]
Use: Consistent key formatting or display per the assignment in the document.
- Convert Text to Number:
Clean Quantity = VALUE('Sales by Store'[quantity_sold_text])[^1]
- Create Product Code:
Product Code = "CAT-" \& FORMAT('Product Lookup'[product_id], "000")[^1]

Assignment references from doc:

- Add Calendar column with “yyyy-mm-dd”[^1]
- Ensure [Cost] is always a currency via a measure using CURRENCY/FORMAT[^1]

***

## 5) Logical Functions

Purpose: Branching logic using conditions. Functions: IF, AND, OR, NOT, TRUE, FALSE, SWITCH, COALESCE.[^1]

Best practices:

- Use operators \&\& and || when checking more than two conditions (cleaner and faster) [^1]
- SWITCH(TRUE(), …) pattern to replace deeply nested IFs[^1]
- COALESCE(Expression1, Expression2, …) returns first non-blank, a cleaner alternative to IF + ISBLANK and engine-optimizable[^1]

Examples:

- Price Tier:
Price Tier = IF('Product Lookup'[current_retail_price] > 20, "Premium", IF('Product Lookup'[current_retail_price] > 10, "Standard", "Budget"))[^1]
- Beverage Class:
Is Beverage = IF( OR( CONTAINSSTRING('Product Lookup'[product_category], "Coffee"), CONTAINSSTRING('Product Lookup'[product_category], "Tea") ), TRUE(), FALSE() )[^1]
- Quarter-Year Label with SWITCH(TRUE()):
Calendar[Qtr-Year] =
VAR q = 'Calendar'[Quarter]
VAR y = 'Calendar'[Year]
RETURN
SWITCH(TRUE(),
q = 1, "Q1-" \& y,
q = 2, "Q2-" \& y,
q = 3, "Q3-" \& y,
q = 4, "Q4-" \& y,
BLANK()
)[^1]
- Fill Blanks with Defaults:
Units (Zero if Blank) = COALESCE( SUM('Sales by Store'[quantity_sold]), 0 )[^1]
- Text placeholder:
Yesterday Revenue Label = COALESCE([Yesterday Customer Revenue], "-")[^1]

***

## 6) Ranking with RANKX

While RANKX is a table iterator function, it returns scalar ranks for each context and is commonly used alongside scalar expressions. It ranks an expression evaluated over a table.

Examples:

- Rank Products by Price:
Price Rank = RANKX(ALL('Product Lookup'), 'Product Lookup'[current_retail_price], , DESC)
Use: Identify premium items for merchandising.
- Rank by Margin:
Margin Rank =
RANKX(
ALL('Product Lookup'),
'Product Lookup'[current_retail_price] - 'Product Lookup'[current_cost],
,
DESC
)
Use: Prioritize profitable items for promotion.

Category-specific ranks (only coffees):

- Coffee Price Rank =
RANKX(
FILTER(ALL('Product Lookup'), CONTAINSSTRING('Product Lookup'[product_category], "Coffee")),
'Product Lookup'[current_retail_price],
,
DESC
)

Note: RANKX requires a table to iterate over and a scalar expression to rank. Always ensure consistent filter context (use ALL/ALLEXCEPT appropriately).

***

## Putting It Together: Patterns and Ideas to Use

- Price and Margin KPIs:
    - Total Revenue = SUMX('Sales by Store', 'Sales by Store'[quantity_sold] * RELATED('Product Lookup'[current_retail_price]))[^1]
    - Margin % Rounded = ROUND( ([Revenue] - [COGS]) / [Revenue], 4 )
- Standardized Presentation:
    - Currency-enforced measures using CURRENCY for consistent type[^1]
    - ISO date strings using FORMAT for alignment with external systems[^1]
- Data Quality:
    - ISBLANK/ISNUMBER checks to gate calculations and show fallbacks[^1]
    - COALESCE to replace blank outputs with zeros or placeholders[^1]
- Time-Bucketing and Labels:
    - SWITCH(TRUE()) for compact, maintainable label logic (e.g., Q1-2020, “Premium/Standard/Budget”)[^1]
- Engine Considerations:
    - Prefer COUNTROWS(VALUES(Column)) over DISTINCTCOUNT for very large models if performance dictates[^1]
    - Understand “syntax sugar” to know when to move from SUM to SUMX (when per-row expression is required)[^1]
- Retail Use Cases:
    - Shelf price harmonization with MROUND/CEILING/FLOOR[^1]
    - Rounding for customer age or loyalty tiers (INT/ROUND)[^1]
    - Ranking for top-N product lists in visuals (RANKX with ALL and category filters)

***

## Quick Reference by Category

- Aggregate: SUM, AVERAGE, MAX, MIN, COUNT, COUNTA, DISTINCTCOUNT, COUNTROWS, and X-iterators[^1]
- Rounding: INT, ROUND, ROUNDUP, ROUNDDOWN, TRUNC, MROUND, CEILING, FLOOR, FIXED, ISO.CEILING[^1]
- Information: ISBLANK, ISERROR, ISLOGICAL, ISNONTEXT, ISNUMBER, ISTEXT[^1]
- Conversion: CURRENCY, INT, FORMAT, DATE, TIME, DATEVALUE, VALUE[^1]
- Logical: IF, AND, OR, NOT, TRUE/FALSE, SWITCH, COALESCE[^1]
- Ranking: RANKX (over a table with a scalar expression)

These functions, applied with proper filter context and model relationships, enable powerful, maintainable DAX for real business intelligence scenarios in the coffee retail dataset.

<div align="center">⁂</div>

[^1]: Advanced-DAX-for-Business-Intelligence-56-70.pdf

