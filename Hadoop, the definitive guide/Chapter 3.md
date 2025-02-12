- distributed file system: filesystems that manage the storage across nw of machines are called distributed filesystems
- distributed file systems handle complexities that arise when we store data across machines
	- these complexities are mainly related to the nw
	- one complexity is to make the fs tolerate node failure without suffering data loss
- Hadoop comes with a Dfs called HDfS
- hadoop also has a general purpose filesystem abstraction -> more on this later
- HDfS is designed for storing very large files with streaming data access patterns, running on clusters of commodity hw
	- dissecting the above:
		- streaming data access
			- HDfs built around the idea that most efficient data processing pattern is write once read many times
			- analyses involve the entire dataset, typically, hence the focus here is to optimize throughput of reading the entire data rather than latency in reading the first record
		- commodity hardware
			- does not require expensive, highly reliable hardware
			- commodity hw: commonly available hw that can be obtained from multiple vendors
			- chance of failure across the cluster is very high
			- HDfs is designed to carry on working without noticeable interruption to the user
- applications for which HDfS is not a good choice:
	- low-latency data access
		- HDfS optimized for delivering a high throughput of data, and this may be at the expense of latency
		- why can we not serve data with high throughput and low latency?
			- it is challenging to achieve this because of trade-offs in system design, architecture, physical limitations
			- HDfS replicates data, serving data from multiple replicas adds coordination overhead and delays
			- HDfS has large block size, which is useful to minimize metadata and maximize throughput for sequential reads/writes but even to read small piece of data entire block need to be transferred
				- bad for random reads
			- high throughput relies on streaming large chunks of data over the nw bw distributed nodes
				- nw comms introduces delays especially when a request involves retrieving data from multiple nodes
			- designing a system that delivers both high throughput and low latency require:
				- increased hw costs
					- SSDs, more mem, low latency nws
				- complex architecture
					- sophisticated indexing and caching mech
	- lots of small files
		- namenode holds md in mem
		- more files, then more md, then more load on namenode
		- each file, dir and block takes about 150 bytes in mem
	- multiple writers, arbitrary file modifications
		- files in HDfs may be written to by a single wrtier
		- writes are always made at the end of the file, in append-only fashion
		- no support for multiple writers or for modifications at arbitrary offsets in the file

## HDfS concepts
- a ***disk*** has a block size, which is the minimum amount of data that can read or written
	- in the order of bytes
- file system builds on the disk block size, by having it's own block size: file system block size
	- this block size is a integral multiple of disk block size
	- in the order of kbs
- for fs users, this is not very useful, but there are tools such as df and fsck which work on the filesystem block level
	- these are used to perform file system maintenance
- HDfS has it's concept of block
	- 128 MB by default
	- files in HDfS are broken into block-sized chunks, which are stored as independent units
	- a file in HDfS that is smaller than the block size does not occupy full block's worth of underlying storage
		- the opposite is true for a file system for a single disk, files with size less than the file system block size occupy the full space
		- the unused space cannot be used for any other storage
- Why use filesystem blocks?
	- simplify how the data is stored and managed
	- blocks are used to allocate space for files and directories
		- fs has to manage fs blocks instead of each and every byte
		- fs md keeps track of blocks rather than individual bytes
	- minimizes the number of seek operations or access operations(SSD) needed to read or write a file
		- rather than seeking each byte of a file
	- blocks allow a file system to support large files without requiring continuous storage on the disk
	- blocks allow fs to maintain a bitmap or a list of free blocks to track unused space
	- if part of a file becomes corrupted, only the affected blocks are lost, not the entire file. makes recovery easier
- why is the block size in HDfS larger than normal file system?
	- for big data, especially when we have data split in to large files, if the block size were to be in KBs:
		- per file there will have to be many seek operations
		- time spent seeking is time not spent transferring(by making use of the disk transfer rate)
		- by making the block size large, we avoid spending more time in seek and spend more time transferring the data
