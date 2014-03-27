---
layout: post
title: "Buidling multi-module projects in sbt"
date: 2014-03-26 17:20:30 +0900
comments: true
categories: 
---

In [the previous post]({% post_url 2014-03-24-sbt %}), I introduced how to use `sbt` for building Java projects. In response to this article my colleagues in [Treasure Data](http://treasure-data.com/) asked me on how to build multi-module projects with sbt. So, today I will talk about this topic.

Configuring multi-module projects with sbt is simple. In Maven, we need to write a parent pom.xml and 
child pom.xml files for all of the sub modules. In sbt you only need to prepare one project file (`build.sbt` or `projecct/Build.scala`).

Suppose we have the following folder structure with a parent module (root folder) and two sub modules _core_ and _util_:
```
project/Build.sbt
src
core/src
util/src

```

Here is a simple multi-module project setting example. Project _core_ depends on [fluent-logger](https://github.com/fluent/fluent-logger-java) library, while project _util_ has no library dependency. Projects are defined by using variables: _root_, _core_ and _util_. You can use these variable names to reference projects in the build file and sbt console. 

``` scala project/Build.scala
import sbt._
import Keys._

object MyBuild extends Build {
	
	lazy val root = project.in(file(".")).aggregates(core, util)

	lazy val core = project
		.settings(
			libraryDependencies += "org.fluentd" % "fluent-logger" % "0.2.10"
		)

	lazy val util = project
}
```

For Scala newbies: `object` means a definition of a singleton class. `lazy val` is a variable defintion that will be evaluated when it is used for the first time.


You can specify module directories with `project.in(file("..."))` syntax. The default directory becomes the same with the variable name. A project with the root directory `file(".")` will be the root project, which will be selected by default when lauching sbt. 


In the above example, the root project aggregates _util_ and _core_ projects. Enter the sbt console, and try `test` command. All test codes in three projects, including _root_, _util_ and _core_, will be tested.

``` sh
> test
[info] Passed: Total 0, Failed 0, Errors 0, Passed 0
[info] Passed: Total 0, Failed 0, Errors 0, Passed 0
[info] Passed: Total 0, Failed 0, Errors 0, Passed 0
[info] No tests to run for root/test:test
[info] No tests to run for util/test:test
[info] No tests to run for core/test:test
[success] Total time: 0 s, completed 2014/03/27 11:52:12
```

To run the test cases in a specific module, select a project name in sbt:
``` sh
> project core
[info] Set current project to core (in build file:/Users/leo/work/tmp/mproj/)
> test
[info] Passed: Total 0, Failed 0, Errors 0, Passed 0
[info] No tests to run for core/test:test
[success] Total time: 0 s, completed 2014/03/27 11:53:50
```

You can see the project settings with `show` commands:
``` sh
> show version
[info] 0.1-SNAPSHOT
> show dependencyClasspath
[info] List(Attributed(/Users/leo/.sbt/boot/scala-2.10.3/lib/scala-library.jar), Attributed(/Users/leo/.ivy2/cache/org.fluentd/fluent-logger/jars/fluent-logger-0.2.10.jar), Attributed(/Users/leo/.ivy2/cache/org.msgpack/msgpack/bundles/msgpack-0.6.7.jar), Attributed(/Users/leo/.ivy2/cache/com.googlecode.json-simple/json-simple/bundles/json-simple-1.1.1.jar), Attributed(/Users/leo/.ivy2/cache/junit/junit/jars/junit-4.10.jar), Attributed(/Users/leo/.ivy2/cache/org.hamcrest/hamcrest-core/jars/hamcrest-core-1.1.jar), Attributed(/Users/leo/.ivy2/cache/org.javassist/javassist/jars/javassist-3.16.1-GA.jar))
```
If you are not sure names of settings keys, use `<TAB>` completion.


## Building jar files

Now, let's go back to the root project:

``` sh
> project root
[info] Set current project to root (in build file:/Users/leo/work/tmp/mproj/)
> show version
[info] core/*:version
[info] 	0.1-SNAPSHOT
[info] util/*:version
[info] 	0.1-SNAPSHOT
[info] root/*:version
[info] 	0.1-SNAPSHOT
```

`package` command creates jar files of projects:
``` sh
> package
[info] Updating {file:/Users/leo/work/tmp/mproj/}root...
[info] Updating {file:/Users/leo/work/tmp/mproj/}core...
[info] Updating {file:/Users/leo/work/tmp/mproj/}util...
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] Resolving org.fluentd#fluent-logger;0.2.10 ...
[info] Packaging /Users/leo/work/tmp/mproj/target/scala-2.10/root_2.10-0.1-SNAPSHOT.jar ...
[info] Done packaging.
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] Resolving org.scala-lang#scala-library;2.10.3 ...
[info] Packaging /Users/leo/work/tmp/mproj/core/target/scala-2.10/core_2.10-0.1-SNAPSHOT.jar ...
[info] Done packaging.
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] Packaging /Users/leo/work/tmp/mproj/util/target/scala-2.10/util_2.10-0.1-SNAPSHOT.jar ...
[info] Done packaging.
```


## Publishing to Maven Repository

`publishLocal` command creates .jar, -source.jar and -javadoc.jar of your projects, then install them to your local ivy repository `$HOME/.ivy2/local`. While `publishM2` commands install them to your local Maven repository `$HOME/.m2/repository`. 

To deploy jars to a remote repositroy, use `publish` command. This is equivalent to `mvn deploy` command in Maven.

See also:

- [Publishing - sbt official doc](http://www.scala-sbt.org/release/docs/Detailed-Topics/Publishing)
- [sbt-sontaype plugin](https://github.com/xerial/sbt-sonatype) A sbt plugin I developed for publishing projects to the Maven central repository. 


## Project dependencies

`aggregate` settings is just for convenience of running commands in multiple projects. 
If some project actually depends on another project's code, use `dependsOn`:

``` scala
lazy val core = project.settings(...).dependsOn(util)
```
Now _core_ project can use classes in _util_ project and its dependent libraries. Try `publishLocal`:

``` sh
> publishLocal
[info] 	published core_2.10 to /Users/leo/.ivy2/local/core/core_2.10/0.1-SNAPSHOT/poms/core_2.10.pom
```
This creates .pom xml files under the target folder of each module. Looking at the generated pom.xml file, you can confirm the dependency to _util_ package is properly set:

``` 
<?xml version='1.0' encoding='UTF-8'?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi\
="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>core</groupId>
    <artifactId>core_2.10</artifactId>
    <packaging>jar</packaging>
    <description>core</description>
    <version>0.1-SNAPSHOT</version>
    <name>core</name>
    <organization>
        <name>core</name>
    </organization>
    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>2.10.3</version>
        </dependency>
        <dependency>
            <groupId>util</groupId>
            <artifactId>util_2.10</artifactId>
            <version>0.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.fluentd</groupId>
            <artifactId>fluent-logger</artifactId>
            <version>0.2.10</version>
        </dependency>
    </dependencies>
</project>
```

If you do not want include Scala library in the dependency, set `autoScalaLibrary` to false in the project settings.


## Reference

- [Multi-project builds - sbt official doc](http://www.scala-sbt.org/release/docs/Getting-Started/Multi-Project.html)


