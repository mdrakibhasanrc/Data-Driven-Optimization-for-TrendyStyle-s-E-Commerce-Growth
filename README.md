### Project Title: Data-Driven Optimization for TrendyStyle's E-Commerce Growth

### Project Overview:

Company: TrendyStyle

Industry: E-commerce (Fashion & Apparel)

### Project Focus: 
Optimizing the website’s conversion rate by analyzing customer behavior, identifying key drop-off points, and implementing data-driven solutions to enhance the user experience.

### Tools and Technologies:

        I) Google Analytics 4 (GA4): For tracking user behavior, events, and conversions across the website and app.

       II) BigQuery: To manage, query, and analyze large datasets collected from GA4 and other sources.

       III) Looker Studio: For creating visual reports and dashboards to monitor and analyze key performance indicators (KPIs).

       IV) Microsoft Clarity: To analyze user sessions, identify friction points, and optimize the website’s user interface (UI).

### Project Phases & Approach:

#### Phase 1: Data Collection & Setup ( Integrate GA4 & Bigquery ):

Configure GA4 to export data into BigQuery for deeper analysis. This allows you to query and analyze granular data, which is not always available directly in GA4.
Set up automated queries to pull in data such as product views, revenue, and user demographics.

#### Phase 2: Data Analysis & Insights:

Analyze GA4 data using BigQuery to identify key drop-off points in the user journey, such as high abandonment during checkout, and segment users by behavior, device, and location to find opportunities for optimization. Measure conversion rates at each funnel stage in BigQuery to pinpoint where users drop off and uncover potential barriers to conversion.

#### Overall Site Performance Important Metrics:

```sql
with flat_data as (
  SELECT  
     user_pseudo_id,
     sum(ecommerce.purchase_revenue) as total_revenue,
     countif(event_name='page_view') as page_view,
     countif(event_name='add_to_cart') as add_to_cart,
     countif(event_name='begin_checkout') as begin_checkout,
     countif(event_name='purchase') as purchase
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name in ('page_view','add_to_cart','begin_checkout','purchase')
group by user_pseudo_id
)

select
    count(distinct user_pseudo_id) as total_user,
    sum(page_view) as total_page_view,
    sum(total_revenue) as total_sales,
    round(safe_divide(sum(add_to_cart)-sum(purchase),sum(add_to_cart))*100,2) as cart_abandonment_rate,
    round(safe_divide(sum(begin_checkout)-sum(purchase),sum(begin_checkout))*100,2) as checkout_abandonment_rate,
    safe_divide(sum(total_revenue),sum(purchase)) as average_order_value,
    round(safe_divide(sum(purchase),count(distinct user_pseudo_id))*100,2) as conversion_rate
from flat_data;

```

Report:
![Screenshot_4](https://github.com/user-attachments/assets/e804542e-f42f-4473-8c48-886fae5e3dcb)


Summary Insights:

       I) High abandonment at both the cart (90.28%) and checkout (85.31%) stages points to potential friction in the purchase journey. Addressing issues like user experience, ease of 
          checkout, or offering incentives could reduce abandonment rates and improve conversions.

      II) Conversion rate (2.11%) is low, which might be expected given the high abandonment rates, but it presents a clear opportunity for optimization in the conversion process (e.g., 
         optimizing for mobile users, improving site speed, or offering incentives for completing purchases).

      III) The Average Order Value of $63.63 suggests that, while revenue is being generated, higher AOV could be achievable through strategies like cross-selling, up-selling, or 
            introducing higher-value products.

#### Device Wise Analysis:

##### ✅ Device Category Performance: Identify Conversion Rate Differences by Device:

This query provides insights into how users are converting across different devices and platforms (desktop, mobile, tablet), helping you determine if certain devices are underperforming.
```sql
SELECT  
    device.category AS device_category,
    COUNT(DISTINCT user_pseudo_id) AS total_users,
    COUNTIF(event_name = 'purchase') AS total_purchase,
    SUM(CAST(ecommerce.total_item_quantity AS INT64)) AS total_sold, 
    SUM(CAST(ecommerce.purchase_revenue AS FLOAT64)) AS total_revenue, 
    ROUND(SAFE_DIVIDE(COUNT(DISTINCT IF(event_name = 'purchase', user_pseudo_id, NULL)), COUNT(DISTINCT IF(event_name = 'page_view', user_pseudo_id, NULL)))*100,2) AS conversion_rate  -- Correct conversion rate calculation
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE event_name IN ('purchase', 'page_view')  -- Consider both purchase and page view events
GROUP BY device_category
ORDER BY conversion_rate DESC;
```
Report:

![Screenshot_1](https://github.com/user-attachments/assets/c9e0f9cf-5fc3-4184-954a-ff59d326bae9)

Summary & Insights: 

        I) Mobile users have a higher conversion rate (1.7) than desktop (1.6), suggesting better purchase likelihood on mobile.
 
        II) Desktop users contribute more revenue ($208,815) and purchases (13,182 items) than mobile ($146,768 and 9,200 items).

        II) Tablet users underperform with the lowest revenue ($6,582) and purchases (111), indicating a need for optimization.




