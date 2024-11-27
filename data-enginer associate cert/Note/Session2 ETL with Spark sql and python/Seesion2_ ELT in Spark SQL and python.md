# ETL with spark sql and python
## Query from files
Note: It is backticks(``) instead of sigle quote

    Select * from file_formate^.`file_fold\files^`
^: 
- file_formate, can be raw_type(text.binaryFile) or common(csv) 
- files can be single/mutiple(escape character)/dic
## Overwrite table

It is better than drop and recreate
- Time Travel
-  ACID gurantee
- Faster
 1. Option 1:

>     Create or replace table table_name
>     As

 2. Option 2:
 This option will cause mismatch schema. That is the only difference with the last one.

> Insert overwrite table_name select
 

## [Merge table](https://docs.databricks.com/en/sql/language-manual/delta-merge-into.html)

    1Merge into source_table a
    2using target_table b  -- can be view
    3on a.id=b.id 
    4when matched and c.email is null and b.email is not null
    5then update set email=b.email, update=u.update
    6when not matched (by target) insert * 
    7when not matched by source update target.fieldname
  - 6: you can choose to `insert *` or `insert value(xx,xx)`  
- 7: you can use `delete` as well. 
- 7: Note than only Target fiels is avaible. 
- 7: You need have the merge_condition unless the last one
## Adavance transform
 1. Json field format known as *string*
	 * How  to query:
`select profile:firsrt_name,profile:last_name`
	* Transform it into *struct* field type:
	`SELECT profile 	FROM customers 	LIMIT  1`

		 `SELECT customer_id, from_json(profile, schema_of_json('{"first_name":"Thomas","last_name":"Lane","gender":"Male","address":{"street":"06 Boulevard Victor Hugo","city":"Paris","country":"France"}}')) profile_struct`
	* How to query:
	`select profile_struct.a,profile_stryct.b.b1,profile_struct.* `
2. Deal with array
	* `explode()` seperate the array
    `select order_id, explode(books) from orders`
    * `collect_set()` combine into array
	```
			SELECT customer_id,
					collect_set(order_id) AS orders_set,
					collect_set(books.book_id) AS books_set
			FROM orders
			GROUP BY customer_id -- Mandatory
	```
	- `flatten()` out of list of list
	`select flatten(collection_set(explode(book)))`
	- `array_distinct()` make flatten record distinct
	`select array_distinct(flatten(collection_set(explode(book))))`
	- filter(): lambda function, keep the array meet the condition:
	`filter(books,i->i.quantity<2)`
	Note that: you can use the `size()` to filter dataset
	- transform(): lambda function, keep the array meet the condition:
`transform(less_sale,i -> CAST(i.subtotal*0.8 AS int) ) times_ten_sale`
Note: the importance of how to use `CAST` function; It will return element instead of whole array
3. User Define Funcion
`Create or Replace my_function(a a_type)
as returns data_formate
return result `
5. Pivot() just transform
```
	select  *from (
				select  col.quantity, col.book_id
				from tem_book)
	pivot(sum(col.quantity) FOR  col.book_id  in (
		'B01', 'B02', 'B03', 'B04', 'B05', 'B06',
		'B07', 'B08', 'B09', 'B10', 'B11', 'B12'));
```
## Appendix: Questions

 1. [What is the spark sql?](https://www.databricks.com/glossary/what-is-spark-sql)
 query data from RDDs(Resillant Distributed Datasets)
 

