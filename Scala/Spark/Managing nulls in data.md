- when we set nullable = false
	- it is not a constraint on the data itself
	- they are a marker for spark to optimize for null
	- this can lead to exception or data errors if broken
		- this is the reason why, in case of infer schema all fields are determined to be nullable

- to remove rows with nulls:
	- `df.select("*").na.drop()`
		- removes rows containing nulls from the df, in the above case all columns are checked for nulls and even if one contains nulls it is dropped
		- drop is a transformation
- to replace null in columns
	- `df.na.fill(0, List("x", "y"))`
	- columns x and y are considered for null replacement and the nulls are replaced with 0
	- not to confuse with replace method
- ifnull and nvl functions are available only in the sql api
	- takes only two expressions as arguments
- similarly nullif function is also available only in sql api
	- this function returns null if two columns or equal or returns the first column's value
- nvl2 takes 3 args
	- returns expr2 as result if expr1 is not null
	- returns expr3 as result if expr1 is null