- implicit parameters are similar to regular method parameters, except they could be passed to a method silently without going through the regular parameters list
- a method can define a list of implicit parameters, that is placed after the list of regular params
```
def draw(text: String)(implicit color: Color, by : DrawingDevice)
```
- when a method is defined with implicit parameters, scala will look up ***implicit values*** in the scope by matching the type
	- if they are not already passed in the implicit parameter list
	- the values I want to pass as implicits must be defined as implicit as below
		- either the variable can be val or var
		- one other important thing is that, we must explicitly mention the types of the implicits
		- when scala matches the implicit values, the parameter and variable names are ignored
			- scala will match implicit value by type
			- so it is not necessary the implicit parameter name must be the same as my implicit variable
```
def draw(text: String)(implicit color: Color, by: DrawingDevice) = {  
  println(s"I am drawing the text: ${text} in color : ${color.value} with a ${by.value}")  
}  
  
case class Color(value: String)  
  
case class DrawingDevice(value: String)  
  
implicit val c : Color = new Color("Blue")  
implicit val d: DrawingDevice = new DrawingDevice("Ipad")  
  
draw("Tindu")
```

- then comes the question of having two parameters expecting the same type
```
def writeTmp(text: String)(implicit color: Color, implicit color2: Color,implicit by: DrawingDevice) = {
	println(s"I am drawing the text: ${text} in colors: ${color.value} and ${color2.value} with a ${by.value}")
}

writeTmp("Tindu")

// assume that the same objects exist for color and drawingDevice
// OP is: I am drawing the text: Tindu in colors: Blue and Blue with a Ipad
```
- now what if there are multiple objects that have the same type which satisfies for an implicit parameter
```
implicit var c1: Color = new Color("greu") // one more implicit variable
// now both the function calls throw an error saying that there are two variables matching and that there is ambiguity
```
- with the function calls as they are we cannot proceed further
- now what we can do is to explicitly pass ***all the required parameters***
	- it is not possible to pass in only some parameters(which share the same type) and avoid passing in others
	- if we have chosen to pass then pass all unless there is a default value specified
```
writeTmp("Tind")(c, c1,d)  
draw("Tind")(c,d)
```
- implicits allow us to define the values anywhere in our code as long as they are within scope
- additionally they can be reused across other methods that look for implicit parameters
```
// without implicit parameter 
model.create(conn, newRecord) 
// with implicit parameter 
model.create(newRecord)
```
- in the second line, by making connection implicit the statement is more readable for a normal person

- disadvantages
	- readability can take a hit if used a lot



https://www.baeldung.com/scala/implicit-parameters#:~:text=Implicit%20parameters%20are%20similar%20to,the%20list%20of%20regular%20parameters.