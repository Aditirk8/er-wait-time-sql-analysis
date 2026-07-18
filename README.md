# Emergency Room Wait Time Analysis

Raw healthcare data modelled into a star schema and analysed using SQL in DuckDB.

---

## Overview
An end-to-end analytical SQL project built on an ER wait time dataset sourced from Kaggle. 
The raw dataset was a single wide table with 19 columns and 5000 rows which is not the most 
structured way to store data. The goal was to take that flat table, decompose it into 
a proper star schema, and then use SQL to extract meaningful operational insights about 
ER performance.

**Tools:** Python · Pandas · DuckDB · Jupyter Notebook  
**Domain:** Healthcare Operations  
**Dataset:** [ER Wait Time Dataset — Kaggle](https://www.kaggle.com/datasets/rivalytics/er-wait-time)

---

## Why This Dataset?
The raw table violated normalization rules such as hospital name, region, and patient details were repeated across every row. It also had natural primary keys already present (Hospital ID, Patient ID, Visit ID) which was not used properly. Hence, I converted it into a star schema to present the data in a more structured format. 

---

## Data Modelling: Star Schema

### What is a Star Schema?
A star schema organises data into one central fact table surrounded by dimension tables. The fact table holds measurable events one row per ER visit, with foreign keys pointing outward to dimensions. The dimension tables hold descriptive context who the patient was, which hospital they visited, and when.

A flat table mixes descriptive attributes with measurable facts, which makes aggregation queries verbose and forces unnecessary data repetition. The star schema separates them cleanly, hospital name and region live in dim_hospital, date attributes live in dim_time, and the fact table holds only foreign keys and measures.

### Dimension Tables

**dim_patient**
| Column | Type | Key |
|---|---|---|
| Patient ID | VARCHAR | PK |

**dim_hospital**
| Column | Type | Key |
|---|---|---|
| Hospital ID | VARCHAR | PK |
| Hospital Name | VARCHAR | — |
| Region | VARCHAR | — |

**dim_time**
| Column | Type | Key |
|---|---|---|
| Visit Date | DATE | PK |
| Day of Week | VARCHAR | — |
| Season | VARCHAR | — |
| Time of Day | VARCHAR | — |

### Fact Table

**fact_er_visits**
| Column | Type | Key |
|---|---|---|
| Visit ID | VARCHAR | PK |
| Patient ID | VARCHAR | FK → dim_patient |
| Hospital ID | VARCHAR | FK → dim_hospital |
| Visit Date | DATE | FK → dim_time |
| Nurse-to-Patient Ratio | INT | Measure |
| Specialist Availability | INT | Measure |
| Facility Size (Beds) | INT | Measure |
| Time to Registration (min) | INT | Measure |
| Time to Triage (min) | INT | Measure |
| Time to Medical Professional (min) | INT | Measure |
| Total Wait Time (min) | INT | Measure |
| Patient Outcome | VARCHAR | Measure |
| Patient Satisfaction | INT | Measure |
| Urgency Level | VARCHAR | Measure |

---

## Why DuckDB?
This project is purely analytical every query aggregates across the full dataset. That is an OLAP (Online Analytical Processing) workload. DuckDB is built specifically for OLAP, whereas SQLite is designed for OLTP (Online Transactional Processing) individual inserts, updates, and record lookups. DuckDB also reads pandas dataframes directly, which made loading the star schema tables into the database straightforward.

---

## Analysis

### Exploratory Data Analysis

**Visits are evenly distributed across urgency levels.**  
All four urgency levels account for roughly 24-26% of visits each, meaning the dataset 
is well balanced and averages are not skewed by any one group.

**Urgency level correctly predicts wait time.**  
Critical patients waited an average of 18 minutes versus 173 minutes for Low urgency 
patients. The triage system is functioning as intended.

**Higher visit volume does not mean lower satisfaction.**  
Riverside Medical Center sees the most visits (1023) but has one of the lower satisfaction 
scores (2.76). The relationship between volume and quality is not linear.

**Evening is consistently the busiest period across all seasons.**  
Evening visits account for 8.5-9.1% of all visits per season. Summer evenings are the 
single busiest combination at 9.12%.

---

### Operational Insights
**Regional wait times are nearly identical, but the bottleneck is the same everywhere.**  
Urban (81.98 min) and Rural (81.82 min) regions show almost no difference. In both cases, 
time to medical professional (~45 min) is the dominant delay — not registration (~11 min) 
or triage (~25 min).

**Nurse-to-Patient Ratio has a far stronger effect on wait time than Specialist Availability.**  
Medium nurse ratio produces average waits of 120-126 minutes regardless of specialist level. 
Low nurse ratio drops that to ~41 minutes. Specialist availability alone shows minimal 
variation within the same nurse ratio bucket.

**16.2% of Low urgency patients left without being seen.**  
Critical patients split almost evenly between admitted (50.4%) and discharged (49.6%). 
Over 1 in 6 Low urgency patients left before receiving care.

**Springfield General Hospital is the worst performer in the Urban region.**  
With an average wait time of 82.70 minutes, it ranks first in the Urban region — above 
Riverside Medical Center (81.81) and Summit Health Center (81.43).

**December is the worst month, September the best.**  
Wait times peaked in December with a month-over-month increase of +33.67 minutes versus 
November. September recorded the lowest average at 64.85 minutes.
