
- serverless architecture
- dwh = consolidates data from disparate sources and performs detailed analysis
	- 360* view of the data
	- be aware of real-time events
	- reduce time to insights
	- for analytics and decision support
		- detailed examination
- decoupled storage and compute
	- allows independent scaling
	- data can be analyzed using std SQL(2011 compliant)
	- connected through petabit nw
	- client libs provided

## Anatomy of a BQ query
- BQ democratizes Big Data
- four components
	- dremel
	- Colossus
	- Jupiter
	- Borg
## Infra
- BQ employs infra tech such as Dremel, Colossus, Jupiter and Borg
- Dremel is query engine
	- turns SQL into execution tree
	- branches of the tree are mixers, which perform the aggregation
	- shuffle takes advantage of Google's Jupiter nw to move data
	- borg takes care of spinning up resources
	- slots are components which perform computation and read data
	- slots are vCPU used by BQ to execute SQL queries
	- dremel is widely used at google, utube, mail etc
- colossus
	- google's latest gen distributed fs
	- each datacenter has its own Colossus cluster
	- each colossus has enough disks
	- handles replication, recover(when disks crash) and distributed management
	- fast enough to allow BQ provide similar performance to many in-mem dbs but much cheaper
	- ColumnIO columnar storage format and compression algo
- borg
	- job scheduler
	- cluster management system
	- runs on huge clusters with thousands of nodes
		- only fraction of the capacity of the Borg cluster is assigned to BQ
	- assigns server resources to jobs
- Jupiter
	- Google's nw
	- 1 Pb/s of bisectional bw
	- makes shuffle easier
## Pricing
- slots have two pricing models
	- on-demand
		- we will have access to up to 2000 concurrent slots for a project
		- BQ will temporarily go beyond this limit to accelerate smaller queries
		- cost based on columns selected even if we limit rows
		- we can cap the max bytes billed by the query
		- user and project level custom cost controls
			- project level quota limit the aggregate usage of all users
			- user-level are separately applied to all users and service accounts within a project
				- not for a specific user, groups then?
	- capacity compute pricing
		- predictable cost
- storage pricing
	- active storage
		- includes any table or partition that has been modified in the last 90 days
	- long-term
		- for any table or partition that has not been modified for 90 consecutive days
		- almost 50% price drop
		- no diff in performance or availability
- slots are assigned to queries fairly
- purchased slots must be assigned to one or more projects, folders or orgs
- projects that do not have an assigned or inherited reservation use on-demand pricing automatically


- GoogleSQL procedural language lets us execute as a multi-statement query
	- run multiple statements in a sequence, with shared state
	- automate management tasks, creating or dropping tables
	- complex programming logic such as If and WHILE
`DECLARE hello INT64 DEfAULT 0;`
- if no default, then null
- if variable type is omitted, Default is mandatory
- variable declarations must appear before other procedural statements
`SET (a, b, c) = (1 + 3, 'foo', false);`

```
from google.cloud import bigquery
import os
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = 'path/to/key/file'
client = bigquery.Client()
job_config = bigquery.LoadJobConfig(source_format=bigquery.Sourceformat.PARQUET)
job = client.load_table_from_uri(uri, table_id, job_config = job_config)
job.result()
```

```
from google.cloud import bigquery
client = bigquery.Client()
query = ""
query_job = client.query(query)
rows = query_job.result()
for row in rows:
	print(row.name)
```
https://cloud.google.com/bigquery/docs/reservations-intro#assignments


## How spark BQ connector works
- SBQC is built on top of BQ storage API and BQ API
- BQ Storage write API combines streaming and batching into one API
- in airflow, we can pass a list of jar paths as an argument to the operator
	- DataprocSubmitPysparkJobOperator
- writing dfs can be done using two methods: direct and indirect
	- direct method writes data directly to BQ using BQStorage write api
		- 'writeMethod':'direct'
	- indirect write method
		- data first written to GCS and then loaded to BQ using a bigquery load job
		- requires `temporaryGcsBucket`
- BQ views are not materialized by default
	- connector materializes and reads
	- affects read performance
	- reading from views is disabled by default
		- 'viewsEnabled' option should be set to true globally
- materialization dataset is where a temp table is created
	- the query that we run writes the result to the temp table and spark reads the temp table
https://medium.com/google-cloud/apache-spark-bigquery-connector-optimization-tips-example-jupyter-notebooks-f17fd8476309
https://github.com/GoogleCloudDataproc/spark-bigquery-connector/issues/585