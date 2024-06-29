- object are entities in python that can contain data and functions that can be used to manipulate the data
- everything in python is an object ***because py has no primitive or unboxed types, no machine values***
	- primitive type: predefined by the lang and is named by a reserved keyword
- this has nothing to do with metaclasses
- objects are python's abstraction for data
- all data in python is represented by objects or by relations between objects
- every object has a type, an identity and a value
- an object's identity never changes once it is created
	- we may think it as the object's address in memory
- the ***is*** operator compares the identity of two objects
- ***id*** function returns an integer representing its identity
- memory in python is managed dynamically
	- values exists until there are no references to them
- assignment in py, never copies data
	- always reference
- an object's type determines the operations that the object supports(ex: does it have a length)
	- also defines the possible values for objects of that type
	- the type fn returns an object's type (which is an object itself)
- immutability is not strictly the same as having an unchangeable value, it is more subtle
	- a tuple containing list is immutable but the values are not unchangeable
		- the list's value can be changed as it is immutable, this in turn changes the values in the tuple
		- immutable means that no other collection than a particular list can take it's place after creation
```
nums = [1,2,3]
other = nums
```
- in the above, there is only one list and two names referring to it
	- hence changes are visible through all names
- immutable values can't alias, especially when these immutable values are rebinded later
	- immutable types: ints,floats, strings, tuples
```
x = "hello"
y = x
x = x + " there"
```
- above y points to "hello" and x points to a different "hello there"
	- so for immutable types, changes are not visible through all names
	- these "changes" to immutable types creates a new value altogether
- "change" is unclear
	- changing an int: rebinding
		- `x = x + 1`
		- initially x is bound to an int, the above operation rebinds x to a new value created by addition
	- changing a list: mutating
		- nums.append
	- we can also rebind lists:
		- `nums = nums + [7]`
		- this creates a new list object
		- does extend method create a new object?
- mutable and immutable are assigned the same
	- assignment is the same for all values
	- aliasing can make it seem different
```
x+=y
#is
x = x+y #conceptually
x = x.\_\_iadd\_\_(y) # actually
```
- this method is implemented by list as well and within the method, extend is called and result of it is assigned to the var
- list elements are references themselves
![[Pasted image 20240528073205.png]]
- the boxes themselves are references to values
- lots of things are references
	- object attrs
	- list elements
	- dict values(and keys!)
	- anything on the left side of an assignment
- lots of things are assignments
	- ![[Pasted image 20240529070648.png]]
- names have scope but not type
- values have type but not scope

https://www.youtube.com/watch?v=_AEJHKGk9ns
https://nedbatchelder.com/text/names1.html
[3. Data model — Python 3.12.3 documentation](https://docs.python.org/3/reference/datamodel.html)
[What does "Everything" mean when someone says "Everything in Python is an object."? - Stack Overflow](https://stackoverflow.com/questions/32083871/what-does-everything-mean-when-someone-says-everything-in-python-is-an-object)
[oop - What is an Object in Python? - Stack Overflow](https://stackoverflow.com/questions/56310092/what-is-an-object-in-python)
[Is everything an object in Python like Ruby? - Stack Overflow](https://stackoverflow.com/questions/865911/is-everything-an-object-in-python-like-ruby)
https://www.reddit.com/r/learnpython/comments/bl9y62/primitive_vs_non_primitive_values_in_python/