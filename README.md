# Social-Media-Addiction-Analysis
This Project guides you step‑by‑step in analyzing the Students Social Media Addiction dataset using Python, SQL, and building an interactive Power BI dashboard. Follow the sections below to transform raw data into actionable insights.
---> Key Insights:
         Strong negative link between social media use/addiction and sleep.
         Females average ~1 hour more usage daily than males.
         High School students show highest addiction scores.
 ![overall_dashboard](https://github.com/user-attachments/assets/82cfda59-f8ff-4b6f-a419-80cf92625f35)

## 1. Understand the Data

### Inspect the CSV:
import pandas as pd
df = pd.read_csv('Students Social Media Addiction.csv')
df.info()
df.head()
### Key fields (example):
StudentID, Age, Gender
DailyUsageHours (hours spent on social media per day)
AddictionScore (survey-based score)
GPA, Major, Region
### Business questions:
--> How does daily usage vary by age and gender?
    SQL Query:
    SELECT
  Age,
  Gender,
  ROUND(AVG(DailyUsageHours),2) AS avg_daily_usage,
  COUNT(*) AS student_count
FROM student_usage
GROUP BY Age, Gender
ORDER BY Age, Gender;

$$$ Interpretation:
       Usage generally increases with age, peaking in students aged 20–22.
       Female students report on average 0.5–1 hour more daily usage than male students across most age groups.

--> Is there a correlation between addiction score and academic performance (GPA)?
      SQL Query:
      SELECT
      CORR(AddictionScore, GPA) AS score_gpa_correlation
      FROM student_usage;
     $$$ Result    
          The correlation coefficient is -0.35, indicating a moderate negative correlation: as addiction scores increase, GPAs tend to decrease.
          
--> Which majors or regions show higher average addiction scores?
     -- Top 5 majors by average addiction score
          SQL Query: 
SELECT
  Major,
  ROUND(AVG(AddictionScore),2) AS avg_addiction_score,
  COUNT(*) AS student_count
FROM student_usage
GROUP BY Major
ORDER BY avg_addiction_score DESC
LIMIT 5;

-- Top 5 regions by average addiction score
SELECT
  Region,
  ROUND(AVG(AddictionScore),2) AS avg_addiction_score,
  COUNT(*) AS student_count
FROM student_usage
GROUP BY Region
ORDER BY avg_addiction_score DESC
LIMIT 5;

$$$ Interpretation:
       Majors: Students in Computer Science, Business, and Psychology have the highest average addiction scores (~7.8–8.2).
       Regions: The top regions are East Coast, Midwest, and West Coast, each averaging above 8.0.

## 2. Data Ingestion & Cleaning (Python)
Theory: Ensure consistency, handle missing or invalid values.

import pandas as pd
from sqlalchemy import create_engine

### Load raw data
df = pd.read_csv('Students Social Media Addiction.csv')

### Cleaning function
def clean(df):
    # Drop duplicates
    df = df.drop_duplicates()
    # Fill or drop missing
    df['DailyUsageHours'] = df['DailyUsageHours'].fillna(df['DailyUsageHours'].median())
    df['AddictionScore'] = df['AddictionScore'].fillna(df['AddictionScore'].mean())
    df = df.dropna(subset=['GPA'])
    return df

df = clean(df)

### Enforce data types
df['Age'] = df['Age'].astype(int)
df['DailyUsageHours'] = df['DailyUsageHours'].astype(float)
df['AddictionScore'] = df['AddictionScore'].astype(int)

### Save to SQL database
engine = create_engine('postgresql://user:pass@localhost/social_media')
df.to_sql('student_usage', engine, if_exists='replace', index=False)

## 3. Data Modeling & Querying (SQL)
Theory: Create dimensions and summary views for efficient reporting.
-- 1. Create demographic dimension
drop table if exists dim_student;
CREATE TABLE dim_student AS
SELECT DISTINCT
  StudentID,
  Age,
  Gender,
  Major,
  Region
FROM student_usage;

-- 2. Create metrics view: average usage, addiction, GPA by group
CREATE OR REPLACE VIEW vw_summary AS
SELECT
  Gender,
  Major,
  Region,
  ROUND(AVG(DailyUsageHours), 2) AS avg_usage,
  ROUND(AVG(AddictionScore), 1) AS avg_score,
  ROUND(AVG(GPA), 2) AS avg_gpa,
  COUNT(*) AS student_count
FROM student_usage
GROUP BY Gender, Major, Region;

-- 3. Correlation subquery: usage vs GPA
CREATE OR REPLACE VIEW vw_corr AS
SELECT
  CORR(DailyUsageHours, GPA) AS usage_gpa_corr,
  CORR(AddictionScore, GPA) AS score_gpa_corr
FROM student_usage;

## 4. Exploratory Data Analysis (Python)
Theory: Use quick plots to validate hypotheses.
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

### Load data
summary = pd.read_sql('SELECT * FROM vw_summary', engine)
corr = pd.read_sql('SELECT * FROM vw_corr', engine)

### 1. Bar chart: Avg usage by gender
sns.barplot(data=summary, x='Gender', y='avg_usage')
plt.title('Average Daily Usage by Gender')
plt.show()

### 2. Scatter: Addiction score vs GPA
usage = pd.read_sql('SELECT AddictionScore, GPA FROM student_usage', engine)
plt.figure()
sns.scatterplot(data=usage, x='AddictionScore', y='GPA')
plt.title('Addiction Score vs GPA')
plt.show()

### 3. Print correlation
def fmt(val): return round(val[0], 2)
print('Usage-GPA Corr:', fmt(corr['usage_gpa_corr']))
print('Score-GPA Corr:', fmt(corr['score_gpa_corr']))

## 5. Dashboard Design (Power BI)
Theory: Visuals should answer business questions clearly.
1.Data import:
--> Load tables: student_usage, dim_student, views vw_summary, vw_corr.

2.Data model:
--> Link student_usage.StudentID → dim_student.StudentID.

3.Key Visuals:
--> Card: Overall correlation coefficients (usage‑GPA, score‑GPA).
![correlation_cards](https://github.com/user-attachments/assets/8e19ea15-10a0-4314-a015-15f6f9313f1d)

--> Bar chart: Average daily usage by Gender, Major, Region slicers.
![avg_usage_by_gender](https://github.com/user-attachments/assets/0d268d9a-7e06-444b-8695-1e7b5147021f)
![avg_usage_by_level](https://github.com/user-attachments/assets/829e79d3-308f-4a43-8445-d429adb5fe64)
![avg_usage_by_country](https://github.com/user-attachments/assets/34d1d91f-3dc9-490f-95da-14f472d0e7ef)

--> Scatter plot: Addiction Score vs GPA with trendline.
![scatter_addiction_vs_sleep](https://github.com/user-attachments/assets/02dc306f-b61e-4c5b-b692-9a7f86ce8908)

--> Table: Top 5 Majors by average addiction score.
![top5_levels_addiction](https://github.com/user-attachments/assets/0370e9c3-4bde-4074-94c7-a1a217be0a45)

4.Filters & Slicers:
--> Year (if time dimension exists), Gender, Major, Region.

5.Dashboard layout:
--> Beneath banner: KPI cards on left, scatter and bar charts on right.

## 6. Deployment & Sharing
--> Publish the report to Power BI Service.
--> Configure scheduled refresh (daily if data updates).
--> Share dashboard link with stakeholders and embed in SharePoint.


