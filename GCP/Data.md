- structured data
	- data that fits into tables and includes discrete data types
	- must conform with the datamodel or schema
- unstructured data cannot fit into table
	- modeling of unstructured data as a table is inefficient
- snowflake schema
	- extension of star schema
	- where DTs are broken into subdimensions
- centralized repository that allows to store all our structured and unstructured data at any scale
	- written to as is
	- no data model applicable
	- dw is a db optimized to analyze data coming from different sources
		- data structure and schema are defined in advance to optimize for fast SQL queries
		- data in dw is cleaned, enriched
		- single source of truth
	- data in dl is not necessarily clean and integrated
	- for both structured and unstructured data
	- schema is not defined when data is captured
	- different types of analytics possible on the data stored in DL
- Stored procedures
	- set of statements that perform some actions
	- similar to fns in programming
	- enables reusability
	- a procedure can return a parameter
```
CREATE PROCEDURE dataset_name.procedure()
BEGIN
END
```

- NoSQL(Not only sql) dbs are non tabular databases and store data differently than relational tables
	- different data models available
		- main types: document, key-value, wide-column, and graph
		- flexible schemas and scale easily with large amounts of data
- document db
	- store and query data as JSON like docs
	- JSON is data interchange format
	- flexible
	- mongo db
	- ease of development
		- JSON docs map to objects
	- good for content management(videos, blogs)
- key-value db
	- stores data as a collection of key-value pairs in which a key serves as a unique identifier
	- both keys and values can be anything
		- ex: simple objects to compound
	- highly partitionable
	- horizonal scalability like no other db
	- usages: session management
	- shopping cart
	- here key is called primary key
	- the schema of the value is defined per value
	- redis
	- limited operations
	- ![[Pasted image 20240121091327.png]]
	- ![[Pasted image 20240121091404.png]]
- wide-column store
	- Google BT is a key-value and wide-column store
	- entity attr value model
		- one table to hold the entities
		- one to store attrs
		- one for attr values
		- these tables are linked through PK and fks, establishing relaationships
		- 
https://stackoverflow.com/questions/62010368/what-exactly-is-a-wide-column-store
https://blog.logrocket.com/nosql-wide-column-stores-guide/
https://www.dremio.com/wiki/entity-attribute-value-model/#:~:text=The%20Entity%2DAttribute%2DValue%20(,entities%2C%20attributes%2C%20and%20values.