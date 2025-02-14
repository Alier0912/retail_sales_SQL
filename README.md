# Retail Sales Data Analysis with SQL

This repository contains SQL queries for analyzing retail sales data with the following schema:

*   `transaction_id`: Unique identifier for each transaction.
*   `customer_id`: Unique identifier for each customer.
*   `price_per_unit`: Price of each unit sold.
*   `amount`: Total amount of the transaction.
*   `product_category`: Category of the product sold.
*   `date`: Date of the transaction.
*   `age`: Age of the customer.
*   `gender`: Gender of the customer.

The SQL queries in this repository cover a range of analyses, from basic aggregations to more advanced techniques, and are designed to answer common business questions related to retail sales.  See below for a detailed list of questions and queries.

---
**Creating table**
```sql
CREATE TABLE retail_sales
(transaction_id	INT,
date	DATE,
customer_id	VARCHAR(100) PRIMARY KEY,
gender	VARCHAR(100),
age	INT,
p_category	VARCHAR(100),
quantity	INT,
price_per_unit	INT,
total_amount INT
);
```
*DATA DEBUGGING*

```sql
SELECT * FROM retail_sales;
```
```sql
SELECT * FROM retail_sales
 WHERE customer_id IS NULL
  OR age IS NULL
  OR p_category IS NULL
  OR quantity IS NULL
  OR price_per_unit IS NULL
  OR total_amount IS NULL;
```
*our data looks clean*

-------------------

## BUSINESS QUESTIONS

### Questions to Explore 
BENCHMARK QUERIES

*1. How does customer age and gender influence their purchasing behavior?*

*2. Are there discernible patterns in sales across different time periods?*

*3. Which product categories hold the highest appeal among customers?*

*4. What are the relationships between age, spending, and product preferences?*

*5. How do customers adapt their shopping habits during seasonal trends?*

*6. Are there distinct purchasing behaviors based on the number of items bought per transaction?*

*7. What insights can be gleaned from the distribution of product prices within each category?*

---

**BENCHMARK QUERIES**

*1. How does customer age and gender influence their purchasing behavior?*

```sql
WITH income_group AS
(
SELECT 
      CASE
          WHEN age BETWEEN 0 AND 20 THEN 'Gen Z'
          WHEN age BETWEEN 21 AND 40 THEN 'Millennials'
          WHEN age BETWEEN 41 AND 60 THEN 'Gen X'
          ELSE 'Boomers'
      END AS age_group,
      gender,
      SUM(total_amount) AS total_sales_amount,
      RANK() OVER (PARTITION BY gender ORDER BY SUM(total_amount) DESC) AS sales_rank
FROM retail_sales
GROUP BY 1,2
ORDER BY 3 DESC
) 

SELECT age_group, total_sales_amount, gender
FROM income_group
WHERE sales_rank BETWEEN 1 AND 5;
```

*2. Are there discernible patterns in sales across different time periods?*
```sql
WITH time_period AS
(
SELECT 
      EXTRACT(MONTH FROM date) AS month,
      SUM(total_amount) AS total_sales_amount,
      CASE WHEN       EXTRACT(MONTH FROM date) BETWEEN 3 AND 5 THEN 'Spring'
           WHEN       EXTRACT(MONTH FROM date) BETWEEN 6 AND 8 THEN 'Summer'
           WHEN       EXTRACT(MONTH FROM date) BETWEEN 9 AND 11 THEN 'Autumn'
           ELSE 'Winter'
      END AS season
FROM retail_sales
GROUP BY 1,3
ORDER BY 1 ASC,2 DESC
) 
SELECT 
     season,
     SUM(total_sales_amount) AS season
FROM time_period
GROUP BY 1
ORDER BY 2 DESC;
```

3. Which product categories hold the highest appeal among customers?
   
```sql
WITH highest_appeal AS
(
SELECT 
      p_category,
      SUM(total_amount) AS total_sales_amount,
      RANK() OVER (ORDER BY SUM(total_amount) DESC) AS sales_rank
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
) 
  SELECT 
         p_category,
         sales_rank
FROM highest_appeal
WHERE sales_rank BETWEEN 1 AND 5;
```
 4. What are the relationships between age, spending, and product preferences?
    