- benefits of block abstraction for a dfs:
	- a file can be larger than any single disk in the nw
		- a file can be split and stored across the nw
	- making the unit of abstraction a block rather than a file simplifies the storage subsystem
		- does not have to deal variable sizes
		- minimizes fragmentation(I believe 128Mb block size saves from external fragmentation):
			- external fragmentation: when a piece of data is not stored in a contiguous space in disk
				- blocks are scattered across disk
				- inefficient as the file system has to piece together data which is an overhead
			- internal fragmentation:
				- piece of data does not fully utilize the block size provided
	- fits well with replication for providing fault tolerance and availability
		- blocks that make up the file are replicated by replication factor across physically separate machines
		- if a block becomes unavailable, a copy can be read from another location
		- a block that is unavailable due to corruption or machine failure can be replicated from its alternate locations to bring back the replication factor to normal
		- `hdfs fsck / -files -blocks`
			- lists the blocks that make up each file in the filesystem
- md of a file is not stored with each an every block
	- md is centralized or in other words managed by a diff entity called name node server
		- data distributed, md centralized
		- separating md helps in modular approach
			- where as in normal file system it is integrated as the data is not huge and designed for a single machine
	- even in normal file system md is stored separately using data structures like inodes
		- md tightly integrated in to file system itself unlike hadoop
		- md stored on same disk but separately
		- a file system driver is responsible for managing both data and metadata

## Namenodes and Datanodes
	- HDfS cluster has two types of nodes operating in a master-worker pattern
- name node - master, data nodes - workers
- the name node manages the filesystem namespace
	- maintains the filesystem tree and md for all the files and directories
		- this information stored persistently on the disk in the form of 2 files: the namespace image and the edit log
	- name node also knows the location in data nodes of all blocks that constitute to all files in HDfS
		- not stored persistently
		- this information is reconstructed from datanodes every time the system starts
- a client is created whenever we want to interact with the HDfS
	- in most cases it is immediately destroyed after use, especially when we are issuing commands using CLI
		- we can also explicitly create and destroy, in which case we have the control of its lifecycle
	- this client is instantiated on the machine where the code runs
		- a dev machine if we are trying to access a remote HDfS cluster
		- a node in the hadoop cluster, ex: edge node
		- usually the client talks to the namenode first as it needs metadata figured out
		- the client presents a filesystem interface similar to a POSIX(Portable System Interface), so the user code does not need to know about the namenode and datanode
- What is POSIX? 
	- set of standardized operating system interfaces that determine how sw interacts with an operating system's core services
		- ex: file operations, process management, inter process communication
	- these standards ensure compatibility and portability of applications across OSs
- data nodes are the workhorses of the filesystem
	- they store and retrieve blocks when they are asked to(by clients or namenodes)
	- they report back to name node periodically with lists of blocks that they are storing
- without the namenode, the filesystem cannot be used
	- because only name node knows what  blocks to piece together across data nodes that constitute a file
- what is a namespace image?
	- it is a file that contains persistent checkpoint of the entire HDfS namespace
	- stores the state of a HDfS at a specific point in time
	- stored in the namenode usually, but in high-availability settings, it is stored in another machine(secondary or backup namenode)
- what is edit log?
	- records incremental changes made to the namespace since the last checkpoint
- the namespace image and edit log are periodically merged
- to ensure resilience of name nodes, hadoop provides two mech
	- back up the files that make up the persistent state of the filesystem metadata
		- hadoop can be configured to write its persistent state to multiple filesystems
			- synchronous and atomic
			- usual config is to write to local disk and a NfS mount
	- it is also possible to run a secondary name node, which despite the name, does not act as a name node
		- main role is to periodically merge the namespace image and edit log to prevent the edit log from becoming too large
		- runs on a separate machine as it requires plenty of CPU and as much memory as the namenode to perform the merge
