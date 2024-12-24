# Sales and Customer Analysis
### Overview
This project was designed to demonstrate my proficiency with PostgreSQL and to gain expertise with Docker. I used PostgreSQL to uncover insights and Apache Superset to create dashboards that provide actionable insights into sales trends, customer behavior, and product performance.

The dataset consisted of transactions from an e-commerce store based in the UK, specializing in unique, all-occasion gifts. Many of the store's customers are wholesalers.


### Tools and Skills Used
- **PostgreSQL**: Used to query and analyze data, uncovering key insights about sales and customer behavior.
- **Apache Superset**: Helped create interactive dashboards to visualize trends and make the data easier to understand.
- **Python**: Used for cleaning and preparing the data before diving into the analysis.

## Skills
- **Data Analysis**: Cohort analysis, retention trends, revenue tracking, and customer segmentation.
- **Data Storytelling**: Turning raw data into actionable insights that drive decision-making.
- **Visualization**: Building intuitive dashboards to effectively communicate findings.


# Customer Selection 
![customer-selection-2024-12-24T02-38-23 445Z](https://github.com/user-attachments/assets/1dfb6174-c8f3-4eeb-8a03-8e3dffa03bb0)


#### **Key Observations**

   
1. **Seasonal Spikes**:
   - The **November peak** suggests a **seasonal trend** (e.g., holiday shopping) that drives both customer engagement and total sales volume.

2. **Stable Months**:
   - **May to July** exhibit steady but lower levels of sales and customer counts, indicating **periods of lower demand**.

3. **Heavy Customer Retention**:
   - Even when unique customer counts drop in certain months like **February** and **December**, total items sold remains high, suggesting **repeat purchases by loyal customers**.

#### **Insights**
- **Leverage Seasonal Trends**:
  - Plan marketing campaigns and promotions in advance of **November** to capitalize on seasonal spikes in customer engagement and sales volume.

- **Stimulate Demand in Lower Periods**:
  - Use promotions or discounts during **May to July** to drive sales during traditionally lower-demand months.


### SQL Query: RFM Analysis

```sql
WITH rfm AS (
    SELECT
        "CustomerID",
        MAX("InvoiceDate") AS most_recent_purchase,
        COUNT("InvoiceNo") AS frequency,
        ROUND(SUM("Quantity" * "UnitPrice")::NUMERIC, 2) AS monetary_value
    FROM online_retail
    GROUP BY "CustomerID"
)
SELECT
    "CustomerID",
    DATE_PART('day', CURRENT_DATE - most_recent_purchase) AS recency,
    frequency,
    monetary_value,
    CASE
        WHEN DATE_PART('day', CURRENT_DATE - most_recent_purchase) <= 30 
             AND frequency >= 5 
             AND monetary_value > 500 THEN 'Loyal Customers'
        WHEN DATE_PART('day', CURRENT_DATE - most_recent_purchase) > 60 
             AND frequency <= 2 THEN 'At Risk'
        ELSE 'Casual Buyers'
    END AS rfm_category
FROM rfm
ORDER BY monetary_value DESC;

```




# Top Performing Products 
![top-performing-products-2024-12-24T02-46-14 618Z](https://github.com/user-attachments/assets/aa08b6ee-8398-4e03-8641-ad81e17a275d)



#### **Seasonal Trends**
- A noticeable spike occurs in **November**, suggesting a **seasonal sales boost**, likely due to holiday shopping.
- Some products like **JUMBO BAG RED RETROSPOT** and **POP ART HOLDER** show a significant increase during this time.

#### **Top Product Categories**
- A specific product category  **StockCode 22457** dominates in terms of both **revenue** and **sales volume**.

#### **Key Observations**
1. **Top Sellers**:
   - **JUMBO BAG RED RETROSPOT** and **ASSORTED COLOUR BIRD ORNAMENT** are clear top sellers in terms of quantity.
2. **High-Revenue Products**:
   - Some products, such as **REGENCY CAKESTAND 3 TIER**, generate disproportionately **high revenue** compared to sales volume.

#### **Insights for Action**
- **Increase marketing and stock availability** for top-performing products like **JUMBO BAG RED RETROSPOT** to maximize sales during peak seasons.
- **Review pricing strategies** for high-revenue, low-quantity products like **REGENCY CAKESTAND 3 TIER** to optimize profitability.

### SQL Query: Seasonal Product Trends
```sql

SELECT
    DATE_TRUNC('month', "InvoiceDate") AS month,
    "Description",
    SUM("Quantity") AS total_quantity_sold
FROM online_retail
GROUP BY month, "Description"
ORDER BY month, total_quantity_sold DESC;

```

#  Customer Analysis Section
![customer-analysis-section-2024-12-24T02-51-32 822Z](https://github.com/user-attachments/assets/086b47a8-6e18-48e9-9490-b099715d50bf)


 
- ### **2.1 Customer Analysis**

#### **Best Performing Cohorts**
- The **2010-12 cohort** retains customers well beyond 12 months, with retention stabilizing around **30–50%** in later months.

#### **Key Observations**
1. **Dominance of Low-Value Customers**:
   - The majority of customers fall into the **Low Value** segment.
2. **Medium and High-Value Customers**:
   - A small number of customers exist in the **High Value** segment, but they contribute significantly to revenue.

#### **Insights**
- Focus retention and upselling efforts on **Medium and High-Value customers** to maximize profitability.
- Develop strategies to move **Low Value customers** into higher segments by:
  - Offering **personalized promotions**.
  - Implementing **cross-selling** and **bundling** strategies.
 
```sql

WITH first_purchases AS (
    SELECT 
        "CustomerID",
        TO_CHAR(MIN("InvoiceDate"), 'YYYY-MM') as cohort_month,
        DATE_TRUNC('month', MIN("InvoiceDate")) as first_purchase_date
    FROM online_retail
    WHERE "CustomerID" IS NOT NULL
    GROUP BY "CustomerID"
),
customer_purchases AS (
    SELECT 
        f."CustomerID",
        f.cohort_month,
        TO_CHAR(o."InvoiceDate", 'YYYY-MM') as purchase_month,
        EXTRACT(YEAR FROM age(DATE_TRUNC('month', o."InvoiceDate"), f.first_purchase_date)) * 12 +
        EXTRACT(MONTH FROM age(DATE_TRUNC('month', o."InvoiceDate"), f.first_purchase_date)) as month_number
    FROM online_retail o
    JOIN first_purchases f ON o."CustomerID" = f."CustomerID"
),
cohort_size AS (
    SELECT 
        cohort_month,
        COUNT(DISTINCT "CustomerID") as num_customers
    FROM first_purchases
    GROUP BY cohort_month
)
SELECT 
    cp.cohort_month,
    cp.month_number,
    COUNT(DISTINCT cp."CustomerID") as num_customers,
    cs.num_customers as original_cohort_size,
    ROUND(100.0 * COUNT(DISTINCT cp."CustomerID") / cs.num_customers, 2) as retention_percentage
FROM customer_purchases cp
JOIN cohort_size cs ON cp.cohort_month = cs.cohort_month
WHERE cp.month_number <= 12
GROUP BY cp.cohort_month, cp.month_number, cs.num_customers
ORDER BY cp.cohort_month, cp.month_number;

```


# Sales and Revenue Trends 
![sales-and-revenue-trends-2024-12-24T03-00-32 976Z](https://github.com/user-attachments/assets/a4ba075d-276a-4bf5-8aa9-4bf1ca6cfb3d)


#### **3.1 Monthly Average Order Value (Line Chart)**
- **Observation**:
  - The average order value remains relatively stable throughout the year, hovering between **15 and 20 units**.
  - Slight increases occur in **March** and **July**, followed by a slight decline in **November**.
- **Insight**:
  - Despite seasonal revenue spikes, the average order value does not fluctuate significantly, indicating **consistent customer purchasing patterns**.
  - **Recommendation**: Target campaigns to increase order value through strategies like:
    - **Bundling**: Offer discounts for purchasing related items together.
    - **Upselling**: Encourage higher-value purchases during high-revenue months.

#### **3.2 Monthly Revenue (Line Chart)**
- **Observation**:
  - Revenue steadily increases from **February** to **October**, peaking in **November**.
  - A sharp drop in revenue occurs after November, likely due to the end of a seasonal sales period or holiday shopping.
- **Insight**:
  - **November** is a significant revenue driver, likely due to **seasonal demand** (e.g., holidays or promotions).
  - **Recommendation**: Plan inventory and marketing campaigns well in advance to capitalize on November’s peak revenue period.
 

```sql
SELECT
    DATE_TRUNC('month', "InvoiceDate") AS month,
     ROUND(AVG("Quantity" * "UnitPrice")::NUMERIC,2) AS average_order_value
FROM online_retail
GROUP BY month
ORDER BY month;
```

