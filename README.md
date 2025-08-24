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

Step 1. Import datasets

```python
import pandas as pd

sleep_df = pd.read_csv('/content/April_sleepDay_merged.csv')
mets_df = pd.read_csv('/content/April_minuteMETsNarrow_merged.csv')
activity_df = pd.read_csv('/content/April_dailyActivity_merged.csv')
heartrate_df = pd.read_csv('/content/April_heartrate_seconds_merged.csv')```
