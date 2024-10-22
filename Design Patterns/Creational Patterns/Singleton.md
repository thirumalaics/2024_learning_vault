## Intent
- ensures that a class has only one instance
- provides a global access point to this instance

## Problem
- solves two problems at the same time, violating the single responsibility principle
1. ensure that a class has just a single instance:
	- this is enforced usually in cases where there is a shared resource
	- trying to create a new object after creating an object will have to return the same old object
	- impossible to implement using a regular constructor

2. provide a global access point to this instance
	- just like global variable singleton patter lets us access some object from anywhere in the program but also protects the instance from being overwritten by other code
	- we don't want the code that solves problem #1 to be scattered all over our program
	- it is better to have it within one class, if the rest of our code already depends on it
- singleton is very popular that people may call something a singleton if one of the above is solved

## Solution
- all implementations of a singleton have these two steps in common:
	- a private constructor that bars code outside the class to create an object by directly calling the constructor
	- have a static method with a static field
		- the method calls the private constructor to create an instance
		- this static field will store reference to the object created, so any further calls to this method returns the object that is already created
- if our code has access to the singleton class, it will have access to the static method

## Real-world analogy
- government
	- country can have only one governing body


## structure
![[Pasted image 20241019164214.png]]
## Applicability
- when a single instance only should be available throughout the program
	- ex: database object shared by different parts of a program
- use singleton pattern when we need stricter control over global variables
	- unlike global variables, singleton pattern guarantees that there is just one instance of a class and nothing other than the singleton class itself, can replace the cached instance

## Pros and cons
- more control over how many instances created
- global access point to that instance
- cons
	- violates single responsibility principle as the pattern solves two problems at the same time
	- can mask bad design, for instance when the components of the program know too much about each other
	- 

https://refactoring.guru/design-patterns/singleton