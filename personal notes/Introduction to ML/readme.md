# Introduction to Machine Learning — Topic List

Course 10 | Foundational machine learning expertise using AWS SageMaker — from data preparation and exploratory analysis to deploying models like XGBoost and AutoGluon. Covers the full ML workflow: feature engineering, model tuning, and evaluation.

---

## Lessons

1. **Introduction to Machine Learning**
   Overview of key background around Machine Learning and preparing for the rest of the course.

2. **Exploratory Data Analysis**
   Using AWS SageMaker Studio to access S3 datasets; data analysis and feature engineering with Data Wrangler and Pandas; labeling new data using SageMaker Ground Truth.

3. **Machine Learning Concepts**
   ML Lifecycles; supervised vs. unsupervised ML; regression methods; classification methods.

_(More lessons may follow — not visible in current screenshot.)_

---

Table commands

---

Managed Table - CREATE then INSERT OR LOAD - Default

CREATE TABLE emp(empno INT, empname STRING, age INT, gender STRING, Salary INT, Designation STRING, DeptID STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n';

External Table - CREATE to REFER THE DATA

hdfs dfs -mkdir emp_ext

hdfs dfs -put emp.csv emp_ext

CREATE EXTERNAL TABLE emp_ext(empno INT, empname STRING, age INT, gender STRING, Salary INT, Designation STRING, DeptID STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY'\n' LOCATION '/user/clouduser/emp_ext' TBLPROPERTIES("skip.header.line.count"="1");

INSERT OR LOAD

---

INSERT INTO TABLE <tblname> VALUES (1, ....);

LOAD DATA INPATH 'path of the file' INTO TABLE <tblname>

LFS load

---

LOAD DATA LOCAL INPATH '<LFS path> INTO TABLE <tblname>;

LOAD DATA LOCAL INPATH 'emp.csv' INTO TABLE emp;

eq to hdfs dfs -put emp.csv /user/hive/warehouse/hadoopdb.db/emp

HDFS load

---

LOAD DATA INPATH '<HDFS path> INTO TABLE <tblname>;

LOAD DATA INPATH 'HadoopDir/emp.csv' INTO TABLE emp;

eq to hdfs dfs -cp HadoopDir/emp.csv /user/hive/warehouse/hadoopdb.db/emp

ALTER TABLE emp SET TBLPROPERTIES("skip.header.line.count"="1");
