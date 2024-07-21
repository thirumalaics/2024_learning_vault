- what is anti join?
	- returns rows from left table for which there are no corresponding matching rows in the rhs table
		- ```SELECT column1, column2,…FROM table1 LEFT JOIN table2 ON table1.column_name = table2.column_name WHERE table2.column_name IS NULL;```
- what is semi join?
	- semi-join returns rows from left table for which there are corresponding matching rows in the right table
		- only one row returned even if the row on the left has multiple matches with the rhs table
	- ```SELECT column1, column2, …FROM table1 WHERE EXISTS (SELECT 1 FROM table2 WHERE table1.column = table2.column);```
	- ```SELECT column1, column2, …FROM table1 WHERE column IN (SELECT column FROM table2);```
- what is pivot operation in SQL?
	- transform rows into columns or vice versa, to provide a summarized data view
	- while aggregating data based on specific criteria
	- more general example:
```
SELECT * from (
	SELECT Department, Year, Salary from Employee_Salaries
) AS SourceTable
PIVOT (
	SUM(Salary) for YEAR IN ([2022],[2023],[2024])
) AS PivotTable;
```
- in the above example, we are pivoting the data based on the YEAR column
	- for each unique combination of non-pivoted columns, a pivot column value may be repeated
		- hence when translating the rows with common pivot column values for unique combination of non-pivoted columns, we apply aggregation
		- basically when there is a one-many relationship between non pivoted columns to pivoted column, an aggregation is used to that the rows can be converted to columns
	- it might also be the case that  there is a 1:1 relationship between non-pivoted columns and pivoted column, even then it is important to provide agg
- for YEAR IN specifies the years for which we want to pivot the data
	```SELECT date,A,B,C from xyz PIVOT(SUM(Temp) for Category in ('A','B','C'))```
	https://medium.com/@dushyanthak/using-pivot-and-unpivot-in-sql-server-b7e0864ef92a