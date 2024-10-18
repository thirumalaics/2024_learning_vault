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
- running sbt without any arguments in a cmd starts a sbt shell
	- has a command prompt with tab completion and history
- we can compile in the sbt shell with a simple `compile` command, provided that we are at the project directory
- what constitutes a project directory?
	- if there is a project folder or a build.sbt in the current directory, then the current directory is called the project directory
	- ![[Pasted image 20241016190545.png]]
- since sbt is a sw tool, it has versions and the problem with having different versions is:
	- two people using two different versions of the sbt launcher to build the same projects might not get consistent results
	- to deal with this and get consistent results, we specify the version of sbt we are using in the project/build.properties directory
	- `sbt.version=1.9.8`
	- ![[Pasted image 20241016191134.png]]
	- how does the above solve the problem?
		- if the required version is not the one available locally, sbt downloads it for us
		- if this file is not present, the sbt launcher will choose and arbitrary version, which is discouraged because it makes our build non-portable
- what is a build definition?
	- a configuration that specifies how our project is built
	- it contains settings, tasks and dependencies that define how to compile, test and package our project
- what are build settings?
	- these are core configurations of our project, such as the name, version of project, Scala version and other parameters that define the build behavior
- what are dependencies?
	- external libraries that our project relies on
	- these can be java libs, scala libs or even other sbt projects
- what are tasks and commands?
	- sbt allows us to define custom tasks and commands in the build definition
	- ex: we can configure how tests are run or define additional tasks like code formatting, packaging
- in the following snipper, we define a subproject located in the current directory
```
lazy val root = (project in file("."))  
	.settings(    
		name := "Hello",
		scalaVersion := "2.12.7"
	)
```
- each subproject is configured by key-value pairs
	- one key is name and it maps to a string value
- the key value pairs are listed under the settings method
- how build.sbt defines settings
	- build.sbt defines subprojects which holds a sequence of key-value pairs called setting expressions using build.sbt domain-specific language
- build.sbt DSL
	- `organization := {"com.example"}` is called setting/task expression
	- organization is key
	- := is an operator
	- {"com.example"} is (setting/task) body
- what does a setting expression contain?
	- consists of three parts
		- key, operator or setting body
```
ThisBuild / organization := "com.example"
ThisBuild / scalaVersion := "2.12.18"
ThisBuild / version      := "0.1.0-SNAPSHOT"

lazy val root = (project in file("."))
  .settings(
    name := "hello"
  )
```
- name, version, scalaVersion are keys
	- a keys is an instance of SettingKey[T], TaskKey[T], InputKey[T] where T is the expected value type
	- name key is typed to SettingKey[String]
	- := operator on name is also typed to String
		- so changing "hello" to 43 will not compile
- build.sbt can also be interspersed with val, lazy val, def
	- top level objects and classes are not allowed in build.sbt
	- those should go in the project directory as scala source files
- what is SettingKey[T] type?
	- a key for value evaluated only once
	- the value is evaluated when loading the subproject, and kept around
- what is TaskKey[T] type?
	- key for a value, which is evaluated each time it is referred to(just like a scala function), potentially with side effects
	- the value is called a task
- what is a task?
	- action such as compile or package
	- they may return Unit or they may return a value related to a task
	- for example `package`  is TaskKey[File] and its value is the jar file it creates
	- whenever we start a task execution, sbt will re-run any tasks involved exactly once
		- task can contain other tasks
		- if multiple tasks depend on the same sub task, it is executed only one
	- for a task, sbt keeps around some executable code for a task such as compile and this code will rerun every time
	- whether to rerun each time is a property of the key and not the value
- what is InputKey[T] type?
	- for a task that has command line arguments as inputs
- what are built-in keys?
	- fields in an object called Keys
	- build.sbt implicitly has an `import sbt.Keys._`
	- so sbt.Keys.name can be referred to as name
- what are custom keys?
	- custom keys are keys of our own which can be defined with their respective creation ***methods***
		- `taskKey`, `inputKey`, `settingKey`
	- all the above three are methods
	- each method above expects the type of the value associated with the key and a description for the key itself:
		- `lazy val hello = taskKey[Unit]("An example task")`
		- definition of a key for a new task called hello
	- the name of the key is taken from the val the key is assigned to
	- here we have used the fact that the .sbt file can contain vals and defs in addition to settings
	- all such definitions are evaluated before settings regardless of where they are defined in the file
	- typically lazy vals are used instead of vals to avoid initialization order problems
https://stackoverflow.com/questions/49890806/explanation-of-sbt-build-file
https://stackoverflow.com/questions/11411242/can-someone-explain-the-right-way-to-use-sbt
https://www.scala-sbt.org/release/docs/sbt-by-example.html
https://www.scala-sbt.org/release/docs/Running.html
https://www.scala-sbt.org/release/docs/Basic-Def.html