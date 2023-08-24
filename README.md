# Project Overview
In this portfolio, I demonstrate my expertise in data analysis by working with a sales dataset. The dataset contains comprehensive information about sales transactions, including product details, customer information, and transaction dates. The stakeholders' primary interest lies in gaining a deep understanding of the sales trends and performance throughout the year 2011. To meet this goal, I employ various analytical techniques, create informative visualizations, and address specific questions that are of utmost importance to the stakeholders.

# Stakeholder-Driven Questions

To address the stakeholders' inquiries, I focused on the following key questions:

**1. What were the average daily sales for each month in the year 2011, and how did the month-over-month percentage change in average daily sales fluctuate?**

```By answering this question, I provide insights into the average daily sales trends for each month in 2011 and the relative change in sales compared to the previous month.```

&nbsp;

**2. How did the total sales per month unfold in 2011, and what were the month-over-month percentage variations in total sales?**

 ``` This analysis uncovers the monthly sales totals for the year 2011 and offers a clear perspective on the percentage change in total sales between consecutive months.```
 
&nbsp;

**3. What is the overall count of transactions for each month in 2011, shedding light on the transactional dynamics throughout the year?**

```This exploration presents the total transaction count for each month in 2011, providing an overview of the activity and engagement level over time.```

&nbsp;

**4. What is the total number of units sold for each month in 2011, and how did this metric evolve over the course of the year?**

```This analysis offers insights into the monthly variations in the total quantity of units sold, illuminating the demand patterns and fluctuations across the year.```

&nbsp;

**5. How did the average sales per transaction fare on a monthly basis in 2011, and what trends can be observed?**

```The analysis of average sales per transaction by month in 2011 provides insights into the values and trends of transactions.```

&nbsp;

**6. What is the average number of units per transaction for each month in 2011, and how did this metric change over time?**

```This analysis provides information about the average number of units purchased per transaction for each month in 2011, offering insights into customer buying behavior.```

&nbsp;

# Data Cleaning
Before diving into the data analysis, a thorough data cleaning process was undertaken to ensure the quality and consistency of the dataset. The following steps were performed to clean and prepare the data for analysis:

## Step 1: Initial Data Assessment
The initial step involved assessing the size of the dataset to understand its scale.

```sql
-- Count the total number of rows in the raw dataset
SELECT COUNT(*) Total_Count FROM sales_raw;
```

## Step 2: Removing Duplicate Entries
To ensure accuracy in the analysis, duplicate entries were eliminated by creating a new table containing only unique values.

```sql
-- Create new table for unique values
CREATE TABLE sales_unique SELECT DISTINCT * FROM sales_raw;
```
```sql
-- Count the total number of rows in the unique dataset
SELECT COUNT(*) Total_Count FROM sales_unique;
```

## Step 3: Handling Missing Values
Missing values were identified and addressed in the Description column.

```sql
-- Count NULL values in the Description column
SELECT COUNT(*) AS null_values
FROM sales_unique 
WHERE Description IS NULL;
```
```sql
-- Count empty string values in the Description column
SELECT COUNT(*) AS empty_values
FROM sales_unique 
WHERE Description = '';
```

```sql -- Update empty strings in the Description column to NULL
UPDATE sales_unique 
SET Description = NULL
WHERE Description LIKE '';
```

```sql
-- Update NULL values in the Description column to 'Unknown'
UPDATE sales_unique 
SET Description = 'Unknown'
WHERE Description IS NULL;
```

## Step 4: Removing Irrelevant Data
Entries with NULL values in the CustomerID column were removed, as they were not needed for the analysis.

```sql
-- Delete rows with NULL values in the CustomerID column
DELETE FROM sales_unique 
WHERE CustomerID IS NULL;
```

## Step 5: Handling Negative Values
Negative values in the Quantity column, which typically indicate cancelled purchases, were removed as they were not required for analysis.

```sql
-- Check for negative values in the Quantity column
SELECT *
FROM sales_unique
WHERE Quantity < 0;
```

```sql
-- Delete rows with negative values in the Quantity column
DELETE FROM sales_unique 
WHERE Quantity < 0;
```

