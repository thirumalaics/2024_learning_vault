- various factors, can be called 5 S
	- spill
		- any oom errors
		- do the executors have enough mem to finish their tasks
	- skew
		- some tasks taking longer that others
	- shuffle
		- large amount of data shuffled across executors
		- join operation
	- storage
		- small files or highly nested dir structure
	- serialization
		- inefficient spark code, or serialization issues 

## Skew
- skew is caused by imbalance partitions
- to fix skew, try to repartition data, or set skew hint when reading in data from disk to mem
- if skew is caused by partition imbalance after shuffling stage, enable skewJoin with AQE
- set the correct shuffle partition size
- skew can lead to spill
- to avoid skew while reading, maxPartitionBytes can be used
- but it is possible that after shuffling, for transformations, some partitions may end up having significantly more data than others
- how to identify skew in UI
	- uneven distribution of tasks in the event timeline of a long-running stage
	- uneven shuffle read/write size in a stage's summary metrics
- soln
	- repartition data
	- enable skew join option with AQE, if skew after shuffling
		- AQE can adapt query plans at run time based on accurate metrics
		- split skewed partitions
		- most effective
		- `spark.sql.adaptive.enabled` and 'spark.sql.adaptive.skewedJoin.enabled'
	- salting
	- set correct shuffle partition size
		- because smaller shuffle partition num can lead to bigger partitions
		- especially for similar keys, the partitions can be much bigger than normal
		- so better to increase shuffle partition

## Spill
- spill is caused by executors lacking of memory to process partitions
- leads to expensive disk reads and writes
- can be caused by maxpartitionbytes too high, explode on an array etc

- how to identify spill in spark ui
	- spill indicators on a stage level
	- or aggregated metrics by executor
- soln
	- add more mem to executor
	- manage the partition sizes by repartitioning 
	- if skew is the reason for the spill, then solve for skew
	- change the shuffle partitions number
	- maxpartitionbytes

## Shuffles
- movement of data across executes due to wide transformation jobs
- shuffle write is the sum of all written serialized data on all executes before transmission
- shuffle read and write happens in different stages
	- write happens first, read happens next
- soln
	- tune spark cluster
		- fewer and larger workers
		- that way different executors on the same machine
	- use predicates and select only the necessary columns
	- turn on AQE to dynamically coalesce shuffle partitions
	- try broadcast join if possible
	- bucketing
		- pre-sorting and shuffling the data into buckets based on the given columns

## Storage
- related to how the data is stored in disk
- small files
- nested directory structure
- schema operations
	- inferring schema for JSON and CSV files
	- may require full read of files to determine the data types
- solns
	- data as tables
	- specify schema during read

https://www.slideshare.net/colorant/spark-shuffle-introduction
https://medium.com/@deepa.account/apache-spark-and-predicate-pushdown-f6a41d53bef5
https://anhcodes.dev/blog/spark-sql-programming/

- spark supports upto 8 GB of broadcast table
	- if more than that, fails
- driver and executor mem should be capable

