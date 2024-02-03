# CHATZOPOULOS GERASIMOS

# PROJECT SUMMARY

This SQL project offers an in-depth exploration of sales data for a fictional retail chain named "WalmartSales" that is not in Greece. It's crafted to mirror the challenges and inquiries faced by data analysts in a typical business environment, handling a spectrum of questions that might be raised by clients regarding their sales functions. The essence of this project lies in its simulation of real-world data analysis tasks, encompassing everything from routine data queries to complex problem-solving scenarios.

By delving into this project, we showcase a proficient application of SQL for robust data querying and manipulation, embodying the kind of day-to-day requests analysts often encounter, such as identifying sales trends, evaluating branch performance, and understanding customer demographics. These tasks are fundamental to a business's ability to make informed decisions, optimize operations, and tailor strategies to meet market demands.

Furthermore, the project transcends basic data handling, venturing into the realm of statistical modeling with R. This extension is crucial for businesses looking to forecast future trends, segment customers based on purchasing behavior, and assess the impact of different variables on sales outcomes. Such analyses are pivotal in transforming raw data into strategic insights, enabling businesses to anticipate market changes and act proactively.

In addition to SQL and R, the project leverages modern visualization tools to present data findings compellingly. The use of platforms like Tableau, Power BI, or R's ggplot2 for creating dynamic dashboards and charts illustrates how data visualization serves as a bridge between complex data analysis and strategic decision-making. Through effective visualization, we translate intricate data patterns into clear, actionable insights, ensuring that stakeholders can easily digest and act upon the information presented.

In essence, this project embodies the quintessential tasks of a data analyst in today's business landscape. It highlights the analyst's role in navigating through data to answer critical business questions, applying statistical models to uncover deeper insights, and employing visualization techniques to communicate findings effectively. This comprehensive approach not only demonstrates the analyst's technical prowess but also their ability to drive business intelligence and support strategic decisions. Through this project, we illustrate the significant impact that data analysis has on shaping business strategies and achieving operational excellence within the retail sector.

# DATA

The data saw taken from kaggle and it was choose at random.


# Client hypothetical questions



SIMPLE QUESTIONS

1. In which city is each branch?
2. How many unique product lines does the data have?
3. What is the most common payment method?
4. What is the most selling product line? 
5. What product line had the largest revenue?
6. What is the city with the largest revenue? 
7. Fetch each product line and add a column to those product line showing "xceding", "lucking". xceding if its greater than average sales
8. Which branch sold more products than average product sold?
9. What is the most common product line by gender?
10. What is the average rating of each product line?
11. Number of sales made in each time of the day per weekday
12. Which of the customer types brings the most revenue?
13. How many unique customer types does the data have?
14. Which customer type buys the most?
15. What is the gender of most of the customers?
16. What is the gender distribution per branch?
17. Which day fo the week has the best avg ratings?
18. Which day of the week has the best average ratings per branch?


MEDIUM DIFFICULTY 

1. Which product line has the highest average sales per transaction, and how does it compare to the overall average?
2. Rank the branches based on their total sales and display their rank within their city.
3. What is the percentage contribution of each product line to the total sales in each city?
5. Identify the top 3 selling products in each branch based on quantity sold.
6. Calculate the month-over-month growth rate in total sales for each month.
7. Compare the average unit price of products sold to male vs. female customers.
8. Find the day of the week with the highest number of transactions.

CHALLENGING

1. Estimate the Customer Lifetime Value (CLV) for each customer type, based on the gross income generated from their purchases. Assume an average customer lifespan of 5 years.

2. Develop a Sales Performance Index (SPI) for each branch, based on their sales volume, average transaction value, and customer satisfaction rating. This index should give a weighted value to each aspect, with sales volume weighted at 40%, average transaction value at 30%, and customer satisfaction rating at 30%.

3. Determine the profit margin for each product line, defined as the gross income divided by total sales, expressed as a percentage. This analysis will help identify which product lines are the most profitable.





# Cleaning


--We can check for null values and repalce with no info character

UPDATE sales SET branch = COALESCE(branch, 'No Info') WHERE branch IS NULL;
UPDATE sales SET city = COALESCE(city, 'No Info') WHERE city IS NULL;
UPDATE sales SET customer_type = COALESCE(customer_type, 'No Info') WHERE customer_type IS NULL;
UPDATE sales SET gender = COALESCE(gender, 'No Info') WHERE gender IS NULL;
UPDATE sales SET product_line = COALESCE(product_line, 'No Info') WHERE product_line IS NULL;
--etc for all collumns
-- textual or VARCHAR columns                   

