- distributed typed collections of JVM objects
- first citizens of spark
	- all higher level APIs reduce to RDDs
- for RDD: partitioning can be controlled
	- order of elements and operations can be controlled
	- hard to work with
	- for complex operations, need to know the internals of spark
	- poor APIs for quick data processing
- spark context used to create RDD, he mentioned only sc can do this
	- parallelize existing collection
`val rdd: RDD[Int] = sc.parallelize(1 to 10000)`
- reading from file
```
Source.fromFile(filepath).getLines() // returns an  Iterator[String]
```
- fromFile returns a BufferedSource