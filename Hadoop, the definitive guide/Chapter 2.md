- Hadoop provides a framework for reliable, scalable storage and analysis
## Map Reduce
- programming model for data processing
- MapReduce programs are inherently parallel
- let's say I do not know what is hadoop and I use awk to do my data processing of a huge dataset, there are a lot of limitations that we can think of right of the bat
	- the implementation will be easier for data that fits within a single machine and messy for data that spans multiple machines
	- i will have to take care of running the job parallely for different parts of the data to utilize multiple cores in my machine
		- this step assumes I do not want to process my dataset in a single thread
		- by doing this parallelization within one machine I achieve the maximum speed for that machine that is allowed awk processing
	- if I am taking care of parallelizing, I will also have to make sure that each thread processes an equal chunk of data
		- the source data files not be of even size, so I will have segregate data into equal chunks
			- if I do not, then some threads will finish early and I will have to make sure some more work is assigned to them
			- instead of doing this additional assignment multiplied by the amount of core, I will benefit from segregating the data into equal chunks beforehand
			- and also I will not be punished by the uneven file sizes, when all files except one is processed, one thread will go on executing while others sit idle
	- taking this parallel approach with equal chunks of data, especially when aggregating or joining I will have to make sure to consolidate the data
		- why? because there is no guarantee that all data corresponding to a key may be present in one chunk
	- I will always be limited by the capacity of the machine that I am working on
	- when start running using multiple machines, I will have to figure out how to coordinate across machines, who runs the overall job, how to deal with failed processes?

- to take advantage of the hadoop fw, we need to express our logic as MapReduce Job

## Map and Reduce
- MapReduce works by breaking the processing into two phases: Map and reduce
- each phase has key value pairs as input and output
	- the types of the key and value can be set by the programmer
- two functions are specified: map function and reduce function
- map function is a good place to filter and prepare our data
	- this data will then be worked upon by the reduce function
- it is not necessary that the key that I received in map needs to be passed on to the reduce
	- the key can be changed
	- as long as there is a key value pair, it should be fine
- the key value pairs sent out from the map function is processed by the map reduce fw before the reduce function can consume it
	- what processing happens bw the map function execution and reduce function execution?
		- the kvps that are sent are sorted and grouped by key
		- let's say that the following data is sent from the map function
		- ![[Pasted image 20241123195651.png]]
		- the mr fw does the processing and sends out the following data to the reduce phase:
		- ![[Pasted image 20241123195737.png]]
	- all the reduce function has to do now is iterate through the list and pick up the maximum reading
	- ![[Pasted image 20241123200048.png]]
- the map function is an abstract method under the Mapper class(generic type)
- when we want to define a map function, we extend the mapper class with a class of our own
	- while extending, we pass in 4 type parameters to the Mapper Class
		- these are input key value pair data types and output key value pair data types
		- MR provides it's own types that are optimized for network serialization
		- looks like these types do not come with handy methods like substring, once inside the map function we can convert them to their java equivalent for easy programming
	- we are expected to override the abstract map method in our class implementation
	- the map method takes 3 arguments, first two are input key, value and the third is context of type Context
	- this context is used to write the output of the map function
![[Pasted image 20250114163752.png]]
- reduce function is an abstract method under Reducer class
	- similar to the map method, 4 type parameters need to passed when extending the class
		- input kvp types and output kvp types
			- the input kvp types should match the output kvp types given during the mapper extension
	- the reduce function takes three arguments
		- key, iterable of values that are collections of values that correspond to the key, context
		- here as well the context is used to write but I am not sure to where
			- the context is used to facilitate comms bw the Reducer and Hadoop fw
			- used to write kvp, configure jobs, and report progress or status during execution
			- looks like this can write to files