UPDATE sales SET unit_price = COALESCE(unit_price, 0) WHERE unit_price IS NULL;
UPDATE sales SET quantity = COALESCE(quantity, 0) WHERE quantity IS NULL;
UPDATE sales SET tax_pct = COALESCE(tax_pct, 0) WHERE tax_pct IS NULL;
UPDATE sales SET total = COALESCE(total, 0) WHERE total IS NULL;
UPDATE sales SET cogs = COALESCE(cogs, 0) WHERE cogs IS NULL;
UPDATE sales SET gross_margin_pct = COALESCE(gross_margin_pct, 0) WHERE gross_margin_pct IS NULL;
UPDATE sales SET gross_income = COALESCE(gross_income, 0) WHERE gross_income IS NULL;
UPDATE sales SET rating = COALESCE(rating, 0) WHERE rating IS NULL;


--Check for dublicate row values and remove them if there are any


DELETE FROM sales
WHERE rowid NOT IN (
  SELECT MIN(rowid)
  FROM sales
  GROUP BY invoice_id
)
AND invoice_id IN (
  SELECT invoice_id
  FROM sales
  GROUP BY invoice_id
  HAVING COUNT(*) > 1
);



--Based on the above questions we should split the date into day of the week and month to extract some useful insights to the time period of sales

ALTER TABLE sales ADD COLUMN day_name VARCHAR(10);
UPDATE sales
SET day_name = CASE 
    WHEN strftime('%w', date) = '0' THEN 'Sunday'
    WHEN strftime('%w', date) = '1' THEN 'Monday'
    WHEN strftime('%w', date) = '2' THEN 'Tuesday'
    WHEN strftime('%w', date) = '3' THEN 'Wednesday'
    WHEN strftime('%w', date) = '4' THEN 'Thursday'
    WHEN strftime('%w', date) = '5' THEN 'Friday'
    WHEN strftime('%w', date) = '6' THEN 'Saturday'
END;

ALTER TABLE sales ADD COLUMN month_name VARCHAR(10);
UPDATE sales
SET month_name = CASE
    WHEN strftime('%m', date) = '01' THEN 'January'
    WHEN strftime('%m', date) = '02' THEN 'February'
    WHEN strftime('%m', date) = '03' THEN 'March'
    WHEN strftime('%m', date) = '04' THEN 'April'
    WHEN strftime('%m', date) = '05' THEN 'May'
    WHEN strftime('%m', date) = '06' THEN 'June'
    WHEN strftime('%m', date) = '07' THEN 'July'
    WHEN strftime('%m', date) = '08' THEN 'August'
    WHEN strftime('%m', date) = '09' THEN 'September'
    WHEN strftime('%m', date) = '10' THEN 'October'
    WHEN strftime('%m', date) = '11' THEN 'November'
    WHEN strftime('%m', date) = '12' THEN 'December'
END;;

--with mysql there are commands like dayname that is quicker, im using sql lite in this example


# SIMPLE QUESTIONS ANSWERS


-- In which city is each branch?

SELECT 
	DISTINCT city,
    branch
FROM sales;

-- How many unique product lines does the data have?

SELECT
	DISTINCT product_line
FROM sales;

-- What is the most common payment method?

SELECT payment, COUNT(*) AS frequency
FROM sales
GROUP BY payment
ORDER BY frequency DESC
LIMIT 1;


-- What is the most selling product line? 

SELECT
	SUM(quantity) as qty,
    product_line
FROM sales
GROUP BY product_line
ORDER BY qty DESC;


-- What product line had the largest revenue?

SELECT
	product_line,
	SUM(total) as total_revenue
FROM sales
GROUP BY product_line
ORDER BY total_revenue DESC;


-- What is the city with the largest revenue? 

SELECT
	branch,
	city,
	SUM(total) AS total_revenue
FROM sales
GROUP BY city, branch 
ORDER BY total_revenue;

 
-- Fetch each product line and add a column to those product line showing "exceding", "lucking". exceding if its greater than average sales


SELECT 
	AVG(quantity) AS average_col
FROM sales;

SELECT
	product_line,
	CASE
		WHEN AVG(quantity) > 6 THEN "exceding"
        ELSE "lucking"
    END AS remark
FROM sales
GROUP BY product_line;



-- Which branch sold more products than average product sold?


SELECT 
	branch, 
    SUM(quantity) AS qnty
