WALMART DATA ANALYSIS: END-TO-END SQL + PYTHON PROJECT (MySQL Only)
====================================================================

PROJECT OVERVIEW
----------------
This project demonstrates an end-to-end data analysis solution using Walmart's sales data.
It focuses on data cleaning, transformation using Python, and advanced business analysis 
using MySQL queries. The goal is to extract insights on sales trends, customer behavior, 
and product performance.

TECHNOLOGIES USED
-----------------
- Python 3.8+
- MySQL (local or hosted)
- Python Libraries: pandas, numpy, sqlalchemy, mysql-connector-python
- Kaggle API (for dataset download)

PROJECT STRUCTURE
-----------------
```plaintext
|-- data/                     # Raw data and transformed data
|-- sql_queries/              # SQL scripts for analysis and queries
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
```
---


STEPS TO RUN THIS PROJECT
--------------------------
1. Set up your environment:
   - Install Python 3.8+
   - Install and configure MySQL

2. Download the dataset:
   - Get Kaggle API token from your Kaggle account
   - Place kaggle.json in your .kaggle/ folder
   - Dataset Link: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets

3. Install dependencies:
   pip install -r requirements.txt

4. Run Python scripts:
   - Clean the dataset
   - Add calculated columns like total_amount
   - Load the cleaned data into MySQL

5. Perform analysis using SQL:
   - Use the scripts inside sql_queries/ to answer business questions such as:
     * Revenue trends by branch and category
     * Peak sales hours and customer behavior
     * Preferred payment methods
     * Top-performing products
       
## Problems Solved & SQL Queries

###  Q1: Payment Methods, Transactions, and Quantity Sold
```sql
SELECT 
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
RESULTS & INSIGHTS
```
Q2: Identify the highest-rated category in each branch display the branch, category, and avg rating
```sql
SELECT branch, category, avg_rating
FROM (
    SELECT 
        branch,
        category,
        AVG(rating) AS avg_rating,
        RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rnk
    FROM walmart
    GROUP BY branch, category
) AS ranked
WHERE rnk = 1;
```
Q3: Identify the busiest day for each branch based on the number of transactions
```sql
SELECT branch, day_name, no_transactions
FROM (
    SELECT 
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions,
        RANK() OVER(
            PARTITION BY branch 
            ORDER BY COUNT(*) DESC
        ) AS rnk
    FROM walmart
    GROUP BY branch, DAYNAME(STR_TO_DATE(date, '%d/%m/%Y'))
) AS ranked
WHERE rnk = 1;
```
Q4: Calculate the total quantity of items sold per payment method
```sql
SELECT 
	payment_method,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```
 Q5: Determine the average, minimum, and maximum rating of categories for each city
```sql
SELECT 
    city,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category;
```
Q6: Calculate the total profit for each category
```sql
SELECT 
    category,
    SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```
Q7: Determine the most common payment method for each branch
```sql
WITH cte AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT branch, payment_method AS preferred_payment_method
FROM cte
WHERE rnk = 1;
```
Q8: Categorize sales into Morning, Afternoon, and Evening shifts
```sql
SELECT
    branch,
    CASE 
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
```
Q9: Identify the 5 branches with the highest revenue decrease ratio from last year to current year (e.g., 2022 to 2023)
```sql
WITH revenue_2022 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2022
    GROUP BY branch
),
revenue_2023 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2023
    GROUP BY branch
)
SELECT 
    r2022.branch,
    r2022.revenue AS last_year_revenue,
    r2023.revenue AS current_year_revenue,
    ROUND(((r2022.revenue - r2023.revenue) / r2022.revenue) * 100, 2) AS revenue_decrease_ratio
FROM revenue_2022 AS r2022
JOIN revenue_2023 AS r2023 ON r2022.branch = r2023.branch
WHERE r2022.revenue > r2023.revenue
ORDER BY revenue_decrease_ratio DESC
LIMIT 5;
```

------------------
- Identified top-selling branches and product categories
- Analyzed sales trends across days and hours
- Observed customer payment preferences and peak shopping times
- Uncovered high-revenue product segments and cities

FUTURE ENHANCEMENTS
-------------------
- Build dashboards using Power BI or Tableau
- Automate ETL workflows using tools like Airflow
- Integrate additional data sources for deeper insights

CREDITS
-------
- Dataset: Kaggle (https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
- Developed by: Shreya Raj

LICENSE
-------
This project is licensed under the MIT License.


## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
- **Inspiration**: Walmart’s business case studies on sales and supply chain optimization.

---
