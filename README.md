<h1>Marketing Channels Analysis: Visitor Acquisition and Funnel Optimization </h1>
<br />
<h2>Description</h2>

<p><b>Objective:</b> To analyze visitor acquisition sources, identify key insights, and provide actionable recommendations to optimize marketing strategies and improve conversion performance.</p>

<p><b>Data Sources:</b></p>
<p>The analysis was conducted using the following datasets:</p>
<ul>
  <li><strong>projects_created.csv:</strong>
    <ul>
      <li><strong>PROJECT_ID</strong> – each business after registration is assigned a unique identifier called PROJECT_ID.</li>
      <li><strong>VISITOR_ID</strong> – a visitor’s unique id; it’s used to track visitors even before their signup.</li>
      <li><strong>PROJECT_REGISTRATION_DATETIME</strong> – the time a project was created.</li>
      <li><strong>PROJECT_PLATFORM</strong> – the platform the project was registered on (you can register a project directly on our website or, alternatively, use one of the platforms you create your websites on, e.g., WordPress or Shopify).</li>
      <li><strong>PROJECT_COUNTRY_ISO</strong> – a two-letter geocode based on a website’s location (ISO 3166-1 alpha-2).</li>
      <li><strong>PROJECT_INDUSTRY</strong> – the industry code chosen by users during project registration.</li>
      <li><strong>SUBSCRIPTION_CREATED_DATE</strong> – the date when the first subscription within a project was created.</li>
      <li><strong>FIRST_CONTRACT_VALUE</strong> – the value of a subscription (in USD).</li>
    </ul>
  </li>
  
  <li><strong>homepage_visits.csv:</strong>
    <ul>
      <li><strong>EVENT_DATETIME</strong> – datetime the homepage was visited.</li>
      <li><strong>VISITOR_ID</strong> – a visitor’s unique id.</li>
      <li><strong>ACQUISITION_CHANNEL</strong> – how the user entered our website and started the new session.</li>
    </ul>
  </li>

  <li><strong>costs.csv</strong> – monthly cost (in USD) generated by each acquisition channel.</li>
</ul>
<p>I’ve additionally used Big Query Public data on iso country codes.</p>

<h3>Scope & Methodology</h3>
<ul>
  <li><strong>Data range</strong> – October 2021</li>
  <li><strong>Methodology</strong> – In the absence of a multiple attribution model, I chose to attribute conversion to the last channel customer interacted with prior to conversion. As conversion, I considered paid subscription.</li>
</ul>

<h3>Tools Used</h3>
<ul>
  <li>Google BigQuery</li>
  <li>Jupyter Notebook</li>
  <li>Excel</li>
</ul>



## Table of Contents

