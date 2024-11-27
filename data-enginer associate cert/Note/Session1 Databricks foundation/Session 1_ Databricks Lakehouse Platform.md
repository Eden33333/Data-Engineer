# Lakehouse Platform
## Databricks Architecture
![enter image description here](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*hizwT2iv1lNj6vu510T4ew.png)
cluster also known as computer resources， which lays on the data plane because it will be more privacy to process the data
Lakehouse structure
How it deployed

## Data Lakehouse
### Definition
- The unify of data lake and data warehouse.(make unstructured data ACID)
- It is a concept but delta lake is how it works
### Lakehouse Platform
- Workspace
- Runtime[spark and delta lake](pre-configured set of core components and libraries that are used to run data processing and machine learning tasks on Databricks clusters.)
### DBFS
- Distributed system: allow user to acess multiple data resources like the local storage
- Preinstall in cluster
- Abstraction layer: persist to the underlaying cloud storage

## Delta lake
### Definition
-  Open source(is not proprietary)
- Framework/layer not dataformat
- Bring reliability to data lakes
### Advantage
- Full audit trail for all changes(because of transaction log)
- Scalable metadata(because of transaction log)
- Bring ACID
- Standard dataformat(Json Parquet)

Keep data in **parquet format** and Transaction log in **Json format**. Transaction log:
- Ordered records of every transaction performed on the table(based on what)
- [Single Source of Truth repository](https://www.databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)(aggregate multiple datasource into the single location)
- Json contains(operations, edited files)
## Delta Table
### Definition
- Table Format (others: parquet, csv, Json and Orc)
- a logical representation of Delta Lake
	- Delta lake are built on top of Delta table to ensure all the things happen.
### Feature of Delta Table
#### Time travle
- Query old version: 
`select * from table_name TIMESTAMP AS OF "2023-02-10"`
`select * from table_name Version AS OF 10` or
`select * from table_name@10`
- Return Table:
`Restore Table table_name to timestamp as of`
#### Compacting
```
Optimize data -- Reduce files
zorder by column_name --colocate the related information in the same files
```
#### Vaccum
The default is 7 days
`Vaccum table_name`
If you want to delete all time travel
    %python 
    SET spark.databricks.delta.retentionDurationCheck.enabled =  false

    %sql
    Vacuum table_name retain 0 hours
Note: [when you should vaccum](https://kb.databricks.com/delta/vacuum-best-practices-on-delta-lake)

## Create Schema
`create schema/database
Location 'path/db_x.db'`
Note: `Location` keyword external table(the opposite is managed table). Specify to the file
## Create table
### CRAS
Note: if you use external location, it will be regarded as external table; No `Use keywords`

    create  table  test2
    comment ""
	partition by (city, date)
    location  'dbfs:/mnt/test2'
    select  *  from json.`${dataset.bookstore}/customers-json/export_001.json`

**So how to create a delta table for csv**:
1. create a tempory view
2. Use the view to apply CRAS

 ### Create table(Define Schema)
 Note: no parentheses in `Location`; You can use for create a structure or directly put data(specify the `Location`); When you include `Location`, it must be external data

    Create or replace table table name
    (field1 field1_type, field2 field2_type)
    [Using CSV]
    [Options(header= true, delimiter= ";")]
    [Location =""]

### Create to connec external data source
-- Create an external table connected to Oracle

>  CREATE  TABLE  IF  NOT  EXISTS  ora_tab
  USING  ORACLE
  OPTIONS  (
  url  '<jdbc-url>',
  dbtable  '<table-name>',
  user  '<username>',
  password  '<password>'
);

### Table constraint

    Alter table table_name
    Add Constraint field_name check (date>'')

### Describe

    Describe table table_name
    Describe extended table_name # You can know if it is managed or external
    Describe history table_name # You can change the version by 


## [Clone Delta table](https://docs.databricks.com/en/delta/clone.html#example-clone-syntax)
In both cases change one will not affect the other one
### Deep clone: data+metadat
```
create table table_name
(deep) clone source_table
```
### Shallow clone: transaction log
```
create table table_name
shallow clone source_table (timestamp as of )/version as of
```

## Create Views
Just query
| temp |global  |persist |
|--|--|--|
| Spark Session View* | Cluster Scope |  `Create View view_name As`|
Spark Session:

 1. New workbook
 2. Detaching and reattaching a cluster
 3. Install python
 4. Restart a cluster

## Primary syntax
### Query from files
Note: It is backticks(``) instead of sigle quote

    Select * from file_formate.`file_fold\file.file_format`


 ## Appendix: questions
 1. What is the relationship between lakehouse and delta lake?
 3. What is the relationship of delta table and delta lake?
 4. When do we need partition by
 5. [What is simutaeously read and wirte and others option like update\[**concurrency control**\]?](https://community.databricks.com/t5/administration-architecture/what-happens-when-multiple-people-write-to-the-same-delta-table/td-p/26442)
3. [What is difference between  OLTP and
    OLAP?](https://aws.amazon.com/cn/compare/the-difference-between-olap-and-oltp/)
	- *OLTP -- CRUD   OLAP --aggredation*
 2. [What is Transaction? What is Acid](https://www.databricks.com/glossary/acid-transactions)
	 -  Any operation  Atoms;Consistent;Isolation;Durable

 4. [How to Create table(Clause)](https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-ddl-create-table-using)
 5. The difference between path and location:
 Path is where you find the data, Location is where you store the data. All of them will be regarded as external data
```
     CREATE  table  books_test5
    (book_id STRING, title STRING, author STRING, category STRING, price DOUBLE)
    USING CSV
    OPTIONS (
    path  =  "${dataset.bookstore}/books-csv/export_*.csv",
    header =  "true",
    delimiter =  ";"
    );
```
4. What is the single source of truth repository(SSOT)? What about others(single version of truth)
5. What is the difference between parquet table and delta table? Delta table are built in top of the parquet table but it add Delta Lake which store the metadata and transaction load.