![[Pasted image 20250114164945.png]]
- there is one more piece of code that we need to write and it takes care of running the mapReduce job
	- we instantiate a Job class
	- this object is used to set job name, input, output paths, set mapper and reducer classes, set output key and value types
		- the reducer's output key and value types must be specified
		- the mapper's output key and value types will default to whatever is set for reducer's key value pair
			- we do not need to set if our mapper's key value pair types match reducer's key value types
			- if not, specify explicitly with exclusive methods
		- all these settings are done by methods provisioned with the Job object
		- the input types are controlled by the input format and the default is text
		- Job object also comes with wait for completion method which is used to submit the job and wait for it's completion
	- Job object forms the spec of the job and gives control over how the job is run
	- when the job is run on a Hadoop Cluster, we will package the code into a JAR file
		- which hadoop will distribute around the cluster
	- while setting the input path, we can call the respective setter method to set many input paths
	- we also set Jar by Class using setJarByClass method
		- this method takes one argument: class
		- hadoop will use this class to locate the relevant JAR file by looking for the jar file containing this class
		- when I submit a job from a client machine, is the jar sent to the cluster?
			- client machine uploads the jar file to a tmp location in hdfs or the cluster's shared filesystem which ensures the jar is accessible to all the worker nodes
			- `/user/<username>/.staging/<job_id>/job.jar`
		- why do i need to specify the jar using this method, can't mr just use the jar path given during command line call?
			- in command line, we provide path to jar file for local node submitting the job
			- setJarByClass method specifies the JAR file to the Hadoop fw so that it can distribute it to the cluster nodes
				- useful when running code in a cluster, especially from an external env(ex: running jobs from a remote sys), the jar file path may not always be static or predictable
		- why cant hadoop use the cl jar path for distribution?
			- cl path points to local file system and worker nodes on the cluster typically dont have access to the client's lfs
			- even while submitting the jar, client ensures to copy it into tmp location
	- but there can be only one output path
		- and the directory specified here must not exist, this is because hadoop by default does not write if the directory already exists -> error
![[Pasted image 20250115094822.png]]
```
export HADOOP_CLASSPATH=hadoop-examples.jar // since we have used relative path, the following command needs to be run from the same dir as this jar
hadoop ClassName input_path output_path
```
- when the hadoop command is invoked with the class name as first argument, it starts a JVM to run the class
	- hadoop commands add the hadoop libs and dependencies to the class path and picks up hadoop configuration too
- to add application classes to the class path, we exported HADOOP_CLASSPATH
- output of the job is helpful for debugging
	- every job is given an ID and every task is given an id
	- statistics of the job is also shown, including number of mapper and reducer tasks
		- including number of records flowing through the job
	- the output dir contains one output file per reducer
- in real world, data is usually stored in a distributed filesystem(typically HDfs)
	- this allows hadoop to move processing to each machine hosting a part of the data
- MR Job is a unit of work we want to be performed
- a job consists of input data, MR program and configuration
- the job is divided into tasks of two types: map and reduce tasks
- tasks are scheduled on the cluster using YARN
	- if a task fails, it will be automatically rescheduled to run on a different node
- data given as input to a MR job is divided into splits by hadoop
	- these are fixed size data pieces
- one map task for each input split, each of which runs the map function
- by this split mechanism, we avoid falling prey to straggler tasks(provided that the splits are small enough and even, so the slow task is due to data characteristic) and failed tasks(one failed task does not derail the entire job)
- smaller splits also allows hadoop to schedule effectively across different machines
- but we should not take it too far
	- if the size of the split is too small, the overhead of managing the splits and map task creation begins to dominate the total job execution time
- for most jobs, good split size tends to be the size of HDfS block: 128 MB
	- this can be changed for the cluster, or specified when each file is created
- hadoop tries to run the map task where the input data resides in HDfS
	- because to process this data there is no need of cluster bw
	- this is called the data locality optimization
	- sometimes all nodes hosting the HDfS block replicas for a map task may be running other map tasks
		- in this case the job scheduler will look for a free map slot on a node in the same rack
		- even if this is not possible, a node from a different rack is used, this results in inter-rack nw transfer