1. [How Does Traffic and Conversion Vary by Channel?](#-1-listening-trends-over-time--11-yearly-spotify-listening-trends)
2. [How did different acquisition sources perform in attracting visitors in October 2021?](#-2-artists-songs-and-genres--21-skipping-behavior-211-device-specific-skipping-patterns-)
3. [Recommendations](#-3-device-analysis-and-lifestyle-changes--31-device-usage-breakdown)
4. [Executive Summary](#-3-device-analysis-and-lifestyle-changes--31-device-usage-breakdown)
5. [Appendix](#conclusion )
  


<h2>Project walk-through:</h2>
<h3>Question 1. How Does Traffic and Conversion Vary by Channel?</h3>

<p><b>The initial step</b> was to conduct an exploratory analysis of the data. I examined each of the .csv files to understand the columns they contained and how each of the tables is connected to each other. In other words, I needed to know which was the primary key, which was the foreign key, and how tables were connected via these keys so I could build a final logic.
</p>
<br />
<div align="center">
<img src="/images/eda_homepage_visits_table_10_rows.png"/>
<p><i><sub>First 10 rows of the homepage_visits dataset</sub></i></p>
</div>

<br />
<div align="center">
<img src="/images/eda_projects_created_table_10_rows.png"  />
<p><i><sub>First 10 rows of the projects_created dataset</sub></i></p>
</div>

<br />
<div align="center">
<img src="/images/eda_costs_table_all_rows.png" />
<p><i><sub>All the rows of the costs dataset</sub></i></p>
</div>

<br />

<p>I started by analyzing the visits data to uncover key metrics: the total number of visits, unique visits, and the number of visits that led to registrations and paid subscriptions. I also calculated the conversion rates between each step to better understand the customer journey.</p>

```sql
-- Combined script to retrieve visits statistics and conversions
WITH visit_stats AS (
   -- Retrieve visit statistics
   SELECT
       COUNT(VISITOR_ID) AS total_visitors,
       COUNT(DISTINCT VISITOR_ID) AS unique_visitors
   FROM `portfolio.project.homepage_visits`
),
registration_stats AS (
   -- Retrieve data about visitors with registrations and paying customers
   SELECT
       COUNT(DISTINCT CASE WHEN p.PROJECT_REGISTRATION_DATETIME IS NOT NULL THEN v.VISITOR_ID END) AS customers_with_registrations,
       COUNT(DISTINCT CASE WHEN p.SUBSCRIPTION_CREATED_DATE IS NOT NULL THEN v.VISITOR_ID END) AS paying_customers
   FROM
       `portfolio.project.homepage_visits` v
   JOIN
       `portfolio.project.projects_created` p
       ON v.VISITOR_ID = p.VISITOR_ID
)
SELECT
   vs.total_visitors,
   vs.unique_visitors,
   rs.customers_with_registrations,
   rs.paying_customers,
   --Calculate conversion rates
   ROUND(100 * rs.customers_with_registrations / vs.unique_visitors,1) AS conversion_to_registration,
   ROUND(100 * rs.paying_customers / rs.customers_with_registrations,1) AS conversion_to_payment
FROM
   visit_stats AS vs,
   registration_stats AS rs


```
<br />


---



<h3>💡 Learnings:</h3>
<p><b>Key Insight:</b> While the company’s home page is attracting a good number of visitors and repeat
visits, there is room for improvement in converting these visitors into registered users and paid subscribers.</p>
<h4>Additional Insights: </h4>
<p>A significant number of repeat visits suggest good engagement and retention. However, a notable drop-off occurs from unique visitors to registrations, with only 4.6%
converting. An even larger drop-off is observed from registrations to subscriptions, as only 4% of
those who register go on to subscribe.</p>
<p>
Identifying and addressing the reasons for these drop-offs can significantly improve
conversion rates. This may involve optimizing the registration and subscription processes,
offering more value or incentives to upgrade to paid plan, ensuring consistency in the
messaging and visuals between ads and the homepage, or enhancing the overall user
experience.
</p>

<p align="center">
<img src="/images/funnel.png" />
<br />

<blockquote style="background-color: #f0f0f0; padding: 15px; border-left: 5px solid #ccc; font-style: italic;">
<i> <b> Disclaimer: </b> While analysing the data, it was noticed that some visitors have no recorded visits before their subscription date in October 2021, only after, making it impossible to attribute revenue to the last
acquisition channel before conversion. This might suggest possible visits in prior months or data tracking issues. Examples are available in the Appendix.</i>
</blockquote>

<br />

---

<p><b>The next step</b> was to analyze the performance of each acquisition channel. This helped me identify which channels drive the most traffic and which ones excel or underperform in terms of converting users. 
  <br />
I simply tweaked the original script by adding the channel metric to gain these insights.

</p>

```sql
WITH visit_stats AS (
   -- Retrieve visit statistics by channel
   SELECT
       ACQUISITION_CHANNEL,
       COUNT(VISITOR_ID) AS total_visitors,
       COUNT(DISTINCT VISITOR_ID) AS unique_visitors
   FROM `portfolio.project.homepage_visits`
   GROUP BY ACQUISITION_CHANNEL
),
registration_stats AS (
   -- Retrieve data about visitors with registrations and paying customers by channel
   SELECT
       v.ACQUISITION_CHANNEL,
       COUNT(DISTINCT CASE WHEN p.PROJECT_REGISTRATION_DATETIME IS NOT NULL THEN v.VISITOR_ID END) AS customers_with_registrations,
       COUNT(DISTINCT CASE WHEN p.SUBSCRIPTION_CREATED_DATE IS NOT NULL THEN v.VISITOR_ID END) AS paying_customers
   FROM
       `portfolio.project.homepage_visits` v
   JOIN
       `portfolio.project.projects_created` p
       ON v.VISITOR_ID = p.VISITOR_ID
   GROUP BY v.ACQUISITION_CHANNEL
)
SELECT
   vs.ACQUISITION_CHANNEL,
   vs.total_visitors,
   vs.unique_visitors,
   rs.customers_with_registrations,
   rs.paying_customers,
   -- Calculate conversion rates by channel
   ROUND(100 * rs.customers_with_registrations / vs.unique_visitors, 1) AS conversion_to_registration,
   ROUND(100 * rs.paying_customers / rs.customers_with_registrations, 1) AS conversion_to_payment
FROM
   visit_stats AS vs
JOIN
   registration_stats AS rs
   ON vs.ACQUISITION_CHANNEL = rs.ACQUISITION_CHANNEL



```
<br />

---



<h3>💡 Learnings:</h3>
<p><b>Key Insight:</b> Almost all channels experience a low conversion rate to paid subscription, indicating that the strategy around encouraging users to upgrade to paid subscriptions may need to be refined.</p>
<h4>Additional Insights: </h4>
<p>Although <b>Channels 1 and 4</b> drive the most traffic to the homepage, they have the lowest
conversion rates to registrations. This may indicate issues such as overly broad targeting,
audience and/or message mismatch, or insufficiently compelling incentives to drive
registrations.</p>
<p>In contrast, <b>Channels 3, 5, 6, and 10</b> have the highest conversion rates to registrations,
with Channel 3 also excelling in paid subscriptions.
Analyzing Channel 3's success could provide valuable insights for other channels.</p>
<p><b>Channels 7 and 10 </b>face significant issues with paid subscriptions despite very high CR to
registrations, suggesting that users may not be seeing enough value during the trial
period or that the price of the paid subscription is perceived as too high.</p>
<p>Notably, there are no unique visitors from <b>Channel 9</b>, despite associated costs, which
could indicate the channel's ineffectiveness, tracking issues, or that it is simply an offline
channel not attributable to online traffic.
</p>
<br />

<p align="center">
<img src="/images/traffic_and_conversions_by_channel.png" />
<br />

<blockquote style="background-color: #f0f0f0; padding: 15px; border-left: 5px solid #ccc; font-style: italic;">
<i> <b> Disclaimer: </b> When analyzed by channels, the number of unique visits is 10.3% higher than the overall count, indicating that users are making multiple visits across different channels. It is advisable to implement multi-touch attribution modelling rather than attributing all the credit to either the first or last channel by default.</i>
</blockquote>
<br />

---


<p><b>Another observation</b> I've made during my analysis was an <b>unusual traffic trend</b>. When I extracted the day of the week for each date in October, I noticed that, unlike most businesses that typically scale back marketing activities over the weekend, this company’s advertising team reduced efforts on Fridays and Saturdays. This led to a significant drop in traffic, followed by a sharp increase on Sundays. </p>
<p>  This pattern is unconventional, and unless the team is specifically targeting Arab countries (which further analysis does not support), it may be worth adjusting the strategy to align with more conventional marketing schedules for potentially better results.
</p>

```sql
select
 EXTRACT (day FROM EVENT_DATETIME) as visit_day,
 FORMAT_TIMESTAMP('%A', event_datetime) AS day_of_week,
 COUNT(VISITOR_ID) as total_visitors,
 COUNT(DISTINCT VISITOR_ID) as unique_visitors
FROM `portfolio.project.homepage_visits`
GROUP BY visit_day,day_of_week
ORDER BY visit_day ASC
```
<br />
<p align="center">
<img src="/images/daily_visits.png" />
<br />

---

<br />
<h3>Question 2. How did different acquisition sources perform in attracting visitors in October 2021?</h3>
<p>After exploring the visits data, the next step was to evaluate the performance of the acquisition sources in terms of revenue, cost, and ROI.</p> 
<p>Given that visitors often come to the website multiple times and through various channels, I needed to determine how to attribute the revenue. Since the project focused on bottom-of-funnel activities, I chose to use a last-touch attribution model, giving credit to the last channel a visitor used before purchasing a paid subscription.</p>

<p>To achieve this, I created a script consisting of multiple Common Table Expressions (CTEs):</p>

<ul>
  <li><strong>Filtering Paid Visitors:</strong> First, I filtered all visitors who had paid subscriptions, along with their first contract value.</li>
  <li><strong>Attribution Logic:</strong> Since I used the last-touch attribution model, I filtered visits that occurred only before the subscription date by comparing the EVENT_DATETIME with the SUBSCRIPTION_CREATED_DATE. During this step, I noticed some visitors had no recorded visits before their subscription date in October 2021, only after it. This made it impossible to attribute revenue to the last acquisition channel before conversion, which could suggest missing visits in prior months or data tracking issues.</li>
  <li><strong>Identifying the Last Channel:</strong> The next step was to filter for the last channel before conversion, which I accomplished with two CTEs:
    <ul>
      <li>The first CTE assigned a row number to each visit for each user in descending order, using a window function so the visit just before the conversion would have a row number of 1.</li>
      <li>The second CTE filtered for channels with row number 1, indicating the last visit before conversion.</li>
    </ul>
  </li>
  <li><strong>Calculating Costs and Revenue:</strong> The next two CTEs calculated the costs of each channel and the revenue. The revenue was calculated as the first contract value multiplied by 8, as the task description indicated that the lifespan of a project was equal to 8 months.</li>
  <li><strong>ROI and Additional Metrics:</strong> In the outer query, I calculated ROI and additional metrics such as revenue per customer and customer acquisition costs. These metrics ensure that your acquisition costs are justified by the revenue generated from new customers. For one channel with zero costs, I added a case statement to assign an extremely high ROI value to indicate infinite ROI.</li>
</ul>

<p>The result was a table with 8 rows and 7 columns. I also noticed that I didn’t have data for 3 out of 11 channels from the costs CSV file.</p>

<p>Finally, the output of the query was exported to Excel for follow-up analysis and visualizations.</p>

```sql
With paid_customers AS ( --CTE to filter all the visitors with paid subscription along with their first_contract_value
   SELECT
       Distinct hv.VISITOR_ID,
       FIRST_CONTRACT_VALUE
   FROM `portfolio.project.tidio.homepage_visits` as hv
   JOIN `portfolio.project.tidio.projects_created` as pc
   ON hv.VISITOR_ID=pc.VISITOR_ID
   WHERE SUBSCRIPTION_CREATED_DATE IS NOT NULL
   GROUP BY hv.VISITOR_ID,FIRST_CONTRACT_VALUE
),


 filtered_visits AS ( --CTE to filter visits which happened before "conversion". I've selected the last attribution method to calculate ROI, thus I was interesed only in visits prior to "conversion".
     SELECT
         Distinct hv.VISITOR_ID,
         ACQUISITION_CHANNEL,
         CAST(EVENT_DATETIME AS DATE) as EVENT_DATETIME,
         PROJECT_REGISTRATION_DATETIME,
         CAST(SUBSCRIPTION_CREATED_DATE AS DATE) as SUBSCRIPTION_CREATED_DATE
     FROM `portfolio.project.homepage_visits` as hv
     JOIN `portfolio.project.projects_created` as pc
     ON hv.VISITOR_ID=pc.VISITOR_ID
     --WHERE EVENT_DATETIME <= PROJECT_REGISTRATION_DATETIME --gives you 214 out of 276 visitors only
     WHERE CAST(EVENT_DATETIME AS DATE)  <= CAST(SUBSCRIPTION_CREATED_DATE AS DATE) --gives you 269 out of 276 visitors
     GROUP BY VISITOR_ID, ACQUISITION_CHANNEL,EVENT_DATETIME, PROJECT_REGISTRATION_DATETIME,SUBSCRIPTION_CREATED_DATE
 ),


 last_channel AS ( --CTE to assign a unique row number to each visit for each visitor oredered by the date of a visit in DESC order, so the visit happened right before conversion will have a row number = 1
     SELECT
         fv.VISITOR_ID,
         ACQUISITION_CHANNEL,
         EVENT_DATETIME,
         ROW_NUMBER() OVER (PARTITION BY fv.VISITOR_ID ORDER BY EVENT_DATETIME DESC) AS row_num
     FROM filtered_visits as fv
     JOIN paid_customers as pc
     ON fv.VISITOR_ID=pc.VISITOR_ID
 ),


 filtered_last_channel AS ( --CTE to filter only the last channel before "conversion"
     SELECT
         VISITOR_ID,
         ACQUISITION_CHANNEL,
         EVENT_DATETIME
     FROM
         last_channel
     WHERE
         row_num = 1
 ),


 channel_costs AS ( -- CTE to bring the costs of running each of the channels
     SELECT
         DISTINCT hv.ACQUISITION_CHANNEL,
         COST AS total_cost
     FROM `portfolio.project.homepage_visits` as hv
     JOIN `portfolio.project.costs` as co
     ON hv.ACQUISITION_CHANNEL=co.ACQUISITION_CHANNEL
     GROUP BY ACQUISITION_CHANNEL,COST
 ),


 channel_revenue AS ( --CTE to calulate the total revenue coming from all the unique paying customers and attribute it to the last channel before "conversion"
     SELECT
         ACQUISITION_CHANNEL,
         COUNT(DISTINCT pc.VISITOR_ID) AS unique_paying_customers,
         ROUND(SUM(pc.FIRST_CONTRACT_VALUE * 8),2) AS total_revenue
     FROM paid_customers as pc
     JOIN filtered_last_channel as flc
     ON pc.VISITOR_ID=flc.VISITOR_ID
     GROUP BY ACQUISITION_CHANNEL
 )
 SELECT --final step - calculting ROI for each of the channels
   cr.ACQUISITION_CHANNEL,
   cr.unique_paying_customers,
   cr.total_revenue,
   cc.total_cost,
   CASE
       WHEN cc.total_cost = 0 THEN 10000 -- a high value to indicate infinite ROI for the channel_11 with zero cost
       ELSE ROUND((cr.total_revenue - cc.total_cost) / cc.total_cost * 100, 2)
   END AS roi,
   ROUND(cr.total_revenue/cr.unique_paying_customers,2) as revenue_per_customer,
   ROUND(cc.total_cost/ cr.unique_paying_customers, 2) as customer_acquisition_cost
   FROM channel_revenue cr
   JOIN channel_costs cc
   ON cr.ACQUISITION_CHANNEL = cc.ACQUISITION_CHANNEL
   ORDER BY roi DESC;



```

---

<h3>💡 Learnings:</h3>
<p><b>Key Insight:</b> To optimize overall performance increase investment in high-performing Channels 1, 3, and 8, address the cost-revenue imbalance in Channels 7, 9, and 10, and improve Channel 2's customer acquisition efficiency.</p>
<h4>Additional Insights: </h4>
<p><b>Channel 11 </b> currently boasts an exceptionally high ROI due to zero costs; however, this
may not be sustainable in the long term.
</p>
<p><b>Channels 1, 3, and 8 </b> are high performers in both revenue and ROI. To maximize returns,
consider increasing investment in these channels. Analyze the successful strategies
employed in Channels 1, 3, and 8, and look for opportunities to replicate these tactics
across other channels.</p>
<p><b>Channel 2</b> requires immediate attention due to its negative ROI, which highlights
inefficiencies in customer acquisition costs. Consider strategies to reduce costs, enhance
marketing efforts, or reallocate resources to more profitable channels.</p>
<p><b>Channels 7, 9, and 10</b> have negative ROI because there are costs but no revenue. These
channels need a thorough review to identify and address the underlying issues.
</p>
<br />


<p align="center">
<img src="/images/revenue_roi_cost_by_channel.png" />
<br />

<blockquote style="background-color: #f0f0f0; padding: 15px; border-left: 5px solid #ccc; font-style: italic;">
<i> <b> Disclaimer: </b> Revenue is calculated using the last attribution methodology. ROI is determined by subtracting the cost from the revenue and then dividing the result by the cost.</i>
</blockquote>
<br />

---

<h3>💡 Learnings:</h3>
<p><b>Key Insight:</b> Maximize ROI by focusing on efficient channels, improving moderate performers, expanding high-revenue segments, and addressing inefficiencies</p>
<h4>Additional Insights: </h4>
<p><b>High Efficiency Channels: </b> channel 1,3 and 8 have a low customer acquisition cost, high
revenue and a high number of unique paying customers, making them highly efficient
channels. These channels should be prioritized for continued investment and
optimization to maintain their strong performance.

</p>
<p><b> Inefficient Channel: </b>  channel 2 has a high customer acquisition cost and a low revenue
per customer. This indicates inefficiency, as the high acquisition cost outweighs the
revenue generated per customer.
</p>
<p><b>Channel 2</b> requires immediate attention due to its negative ROI, which highlights
inefficiencies in customer acquisition costs. Consider strategies to reduce costs, enhance
marketing efforts, or reallocate resources to more profitable channels.</p>
<p><b>Moderate Performance:</b>  channels 4 and 6 perform moderately well but has room for
improvement in reducing acquisition costs or increasing the customer base.
</p>
<p><b>High Revenue but Low Customer Base:</b>  channel 5 has the highest revenue per
customer but only 2 unique paying customers. The high CAC suggests that while the
channel attracts premium customers, it needs to expand its customer base to improve
overall efficiency.
</p>
<br />


<p align="center">
<img src="/images/renenue_vs_acquisition_costs.png" />
<br />

<blockquote style="background-color: #f0f0f0; padding: 15px; border-left: 5px solid #ccc; font-style: italic;">
<i> <b> Disclaimer: </b>  The size of each bubble indicates the number of unique paying customers. Larger bubbles represent a higher number of paying customers.<br />
CAC = Customer Acquisition Cost<br />
The actual numbers can be checked in Appendix.
</i>
</blockquote>
<br />

---
<p><b>To find additional insights</b>, I also included additional metrics available in the provided .csv files, such as project industry, country, and project platform. I calculated the number of days between the last visit and subscription, as well as between registration and subscription dates, to see if any relationships emerged. The results were further analyzed in Python, but I did not find any significant insights worth including in the final report. Instead, I decided to use the additional metrics in the recommendations.</p>

<p><b>For well-performing channels</b>, I recommended continuing the current strategy and focusing on the countries, platforms, and industries that contribute the most to revenue.</p>

<p><b>For channels needing optimization</b>, I tailored recommendations based on specific needs:</p>
<ul>
  <li><strong>If customer acquisition costs (CAC) were too high:</strong> I suggested adopting strategies from channels with lower CAC.</li>
  <li><strong>If expanding customer reach was necessary:</strong> I recommended targeting industries or platforms that have shown strong performance in other channels.</li>
</ul>

```sql
With paid_customers AS (
  SELECT
      Distinct hv.VISITOR_ID,
      PROJECT_INDUSTRY,
      PROJECT_COUNTRY_ISO,
      cc.country_name,
      PROJECT_PLATFORM,
      FIRST_CONTRACT_VALUE,
      COUNT(CASE WHEN CAST(EVENT_DATETIME AS DATE)  <= CAST(SUBSCRIPTION_CREATED_DATE AS DATE) THEN EVENT_DATETIME END) as count_sessions_before_subscr
  FROM `portfolio.project.homepage_visits` as hv
  JOIN `portfolio.project.projects_created` as pc
  ON hv.VISITOR_ID=pc.VISITOR_ID
  LEFT JOIN `bigquery-public-data.country_codes.country_codes` as cc
  ON pc.PROJECT_COUNTRY_ISO=cc.alpha_2_code
  WHERE SUBSCRIPTION_CREATED_DATE IS NOT NULL
  GROUP BY hv.VISITOR_ID,PROJECT_INDUSTRY,PROJECT_COUNTRY_ISO,cc.country_name, PROJECT_PLATFORM,FIRST_CONTRACT_VALUE
),


filtered_visits AS (
    SELECT
        Distinct hv.VISITOR_ID,
        ACQUISITION_CHANNEL,
        CAST(EVENT_DATETIME AS DATE) as EVENT_DATETIME,
        PROJECT_REGISTRATION_DATETIME,
        CAST(SUBSCRIPTION_CREATED_DATE AS DATE) as SUBSCRIPTION_CREATED_DATE,
        DATE_DIFF(CAST(SUBSCRIPTION_CREATED_DATE AS DATE), CAST(PROJECT_REGISTRATION_DATETIME AS DATE), DAY) AS days_between_reg_and_subscr
    FROM `portfolio.project.homepage_visits` as hv
    JOIN `portfolio.project.projects_created` as pc
    ON hv.VISITOR_ID=pc.VISITOR_ID
    WHERE CAST(EVENT_DATETIME AS DATE)  <= CAST(SUBSCRIPTION_CREATED_DATE AS DATE)
    GROUP BY VISITOR_ID, ACQUISITION_CHANNEL,EVENT_DATETIME, PROJECT_REGISTRATION_DATETIME,SUBSCRIPTION_CREATED_DATE
),


last_channel AS (
    SELECT
        fv.VISITOR_ID,
        ACQUISITION_CHANNEL,
        EVENT_DATETIME,
        ROW_NUMBER() OVER (PARTITION BY fv.VISITOR_ID ORDER BY EVENT_DATETIME DESC) AS row_num,
        fv.days_between_reg_and_subscr
    FROM filtered_visits as fv
    JOIN paid_customers as pc
    ON fv.VISITOR_ID=pc.VISITOR_ID


),


filtered_last_channel AS (
    SELECT
        VISITOR_ID,
        ACQUISITION_CHANNEL,
        EVENT_DATETIME,
        days_between_reg_and_subscr
    FROM
        last_channel
    WHERE
        row_num = 1
),


channel_costs AS (
    SELECT
        DISTINCT hv.ACQUISITION_CHANNEL,
        COST AS total_cost
    FROM `portfolio.project.homepage_visits` as hv
    JOIN `portfolio.project.costs` as co
    ON hv.ACQUISITION_CHANNEL=co.ACQUISITION_CHANNEL
    GROUP BY ACQUISITION_CHANNEL,COST
),


channel_revenue AS (
    SELECT
        pc.VISITOR_ID,
        ACQUISITION_CHANNEL,
        pc.PROJECT_INDUSTRY,
        pc.PROJECT_COUNTRY_ISO,
        pc.country_name,
        pc.PROJECT_PLATFORM,
        flc.days_between_reg_and_subscr,
        pc.count_sessions_before_subscr,
        ROUND(SUM(pc.FIRST_CONTRACT_VALUE * 8),2) AS total_revenue
    FROM paid_customers as pc
    JOIN filtered_last_channel as flc
    ON pc.VISITOR_ID=flc.VISITOR_ID
    GROUP BY pc.VISITOR_ID,ACQUISITION_CHANNEL, pc.PROJECT_INDUSTRY, pc.PROJECT_COUNTRY_ISO, pc.country_name, pc.PROJECT_PLATFORM, flc.days_between_reg_and_subscr, pc.count_sessions_before_subscr
)
SELECT
  cr.VISITOR_ID,
  cr.ACQUISITION_CHANNEL,
  cr.PROJECT_INDUSTRY,
  cr.PROJECT_COUNTRY_ISO,
  cr.country_name,
  cr.PROJECT_PLATFORM,
  cr.days_between_reg_and_subscr,
  cr.count_sessions_before_subscr,
  cr.total_revenue
  FROM channel_revenue cr
  JOIN channel_costs cc
  ON cr.ACQUISITION_CHANNEL = cc.ACQUISITION_CHANNEL
  ORDER BY cr.total_revenue DESC;

```

---
