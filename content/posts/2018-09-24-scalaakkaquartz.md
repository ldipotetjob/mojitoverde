---
title: Scheduling in Akka with Quartz - a wise solution
date: 2018-09-24 09:00:00
tags:
    - Scala
    - Akka
categories:
    - scala programming 
keywords:
    - Scala
    - Akka
    - Quartz
---

The problem I bring: how to execute several tasks following a specific schedule (day/time) for the long term and different patterns? The previous schedule must be able to vary in time.

Our scenario:

- A process like Linux cron services for running commands following a predetermined schedule.
- Objects that process configuration files that contain a list of command lines. The config file, which you can configure dynamically, contains the schedule of command invocation.

Why do not use Linux cron + crontab and execute shell scripting files:

1. We have to deal with the scripting file configuration.
2. We need scalability.
3. We have to deal with the scripting file complexity (database connections, parsing thousands of URLs, parallel processing on the net, and so on).

Our previous scenario happens in an environment of concurrency. Some options available on the market:

1. [Apache Camel](https://camel.apache.org/components/next/timer-component.html)
2. [Quartz](https://www.quartz-scheduler.org/)
3. [Akka-Quartz Scheduler](https://github.com/enragedginger/akka-quartz-scheduler)

I will implement the last one, the Akka-Quartz Scheduler, because  Apache Camel IMO is too complicated for only his Timer, the same problem with Quartz,  oriented to the Java community.

Our roadmap:

1. Execute multiple tasks following a specific schedule(day/time) seamlessly until any failure.
2. **We can change the previous schedule(1.) in specific scenarios without stop any actors[^1], which are busy running other tasks**

As **Akka-Quartz Scheduler** explain, its goal is to provide Akka with a scheduling system closer to what one would expect for Cron type jobs.

Problem to solve:

We have a platform that collects information from a website. The website publishes information about films and indicates monthly when the platform will add new film pages info.

We base the foundation of our solution[^2] in AkkaScheduler. The cornerstone are the configuration files:

1. The first one indicates I have to do something monthly. It is an internal configuration in an internal file. In our case will be in the akka configuration file(application.conf[^3]): 


```shell
quartz {
   defaultTimezone = "Europe/London"
   schedules {
    moviepages {
     description = "A cron job that fires off every month"
     expression = "0 30 2 2 * ? *"
    }
   }
  }
```

See reference to [CronExpression](https://www.quartz-scheduler.org/api/2.1.7/org/quartz/CronExpression.html). Because the info is published every first day monthly, we have to check at the second one. So this is an internal configuration(**previous code**) which never change(I have to review every month for a schedule about the pages related to new films that will be added or updated)

2. The second one configuration file[^4](external) indicates when the website will update each information page, indicating when the platform will update or add new page throughout the month so the actor can collect the specific info about the film those days. The external configuration file is changed monthly with another new schedule.


```shell
schedule {
  defaultTimezone = "Europe/London"
  moviepagemejorenvo = [
    {
      cronexpresssion = "0 4 2 20 9 ? *"
      moviepage = "40"
    }
    {
      cronexpresssion = "0 4 2 20 9 ? *"
      moviepage = "41"
    }
  ]
}
```

The previous cronexpression means when the actor has to update the page belongings to the film website.  The software will group all movie pages with the same cronexpression and submit the jobs with each common group.

Our main points:

* For initialize the process [BootNotScheduled](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/BootNotScheduled.scala). In this case we know what pages we want to update so we do not use the schedule.
* For start the schedule process [BootScheduled](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/BootScheduled.scala). This is the main program that starts the scheduler. 


```scala
object BootScheduled extends App {
  //https://github.com/lambdista/config
  val system = ActorSystem(ConstantsObject.ScheduledIoTDaemon)
  // This actor will control the whole System ("IoTDaemon_Scheduled")
  val daemonScheduled = system.actorOf(IoTAdmin.props, ConstantsObject.ScheduledIoTDaemonProcessing)
  // This actor will control the reScheduling about time table for update new pages with films in original version
  val reeschedule = system.actorOf(ReSchedulingJob.props, ConstantsObject.DaemonReShedulingVoMoviePages)
  //Use system's dispatcher as ExecutionContext
  //QuartzSchedulerExtension is scoped to that ActorSystem and there will only ever be one instance of it per ActorSystem
  QuartzSchedulerExtension(system).schedule("moviepages", reeschedule, FireSchedule(daemonScheduled))
}
```

The previous code in line 10 will use the app internal config file[^5]. This line is responsible for every second day of the month at 2.30 am submit a job to read a new scheduler(ref. cronmoviepages.conf)  [^4]. The actor can read the external config file  periodically, which indicates what jobs have been re-scheduled every time with the new schedule.


```scala 
.....................

      cronExpreesionMatches.foreach(
        cronExpreesionMatch => {
          val (cronExpreesion, listOfMoviePages) = cronExpreesionMatch
          log.info(
            "Programming Schedule for cronexpression: {} including matches: {}",
            cronExpreesion,listOfMoviePages.mkString("-")
          )
          val headList = listOfMoviePages.head

          // TODO it is important to kill all Scheduled jobs that has been created ONCE time that the work has done

          /** This code generate a warning because the reschedule job never exist
            * any more because headList used for named it never is the same*/
          QuartzSchedulerExtension(system).rescheduleJob(
            s"Movie-Page-Scheduler$headList",
            scheduledDaemon,
            ParseUrlFilms(listOfMoviePages),
            Option("Scheduling "), cronExpreesion, None, defaultTimezone)
        }
      )
      log.info("schedule get new MoviePages")
  }

.....................
```

In the previous code in line 16 we reschedule an actor ([IoTAdmin](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/actors/IoTAdmin.scala)) to follow the external configuration  and launch as many actors([ProcessMoviePage](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/scala/com/ldg/actors/ProcessMoviePage.scala)) as many pages need to be updated. You can appreciate that in the code below.


```scala
........

class IoTAdmin extends Actor with ActorLogging with MessageToTargetActors{
  override def preStart(): Unit = log.info("Start process in Akka Scheduler Example")
  override def postStop(): Unit = log.info("Stop process in Akka Schedule Example")
  // TODO checking perfromance: http://docs.scala-lang.org/overviews/collections/performance-characteristics.html

  var watched = Map.empty[ActorRef, Int]

  override def receive = {

    case ParseUrlFilms(urlPageNunmList, optActorRef) =>

      urlPageNunmList.foreach(
              urlpagenumber => {
                val currentActorRef: ActorRef = context.actorOf(ProcessMoviePage.props)
                watched += currentActorRef -> urlpagenumber
                // watcher for every actor that is created 'cause the actor need to know when the process have finished
                context.watch(currentActorRef)
                // TODO urlspatterns<=varconf
                val processUrlFilmMessage = ProccessPage(
                  s"http://mejorenvo.com/p${urlpagenumber}.html",
                  optActorRef.fold(self)(ref=>ref)
                )
                sendParserUrlMessage(currentActorRef, processUrlFilmMessage)
              })
.............
```

Every specific day that was configured in cronmoviepages.conf[^4] file the **IoTAdmin** actor will receive a **ParseUrlFilms** message to process the new page that should be updated.

This post and the project can be a base or skeleton if you want to create a process with the following specifications:
* You need to execute scheduled tasks and these tasks occur at a specific moment in time and do not need to be executed periodically.
* You  need to seamlessly execute several tasks following a customizable schedule (day/time).


<br><br/>

<br><br/>

<br><br/>



[^1]: [Introduction to Actors](https://doc.akka.io/docs/akka/current/typed/actors.html)
[^2]: [Akka-Scheduler Module](https://github.com/ldipotetjob/akka-quickstart-scala/tree/master/modules/AkkaScheduler)
[^3]: [application.conf](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/modules/AkkaScheduler/src/main/resources/production.conf)
[^4]: [cronmoviepages.conf](https://raw.githubusercontent.com/ldipotetjob/akka-quickstart-scala/master/scriptsdb/cronmoviepages.conf)
[^5]: Internal configuration	
	
	```shell
		quartz {
			defaultTimezone = "Europe/London"
			schedules {
			   moviepages {
			   description = "A cron job that fires off every month"
			   expression = "0 30 2 2 * ? *"
			  }
			 }
	     })
    ```