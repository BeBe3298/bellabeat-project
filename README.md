# Bellabeat Case Study
Capstone project for the Google Data Analytics Certificate. Analyzing Fitbit data to uncover user behavior trends and generate insights for Bellabeat’s marketing strategy.

- 🎯 [Executive Summary](#Executive-Summary)
- 🌿 [Overview](#Bellabeat-Overview)
  - 🧑‍💼 [Stakeholders](#Stakeholders)
  - 📊 [Dataset](#Dataset)
  - 🛠️ 📅 [Tools and Target Completion Date](#Tools-and-Target-Completion-Date)
  
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
- sleepDay_merged.csv\
  \
Limitations: The dataset includes data from only 33 anonymized users over a two-month period. \
This limited sample size, timeframe and timeslot may restrict the ability to fully represent the user habits and long-term behavior nowadays. 

## Tools and Target Completion Date
Tools: Google Colab (Pyhton), SQL, Excel, Tableau\
Target Completion date: Thursday, 28 Aug 2025

## Data Preparation

**Context**
- METs file too large for Excel / Power Query  
- BigQuery only accepts `yyyy-mm-dd` date format  
- Raw data had mixed date formats (e.g. `mm/dd/yyyy`, `yyyy-mm-dd`, timestamps)

**Solution**  
Use Google Colab + Python for preprocessing  

***Step 1. Import datasets***
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
```
Findings:
Sleep Data: 4/12/2016 12:00:00 AM
METs Data: 4/12/2016 12:00:00 AM
Activity Data: 4/12/2016

```
#Structure preview
print("sleep_df：")
sleep_df.info()

print("METs mets_df：")
mets_df.info()

print("activity_df：")
activity_df.info()
```
**Findings**  
- Date columns stored as `object` type (strings), not proper `datetime`.  
- Formats inconsistent across datasets (`mm/dd/yyyy`, `yyyy-mm-dd`, timestamps).  

**Follow-up Action**  
Convert all date columns to standardized `datetime` format to enable accurate time-based analysis and joins in BigQuery.


***Step 2. Fix datetime format***
```
sleep_df['SleepDay'] = pd.to_datetime(sleep_df['SleepDay'])
mets_df['ActivityMinute'] = pd.to_datetime(mets_df['ActivityMinute'])
activity_df['ActivityDate'] = pd.to_datetime(activity_df['ActivityDate'])
```

```# Check the data type of date columns
print(sleep_df['SleepDay'].dtype)
print(mets_df['ActivityMinute'].dtype)
print(activity_df['ActivityDate'].dtype)
```
Result: Converted the data type successfully

```
#Standardize the date formate
#  Sleep Data ➜ Keep only the date (as string), discard the time
sleep_df['SleepDate'] = sleep_df['SleepDay'].dt.strftime('%Y/%m/%d')

# Activity Data ➜ Keep only the date (as string), discard the time
activity_df['ActivityDateOnly'] = activity_df['ActivityDate'].dt.strftime('%Y/%m/%d')

# METS Data ➜ Keep both date and time (stored as string, for BigQuery compatibility)
mets_df['ActivityDate'] = mets_df['ActivityMinute'].dt.strftime('%Y/%m/%d')
mets_df['ActivityTime'] = mets_df['ActivityMinute'].dt.strftime('%H:%M:%S')
```
```
#check if changed successfully
print(sleep_df[['SleepDay', 'SleepDate']].head())
print(activity_df[['ActivityDate', 'ActivityDateOnly']].head())
print(mets_df[['ActivityMinute', 'ActivityDate', 'ActivityTime']].head())
```
Results:
All date formats across all datasets have been successfully converted to the `yyyy/mm/dd` format.

