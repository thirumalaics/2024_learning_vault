## YARN
- yet another resource negotiator
- cluster resource management system
- YARN was introduced in hadoop 2 to improve mapreduce implementation
	- but it is general enough to support other distributed computing paradigms as well
- YARN provides APIs for requesting and working with cluster resources
	- as users, we do not interact with these APIs
	- we work with higher-level APIs provided by distributed computing fws
	- these fws are built on YARN and hide the resource management details from the user

![[Pasted image 20250103154853.png]]
- there can be another layer that build on the fws shown in the above diagram
	- these fws run on MR, Spark or Tez or all three and they do not interact with YARN directly
	- ex: pig, Hive, Crunch

## Anatomy of a YARN application run
- YARN provides it's core services via two types of long-running daemons: 
	- a resource manager
	- node managers
- resource manager
	- one daemon per cluster
	- to manage the use of resources across the cluster
- node manager
	- one per node
	- to launch and manage containers
- container:
	- container is a collection of physical resources(CPU, memor, disk and nw) allocated to a specific task or application by YARN resource manager
	- they are ***resource allocation unit***
	- provide logical abstraction of resources
	- tasks running in containers are processes or threads managed by the host OS and not isolated virtualized envs
	- resource isolation is achieved via OS features such as Control groups
	- executes and application specific process with a constrained set of resources(mem, CPU and so on)
	- depending upon how YARN is configured, a container ***may be a unix process or a Linux Cgroup***
- what is a linux Cgroup?
	- kernel feature that provides a mechanism for managing and controlling the resource usage of a group of processes
	- using cgroup we can restrict amount of resources(CPU, memory, I/O etc.) that a group of processes can consume
	- allocate resources more favorable to certain groups over others
	- ensure one group of resources does not interfere with others by consuming excessive resources
	- Cgroups operate by organizing processes into hierarchical groups
	- each group is associated with a set of resource controllers(called subsystems) that enforce limits and monitor usage
	- subsystems in cgroups represent different types of resources that can be controlled
- the following steps are followed to run an application on YARN
	1. client contacts resource manager requesting it to run an application manager process
	2. the resource manager finds a node manager that can launch the application master in a container
		- what the application master does once it is spawned depends on the application
		- it could run a computation and return the result to a client
		- it could request more containers from the resource managers and use them to run a distributed computation
			- this is what a MAPREDUCE YARN app does
