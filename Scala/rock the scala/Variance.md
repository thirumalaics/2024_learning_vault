- covariant:
	- for any type X extending A(X<:A), type `MyList[X]` can be seen as `MyList[A]`
	- I can create a collection containing both types and preserve the `Box[Animal]` type
- lower bounded type in animal
- `class Cage[A >: Animal](animal: A)`
	- this conveys that Cage only accepts class that are super types of Animal
- bounded type solves a variance problem ![[13. Generics#^ce1c9c]]
- adding a dog to a list of cats will turn the list into something more generic, it will turn it into an Animal
```
class MyList[+A]{
def add(value: A): MyList[A] = ??? // the definition itself errors out
}
```