![[Pasted image 20241128201753.png]]
- why the optimal size of input split is the same as the block size?
	- when both are equal each split corresponds to exactly one block
	- overlapping blocks: where a split includes data spanning across two or more physical storage blocks
		- when split size > block size
			- it might be that blocks needed per split might not be located in the same node
		- also includes when split size < block size
			- ![[Pasted image 20241128203014.png]]
			- this adds complexity as well because MR should understand block 2 is shared bw splits
- map task writes the output to local disk, not HDfS
	- why because this output is only useful until reduce task completes and it is deleted upon completion
		- so it is overkill to store the data in HDfS with replication and so on
- reduce task does not have the advantage of data locality
	- the input to a single reduce task is normally the output from all mappers
	- the sorted map outputs have to be transferred across the nw to the node where the reduce task is running
	- output of a reduce function is normally stored in HDfs for reliability
	- first replica is always stored on the local node, with other replicas being stored on off-rack nodes for reliability
	- writing reduce output consumes bandwidth
![[Pasted image 20241128204127.png]]
- the number of reduce tasks can be specified independently
- when there are multiple reducer tasks, map task partitions their output
	- each creating one partition for each reduce task
	- there can be many different keys in a partition
	- but the records for any given key are all in a single partition
- partitioning can be controlled by providing a partitioning function, but normally the default partitioner works very well
- data flow between map and reduce tasks is colloquially known as the shuffle
![[Pasted image 20241128204911.png]]
- there are scenarios where 0 reducer tasks are used
	- only map, and mapper takes care of writing to HDfS
#### Combiner functions
- this function is specified to optimize the shuffle
	- as many MR jobs are limitted by the bw available on the cluster
	- so it is important to reduce data transferred
- hadoop allows the user to specify a combiner function to be run on the map output
- there is no guarantee on how many times this function will be called on a record, how many ever times it is called, the output should be same
- the combiner function can also be tasked with performing duties of a reducer function on the output of map tasks before transferring them
- ![[Pasted image 20241128205725.png]]
- obviously not all functions possess this property
	- ex: mean
- combiner function is not a replacement for reducer function
	- as reducers will still be needed to process records with the same key from different maps
- combiner function is defined using the Reducer class only but the only difference is we need to set the combiner class on the job using setCombinerClass

In distributed data processing frameworks like Hadoop MapReduce or Spark, values for a key are put into a list before the `reduce` phase through a **shuffle and sort process**. Here's a step-by-step explanation of how this happens:

### 1. **Map Phase**

- The input data is processed by the **mapper function**, which outputs key-value pairs in the form of `(key, value)`.
- For example, in a word count program:
    
    ```
    Input: ["hello world", "hello again"]
    Mapper Output: [("hello", 1), ("world", 1), ("hello", 1), ("again", 1)]
    ```
    

### 2. **Partitioning**

- After the mapper outputs, each key-value pair is assigned to a specific **partition** based on the key. This ensures that all data for the same key will be sent to the same reducer.
- A **partitioner function** (like a hash function) is used to determine which partition a key belongs to.

### 3. **Shuffle Phase**

- The shuffle process **moves key-value pairs across the network** so that all pairs with the same key end up on the same reducer.
- During this phase, the framework groups the values for each key.

### 4. **Sorting**

- The framework sorts the keys in a defined order (usually lexicographically) before passing them to the reducer.
- Within each key, the corresponding values are **grouped into a list**. The result at this stage is something like:
    
    ```
    [("hello", [1, 1]), ("world", [1]), ("again", [1])]
    ```
    

### 5. **Reduce Phase**

- Each reducer receives a key and its associated list of values. The reducer then applies the reduce function to aggregate, transform, or process the values for the key.
- Example (word count reduce):
    
    ```
    Reducer Input: ("hello", [1, 1])
    Reducer Output: ("hello", 2)
    ```
    

### Key Notes:

- This grouping into a list happens **automatically** as part of the framework's internal mechanism.
- In Spark, this is done by the **groupByKey** or similar transformations before actions like reduceByKey.
- The shuffle and sort process is one of the most resource-intensive parts of distributed data processing, as it involves network communication, disk I/O, and sorting.
