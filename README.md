# Cyclistic Bike-Share Case Study: Converting Casual Riders to Annual Members
### Google Data Analytics Professional Certificate Capstone Project
**Author:** Moise Kobokiba  
**Tools Used:** Google Cloud Storage, Google BigQuery (SQL), Tableau Public

---

## 1. The Business Task
The objective of this analysis is to identify the core behavioral differences between casual riders and annual members of Cyclistic. By understanding these distinct usage patterns, the marketing team can develop targeted campaigns to convert high-value casual riders into profitable, long-term annual members, thereby driving sustainable revenue growth.

* **Primary Stakeholder:** Lily Moreno (Director of Marketing)
* **Secondary Stakeholders:** Cyclistic Executive Team

---

## 2. Data Preparation & Source Documentation
* **Source Data:** 12 months of historical bike trip data from 2014 (Q1Q2 and Q3Q4 datasets) provided openly by Motivate International Inc.
* **Data Privacy:** The dataset contains no personally identifiable information (PII). No credit card information, names, or addresses were processed, ensuring complete rider anonymity.
* **Storage & Scale:** Due to the large file sizes exceeding browser upload limits, the raw datasets were staged in a Google Cloud Storage bucket before being ingested into Google BigQuery.

---

## 3. Data Processing & Cleaning Documentation
To ensure data integrity, structural anomalies were cleaned using BigQuery SQL. Legacy schema fields (`Subscriber` and `Customer`) were standardized to match modern business terminology (`member` and `casual`). Trips under 1 minute (potential false dockings) and over 24 hours (lost/stolen assets) were filtered out.

### SQL Pipeline Script:
```sql
CREATE OR REPLACE TABLE `cyclistic_data_2014.combined_trips_2014` AS
WITH raw_combined AS (
  SELECT trip_id, starttime, stoptime, tripduration, usertype 
  FROM `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.trips_q1q2`
  UNION ALL
  SELECT trip_id, starttime, stoptime, tripduration, usertype 
  FROM `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.trips_q3q4`
)
SELECT 
  trip_id,
  CASE 
    WHEN usertype = 'Subscriber' THEN 'member'
    WHEN usertype = 'Customer' THEN 'casual'
    ELSE usertype 
  END AS member_casual,
  TIMESTAMP(starttime) AS started_at,
  TIMESTAMP(stoptime) AS ended_at,
  ROUND(tripduration / 60, 2) AS ride_length_minutes,
  EXTRACT(DAYOFWEEK FROM TIMESTAMP(starttime)) AS day_of_week,
  EXTRACT(MONTH FROM TIMESTAMP(starttime)) AS month,
  EXTRACT(HOUR FROM TIMESTAMP(starttime)) AS hour_of_day
FROM 
  raw_combined
WHERE 
  tripduration >= 60 
  AND tripduration <= 86400;

```
## 4. Key Findings & Data Story (Analyze & Share)

📊 **View the Interactive Tableau Dashboard Here** - https://public.tableau.com/views/CyclisticBike-ShareAnalysisUserBehaviorInsights2014/Dashboard?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

**The Data Presentation:** https://docs.google.com/presentation/d/1OD9sBvTApkjMsw83J4v4a06j2vw690XF9XVEbLAyBaQ/edit?usp=sharing

**Finding 1:** The Commuter Peak (Hourly Trends)
Insight: Annual members exhibit extreme volume spikes at 8:00 AM and 5:00 PM, matching typical corporate rush hours. This indicates their primary usage is utilitarian (work/school commuting).
Casual Behavior: Casual riders show no morning spike; instead, their usage builds a smooth, steady afternoon curve peaking between 12:00 PM and 3:00 PM.

**Finding 2:** The Weekend Takeover (Weekly Trends)Insight: Monday through Friday, annual members heavily dominate total system volume.
Casual Behavior: On Saturdays and Sundays, member activity drops slightly while casual rider counts surge significantly, matching or exceeding member volume. Casuals use the network for leisure and weekend recreation.

**Finding 3:** The Value Gap (Average Duration)Insight: While members take more frequent individual trips, their rides are highly calculated and short (averaging ~12 minutes).  Casual Behavior: Casual riders keep bikes checked out for an average of nearly 30 minutes per trip—three times longer than members—indicating deep experiential and leisure engagement with the fleet.

## 5. Strategic Recommendations (Act)
Based directly on the data insights above, here are the top three strategic recommendations to maximize annual memberships:

**The "Weekend Warrior" Annual Membership:** Since casual rider volume spikes drastically on Saturdays and Sundays, introduce an affordable, weekend-only subscription tier. This lowers the entry barrier for leisure riders who do not need a weekday commuter pass.

**Contextual App Promotions Near Leisure Hotspots:** Leverage real-time trip length metrics. When a casual user exceeds a 20-minute ride on a weekend afternoon, trigger a mobile app notification offering an instant discount on an annual membership upgrade.

**Refocused Marketing Creative Asset Design:** Pivot digital advertising away from professional, workplace utility images. Design creative campaigns showcasing weekend exploration, health, fitness, and lifestyle benefits to speak directly to the emotional drivers of casual users.
