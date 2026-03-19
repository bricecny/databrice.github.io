---
layout: post
title: SQL Fundamentals
description: >-
  SQL Cheatsheet covering main functions in PostgreSQL
author: bricecny
date: '2026-03-12 17:39:56 -0700'
categories: [Coding, SQL]
---

## SQL DDL (Data Definition Language) commands
- ```
  CREATE TABLE {schema_name}.{table_name} (
	  {col_name} {data_type} {constraint?}
	  )
  ```
- `ALTER`: modify existing object e.g. to add and drop a column to a table
	- `ALTER TABLE {table_name} ADD {col_name} {data_type}`
	- `ALTER TABLE {table_name} DROP COLUMN {col_name}`
- `DROP TABLE/INDEX/DATABASE {table/index/database_name}`: delete object

## SQL DML (Data Modification Language) commands
### WRITE
- `INSERT INTO {table_name} ({[col_names]}) VALUES ({[values]})`: no need to specify columns if you populate all fields.
- `UPDATE {table_name} SET {col_name}={value}, {col_name2}={value2} WHERE {condition}`
- `DELETE FROM {table_name} WHERE {condition}`: deletes entire rows
### READ
- `SELECT {[col_names]} FROM {tables} WHERE {...} GROUP BY {...} HAVING {...} ORDER BY {...} LIMIT {...}`: read data from DB
#### Order of operations
`SELECT` is the odd one here, it's 1st in order of writing but 5th in order of operation
1. `FROM`: use joins to reduce rows returned
2. `WHERE {conditions}`: use filters to reduce rows returned 
3. `GROUP BY`: before `SELECT`meaning you "usually" cannot use column alias, you can use indexes to refer to columns in the `SELECT` e.g. 1,2,3
4. `HAVING {conditions}`: before `SELECT` meaning you "usually" cannot use column alias and need to retype column logic, can use filters on aggregated rows
5. `SELECT`: use columns to reduce rows returns. `DISTINCT` to remove duplicates in the result set
6. `ORDER BY`: 
7. `LIMIT`: use limit to reduce rows returned. `OFFSET n` to skip some leading rows 

- CTEs and subqueries have the same performance:
	- use CTEs if you need to reuse the logic multiple times
	- use subqueries for short inline logic for readability

### Conditional Logic

```
CASE 
	WHEN {condition} THEN {value}
	WHEN {condition} THEN {value}
ELSE {value} END AS {col_name}
```
- Other DB can use `IF({condition}, {valueTRUE}, {valueFALSE})`

### Aggregations functions
The result set in smaller than the selected set, with cardinality the product of unique number of values of each columns in the group by
- `COUNT(...), COUNT(DISTINCT ...)`
- `SUM(...)`

- `STRING_AGG(..., ',' ORDER BY ...)`
### Window functions
The result set is exactly the same size as the selected set.
- Compare with other rows
	- `LEAD(...), LEAD(...,n) OVER (PARTITION BY {[col_names]} ORDER BY {[col_names]})` 
	- `LAG(...)`

- Ranking
	- `ROW_NUMBER()`: if there is a tie, the results are non deterministic
	- `RANK()`: skip numbers when there are ties e.g. 1,2,2,4
	- `DENSE_RANK()`: do not skip numbers when there are ties e.g. you can have 1,2,2,3

You can select a moving window by using  a frame clause `ROWS BETWEEN {frame_start} AND {frame_end}` inside the partition. Placeholders can be any of:
- `n/UNBOUNDED PRECEDING/FOLLOWING`
- `CURRENT ROW`

### Type-specific functions
#### Math
- `MOD(p,q)=r`: returns the reminder of p/q 
- `GREATEST/LEAST({[col_names]})`: return the largest/smallest of 2 values from a row
- `FLOOR(n), CEILING(n)`: rounds a number to the immediate smaller/larger integer
### Date
- `TO_CHAR({date_col}, {str_format})`: converts a date to a specific string format
- `{date_col}::DATE + INTERVAL '10 days'`: date arithmetic. make sure the {date_col} is cast as a date

### String
- `UPPER(...), LOWER(...), INITCAP(...)`: play with capitalization 
- `RTRIM(...), LTRIM(...)`: trim whitespaces
- `LENGTH(...)`
- `SUBSTRING(..., {start_index}, {number_char})`: extract a substring, 1-indexed
- `ILIKE/LIKE '{regex-like pattern}'`: case in/sensitive string pattern matching
	- `%`: match any number of characters
	- `_`: match any 1 character
	- can escape the previous 2 characters with `\\` or specify another way with `ESCAPE '{char}'

### Set operations
These are all made on axis=0 (the opposite of a join)
- `UNION (ALL)` : concatenate 2 sets along axis=1 (use ALL to keep duplicates)
- `EXCEPT`: also called minus, remove set 2 records from set 1
- `INTERSECT`: keeps only the rows that overlap on in the 2 sets