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
- iterators have a method called drop which can be used to drop the first n elements
`Source.fromFile(filepath).getLines().drop(1)` 
- drops the first line
```
def readStocks(path: String): List[StockValue] = {  
  Source.fromFile(path).getLines()  
    .map(_.split(","))   //returns an array of Strings
    .drop(1)  
    .map(array => StockValue(array(0), array(1), array(2).toDouble))  
    .toList  
}  
val myfirstRDD = sc.parallelize(readStocks(filepath))
```

- there is another method to read from files as rDD
```
sc.textFile("src/main/resources") // returns an RDD oF String
```
```
val mySecondRDD: RDD[Array[String]] = sc.textFile(filepath)  
  .map(_.split(",")).filter(_(3) != "price")
``` 

- the easiest way to create a rdd is from a dataframe
	- convert it to a Dataset and then access the rdd attribute
```
import spark.implicits._  
val ds = spark.read.format("csv").options(Map(  
  "header" -> "true",  
  "inferSchema" -> "true"  
)).load(filepath).as[StockValue]  
  
val myThirdRDD: RDD[StockValue] = ds.rdd
```
- dataframes also have rdd attribute but they return rdds of type row
	- `df.rdd`
- rdds can be converted to df using the toDF method available For RDDs
	- but For some reason there was no toDF method available For RDD[Row]
	- whereas the same method was available for RDD[StockValue]
- when we convert an RDD to Df, we lose the type information
`val numbersDS = spark.createDataset(numbersRDD)`
