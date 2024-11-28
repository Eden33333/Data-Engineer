
## Delta Live Table(DLT)
Note: You can not search the table directly from the notebook. You can use it after you bluid a pipeline
- Workflows: Storage_Path, Target Schema,Configure
- We can join the live tables in different notebook
### Cluster
- continuous vs trigger
- Develop(validation 2 hours) vs Prod
### Storage
- You specify the location
	- path/tables ## You live table
	- path/system ## event log delta table 
- Metastore: find the schema
	- You can find in the graph interface
### Syntax
live table which is used in the pipeline:
```
create or refresh (streaming) live table
As
```
materilized view: precompute
```
create or refresh materialized view
As
```
### Data quality
[Constraint Violation](https://docs.databricks.com/en/delta-live-tables/expectations.html#language-sql)
- Drop load
- Fail Update
- Omit(warn default)

## Create Orchestrate jobs
1. Rerun the fail job
## Databricks Sql

## [Change Data Capture]
- [Apply changes into live_table](https://docs.databricks.com/en/delta-live-tables/cdc.html#example-scd-type-1-and-scd-type-2-processing-with-cdf-source-data&language-sql)
```

-- Create and populate the target table.
CREATE  OR  REFRESH  STREAMING  TABLE  target;

[APPLY  CHANGES  INTO](https://docs.databricks.com/en/delta-live-tables/cdc.html#example-scd-type-1-and-scd-type-2-processing-with-cdf-source-data&language-sql)
  live.target
FROM
  stream(cdc_data.users)
KEYS
  (userId)
APPLY  AS  DELETE  WHEN
  operation  =  "DELETE"
APPLY  AS  TRUNCATE  WHEN
  operation  =  "TRUNCATE"
SEQUENCE  BY
  sequenceNum
COLUMNS  *  EXCEPT
  (operation,  sequenceNum)
STORED  AS
  SCD  TYPE  1;
```
prons:
1. Support applying changes as SCD Type1(default,overwriet) or SCD Type2(new records)
Cons:
2. Not stream table anymore
- [Apply changes into snapshot](https://www.databricks.com/blog/how-perform-change-data-capture-cdc-full-table-snapshots-using-delta-live-tables)


## Appendix: questions
1. When I should use `cloud_files(location,data_format,map())`
2. How to create [streaming table](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-streaming-table.html)?
3. What is the difference between streaming table and [structured stream](https://www.databricks.com/glossary/what-is-structured-streaming#:~:text=Structured%20Streaming%20is%20a%20high,them%20in%20a%20streaming%20fashion.)?[the former is the logit model for the later; Structured stream is [a high level API](https://stackoverflow.com/questions/30897001/what-is-the-difference-between-a-high-level-and-low-level-java-api)(concept)]
4. [Streaming Table; Materialized View; View](https://docs.databricks.com/en/delta-live-tables/index.html#dlt-streaming-table) https://www.hph93.com/2024/01/22/streaming-tables-vs-materialized-view/
5. [What is the difference between prod/develop and trigger and continue pipepline?](A%20Delta%20Live%20Table%20pipeline%20includes%20two%20datasets%20defined%20using%20STREAMING%20LIVE%20TABLE.%20Three%20datasets%20are%20defined%20against%20Delta%20Lake%20table%20sources%20using%20LIVE%20TABLE.%20The%20table%20is%20configured%20to%20run%20in%20Development%20mode%20using%20the%20Triggered%20Pipeline%20Mode.%20Assuming%20previously%20unprocessed%20data%20exists%20and%20all%20definitions%20are%20valid,%20what%20is%20the%20expected%20outcome%20after%20clicking%20Start%20to%20update%20the%20pipeline?A.%20All%20datasets%20will%20be%20updated%20once%20and%20the%20pipeline%20will%20shut%20down.%20The%20compute%20resources%20will%20be%20terminated.%20B.%20All%20datasets%20will%20be%20updated%20at%20set%20intervals%20until%20the%20pipeline%20is%20shut%20down.%20The%20compute%20resources%20will%20be%20deployed%20for%20the%20update%20and%20terminated%20when%20the%20pipeline%20is%20stopped.%20C.%20All%20datasets%20will%20be%20updated%20at%20set%20intervals%20until%20the%20pipeline%20is%20shut%20down.%20The%20compute%20resources%20will%20persist%20after%20the%20pipeline%20is%20stopped%20to%20allow%20for%20additional%20testing.%20D.%20All%20datasets%20will%20be%20updated%20once%20and%20the%20pipeline%20will%20shut%20down.%20The%20compute%20resources%20will%20persist%20to%20allow%20for%20additional%20testing.%20E.%20All%20datasets%20will%20be%20updated%20continuously%20and%20the%20pipeline%20will%20not%20shut%20down.%20The%20compute%20resources%20will%20persist%20with%20the%20pipeline.)
	- In summary,  the delay time for develop is 2 hours for testing but for product is 0s. You can reset using Json configuration 
	- `{"configuration":  {
  "pipelines.clusterShutdown.delay":  "60s"}}`
  - You need to set continue pipeline in Json or like the picture. [Here is the main different](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/pipeline-mode)
  `{
  "configuration": {
    "pipelines.trigger.interval": "10 seconds"
  }
}`
6. [The difference between `create live table` and `create streaming live table`](https://stackoverflow.com/questions/72773159/difference-between-live-table-and-streaming-live-table)
7. What is the difference between `partition` and `zorder`?
	- `Partition by` make data into small chunk, suitable for low cardinality columns
	- `zorder` use to data skipping, the goal is to touch as low as data set. suitable for high cardinality columns[high volumn of distinguish value]