## Block Caching
- normally a data node reads blocks from disk
	- but for frequently accessed files ***the blocks*** may be explicitly cached in the data node's off-heap block cache mem
	- by default, a block is cached in only one data node's memory, configurable on a per-file basis
	- users or applications instruct the name node on which files to cache(and for how long) by adding a cache directive to a cache pool
		- cache pools are an administrative grouping for managing cache permissions and resource usage
		- what is a cache directive?
			- command
			- issued to the namenode to specify which files or dirs should be cached in mem for faster access
			- also specifies for how long, which cache pool is this directive part of, replication
		- what is cache pool?
			- logical grouping of cached files that helps manage caching policies across multiple users or apps
			- it enforces policies like access control and resource allocation
			- cache pools help by :
				- setting limits on how much memory can be used for caching
				- providing access control (who can cache and uncache)
				- organizing cached files under different projects or teams
			- these are created by the admin
			- allows teams to submit cache directives to their respective cache pool
		- how does cache pool and cache directive work?
			- instead of caching files randomly, users define cache pools and then add cache directives(requests to cache specific files) under them
			- a cache pool has properties like:
				- owner: who can manage the cache pool
				- group & permissions: who can use the pool
				- max cache size: prevents excessive memory usage
				- time to live: how long files remain cached
Imagine a company has two teams working on different projects:

1. **Team A (Machine Learning)** needs fast access to training datasets.
2. **Team B (Analytics)** needs quick access to logs for real-time queries.

The admin can create two separate **cache pools**:

- **ML_Cache_Pool** (for ML team’s datasets)
## HDfS federation
- for large clusters with many files, scaling might become difficult with a single namenode
- HDfS federation was introduced in Hadoop 2.x
	- allows a cluster to scale by adding namenodes
	- each of which manages a portion of the filesystem namespace
		- ex: one namenode manages files rooted under /user
- under federation, each nodenode manages
	- a namespace volume, which is made up of the metadata for the namespace
		- independent of each other, name nodes dont comms with each other
		- failure of one does not affect others
	- block pool containing all the blocks for the files in the namespace
		- block pool is a logical grouping of data blocks, managed independently by the cluster
		- blocks from one pool are managed independently of blocks from other pools, even though they may be stored on the same set of Data Nodes
		- block pool storage not partitioned
		- so data-nodes register with each namenode in the cluster and store blocks from multiple block pools

## HDfS high availability
- the combination of replicating namenode md on multiple fs and using secondary namenode to create checkpoints protects against data loss
	- but does not provide high availability of the filesystem
- namenode is still the single point of failure
	- whole hadoop system would be out of service until a new name-node could be brought online
	- a new primary namenode is started with one of fs metadata replicas
		- configure datanodes and clients to use this as the new namenode
	- new namenode cannot serve requests until
		- it has loaded its namespace image into memory
		- replayed its edit log
		- received enough block reports from the data nodes to leave safe mode
- until Hadoop 2, this restarting of name node or instantiating a new name node took so much time
	- this affected maintenance more
- Hadoop 2 introduced HDfS high availability implementation
	- this allows active-standby configuration: pair of namenodes
	- in the event of the failure of active, standby takes its place without significant interruption within few tens of seconds
- this new implementation required architecture changes
	- the namenodes must use highly available shared storage to share the edit log
		- when a standby comes up, it uses the edit log to synchronize state with the active namenode
	- data nodes must send block reports to both namenodes, as the file-block mapping is stored in memory
	- clients must be configured to handle namenode failure, using a mech that is transparent to the users
		- clients mean, applications or tools that interacts with the HDfS
	- the secondary namenode's role is subsumed by the standby, which takes periodic checkpoints of the active namenode's namespace
- there are two options for the highly available shared storage: an NfS filer, or a quorum journal manager(QJM)(recommended)
	- QJM is a dedicated HDfS implementation
		- designed for providing highly available edit log
	- QJM runs as a group of journal nodes, and each edit must be written to a majority of journal nodes
	- typically there are 3 journal nodes, so the system can tolerate the loss of one of them
- if the active namenode fails, the standby can take it's place within 10s of seconds
	- as the latest state is in memory: both the latest edit log entries and an up-to-date block mapping
	- but the actual failover time observed will be longer in practice(around a minute or so)
		- because the system needs to be conservative in deciding that the active namenode has failed
