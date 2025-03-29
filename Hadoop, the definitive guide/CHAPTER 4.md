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
			- containers are requested on specific nodes where the required data already resides
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
- the queuePlacementPolicy can be ignored entirely, if done so the following is the default:
	- specified -> user
	- if the user queue does not exist, it is created
- another simple queue placement policy is one where all apps are placed in the same queue = default
	- in this case resources are shared fairly between applications rather than users
	- this behavior must be configured, there are two ways: 
		- xml config file
		- yarn.scheduler.fair.user-as-default-queue = false
			- apps will be placed in the default queue rather than a per-user queue
			- one more config need to be set:
				- yarn.scheduler.fair.allow-undeclared-pools should be set to false so that users can't create queues on the fly

##### Preemption
- when a job is submitted to an empty queue on a busy cluster, the job cannot start immideately
	- until resources free up from jobs that are already running on the cluster
- to make the time taken for a job to start more predictable, the fair scheduler supports preemption
- preemption allows the scheduler to kill containers for queues that are running with more than their fair share of resources
	- so that the resources can be allocated to the queue that is under its fair share
- preemption reduces cluster efficiency, since the terminated containers need to be re-executed
- preemption is enabled globally by setting yarn.scheduler.fair.preemption to true
- there are two relevant preemption timeout settings: one for minimum share and one for fair share
- both of these time out settings are not set by default, so we have to set at least one to allow preemption
- if a queue waits for as long as its minimum share preemption timeout without receiving its minimum guaranteed share, then the scheduler may preempt other containers
- defaultMinSharePreemptionTimeout = default for all queues and specified in the top-level element in the allocation file
	- on a per-queue basis by setting the minSharePreemptionTimeout element for a queue
- if a queue remains below half of its fair share for as long as the fair share preemption timeout, then the scheduler may preempt other containers
	- the default timeout is set for all queues via: defaultfairSharePreemptionTimeout top-level element in the allocation file
	- on per-queue basis by setting fairSharePreemptionTimeout on a queue
	- the threshold may also be changed from its default of 0.5 by setting defaultfairSharePreemptionThreshold and fairSharePreemptionThreshold(per-queue)

#### Delay Scheduling
- all the YARN schedulers try to honor locality requests
- on a busy cluster, if an app requests a particular node, there is a good chance that other containers are running on it at the time of the request
- obvious course of action is to loosen the locality requirement and allocate a container on the same rack
	- but waiting even for few seconds increases the chance of meeting the locality requirement dramatically, which in turn increases the efficiency of the cluster
	- this feature is called delay scheduling and it is supported by both the Capacity and fair scheduler
- every node manager in YARN cluster periodically sends a heartbeat request to the resource manager - by default, one per second
- heartbeats carry information about the node manager's running containers and resources available for new containers
- each heartbeat is potential scheduling opportunity for an app to run a container
- when using delay scheduling, the scheduler does not simply use the first scheduling opportunity it receives 
	- waits for up to a given maximum number of scheduling opportunities to occur before loosening the locality constraint and taking the next scheduling opportunity
- for the capacity scheduler, delay scheduling is configured by setting yarn.scheduler.capacity.node-locality-delay
	- value must be a positive integer representing the number of scheduling opportunities that is prepared to miss before loosening the locality constraint
- fair scheduler also users the number of scheduling opportunities to determine the delay
	- it is expressed as a proportion of cluster size
	- node locality threshold: specifies the proportion of nodes in the cluster that must present scheduling opportunities before the scheduler considers allocating the task to a different node in the same rack
		- within the time the threshold is reached, if there is node that meets the locality constraints, it is allocated to the job
		- yarn.scheduler.failr.locality.threshold.node to 0.5 means that the scheduler should wait until half the nodes in the cluster have presented scheduling opportunities before accepting another node in the same rack
	- yarn.scheduler.fair.locality.threshold.rack
		- specifies the proportion of racks in the cluster that must present scheduling opportunities before the scheduler considers allocating the task to a rack other than the one initially requested
		- ex: if set to 0.5, half of the racks in the cluster have had scheduling opportunities before considering scheduling on a rack other than the requested one

#### Dominant Resource fairness
- how to decide fairness of resource allocation?
- when there is only a single resource type being scheduled, such as memory, then the concept of capacity or fairness is easy to determine
- if two users are running apps, we can measure the amount of mem that each is using to compare the two apps
- but there are multiple resource types in play, things get more complicated
	- how to compare in the case where one user's app requires more CPU than mem, while the other requires more mem than CPU
- how the schedulers in YARN address the problem of determining fairness:
	- they look at each user's dominant resource and use it as a measure of cluster usage
	- this approach is called Dominant resource fairness or DRf
	- ex: Cluster: 100 CPUs and 10TB of memory
		- App A requests containers of: 2 CPUs, 300 GB
		- App B requests containers of: 6 CPUs, 100 GB
		- A's request in terms of percentage of resources (2%, 3%)
		- B's request in terms of percentage of resources (6%, 1%)
		- since B's container requests consumes twice as much of its dominant resource(6%) compared to A's dominant resource consumption(3%), app B will be allocated half as many ***containers*** as App A to ensure fairness
	- under DRf, the scheduler will ensure that both apps receive an equal share of their dominant resources
		- App A will get more containers because its dominant resource usage (3% per container) is smaller than the app b's dominant resource usage (6% per container)
		- App B will be allocated fewer containers to avoid disproportionate resource consumption
- DRf is not used by default, so during resource calculations, only memory is considered and CPU is ignored
- capacity scheduler can be configured to use DRf by setting yarn.scheduler.capacity.resource-calculator to org.apache.hadoop.yarn.util.resource.DominantResourceCalculator in capacity-scheulder.xml
- for fair scheduler, drf can be enabled by setting the top-level element defaultQueueSchedulingPolicy in the allocation file to drf

- without the DRf being enabled, how does fair scheduler determine fairness?
	- fairness based on absolute resource shares like CPU cores and memory rather than considering the dominant resource
		- assumes that all resources are equally important
	- with the same example as above, scheduler allocates and equal fraction of each resource (50% CPU and 50 % mem to each app if 2 apps are running)
		- regardless of whether one app consumes more of one resource type
		- this can lead to underutilization of one resource if an app does not full use its allocation of other resource
- Let’s revisit the earlier example with Application A and B:

- Application A’s container: **(2 CPUs, 300 GB)**.
- Application B’s container: **(6 CPUs, 100 GB)**.

#### Scenario:

- **Resource division (CPU)**: 50 CPUs for A and 50 CPUs for B.
    
    - Application A can run **50 CPUs ÷ 2 = 25 containers**.
    - Application B can run **50 CPUs ÷ 6 ≈ 8 containers**.
- **Resource division (Memory)**: 5 TB for A and 5 TB for B.
    
    - Application A can run **5,000 GB ÷ 300 GB ≈ 16 containers**.
    - Application B can run **5,000 GB ÷ 100 GB = 50 containers**.
- final containers for A = 16
- final containers for B = 8
- when the resources are divided equally, final num of containers allocated to each app depends on the most restrictive resource for that application
	- this often leads to underutilization of the less restrictive resource, hence DRf preferred
1354