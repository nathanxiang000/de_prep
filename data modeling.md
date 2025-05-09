# Data Modeling Practices

## Practice: E-commerce Platform

**Requirements**

- Track daily sales totals by product category
- Analyze customer purchase history for segmentation
- Calculate average order value (AOV) and customer lifetime value (CLV)
- Monitor product inventory levels
- Evaluate the effectiveness of marketing campaigns


> in a Type 2 Slowly Changing Dimension (SCD), it is expected and acceptable that customer_id and product_id (the business or source keys) will not be unique per row. This is because each time an attribute changes (e.g., customer address, product category), a new row is inserted with the same business key but updated attribute values and new validity dates or flags.

> To tie each campaign to an order, the most common approach is to store a campaign_id directly in the order or order_line table. This is typically done by capturing a unique campaign identifier from the marketing source (such as an email, ad, or referral link) when the customer clicks through and places an order. This ID is then passed through the checkout process and saved with the order record. 

+---------------------+         +---------------------+         +----------------------+
|     customer        |         |      product        |         |      campaign        |
|---------------------|         |---------------------|         |----------------------|
| customer_id (PK)    |         | product_id (PK)     |         | campaign_id (PK)     |
| first_name          |         | product_title       |         | campaign_name        |
| last_name           |         | product_brand       |         | campaign_start_time  |
| email               |         | product_category_id |         | campaign_end_time    |
| phone               |         | unit_price          |         | campaign_description |
| city                |         | sku                 |         +----------------------+
| state               |         | created_at          |
| country             |         | updated_at          |
| customer_age        |         | is_current          |
| updated_at          |         +---------------------+
| is_current          |
+---------------------+
         |                                       
         |                                       
         |                                      
         |                                       
         |                                       
         v                                       
+-------------------------------------------------------------------------------------+
|                                   order_line                                        |
|-------------------------------------------------------------------------------------|
| line_item_id (PK)    | order_id         | order_time      | customer_id (FK)        |
| campaign_id (FK)     | store_id         | product_id (FK) | product_category_id (FK)|
| count                | amt              | discount        | unit_price_at_purchase  |
| tax_amt              | shipping_amt     | line_number     |                         |
+-------------------------------------------------------------------------------------+
         |
         |
         v
+---------------------+         +---------------------+
|  inventory_event    |         | inventory_snapshot  |
|---------------------|         |---------------------|
| inventory_event_id  |         | snapshot_date       |
| event_type          |         | store_id            |
| event_time          |         | product_id          |
| store_id            |         | quantity_on_hand    |
| product_id          |         +---------------------+
| quantity_change     |
| reference_order_id  |
+---------------------+

+---------------------+
| product_category    |
|---------------------|
| product_category_id |
| category_name       |
| parent_category_id  | (optional, for hierarchy)
+---------------------+


## Practice: Social Media Analytics

**Requirements**

- Track daily and monthly active users (DAU/MAU)
- Analyze post engagement (likes, comments, shares) by content type
- Monitor user growth and churn rates
- Measure ad performance (impressions, clicks, conversions)
- Analyze user behavior across different devices

+-------------------+        +-------------------+        +-------------------+
|      user         |        |     content       |        |    campaign       |
|-------------------|        |-------------------|        |-------------------|
| user_id (PK)      |        | content_id (PK)   |        | campaign_id (PK)  |
| status            |        | content_type      |        | campaign_type     |
| name              |        | content_name      |        | campaign_name     |
| email             |        | content_desc      |        | campaign_desc     |
| phone             |        | creator_id (FK)   |        | created_at        |
| created_at        |        | created_at        |        | starttime         |
| updated_at        |        | updated_at        |        | endtime           |
| is_current        |        | is_current        |        +-------------------+
| last_active_date  |        +-------------------+
| churned_date      |
+-------------------+
        |                         |
        |                         |
        v                         v
