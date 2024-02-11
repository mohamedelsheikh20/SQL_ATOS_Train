# Update Queries
Here you will find some use cases and SQL code. 
- Analyze the use cases and find more efficient code to enhance the logic.
- Include the improved SQL code for your choice, along with an explanation of why it is better.
---


## Part1

### Dataset:

* Table: sales
* Columns: product_id, customer_id, sale_date, amount, quantity


### 1.  Identify the most popular products in terms of sales quantity during a specific timeframe.
- Finding top N products sold within a specific dates:

#### old sql:
```
SELECT *
FROM sales
WHERE sale_date BETWEEN '2023-01-01' AND '2023-02-07'
ORDER BY quantity DESC
LIMIT 10;
```


#### new sql:
```
with 

get_top_product_sold as (
	select

		*,
		-- assuming that the product_id is exists many times
		sum(quantity) as sum_quantity,
		dense_rank() OVER (ORDER by sum(quantity) DESC) as product_rnk
		
	from sales
	where sale_date between '2023-01-01' and '2023-02-07'
	group by product_id
)

select 
	*
from get_top_product_sold
where product_rnk <= 10;  -- or replace 10 with any number `N`
```

#### justification:

1. assume that there is repeated product ids.
2. get the sum of quantity per product.
3. dense rank this sum.
4. add `where` to get the top N products.
------------------------------------------------------------------------------------------------------------------------------------------------


### 2. Calculating cumulative sales by product category:
#### old sql:
```
  SELECT product_category, total_sales
  FROM sales
  ORDER BY product_category, sale_date
```

#### new sql:
```
SELECT 

	product_category, 
	amount,
	sum(amount) over (PARTITION by product_category order by sale_date) as total_sales,
	sale_date

FROM sales
```


#### justification:

1. cumulative mean `sum per time`.
2. get the sum per `product_category` over time `sale_date`.
------------------------------------------------------------------------------------------------------------------------------------------------
### 3. Identifying customers with the highest average order value within a region:
#### old sql:
```
SELECT customer_id, region , amount
    FROM sales
```


#### new sql:
```
with

customers_avg_per_region as (
	SELECT 
		customer_id, 
		region, 
		avg(amount) as avg_amount_per_cust		
	FROM sales
	group by region, customer_id
)

select 
	region,
	customer_id,  
	max(avg_amount_per_cust) as max_avg_per_customer

from customers_avg_per_region
group by region
```

#### justification:

1. first CTE to get the avg sales per customers in each region.
2. finally we get the max of the avg.
------------------------------------------------------------------------------------------------------------------------------------------------
### 4. Finding products with the biggest year-over-year sales growth:
#### old sql:
```
SELECT product_id,amount, sale_date
FROM sales;
```


#### new sql:
```
-- year over year growth ((Value Current Year – Value Last Year) / Value Last Year) x 100

with 

-- for each year get sum of sales for each product
sales_products_per_year as (

    SELECT 
    
    	product_id,
    	sum(amount) as amount_per_product,
    	extract(year from sale_date) as year_date
    
    FROM sales
    group by year_date, product_id
),

-- get the previous year amount value then calculate the equation 
add_sales_per_previous_year as (
    select 
    
        *,
        lag(amount_per_product) over (PARTITION by product_id order by year_date) as previous_year_amount,

        -- using equation year over year growth = ((Value Current Year – Value Last Year) / Value Last Year) x 100
        ((amount_per_product - previous_year_amount) / previous_year_amount) * 100 as YOY_Growth
    
    from sales_products_per_year
)

-- get the max products using rank of window function
select

    product_id,
    YOY_Growth,
    dense_rank() over (order by YOY_Growth desc) as max_yoy_growth
    
from add_sales_per_previous_year
where YOY_Growth is not null  -- null will be number 1 (the biggest value) so I removed nulls
```

#### justification:

1. for each year get sum of sales for each product.
2. using `lag` get the previous year amount value per product then per year (each product and amount per all years).
3. using previous `lag` and the amount value get the `yoy` using the equation `((Value Current Year – Value Last Year) / Value Last Year) x 100`.
4. Finally, use window function with dense rank to rank the biggest products growth.
------------------------------------------------------------------------------------------------------------------------------------------------


## Part 2: Those queries have errors. Please identify the errors and write the correct queries.

- Here are some examples of queries with errors related to window functions:


1. 
```
SELECT *, RANK() OVER (PARTITION BY customer_id) AS rank
FROM sales
ORDER BY rank DESC;
```

* rank in window function require an order by method.
* assuming that we rank amounts per each customer id.

```
SELECT 
    *,
    RANK() OVER (PARTITION BY customer_id order by amount) AS rank
FROM sales
ORDER BY rank DESC;
```
---

2.

```
SELECT *, SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales;
```

* Cumulative sum per all dates ordered.
* to make it more specific we can assume that per each customer get the cumulative amount depending on time.

```
SELECT 
-- 	*,
	sale_date, amount, customer_id,
	SUM(amount) OVER (PARTITION by customer_id ORDER BY sale_date) AS running_total
FROM sales;
```
---

3.
```

SELECT AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount,
       COUNT(*) FROM sales;
```

* count(*) is giving us one value = count the rows `raise error`.
* create a window function to count the number of rows.

```
SELECT 
	*,
    AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount,
	COUNT(*) OVER () AS total_rows_number
   
FROM sales;
```
---

4.
```
SELECT *, ROW_NUMBER() OVER (ORDER BY sale_date) AS row_num,
       ROW_NUMBER() OVER (ORDER BY amount) AS another_row_num
FROM sales;
```

* the query working well.
* I don't know what you need to change here ??

```
SELECT 
    *,
    ROW_NUMBER() OVER (ORDER BY sale_date) AS row_num_date,
    ROW_NUMBER() OVER (ORDER BY amount) AS row_num_amount
FROM sales;
```
---