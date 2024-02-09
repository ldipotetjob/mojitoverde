---
title: Generating scala application distribution in akka & play for different environments
date: 2019-02-12 09:00:00
tags:
    - Scala
categories:
    - software distribution 
keywords:
    - scala
    - sbt-native-packager
    - sbt
---

![sbt-copy_img](/mojitoverde/images/sbt-copy.jpeg#floatleft)
From my perspective, Sbt is quite complex. I need to work with the [sbt documentation](https://www.scala-sbt.org/1.x/docs/index.html) at hand. Many [people think that SBT is confusing, complicated, and opaque](https://www.lihaoyi.com/post/SowhatswrongwithSBT.html). 
My goal today: **how to generate distribution packages for different environments ** just in one command line clean + compile + build distribution + specific environment.

### What do we need:
1. Pass parameters to our process indicating what environment you want to build. 
2. Create a plugin to read the parameters. 
3. Create a task that will let you add the project to the appropriate configuration files before the compilation process. 
4. Create a command to package a distribution with the suitable option.

To achieve the previous tasks, we will use [sbt native packager](https://www.scala-sbt.org/sbt-native-packager/) to generate distribution for different environments. 

How to approach the point 1 in [What do we need](#what-do-we-need):

The first thing that we need to do is to pass a variable in our building process:

**sbt -Dour_system_property_var=value_to_assign**

I need to indicate the environment that we are building for:
I must read this environment, set the proper configuration file, and add it to our jar distributions.

You ought to code what to do, considering the input parameters. You can take some reference about [SBT parameters and Build Environment](https://sbt-native-packager.readthedocs.io/en/latest/recipes/package_configuration.html#sbt-parameters-and-build-environment).

Our next steps :
1. Read variables to indicate what environment for the new package distribution.
2. we need to indicate what config files we want to use in our compilation process, considering the input parameters. 

We need to create an sbt plugin([BuildEnvPlugin.scala](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/BuildEnvPlugin.scala)) to carry out the previous process:

### buildenvplugin code 1.0
```scala
import sbt.Keys._
import sbt._
import sbt.plugins.JvmPlugin

/**
 * make calls in this way to generate development distro
 * sbt -Denv=dev reload clean compile stage
 *
 * make calls in this way to generate production distro
 * sbt -Denv=prod reload clean compile stage
 *
 */

/** sets the build environment */
object BuildEnvPlugin extends AutoPlugin {

 // make sure it triggers automatically
 override def trigger = AllRequirements
 override def requires = JvmPlugin

 object autoImport {
  object BuildEnv extends Enumeration {
   val Production, Test, Development = Value
  }
  val buildEnv = settingKey[BuildEnv.Value]("the current build environment")
 }
 import autoImport._

 override def projectSettings: Seq[Setting[_]] = Seq(
  buildEnv := {
   sys.props.get("env")
    .orElse(sys.env.get("BUILD_ENV"))
    .flatMap {
     case "prod" => Some(BuildEnv.Production)
     case "test" => Some(BuildEnv.Test)
     case "dev" => Some(BuildEnv.Development)
     case unknown => None
    }
    .getOrElse(BuildEnv.Development)
  },
  // message indicating in what environment you are building on
  onLoadMessage := {
   val defaultMessage = onLoadMessage.value
   val env = buildEnv.value
   s"""|$defaultMessage
     |Running in build environment: $env""".stripMargin
  }
 )
}
```
Any other process must read the build environment that the plugin code above returns. We are going to solve that problem in the main build.sbt file.

Our main [build.sbt](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/build.sbt) file (**snapshot of code below**), depending on the environment selected, we are going to copy the proper configuration file to conf/application.conf configuration file **@see line 9 in code below, [code 1.1](#build-code-1.1)**, pay attention that **buildEnv.value** in line 4 is created/loaded in our customized sbt plugin [code 1.0](#buildenvplugin-code-1.0)

### build code 1.1
```scala
.............

mappings in Universal += {
 val confFile = buildEnv.value match {
  case BuildEnv.Development => "development.conf"
  case BuildEnv.Test => "test.conf"
  case BuildEnv.Production => "production.conf"
 }
 ((resourceDirectory in Compile).value / confFile) -> "conf/application.conf"
}

.............
```

First, we select the environment, copy all suitable config files, and then add this code to a task. We can pass parameters to our process 
at this point, indicating what environment we want to build. We have coded a plugin that can read parameters. [CommandScalaPlay.scala](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/CommandScalaPlay.scala) will let us execute in one line reload + clean + compile + **tgz distribution of our project.**

### commandscalaplay code 1.2
```scala
import sbt._

object CommandScalaPlay {

/** https://www.scala-sbt.org/sbt-native-packager/gettingstarted.html#native-tools */
 // A simple, multiple-argument command that prints "Hi" followed by the arguments.
 // Again, it leaves the current state unchanged.
 // launching from the console a production building: sbt -Denv=prod buildAll
 def buildAll = Command.args("buildAll", "<name>") { (state, args) =>
  println("buildAll command generating tgz in Universal" + args.mkString(" "))
  "reload"::"clean"::"compile"::"universal:packageZipTarball"::
  state
 }
}
```

In the above code, we have created a command that reloads + clean + compile + **tgz distribution of our project** in line 11. You can glance over [sbt-native-packager output format in universal](https://sbt-native-packager.readthedocs.io/en/latest/formats/universal.html#build) if you are interested in other output formats different than tgz.

We need to add the command created above in [code 1.3.](#comonsettings-code-1.3) in our [build.sbt](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/build.sbt) main file. So in code below **we add the command @see line 11 in code below, [code 1.3.](#comonsettings-code-1.3)**

### comonsettings code 1.3
```scala
.....

lazy val commonSettings = Seq(
  name:= """sport-stats""",
  organization := "com.mojitoverdeintw",
  version := "1.0",
  scalaVersion := "2.11.7"
 )
................

settings(commonSettings,
  resourceDirectory in Compile := baseDirectory.value / "conf",
  commands ++= Seq(buildAll))
................

```

In the code above, we say to the compiler that our resource directory will be /conf **@see line 12 in code above, [code 1.3.](#comonsettings-code-1.3)**. It should not be necessary because the resource directory( resourceDirectory in sbt) in Scala apps sbt resourceDirectory points to **src/main/resources**, but in play framework, the resource directory should point to **/conf**.

If you want to create a specific distribution:

go to your HOME_PROJECT and then:
1. a tgz for production env **sbt -Denv=prod buildAll**
2. a tgz for development env **sbt -Denv=dev buildAll**
3. a universal for development env **sbt -Denv=dev stage**
4. a universal for production env **sbt -Denv=dev stage**

If you want to implement the **above scenario** in your project, you need to take a look at the following points:
1. sbt version: in the above scenario is 0.13.11, reference file: [build.properties](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/build.properties) in my github repository.
2. [plugins.sbt](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/plugins.sbt)(in my github repository): pay attention to this line: **addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.3.2")** in my github repository.
3. [build.sbt](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/build.sbt)(in my github repository): Pay attention to this file, mainly about [code 1.1](#build-code-1.1) and [code 1.4][#buildenvplugin-code-1.4] below in this post.

The picture below shows us where are the files that we mention previously([CommandScalaPlay.scala](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/CommandScalaPlay.scala), [BuildEnvPlugin.scala](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/BuildEnvPlugin.scala), [plugins.sbt](https://raw.githubusercontent.com/ldipotetjob/restfulinplay/master/project/plugins.sbt))

![tree_pluginbuildenv_img](/mojitoverde/images/tree_pluginbuildenv.png#floatcenter)

**Remember that in our configuration folder(in playframework)** [/conf](https://github.com/ldipotetjob/restfulinplay/tree/master/conf) **should be all configuration files that we want in different environments during the distro generation process, so at the end, one of those files will be our application.conf configuration file.** 

In another scenario **instead of creating the application.conf file in our main build.sbt file, I reckon that should be done in code because build.sbt should be as clean as possible**. 

![tree-configfile_img](/mojitoverde/images/tree-configfile.png#floatcenter)

### buildenvplugin code 1.4
```scala
import sbt._
import sbt.Keys._
import sbt.plugins.JvmPlugin

/**
 * make calls in this way to generate development distro
 * sbt -Denv=dev reload clean compile stage
 *
 * make calls in this way to generate production distro
 * sbt -Denv=prod reload clean compile stage
 *
 */

/** sets the build environment */
object BuildEnvPlugin extends AutoPlugin {

 ............................

 lazy val configfilegeneratorTask = Def.task {
  val confFile = buildEnv.value match {
   case BuildEnv.Development => "development.conf"
   case BuildEnv.Test => "test.conf"
   case BuildEnv.Stage => "stage.conf"
   case BuildEnv.Production => "production.conf"
  }
  val filesrc = (resourceDirectory in Compile).value / confFile
  val file = (resourceManaged in Compile).value / "application.conf"
  IO.copyFile(filesrc,file)
  Seq(file)
 }
.....................
}
```

Pay special attention to the task definition **configfilegeneratorTask in line 19** above [code 1.4][#buildenvplugin-code-1.4] in [BuildEnvPlugin.scala](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/build.sbt)(in my github repository) and then modify the build.sbt file. See the code below and the changes done in our main [build.sbt](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/build.sbt)(in my github repository).

```scala
...................

lazy val root = (project in file(".")).
 aggregate(internetOfThings).
 aggregate(AkkaSchedulerPoc).
 settings(commonSettings,commands ++= Seq(buildAll))

lazy val AkkaSchedulerPoc = (project in file("modules/AkkaScheduler")).
 settings(commonSettings,libraryDependencies ++= Seq(
  "com.enragedginger" %% "akka-quartz-scheduler" % "1.6.1-akka-2.5.x"
 ),resourceGenerators in Compile += configfilegenerator)

...................

```

About the previous code **line 11**, we add our task **configfilegenerator** in our sbt compilation process. You can have a look at these files: [build.properties](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/project/build.properties), [plugins.sbt](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/project/plugins.sbt), [build.sbt](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/build.sbt)

**Remember that the plugin and task should be under the root project folder. (fig 1.1 above in our post)**

**Remember that in our configuration folder (in akka)** [/resource](https://github.com/ldipotetjob/akka-quickstart-scala/tree/master/modules/AkkaScheduler/src/main/resources) folder [/conf](https://github.com/ldipotetjob/restfulinplay/tree/master/conf) **should be all configuration files that we want in different environments during the distro creation process so at the end one of those files will be our application.conf configuration file.** 

![tree_akkaconf_img](/mojitoverde/images/tree_akkaconf.png#floatcenter)

When you are working with autoplugin, when you create a task or other object that executes any processes in a specific phase **you could face some problems with the initialization process** because some settings are loaded twice. So the [sbt initialization](https://www.scala-sbt.org/1.x/docs/Setting-Initialization.html) can be a fruitful reading.

There are different ways to generate distributions in several environments. Sbt has many other solutions, **but in my opinion, they are specific to each file and not for a complete configuration** like (log specification file + application configuration file + other properties or metadata file that can change when we change environment)

We show you examples in Playframework and Akka but how to solve if you want to generate any implementation or distribution without any plugins so you can choose the options that SBT gives you:

sbt **-Dconfig.file**=<Path-to-my-specific-config-file>/my-environment-conf-file.conf other-phases 

An example of the previous statement:

   **sbt -Dconfig.file=/home/myusr/development.conf reload clean compile stage** 

You can use **-Dconfig.resource** in the same way no need to specify the path to the file in this case. Sbt will try to find out in the resource directory.

You can do the same with **logging configuration** so in the same way:

sbt **-Dlogger.file** <Path-to-my-specific-logger-config-file>/my-environment-logger-conf-file.conf 

**or** like config.resource you can use **-Dlogger.resource**. You do not  need to specify the path to the log configuration file because the file is loaded from the classpath in this case

There are some interesting links in Sbt explaining how to deal with the distro generation process in different environments :

1. [Specifying different configurations files](https://www.playframework.com/documentation/2.6.x/ProductionConfiguration#Specifying-an-alternate-configuration-file). 
2. [Specifying different logging configurations files](https://www.playframework.com/documentation/2.6.x/ProductionConfiguration#Logging-configuration). 

Keep in mind: 
To pay attention to [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)(Human-Optimized Config Object Notation) and how it works. It can be helpful to proccess configuration files.

