---
layout: post
title: "Visualizing Project Dependencies in sbt"
date: 2014-03-27 12:50:58 +0900
comments: true
categories: sbt
---

For pretty printing dependencies, use [sbt-dependency-graph plugin](https://github.com/jrudolph/sbt-dependency-graph). To use this plugin, add sbt plugin settings:
``` scala build/plugins.sbt
addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.7.4")
```

Next update the settings as follows (append graphSettings to every poject):
``` scala project/Build.scala
import sbt._
import Keys._

object MyBuild extends Build {

    private val commonSettings = net.virtualvoid.sbt.graph.Plugin.graphSettings

    lazy val root = project
       .in(file("."))
       .settings(commonSettings:_*)
       .aggregate(core, util)

    lazy val core = project
      .settings(commonSettings:_*)
      .settings(
        libraryDependencies += "org.fluentd" % "fluent-logger" % "0.2.10"
      )

    lazy val util = project
      .settings(commonSettings:_*)

}
```

Then you can see the detailed dependency graph with `dependenty-tree` command:
``` sh
> project core
[info] Set current project to core (in build file:/Users/leo/work/tmp/mproj/)
> dependency-tree
[info] core:core_2.10:0.1-SNAPSHOT [S]
[info]   +-org.fluentd:fluent-logger:0.2.10
[info]     +-org.msgpack:msgpack:0.6.7
[info]       +-com.googlecode.json-simple:json-simple:1.1.1
[info]       | +-junit:junit:4.10
[info]       |   +-org.hamcrest:hamcrest-core:1.1
[info]       |
[info]       +-org.javassist:javassist:3.16.1-GA
[info]
```

