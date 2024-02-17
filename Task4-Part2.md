## old quert
```
SELECT 
"Purchase Transaction ID", 
"Purchase_DATEID",
"Ship Mode", 
"Segment",
"Region",
"Category",
"Sub-Category",
"Type",
"Sales", 
"Quantity",
"Discount", 
"Profit"
FROM fct_SampleSuperstore_Purchase
WHERE 
"Type" IN ('Second Class - Consumer', 'Standard Class - Consumer','First Class - Consumer','Same Day - Consumer')
and Purchase_DATEID IN 
('2022-12-01',
'2022-11-01',
'2022-01-01' ,
'2022-10-01' ,
'2022-06-01' ,
'2022-04-01' ,
'2022-09-01' ,
'2022-02-01' ,
'2022-07-01' ,
'2022-05-01' ,
'2022-08-01' ,
'2022-03-01')
```

## new query
```
with

-- this cte must be in the stage phase or update the main table at the begining (depending on our schema)
-- alter the old table with the new structure then update it with the new data
simplify_main_fct as (

	select
        -- old columns
        purchase_transaction_id,
        ship_mode,
        region,
        category,
        "Sub-Category",
        sales,
        quantity,
        discount,
        profit,

        -- new type split columns
		split_part(Type, ' - ', 1) as shipment_type,
		split_part(Type, ' - ', -1) as customer_type,

        -- new date split columns
        day(purchase_dateid) as purchase_day,
        month(purchase_dateid) as purchase_month,
        year(purchase_dateid) as purchase_year
		
	from fct_SampleSuperstore_Purchase

),

-- old query filters
filters as (

    select
    
        *
    
    from simplify_main_fct

    where 
        customer_type = 'Consumer' and
        purchase_day = 1 and
        purchase_year = 2022
    
)

SELECT * from filters

```