FROM sales
GROUP BY branch
HAVING SUM(quantity) > (SELECT AVG(quantity) FROM sales); 

         
-- What is the most common product line by gender?    

SELECT
	gender,
    product_line,
    COUNT(gender) AS total_cnt
FROM sales
GROUP BY gender, product_line
ORDER BY total_cnt DESC;


-- What is the average rating of each product line?

SELECT
	ROUND(AVG(rating), 2) as avg_rating,
    product_line
FROM sales
GROUP BY product_line
ORDER BY avg_rating DESC;



-- Which customer type buys the most?

SELECT
	customer_type,
    COUNT(*)
FROM sales
GROUP BY customer_type;


-- What is the gender of most of the customers?

SELECT
	gender,
	COUNT(*) as gender_cnt
FROM sales
GROUP BY gender
ORDER BY gender_cnt DESC;
                        

-- Which day of the week has the best average ratings per branch?   

SELECT 
	day_name,
	COUNT(day_name) total_sales
FROM sales
WHERE branch = "C"
GROUP BY day_name
ORDER BY total_sales DESC;


# LITTLE MORE CHALLENGING

-- Which product line has the highest average sales per transaction, and how does it compare to the overall average?
SELECT 
    product_line,
    AVG(total) AS avg_sales_per_transaction,
    (SELECT AVG(total) FROM sales) AS overall_avg_sales
FROM sales
GROUP BY product_line
ORDER BY avg_sales_per_transaction DESC;

-- Rank the branches based on their total sales and display their rank within their city.
SELECT 
    city,
    branch,
    SUM(total) AS total_sales,
    RANK() OVER (PARTITION BY city ORDER BY SUM(total) DESC) AS sales_rank_within_city
FROM sales
GROUP BY city, branch
ORDER BY city, total_sales DESC;

-- What is the percentage contribution of each product line to the total sales in each city?
SELECT 
    city,
    product_line,
    SUM(total) AS product_line_sales,
    (SUM(total) / SUM(SUM(total)) OVER (PARTITION BY city)) * 100 AS percentage_contribution
FROM sales
GROUP BY city, product_line
ORDER BY city, percentage_contribution DESC;

-- Identify the top 3 selling products in each branch based on quantity sold.
WITH RankedProducts AS (
    SELECT
        branch,
        product_line,
        SUM(quantity) AS total_quantity,
        DENSE_RANK() OVER (PARTITION BY branch ORDER BY SUM(quantity) DESC) AS rank
    FROM sales
    GROUP BY branch, product_line
)
SELECT 
    branch,
    product_line,
    total_quantity
FROM RankedProducts
WHERE rank <= 3
ORDER BY branch, rank;

-- Calculate the month-over-month growth rate in total sales for each month.
SELECT 
    month_name,
    SUM(total) AS total_sales,
    (SUM(total) - LAG(SUM(total), 1) OVER (ORDER BY MIN(date))) / LAG(SUM(total), 1) OVER (ORDER BY MIN(date)) * 100 AS month_over_month_growth
FROM sales
GROUP BY month_name
ORDER BY MIN(date);


-- Compare the average unit price of products sold to male vs. female customers.
SELECT 
    gender,
    AVG(unit_price) AS average_unit_price
FROM sales
GROUP BY gender;

-- Find the day of the week with the highest number of transactions.
SELECT 
    day_name,
    COUNT(*) AS total_transactions
FROM sales
GROUP BY day_name
ORDER BY total_transactions DESC
LIMIT 1;

# CHALLENGING

--Estimate the Customer Lifetime Value (CLV) for each customer type, based on the gross income generated from their purchases. Assume an -average customer lifespan of 5 years.

WITH CustomerPurchases AS (
    SELECT
        customer_type,
        SUM(gross_income) AS total_gross_income
    FROM sales
    GROUP BY customer_type
),
CustomerCounts AS (
    SELECT
        customer_type,
        COUNT(DISTINCT invoice_id) AS total_purchases
    FROM sales
    GROUP BY customer_type
),
CLV AS (
    SELECT
        a.customer_type,
        a.total_gross_income,
        b.total_purchases,
        (a.total_gross_income / b.total_purchases) * 12 * 5 AS estimated_clv -- Assuming 12 purchases a year over 5 years
    FROM CustomerPurchases a
    JOIN CustomerCounts b ON a.customer_type = b.customer_type
)
SELECT *
FROM CLV;