+-------------------+        +-------------------+
|  content_event    |        |    ad_event       |
|-------------------|        |-------------------|
| event_id (PK)     |        | event_id (PK)     |
| event_time        |        | event_time        |
| event_type        |        | event_type        |
| content_id (FK)   |        | campaign_id (FK)  |
| creator_id (FK)   |        | user_id (FK)      |
| viewer_id (FK)    |        | device_id (FK)    |
| device_id (FK)    |        | ip_address        |
| ip_address        |        | revenue           |
+-------------------+        | cost              |
                             +-------------------+
        |                         |
        |                         |
        v                         v
+-------------------+
|     device        |
|-------------------|
| device_id (PK)    |
| device_type       |
| os                |
+-------------------+


Facebook and Instagram track user churn using a combination of detailed event logging, user-level activity tables, and large-scale ELT (Extract, Load, Transform) pipelines that aggregate and analyze user activity at daily and monthly intervals.

### **How Facebook/Instagram Design Tables for Churn Stats**

- **Event-Level Logging:**  
  Every user interaction (logins, likes, posts, ad views, etc.) is logged in massive event tables, often partitioned by day for scalability. These tables typically include user IDs, event types, timestamps, and device information[5][6].

- **User Activity Aggregates:**  
  Nightly or hourly ELT jobs process these raw events to create aggregate tables:
  - **Daily Active Users (DAU):** Aggregated by distinct user IDs active per day.
  - **Monthly Active Users (MAU):** Aggregated by distinct user IDs active per month.
  - **Churn Candidates:** Users who were active in a prior period but not in the current period are flagged as churned.

- **Churn Table Example:**
  ```
  user_churn_stats
  ├── user_id
  ├── last_active_date
  ├── churned_flag
  ├── churned_date
  ├── resurrected_flag
  ├── period (day/month)
  ```

- **Resurrected Users:**  
  Facebook/Instagram also track "resurrected users"-those who were inactive in the prior period but return in the current one, as this provides insight into re-engagement effectiveness[6].

### **ELT for Aggregation**

- **Daily/Monthly Jobs:**  
  ELT pipelines run on distributed systems (e.g., Hive, Spark, Presto) to scan event logs, compute aggregates, and update summary tables for DAU, MAU, churn, and resurrection metrics[6].
- **Churn Calculation:**  
  Churn is typically defined as users who were active in the previous period (e.g., last month) but not in the current period. The calculation is automated and updated regularly to inform dashboards and machine learning models[7][8].

-- example churn calc
``` SQL
with month_user as (
select date_trunc('month',event_date) as month, user_id
from events
group by all 
order by 1,2
),
t2 as (
select current.month, current.user_id as active, next.user_id as churned
from month_user current
left join month_user next on current.user_id = next.user_id and current.month = next.month - interval '1 month'
order by 1,2
)
select month, sum(case when churned is null then 1 else 0 end)*100.0/count(*) as churned_rate
from t2
group by 1 
order by 1
```


## Practice: Video Streaming Service

**Requirements**
- Track viewing time and completion rates for content
- Analyze user preferences and viewing patterns
- Measure the effectiveness of content recommendations
- Monitor streaming quality and user experience metrics
- Analyze content performance by genre, creator, and release date

> In a data modeling interview for a video streaming service like Netflix, **the best practice is to model at the most granular (“event-level”) fact table possible**, rather than just at the aggregated level. Here’s why and how this applies, with reference to industry practice and your requirements:

---

### **Granularity: Raw Events vs. Aggregations**

#### **What Netflix Actually Does**

- **Netflix and similar companies collect raw, event-level data** for every user interaction: every play, pause, seek, quality change, recommendation shown, click, etc. This is stored in massive, partitioned event tables.
- **Aggregated “fact” tables** (like your Viewing_Fact) are then built via ELT/ETL jobs on top of these raw events for analytics and reporting.
- This approach allows for maximum flexibility: you can always aggregate up, but you can’t “drill down” if you only have pre-aggregated data.

### **What to Present in an Interview**

- **State that you would design the fact tables at the event level.**
    - For example: One row per viewing session or per playback event (e.g., play, pause, stop).
    - Include all relevant foreign keys and metrics (timestamps, user, content, device, etc.).
