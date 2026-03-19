---
layout: post
title: Database Fundamentals
date: '2026-03-18 20:06:05 -0700'
description: >-
  What you need to know about data modelling
author: bricecny
date: '2026-03-12 17:39:56 -0700'
categories: [Data Engineering, Data Modelling]
---

## DB Access pattern - WHAT do you want to do
- **OLTP** (Online Transaction Processing): a lot of small read/writes
- **OLAP** (Online Analytical Processing): designed for large, complex queries over a lot of data. Scanning M+ rows and aggregating.

## DB Types - HOW do you want to do it
- **Relational DB**
	- Row-oriented (MySQL) better for OLTP
	- Column-oriented/distributed (Redshift, Snowflake) better for OLAP-> do a graph for this with library analogy
- **NoSQL** (MongoDB)
	- data are linked with nodes and edges

|                    | Relational DB                     | NoSQL                              |
| ------------------ | --------------------------------- | ---------------------------------- |
| Data schema        | - Needs to be known in advance    | + flexible                         |
| Data integrity     | + Easy with schema + ACID         | - weak consistency because of BASE |
| Data availability  |                                   | + Easy                             |
| Horizontal scaling | - Difficult                       | + Easy                             |
| Vertical scaling   | + Easy                            |                                    |
| Querying data      | + use of SQL widespread across DB | - can be difficult depending on DB |

### Properties of a DB transaction
**ACID Framework** (Relational DB)
- **Atomicity**: a transaction is a smallest unit of change, it happens or not and there is nothing in between which would affect data integrity
- **Consistency**: checks that DB is consistent before and after a transaction
- **Isolation**: a transaction occurs in isolation i.e. independently from each others so there can be concurrency
- **Durability**: once transaction completed, the DB is properly updated forever

**BASE Model** (NoSQL):
- **Basically Available**: there will be a response to every request but not always consistent
- **Soft state**: System's state can change even without input, this is due to the eventual consistency
- **Eventual consistency**: data will eventually converge to a consistent state but no guarantee of when

## Relational DB
### Keys and normalization
A table can have keys:
- **Primary key**:
	- uniquely identify data in the table
	- it's an index so it's optimized for filtering
	- A good PK has:
		- stability: does not change over time
		- uniqueness: self-explanatory
		- irreducibility: no subset of the key is itself a PK
- **Foreign key**: 
	- It's linked to a PK in another table
	- benefit is to store the data only once and only make references to it + data integrity
	- between 2 tables PK->FK is a one-to-many relationship
### Storage vs performance trade-off:
- Normalization = separate data in multiple tables to optimize storage and prevent redundancy
- Makes read times more expensive as we need to now use `JOINS`, might want to de-normalize 
### Keys and performance

![libray_analogy](/assets/img/posts/library_analogy.jpeg){: width="600" height="300" .w-75 .normal}


- Row-oriented
	- **Index**: data structure (object) maintained alongside a table to speed up data retrieval, efficient for filtering
- Column-oriented/distributed
	- **Distribution key**: for each value of the col (hash value), data will live on a given node. For joins this is efficient if the 2 cols where used for the distribution, rows will be co-located.
	- **Partition key**: splits data into sub-tables or chunks, efficient for filtering (partition pruning: some partitions are not even scanned)
	- **Sort key**: sort data on disk, efficient for filtering (skips chunks based on zone map i.e. min-max metadata)
	- index less relevant

## Scaling DB
- **vertical scaling**: scaling "up" means making the servers bigger (CPU+RAM). Has a ceiling and can be expensive
- **horizontal scaling**: scaling "out" , add more machines (nodes). Cheaper but has challenges related to distributed computing e.g. consistency
	- **sharding**: database rows are split between nodes using a sharding mechanism to decide which node. This allows better write performance.


## Distributed DB

### Properties of a distributed DB
**CAP Theorem**, pick any two out of the 3
- **Consistency**: every user see the same data
- **Availability**: the system is always available, always returns a non-error response but the data might not be the latest
- **Partition** **tolerance**: the system functions even if communications between the partition is delayed or lost.
P is not something that can avoided so the real trade off is CP and AP:
- CP: consistency = fails loudly by returning no result at all 
- AP: availability = fail gracefully by returning something even though stale

### Distributed computing
What: MapReduce algorithm
- Split: split data into chunks and distribute them on worker nodes
- Map: each node applies a mapping function to output (key, value) pairs
- Shuffle: secret sauce, reshuffle the data and distribute it based on keys. Only intra node communication
- Reduce: aggregate the pairs

How:
- Hadoop: relies on HDFS + MapReduce + YARN (scheduler+monitoring)
- Spark: open source, RAM calculation (expensive), built-in scheduler+monitoring

