# Bellabeat Case Study
Capstone project for the Google Data Analytics Certificate. Analyzing Fitbit data to uncover user behavior trends and generate insights for Bellabeat’s marketing strategy.

- 🎯 [Executive Summary](#Executive-Summary)
- 🌿 [Overview](#Bellabeat-Overview)
  - 🧑‍💼 [Stakeholders](#Stakeholders)
  - 📊 [Dataset](#Dataset)
  - 🛠️ 📅 [Tools and Target Completion Date](#Tools-and-Target-Completion-Date)
- [Data Preparation](#Data-Preparation)
- [Data Cleaning](#Data-Cleaning)
## Executive Summary
As part of the final capstone for the Google Data Analytics Professional Certificate, this case study explores user activity data from Fitbit devices. Acting as a Junior Data Analyst on Bellabeat’s marketing analytics team, I analyzed public data from Kaggle to identify behavioral trends and provide actionable recommendations. These insights aim to support Bellabeat’s strategic decisions in product development and targeted marketing.

## Bellabeat Overview
Bellabeat is a health-tech company offering smart wellness products for women, including the Bellabeat app, Leaf Tracker, Time smartwatch, Spring hydration bottle, and a subscription-based membership. These devices sync with the app to track activity, sleep, stress, and reproductive health.

## Stakeholders
- Urška Sršen: Bellabeat’s cofounder and Chief Creative Officer 
- Sando Mur: Mathematician and Bellabeat’s cofounder; key member of the Bellabeat executive team 
- Bellabeat marketing analytics team: A team of data analysts responsible for collecting, analyzing, and reporting data that helps guide Bellabeat’s marketing strategy. 

## Dataset
Source: FitBit Fitness Tracker Data from Kaggle\
Period: From 12 March 2016 to 12 May 2016\
Files used:
- dailyActivity_merged.csv
- minuteMETsNarrow_merged.csv
- sleepDay_merged.csv
  
Limitations: The dataset includes data from only 33 anonymized users over a two-month period. \
This limited sample size, timeframe and timeslot may restrict the ability to fully represent the user habits and long-term behavior nowadays. 

## Tools and Target Completion Date
Tools: Google Colab (Pyhton), SQL, Excel, Tableau\
Target Completion date: Thursday, 28 Aug 2025

## Data Preparation

***Context***
- METs file too large for Excel / Power Query  
- BigQuery only accepts `yyyy-mm-dd` date format  
- Raw data had mixed date formats (e.g. `mm/dd/yyyy`, `yyyy-mm-dd`, timestamps)

***Solution*** 
Use Google Colab + Python for preprocessing  

**Step 1 Import Datasets & Data Preview**
```python
import pandas as pd

sleep_df = pd.read_csv('/content/April_sleepDay_merged.csv')
mets_df = pd.read_csv('/content/April_minuteMETsNarrow_merged.csv')
activity_df = pd.read_csv('/content/April_dailyActivity_merged.csv')

# Preview first rows
print("Sleep Data:")  
print(sleep_df.head())  

print("\nMETS Data:")  
print(mets_df.head())  

print("\nActivity Data:")  
print(activity_df.head())  


#Structure preview
print("sleep_df：")
sleep_df.info()

print("METs mets_df：")
mets_df.info()

print("activity_df：")
activity_df.info()
```

- ***Findings:***
  - Formats inconsistent across datasets (`mm/dd/yyyy`, `yyyy-mm-dd`, timestamps).
    - Sleep Data: 4/12/2016 12:00:00 AM
    - METs Data: 4/12/2016 12:00:00 AM
    - Activity Data: 4/12/2016
  - Date columns stored as `object` type (strings), not proper `datetime`.  

- ***Follow-up Action in Step 2.1 & 2.2:***
  - Fix the date column type.
  - Convert all date columns to standardized `datetime` format to enable accurate time-based analysis and joins in BigQuery.


**Step 2.1 - Fix Date Type & Check**
```python
sleep_df['SleepDay'] = pd.to_datetime(sleep_df['SleepDay'])
mets_df['ActivityMinute'] = pd.to_datetime(mets_df['ActivityMinute'])
activity_df['ActivityDate'] = pd.to_datetime(activity_df['ActivityDate'])

# Check the data type of date columns
print(sleep_df['SleepDay'].dtype)
print(mets_df['ActivityMinute'].dtype)
print(activity_df['ActivityDate'].dtype)
```
- ***Result:***
  - Converted the data type successfully


**Step 2.2 - Standardize Date Format & Check**

```python
#Standardize the date format
#  Sleep Data ➜ Keep only the date (as string), discard the time
sleep_df['SleepDate'] = sleep_df['SleepDay'].dt.strftime('%Y/%m/%d')

# Activity Data ➜ Keep only the date (as string), discard the time
activity_df['ActivityDateOnly'] = activity_df['ActivityDate'].dt.strftime('%Y/%m/%d')

# METS Data ➜ Keep both date and time (stored as string, for BigQuery compatibility)
mets_df['ActivityDate'] = mets_df['ActivityMinute'].dt.strftime('%Y/%m/%d')
mets_df['ActivityTime'] = mets_df['ActivityMinute'].dt.strftime('%H:%M:%S')

#check if changed successfully
print(sleep_df[['SleepDay', 'SleepDate']].head())
print(activity_df[['ActivityDate']].head())
print(mets_df[['ActivityMinute', 'ActivityDate', 'ActivityTime']].head())
```
- ***Results:***
  - All date formats across all datasets have been successfully converted to the `yyyy/mm/dd` format.

**Step 2.3 - Download CVS files**

```python
sleep_df.to_csv('sleep_data_cleaned.csv', index=False)
activity_df.to_csv('activity_data_cleaned.csv', index=False)
mets_df.to_csv('mets_data_cleaned.csv', index=False)
```


## Data Cleaning

Upload cvs files onto Big Query to analysis

**Step 1 - Check Number of Distinct ID & Rows**

```SQL
#Check Number of Distinct ID
SELECT 'Activity_Data' AS table_name, COUNT(DISTINCT id) AS distinct_ids
FROM `lucky-sphinx-461610-i6.BELLABEAT.Activity`

UNION ALL
SELECT 'Sleep_Data' AS table_name, COUNT(DISTINCT id) AS distinct_ids
FROM `lucky-sphinx-461610-i6.BELLABEAT.Sleep`

UNION ALL
SELECT 'Mets_data' AS table_name, COUNT(DISTINCT id) AS distinct_ids
FROM `lucky-sphinx-461610-i6.BELLABEAT.mets`;

#Check Number of Rows
SELECT COUNT(*) AS total_before
FROM lucky-sphinx-461610-i6.BELLABEAT.Activity;

SELECT COUNT(*) AS total_before
FROM lucky-sphinx-461610-i6.BELLABEAT.Sleep;

SELECT COUNT(*) AS total_before
FROM lucky-sphinx-461610-i6.BELLABEAT.mets;
```

- ***Results:***
  - Number of Distinct ID
    ```SQL
    Row	table_name	distinct_ids
    1	Activity_Data	33
    2	Sleep_Data	24
    3	Mets_data	33
      ```
  - Number of Rows
    - Activity_Data:  940
    - Sleep_Data: 413
    - MEts_data: 1325580
 
**Step 2 - Find Duplicated Rows, Remove & Check**
**2.1 - Activity Data**
- **2.1.1 - Find Duplicated**
  ```SQL
  CREATE OR REPLACE TABLE lucky-sphinx-461610-i6.BELLABEAT.Activity_Duplicates AS
  SELECT * FROM lucky-sphinx-461610-i6.BELLABEAT.Activity
  QUALIFY COUNT(*) OVER (
  PARTITION BY Id, ActivityDate, TotalSteps,
               CAST(TotalDistance AS STRING),
               CAST(TrackerDistance AS STRING),
               CAST(LoggedActivitiesDistance AS STRING),
               CAST(VeryActiveDistance AS STRING),
               CAST(ModeratelyActiveDistance AS STRING),
               CAST(LightActiveDistance AS STRING),
               CAST(SedentaryActiveDistance AS STRING),
               VeryActiveMinutes, FairlyActiveMinutes,
               LightlyActiveMinutes, SedentaryMinutes, Calories) > 1;
  ```
  - ***Result:***
  - No duplicated Activity data need to be removed.
            
- **2.2 - Sleep Data**
  - **2.2.1 - Find Duplicates**
    ```SQL
    CREATE OR REPLACE TABLE lucky-sphinx-461610-i6.BELLABEAT.Sleep_Duplicates AS
    SELECT *
    FROM lucky-sphinx-461610-i6.BELLABEAT.Sleep
    QUALIFY COUNT(*) OVER (
    PARTITION BY Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed
    ) > 1;
    ```
    
    - ***Result:***
      - 3 duplicates rows
      - ```SQL
        Row	Id	SleepDay	TotalSleepRecords	TotalMinutesAsleep	TotalTimeInBed
        1	8378563200	2016-04-25	1	388	402
        2	8378563200	2016-04-25	1	388	402
        3	4388161847	2016-05-05	1	471	495
        4	4388161847	2016-05-05	1	471	495
        5	4702921684	2016-05-07	1	520	543
        6	4702921684	2016-05-07	1	520	543
        ```

  - **2.2.2 - Remove Duplicates & Check**
    ```SQL
    #Remove duplicates
    CREATE OR REPLACE TABLE lucky-sphinx-461610-i6.BELLABEAT.Sleep_Data_NoDupes AS
    SELECT *
    FROM (
    SELECT *, ROW_NUMBER() OVER (
    PARTITION BY Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed
    ORDER BY SleepDay) AS rn
    FROM lucky-sphinx-461610-i6.BELLABEAT.Sleep)
    WHERE rn = 1;
    ```

     ```sql
     #Check number of rows
     SELECT COUNT(*) AS total_before
     FROM lucky-sphinx-461610-i6.BELLABEAT.Sleep;
     ```
    - ***Result:***
      - ```SQL
        Number of rows
        410
        ```

- **2.3 - METs Data**
  - **2.3.1 - Find Duplicates**

    ```SQL
    CREATE OR REPLACE TABLE lucky-sphinx-461610-i6.BELLABEAT.mets_Duplicates AS
    SELECT *
    FROM lucky-sphinx-461610-i6.BELLABEAT.mets
    QUALIFY COUNT(*) OVER (
    PARTITION BY Id, ActivityMinute, METs
    ) > 1;
    ```
    - ***Result:***
      - No duplicated METs data need to be removed.
    
**Step 3 - Find Missing Value**
- **3.1 Activity Data**
  ```SQL
  SELECT
  COUNT(*) AS total_rows,
  COUNTIF(Id IS NULL) AS missing_Id,
  COUNTIF(ActivityDate IS NULL) AS missing_ActivityDate,
  COUNTIF(TotalSteps IS NULL) AS missing_TotalSteps,
  COUNTIF(TotalDistance IS NULL) AS missing_TotalDistance,
  COUNTIF(TrackerDistance IS NULL) AS missing_TrackerDistance,
  COUNTIF(LoggedActivitiesDistance IS NULL) AS
  missing_LoggedActivitiesDistance,
  COUNTIF(VeryActiveDistance IS NULL) AS missing_VeryActiveDistance,
  COUNTIF(ModeratelyActiveDistance IS NULL) AS
  missing_ModeratelyActiveDistance,
  COUNTIF(LightActiveDistance IS NULL) AS missing_LightActiveDistance,
  COUNTIF(SedentaryActiveDistance IS NULL) AS
  missing_SedentaryActiveDistance,
  COUNTIF(VeryActiveMinutes IS NULL) AS missing_VeryActiveMinutes,
  COUNTIF(FairlyActiveMinutes IS NULL) AS missing_FairlyActiveMinutes,
  COUNTIF(LightlyActiveMinutes IS NULL) AS missing_LightlyActiveMinutes,
  COUNTIF(SedentaryMinutes IS NULL) AS missing_SedentaryMinutes,
  COUNTIF(Calories IS NULL) AS missing_Calories,
  COUNTIF(ActivityDateOnly IS NULL) AS missing_ActivityDateOnly
  FROM lucky-sphinx-461610-i6.BELLABEAT.Activity;
  ```
  - **Result**
    - No missing value

- **3.2 Sleep Data**
  ```sql
  SELECT
  COUNT(*) AS total_rows,
  COUNTIF(Id IS NULL) AS missing_Id,
  COUNTIF(SleepDay IS NULL) AS missing_SleepDay,
  COUNTIF(TotalSleepRecords IS NULL) AS missing_TotalSleepRecords,
  COUNTIF(TotalMinutesAsleep IS NULL) AS missing_TotalMinutesAsleep,
  COUNTIF(TotalTimeInBed IS NULL) AS missing_TotalTimeInBed
  FROM lucky-sphinx-461610-i6.BELLABEAT.Sleep;
  ```
  - **Result**
    - No missing value
      
- **3.1 METs Data**
  ```SQL
  SELECT
  COUNT(*) AS total_rows,
  COUNTIF(Id IS NULL) AS missing_Id,
  COUNTIF(ActivityMinute IS NULL) AS missing_ActivityMinute,
  COUNTIF(METs IS NULL) AS missing_METs
  FROM lucky-sphinx-461610-i6.BELLABEAT.mets;
  ```

  - **Result**
    - No missing value
   
**Step 4 - Add METs into Activity Data**
```sql
CREATE OR REPLACE TABLE `lucky-sphinx-461610-i6.BELLABEAT.Activity_with_METs` AS
SELECT
  a.*,
  m.Total_METs
FROM
  `lucky-sphinx-461610-i6.BELLABEAT.Activity` AS a
LEFT JOIN
  `lucky-sphinx-461610-i6.BELLABEAT.mets_summary` AS m
ON
  a.Id = m.Id AND a.ActivityDate = m.ActivityDate
ORDER BY
  a.Id,
  a.ActivityDate;
```

 
**Step 5 - Data Consolidation**
**#1 - Join sleep data to Activity data**
**#2 - Classify sleep category**
**#3 - Sum total activity minutes**
**#4 - Identify Weekday**
**#5 - Identify Weekday or Weekend**
```sql
#1 - Join sleep data to Activity data
CREATE OR REPLACE TABLE `lucky-sphinx-461610-i6.BELLABEAT.activity_with_METs2` AS
SELECT
  `lucky-sphinx-461610-i6.BELLABEAT.Activity_with_METs`.Id,
  ActivityDate,
  TotalSteps,
  TotalDistance,
  TrackerDistance,
  LoggedActivitiesDistance,
  VeryActiveDistance,
  ModeratelyActiveDistance,
  LightActiveDistance,
  SedentaryActiveDistance,
  VeryActiveMinutes,
  FairlyActiveMinutes,
  LightlyActiveMinutes,
  SedentaryMinutes,
  Calories,
  Total_METs,

  -- sleep data
  TotalMinutesAsleep,
  TotalTimeInBed,

  –#2 - Classify sleep category
  CASE
    WHEN TotalMinutesAsleep IS NULL THEN "Unknown"
    WHEN TotalMinutesAsleep < 420 THEN "Short"
    WHEN TotalMinutesAsleep BETWEEN 420 AND 540 THEN "Normal"
    WHEN TotalMinutesAsleep > 540 THEN "Long"
  END AS SleepCategory,

  -- #3 - Sum total activity minutes
  (VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes) AS TotalActivityMinutes,

  –#4 - Identify Weekday
  FORMAT_DATE('%A', DATE(ActivityDate)) AS Weekday,

  –#5 - Identify Weekday or Weekend
  CASE
    WHEN FORMAT_DATE('%A', DATE(ActivityDate)) IN ("Saturday", "Sunday") THEN "Weekend"
    ELSE "Weekday"
  END AS DayType

FROM
  `lucky-sphinx-461610-i6.BELLABEAT.Activity_with_METs`
LEFT JOIN
  `lucky-sphinx-461610-i6.BELLABEAT.Sleep_Data_NoDupes`
ON
  `lucky-sphinx-461610-i6.BELLABEAT.Activity_with_METs`.Id = `lucky-sphinx-461610-i6.BELLABEAT.Sleep_Data_NoDupes`.Id
  AND `lucky-sphinx-461610-i6.BELLABEAT.Activity_with_METs`.ActivityDate = `lucky-sphinx-461610-i6.BELLABEAT.Sleep_Data_NoDupes`.SleepDay;
```
  - **Result**
    - Get a new table: lucky-sphinx-461610-i6.BELLABEAT.activity_with_METs2

      - **Check number of distinct ID of the new table**
      ```sql
      SELECT
      COUNT(DISTINCT Id) AS Unique_User_Count
      FROM
      `lucky-sphinx-461610-i6.BELLABEAT.activity_with_METs2`;
      ```
     
        - **Result**
          ```sql
          Row	Unique_User_Count 1	33
          ```

           
**Step 6 - Create new table and count the average data value per ID**
```sql
CREATE OR REPLACE TABLE `lucky-sphinx-461610-i6.BELLABEAT.average_per_user` AS
SELECT
  Id,
  AVG(TotalSteps) AS avg_TotalSteps,
  AVG(TotalDistance) AS avg_TotalDistance,
  AVG(TrackerDistance) AS avg_TrackerDistance,
  AVG(LoggedActivitiesDistance) AS avg_LoggedActivitiesDistance,
  AVG(VeryActiveDistance) AS avg_VeryActiveDistance,
  AVG(ModeratelyActiveDistance) AS avg_ModeratelyActiveDistance,
  AVG(LightActiveDistance) AS avg_LightActiveDistance,
  AVG(SedentaryActiveDistance) AS avg_SedentaryActiveDistance,
  AVG(VeryActiveMinutes) AS avg_VeryActiveMinutes,
  AVG(FairlyActiveMinutes) AS avg_FairlyActiveMinutes,
  AVG(LightlyActiveMinutes) AS avg_LightlyActiveMinutes,
  AVG(SedentaryMinutes) AS avg_SedentaryMinutes,
  AVG(Calories) AS avg_Calories,
  AVG(Total_METs) AS avg_Total_METs,
  AVG(TotalMinutesAsleep) AS avg_TotalMinutesAsleep,
  AVG(TotalTimeInBed) AS avg_TotalTimeInBed,
  AVG(TotalActivityMinutes) AS avg_TotalActivityMinutes
FROM
  `lucky-sphinx-461610-i6.BELLABEAT.activity_with_METs2`
GROUP BY
  Id;
```
- **Result:**
  - Get new table: lucky-sphinx-461610-i6.BELLABEAT.average_per_user

##

  
