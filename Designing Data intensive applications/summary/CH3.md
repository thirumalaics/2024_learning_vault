- [[27]] - [[Designing Data intensive applications/Mar/Week3/18|18]]
- we saw
	- what happens when we store data in a db
	- what does the db do when we query for the data again
		- [[29#^afe100]]
- storage engines fall into two broad categories
	- OLTP(optimized for TP)
	- OLAP(optimized for AP)
- big differences bw the categories is the access pattern
	- OLTP
		- user-facing
		- huge volume of request
		- apps touch small num of records in each query
		- app requests records using some kind of key
		- storage engine uses an ***index*** to find the data for the key
		- disk seek time is often the bottleneck
	- OLAP
		- used by analysts and not by end users
		- handle much lower volume of queries than OLTPS
		- each query is very demanding
			- millions of records to be scanned in short time
		- disk bw is often the bottle neck
		- COS is a popular soln
- Storage engines for OLTP
	- log-structured [[28]],[[03]],[[04]]
		- append only
		- deletes obsolete files
		- no update to files that have been written
		- relatively new tech
		- systematically turn random-access writes to sequential writes on disk
		- enables high write throughput
			- because of only append
		- disk-friendly
			- because disk is performant for sequential writes and non-overwrites
		- LSMT have higher through put that BT
			- due to lower Write amplification
				- which might depend on the data and config
			- no seeks b4 writes as the writes are append
				- no time spent on seek
	- update-in-place [[05]],[[07]]
		- treats disk as a set of fixed-size pages that can be overwritten
		- BTrees are the biggest example
			- used in many RDB and NRDB
- [[08#Advantages of LSMT| Adv of LSMT]] v [[10#Downsides of LSMT|Disadv of LSMT]]
- some in-mem dbs were seen
- high-level arch of a dwh was seen
- indexes are much less relevant when we need to scan across a large number of rows
	- for such access patterns it is important to minimize the amount of data that the query needs to read from disk
	- COS achieves this