- if standby is also down, from an operational point of view it is an improvement
	- because the process is a standard operational procedure built into hadoop
	- in both non-HA and HA with failed standby there is manual work involved in restarting the node. 
		- but in case of HA, the procedure is part of a well-defined, pre-configured fw
			- the process is structured and built into the hadoop's HA architecture
			- md recovery is not needed as active and standby would have shared the same md
			- no complex recovery procedure
		- in case of non-HA, recovery involves restarting NN, potentially running many commands
			- downtime is generally higher
			- manual checks required, validations or recovery steps
			- we will have to take care rebuilding the state

#### failover and fencing
- transition from an active to standby namenode is managed by a new entity in the system called failover controller
	- there are various failover controllers
	- but the default implementation uses Zookeeper
	- uses simple heartbeating mechanism
	- this controller is a process whose job is to monitor its namenode for failure and trigger a failover
	- looks like the controller runs on the same machine
	- one controller for one namenode
- failover may also be initiated manually by an administrator
	- called graceful failover
- in case of ungraceful failover, it is impossible to ensure that the failed namenode has stopped running
	- ex: slow nw, or a nw partition can trigger a failover transition
		- the namenode will continue to think it is the active one
- HA implementation goes to great lengths to prevent previously inactive namenode from doing any damage
	- method called fencing
- there are fencing mechanisms
	- depending upon the shared edit log, the complexity increases
		- QJM only allows one namenode to write to the edit log at one time
			- it is still possible for the previously active namenode to serve stale read requests to clients
				- stale read-requests: provide file system metadata or data that might be outdated
			- so setting up an SSH fencing command that will kill namenode's process is a good idea
				- mechanism to isolate a problematic node from the rest of the cluster
				- involves using SSH to access the node and perform actions like rebooting, shutting down or disabling services
		- stronger fencing mechs are required while using Nfs filre for the shared edit log
			- since nfs filer accepts more than one writer to write at a time
			- fencing mech:
				- revoke access of namenode to the shared storage dir
				- diable it's nw port via a remote management command
				- STONITH: Shoot the other node in the head
					- specialized power distribution unit to forcibly shutdown the host
- client failover: client operations(like read/write) continue transparently even if the primary server fails
	- client requests are redirected to standby
	- can be handled in client side:
		- HDfS clients(hdfs dfs) use the HDfS client lib to interact with HDfS
		- handled by configuring host name to two namenode addresses in a hdfs-site.xml
		- client library tries each namenode address until operation succeeds

## The CLI
- two important configs are discussed below and these configs can be provided in many places:
	- core-site.xml
	- CLI argument
		- `hadoop fs -Dfs.defaultFS=hdfs://namenode:9000 -ls /`
	- java prog
	- env var
	- hadoop-config.properties
- fs.defaultFS: sets the default filesystem for Hadoop
	- by default HDfS
	- specified as a URI. ex: hdfs://localhost/
	- this is used to identify the host and port for the HDfS namemode
		- used by clients and HDfS daemons
	- in the above example, host is localhost and port is the default 8020
	- if this property is not set, hadoop falls back to using file://
- dfs.replication
	- replication factor that decides how many datanodes does this block needs to be replicated in
- hadoop fs -help
- `hadoop fs -copyFromLocal input/docs/quangle.txt hdfs://localhost/user/thirs/quangle.txt`
	- we could have omitted the scheme and host of the URI and picked up the default, hdfs://localhost as specified in core-site.xml
	- what does scheme mean?
		- scheme indicates how the resource should be accessed and identifies the protocol to be used such as http, https, ftp etc...
		- scheme here is hdfs
	- `hadoop fs -copyFromLocal input/docs/quangle.txt /user/thirs/quangle.txt`
	- we also could have used a relative path and copied the file to our home dir
		- `hadoop fs -copyFromLocal input/docs/quangle.txt /user/thirs/quangle.txt`
- `hadoop fs -copyToLocal quangle.txt quangle.copy.txt`
	- to check they are same: md5 input/docs/quangle.txt quangle.copy.txt
