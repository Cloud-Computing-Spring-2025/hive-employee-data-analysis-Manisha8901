# Hive Employee Analysis

This project showcases how to analyze employee data using Hive, Hadoop, and Docker. It includes a dataset with employee and department details, which are examined through various SQL queries.

## Prerequisites

Before starting, ensure the following are set up on your system:

### 1. Docker
Make sure **Docker** is installed and running on your system. It will be used to run Hadoop and Hive containers.

### 2. Hadoop
A **Hadoop** cluster should be set up within Docker containers for HDFS storage. You’ll need a working Hadoop environment to store and manage the datasets.

### 3. Hive
**Hive** should be installed and configured to interact with Hadoop. You’ll use Hive to query the data stored in HDFS.

## Dataset

This project uses two datasets:

- **employees.csv**: Contains employee details including ID, name, age, job role, salary, project, join date, and department.
- **departments.csv**: Contains department details such as department ID, name, and location.

## Setup Instructions

### 1. Docker Setup

####  A. Start the Hadoop Cluster
To start the Hadoop cluster, run the following command:
```sh
docker compose up -d
```


#### B. Copy the CSV Files to Docker Container
To copy the CSV datasets (`employees.csv` and `departments.csv`) into the Docker container, run:
```sh
docker cp /path/to/employees.csv resourcemanager:/tmp/
docker cp /path/to/departments.csv resourcemanager:/tmp/
```

#### C. Create HDFS Directories
Ensure that the required HDFS directories are created:
```sh
docker exec -it resourcemanager hadoop fs -mkdir -p /input
```

#### D. Upload the CSV Files to HDFS
Now, move the datasets into HDFS:
```sh
docker exec -it resourcemanager hadoop fs -put /tmp/departments.csv /input/
docker exec -it resourcemanager hadoop fs -put /tmp/employees.csv /input/
```

#### E. Verify Files in HDFS
To confirm the files are uploaded successfully, use:
```sh
docker exec -it resourcemanager hadoop fs -ls /input/
```

#### F. Access the Resourcemanager Container
To access the resourcemanager container for internal inspection or modification:
```sh
docker exec -it resourcemanager /bin/bash
```

You can also list files in the `/tmp/` directory to ensure they were copied:
```sh
docker exec -it resourcemanager ls -l /tmp/
```

### 2. Set up Hive and Create Tables

#### A. Open the Hive Shell
To open the Hive shell, run:
```sh
docker exec -it hive-server /bin/bash
hive
```

#### B. Create a Database (if not exists)
Create a new database for the analysis:
```sql
CREATE DATABASE IF NOT EXISTS web_logs;
```

#### C. Use the Database
Switch to the `web_logs` database:
```sql
USE web_logs;
```

#### D. Create Tables
Create the tables for employees and departments:
```sql
CREATE TABLE IF NOT EXISTS employees_temp (
    emp_id INT,
    name STRING,
    age INT,
    job_role STRING,
    salary DOUBLE,
    project STRING,
    join_date DATE,
    department STRING
);

CREATE TABLE IF NOT EXISTS departments (
    department_id INT,
    department_name STRING,
    location STRING
);
```

#### E. Load Data into Hive Tables
Now, load the CSV files into the Hive tables:
```sql
LOAD DATA INPATH '/input/employees.csv' INTO TABLE employees_temp;
LOAD DATA INPATH '/input/departments.csv' INTO TABLE departments;
```

### 3. Running Queries

#### A. Retrieve_Employees_After_2015
```sql
SELECT * FROM employees WHERE join_date > '2015-12-31';
```

#### B. Retrieve employees whose salary is above the average salary of their department
```sql
SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;
```

#### C. Identify employees working on the 'Alpha' project
```sql
SELECT * FROM employees WHERE project = 'Alpha';
```

#### D. Count how many employees are there in each job role
```sql
SELECT job_role, COUNT(*) AS employee_count FROM employees GROUP BY job_role;
```

#### E. Retrieve employees whose salary is above the average salary of their department
```sql
SELECT e.* FROM employees e
JOIN (SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department) d
ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

#### F. Find the department with the highest number of employees
```sql
SELECT department, COUNT(*) AS emp_count FROM employees GROUP BY department ORDER BY emp_count DESC LIMIT 1;
```

#### G. Check for employees with null values in any column and exclude them from analysis
```sql
SELECT * FROM employees WHERE emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL AND
job_role IS NOT NULL AND salary IS NOT NULL AND project IS NOT NULL AND join_date IS NOT NULL AND department IS NOT NULL;
```

#### H. Join the employees and departments tables to display employee details along with department location
```sql
SELECT e.*, d.location FROM employees e
JOIN departments d ON e.department = d.department_name;
```

#### I. Rank employees within each department based on salary
```sql
SELECT emp_id, name, department, salary,
RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

#### J. Find the top 3 highest-paid employees in each department
```sql
SELECT * FROM (
    SELECT emp_id, name, department, salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
    FROM employees
) ranked_employees WHERE salary_rank <= 3;
```
---

Let me know if you need further modifications! You can now directly copy this into your `README.md` file.
