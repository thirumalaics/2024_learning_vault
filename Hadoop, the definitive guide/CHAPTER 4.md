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
	- users can run their own instances of an application on a cluster, independently of other users, which means that different users can run different versions of the same app
1434