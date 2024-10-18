- lazy val defers the initialization of a variable
```
lazy val foo = {
	println("Initialized")
	1
}
```
- compiler does not immediately evaluate the bound expression of a lazy val
- it evaluates the variable only on its first access
- upon initial access, the compiler evaluates the expression and stores the result in the lazy val
- whenever we access this val at a later stage, no execution happens, and the compiler returns the result
- evaluated once during initial access and never evaluated again
	- this can be used when an operation takes long time to complete and we are not sure if it will be used at all

```
val z = {  
  println("computing z")  
  12  
}  
  
lazy val z1 = {  
  println("computing z1")  
  11  
}
```
- if I execute the above code block as is, I will get an output saying:
	- computing z
	- even though I am not accessing z
	- at the time of initialization itself the code block is evaluated
```
println(z) // 12
println(z1) // computing z1; 11 
println(z) // 12
println(z1) // 11
```
- even vals are evaluated only once but the difference is they are evaluated at the time of initialization itself
```
class Person1:  
  val x = {println(Thread.sleep(2000)); 12}  
  
end Person1  
  
class Person2:  
  val x = {Thread.sleep(2000); 12}
```
- executing the above without any print statements does not print anything as we are not initializing any object and classes are just templates
```
new Person1 // takes 2 seconds and prints ()
new Person2 // immideate execution
```
- even though x is not used we see that time taken while instantiating the class and this is because the x is evaluated immediately
- whereas Person2 object creation does not take much time at all because the x is not evaluated until it is accessed
- a compiled scala file results in a class file
- class files can be decompiled using a decompiler and it results in a equivalent java code for the scala code
https://stackoverflow.com/questions/7484928/what-does-a-lazy-val-do
https://www.baeldung.com/scala/lazy-val#:~:text=Scala%20provides%20a%20nice%20language,val%20has%20some%20subtle%20issues. - did not go through other than the basics