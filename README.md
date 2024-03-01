# Employee Performance Data Warehouse Design Case Study

## Introduction

Company XYZ, a large multinational corporation, aims to analyze and improve employee performance across its various departments. With a diverse workforce comprising employees from different educational backgrounds and experience levels, understanding the factors influencing employee performance is crucial. To achieve this, Company XYZ has decided to implement a comprehensive data warehouse solution.

## Data Warehouse Design

### Dimension Tables

1. **dim_department**: 
   - `department_id` (Primary Key): Unique identifier for each department.
   - `department_name`: Name of the department.

2. **dim_education**: 
   - `education_id` (Primary Key): Unique identifier for each education level.
   - `education_level`: Level of education attained (e.g., Bachelor's, Master's).

3. **dim_employee**: 
   - `employee_id` (Primary Key): Unique identifier for each employee.
   - `first_name`: Employee's first name.
   - `last_name`: Employee's last name.
   - `birth_date`: Employee's date of birth.
   - `hire_date`: Employee's date of hire.
   - `gender`: Employee's gender.
   - `department_id` (Foreign Key): References `dim_department.department_id`.
   - `education_id` (Foreign Key): References `dim_education.education_id`.

### Fact Table

- **fact_employee_performance**: 
   - `performance_id` (Primary Key): Unique identifier for each performance record.
   - `employee_id` (Foreign Key): References `dim_employee.employee_id`.
   - `performance_score`: Score assigned to the employee's performance.
   - `performance_date`: Date of the performance evaluation.