```sql
WITH age_spending AS
(
SELECT 
       age,
         p_category,
         SUM(total_amount) AS total_sales_amount,
            RANK() OVER (PARTITION BY p_category ORDER BY SUM(total_amount) DESC) AS sales_rank,
            CASE WHEN age BETWEEN 0 AND 20 THEN 'Gen Z'
                WHEN age BETWEEN 21 AND 40 THEN 'Millennials'
                WHEN age BETWEEN 41 AND 60 THEN 'Gen X'
                ELSE 'Boomers'
            END AS age_group
FROM retail_sales
GROUP BY 1,2
ORDER BY 3 DESC
) 
SELECT 
     DISTINCT age_group,
      p_category,
      SUM(total_sales_amount) AS total_sales_amount,
      sales_rank
FROM age_spending
WHERE sales_rank BETWEEN 1 AND 5
GROUP BY 1,2,4
ORDER BY 3 DESC;
```
5. How do customers adapt their shopping habits during seasonal trends?

```sql

WITH seasonal_trends AS
(
   SELECT 
           EXTRACT(MONTH FROM date) AS month,
              SUM(total_amount) AS total_sales_amount,
                CASE WHEN       EXTRACT(MONTH FROM date) BETWEEN 3 AND 5 THEN 'Spring'
                     WHEN       EXTRACT(MONTH FROM date) BETWEEN 6 AND 8 THEN 'Summer'
                     WHEN       EXTRACT(MONTH FROM date) BETWEEN 9 AND 11 THEN 'Autumn'
                     ELSE 'Winter'
                END AS season
    FROM retail_sales
    GROUP BY 1,3
    ORDER BY 2 DESC
) 
 SELECT 
       season,
       SUM(total_sales_amount) AS total_sales_amount
FROM seasonal_trends
GROUP BY 1
ORDER BY 2 DESC;
```
6. Are there distinct purchasing behaviors based on the number of items bought per transaction?
   
```sql
WITH purchase_behavior AS
(
 SELECT 
      DISTINCT p_category,
       ROUND(AVG(price_per_unit),2) AS avg_price_per_unit,
       SUM(quantity) AS total_quantity,
       SUM(transaction_id) AS total_transactions,
       SUM(total_amount) AS total_sales_amount,
       RANK() OVER (ORDER BY SUM(total_amount) DESC) AS sales_rank
FROM retail_sales
GROUP BY 1 
ORDER BY 5 DESC
)
SELECT
      p_category,
      total_transactions,
      sales_rank
FROM purchase_behavior
WHERE sales_rank BETWEEN 1 AND 5
GROUP BY 1,2,3
```

7. What insights can be gleaned from the distribution of product prices within each category?

```sql
WITH price_distribution AS
(
  SELECT 
         p_category,
         SUM(transaction_id) AS total_transactions,
        ROUND(AVG(price_per_unit),2) AS avg_price_per_unit,
         SUM(quantity) AS quantity_sold
    FROM retail_sales
    GROUP BY 1
    ORDER BY 2 DESC
)
 SELECT p_category, quantity_sold,avg_price_per_unit, total_transactions
FROM price_distribution
```

-----

**Easy SQL Queries**

1. What is the total sales amount for each product category?


  ``` sql 
  SELECT * FROM retail_sales
```

```sql
  SELECT 
         p_category,
            SUM(total_amount) AS total_sales_amount
    FROM retail_sales
    GROUP BY 1
    ORDER BY 2 DESC;
```

2. How many transactions were there in total?
   
```sql
SELECT COUNT(transaction_id) AS total_transactions
FROM retail_sales;
```

3. What is the average transaction amount?

```sql
SELECT 
      ROUND(AVG(total_amount),2) AS avg_transaction_amount
FROM retail_sales;
```

4. What is the total sales amount for each gender?

```sql
SELECT
     gender,
        SUM(total_amount) AS total_sales_amount
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```

5. How many customers are there in the dataset?

```sql
SELECT 
      COUNT(DISTINCT customer_id) AS total_customers
FROM retail_sales;
```

6. What are the unique product categories?

```sql
SELECT DISTINCT p_category
FROM retail_sales;
```

7. What is the minimum and maximum transaction amount?

```sql
SELECT 
      MIN(total_amount) AS min_transaction_amount,
      MAX(total_amount) AS max_transaction_amount
FROM retail_sales;
```

8. How many transactions occurred on each date?

