## Basics
![[Pasted image 20240120210431.png]]
- Airflow's 3 main components pg: 37
	- scheduler
	- workers
	- webserver
- scheduler
	- parses DAGs and stores extracted information in the db
	- checks their schedule interval and if the DAG's schedule has passed, starts scheduling the DAG tasks for execution by passing them to the worker
- Worker
	- pickup up tasks that are scheduled and executes them
	- actually responsible for doing the work
- webserver
	- visualizes the DAGs parsed by the scheduler and provides the main interface for users to monitor DAG runs
- at a high level, scheduler runs through the following steps:
	1. python files in the DAG folder are read by the scheduler
		- extracts corresponding tasks, dependencies and schedule interval of each DAG
	2. for each DAG, the scheduler then checks whether the schedule interval has passed since it's last read. if so tasks in the DAG are scheduled for execution
	3. for each scheduled task, the scheduler checks for the dependencies have been completed. those tasks are added to the execution queue
	4. go back to step 1 after waiting for certain time
![[Pasted image 20240120211123.png]]

- metastore stores xcom, variables and connections
- start the db first, then scheduler then webserver
## Scheduler in depth
- the scheduler has multiple responsibilities
	- parsing DAG files and storing extracted information in the db
	- determining which tasks are ready to execute and placing these in the queued state
	- fetching and executing tasks in the queued state
- scheduler has three components -> pg 312
	- the DAG processor
	- Task Scheduler
	- Task Executor
- The DAG processor
	- periodically processes py files in the DAGs dir(configurable)
		- even if there is no change in the DAG file
	- this processing takes processing power
	- the interval of DAG processing is controlled by 4 configurations
		- time to wait after a scheduler loop -> if small, then faster DAG parsing
		- min time to refresh the list of files in the DAGs folder
		- max number of processes(not threads) to use for parsing
	- DAG file processing statistics can be seen in dag_processor_manager.log
		- we can configure how often stats get written to this file
- Task Scheduler
	- determines which task instances may be executed
	- a while True loop periodically checks for each task instance if a set of conditions are met
		- all upstream dependencies are satisfied
		- end of schedule interval is reached
		- if the task instance in the previous DAG ran successfully(if depends_on_past=True)
	- when all these conditions are met, it is set to a scheduled state
	- another loop in the scheduler determines another set of conditions that transfer tasks to queued state
	- the conditions are:
		- enough open slots are available
			- slots: a fixed number that represents access to a resource
			- resource pools are a way to restrict traffic to a resource
		- if certain tasks have priority over others(priority_weight)
	- task instance will be added to execution queue
	- converts the state as queued
		- no longer responsibility of the scheduler
		- responsibility of executor to read task instance from the queue and start the task on a worker
- task executor
	- executor fetches task instance from queue and executes it
	- registers each state change in the metastore
	- executing means, creating new process for the task to run in
		- runs `airflow tasks run` in the new process
			- right before executing this command, airflow registers the state of the TI as running in the metastore
		- so that it does not bring down airflow if something fails
	- the process is periodically checked for status
		- another while true loop
		- wait time configurable
		- status successful only if exit 0
	- type of queue and how a task instance is processed once it is on a queue depends on the executor
	- the executor part of the scheduler can be configured in many ways
		- from single process on a single machine to multiple processes distributed over multiple machines



## Task Instances
- Task: basic unit of execution in Airflow
- three basic kinds of Task:
	- Operators: predefined task templates that we can put together quickly to build most parts of our DAGs
	- Sensors: a special subclass of Operators which wait for an external event to happen
	- Taskflow decorated @task, which is a custom py fn packaged up as a task
	- all these are subclasses of Airflow's BaseOperator
	- Operators and Sensors are templates
	- when we call one in a DAG file, we are making a task
- DAG is instantiated into a ***DAG run*** each time it runs
- tasks under a DAG are instantiated into ***Task Instances***
- an instance of a Task is a specific run of that task for a given DAG and execution date
	- they represent the task with it's state
