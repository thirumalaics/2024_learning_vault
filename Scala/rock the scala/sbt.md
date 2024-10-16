![[20. Packaging and Imports#^360fee]]
![[20. Packaging and Imports#^af6991]]

- does sbt have it's own language to describe a project?
	- it's bit of proprietary language and also scala
	- we can do most scala operations in an sbt file
		- import and use external dependencies, write custom code
		- we cannot define classes
	- sbt language is a domain specific language written in scala(:= , in ,%\%, % are all ***functions*** written in scala)
	- libraryDependencies is not a reserved keyword, but it is a way of configuring our project
		- when we write : `libraryDependencies := Seq(...)`, we are setting the list of deps to be downloaded
	- \\%\% and \% are functions we use to specify what modules should be downloaded and added to the class-path


https://stackoverflow.com/questions/49890806/explanation-of-sbt-build-file
https://stackoverflow.com/questions/11411242/can-someone-explain-the-right-way-to-use-sbt
https://www.scala-sbt.org/release/docs/sbt-by-example.html
https://www.scala-sbt.org/release/docs/Running.html
https://www.scala-sbt.org/release/docs/Basic-Def.html