- hadoop fs -mkdir books
- hadoop fs -ls
	- similar to ls -l
	- first column: file permissions
	- second is replication factor of the file
		- 0 for directories and replication factor does not apply to dirs
		- directories are treated as metadata and stored by the name node, not the datanodes
	- 3rd and 4th: owner and the group
	- 5th: size of the file in bytes, 0 for directories
	- 6th and 7th are modified date and time
	- 8 the is the name
![[Pasted image 20241218122023.png]]
- HDfS has a permissions model for files and directories that is much like the posix model
- 3 types of permissions: r, w, x
	- r: to read files or list contents of a dir
	- w: to write a file or create or delete files and directories in a directory
	- x: ignored, as we cannot execute files on HDfS, and for a directory this permission is required to access its children
- by default hadoop runs with security disabled
	- client's identity is not authenticated
	- it is easy to impersonate, let's say there is a user x on hadoop. A client from a remote machine can have the same user name and login to hadoop and impersonate
	- but it is better to have permissions enabled to avoid accidental modification or deletion
- when permission checks are enabled, owner permissions are checked if the username matches the owner, group permissions are checked if the client is a member of the group, otherwise others permissions are checked 
- each file and directory has an owner, a group and a mode
- mode is made up of permissions for the users who is the owner, users who are members of the group, and the permissions for users who are neither the owner nor member of the group
- there is a concept of superuser, which is the identity of the namenode process
	- Permissions checks are not performed for the superuser

## Hadoop filesystems
- what is a filesystem?
	- a filesystem is a system that manages how data is stored, organized, retrieved and managed on a storage device
	- it acts as an abstraction layer between the operating system and physical storage
	- filesystems structure data in a hierarchy of files and directories to make it easier to locate and manage
	- provides mechanism to locate and retrieve data stored on the storage medium
	- ensure that the data is not corrupted and provide mechanisms to recover it in case of failures
	- enforce permissions and security rules
	- store metadata
- why do we need a filesystem?
	- without a fs, data would be stored as a raw sequence of bytes with no structure or labels, making it impossible to differentiate between files and data types
	- allow efficient allocation of storage space and retrieval of files
		- help avoid fragmentation and optimize disk usage
	- provide and abstraction for users to work with
	- data sharing via nwed filesystems
	- security
	- advanced filesystems target fault tolerance and backup
- hadoop provides many interfaces to its filesystems
	- hdfs is just one implementation
- hadoop ships with many such implementations
- hadoop can interact with different types of filesystems, not just HDfS
- hadoop is written in java, so most Hadoop filesystem interactions are mediated through the JAVA APi
	- by exposing its filesystem interface as a JAVA API, hadoop makes it harder for non-java apps to access HDfS
	- the HTTP rest API exposed by the WebHDfS protocol makes it easier for other languages to interact with HDfS
	- HTTP interface is slower than native java client, so should be avoided for very large data
	- there are two ways to access HDfS over HTTP: 
		- directly, where the HDfS daemons serve HTTP requests to clients
		- via a proxy, which accesses HDfS on the client's behalf using the usual Distributedfilsystem API
- Hadoop generally uses the URI scheme to pick the correct filesystem instance to communicate with

![[Pasted image 20241220131140.png]]
- in the first case, the embedded web servers in the namenode and datanodes act as WebHDfS endpoints
	- configurable, we can switch on or off this behavior
	- file md operations handled by namenode, while read(and write) operations are sent first to the namenode
		- which sends an HTTP redirrect to the client indicating the data node to stream file data from (or to)
		- after receiving the redirect, the client directly interacts with the specified datanodes to read or write
		- this design prevents namenode from becoming a bottleneck
- second way involves interacting with one or more proxy serverrs
	- these proxies are stateless, so they can behind a standard load balances
	- all traffic to the cluster passes through the proxy
	- client never accesses namenode or datanode
		- allows for stricter firewall and bandwidth limiting policies
			- how?
				- by centralizing access, sys admins can enforce security and traffic rules at a single point(proxy servers), rather than having to manage across datanodes and namenodes
				- attack surface is limited by banning connections to datanodes and namenodes
					- clusters can have strict firewall rules allowing only connections to/from proxies
				- rate limiting can be applied to the proxy layer
	- common to have proxies for 
		- transfer between hadoop clusters located in different data centers,
		- when accesing a Hadoop cluster running in the cloud from an external nw
