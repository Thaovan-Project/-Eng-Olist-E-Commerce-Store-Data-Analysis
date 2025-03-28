# Olist E-Commerce Store Data Analysis

## Introduction
This project analyzes the public dataset from Olist, a Brazilian e-commerce platform, to extract key insights into business performance, sales efficiency, customer behavior, seller performance, and delivery operations during the 2016–2018 period.

## About the Dataset
The raw dataset was publicly sourced from Kaggle and consists of the following 9 tables:
- olist_customers
- olist_geolocation
- olist_order_items
- olist_order_payments
- olist_order_reviews
- olist_orders
- olist_products
- olist_sellers
- product_category_name

Each table stores specific details about orders, including: order dates, product information, payments, delivery status, customer reviews, seller profiles, as well as customer behavioral and demographic data. This dataset enables end-to-end analysis of the entire customer journey—from order placement to product delivery and post-purchase feedback.

## Project Objectives
The primary objective of this project is to deliver actionable insights that optimize operational performance and drive sustainable growth for Olist by addressing key business questions across the following domains:

### Sales Performance & Revenue Analysis
1. What is Olist's total revenue? How does revenue change over time? Are there any periods with significant spikes or declines?
2. How many orders are placed on Olist each month? Are there any seasonal fluctuations or notable trends?
3. What are the most popular product categories? What are their sales volumes? Are there any underutilized product categories with high potential?
4. What is the average order value (AOV)? Are there any categories that generate exceptionally high order values?
5. What is the average order cancellation rate? Which product categories have the highest number of canceled orders?

### Seller Performance and Customer Behavior/Experience
1. How many active sellers are on Olist? How has this number changed over time?
2. What are the most used payment methods by customers? How does this vary by product category and geographic region?
3. How many customers make repeat purchases? What percentage of total revenue do they account for? Which product categories do they buy most frequently?
4. What is the current delivery status? What percentage of orders are delivered on time?
5. What is the average customer rating? Is there any relationship between review scores and delivery time?
6. Are there any product categories with notable review scores?

## Data Cleaning Using Power Query
- Changed column profiling mode from "Top 1000 rows" to "Entire dataset"
- Enabled "Column quality", "Column profile", and "Column distribution" features for comprehensive data quality assessment
- Promoted first row to headers for tables with incorrect formatting (Use First Row as Header)
- Verified and corrected data formats for each column
- Converted currency columns (product price, payment amount, etc.) to Fixed Decimal format
- Capitalized proper nouns
- Corrected misspellings (e.g., "são paulo" → "São Paulo") using Replace Value
- Split datetime columns into separate date columns for easier time-based analysis
- Merged "Product_category_name" with "Olist_products" to create a complete product table. Kept only relevant columns: Product_id and Product_category_name_english. Removed unnecessary columns (e.g., Product_name_length, etc.)
- Removed duplicate records. Eliminated redundant columns across all tables

## Data Modelling
The data was standardized and structured in a Star Schema model, consisting of:
- Main Fact Table: Olist_order_items contains detailed transactional data for each product in every order (the most granular fact table)
- Secondary Fact Table: Olist_orders stores detailed order information including order status, order timestamps, and actual/estimated delivery dates
- 6 Dimension Tables: Olist_customers, Olist_geolocation, Olist_order_payments, Olist_order_reviews, Olist_products, Olist_sellers
- Date Table: Created using:

```dax
Date = 
VAR FirstSalesDate = MIN(olist_orders_dataset[order_purchase_timestamp.1])
VAR YearFirstOrder = YEAR(FirstSalesDate)
VAR Dates = 
 FILTER(CALENDARAUTO(),YEAR([Date]) >= YearFirstOrder) RETURN
    ADDCOLUMNS(
        Dates,
        "Year", YEAR([Date]),
        "Month", FORMAT([Date], "mmm"),
        "Month Number", MONTH([Date]),
        "Quarter", FORMAT([Date], "\QQ"),
        "Day of Week", FORMAT([Date], "ddd"),
        "WeekDayNumber", WEEKDAY([Date]))
```

Table Relationships:

Olist_order_items connects to:
- Olist_orders via order_id
- Olist_sellers via seller_id
- Olist_products via product_id

Olist_orders connects to:
- Olist_customers via customer_id
- Olist_order_payments via order_id
- Olist_order_reviews via order_id
- DimDate via order_purchase_date

