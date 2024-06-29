- java database connectivity
- most dbs support jdbc
- jdbc mainly for comms bw application and db
- JDBC is a java API to connect and execute queries with the db
- provides a std abstraction to achieve above results
- JDBC drivers are client-side adapters(installed on client machine, not on the server)
	- the adapter: sw design pattern that allows the i/f of an existing class to be used as another i/f
	- JDBC drivers convert requests from Java Program to a protocol that the dbms can understand
	- ![[Pasted image 20240626190607.png]]
## jdbc Architecture
- what is the JDBC architecture
	- consists 2 components
		- JDBC API
			- JDBC Driver Manager
		- JDBC Drivers
- two key layers
- JDBC API: 
	- collection of various classes, methods and i/fs for easy communication with a db
	- two major sets of I/fs
		- JDBC Core API(java.sql)
			- provides basic connectivity and SQL functionality
			- Driver Manager part of this API
		- JDBC Optional Package(javax.sql package)
			- offers advanced features like connection pooling, distributed txns, row sets
- JDBC Driver Manager 
	- class in JDBC Core API that loads a db-specific driver in Java App
	- driver manager makes a call to db using driver to process request of user
	- the loading of db specific driver is not done by any method of Driver Manager
	- there is `Class.forName("com.mysql.jdbc.Driver")` - this is more general
		- https://www.geeksforgeeks.org/class-forname-method-in-java-with-examples/
- Drivers: 
	- jdbc drivers are db specific implementations provided by db vendors or third parties
		- facilitate actual connection to the db
	- library that implements JDBC API
### JDBC Drivers
- diff types
- type1: JDBC-ODBC Bridge driver
	- provides access via odbc drivers
	- not commonly used
	- requires an ODBC driver to be installed on client machines
		- odbc drivers written in C or C++
	- can be used when we want to connect to a db which does not have a native JDBC driver
	- poor performance and security vulnerabilities
	- ![[Pasted image 20240625180733.png]]
- type2: native API driver
	- convert JDBC calls into DB specific calls
	- requiring native db libs on the client machine
	- the native library comms with the dbs through the db's native API
	- job of this driver is to convert jdbc calls into native API calls
	- faster than type 1 but it is not portable
		- as there is a requirement of external lib
	- bound to the native api
	- to be avoided in new apps
	![[Pasted image 20240625181006.png]]
- type3 NW protocol driver
	- translates jdbc calls into a db-independent nw protocol to a server
		- this is mostly a server side middleware
		- comms with this server is through db independent protocol
	- the server then translates the calls to db specific calls
	- does not require db specific drivers on client machines
	- the independent protocol is written in Java, and the driver translates the JDBC calls into the protocol
	- the calls are sent to a middleware server that translates protocol into the db's native API
	- slower than type 2 because of extra comms
	- portable
	- below diagram is wrong in the middle ware section
	- ![[Pasted image 20240625181349.png]]
- type4: thin driver
	- direct-to-db pure java driver
	- converts JDBC calls directly into the db specific protocol
	- most popular
	- does not require any additional setup or libraries on client machine
	- comms with db using db's native nw protocol
	- most efficient and portable
	- no additional layer of comms
![[Pasted image 20240625181607.png]]- 


![[Pasted image 20240624184616.png]]


## ODBC
- C programming language interface
- allows apps to access data from variety of dbms
- specifically developed for relational data stores
- developers of dbms specific drivers implement ODBC API fns
- i/f bw app and data source
- separate driver for each data source
- 

https://rajasrakhe20.medium.com/java-database-connectivity-jdbc-7f148ad06286
https://medium.com/@AlexanderObregon/effective-jdbc-programming-a478947c8426
https://medium.com/@cyberblogger007/jdbc-java-database-connectivity-86902aca1b6c

https://en.wikipedia.org/wiki/Java_Database_Connectivity#JDBC_drivers
https://refactoring.guru/design-patterns/adapter
https://www.scaler.com/topics/jdbc-architecture/
[What is an ODBC Driver (devart.com)](https://www.devart.com/odbc/what-is-an-odbc-driver.html)
[Difference Between JDBC and ODBC - Naukri Code 360](https://www.naukri.com/code360/library/difference-between-jdbc-and-odbc)