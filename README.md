Project Title & Subtitle: Cyclistic-Bike-Share-Case-Study

Introduction & Business Task: The objective is to identify key behavioral differences between casual riders and annual members to help design a targeted marketing campaign aimed at converting existing casual riders into profitable, long-term annual members.

The Tech Stack: Google Cloud Storage, Google BigQuery (SQL), and Tableau Public.

Data Cleaning & Transformation Documentation:

Step 1: Combine & Clean the Data

CREATE OR REPLACE TABLE `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.combined_trips_2014` AS
WITH raw_combined AS (
  SELECT trip_id, starttime, stoptime, tripduration, usertype 
  FROM `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.trips_q1q2`
  UNION ALL
  SELECT trip_id, starttime, stoptime, tripduration, usertype 
  FROM `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.trips_q3q4`
)
SELECT 
  trip_id,
  -- Standardize legacy user types to match the case study goals
  CASE 
    WHEN usertype = 'Subscriber' THEN 'member'
    WHEN usertype = 'Customer' THEN 'casual'
    ELSE usertype 
  END AS member_casual,
  
  TIMESTAMP(starttime) AS started_at,
  TIMESTAMP(stoptime) AS ended_at,
  
  -- Calculate ride length in minutes from tripduration (which is in seconds)
  ROUND(tripduration / 60, 2) AS ride_length_minutes,
  
  -- Time extractions for the analysis phase
  EXTRACT(DAYOFWEEK FROM TIMESTAMP(starttime)) AS day_of_week,
  EXTRACT(MONTH FROM TIMESTAMP(starttime)) AS month,
  EXTRACT(HOUR FROM TIMESTAMP(starttime)) AS hour_of_day
FROM 
  raw_combined
WHERE 
  tripduration >= 60 -- Removes trips under 1 minute (accidental docks/tests)
  AND tripduration <= 86400; -- Removes trips over 24 hours (stolen or lost bikes)

Step 2: Run the Analysis Queries

1. High-Level Overview (Averages and Volumes)

SELECT 
  member_casual,
  COUNT(trip_id) AS total_rides,
  ROUND(AVG(ride_length_minutes), 2) AS average_ride_length_minutes
FROM 
  `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.combined_trips_2014`
GROUP BY 
  member_casual;

2. Weekly Habits (Which days are most popular?)

SELECT 
  member_casual,
  day_of_week,
  COUNT(trip_id) AS total_rides,
  ROUND(AVG(ride_length_minutes), 2) AS average_ride_length_minutes
FROM 
  `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.combined_trips_2014`
GROUP BY 
  member_casual, day_of_week
ORDER BY 
  member_casual, day_of_week;

3. Hourly Habits (When do they ride?)

SELECT 
  member_casual,
  hour_of_day,
  COUNT(trip_id) AS total_rides
FROM 
  `project-242eb54f-3251-46ae-af5.cyclistic_data_2014.combined_trips_2014`
GROUP BY 
  member_casual, hour_of_day
ORDER BY 
  member_casual, hour_of_day;

View the Interactive Tableau Dashboard Here: https://public.tableau.com/views/CyclisticBike-ShareAnalysisUserBehaviorInsights2014/Dashboard?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

Final Recommendations: First, we need to create a 'Weekend Warrior' annual pass. Casuals don't want a full membership because they don't commute on Tuesdays, so let’s give them a tier that fits their weekend habits. Second, we should use geo-targeted mobile app notifications. When a casual rider passes the 20-minute mark near a park or waterfront on a sunny Saturday, we hit them with an in-app prompt showing how much money they would save today by converting to a member. Finally, our digital marketing creative needs to pivot: stop showing people commuting in suits to attract casuals; show them enjoying a weekend cruise. If we implement these steps, we can smoothly convert our highest-value casual riders into recurring annual revenue. Thank you, and I’d love to open the floor to your questions.
