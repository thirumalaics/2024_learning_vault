- methods operate on objects
- scala has both functions and methods:
```
import scala.math._
sqrt(2) // A function
BigInt.probablePrime(100, scala.util.Random) // a method
```
- to invoke methods without parameters usually don't use ():
	- `"Hello".distinct`
	- rule of thumb is: () only required for mutators(does he mean setter), no parenthesis for accessor methods(does he mean getters?)
	- "Hello".length // the length method here does not mutate the string
- "Hello"(4) // yields 'o'
	- "Hello".apply(3) // same as above
## Conditional and block expressions
- an if expression has a value in scala
- val result = if (x>0) 1 else -1