```sql
SELECT 
      date,
      COUNT(transaction_id) AS total_transactions
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```
-----------

## Medium

9. What is the total sales amount for each product category, broken down by gender?

```sql
SELECT 
      p_category,
      gender,
      SUM(total_amount) AS total_sales_amount
FROM retail_sales
GROUP BY 1,2
ORDER BY 3 DESC;
```

10. What is the average transaction amount for each age group (e.g., create age ranges)?

```sql
SELECT 
      CASE
          WHEN age BETWEEN 0 AND 20 THEN 'Gen Z'
          WHEN age BETWEEN 21 AND 40 THEN 'Millennials'
          WHEN age BETWEEN 41 AND 60 THEN 'Gen X'
          ELSE 'Boomers'
      END AS age_group,
      ROUND(AVG(total_amount),2) AS avg_transaction_amount
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```

11. Which product category had the highest sales amount?
    
```sql
SELECT 
      p_category,
      SUM(total_amount) AS total_sales_amount
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

12. What is the total sales amount for each customer?

```sql
SELECT 
      customer_id,
      SUM(total_amount) AS total_sales_amount
FROM retail_sales
GROUP BY 1  
ORDER BY 2 DESC;
```

13. What is the average price per unit for each product category?

```sql
SELECT 
      p_category,
      ROUND(AVG(price_per_unit),2) AS avg_price_per_unit
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```

14. How many transactions were made by each customer?

```sql
SELECT 
       customer_id,
      SUM(transaction_id) AS total_transactions
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```

15. What are the top 10 customers by total sales amount?

```sql
SELECT 
      customer_id,
      SUM(total_amount) AS total_sales_amount
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```
---------

## Advanced

16. What is the rolling average of the total sales amount over time (monthly)?

```sql
SELECT 
       EXTRACT(MONTH FROM date) AS month,
      date,
      SUM(total_amount) AS total_sales_amount,
        AVG(SUM(total_amount)) OVER (ORDER BY EXTRACT(MONTH FROM date) ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS rolling_avg_sales
FROM retail_sales
GROUP BY 1,2
ORDER BY 3 DESC, 4 DESC;
```

17. What is the percentage contribution of each product category to the total sales amount?

```sql
SELECT 
      p_category,
      SUM(total_amount) AS total_sales_amount,
     ROUND(SUM(total_amount) / 456000 * 100,2) AS percentage_contribution
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC, 3 DESC;
```

18. For each product category, what is the average transaction amount for male vs. female customers?

```sql
SELECT 
      p_category,
      ROUND(AVG(total_amount),2) AS avg_transaction_amount,
      gender
FROM retail_sales
WHERE gender = 'Male'
GROUP BY 1,3
ORDER BY 2 DESC;
````
 *Repeat for Female*
 
 ```sql
 SELECT 
      p_category,
      ROUND(AVG(total_amount),2) AS avg_transaction_amount,
      gender
FROM retail_sales
WHERE gender = 'Female'
GROUP BY 1,3
ORDER BY 2 DESC;
```


19. Identify customers who have made more than 500 transactions (X being a parameter you can set)
  and calculate their average transaction amount.

```sql
WITH customer_transaction
AS
(
SELECT 
      customer_id,
      SUM(transaction_id) AS total_transactions,
      ROUND(AVG(total_amount),2) AS avg_transaction_amount,
    RANK() OVER (PARTITION BY ROUND(AVG(total_amount),2) ORDER BY SUM(transaction_id) DESC) AS transaction_rank
FROM retail_sales
GROUP BY 1
HAVING
     SUM(transaction_id)  > 500
)
SELECT 
      customer_id,
      total_transactions,
      avg_transaction_amount
FROM customer_transaction
WHERE transaction_rank  BETWEEN 1 AND 10
```



20. Create a query that identifies returning customers (customers who have made more than one purchase) 
   and calculates their total spending compared to new customers.
 Calculate the customer retention rate. (This will likely involve subqueries or CTEs

```sql
WITH customer_purchases AS
 (
    SELECT 
          customer_id,
          SUM(transaction_id) AS total_purchases,
          SUM(total_amount) AS total_spending
    FROM retail_sales
   GROUP BY 1
    HAVING SUM(transaction_id) > 1
)
SELECT 
      CASE
          WHEN total_purchases > 1 THEN 'Returning Customer'
          ELSE 'New Customer'
      END AS customer_type,
      COUNT(customer_id) AS total_customers,
      SUM(total_spending) AS total_spending
FROM customer_purchases
GROUP BY 1
ORDER BY 2 DESC, 3 DESC;
```

*BONUS QST. 
What is the average transaction amount for each product category, broken down*

```sql
SELECT 
      p_category,
      ROUND(AVG(total_amount),2) AS avg_transaction_amount
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC;
```

--------

#### RECOMMENDATION AND CONCLUSION
**Based on the analysis, we can make the following recommendations to the retail business:**

. Focus on marketing strategies that target Millennials and Gen X customers, as they have the highest total sales amount.
   
. Tailor product offerings and promotions based on seasonal trends, as sales vary significantly across different time periods.
   
. Invest in product categories that have the highest appeal among customers, such as Electronics and Apparel.
   
. Consider age group preferences when designing marketing campaigns and product offerings, as spending patterns vary across different 
   age groups.
 
. Adapt marketing strategies and product offerings to align with seasonal trends, as customers adjust their shopping habits based on
    the time of year.
   
. Analyze purchasing behaviors based on the number of items bought per transaction to identify opportunities for upselling and cross-selling.

. Monitor the distribution of product prices within each category to identify pricing strategies that maximize sales and profitability.

*Overall, by leveraging data analytics and insights from the retail sales data, the business can make informed decisions to drive growth and profitability.*

------

##### Further Analysis

1. **Customer Age and Gender Influence on Purchasing Behavior**:
    Millennials and Gen X customers have the highest total sales amount.
    Gender also plays a role, with distinct purchasing patterns observed between male and female customers.

2. **Patterns in Sales Across Different Time Periods**:
    Sales vary significantly across different seasons, with certain periods like Spring and Summer showing higher sales.

3. **Product Categories with Highest Appeal**:
    Certain product categories, such as Electronics and Apparel, have the highest total sales amount and appeal among customers.

4. **Relationships Between Age, Spending, and Product Preferences**:
    Different age groups have distinct spending patterns and product preferences.
    Millennials and Gen X customers tend to spend more on specific product categories.

5. **Customer Adaptation to Seasonal Trends**:
    Customers adjust their shopping habits based on the time of year, with noticeable changes in spending during different seasons.

6. **Purchasing Behaviors Based on Number of Items Bought per Transaction**:
    There are distinct purchasing behaviors based on the number of items bought per transaction, which can be leveraged for upselling and cross-selling opportunities.

7. **Distribution of Product Prices Within Each Category**:
    Monitoring the distribution of product prices within each category can help identify pricing strategies that maximize sales and profitability.

8. **Total Sales Amount for Each Product Category**:
    The total sales amount varies across different product categories, with some categories contributing more significantly to overall sales.

9. **Total Transactions**:
    The total number of transactions provides insight into customer engagement and sales volume.

10. **Average Transaction Amount**:
     The average transaction amount helps understand customer spending behavior and can guide pricing strategies.

11. **Total Sales Amount for Each Gender**:
     Gender-based analysis reveals differences in spending patterns, which can inform targeted marketing strategies.

12. **Number of Customers**:
     The total number of customers indicates the customer base size and potential market reach.

13. **Unique Product Categories**:
     Identifying unique product categories helps in understanding the product mix and inventory management.

14. **Minimum and Maximum Transaction Amount**:
     Analyzing the range of transaction amounts provides insights into customer spending limits and potential for high-value sales.

15. **Transactions on Each Date**:
     Understanding the number of transactions on each date helps identify peak sales periods and plan inventory accordingly.

16. **Sales Amount by Product Category and Gender**:
     Breaking down sales by product category and gender provides a detailed view of customer preferences and can guide product development and marketing.

17. **Average Transaction Amount by Age Group**:
     Age group analysis helps tailor marketing campaigns and product offerings to different customer segments.

18. **Highest Sales Amount by Product Category**:
     Identifying the product category with the highest sales amount helps prioritize inventory and marketing efforts.

19. **Total Sales Amount for Each Customer**:
     Analyzing total sales by customer helps identify high-value customers and develop loyalty programs.

20. **Average Price Per Unit for Each Product Category**:
     Understanding the average price per unit helps in pricing strategy and competitive analysis.

--------
*Analyst : Awal Alier*

*Reading between the data*
