## RDD (Resilient Distributed Dataset)
- fundamental abstraction of spark
- data is distributed across storage
- properties
	- distributed
	- resilient and immutable
	- compile-time type-safety
	- Unstructured/Structured Data and ability parse them
	- lazy
- on this abstraction we can write our lambda fns and queries
	- queries are executed in parallel on partitions
- rdds are abstractions of the distributed data across the cluster
- resilient means that ability to recreate the RDD at any point in time
	- because data lineage is captured
	- lineage is a acyclic graph
- we have RDDs of a particular type: ex- Integer RDD, String or Text Rdd
- we can parse the data within RDD and broken them down into columns with their particular type
- not materialized until an action is performed
	- on the point of materialization, entire chain of acyclic graph is going to be executed(for the requested rdd)
	- this gives room for spark to optimize by coalescing operations(ex: map and filter can be coalesced into one stage)
![[Pasted image 20240220183805.png]]

## Why Use RDDs?
- offer control and flexibility
- low-level API
- type-safe
- encourage how-to, we are telling Spark how-to-do

![[Pasted image 20240220184224.png]]

- first piece of code represents reading a text file and parsing it
	- this parsing includes matching 3 mandatory fields: project, page, numRequests
- second part of the code involves operations
	- filter only english pages
	- discard the project field, create a tuple of page and numRequests
	- reduce by key, sum numRequests for the same page(key)
	- action(take) - > kickoffs materialization

## When to use RDDs
- low-level API & control/awareness of dataset
- dealing with unstructured data(media streams or texts)
- manipulate data with lambda functions than DSL
- don't care schema or structure ofdata
- sacrifice optimization, performance & inefficiencies

## Problem with RDDs
- not optimized by spark
- slow for non-JVM langs like Py
- inadvertent inefficiencies:
![[Pasted image 20240220185530.png]]
- since we told the order it has to be performed, the filter is performed after reducebykey
	- if it was the other way around, shuffling due to RBK would have dealt with less burden
- high-level APIs could have avoided this and optimized the filter to be before RBK

f
## What is in an RDD?
- dependencies
- partitions (with opional localityinfo)
	- it knows about where partitions are stored, so that the code can be sent to the right partition
- compute fn, a fn to compute each split
- partitioner
- the compute fn and data is opaque to spark
![[Pasted image 20240222194927.png]]
- df and SQL will be equivalent in terms of speed

![[Pasted image 20240222195450.png]]

## Why Structured APIs?
- declarative
- the operations are supported by SparkSQL engine which built upon Catalyst optimizer and Tungsten code gen

![[Pasted image 20240222195711.png]]
- the rdds generated at the end are far efficient
	- why create rdds? in spark, everything gets decomposed to RDDs
![[Pasted image 20240222195949.png]]
- it is more efficient to do the filtering upon parquet(predicate pushdown)a
- Dataset:
	- each row is a JVM object
	- allows us to use Java notations while defining operations
![[Pasted image 20240222200333.png]]
![[Pasted image 20240222200419.png]]
- why python rdd performance is worst
	- when operations are executed they are pickled and sent to a external py process, it executes there and results are sent back pickled and then unpickling happens
- datasets take less mem when caching
	- because tungsten code gen uses encoders to create ^^^
- encoders are faster in desr/ser performance
- why use?
	- high-level APIs and DSL(domain specific language)
	- strong type-safety
	- ease of use and readability
	- declarative
- when?
	- structured data schema
	- code optimization and performance
	- space efficiency with tungsten
- CBO uses statistics associated with our query to optimize

https://www.youtube.com/watch?v=Ofk7G3GD9jk&t=852s