### Schema Description (Snowflake Schema)
![Employee](https://github.com/ikhsannur1996/Sample-Answer-Data-Warehouse-Design-Assignment/assets/32507742/7325ace5-21a1-45a8-8fbf-30236bbf98a2)


In the snowflake schema design:
- The fact table, `fact_employee_performance`, remains at the center, containing measures or metrics related to employee performance.
- Dimension tables, such as `dim_department` and `dim_education`, are normalized to reduce redundancy and improve data integrity. However, `dim_employee` is denormalized to include both department and education information directly.

**Snowflake Schema:**
In a Snowflake schema, data is organized into a centralized fact table surrounded by multiple dimension tables, forming a hierarchical structure. The fact table typically contains transactional or event data at its core, while dimension tables contain descriptive attributes related to the facts.

- **Parents**: The central fact table, in this case, `fact_employee_performance`, likely contains performance metrics or indicators related to employees.
- **Children Level 1**: The first level of children consists of dimension tables directly related to the employees, such as `dim_employee`, which might contain attributes like employee ID, name, etc.
- **Children Level 2**: The second level of children includes dimension tables that provide further context about employees, such as `dim_department` containing information about the departments they belong to, and `dim_education` containing details about their educational background.


### Sample Queries: Employee Data Mart Tables

**1. Employee Performance Summary by Department**

```sql
CREATE TABLE data_mart_employee_performance_department AS
SELECT 
    d.department_name,
    COUNT(f.employee_id) AS num_employees,
    AVG(f.performance_score) AS avg_performance_score
FROM 
    fact_employee_performance f
JOIN 
    dim_employee e ON f.employee_id = e.employee_id
JOIN 
    dim_department d ON e.department_id = d.department_id
GROUP BY 
    d.department_name;
```

**Explanation**: 
This table summarizes employee performance by department, providing the number of employees and the average performance score for each department.

**2. Employee Education Distribution**

```sql
CREATE TABLE data_mart_employee_education AS
SELECT 
    edu.education_level,
    COUNT(e.employee_id) AS num_employees
FROM 
    dim_employee e
JOIN 
    dim_education edu ON e.education_id = edu.education_id
GROUP BY 
    edu.education_level;
```

**Explanation**: 
This table displays the distribution of employees based on their education level.

**3. Monthly Performance Trend**

```sql
CREATE TABLE data_mart_monthly_performance AS
SELECT 
    DATE_TRUNC('month', f.performance_date) AS month,
    AVG(f.performance_score) AS avg_performance_score
FROM 
    fact_employee_performance f
GROUP BY 
    DATE_TRUNC('month', f.performance_date)
ORDER BY 
    month;
```

**Explanation**: 
This table presents the trend of average performance scores on a monthly basis, allowing stakeholders to track performance changes over time.


## Sales Data Warehouse Design Case Study

### Introduction

Company XYZ, a retail corporation, aims to analyze its sales data to gain insights into product performance, customer demographics, and sales trends. To achieve this, Company XYZ has designed a comprehensive data warehouse schema to store and analyze sales-related information.

### Data Warehouse Design

#### Dimension Tables

1. **dim_product**: 
   - `product_id` (Primary Key): Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category`: Category to which the product belongs.

2. **dim_store**: 
   - `store_id` (Primary Key): Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `state`: State where the store is located.
   - `country`: Country where the store is located.

3. **dim_time**: 
   - `time_id` (Primary Key): Unique identifier for each time record.
   - `date`: Date of the sales transaction.
   - `day_of_week`: Day of the week of the sales transaction.
   - `month`: Month of the sales transaction.
   - `year`: Year of the sales transaction.

4. **dim_sales_name**: 
   - `sales_name_id` (Primary Key): Unique identifier for each salesperson.
   - `sales_name`: Name of the salesperson.
   - `sales_age`: Age of the salesperson.
   - `sales_gender`: Gender of the salesperson.

#### Fact Table

- **fact_sales**: 
   - `sale_id`: Identifier for each sales transaction.
   - `store_id` (Foreign Key): References `dim_store.store_id`.
   - `sales_name_id` (Foreign Key): References `dim_sales_name.sales_name_id`.
   - `time_id` (Foreign Key): References `dim_time.time_id`.
   - `product_id` (Foreign Key): References `dim_product.product_id`.
   - `quantity`: Quantity of the product sold.
   - `price`: Price per unit of the product.

### Schema Description (Star Schema)
![Sales](https://github.com/ikhsannur1996/Sample-Answer-Data-Warehouse-Design-Assignment/assets/32507742/c3e005c9-3b9c-4d5e-be21-a4fc2237dad1)


In a star schema design, the fact table (e.g., `fact_sales`) sits at the center, surrounded by dimension tables (e.g., `dim_product`, `dim_store`). This design simplifies queries and enhances performance by denormalizing data.

**Star Schema:**
In a Star schema, data is organized into a central fact table surrounded by multiple dimension tables, but unlike the Snowflake schema, these dimension tables are not further normalized. Instead, they directly relate to the fact table.

- **Parents**: The central fact table, `fact_sales`, contains transactional data related to sales.
- **Children**: Dimension tables directly connected to the fact table include `dim_product`, `dim_store`, `dim_time`, and `dim_sales_name`. These dimensions provide details such as product information, store details, time-related attributes, and salesperson details respectively.


### Sample Data Mart Tables: Sales Analysis

**1. Product Performance Analysis**

```sql
CREATE TABLE data_mart_product_performance AS
SELECT 
    p.product_id, 
    p.product_name, 
    p.category,
    SUM(f.quantity) AS total_quantity_sold,
    SUM(f.price) AS total_revenue
FROM 
    fact_sales f
JOIN 
    dim_product p ON f.product_id = p.product_id
GROUP BY 
    p.product_id, p.product_name, p.category;
```

**Explanation**: 
This table analyzes sales data to identify top-selling products and product categories.

**2. Store Performance Analysis**

```sql
CREATE TABLE data_mart_store_performance AS
SELECT 
    s.store_id, 
    s.store_name, 
    s.city, 
    s.state, 
    s.country,
    COUNT(f.sale_id) AS total_sales,
    SUM(f.price) AS total_revenue
FROM 
    fact_sales f
JOIN 
    dim_store s ON f.store_id = s.store_id
GROUP BY 
    s.store_id, s.store_name, s.city, s.state, s.country;
```

**Explanation**: 
This table evaluates sales performance across different store locations.

**3. Time-based Sales Analysis**

```sql
CREATE TABLE data_mart_time_analysis AS
SELECT 
    t.year, 
    t.month,
    COUNT(f.sale_id) AS total_sales,
    SUM(f.price) AS total_revenue
FROM 
    fact_sales f
JOIN 
    dim_time t ON f.time_id = t.time_id
GROUP BY 
    t.year, t.month
ORDER BY 
    t.year, t.month;
```

**Explanation**: 
This table identifies sales trends over time, such as monthly sales patterns and seasonal variations.
