starts from: [[Designing Data intensive applications/JAN/Week1/17|17/01]]
ends at : [[16|16/02]]
Contents:
1. [[Designing Data intensive applications/JAN/Week1/17#CH1 Reliable, Scalable and Maintainable Apps]]
	- technology around dbs and data processing have been rapidly progressing
		- driven by many factors
	- we call an app ***data-intensive*** if the primary challenge is data and data-related
		- challenges: quantity, complexity of data and the speed at which it is changing
	- behind the rapidly changing tech, there are principles that remain true, which helps us
	- DIA are built on top of building blocks which provide common functionalities
		- there many different building blocks to achieve the goal, we have to pick the one that suits us
	
2. [[Designing Data intensive applications/JAN/Week1/17#Thinking about data systems]]
	- access patterns: how a data storage system allows to read and write data
	- data storage and processing tools differ in what they are optimized for
	- one app can have multiple requirements that cannot be satisfied with one tool
		- work is broken down based on the suitable tool and different tools are stitched together by the app code
	- Questions while designing a data system: how do we
		- ensure data correctness and completeness, even when things go wrong internally?
		- provide consistent performance to clients, even when parts of the system are degraded?
		- how do we scale to meet increased load?
		- what does a good API for a service look like?
	- answer to these questions depend on many factors like time, people involved etc.
4. [[Designing Data intensive applications/JAN/Week1/18#Reliability]]
	- while designing a data system, 3 important concerns:
		- reliability
			- system should continue to perform correct function at expected performance even in case of different faults
		- scalability 
			- system should have reasonable ways of dealing with growth(wrt volume, speed and complexity)
		- maintainability
			- system should be easy to maintain by current and future developers
	- reliability expectations include
		- expected functionality and expected performance at the expected load
		- prevent unauthorized access or use
		- tolerate user mistakes or when sw is not used properly
	- fault: system component that deviates from it's specification
	- fault tolerant mechs prevent failures by dealing with faults
	- systems that can anticipate possible faults and cope with them are fault-tolerant and resilient
		- not all faults can be anticipated
	- failure: when the system stops providing expected service
	- faults that can be cured: hw, sw and human errors
5. [[Designing Data intensive applications/JAN/Week1/18#HW faults]]
	- ex: disk crashes
	- first response is a failover instance, which is a form of redundancy
		- until recently redundancy was sufficient
	- due to volume of data, num of machines used in general has increased
		- this increases rate of hw faults
		- this called for sw fault tolerant tech ***in addition to*** hw redundancy
		- this addition eases some pressure on the hw and allows time for updates
	- goal is to have systems that tolerate loss of entire machines
	- hw faults thought to be random and weakly correlated
6. [[Designing Data intensive applications/JAN/Week1/19#SW Errors]]
	- systematic sw errors are harder to anticipate
	- highly correlated across nodes and hence cause many sys failures
	- check for examples under above heading
	- usually caused by dormant bugs that are triggered by unusual circumstances
		- usually happens when a code's assumptions stops being true
	- no quick soln, check above heading for some best practices
7. [[Designing Data intensive applications/JAN/Week1/19#Human Errors]]
	- config errors
	- design systems that minimize the opportunity for error
	- decouple places where people make mistakes from the place where mistakes are not accepted
	- look for more points under above heading
8. [[Designing Data intensive applications/JAN/Week1/19#Scalability]]
	- ability of a system to cope with increased load
	- discussing scalability means considering questions like:
		- If the system grows in a particular way, what are our options for coping with the growth?
		- how can we add computing resources to handle the additional load?
	- describe current load before even trying to answer these questions
	- numerically define performance and load
9. [[Designing Data intensive applications/JAN/Week1/19#Describing load]]
	- load described with a few numbers called as load parameters
	- best set of params for the system depends on the architecture
	- see example twitter use case under above heading
10. [[Designing Data intensive applications/feb/Week3/12#Describing performance]]
	- define the normal performance numerically first before going on about scalability
	- in batch processing 2 metrics:
		- throughput: num of records processed per second
		- total time: to process the entire data
			- cannot be derived from throughput as this metric is affected by skews or other reasons
	- in online systems:
		- response time: time bw a client sending a request and receiving a response
		- this time includes: service time(processing time),queuing delays and nw delays
		- latency != response time
		- latency: duration during which a request is waiting to be handled(awaiting service)
		- response time should be evaluated as a distribution and not as an average
11. [[Designing Data intensive applications/feb/Week3/13#Approaches for coping with load]]
	- percentiles used in SLOs and SLAs
	- queuing delays responsible for response time in the higher percentiles
		- amount of time spent by a request in queue waiting to be processed
	- head-of-line blocking: slow requests at the head, blocking the processing of next requests
	- high percentile important in backend services
		- called multiple times even for a single request
		- even if small % of backend requests are slow, due to the amount of calls of BES, the effect is huge - tail latency amplification
		- so higher proportion of requests end up slow
		- a small percentage of high latency affecting the response time of many requests
	- good arch includes both scaling out(shared nothing arch) and up
	- systems that scale automatically: elastic
	- distributing statless workload is easy but stateful wl distribution is complex 
	- ex: a sys that is designed to handle 100,000 requests per second, each 1kb in size is not the same as
		- a system designed for 3 requests per minute, each 2 GB in size
		- both systems have the same throughput though
12. [[Designing Data intensive applications/feb/Week3/14#Maintainability]]
	- majority of the cost of sw is in its on-going maintenance
		- examples in above heading
	- legacy system: existing system may or may not be based on outdated tech but is critical to day-to-day operations
		- can also mean the systems that were forced to do things they were never intended for
	- maintenance can be made easy by prioritizing
		- operability
			- make it easy for operations teams to keep the system running smoothly
		- simplicity
			- make it easy for new engineers to understand the system, remove  complexity from the system
		- evolvability
			- make easy for changes
13. [[Designing Data intensive applications/feb/Week3/14#Operability Making life easy for operations]]
	- operations: management and maintenance of IT infra within a business
		- involves automation for repetitive tasks
	- ensures systems are secure, reliable and compliant
	- good operations can often work around limitations of bad(or incomplete) sw
	- good sw cannot run reliably with bad operations
	- check above heading for responsibilities of operations team
	- operability also means making routine tasks easy
		- data systems should allow automation and integration with std tools
		- self-healing, good default behavior, avoiding single dependency
14. [[16#Simplicity Managing Complexity]]
	- complexity increases cost of maintenance
	- invites bugs while changes
	- accidental complexity: complexity is not in the problem that the sw solves but arises only from the implementation
		- soln: abstraction
15. [[16#Evolvability Making Change easy]]
	- agile working patterns
		- test-driven development and refactoring