--Develop a Sales Performance Index (SPI) for each branch, based on their sales volume, average transaction value, and customer satisfaction --rating. This index should give a weighted value to each aspect, with sales volume weighted at 40%, average transaction value at 30%, and --customer satisfaction rating at 30%.


WITH SalesMetrics AS (
    SELECT
        branch,
        SUM(quantity) AS total_volume,
        AVG(total) AS avg_transaction_value,
        AVG(rating) AS avg_rating
    FROM sales
    GROUP BY branch
),
SPI AS (
    SELECT
        branch,
        (total_volume / SUM(total_volume) OVER () * 0.4) +
        (avg_transaction_value / MAX(avg_transaction_value) OVER () * 0.3) +
        (avg_rating / MAX(avg_rating) OVER () * 0.3) AS sales_performance_index
    FROM SalesMetrics
)
SELECT *
FROM SPI
ORDER BY sales_performance_index DESC;


--Determine the profit margin for each product line, defined as the gross income divided by total sales, expressed as a percentage. This --analysis will help identify which product lines are the most profitable.

SELECT
    product_line,
    SUM(gross_income) AS total_gross_income,
    SUM(total) AS total_sales,
    (SUM(gross_income) / SUM(total)) * 100 AS profit_margin_percentage
FROM sales
GROUP BY product_line
ORDER BY profit_margin_percentage DESC;


# THIS IS THE END OF OUR SQL QUARIES

# R
# Load data from a CSV file into a data frame
my_data <- read.csv(file = "C:/Users/forta/OneDrive/Desktop/project/sales.csv", header = TRUE, sep = ",")
 
# View the first few rows of the data frame
head(my_data)

# Histogram of the total sales amount
hist(my_data$Total, main = "Histogram of Total Sales", xlab = "Total Sales", col = "lightblue")

# Q-Q plot for total sales amount
qqnorm(my_data$Total)
qqline(my_data$Total, col = "red")



glm_model <- glm(Total ~ Customer.type, data = my_data, family = gaussian())

# Summary of the model to see coefficients and significance
summary(glm_model)


shapiro.test(my_data$Total)

# NOT NORMALITY SO WE CAN TREY TO LOG P<A

# Log-transforming the total sales amount
my_data$log_total <- log(my_data$Total + 1)  # Adding 1 to avoid log(0)

# Fitting a GLM to the log-transformed total
glm_model_log <- glm(log_total ~ Customer.type, data = my_data, family = gaussian())

# Summary of the model with the log-transformed total
summary(glm_model_log)
---------
Call:
glm(formula = log_total ~ Customer.type, family = gaussian(), 
    data = my_data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.9871  -0.6027   0.1010   0.7322   1.5286  

Coefficients:
                    Estimate Std. Error t value Pr(>|t|)    
(Intercept)          5.44482    0.04110  132.49   <2e-16 ***
Customer.typeNormal -0.02615    0.05818   -0.45    0.653    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for gaussian family taken to be 0.8461879)

    Null deviance: 844.67  on 999  degrees of freedom
Residual deviance: 844.50  on 998  degrees of freedom
AIC: 2674.9

Number of Fisher Scoring iterations: 2
---------

## EXPLANATION 


(Intercept) Estimate: The estimated log total sales for the baseline group of the Customer.type variable, which is likely 'Member' if 'Normal' is mentioned as another level. The intercept is significant (p < 0.001), indicating that the log total sales for the baseline group is significantly different from zero.

Customer.typeNormal: The estimated difference in log total sales between the 'Normal' customer type and the baseline group. The estimate is -0.02615 with a p-value of 0.653, suggesting that there is no statistically significant difference in log total sales between 'Normal' customers and the baseline group at the usual alpha levels (e.g., 0.05).      


Null Deviance vs. Residual Deviance: The null deviance (844.67) represents the goodness of fit of a model with only the intercept, while the residual deviance (844.50) is for your model. The small difference between these suggests that adding Customer.type as a predictor does not greatly improve the model fit over the null model.

AIC: The Akaike Information Criterion is a measure of the relative quality of the statistical model for a given set of data. With a value of 2674.9, it can be used to compare this model against others with the same data; lower AIC values indicate a better fit.
AIC(glm_model_log)
[1] 2674.861
AIC(glm_model)
[1] 13852.22


Interpretation and Next Steps
Customer Type's Influence: The analysis suggests that customer type ('Normal' vs. the baseline) does not significantly impact the log-transformed total sales, given the high p-value. This might indicate that factors other than customer type have a more substantial effect on total sales.








