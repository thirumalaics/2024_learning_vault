- fn that takes another fn as an argument and extends the behavior of the argument fn without explicitly modifying it
## first-class objects
- in functional programming, we work entirely with ***pure functions***
	- pure functions ***do not have side effects*** like modifying a global variable
	- pure fns return the ***same result for the same input***
	- only result of a pure fn is the return value
	- input -> process -> output
	- ex: pow, sqrt
- python is not a purely functional language
	- but it supports many fp concepts, including treatment of fns as ***first-class objects***
		- what is first-class object?
			- no restrictions on the object's use
			- it can be dynamically created, destroyed, passed to a function, returned as a value, and have all the rights as other variables in the PL have
			- ![[Pasted image 20240517092023.png]]
		- this means that functions can be passed around and used as arguments
		- just like any other object like str, int etc...
- a fn name without the parentheses is a reference to a function

## Inner functions
- these are functions that are defined inside other functions
![[Pasted image 20240516092406.png]]

- the order of definition of the inner functions does not matter
- the code under the inner functions are executed only when they are called
- the inner functions are not defined until the parent function is called
	- they are locally scoped to parent
	- they exist as local variables within the parent function
	- cannot be referenced outside the parent
#### functions as return values
- functions can be returned from other functions
![[Pasted image 20240516092739.png]]

- in the above, function references are returned 
	- now they can be called outside
	- but we still cannot directly access first_child() or the second_child function

## Simple decorators in python
![[Pasted image 20240516093117.png]]
- say_whee is redefined outside by applying decorator to the original say_whee
	- after applying the decorator function, the say_whee variables points to the wrapper function
	- and the wrapper function has the reference to the original say_whee function and calls it in between the print statements
- when we call the decorated fn now, two print statements position themselves before and after the call of say_whee()
- the way a decorator modifies a function can change dynamically:
- ![[Pasted image 20240516093738.png]]
- ![[Pasted image 20240517093112.png]]
![[Pasted image 20240517093149.png]]
- the decoration gets hidden away below the definition of the function
- python allows to use decorators in a simpler way with the @ symbol, sometimes called the ***pie syntax***
![[Pasted image 20240517093653.png]]
- same output as above
- since decorators are functions, we can import these functions from different modules and use them to decorate other functions
	- the do it twice fn is not generic below, we can make it generic with the usage of args and kwargs
![[Pasted image 20240517095154.png]]
![[Pasted image 20240517094415.png]]
![[Pasted image 20240517094439.png]]
- this following works perfectly fine and prints "hello world"
	- empty args and kwargs can be unpacked, and when they are unpacked no arguments are passed
	- so this unpacking can be used even to the functions that do not take any arguments
	- helps in generalizing fn call statements
![[Pasted image 20240517094739.png]]

- if any return value is expected from the original function, we have to make sure the return statement is present in the wrapper as well
	- the inner function is called the wrapper and the outer function is called the decorator(at least as per the article)
![[Pasted image 20240517095419.png]]

## finding yourself
- in python, a power capability of objects is ***introspection***
	- objects knows about it's own attrs at runtime
	- ex: it's own name and documentation
![[Pasted image 20240520093635.png]]

![[Pasted image 20240520093723.png]]
- in the above screenshot, string repr of say_whee is object spec and it has absolute path to the function: do_twice\[decorator\].\<locals\>\[scope\].wrapper_do_twice\[wrapper function name\]
- as we can see, the function sub is decorated by func_do_it_twice, so the `name` attr of the function rightly points to the wrapper
 ![[Pasted image 20240520094713.png]]
- adding the following decorator: @functools.wraps(reference_to_wrapped_func), will preserve info about the original function
- only change is to add the new decorator to the wrapping function
![[Pasted image 20240520094812.png]]
- running the code after the above change, gives the following output
![[Pasted image 20240520094902.png]]

- functools.wraps decorator uses functools.update_wrapper() to update special attrs like name and doc that are used in introspection
### Real world uses
- following is a code that times execution of wrapped function
![[Pasted image 20240520095849.png]]

- following is a function that prints that arguments to a function and it's return value, this will help in debugging
	- this can be helpful when we decorate functions that we indirectly call somewhere in the code
![[Pasted image 20240520100142.png]]

### Registering Plugins
- decorators do not have to wrap the func they are decorating
	- in the example below, register(decorator) does not have a wrapper around the functions it is decorating
	- register returns the function as is after storing a reference in the PLUGINS dict
![[Pasted image 20240521094941.png]]
![[Pasted image 20240521094959.png]]
- the PLUGINS dict contains a reference to sub and add as soon as the script is run
	- as py applies decorator when we define a function
	- add and sub are immediately registered
- decorators can also be used in cases where authentication is required before running a function
![[Pasted image 20240521095729.png]]

## Decorating classes
- two different ways that we can use decorators on classes
	- decorate the methods
		- ex: [staticmethod, classmethod](https://realpython.com/instance-class-and-static-methods-demystified/), property
		- the static method and classmethod decorators are used to define a method inside a class namespace that are not connected to a particular instance of that class
		- [property](https://realpython.com/python-property/) decorator is used to customize getters and setters for class attrs
	- decorate the whole class
		- ex: dataclass decorator
![[Pasted image 20240521100723.png]]

- common use of class decorators is to be a simpler alternative to some use cases of [meta classes](https://realpython.com/python-metaclasses/), [[metaclasses]]
- class decorator take class as an argument
	- all the decorators we saw above will work as class decorators
- decorating a class does not decorate it's methods
	- if I apply timer decorator to the class, it will only return the time taken to instantiate the class