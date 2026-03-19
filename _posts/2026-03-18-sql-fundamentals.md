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

```sql
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


![sql_functions_taxonomy](/assets/img/posts/sql_functions_taxonomy.jpeg){: width="600" height="300" .w-75 .normal}

### Predicate functions → returns TRUE / FALSE
#### Set
- `IN (...)`: provide a set of values
#### Existence
- `EXISTS` runs a subquery linked to the active row and check whether at least one row is returned and short circuits as soon as it finds one
```sql
SELECT
FROM table1 t1
WHERE EXISTS (SELECT 1 FROM table2 t2 WHERE t1.id = t2.id)
```
#### Comparison 
- `<>, =, <, >, BETWEEN`
- `ILIKE/LIKE '{regex-like pattern}'`: case in/sensitive string pattern matching
	- `%`: match any number of characters
	- `_`: match any 1 character
	- can escape the previous 2 characters with `\\` or specify another way with `ESCAPE '{char}'`
### Aggregations functions → N input rows → 1 output value (collapses rows)
#### Regular
The result set in smaller than the selected set, with cardinality the product of unique number of values of each columns in the group by
- `COUNT(...), COUNT(DISTINCT ...)`
- `SUM(...)`

- `STRING_AGG(..., ',' ORDER BY ...)`

#### Ordered-set aggregate functions
Similar to aggregation function but the order within the group matters to compute the result.
- `PERCENTILE_DISC(0.9) WITHIN GROUP (ORBER BY ...)`

### Window functions → N input rows → N output values (keeps rows)
The result set is exactly the same size as the selected set.
You can select a moving window by using  a frame clause `ROWS BETWEEN {frame_start} AND {frame_end}` inside the partition. Placeholders can be any of:
- `n/UNBOUNDED PRECEDING/FOLLOWING`
- `CURRENT ROW`

#### Navigation
- Compare with other rows
	- `LEAD(...), LEAD(...,n) OVER (PARTITION BY {[col_names]} ORDER BY {[col_names]})` 
	- `LAG(...)`
#### Ranking
- `ROW_NUMBER()`: if there is a tie, the results are non deterministic
- `RANK()`: skip numbers when there are ties e.g. 1,2,2,4
- `DENSE_RANK()`: do not skip numbers when there are ties e.g. you can have 1,2,2,3
#### Regular
- `SUM(...) OVER (PARTITION BY ...)`

### Scalar functions → 1 input row → 1 output value
#### Math
- `MOD(p,q) = p % q = r`: returns the reminder of p/q 
- `GREATEST/LEAST({[col_names]})`: return the largest/smallest of 2 values from a row
- `FLOOR(n), CEILING(n)`: rounds a number to the immediate smaller/larger integer
#### Date
- `TO_CHAR({date_col}, {str_format})`: converts a date to a specific string format
- `{date_col}::DATE + INTERVAL '10 days'`: date arithmetic. make sure the {date_col} is cast as a date

#### String
- `UPPER(...), LOWER(...), INITCAP(...)`: play with capitalization 
- `RTRIM(...), LTRIM(...)`: trim whitespaces
- `LENGTH(...)`
- `SUBSTRING(..., {start_index}, {number_char})`: extract a substring, 1-indexed

#### Conditional

```sql
CASE 
	WHEN {condition} THEN {value}
	WHEN {condition} THEN {value}
ELSE {value} END AS {col_name}
```
- Other DB can use `IF({condition}, {valueTRUE}, {valueFALSE})`
### Set operations
These are all made on axis=0 (the opposite of a join)
- `UNION (ALL)` : concatenate 2 sets along axis=1 (use ALL to keep duplicates)
- `EXCEPT`: also called minus, remove set 2 records from set 1
- `INTERSECT`: keeps only the rows that overlap on in the 2 sets