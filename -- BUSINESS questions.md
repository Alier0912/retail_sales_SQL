-- BUSINESS questions
Easy:

--What is the total sales amount for each product category?

 SELECT * FROM retail_sales;
How many transactions were there in total?
What is the average transaction amount?
What is the total sales amount for each gender?
How many customers are there in the dataset?
What are the unique product categories?
What is the minimum and maximum transaction amount?
How many transactions occurred on each date?
Medium:

What is the total sales amount for each product category, broken down by gender?
What is the average transaction amount for each age group (e.g., create age ranges)?
Which product category had the highest sales amount?
What is the total sales amount for each customer?
What is the average price per unit for each product category?
How many transactions were made by each customer?
What are the top 10 customers by total sales amount?
Advanced:

What is the rolling average of the total sales amount over time (e.g., daily or weekly)? (This requires window functions)
What is the percentage contribution of each product category to the total sales amount?
For each product category, what is the average transaction amount for male vs. female customers?
Identify customers who have made more than X transactions (X being a parameter you can set) and calculate their average transaction amount.
Create a query that identifies returning customers (customers who have made more than one purchase) and calculates their total spending compared to new customers. Calculate the customer retention rate. (This will likely involve subqueries or CTEs).