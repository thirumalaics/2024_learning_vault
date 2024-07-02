- what is the difference between row_number, rank and dense_rank?
	- rank function
		- assigns the same rank for ties
		- next ranking is skipped
	- dense_rank
		- ties are given the same rank
		- ranks are consecutive
- fastest method to find duplicate records
	- distinct will cause results to be ordered
	- group by all our columns
	- https://stackoverflow.com/questions/2129717/how-to-verify-if-two-tables-have-exactly-the-same-data
	- https://stackoverflow.com/questions/5341276/sql-distinct-keyword-bogs-down-performance
- row_number performance
	- window function return a row for each input row
	- where as group by focuses on data reduction
	- use group by when we want to combine both aggregated and non-aggregated columns in a single query
	- ![[Pasted image 20240123094410.png]]
- shuffling in one executor with more cores than 10 machines
	- 
- how does set operations work and are they performant?
- how to find if three lines form a triangle
	- the sum of two side lengths of a triangle is always greater than the third side
- how to apply row number function using data frame api
```
from pyspark.sql import Window
from pyspark.sql.functions import row_number
partition = Window.partitionBy("").orderBy("")
df.select(row_number().over(window),"*")
```
- how to perform group by aggregation and filter based on the aggregation
- how to identify new, unchanged and updated records between source and target?
- how are nulls handled in bq order by and group by?
	- in bq order by nulls are considered the least possible values, so in ascending they appear first and in descending they appear last
https://www.google.com/search?q=why+group+by+performace+is+better+than+windowing&rlz=1C1RXQR_enIN964IN964&oq=why+group+by+performace+is+better+than+windowing&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIJCAEQIRgKGKABMgkIAhAhGAoYoAEyBggDECEYCtIBCTE1ODEzajBqN6gCALACAA&sourceid=chrome&ie=UTF-8
- following is one of the best explanations
	- https://stackoverflow.com/questions/13997177/why-no-windowed-functions-in-where-clauses
- continue in notebook
https://stackoverflow.com/questions/32565829/simple-way-to-measure-cell-execution-time-in-ipython-notebook
https://medium.com/swlh/revealing-apache-spark-shuffling-magic-b2c304306142


https://www.youtube.com/watch?v=t2R0-xcKw44
https://beginner-sql-tutorial.com/sql-query-tuning.htm
https://www.reddit.com/r/SQL/comments/siunp1/want_to_learn_query_optimization/