```
val x = 1 + 2 //EXPRESSION
```
- expressions are evaluated to produce a value
- bitwise operators: &, |, ^,~, <<, >>, >>>(right shift with zero extension)
- equality check: 1 == x
- not equality: !=
- greater than: >
- greater than equal to: >=
- obviously <, <=
- boolean: !(x\=\=1), &&, ||
	- not, and and or
- +=, \-=,\*=,/=, \%= \-> side effects
- instructions vs expressions
	- instruction is some thing we tell the computer to do
		- change the var value, print to console
		- instructions are executed
	- expression: some thing that has a value and or a type
		- expressions are evaluated
		- in functional programming, we will think in terms of expressions
		- every single bit of our code will compute a value
		- ex: if expression
		- ```val aConditionedValue = if aCondition 5 else 3; val aConditiedValue```
		- here we call if expression and not instruction
		- this if condition can even be used in a println method
- loops
	- discouraged in fnal programming as they have side effects
	- they do not return anything meaningful
```
while (i < 10){ // this parantheses is important, if not given then scala asks for do keyword
	println(i)
	i+=1
}
```
- looping is very specific to imperative programming
- scala forces everything to be an expression
- only things like definitions of class or variables are not expression
- everything else is expression
	- calling func
	- reassigning a variable
```
val aVariable = 3
val aWeirdValue = (aVariable = 1)
```
- type of the weirdValue comes out to be Unit
	- special type that is equivalent to void in other languages
	- when aWeirdValue is printed the output is: '()'
- side effects in scala are expressions that returning a type unit
- while loops are side effects and that expression also returns Unit
```
val aWhile = while (i < 10){
	println(i)
	i+=1
}// aWhile is unit type
```
- side effects: println(), whiles, reassigning
- code blocks
	- special kind of expressions as they have special properties
```
val aCodeBlock = {
	val y = 2
	val z = y+1 // code block can have var as well
	if (z>2) "hello" else "goodbye" // the parantheses here is important, if not given then then keyword is expected
}
```
- the above piece of code is called code block
	- as mentioned above, code block is an expression
	- the value returned by the block is the value of the last expression
	- hence the type of aCodeBlock is string as the data type is also determined by the value returned by the last expression in the code block
	- code blocks can have their own definition of values, variables, so on and so forth
	- everything defined within a code block is visible within the code block and within the block only
	- even while loops can be nested within the code block