- **Explain that aggregates (like daily completion rates) are built from these raw events via ELT pipelines.**
    - This is exactly how Netflix and other large-scale streaming platforms operate.

---

## **Example: Event-Level Fact Table**

**Viewing_Event**
- event_id (PK)
- event_time
- user_id (FK)
- content_id (FK)
- device_id (FK)
- action_type (play, pause, stop, seek, etc.)
- position_seconds
- duration_seconds
- is_recommended (0/1)
- session_id

You can then build **Viewing_Fact** (e.g., daily user-content aggregates) by summarizing Viewing_Event.

Viewing_Fact (
    date_key (FK),
    time_key (FK),
    user_key (FK),
    content_key (FK),
    device_key (FK),
    viewing_duration,
    completion_percentage,
    is_recommended (0/1)
)

## **How to Explain in an Interview**

> “I would design the core fact tables at the event level, capturing every user interaction with content (play, pause, stop, etc.). From these raw events, I’d build aggregate tables-like daily viewing facts or completion rates-using scheduled ELT pipelines. This approach is scalable and matches how leading streaming platforms like Netflix operate, ensuring we can answer both current and future analytics questions.”

**Schema**

```
+---------------------+         +---------------------+         +---------------------+
|      user           |         |      device         |         |      date           |
|---------------------|         |---------------------|         |---------------------|
| user_id (PK)        |         | device_id (PK)      |         | date_key (PK)       |
| name                |         | device_type         |         | calendar_date       |
| email               |         | os                  |         | day_of_week         |
| signup_date         |         | app_version         |         | month               |
| ...                 |         | ...                 |         | year                |
+---------------------+         +---------------------+         +---------------------+

         |                               |                               |
         |                               |                               |
         v                               v                               v

+---------------------+         +---------------------+         +---------------------+
|      genre          |         |     creator         |         |      content        |
|---------------------|         |---------------------|         |---------------------|
| genre_id (PK)       |         | creator_id (PK)     |         | content_id (PK)     |
| genre_name          |         | creator_name        |         | title               |
| ...                 |         | ...                 |         | genre_id (FK)       |
+---------------------+         +---------------------+         | creator_id (FK)     |
                                                                | release_date        |
                                                                | content_type        |
                                                                | ...                 |
                                                                +---------------------+

         |                               |                               |
         |                               |                               |
         v                               v                               v

+--------------------------------------------------------------------------------------+
|                                viewing_event                                         |
|--------------------------------------------------------------------------------------|
| event_id (PK)           | date_key (FK)           | time_key (FK)                   |
| user_id (FK)            | content_id (FK)         | device_id (FK)                  |
| session_id              | event_time              | viewing_duration                |
| action_type (play/pause/stop/seek)                | position_seconds                |
| is_recommended (0/1)     | completion_percentage  | ...                             |
+--------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------+
|                             recommendation_event                                     |
|--------------------------------------------------------------------------------------|
| rec_event_id (PK)         | date_key (FK)           | user_id (FK)                   |
| content_id (FK)           | recommendation_algorithm| was_clicked (0/1)              |
| was_watched (0/1)         | rec_shown_time          | ...                            |
+--------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------+
|                             streaming_quality_event                                  |
|--------------------------------------------------------------------------------------|
| quality_event_id (PK)      | date_key (FK)           | time_key (FK)                  |
| user_id (FK)               | content_id (FK)         | device_id (FK)                 |
| buffering_instances        | average_bitrate         | stream_start_time              |
| stream_end_time            | quality_change_type     | ...                            |
+--------------------------------------------------------------------------------------+

+---------------------+
|     time            |
|---------------------|
| time_key (PK)       |
| hour                |
| minute              |
| second              |
+---------------------+
```

## Practice: Ride-sharing Platform

> Scenario: You're designing a data model for Uber to analyze ride performance, driver efficiency, and market dynamics.

**Requirements**

Track ride metrics (distance, duration, fare) by city and time
Analyze driver performance and earnings
Monitor supply-demand balance in real-time
Measure the effectiveness of surge pricing
Analyze user retention and frequency of use



