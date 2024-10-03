- datasets are typed dfs
	- distributed collections of jvm object
- dfs are distributed collections of type row
- to manipulate any data, so far we have been resorting to use methods provided for the dataframe object
	- what if we want to use predicate defined using a scala function
- to create a ds of single column
```
import org.apache.spark.sql.{Dataset, Encoders, SparkSession}
implicit val intEncoder = Encoders.scalaInt // exp that will return an encoder of int
val numbersDS: Dataset[Int] = df.as[Int]
```
- an encoder of int has the capability of turning a row in df to an int
	- which will be the type of jvm objects of numberDs