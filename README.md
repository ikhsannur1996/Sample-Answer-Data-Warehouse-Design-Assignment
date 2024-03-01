# Sample-Answer-Data-Warehouse-Design-Assignment

# Employee Performance Analysis Case Study

## Introduction

Company XYZ, a large multinational corporation, is interested in analyzing and improving employee performance across its various departments. The company has a diverse workforce with employees from different educational backgrounds and experience levels. To better understand the factors influencing employee performance, Company XYZ has decided to implement a comprehensive data warehouse solution.

## Data Warehouse Design

### Dimension Tables

1. **dim_department**: Stores information about different departments within the organization.
   - `department_id` (Primary Key): Unique identifier for each department.
   - `department_name`: Name of the department.

2. **dim_education**: Holds data related to different levels of education attained by employees.
   - `education_id` (Primary Key): Unique identifier for each education level.
   - `education_level`: Level of education attained (e.g., Bachelor's, Master's).

3. **dim_employee**: Contains details about employees, including personal information, department, and education.
   - `employee_id` (Primary Key): Unique identifier for each employee.
   - `first_name`: Employee's first name.
   - `last_name`: Employee's last name.
   - `birth_date`: Employee's date of birth.
   - `hire_date`: Employee's date of hire.
   - `gender`: Employee's gender.
   - `department_id` (Foreign Key): References `dim_department.department_id`.
   - `education_id` (Foreign Key): References `dim_education.education_id`.

### Fact Table

- **fact_employee_performance**: Records performance scores of employees over time.
   - `performance_id` (Primary Key): Unique identifier for each performance record.
   - `employee_id` (Foreign Key): References `dim_employee.employee_id`.
   - `performance_score`: Score assigned to the employee's performance.
   - `performance_date`: Date of the performance evaluation.

Schema Description (Snowflake Schema)
![drawSQL-image-export-2024-03-01](https://github.com/ikhsannur1996/Sample-Answer-Data-Warehouse-Design-Assignment/assets/32507742/f9dd40ef-10fd-4cf1-ac3a-4123c74e3ea7)

In the snowflake schema design:
- The fact table, `fact_employee_performance`, remains at the center, containing measures or metrics related to employee performance.
- Dimension tables, such as `dim_department` and `dim_education`, are normalized to reduce redundancy and improve data integrity. However, `dim_employee` is denormalized to include both department and education information directly.

Sample Queries (Data Mart Tables):

Data Mart Table 1: Employee Performance Summary by Department
```sql
CREATE TABLE data_mart_employee_performance_department AS
SELECT d.department_name,
       COUNT(f.employee_id) AS num_employees,
       AVG(f.performance_score) AS avg_performance_score
FROM fact_employee_performance f
JOIN dim_employee e ON f.employee_id = e.employee_id
JOIN dim_department d ON e.department_id = d.department_id
GROUP BY d.department_name;
```

Data Mart Table 2: Employee Education Distribution
```sql
CREATE TABLE data_mart_employee_education AS
SELECT edu.education_level,
       COUNT(e.employee_id) AS num_employees
FROM dim_employee e
JOIN dim_education edu ON e.education_id = edu.education_id
GROUP BY edu.education_level;
```

Data Mart Table 3: Monthly Performance Trend
```sql
CREATE TABLE data_mart_monthly_performance AS
SELECT DATE_TRUNC('month', f.performance_date) AS month,
       AVG(f.performance_score) AS avg_performance_score
FROM fact_employee_performance f
GROUP BY DATE_TRUNC('month', f.performance_date)
ORDER BY month;
```

Explanation of Data Mart Tables:
1. `data_mart_employee_performance_department`: This table provides a summary of employee performance by department, including the number of employees and the average performance score for each department. It joins `dim_employee` with `dim_department` to access department information.
2. `data_mart_employee_education`: This table shows the distribution of employees based on their education level. It directly joins `dim_employee` with `dim_education` to access education information.
3. `data_mart_monthly_performance`: This table displays the trend of average performance scores on a monthly basis, allowing stakeholders to track performance changes over time. It utilizes the `fact_employee_performance` table.
