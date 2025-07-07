# SQL & R Business Analytics Project – Walmart Sales Data

##  Author

**Gerasimos Chatzopoulos**

---

##  Project Overview

This personal project simulates a real-world business analytics workflow using **SQL** for data exploration and **R** for basic statistical modeling.  
The dataset, based on a fictional retail chain named *WalmartSales*, was sourced from Kaggle and is used to mirror the typical tasks a data analyst might perform when answering business-oriented questions about:

- Sales performance  
- Customer behavior  
- Branch & product profitability  
- Transaction-level insights  

The project moves from **data cleaning** ➝ **SQL queries** ➝ **R modeling** ➝ **interpreting results**, much like a full analytics pipeline.

---

The project addresses a wide range of client-style questions, grouped by difficulty:

###  Simple
- What is the most selling product line?
- What is the most common payment method?
- Which city generates the highest revenue?
- What is the average rating per product line?
- Which gender buys the most?
- And more...

###  Medium
- Top 3 best-selling product lines per branch
- MoM sales growth rate
- Rank branches by sales within cities
- Compare average unit price by gender

###  Little Challenging
- Calculate Customer Lifetime Value (CLV)
- Build a Sales Performance Index (SPI) using weighted KPIs
- Determine profit margins per product line

---

##  Data Cleaning Steps

- Filled missing values using `COALESCE`
- Removed duplicates by `invoice_id`
- Extracted weekday and month from the date using SQLite date functions
- Converted textual columns and numeric types appropriately

---

##  Tools Used

| Tool        | Purpose                                 |
|-------------|-----------------------------------------|
| **SQL**     | Data extraction & transformation        |
| **SQLite**  | SQL engine used for this project        |
| **R**       | Basic statistical modeling (GLM)        |
| **ggplot2** | (Optional) Data visualization (R)       |
| **Kaggle**  | Source of the sales dataset             |

---

##  R Modeling

- Used a Generalized Linear Model (GLM) to assess the impact of `Customer.type` on `Total` sales
- Performed log transformation due to non-normality (confirmed with Shapiro test)
- Compared model AIC before and after transformation
- Interpreted statistical significance and practical impact
