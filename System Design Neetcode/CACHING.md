- CPU has it's own cache
- reading from CPU's cache is the fastest for the CPU when compared to reading from memory and Disk
- cpu cache is limited in size: DISK > MEM > CACHE
- caching is a process of storing subset of data(the entire data might be in RAM or disk) in cache for speed
- our browser does caching of things like javascript scripts, html, css
- the http header of the response can contain cache-control which specifies the max duration an element of the response can be cached(or cache policy details) before being requested again
	- cache-control can also be present as part of the user request
- the decision to cache or not depends on the browser config set by user
- client throws the item to be cached into disk, before sending the request for the same data, it checks if the cache location has the same data and it has not expired 
	- remember reading from disk is faster than sending it over the nw
	- even if the data is present but it has expired, it is considered as a cache miss
- cache ration: cache hit / (cache hit + cache miss)
- redis is used for caching and it is a database based on mem
- server side caching concept explained in case of twitter
	- certain subset of tweet may have a large viewership
	- so these tweets can be read from the database everytime and if it passes the threshold to be stored in memory, the tweet can be stored in the in-memory database such as redis
		- write around caching
		- the data is not directly written to the cache the first time as it is written to disk
		- avoids worthless writes to limited in-memory store
	- following requests to the tweet can be served from the cache
- write-through cache
	- the data is immediately written to cache after that written to disk
	- no cache misses initially
- write back caching
	- faster way of doing things, less reliable causing inconsistencies
	- throw to mem and skip disk
	- requests are served from mem
	- periodic backup/write to disk
	- some amount of data loss
- eviction policies for data stored in cache
	- fifo
	- random
	- lru
		- for every access, the tweet is popped from it's old location and it is added to the end of the queue
	- least frequently used
		- key value pairs
		- key is the tweet itself and the value is the number of times the tweet was read
		- similar to the queue, only limited kvps can exist, the key with least value is evicted if the container fills up
		- this factors in read count in the lifetime of key and updates the value only if the tweet is read recently
		- tweets with most views will continue to be in the container even if it was not read recently until another popular tweet overtakes it
		- 
resume 1800

0949