##### ✅ Cart Abandonment rate and checkout abandonment rate by Device Category:

To improve Conversion Rate Optimization (CRO), especially around Average Order Value (AOV), Cart Abandonment, and Checkout Abandonment, understanding device-specific behavior is key. Optimizing these areas across different devices can significantly impact overall conversion rates. Below are important queries to identify potential problems in these areas using GA4 data in BigQuery.

```sql
with flat_data as (
  SELECT  
     device.category as device_category,
     count(distinct user_pseudo_id) as total_users,
     countif(event_name='add_to_cart') as add_to_cart,
     countif(event_name='begin_checkout') as begin_checkout,
     countif(event_name='purchase') as purchase,
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name in ('add_to_cart','begin_checkout','purchase')
group by device_category
)

select
   device_category,
   total_users,
   sum(add_to_cart) as total_add_to_cart,
   sum(begin_checkout) as total_begin_checkout,
   sum(purchase) as total_purchase,
   round(safe_divide(sum(add_to_cart)-sum(purchase),sum(add_to_cart))*100,2) as cart_abandonment_rate,
   round(safe_divide(sum(begin_checkout)-sum(purchase),sum(begin_checkout))*100,2) as checkout_abandonment_rate
from  flat_data
group by device_category,total_users;

```

Report:


![Screenshot_2](https://github.com/user-attachments/assets/1458ec8b-457c-4b56-93d8-63148a785916)


Summary & Insight: 

      I) Mobile vs. Desktop: Both have similar abandonment rates, but desktop users convert better, with more total purchases (3,226 vs. 2,355).

     II) Mobile vs. Tablet: Mobile has lower abandonment rates and higher purchases (2,355 vs. 111), while tablet users have the highest abandonment rates.

     III) Desktop vs. Tablet: Desktop users convert much better (3,226 vs. 111), with tablet users showing the highest abandonment rates and lowest purchases.




##### ✅ Average Order Value (AOV) by Device Category:

This query calculates the Average Order Value (AOV) for each device category. By understanding how much users spend on average across different devices, you can identify if certain devices are underperforming in terms of revenue generation.

```sql
with flat_data as (
  SELECT  
     device.category as device_category,
     count(distinct user_pseudo_id) as total_users,
     sum(ecommerce.purchase_revenue) as total_sales
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name ='purchase'
group by device_category
)

select
   device_category,
   total_users,
   total_sales,
   round(safe_divide(total_sales,total_users),2) as average_order_value
from  flat_data
group by device_category,total_users,total_sales;

```

Report:


![Screenshot_3](https://github.com/user-attachments/assets/faa7da23-adce-4d18-9862-3d76dd9f74e4)


Summary & Insight: 

       I) Desktop users have the highest total sales ($208,815) and AOV ($82.18), indicating larger purchases.

      II) Mobile users contribute significant sales ($146,768) with a slightly lower AOV ($79.29), suggesting smaller purchases.

      III) Tablet users have the lowest total sales ($6,582) and AOV ($67.86), indicating a need for optimization.




#### Phase 3: User Behaviour Data Analysis using Microsoft Clarity :

##### 1. Homepage:
   
Insight: Users engage most with the top banner and featured products, but many drop off quickly (45% bounce rate).

Data: Heatmaps show concentrated clicks on top navigation, banner images, and first few products, but users don’t scroll past the fold.

Recommendation: Optimize the first section with more engaging content and clear calls-to-action (CTAs) to reduce bounce rate. Consider adding trust signals (reviews, secure payment icons) higher up on the page.


##### 2. Category Page

Insight: Users are not engaging with filters and sorting options (only 12% interaction).

Data: Heatmaps reveal that users mostly scan products but ignore sorting/filter options, leading to frustration on large product lists.

Recommendation: Make filters more prominent and user-friendly. Offer quick product previews or better category navigation to encourage deeper exploration.


##### 3. Product Page:

Insight: Product images and descriptions are the primary engagement points, but only 5% of users click on reviews or ratings.

Data: Session recordings show users engaging with the product image gallery, then quickly exiting without interacting with reviews.

Recommendation: Make product reviews more visible and accessible. Add "Frequently Bought Together" or similar cross-sell recommendations to boost user interaction and confidence.


##### 4. Checkout Page:

Insight: High drop-off rate (55%) during the checkout process, especially at the payment stage.

Data: Session recordings show users hesitating when entering payment details or when unexpected shipping costs are revealed.

Recommendation: Simplify the checkout flow, highlight cost transparency earlier (shipping fees, taxes), and introduce trust signals like secure checkout badges.


##### 5. Thank You Page:

Insight: Low engagement on the Thank You page with minimal clicks to share on social or review the product.

Data: Heatmaps show a lack of interaction with social sharing buttons or post-purchase suggestions.

Recommendation: Add incentives for sharing (discounts for referrals) or encourage users to leave reviews. Offer recommendations for related products or loyalty program sign-ups to increase engagement.
