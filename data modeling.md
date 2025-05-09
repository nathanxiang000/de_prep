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