- HTTPfs proxy exposes same HTTP (and HTTPS) inteface as webHDfS
	- HTTPfS and WebHDfS both enable HTTP/HTTPs based access to HDfS, but they serve diff purposes and have distinct diff
	- WebHDfS: Native built in rest api provided by HDfS to directly access HDfS over HTTP
		- server runs on each namenode and datanode
	- HTTPfS:
		- proxy server for HDfS that provides an HTTP i/f to access HDfS
		- but routes all requests through a central proxy
		- main diff is this is a server run by a specific script
- Hadoop also provides C lib that mirrors the one in Java
	- it can be used to access any Hadoop filesystem
	- works using the Java Native Interface to call a java filesystem client
		- similar to py4j but for C
- Hadoop can run with multiple filesystem implementations at the same time
	- but hadoop does not support storing a single file across multiple filesystems
	- hadoop can store different files in different filesystems simultaneously
- HDfS can be mounted on a local client's filesystem using Hadoop NfSv3
	- we can then use unix utilities to interact with the filesystem
	- appending to a file works, but random modifications of a file do not
	- another way to mount HDfS as a standard local filesystem is fuse(filesystem in userspace)
Java interface for hadoop is ignored in the notes
- buffer size:
	- while reading data, chunks of data are read into memory(buffer) in blocks of size bufferSize
	- the application reads from this buffer, which minimizes direct interactions with the filesystem, reducing latency and improving performance

## Data flow
- ![[Pasted image 20241222145429.png]]
- DistributedFilesystem is a class which provides interfaces to interact with HDfS
1. Client opens a file by calling open() on the DistributedFilesystem object
2. DistributedFilesystem calls the namenode, using RPCs, to determine the locations of the first few blocks in the file
	- for each block, the namenode returns the addresses of the datanodes that have a copy of that block
	- datanodes are sorted according to their proximity to the client
		- incase of a mapreduce task, the client reside within a datanode, so the client will read from the local datanode if the datanode hosts a copy of the block
3. The DistributedfileSystem returns an fSDataInputStream (an input stream that supports file seeks) to the client for it to read from
	- fSDataInputStream in turn wraps a DfSInputStream, which manages the datanode and namenode IO
		- what this means is fSDataInputStream internall uses or delegates to DfSInputStream to handle actual operations required to read data from HDfS
		- DfSInputStream responsible to deal with HDfS components like NN and DN
4. the client calls read on the stream, DfSInputStream
	- DfSInputStream has stored the datanode addresses for the first few blocks in the file
	- then connects to the first(closest) datanode for the first block in the file
	- data is streamed from the datanode back to the client
	- which calls read repeatedly on the stream
5. Once end of block is reached, DfSInputStream will close the connection to the datanode, then find the best datanode for the next block
6. It is important to remember the interface may not have details of all blocks at first itself
	- this might need follow up calls to namenode as needed
	- blocks are also read in order
	- client calls close on the interface once it has finished reading
- additional steps:
	- if the interface is not able to retrieve block from a particular datanode, it tries with the next datanode which has the block
		- the interface remembers the bad datanode, so as to avoid retrying in the future for some other block
	- the interface also verifies checksums of the blocks transferred to it from the datanode
		-  if a block is found to be corrupted, the interface redirects to another dn
		- reports this to NN as well
	- namenode is a guide to data rather than a servant of data
		- hence it can spread out multiple client requests to the same data across the cluster
		- nn does not become a bottleneck as it only takes care of guiding clients with block location requests
			- stores in memory, hence efficient
- what does it mean for two nodes in a local nw to be close?
	- bandwidth bw two nodes as a measure of distance
	- measuring bandwidth bw nodes is difficult in practice
	- hadoop takes a simple approach in which the nw is represented as a tree and the distance bw two nodes is the sum of their distances to their closest common ancestor
	- levels in the tree are not predefined, but it is common to have leves that correspond to data center, the rack and the node that a process is going on
	- bw available for each of the following scenarios becomes progressively less:
		- process on the same node
		- different nodes on the same rack
		- nodes on different racks in the same data center
		- nodes in different data centers
