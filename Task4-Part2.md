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

    -- using string methods
    Type like '% - Consumer'  and 
    
    -- if the date was string
    -- Purchase_DATEID like '1/%/2022'
    
    -- using date methods (task3 ex 1 of snowflake)
    year(Purchase_DATEID) = 2022 and day(Purchase_DATEID) = 1

```

