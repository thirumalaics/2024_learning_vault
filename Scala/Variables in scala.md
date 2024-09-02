## Data types
- Int, Float, Char, Byte
- using char, we can store string as well

- val and var
	- `val myvariable : Int = 10`
	- ![[Pasted image 20240902205930.png]]
	- while trying to compile, I got the above error
		- because I tried changing a immutable value
	- val defines an immutable variable and value of which can only be set during declaration
	- var myVariable2 : String = "How are you"
## Operations on variables

- dividing integer variable by integer variable gave me a rounded down integer
- string has builtin method length(): s1.length()
- print will not print a new line after printing whatever we give as argument, but println ends with a new line
- I cannot use single quotes for strings
- length method does not accept any argument, so it can be called without parentheses
- main function is mandatory in scala
- string concatenation: s1.concat(s2)
- single quotes is for char
- string cannot be concatenated with char
`C:\Users\malai\IdeaProjects\My first ever scala project\vars.scala`
`C:\Users\malai\IdeaProjects\My first ever scala project\operations_on_vars.scala`