![[Pasted image 20241224202018.png]]

![[Pasted image 20241224202333.png]]
- hadoop assumes by default, that the nw is flat - a single level hierarchy
	- all nodes are on single rack in a single data center
- what is a block ID?
	- unique identifier assigned by the Namenode to blocks
	- all replications of a block share the same block id
	- namenode maintains a block id to list of datanode mapping
	- blockid changes if the block is modified or recreated
		- pipeline reconfiguration after a datanode failure
		- file truncation or overwrite operation
		- rebalancing or other operations that command block recreation
### Anatomy of a file write

![[Pasted image 20250101100217.png]]
1. Client creates a file by calling create() on DistributedfileSystem
2. DistributedfileSystem makes an RPC call to the namenode to create a new file in the filesystem's namespace with no blocks associated with it
3. namenode performs various checks to make sure that the file does not already exist and that the client has necessary permissions
4. if the checks pass, namenode makes a record of the new file, otherwise, file creation fails and the client is thrown an IOException
5. Distributedfilesystem returns an fSDataOutputStream for the client to start writing data to
	- fSDataOutputStream wraps a DfSOutputStream
	- handles communication with namenode and datanodes
6. As the client writes the data, DfsOutputStream splits it into packets and writes it into an internal queue called data queue
	- Data queue is consumed by the DataStreamer, which is responsible for asking the namenode to allocate new blocks 
	- namenode picks a list of suitable datanodes to store the replicas
7. the list of datanodes returned forms a pipeline
	- if the replication factor is 3, then the list of nodes contain 3 nodes
	- DataStreamer streams the data to the first node in the pipeline
	- first node stores each packet and forwards it to the second datanode in the pipeline
	- second datanode stores each packet and forwards it to the last datanode
	- DfsOutputStream also maintains an internal queue of packets that are waiting to be acknowledged by datanodes, called the ack queues
		- this is not the same as ack queue
	- the packet is removed from the ack queue only when it has been acked by all the datanodes in the pipeline
	- packet and block are not the same
	- multiple packets make up a single block
8. client calls close on the stream after it finishes writing data
	- this action flushes all the remaining packets to the datanode pipeline and waits for acknowledgements
9. once all acks are received, client signals to namenode that the write is complete
	- since namnode already knows which blocks constitute a file(because name node allocates blocsks before datastreamer can start writing)
	- so client only needs to wait for blocks to be minimally replicated b4 returning successfully
- if any datanode fails while data is being written to it, the following actions are taken:
	- the pipeline is closed
	- any packets in the ack queue are added to the front of the data queue so that datanodes that are downstream from the failed node will not miss any packets
	- if the current block had been written to one or more datanodes before failure, the successful blocks are given new block ID
		- this is communicated to namenode
		- so that partial block on the failed datanode will be deleted if the failed data node recovers as it will be an orphan block
	- the failed datanode is removed from the pipeline, and a new pipeline is constructed from the two good datanodes
	- namenode notices that the block is under-replicated, and it arranges for a further replica to be created on another node
	- in the unlikely failure of multiple datanodes in the pipeline, the write will succeed as long as dfs.namenode.replication.min replicas are written
		- defaults to one
		- the blocks will be asynchronously replicated across the cluster until its target replication factor is reached
- replica placement
	- there is a tradeoff between reliability and write and read bw
		- ex:placing all replicas in one node incurs the least write bandwidth penalty
		- but this offers no real redundance and it is highly unrealiable in case of node failure
		- read bandwidth is high for off-rack reads
			- off-rack reads: data is located in a different rack than the processing node
		- placing replicas in diff datacenters maximizes redundancy but at the cost of bw
	- even within the same datacenter there are different placement strats
	- default strat:
		- first replica is on the same node where the client runs
			- for cases where the client runs outside the cluster, a node is chosen at random
			- NN tries not to pick nodes that are too full or too busy
		- second replica placed on a different rack from the first(off-rack), chosen at random
		- third replica stored in the same rack as second but on a diff node chosen at random
		- further replicas are placed on random nodes in the cluster, while trying to avoid placing too many copies on the same rack
	- once the locations are chosen, a pipeline is built, taking nw topology into account