Olist_geolocation connects to Olist_customers via zip_code_prefix

## Data Analysis & Reporting

### Sales Performance & Revenue Overview
The following presents a comprehensive analysis of Olist's sales performance and revenue metrics during the 2016-2018 period:

![Ảnh chụp màn hình 2025-03-26 212300](https://github.com/user-attachments/assets/41e3273d-c31a-4770-a386-0baec3dad645)

#### 1. What is Olist's total revenue? How does revenue change over time? Are there any periods with significant spikes or declines?
Olist e-commerce platform's total revenue from 2016 to 2018 was **R$15,422,461.77**. This figure was calculated by summing the payment_value of orders with "delivered" status (since revenue is only recognized when orders are successfully delivered to customers), using the following DAX formula:

```dax
Total Revenue = 
    CALCULATE(
        SUM(olist_order_payments[payment_value]),
        olist_orders[order_status] = "delivered"
    )
```

Olist experienced steady annual revenue growth throughout this period, with the most dramatic surge occurring between 2016 and 2017.

![Ảnh chụp màn hình 2025-03-19 222734](https://github.com/user-attachments/assets/f2aad862-bf7b-46b2-8f6a-c17733ea957b)

The chart below shows revenue surged dramatically on November 24, 2017, peaking at **R$175K**, likely due to Black Friday or a major short-term promotion. Shortly after, revenue dropped sharply and stabilized around **R$50K - R$60K**, indicating that this was only a temporary surge rather than a breakthrough in market growth. However, this suggests that short-term promotions can effectively drive sales for a limited period, but additional strategies are needed to retain customers and ensure sustainable growth.

![Ảnh chụp màn hình 2025-03-18 192717](https://github.com/user-attachments/assets/f4704c02-1cb4-40a9-9945-9c571aec474a)

#### 2. How many orders are placed on Olist each month? Are there any seasonal fluctuations or notable trends?
During the 2016-2018 period, Olist processed a total of **99.44K** orders. Of these, **96K** orders were successfully placed and delivered to customers (defined as "Success Orders"), calculated using the following formula:

```dax
Success Order = 
CALCULATE(
    COUNT('olist_orders_dataset'[order_id]), 
    'olist_orders_dataset'[order_status] = "delivered"
)
```

![Ảnh chụp màn hình 2025-03-18 200051](https://github.com/user-attachments/assets/785420e7-f898-4fee-95fd-0520f6bfe8d4)

Overall, the number of orders follows the same trend as revenue, with no major changes.

#### 3. What are the most popular product categories? What are their sales volumes? Are there any underutilized product categories with high potential?
The chart shows that *bed_bath_table*, *health_beauty*, and *sports_leisure* have the highest number of orders, indicating strong customer demand. In contrast, categories like *watches_gifts* generate high revenue despite having fewer orders, suggesting that these products have high value but are purchased less frequently. On the other hand, the *telephony* category has a high order volume but low revenue, meaning the average order value is relatively low. This could be because most items in this category are lower-value accessories or replacement parts rather than high-priced products like smartphones. Meanwhile, categories at the bottom of the chart, such as *home_appliances*, have both low revenue and low order volume, raising questions about competition and actual customer demand.

![Ảnh chụp màn hình 2025-03-18 230134](https://github.com/user-attachments/assets/68bf9239-a7e6-48c4-ac2b-89e56474a954)

To optimize performance, Olist should focus on promoting high-demand categories while implementing upsell and cross-sell strategies for categories with high order value but low order volume. For categories with a large number of orders but suboptimal revenue, introducing product bundles or volume discounts could help increase the average order value. Lastly, Olist should reassess low-performing categories to identify potential issues and adjust sales strategies accordingly—or consider removing them from the catalog if necessary.

#### 4. What is the average order value (AOV)? Are there any categories that generate exceptionally high order values?
Olist's average order value is **R$159.85**, calculated using the following formula:

![Ảnh chụp màn hình 2025-03-18 225233](https://github.com/user-attachments/assets/08dd5e10-afb2-4e67-8267-28ad678c97ef)

```dax
Average Order Value = 
AVERAGEX(
    ALLSELECTED(olist_orders_dataset[order_id]), 
    [Total Revenue]
)
```

The chart shows that *computers* have the highest average order value **(R$1,300.8)**, nearly double that of the second-highest category (*small_appliances* – R$684.3). This indicates that computers are high-value items that contribute significantly to revenue, even if the number of orders is not very high. However, most other product categories have an average order value ranging between **R$200 - R$500**.


![Ảnh chụp màn hình 2025-03-18 225739](https://github.com/user-attachments/assets/2704741e-9a20-4566-867c-9f30a93133d6)

#### 5. What is the average order cancellation rate? Which product categories have the highest number of canceled orders?
A total of **625** orders were canceled between 2016 and 2018. The chart shows a steady increase in cancellations from 2016, peaking in Q3 2018 with **140** orders. However, in Q4 2018, cancellations dropped sharply to just **4** orders, suggesting possible changes in policies, processes, or a decline in market demand. The rising trend from Q1 2017 to Q3 2018 raises concerns about customer experience, product quality, or order fulfillment issues. To optimize, it’s essential to analyze the reasons behind the peak in Q3 2018 and identify the factors that led to the sharp decline in Q4 2018 to implement effective strategies moving forward.

![Ảnh chụp màn hình 2025-03-19 220307](https://github.com/user-attachments/assets/52dab1a9-6740-45bf-921a-5d63d7470ac6)

The chart shows that *sports_leisure* has the most canceled orders (**47 orders**), followed by *housewares* (**37 orders**) and *health_beauty* (**36 orders**). These categories likely have high demand but also a higher risk of cancellations, possibly due to customer changes of mind or issues with pricing and product quality. To improve, Olist should analyze the reasons behind these cancellations and adjust sales strategies, return policies, or quality control as needed.

![Ảnh chụp màn hình 2025-03-19 223836](https://github.com/user-attachments/assets/edf725ee-5aa1-4a22-a3c6-d157341d7a55)

### Seller Performance and Customer Behavior/Experience
#### 1. How many active sellers are on Olist? How has this number changed over time?
The data shows a total of **3,095** sellers, with strong growth from 2016 to 2018. The number of sellers steadily increased each quarter, from almost none in Q3 2016 to **1.7K** in Q2 2018, indicating that more sellers joined the platform over time. However, in Q3 2018, the number dropped slightly to **1.6K**, suggesting possible factors like rising competition, policy changes, or market saturation. To sustain growth, Olist should analyze the reasons behind this decline and implement strategies to attract and retain sellers more effectively.

<img width="546" alt="Ảnh màn hình 2025-03-19 lúc 09 42 26" src="https://github.com/user-attachments/assets/4c84f5d6-271b-41dd-8050-27d2f96edb29" />

#### 2. What are the most used payment methods by customers? How does this vary by product category and geographic region?

<img width="322" alt="Ảnh màn hình 2025-03-19 lúc 09 15 02" src="https://github.com/user-attachments/assets/603c170e-5b38-451b-8e76-3fa0dea830a4" />

The chart shows that *São Paulo* has the highest number of orders, making it a key market to focus on. *Credit cards* are the most popular payment method, especially in major cities like São Paulo and Rio de Janeiro, indicating a preference for flexible spending. To boost sales, businesses should consider offering promotions for credit card payments.

However, the *"boleto"* payment method (a traditional payment option) still holds a significant share, especially in São Paulo, suggesting that some customers prefer traditional payment methods. This could be due to a desire for better spending control or a lack of familiarity with digital payments. To keep this customer segment, businesses should consider making boleto payments easier while also promoting the benefits and security of digital payment options to gradually encourage adoption.

#### 3. How many customers make repeat purchases? What percentage of total revenue do they account for? Which product categories do they buy most frequently?
To calculate the Retention Rate, follow these steps:

Step 1: Identify customers who made more than one purchase

The Repeat_Purchases formula checks whether each customer (customer_unique_id) has placed more than one order. If they have, it returns 1; otherwise, it returns 0.

```dax
Repeat_Purchases = CALCULATE(
                      IF(COUNT('olist_orders_dataset'[order_id])>1,1,0),
                          ALLEXCEPT(olist_customers_dataset,olist_customers_dataset[customer_unique_id]))
```

Step 2: Count the Number of Repeat Customers

The following Repeat_Purchase_Customers formula counts the number of unique customers (customer_unique_id) who meet both of these conditions:
- Have at least one successfully delivered order (order_status = "delivered").
- Have a Repeat_Purchases value of 1 (meaning they have placed at least two orders).

``` dax
Repeat_Purchase_Customers = 
CALCULATE(
    DISTINCTCOUNT(olist_customers_dataset[customer_unique_id]),
    olist_orders_dataset[order_status] = "delivered",
    FILTER(
        VALUES(olist_customers_dataset[customer_unique_id]), 
        [Repeat_Purchases] = 1 
    )
)
```

Step 3: Calculate the Retention Rate

The Retention Rate is calculated by dividing the number of repeat customers by the total number of unique customers who have ever made a purchase (DISTINCTCOUNT(customer_unique_id)).
To prevent division errors, the formula returns 0 if the denominator is 0.

```dax
Retention Rate = 
DIVIDE(
    [Repeat_Purchase_Customers], 
    DISTINCTCOUNT(olist_orders_dataset[customer_id]), 
    0
)
```

<img width="380" alt="Ảnh màn hình 2025-03-19 lúc 09 14 26" src="https://github.com/user-attachments/assets/e59dcaaa-5cce-4952-be17-dfc4c47d983f" />

The data reveals that Olist has a total of **99.44K** customers, yet the retention rate is only **3.0%**, indicating that most customers make just one purchase and do not return.

Among product categories, *bed_bath_table*, *sports_leisure*, *health_beauty*, and *computers_accessories* have the highest number of returning customers, suggesting that these items may have high repurchase demand or provide a positive shopping experience that encourages repeat purchases.

However, retention rates in other categories are significantly lower. For instance, *books_general_interest and luggage_accessories* have only 6 returning customers, highlighting the need for strategies to enhance customer experience and encourage repeat purchases. Possible approaches might include loyalty programs, personalized promotions, and improved post-purchase services.

![Ảnh chụp màn hình 2025-03-19 225326](https://github.com/user-attachments/assets/b2a03016-2240-4b24-b4b4-79c688727d96)

The chart shows that the majority of revenue **(94.48%, or R$15.42M)** comes from first-time customers, while repeat customers—those who make more than one purchase—contribute only **5.52% (R$0.90M)**.

This shows that Olist depends more on bringing in new customers rather than keeping and getting more value from existing ones. With a low customer retention rate, it's important to find ways to encourage repeat purchases. Some effective strategies could be loyalty programs, personalized discounts, and better follow-up after purchases to keep customers coming back and boost long-term revenue.

#### 4. What is the current delivery status? What percentage of orders are delivered on time?

Here is an overview of Olist's delivery performance. The chart shows that the On-Time Delivery Rate is **93.4%**, meaning most orders are delivered on time, with only 6.6% experiencing delays. Additionally, the average delivery time (AVG Delivery Day) is **12.5 days**.

<img width="596" alt="Ảnh màn hình 2025-03-20 lúc 14 07 22" src="https://github.com/user-attachments/assets/e785568b-e024-4bbe-8b00-22a99ce24482" />

To calculate the On-Time Delivery Rate, follow these steps:

Step 1: Calculate the Actual Delivery Time

Create a column "Deliver Time" to determine the number of delivery days. This is done by calculating the days between the order purchase date (order_purchase_timestamp.1) and the actual delivery date (order_delivered_customer_date.1).

```dax
Deliver time = DATEDIFF(olist_orders_dataset[order_purchase_timestamp.1],olist_orders_dataset[order_delivered_customer_date.1],DAY)
```

Step 2: Classify Orders as On-Time or Late

Create a new column "Delivery_On_Time" to determine whether an order was delivered on time.
If the actual delivery date (order_delivered_customer_date.1) is earlier than or equal to the estimated delivery date (order_estimated_delivery_date.1), mark it as "On Time".
Otherwise, mark it as "Late".

```dax
Delivery_On_Time = 
IF(
    olist_orders_dataset[order_delivered_customer_date.1] <= olist_orders_dataset[order_estimated_delivery_date.1],
    "On Time", 
    "Late"
)
```

![Ảnh chụp màn hình 2025-03-19 221834](https://github.com/user-attachments/assets/ad27c041-ebf8-499c-a5c5-50c9cf70cd29)

Overall, the delivery system is performing quite efficiently, ensuring that most orders are delivered on time as promised. However, a small percentage (6.6%) of late deliveries could negatively impact the customer experience. It’s important to analyze the causes of these delays—whether they stem from logistics issues, shipping provider capacity, or inventory management—to identify areas for improvement and optimize the delivery process further.

#### 5. What is the average customer rating? Is there any relationship between review scores and delivery time?

<img width="499" alt="Ảnh màn hình 2025-03-21 lúc 14 37 08" src="https://github.com/user-attachments/assets/02136bf7-8979-4d6f-99c1-8c64ca45af64" />

The average rating is **4.09/5**, indicating that most customers have a positive experience with the products or services. While this is a relatively high score, there are still some lower ratings, suggesting that a portion of customers are not fully satisfied. A deeper analysis of the factors affecting ratings, such as product categories with lower scores or common negative feedback can help identify areas for improvement to enhance the overall customer experience.

![Ảnh chụp màn hình 2025-03-27 223807](https://github.com/user-attachments/assets/c947b807-7272-4326-9361-02328135ea9c)

The chart above illustrates the relationship between *average delivery time* and *customer ratings* from 2016 to 2018. Initially, delivery times were extremely long (**55 days in Q3 2016**) but significantly improved to **under 15 days from 2017 onward**, indicating that Olist made substantial progress in its shipping process.  

At the same time, *average ratings increased from 1.00 to over 4.00*, reflecting higher customer satisfaction as delivery times shortened. However, in Q4 2018, both metrics declined, with ratings dropping sharply to **2.25**, suggesting potential issues that may have negatively impacted the customer experience.

![Ảnh chụp màn hình 2025-03-19 221736](https://github.com/user-attachments/assets/9b74250f-1a30-4364-86a3-37cb0e7b19e6)

The charts above illustrate the relationship between delivery time, customer ratings, and order volume.

The left chart shows that shorter delivery times lead to higher ratings. Specifically, orders with a 1-star rating had an average delivery time of 21.25 days, whereas 5-star orders were delivered in just 10.62 days. This confirms that delivery speed has a direct impact on customer satisfaction.

The right chart shows that 5-star orders make up the majority, totaling 57K orders, while lower-rated orders, such as 1-star (11K orders) and 2-star (3K orders), are significantly lower. This suggests that good delivery service and a positive experience encourage more purchases, while poor service can result in negative reviews and fewer repeat customers.

Overall, Olist should focus on optimizing delivery speed to not only enhance customer experience but also drive more orders in the future.

#### 6. Are there any product categories with notable review scores?

![Ảnh chụp màn hình 2025-03-19 222027](https://github.com/user-attachments/assets/083671b4-2b2d-4c10-8ddf-cb3999c7c3cc)

The chart shows that the *security_and_services* category has the lowest average rating (**2.50**), indicating customer dissatisfaction, possibly due to slow seller response times, delayed deliveries, or unmet expectations. Meanwhile, other categories have average ratings ranging from **3.62 to 3.93**, suggesting relatively stable customer satisfaction, though there's room for improvement toward higher ratings.  

Notably, a "(Blank)" category has an average rating of **3.90**, likely due to misclassified data, which could impact report accuracy. To improve, it's essential to analyze why *security_and_services* received low ratings, gather customer feedback to enhance the buying experience, and review data gaps to ensure accurate reporting.

## Key Insights  

- **Total revenue is high and trending upward**, especially during **Black Friday or seasonal promotions**, indicating strong demand during peak periods.  
- Some categories with **high average order value (AOV)** significantly contribute to revenue—these could be the focus for upsell/ cross-sell strategies.  
- **Order cancellation rates are low** but have shown an upward trend during certain periods. High cancellation rates in some categories may be due to product quality or delivery time issues.  
- **Credit card and boleto (traditional method) are the dominant payment options**.  
- **Average delivery time is 12 days**, with **6.6% of orders delayed**. Faster delivery could significantly enhance customer experience and ratings.  
- **São Paulo is the key market**, leading in both revenue and order volume. Prioritizing promotions and logistics improvements in this region would be beneficial.

## Recommendations

- Grow revenue by focusing on high-performing categories (e.g., bed_bath_table, health_beauty, etc.) and expanding into high-revenue cities like São Paulo. Implement seasonal or quarterly discount programs to stimulate demand.
- Ensure product quality control and optimize pricing to reduce cancellation rates and enhance customer satisfaction.
- Optimize payment methods and encourage credit card transactions (the most popular option) through discounts or cashback incentives.
- Increase customer loyalty by implementing loyalty programs for returning customers to boost repeat purchases, particularly in areas with a strong customer base.

## Conclusion  
Olist is a high-revenue e-commerce platform with strong growth, especially during major promotions. However, there is still room for **optimization to increase profitability and expand the market** by improving **delivery times, customer experience, and product category strategies**.