- possible states of [Task Instance](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html#:~:text=The%20possible-,states,-for%20a%20Task)
## Executor
- different execution modes are configured by the type of executor
- executors are the mechanism by which ***task instances*** get run
- pluggable component
- airflow provides some out of the box options configured in core section of the configuration file
	- external executors can be written and used
- 4 types at the time of writing:
	- Sequential, Local, Celery and Kubernetes
![[Pasted image 20240617084933.png]]
- Sequential
	- one machine
	- tasks run sequential
	- used for testing and demo
	- default
- Local
	- single machine
	- not limited to a single task at a time
		- multiple tasks in parallel
	- internally registers tasks to execute in a py fifo queue, which worker processes read and execute
	- can run upto 32 parallel processes(configurable)
- for distributing workloads across machines:
	- Celery, Kubernetes
	- need: redundancy, faster runs, not being limited by resources
	- Celery
		- internally applies Celery Distributed task queues
		- workers read and process tasks from the queue
		- for user's perspective, it works the same as LE
		- main diff: all components can run on diff machines
		- Celery supports RabbitMQ, Redis and AWS SQS for the queuing mech(called the broker)
		- comes with monitoring tool called flower for inspecting the Celery System
		- Celery is py lib that integrates well with Airflow
		- CLI command: `airflow celery worker` starts a celery worker
		- only external dep for the setup is the queuing mechanism
	- Kubernetes:
		- requires setup and configuration of a Kubernetes cluster on which to run Airflow
		- the executor integrates with the Kubernetes APIs for distributing Airflow tasks
		- every task in Airflow DAG is run in a Kubernetes Pod
- Page 316
- following starts from page 317
### SequentialExecutor
- task executor part of the scheduler is run in a single subprocess
	- within this tasks are run one by one
	- slowest method of task execution
![[Pasted image 20240622095322.png]]

### LocalExecutor
- architecture is similar to the SequentialExecutor
	- only diff is multiple subprocesses
- each subprocess executes one task
- subprocesses can run in parallel
- subprocesses created by the executor run on the same machine
- but the other components can run on diff machines
- executor is a configurable components
- max num of subprocesses that can be spawned by the executor is configurable - default 32
- the subprocesses are not new processes but rather processes forked from the parent(scheduler) process
- the system is limited by the resources of the scheduler machine
![[Pasted image 20240622095606.png]]
### CeleryExecutor
- built on top of the celery project
- both scheduler and celery workers require access to both DAGs and the dbs
- access to DB is not a problem as the workers can connect with a client
- for the DAG folder to be available to all machines, we need to have a shared file system or a containerized setup where the DAGs are built into an image with Airflow
	- in containerized setup, any change to DAG code means re-deployment
- download of Apache airflow celery required
- Celery requires a queuing system and it should be supported by Airflow - Redis, RabbitMQ etc..
- queue is called the broker
- after installation of broker, we need to configure Airflow to use the broker
- scheduler sends messages to the queues
- celery workers need to be explicitly started
- in flower we can see the workers registered, status of them and high level info on the tasks each worker has processed
- celery relatively easy to set up than kube
	- celery has one extra comp: queue
- flower integrated into airflow
![[Pasted image 20240622101024.png]]

## kubernetes executor
### Workers
- are processes which can listen to one or more queues of tasks
- workers can be configured to listen to only specified queues
## different levels of configurations
## Web Server
- Provides the UI of Airflow
- manage and monitor DAGs
- visualizing our workflows
## Metastore
- everything that happens in airflow is registered in a db(metastore)
	- scheduler interprets and stores all components of a DAG in mdb
- airflow performs all db operations using SQLAlchemy
	- used to write Py objects directly into db
	- only dbs supported by SQLAlch is supported by Airflow
	- Airflow reccos PostgresSQL and MySQL
	- SQLlite supported but only with SequentialExecutor, as no support for concurrent writes
- airflow db init creates a SQLite db in $Airflow_Home/airflow.db
- in case we want postgres or mysql, create the db first and next configure in Airflow with the connection string
![[Pasted image 20240617091414.png]]
- stores 
	- user login information and permissions
	- connections, variables and XComs
	- Data about dag and task runs which are created by the scheduler
	- tables which store DAG code in diff formats
## Default Arguments
## XCOM
- mechanism that let Tasks talk to each other
	- because tasks are isolated and may be running on different machines
	- only small pieces of data
	- enables some level of shared state
- XCOM is identified by a key
	- as well as task_id and dag_id
- the value should be serializable
- many operators will auto-push their results into an XCOM key called `return_value`
	- xcoms are pushed by default
	- key is not mandatory for xcom pull
- XCOMs are per-task instance and designed for comms within a DAG run
	- but it is entirely possible to pull XCOMs from other DAGs and execution dates
		- we can pull xcoms for any combination of DAG and execution date
- XCOMs published using xcom_push method available in task context
 ![[Pasted image 20240613093353.png]]
 - the model_id value is registered for the combination of: task_id, dag_id and execution date
![[Pasted image 20240121093700.png]]
- xcoms introduce implicit dependencies bw tasks
	- this dependency not taken into account while scheduling tasks
- some operators allow for automatically pushing xcom values(configurable)
	- ex: xcom_push=True in BashOperator pushes the last line written to stdout to XCOM
	- ex:py pushes any value returned from a py callable to Xcom

- DAG run is a physical instance of a DAG, containing task instances that run for a specific execution date
- DAG is a collection of all tasks we want to run
- execution_date is the logical date and time which the DAG run, and the task instances, are running for
When not to use XCOM?

## Templates
- double curly braces denote a variable inserted at runtime aka Jinja templated string
- Jinja is a templating engine
	- replaces variables and/or expressions in a templated string at run time
- any python var or expression can be provided
- not all operator args can be templates
- every operator can keep an allowlist of attrs that can be made into templates
	- this list is set by attr template_fields
	- value inside the above list are argument names to the operator
![[Pasted image 20240121094706.png]]
- all the variables available within the context are available at runtime 
- in airflow 2, the PythonOperator determines which context variables must be passed along to our callable by inferring the callable argument names
- context variable is a dict of all context variables
- only strings can be templated
![[Pasted image 20240121114012.png]]

- each operator can read and template files with specific extensions by providing the file path to the operator
	- the list of allows file extension that can be templated are stored in template_ext
- jinja requires us to provide path to search for files that can be templated
	- by default, only the path of the DAG file is searched for
	- template_searchpath is a DAG level argument that can be provided to add additional paths
- templates_dict
[python - What is a difference between op_kwargs and templates_dict od PythonOperator in Airflow? - Stack Overflow](https://stackoverflow.com/questions/72710645/what-is-a-difference-between-op-kwargs-and-templates-dict-od-pythonoperator-in-a)

1. different states in airflow(ex: scheduled, queued, etc...) and the flow
2. more details about the metastore
3. more details about the executors
4. what is a task instance
5. execution date and schedule interval