![[Pasted image 20250101100134.png]]

## Coherency model
- coherency model for a filesystem describes the data visibility of reads and writes for a file
- HDfS trades off some POSIX requirements for performance, so some operations may behave differently
	- ex: after creating a file, the file is visible in the namespace but any content written to the file is not ***guaranteed*** to be visible
		- even if the stream is flushed
		- so file appears to have a length of zero
- always, current block that is being written is not visible to other readers
- once more than a block's worth of data has been written, the first block will be visible to new readers
- HDfS provides a way to force all buffers to be flushed to the datanodes via hflush() method on fSDataOutputStream
	- after successful return from hflush, HDfS guarantees that the data written up to the point in the files has reached all the datanodes in the write pipeline and is visible to all new readers
- hflush does not guarantee that the datnodes have written the data to disk
	- only that it's in the datanode's memory
	- data could be lost in the event of a power outage
	- closing a file in HDfS performs an implicit hflush too
- hsync can be used instead for stronger guarantee
	- similar to fsync system call in POSIX that commits buffered data for a file descriptor
- hflush and hsync differ in the guarantees that they provide and use cases
	- hflush
		- flushes data written to the output stream so that it is available to readers
		- no guarantee on disk write of the data in DN
		- for quick reader visibility
			- ex: logs that can tolerate data loss if failure occurs
	- hsync
		- ensures that the data is flushed and synchronized to the disks of all datanodes in the pipeline
		- similar guarantee like hflush, but one extra guarantee of persisting to disk on all datanodes
		- for apps that require durability and fault tolerance
![[Pasted image 20250102185004.png]]

- what does this mean for our applications?
	- with no calls to either of these methods, we could loose up to a block of data in the event of client or system failure
	- so it is better to call the method  at suitable points
	- hsync has higher overhead than hflush
	- there is a tradeoff between data robustness and throughput

### Parallel Copying with distcp
- so far, HDfS access patterns seen involve only single-threaded access
- it is possible to act on a collection of files, but for efficient parallel processing of these files, we will have to write our own program
- distcp is one such command that allows us to parallely copy data to and from Hadoop filesystems in parallel
`hadoop distcp file1 file2`
- if dir2 does not exist, it will be created, the contents of the dir1 are copied there
`hadoop distcp dir1 dir2`
- multiple source paths can be specified
- if dir2 already exists, then dir1 will be copied under dir2
	- we can supply -overwrite option to overwrite contents of dir2 with dir1's contents
- -update option available to update only the files that have changed
- implemented using a mapreduce job
- work of copying is done by the maps that run in parallel, with no reducers involved
- each file copied by a single map, with each map given approximately the same amount of data by bucketing files into roughly equal allocations
- by default 20 maps are used, can be changed by providing -m argument
- -delete flag causes distcp to delete any files or dirs from the dest that are not in the source
- -p option to preserve attributes like permissions, block size and replication
- if two clusters are running incompatible versions of HDfS, then we can use webhdfs protocol to distcp between them:
	- `hadoop distcp webhdfs://namenode1:50070/foo webhdfs://namenode2:50070/foo`
- another variant is to use HTTPfS proxy as the distcp source or destination
	- again using the webhdfs protocol
	- has advantage of bw and firewall control
- when copying data to HDfS, it is important to consider cluster balance
	- file blocks must be evenly spread across the cluster
- when using distcp we must ensure it does not disrupt this
- when using single map, it does not only slow the entire process due to single thread
	- first replica of each block would reside on the node running the map(until disk filled up)
	- the second and third replicas would be spread across the cluster
- by having more maps than nodes in the cluster, this problem is avoided
	- for that reason, it's best to start with 20 maps per node, which is the default
- but it is not always possible to prevent a cluster from becoming unbalanced
	- we might have to decrease the number of maps so that some of the nodes can be used by other jobs
	- in this case balancer tool can help us to distribute blocks
1537