- implicit parameters are similar to regular method parameters, except they could be passed to a method silently without going through the regular parameters list
- a method can define a list of implicit parameters, that is placed after the list of regular params
```
def draw(text: String)(implicit color: Color, by : DrawingDevice)
```
- when a method is defined with implicit parameters, scala will look up implicit values in the scope by matching the type
	- if they are not already passed in the implicit parameter list
```
def draw(text: String)(implicit color: Color, by: DrawingDevice) = {
	println(s"I am drawing the text: ${text} in color : ${color.value} with a ${by.value}")
}

case class Color(value: String)
case class DrawingDevice(value: String)
val c = new Color("Blue")
val d = new DrawingDevice("Ipad")

draw("Tindu")
```


https://www.baeldung.com/scala/implicit-parameters#:~:text=Implicit%20parameters%20are%20similar%20to,the%20list%20of%20regular%20parameters.