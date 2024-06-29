- historically data was represented as one big tree - hierarchical model
	- not good for MTM
- RM was invented to solve the above problem
	- recently developers found that, some apps do not fit well in the RM either
		- cases which have 
			- complex many to many
			- non-conformance to structure
- for this, non-relational NoSQL datastores have diverged in two main directions:
	- DDBs: for use cases where data comes in self-contained documents(self-contained means nested)
		- relationships bw one document and another are rare
		- even if there is a relationship, then the entire record is nested within
	- GDB: use cases where anything is potentially related to everything
- RDM, DDM and GDMs are widely used today
- each is good in its respective domain
- one model can be emulated in terms of another model
	- ex: GDM emulated by RDM
		- result is awkward
	- that is why we have ***different systems*** for ***different purposes***
- similarities: DDB and GDB do not enforce schema
	- best for changing requirements
	- this is different from no-schema
	- the data is assumed to have a certain structure, it is either explicit(SOW) or implicit(SOR)
- Query langs discussed: SQL, MR, MDB's Agg pipeline, Cypher, SPARQL, and Datalog
- some other DMs that were not discussed
	- GenBank specialized to handle ***sequence similarity searches*** of genome data(strings of DNA)

1. [[16#CH 2 Data Models and Query Languages]]
	- dm determine how the sw is written and how do we think about the problem we are solving
	- most apps are built by layering one data model on top of another
	- a layer hides complexity of the layers below it
	- data models have it's own assumptions on how the data is going to be used
		- some operations are faster and others are slow/impossible
	- data model determines what the sw above it can or cannot do
2. [[Designing Data intensive applications/feb/Week3/17#RM v Document Model]]
	- involves relations(tables in SQL) and tuples(rows in SQL)
		- covered multiple times, check book
	- use cases of RM are txn processing(entering txns) and batch processing(customer invoicing, payroll)
		- definition for both in above heading
	- batch processing acts on the data written by txn processing
	- back in the days other dbs forced app devs to think about internal repr
	- RM hid the implementation details
	- RM later generalized for diff use cases
3. [[Designing Data intensive applications/feb/Week3/17#Birth of NoSQL]]
	- then came other requirements
		- greater scalability than RM can easily achieve
		- preference for open-source instead of commercial db products
		- specialized query operations not supported by RM
		- desire for dynamic and expressive data model
	- these requirements gave birth to nosql
4. [[Designing Data intensive applications/feb/Week3/17#Object relational mismatch]]
	- most app dev today in OOP lan
	- a translation layer is required OOP and rdb structures
		- the disconnect bw the two models is called impedance mismatch
	- ORM fws reduce the amount of boiler plate code required for translation layer
	- example how multi valued data can be stored in the above heading
	- some devs feel impedance mismatch is reduced with JSON model
		- but there are problems with json as a data encoding format - [[CH4]]
	- json has better locality - all info about one entity in one place
5. [[Designing Data intensive applications/feb/Week3/17#Many-to-one and Many-to-Many relationships]]
	- std way to store std values in above heading and it's adv
	- ddb faces difficulty addressing mtm relationships
	- similar to what hm faced back in the days
	- eventually the dms will have to start storing mtm data
	- hm: every record has one parent
6. [[Designing Data intensive applications/feb/Week4/18#NW model]]
	- CODASYL model
	- was introduced to solve limitations of HM during the same time RM was also proposed
	- nm: a record could have many parents
	- allowed MTO and MTM to be modeled
	- links bw records were more like pointers than fk
	- only way to access a record was to follow a path from root record along these chains of links
		- called ***access path***
		- dev need to keep track of what record has been visited because of MTM relationship
	- code becomes complex because of the memorization of path
	- knowing the path to required record is important and makes things easier
	- change in path is change in code
7. [[Designing Data intensive applications/feb/Week4/18#relational model]]
	- in RDBMS, query optimizer determines what order should the parts of the query need to be executed and what indexes to use
		- can be seen as similar to access paths
	- queries do not change in order to make use of new index
		- appropriate index chosen automatically
8. [[Designing Data intensive applications/feb/Week4/18#comparison to document dbs]]
	- ddbs store nested records within their parent record, similar to hm
	- fk concept in both to represent MTO and MTM rls
		- document reference in DM
		- fk in RM
9. [[Designing Data intensive applications/feb/Week4/18#Relational v DDBs today]]
	- many differences: fault tolerance properties and concurrence handling
	- wrt data model:
		- DocM: schema flexibility, better performance due to data locality and similarities with data structures of some applications
		- RM: better support for joins, MTO and MTM
	- doc-like structure: tree of OTM RLs, where entire tree is loaded at once
		- use DM if data had doc-like structure
		- as the doc gets more nested, it becomes difficult to access elements
	- denormalization comes with the cost of keeping the data consistent
	- if joins are required Doc model pushes complexity to code
	- for highly connected data RM is acceptable and graph models are natural
10. [[Designing Data intensive applications/feb/Week4/19#rdb v ddbs compared contd.]]
	- most ddbs and json support in RM does not enforce schema on data in docs
	- XML support in RDB usually comes with optional schema validation
	- schemaless: an implicit structure is there but the db does not enforce it
		- code assumes some kind of structure
		- most accurate term for ddbs is schema-on-read
		- similar to runtime type checking
	- schema-on-write: schema is explicit and db ensures all written data conforms to it
		- similar to compile-time type checking
	- schema changes in docdb is as simple as just starting to write in new schema and code should be able to handle while reading old and new data
	- in rdb table has to be altered and updated
	- heterogenous data may be due to
		- each different object cannot be put into it's own table
		- the structure is determined by an external system
11. [[Designing Data intensive applications/feb/Week4/19#data locality for queries]]
	- a doc usually stored as a single string, encoded as JSON, XML or a binary variant
	- better for apps which need access to the entire/large part of doc
	- db loads up entire doc even if only small portion required
	- updates usually rewrite entire doc
		- only modifications that do not change the encoded size of the doc can easily be performed in place
	- data locality feature can be seen in some RDBMSs: Google Spanner, oracle with multi table index cluster tables
12. [[Designing Data intensive applications/feb/Week4/19#convergence of document and relational dbs]]
	- most rdbs have supported XML including modifications to parts of XML, indexing and querying inside XML docs
		- allows apps to use dm similar to ddm
		- similar support for JSON is seen
	- ddbs support for relation-like joins are seen in some ddbs(joins in client side)
		- likely slower due to nw round trips
13. [[20#Query Langs for data]]
	- SQL is declarative
	- CODASYL(NWM) queried imperative
	- imperative: what to do and in what order
		- hard to parallelize due to spec of order
		- less visibility with the query runner
	- declarative: what to do but not how
		- how taken care by query optimizer
		- performance improvements without requiring any changes to queries
		- lends themselves easily to parallel executions
14. [[20#Declarative queries on the web]]
	- check heading
15. [[21#MapReduce Querying]]
	- MR is programming model for large data processing across machines
	- neither imperative nor declarative fully
	- snippets of code called repeatedly by processing fw
	- check above heading for ex of a MR Code
	- restrictions: map and reduce fns should be pure functions
		- no side effects, use only data passed as input
		- allows running queries anywhere in any order and rerun them on failure
	- many distributed implementations of SQL that dont use MR are present
	- SQL can be implemented as a pipeline of MR operations
	- some SQL dbs allow to write js funcs in queries
	- MR requires two coordinated functions instead of one single query
		- auto-query optimization not included in MR
	- in account of the the draw backs DQL was introduced in MongoDB
16. [[21#Graph-like DMs]]
	- as seen handling of MTM rls is a distinguishing factor among data models
	- two objects in graphs
		- vertices(nodes or entities) and edges(relationships)
	- exs in above heading
	- graphs not limited to homogenous data
17. [[]]