- YARN itself does not provide any communication mechanism for the parts of an app(client, master, process)
	- most common YARN apps use some form of remote communication(such as Hadoop's RPC layer) to pass status updates and results back to the client
		- but even this is specific to the application
- YARN allows clients to request for resource in with less friction
	- a request can specify amount of compute and mem per container and provide ***locality constraints*** for the containers in that request
- what is locality constraint?
	- preferences or requirements for where containers should be allocated within the cluster
	- these constraints help ensure that the containers are placed in locations that optimize performance, resource usage, or data access
	- common type of locality constraints:
		- node-local
			- containers are request on specific nodes where the required data already resides
		- rack-local
			- if node-local placement is not possible, containers can be placed on any node within the same rack
		- Any node
			- when both of the above are not possible
- sometimes the locality constraint cannot be met, in which case either no allocation is made or, optionally the constraint is loosened
- a YARN app can make resource requests at any time while it is running
	- either upfront or dynamically after the app starts
	- std spark takes the first approach
	- std mr is dynamic in case of reduce containers
		- map containers are requested up from
		- in case of any task failures, containers are requested dynamically
#### Application lifespan
- lifespan of YARN application vary dramatically
- it is useful to categorize apps in terms of how they map to the jobs that users run, following are the different categories
	- simplest case is one application per user job, which is the approach that MR takes
	- run one app per workflow or user session(possibly unrelated) jobs
		- efficient as containers can be reused between jobs
		- additional potential of caching data bw jobs
		- spark is an example that uses this model
	- long-running app that is shared by diff users
		- imapala, apache slider
		- users experience low latency since overhead of start a new app master is avoided
		- such an app often acts in some kind of coordination role

#### Building YARN apps
- writing a YARN app from scratch is not necessary in many cases as it is possible to use an existing app that fits the bill
- there are couple of projects that simplify the process of building a YARN app
	- apache slider allows us to run existing distributed apps on YARN
		- ex: HBase, Accumulo and other big data systems
	- with slider, users can run their own instances of an application on a cluster, independent of other users
		- ex: one might run HBase 1.2 and other might run HBase V 2.0
	- slider provides tools to change the number of nodes allocated to an application dynamically
	- allows users to suspend and later resume a running app
		- useful in cases where resources need to be allocated temporarily for other priorities
	- YARN at times does not simplify running complex distributed apps
	- apache slider acts as a middle layer bw YARN and distributed applications
		- enables flexible and independent application management, resource scaling and efficient utilization of hadoop cluster
- Apache twill is similar to slider
	- simple programming model for developing distributed applications on YARN
	- provides the following capabilities, among others:
		- real-time logging(logs streamed from app to client)
		- command messages (sent from client to runnables)
- in cases where none of the options are sufficient, then the distributed shell application that is part of YARN serves as an example of how to write a YARN application
	- it demonstrates how to use  YARN client APIs to handle comms between client or AM and the YARN daemons

## YARN compared to MR1
- Hadoop 1 did not have YARN
	- distributed implementation of MR in the original version of Hadoop is sometimes referred to as MR1
- MR2 uses YARN from Hadoop 2
- old and new MR APIs are not the same thing as MR1 and MR2 implementations
	- the APIs are user-facing client-side features and determine how we write MR programs
	- whereas implementations are just different ways of running MR programs
	- all four combinations are supported: both old and new MR APIs run on both MR1 and MR2
- in MR1, there are two types of daemons
	- jobtracker and one or more tasktrackers
		- jobtracker schedules tasks to run on tasktrackers
			- splits the mr job submitted by client into smaller map and reduce tasks
			- keeps track of overall progress of each job
			- if a task fails, the jobtracker can reschedule it on a different tasktracker
			- always running daemon process on the master node of the cluster
			- even when there are no jobs running, the jobtracker is responsible for monitoring the health and status of the TaskTracker nodes in the cluster
			- stores job history for completed jobs
				- but also possible to run a job history server as a separate daemon to take the load off the jobtracker
		- tasktrackers run tasks and send progress reports to the jobtracker
- in YARN, the responsibility of scheduling and task progress monitoring  is taken care by separate entities: the resource manager and an application master(one for each MR job) 
- in YARN, the responsibility of app history tracking is taken care by timeline server
- YARN equivalent of tasktracker is a node manager
- the following is a mapping of components between MR1 and YARN
![[Pasted image 20250106220811.png]]
- benefits to using YARN include the following:
	- scalability
		- MR1 hits bottlenecks in the region of 4000 nodes and 40000 tasks
			- because jobtracker has to manager both jobs and tasks
		- YARN overcomes these limitations by the virtue of its split resource manager/application master architecture
			- can scale up to 10000 nodes and 100000 tasks
		- in contrast to the job tracker, each instance of an application has a dedicated application master, which runs for the duration of the application
			- remember in MR1, jobtracker takes care of splitting the job submitted by user into map and reduce tasks
	- availability
		- high availability is usually achieved by replicating state of a service daemon, so that another daemon can pick up the left over work when current one fails
		- the state in jobtracker's memory is large, complex and constantly changing within seconds
			- each task status is updated every few seconds
			- these factors make it very difficult to retrofit HA into the jobtracker service
		- in YARN, the responsibilities of jobtrackers are divided between RM and AM
			- now to be highly available, divide an conquer
			- provide HA for each of the above daemons separately
			- Hadoop 2 supports HA both for the resource manager and for the application master for MR jobs
	- Utilitzation
		- in MR1, each tasktracker is configured with a static allocation of fixed-size slots
			- which are divided into map slots and reduce slots at configuration time
			- a map slot can only run a map task
		- in YARN, a node manager manages a pool of resources, rather than a fixed number of designated slots
			- there will never be a situation where MR will have to wait because only map slots are available on the cluster
			- if resources to run the task are available, then the application will be eligible for them
			- resources are more finegrained, so application can request in a granular way
				- in mr1, slots are indivisible
	- multitenancy
		- YARN opens up Hadoop to other types of distributed application beyond MR
		- allows us to run different versions of MR on the same YARN cluster, which makes the process of upgrading MR more manageable
			- some parts of MR - job history server and the shuffle handler, as well as YARN itself, still need to be upgraded across the cluster
### Scheduling in YARN
- job of the YARN scheduler is to allocate resources to applications according to some defined policy
	- there is no one "best policy"
	- that is why YARN provides a choice of schedulers and configurable policies
##### Scheduler options
- three schedulers are available in YARN: the fifo, capacity, and fair schedulers
- fifo
	- as the name suggests
	- requests for the first application in the queue are allocated first
	- simple to understand and not needing any configuration
	- not suitable for shared clusters
		- as large applications will use all the resources in a cluster
	- ![[Pasted image 20250107210041.png]]
- capacity
	- better for shared cluster
	- allows long running jobs to complete in a timely manner 
		- while allowing users who are running concurrent smaller ad hoc queries to get results back in a reasonable time
	- a separate dedicated queue allows the small job to start as soon as it is submitted
		- how is a job identified as small
		- this at the cost of overall cluster utilization since the queue capacity is reserved for jobs in that queue
		- large job finishes later than when using the fifo scheduler
	- ![[Pasted image 20250107210057.png]]
- fair
	- there is no need to reserve a set amount of capacity
	- it will dynamically balance resources between all running jobs
	- just after the first(large) job starts, it is the only job running, so it gets all the resources in the cluster
		- when the second job starts, it is allocated half of the cluster resources so that each job is using its fair share of resources
	- there is a lag between the time the second job starts and when it receives its fair share
		- it has to wait for resources to free up as containers used by the first job complete
	- after the small job completes and no longer requires resources, the large job goes back to using the full cluster capacity again
	- overall effect is both high cluster utilization and timely small job completion
	- better for shared cluster
	- allows long running jobs to complete in a timely manner 
		- while allowing users who are running concurrent smaller ad hoc queries to get results back in a reasonable time
	- ![[Pasted image 20250107210132.png]]

###### Capacity Scheduler Configuration
- allows sharing of Hadoop cluster along organizational lines
- each org is allocated a certain capacity of the overall cluster
- each org is set up with a dedicated queue that is configured to use a given fraction of the cluster capacity
- queues may be further divided in hierarchical fashion
	- allowing each org to share its cluster allowance between different groups of users within the org
- within a queue, apps are scheduled using fifo
- in capacity scheduler, a single job does not use more resources than its queue's capacity
- if there is more than one job in the queue, and there are idle resources available, then the CS may allocate the spare resources to the jobs in the queue
	- even if that causes the queue's capacity to be exceeded
	- the spare resources can be from other queue as well
	- this behavior is known as ***queue elasticity***
- in normal operation, CS does not preempt containers by forcibly killing them
	- a queue may be under capacity when it does not utilize all the resources allocated to it because of lack of demand
	- if the demand suddenly increases, it cannot immediately utilize resources which may have been lent to other queues 
		- and the scheduler does not forcibly reclaim them from other queues
	- the queue will only return to using the full amount of resources gradually
		- this happens as running containers in other queues complete and release resources
	- it is possible to mitigate this by configuring queues with a maximum capacity so that they dont eat into other queue's capacities too much
	- queues that are fully utilized and have increased demand **can allocate resources from other queues** to meet the new demand **only if resources become available** in those other queues
- it is possible to mitigate this by configuring queues with maximum capacity so that they dont eat into other queue's capacities too much
	- reasonable number found by trial and error
- capacity scheduler config file: capacity-scheduler.xml
	- a particular queue is configured by setting configuration properties of the form yarn.scheduler.capacity.queue_path.sub_property
	- queue_path: hierarchical dotted path of the the queue
		- root.dev.eng
	- this hierarchical name is only used and recognized in the config file

![[Pasted image 20250108100334.png]]
- dev's max capacity is 75 meaning, it's actual capacity 60 + 15 from prod in cases of high demand and free resources in prod
	- prod always has 25% exclusively for itself
- since prod does not have a max capacity it can end up using the full cluster
- capacity's unit is %
- we can also configure
	- max resources a single user or application can be allocated
	- max apps that can be running at any time
	- ACLs on queues
###### Queue placement
- the way to specify which queue an application is placed in is specific to the app
- in MR: mapreduce.job.queuename
	- if queue does not exist, we get error at submission time
- if no queue specified, app placed in a queue called default

#### fair scheduler config
- attempts to allocate resources so that all running apps get the same share of resources
- fair sharing actually works between queues, too
- in the context of fair scheduler, the terms pool and queue are used interchangeably 
- imaging 2 users A and B
	- A starts an application and since there are is no demand from B, all resources are allocated to A's app
	- then B starts a job while A's job is still running, and after a while each job is using half of the resources in the cluster
	- now if B starts a second job, it will share it's resources with B's other job
		- B's apps end up using 1/4 of the resources in the cluster while A's job continues to use 1/2 the resources in the cluster
	- resources are shared fairly between users
![[Pasted image 20250109083633.png]]

##### Enabling the fair scheduler
- the scheduler is determined by the setting of `yarn.resourcemanager.scheduler.class`
- the capacity scheduler is used by default
	- fair scheduler default in some Hadoop Distributions
- the config must be set in yarn-site.xml
- the value to be set must be a fully qualified classname of the scheduler

###### Queue configuration
- configured using an allocation file named fair-scheduler.xml
	- loaded from classpath
- the name of the file is also configurable
	- yarn.scheduler.fair.allocation.file
- in the absence of an allocation file, the fair scheduler operates as follows
	- each user gets a dedicated queue named after them
	- a queue for a user is created once the user submits the first application
- per queue configuration is specified in the config file
	- we can have hierarchical queues
![[Pasted image 20250109084511.png]]
- queue hierarchy is defined using nested queue elements
- all queues are children of the root queue
	- even if not actually nested in a root queue element
- weights are numbers that are used in the fair share calculation
	- in above case, the cluster allocation is considered fair when it is divided into a 40:60 proportion between prod and dev
	- since eng and science do not have weights specified, they are divided evenly
- weights are not the same as percentages, even though in the above example the weights add up to 100
	- weights can be thought of as ratios
	- even 2 and 3 could be given in the above example
- default queue and dynamically created queues are not specified in the allocation file but still have weight 1
- queues have different scheduling policies
- the default policy for queues can be set in the top-level defaultQueueSchedulingPolicy element
	- if omitted, fair scheduling is used
- fair scheduler supports fifo and Dominant Resource fairness policies on queues
- the policy for particular queue can be overriden using the scheduling policy element for the queue
	- prod above uses fifo
	- fair sharing is still used to divide resources between prod and dev queues, as well as between(and within) the eng and science queues
- queues can be configured with minimum and maximum resources, and a maximum number of apps
- min resources is not a hard limit, but rather is used by the scheduler to prioritize resource allocations
- if the two queues are below their fair share, then the one that is furthest below its minimum is allocated resources first
- the min resource setting is also used for preemption

##### Queue placement
- fs uses rules-based system to determine which queue an application is placed in
- queuePlacementPolicy element contains a list of rules, each of which is tried in turn until a match occurs
- in the above example
	- specified rule places an app in the queue it is specified with
		- if none specified or if the specified queue does not exist, the rule does not match
	- primaryGroup
		- tries to place an app in a queue with the name of the user's primary Unix group
		- if there is no such queue, rather than creating it, the next rule is tried
	- the default rule is a catch all and always places the application in the dev.eng queue
0831
0856