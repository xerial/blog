---
layout: post
title: "Buidling multi-module projects in sbt"
date: 2014-03-26 17:20:30 +0900
comments: true
categories: 
---

In [the previous post]({% post_url 2014-03-24-sbt %}), I introduced how to use `sbt` for building Java projects. 
After that my colleagues asked me on how to build multi-module projects with sbt. So, today I will talk about this topic.

Configuring multi-module projects in sbt is simple. In Maven, we need to write a parent pom.xml and 
child pom.xml files for all sub modules. In sbt you only need to prepare one `project/Build.scala` file.

Suppose we have the following folder structure with a parent module (root folder) and two sub modules A and B:
```
project/Build.sbt
src
A/src
B/src

```

To build a multi-module projects, you need to use `project/*.scala` file. There is no way to do it in `.sbt` file. Here is a simple multi-module project example. Project A depends on [fluent-logger](https://github.com/fluent/fluent-logger-java) library, while B has no library dependency. The root project, named `"myproj"`, is the default project. 

``` scala project/Build.sbt
import sbt._
import Keys._

object MyProject extends sbt.Project {
	
	lazy val root = Project(
		id = "myproj",
		base = file(".")
    ) aggregates(A, B)

    lazy val A = Project(
    	id = "A",
    	base = file("A"),
		settings = Seq(
			libraryDependencies += "org.fluentd" % "fluent-logger" % "0.2.10"
    	)
    )

    lazy val B = Project(
    	id = "B",
    	base = file("B")
    )
}

```

For Scala newbie, let me explain some Scala syntaxes. The `object` means a definition of a singleton class. `lazy val` means a variable defintion that will be lazily evaluated when needed. `Project(id="...", ...)` is equivalent to call `new Project("...", ...)`. Scala has a notion of _case class_, in which we can omit `new` and can supply constructor parameters by using field names (by-name argument) like a hash in Ruby.


## 

