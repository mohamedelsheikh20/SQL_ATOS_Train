# Update Queries
Here you will find some use cases and SQL code. 
- Analyze the use cases and find more efficient code to enhance the logic.
- Include the improved SQL code for your choice, along with an explanation of why it is better.
---

## Content
1. [Example 1](#Example-1-Identify-the-salesperson-with-the-highest-sales-for-each-quarter-of-the-year-for-the-past-two-years)
   - [ex 1 old sql.](#ex-1-old-SQL-code)
   - [ex 1  new sql.](#ex-1-new-SQL-code)
   - [ex 1  Updates.](#Example-1-updates)

2. [Example 2](#Example-2-Identifying-the-top-5-salespeople-based-on-total-sales-for-the-year)
   - [ex 2 old sql.](#ex-2-old-SQL-code)
   - [ex 2  new sql.](#ex-2-new-SQL-code)
   - [ex 2  Updates.](#Example-2-updates)

3. [Example 3](#Example-3-Return-a-list-of-all-the-employees-in-those-departments)
   - [ex 3 old sql.](#ex-3-old-SQL-code)
   - [ex 3  new sql.](#ex-3-new-SQL-code)
   - [ex 3  Updates.](#Example-3-updates)

4. [Example 4](#Example-4-Find-Duplicate-Rows)
   - [ex 4 old sql.](#ex-4-old-SQL-code)
   - [ex 4  new sql.](#ex-4-new-SQL-code)
   - [ex 4  Updates.](#Example-4-updates)

5. [Example 5](#Example-5-Compute-a-Year-Over-Year-Difference)
   - [ex 5 old sql.](#ex-5-old-SQL-code)
   - [ex 5  new sql.](#ex-5-new-SQL-code)
   - [ex 5  Updates.](#Example-5-updates)

6. [Example 6](#Example-6-Return-the-MAX-sales-and-2nd-max-sales)
   - [ex 6 old sql.](#ex-6-old-SQL-code)
   - [ex 6  new sql.](#ex-6-new-SQL-code)
   - [ex 6  Updates.](#Example-6-updates)

7. [Example 7](#Example-7)
   - [ex 7 old sql.](#ex-7-old-SQL-code)
   - [ex 7  new sql.](#ex-7-new-SQL-code)
   - [ex 7  Updates.](#Example-7-updates)
---


## Example 1: Identify the salesperson with the highest sales for each quarter of the year for the past two years.

### Ex 1: old SQL code
```
SELECT
    sp.salesperson_id,
    q.quarter_name,
    SUM(t.order_amount) AS total_sales
FROM sales_transactions AS t
JOIN salespeople AS sp ON t.salesperson_id = sp.salesperson_id
JOIN quarters AS q ON t.transaction_date BETWEEN q.start_date AND q.end_date
GROUP BY
    sp.salesperson_id,
    q.quarter_name
```

### Ex 1: new SQL code
```
with 

-- past 2 years data
past_two_years_sales as (
    select
        
        salesperson_id,
        transaction_date,
        order_amount,
        extract(year from sales_transactions.transaction_date) as transaction_year, 
        year(getdate()) AS current_year
        
    from sales_transactions
    where 
        transaction_year = current_year - 1 or  -- past year
        transaction_year = current_year - 2     -- past second year
),

-- sum per each quarter
sum_per_each_querter as (
    
    select 
    
        quarters.quarter_name,
        past_two_years_sales.salesperson_id,
        sum(past_two_years_sales.order_amount) as total_sales
    
    from past_two_years_sales
    
    join quarters on 
        past_two_years_sales.transaction_date between quarters.start_date and quarters.end_date
        
    group by
        quarters.quarter_name,
        past_two_years_sales.salesperson_id
),

-- max seller per each quarter
max_sales_per_quarter as (

    select
    
        quarter_name,
        max(total_sales) as max_total_sales
    
    from sum_per_each_querter

    group by
        quarter_name
)

-- join to get the id of person who has the max sales amount
select

    max_sales_per_quarter.quarter_name,
    sum_per_each_querter.salesperson_id,
    max_sales_per_quarter.max_total_sales

from max_sales_per_quarter
join sum_per_each_querter on 
    max_sales_per_quarter.quarter_name = sum_per_each_querter.quarter_name and
    max_sales_per_quarter.max_total_sales = sum_per_each_querter.total_sales
```

### **Example 1: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* there is no need to join and group by `salespeople` because we use only the id (in this business case).
* get the past 2 years only from the current date year.
* get sum of sales for each quarter for each sales person.
* get the max sales per each quarter.
* join with the sum of sales to get the sales person id.
---


## Example 2: Identifying the top 5 salespeople based on total sales for the year.

### Ex 2: old SQL code
```
SELECT
    sp.salesperson_id,
    SUM(t.order_amount) AS total_sales
FROM sales_transactions AS t
JOIN salespeople AS sp ON t.salesperson_id = sp.salesperson_id
GROUP BY
    sp.salesperson_id
ORDER BY total_sales DESC
LIMIT 5
```

### Ex 2: new SQL code
```
with 

-- get the year data
year_data as (
    select
    
        salesperson_id,
        order_amount,
        extract(year from transaction_date) as order_year,
        year(getdate()) AS current_year
    
    from sales_transactions
    
    where transaction_date = current_year - 1

),

-- get the top 5 sales person
top_five_sales_person as (
    select
    
        salesperson_id,
        sum(order_amount) as total_sales_per_person
    
    from year_data
    group by salesperson_id
    
    order by total_sales_per_person desc
    limit 5

)

select * from top_five_sales_person
```

### **Example 2: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* there is no need to join and group by `salespeople` because we use only the id (in this business case).
* get the year data (current = -0), (past = -1), etc..
* select the top 5 sales persons using `limit 5`.
---


## Example 3: Return a list of all the employees in those departments.

### Ex 3: old SQL code
```
SELECT 
  first_name,
  last_name
FROM employee e1
WHERE department_id IN (
   SELECT department_id
   FROM department
   WHERE manager_name=‘John Smith’)
```

### Ex 3: new SQL code
```
select 

  employee.first_name,
  employee.last_name

from employee

join department on
    employee.department_id = department.department_id

where 
    department.manager_name=‘John Smith’
```

### **Example 3: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* join department instead of making a sub query, with adding the same `where` statement.
---


## Example 4: Find Duplicate Rows

### Ex 4: old SQL code
```
SELECT 
  employee_id,
  last_name,
  first_name,
  dept_id,
  manager_id,
  salary
FROM employee
GROUP BY   
  employee_id,
  last_name,
  first_name,
  dept_id,
  manager_id,
  salary
HAVING COUNT(*) > 1
```

### Ex 4: new SQL code
```
select distinct

  employee_id,
  last_name,
  first_name,
  dept_id,
  manager_id,
  salary

from employee
```

### **Example 4: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* select distinct values only to remove all duplicates.
---


## Example 5: Compute a Year-Over-Year Difference
-  The main query should return the year, year_amount, revenue_previous_year, Year-Over-Year_diff_value,
        and Year-Over-Year_diff_perc(%) from the year_metrics

- Equations:
  - Year-over-year difference = Value Current Year – Value Last Year
  - Year-over-year percentage `Growth` = ((Value Current Year – Value Last Year) / Value Last Year) x 100

### Ex 5: old SQL code
```
SELECT
    extract(year from day) as year,
    SUM(daily_amount) as year_amount
  FROM sales
  GROUP BY year
```

### Ex 5: new SQL code
```
data_year_amount as (

    select
    
        extract(year from day) as year,
        sum(daily_amount) as year_amount
    
      from sales
      
      group by year

),

previous_year_amount as (

    select
    
        year,
        year_amount,
        lag(year_amount) over (
            order by year
        ) as previous_year_amount
    
    from data_year_amount

)

select
    
    year,
    year_amount,
    (year_amount - previous_year_amount) as Year_Over_Year_diff_value
    ((year_amount - previous_year_amount) / previous_year_amount) * 100 as Year_over_year_percentage


from previous_year_amount
```

### **Example 5: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* add CTE to get the amount for each year.
* add CTE with new column which have the value of the previous year.
* depending on the equations of `Year_Over_Year_diff_value` and `Year_over_year_percentage` add new columns.
---


## Example 6: Return the MAX sales and 2nd max sales

### Ex 6: old SQL code
```
SELECT
(SELECT MAX(sales) FROM Sales_orders) max_sales,
(SELECT MAX(sales) FROM Sales_orders
WHERE sales NOT IN (SELECT MAX(sales) FROM Sales_orders )) as 2ND_max_sales;
```

### Ex 6: new SQL code
```
with 

get_max_2_sales as (
    select
    
        sales
    
    from Sales_orders
    order by sales desc
    limit 2
)


select
    (select max(sales) from get_max_2_sales) as max_sales,
    (select min(sales) from get_max_2_sales) as second_max_sales

```

### **Example 6: updates**
* lower all statement and improve the structure (spaces, removing leading words like `as X`)
* get the max 2 values from the table.
* get the max one and the remain one will be the min (because there are only 2 values).

> depending on business case output we can use or add window function.
---


## Example 7:
- This query will select the `Essns` of all employees who work the same
(project, hours) combination on same project that employee ‘John Smith’ (whose Ssn =‘123456789’) works on.

- from this query we can assume that:
  - project table has : Dnum (department number) - Pnumber (project number).
  - department table has: Dnumber (department number) - mgr_ssn (manager ss number).
  - employee table has: Ssn (ss number) - Fname (first name) - Lname (last name).
  - works_on table has: Essn (employee ss number) - Pno (project number).

### Ex 7: old SQL code
```
SELECT DISTINCT Pnumber
FROM PROJECT
WHERE Pnumber IN  
( SELECT Pnumber
FROM PROJECT, DEPARTMENT, EMPLOYEE
WHERE Dnum=Dnumber AND
Mgr_ssn=Ssn AND Lname=‘Smith’ )
OR
Pnumber IN
( SELECT Pno
FROM WORKS_ON, EMPLOYEE
WHERE Essn=Ssn AND Lname=‘Smith’ )
```

### Ex 7: new SQL code
```
with

-- get all projects where smith manage it's department 
john_smith_manage as (

    select distinct
    
        project.Pnumber
    
    from project
  
    join department on 
        project.Dnum = department.Dnumber
        
    join employee on 
        department.mgr_ssn = employee.Ssn
    
    where employee.Ssn = 123456789
 
),

-- get all projects where smith worked in 
john_smith_projects as (

    select distinct
      
        works_on.Pno as Pnumber
  
    from works_on
      
    join employee on 
        works_on.Essn = employee.Ssn

    where employee.Ssn = 123456789
    
),

-- union data (using union to remove duplicates)
smith_departments_projects as (

    select
        *
    from john_smith_manage
    
    union

    select
        *
    from john_smith_projects
    
)

select distinct

    works_on.Essn

from smith_departments_projects

join works_on on
    works_on.Pno = smith_departments_projects.Pnumber
```

### **Example 7: updates**

* lower all statement and improve the structure (spaces, removing leading words like `as X`).
* as shown above I have assumed the tables and what it contains.
* CTEs:
  1. get all projects where `John Smith` manage its department, where his Ssn = `123456789` because it is a unique number.
  2. get all projects where smith worked in.
  3. union data to get all project numbers which `smith` manage and work in.
  4. join the output of union with the `works_on` table to get all employees who worked in the same projects.
---
