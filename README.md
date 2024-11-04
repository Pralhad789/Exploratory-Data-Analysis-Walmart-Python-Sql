# Exploratory-Data-Analysis-Walmart-Python-Sql

## Business Overview
Walmart, as one of the world’s leading retail giants, faces an ongoing challenge to understand and respond effectively to customer needs and market trends. As a business that operates across various cities and branches, Walmart’s ability to manage inventory, optimize staffing, and refine marketing strategies is essential to maintain profitability and customer satisfaction.

This project aimed to empower Walmart with deeper insights into their sales data by analyzing transaction patterns, customer preferences, and revenue trends. Through exploratory data analysis approach using Python and SQL, the project’s goal is to provide Walmart with actionable information to enhance operational efficiency and make data-driven decisions.

## Project Overview
The company has access to a wealth of transactional data across its branches and cities but needed a streamlined approach to address critical business questions:

* Which branches and product categories generate the highest revenue?
* What payment methods do customers prefer, and how do preferences vary across locations?
* When do stores experience peak shopping hours?
* Which product categories show the highest and lowest profit margins?
* Are there any branches with notable year-over-year revenue growth or decline?

Answering these questions would enable the company to concentrate on high-profit product lines, optimize inventory management, adjust staffing schedules effectively, and implement targeted, location-specific promotions.

## Dataset
The project utilizes the Walmart Sales Dataset from Kaggle, which includes 10,000 records of transaction data across multiple branches, categories, and cities. Key characteristics of the data are the following:

* Columns: Contains fields such as date, branch, city, category, payment_method, rating, and quantity_sold.
* Timeframe: Covers daily transaction data for an extended period, enabling year-over-year analysis.
* Characteristics: Contains ratings and transactional details that allow for customer sentiment analysis and sales pattern identification.

Dataset Link : https://www.kaggle.com/datasets/najir0123/walmart-10k-sales-datasets?resource=download

## Data Cleaning and Transformation using Python
Python and Pandas were integral to the data cleaning, processing, and transformation stages of this project. Leveraging Python’s flexibility and Pandas’ robust data manipulation capabilities, we ensured the dataset was accurate, consistent, and ready for in-depth analysis.

Key steps included:

* Duplicate Removal: Identified and removed duplicate entries to maintain data integrity and avoid skewed results.
* Missing Value Handling: Addressed missing values by removing them where appropriate.
* Data Type Standardization: Ensured consistent data types across the dataset, converting dates to datetime and prices to floats to enable precise calculations.
* Currency Standardization: Cleaned and formatted currency fields for accurate financial analysis.
* Feature Engineering: Created a Total Amount column by calculating unit price × quantity sold, adding a valuable metric for further analysis.

These transformations produced a clean, structured dataset, setting a strong foundation for detailed exploration and insight generation.

## SQL Analysis
After cleaning, the data was loaded into MySQL database, where SQL was used to answer complex business questions. Key SQL queries were structured to address:

1. Identify Branches with Highest Revenue Increase Year-Over-Year
* Question: Which branches experienced the largest increase in revenue compared to the previous year?
* Purpose: Detecting branches with rising revenue highlights successful areas, providing insights that can be used to replicate success in other branches and support continued growth.

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
    ROUND(((r2023.revenue - r2022.revenue) / r2022.revenue) * 100, 2) AS revenue_increase_ratio
FROM revenue_2022 AS r2022
JOIN revenue_2023 AS r2023 ON r2022.branch = r2023.branch
WHERE r2023.revenue > r2022.revenue
ORDER BY revenue_increase_ratio DESC
LIMIT 5;
```

2. Identify Branches with Highest Revenue Decline Year-Over-Year
* Question: Which branches experienced the largest decrease in revenue compared to the previous year?
* Purpose: Detecting branches with declining revenue is crucial for understanding possible local issues and creating strategies to boost sales or mitigate losses.

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

3. Analyze Sales Shifts Throughout the Day
* Question: How many transactions occur in each shift (Morning, Afternoon, Evening) across branches?
* Purpose: This insight helps in managing staff shifts and stock replenishment schedules, especially during high-sales periods.

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

4. Determine the Most Common Payment Method per Branch
* Question: What is the most frequently used payment method in each branch?
* Purpose: This information aids in understanding branch-specific payment preferences, potentially allowing branches to streamline their payment processing systems.

```sql
WITH cte AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank_payment
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT branch, payment_method AS preferred_payment_method
FROM cte
WHERE rank_payment = 1;
```

5. Determine the Busiest Day for Each Branch
* Question: What is the busiest day of the week for each branch based on transaction volume?
* Purpose: This insight helps in optimizing staffing and inventory management to accommodate peak days.

```sql
with cte as(
    SELECT 
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank_transactions
    FROM walmart
    GROUP BY branch, day_name
)

select branch, day_name, no_transactions
from cte
WHERE rank_transactions = 1;
```

6. Analyze Payment Methods and Sales
* Question: What are the different payment methods, and how many transactions and items were sold with each method?
* Purpose: This helps understand customer preferences for payment methods, aiding in payment optimization strategies.

```sql
SELECT 
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```

7. Identify the Highest-Rated Category in Each Branch
* Question: Which category received the highest average rating in each branch?
* Purpose: This allows Walmart to recognize and promote popular categories in specific branches, enhancing customer satisfaction and branch-specific marketing.

```sql
with cte as(
    SELECT 
        branch,
        category,
        AVG(rating) AS avg_rating,
        RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rank_rating
    FROM walmart
    GROUP BY branch, category
) 

select branch, category, avg_rating
from cte
where rank_rating = 1;
```

8. Calculate Total Quantity Sold by Payment Method
* Question: How many items were sold through each payment method?
* Purpose: This helps Walmart track sales volume by payment type, providing insights into customer purchasing habits.

```sql
SELECT 
    payment_method,
    SUM(quantity) AS total_qty_sold
FROM walmart
GROUP BY payment_method;
```

9. Analyze Category Ratings by City
* Question: What are the average, minimum, and maximum ratings for each category in each city?
* Purpose: This data can guide city-level promotions, allowing Walmart to address regional preferences and improve customer experiences.

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

10. Calculate Total Profit by Category
* Question: What is the total profit for each category, ranked from highest to lowest?
* Purpose: Identifying high-profit categories helps focus efforts on expanding these products or managing pricing strategies effectively.

```sql
SELECT 
    category,
    SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```