## Step 6: Additional Checks
The UnitPrice column was examined to ensure there were no negative values.

```sql
-- Check for negative values in the UnitPrice column
SELECT * 
FROM sales_unique
WHERE UnitPrice < 0;
```
## Step 7: Filtering Data for Analysis Year
Entries from years prior to 2011 were removed to align with the stakeholders' requirement to focus solely on the analysis of the year 2011.

```sql
-- Delete entries before 2011 to narrow analysis scope
DELETE FROM sales_unique 
WHERE YEAR(InvoiceDate) < 2011;
```
Following these data cleaning steps, the dataset was prepared and optimized for in-depth analysis. The cleaned dataset was then used to derive valuable insights into the sales trends and performance during the year 2011.

# Data Analysis

By employing various analytical techniques and SQL queries, I address the above questions and derive meaningful insights. The analysis encompasses a wide range of metrics, visualizations, and tables to present the findings effectively and comprehensively. These insights empower stakeholders to make informed decisions and strategic plans that capitalize on the trends and patterns revealed by the data.

I  will answer the Stakeholder-Driven Questions through SQL.

**1. What were the average daily sales for each month in the year 2011, and how did the month-over-month percentage change in average daily sales fluctuate?**

```sql
WITH daily_sales AS (
  SELECT
      DATE_FORMAT(InvoiceDate, '%Y-%m-%d') AS date
    , SUM(quantity * unitprice) AS total_sales
  FROM sales_unique
  WHERE YEAR(InvoiceDate) = 2011
  GROUP BY DATE_FORMAT(InvoiceDate, '%Y-%m-%d')
),
monthly_avg_daily_sales AS (
  SELECT
      MONTH(date) AS month
    , ROUND(AVG(total_sales), 2) AS avg_daily_sales
    , ROUND(LAG(AVG(total_sales), 1) OVER (ORDER BY MONTH(date)), 2) AS previous_avg_daily_sales
  FROM daily_sales
  GROUP BY MONTH(date)
)
SELECT
    month
  , DATE_FORMAT(CONCAT('2011-', month, '-01'), '%b') AS month_name
  , avg_daily_sales
  , COALESCE(previous_avg_daily_sales, 0) AS previous_avg_daily_sales
  , COALESCE(ROUND((avg_daily_sales - previous_avg_daily_sales) / previous_avg_daily_sales * 100, 2), 0) AS mom_percent_growth
FROM monthly_avg_daily_sales
ORDER BY month;
```
| month | month_name | avg_daily_sales | previous_avg_daily_sales | mom_percent_growth |
|-------|------------|-----------------|--------------------------|--------------------|
| 1     | Jan        | 23670.89        | 0                        | 0                  |
| 2     | Feb        | 18586.87        | 23670.89                 | -21.48             |
| 3     | Mar        | 22003.03        | 18586.87                 | 18.38              |
| 4     | Apr        | 22303.54        | 22003.03                 | 1.37               |
| 5     | May        | 27094.21        | 22303.54                 | 21.48              |
| 6     | Jun        | 25386.39        | 27094.21                 | -6.3               |
| 7     | Jul        | 23037.03        | 25386.39                 | -9.25              |
| 8     | Aug        | 24771.19        | 23037.03                 | 7.53               |
| 9     | Sep        | 36565.01        | 24771.19                 | 47.61              |
| 10    | Oct        | 39832.4         | 36565.01                 | 8.94               |
| 11    | Nov        | 44469.45        | 39832.4                  | 11.64              |
| 12    | Dec        | 64648.8         | 44469.45                 | 45.38              |



Visualizations
The analysis is enriched with visual representations such as line charts and tables, which facilitate the communication of insights to stakeholders. These visualizations enable a clear and concise understanding of the sales trends and performance metrics for the year 2011.

Conclusion
Through this comprehensive data analysis, I provide stakeholders with valuable insights into the sales performance of the year 2011. The answers to the stakeholders' questions shed light on various facets of sales trends, enabling data-driven decision-making and strategic planning. These insights hold the potential to enhance sales strategies and optimize business performance in the future.
