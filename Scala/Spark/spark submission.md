- spark.master config's value determines the cluster manager to connect to
	- if the url starts with spark, then the cluster manager used is spark standalone
	- if we need to use yarn, the value must be yarn, looks like no need of a url port and all
		- how is it possible without a URL
https://spark.apache.org/docs/latest/quick-start.html#self-contained-applications

https://spark.apache.org/docs/latest/running-on-yarn.html