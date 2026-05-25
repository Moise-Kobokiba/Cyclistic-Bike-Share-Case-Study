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
View the Interactive Tableau Dashboard Here: https://public.tableau.com/views/CyclisticBike-ShareAnalysisUserBehaviorInsights2014/Dashboard?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

Final Recommendations: We have reached the peak of our journey, and here is our path forward to drive revenue. First, we need to create a 'Weekend Warrior' annual pass. Casuals don't want a full membership because they don't commute on Tuesdays, so let’s give them a tier that fits their weekend habits. Second, we should use geo-targeted mobile app notifications. When a casual rider passes the 20-minute mark near a park or waterfront on a sunny Saturday, we hit them with an in-app prompt showing how much money they would save today by converting to a member. Finally, our digital marketing creative needs to pivot: stop showing people commuting in suits to attract casuals; show them enjoying a weekend cruise. If we implement these steps, we can smoothly convert our highest-value casual riders into recurring annual revenue. Thank you, and I’d love to open the floor to your questions.
