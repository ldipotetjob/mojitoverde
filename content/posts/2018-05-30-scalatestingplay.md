---
title: Api Rest - Playframework - Testing 
date: 2018-05-30 09:00:00
tags:
    - Scala
    - PlayFramework
    - Testing
categories:
    - scala programming 
keywords:
    - Scala
    - PlayFramework
    - Testing
---

As was  argued by [Robert C. Martin in his Clean Code bible](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), there are several aspects that we have to pay attention when we design our tests:

- Minimize the number of assert per concept.
- Test just one concept per test function.
- Test should not depend on each other. 
- Test should run in any environment.

Test should pass or fail BUT you should not make process or any other thing for check if the Test was OK or failed.
Write the TEST first OR with the code, NOT after it be running in a production environment.

I am going to talk about Test Implementation in Scala and more specific in Scala  Playframework applications. If you come from Java programming language perhaps the more easy way is by [ScalaTest - FlatSpec](https://www.scalatest.org/at_a_glance/FlatSpec), it is a good starting point.

So when we are trying UNIT TEST in Play Framework, our minimal tests should include the followings artifacts:

* Controller
* Actions
* Routes
* Our services or functionalities, obviously

For make the previous topics more easy I have some important tips:

1. Encapsulate your operations in services.
2. Inject the classes that manage the above services.
3. Create the relationship Class - Trait. It will let you create binding in your tests with 
4. Guice in a more easy way and at the same time get the benefits already known of this programming practice.

The main ideas are:

* Mock services that you don't want to test. Your tests must be focused as much as you can in the functionalities that you want try.
* Test the Controller.
* Test the actions: When we talk about test actions that means test our action builders,  test the definitions that process every path in each request. A good idea is that action builders can process all related with our request as for example content-type. It will let us  filter for an expected Content-Type request and if it isn't what we expect stop the processing.
* Test the routes: It is important to know that all routes defined for us are working properly or at least can be reached by the proper path.
* Test any other Function or Service related with HTTP request. For example methods/functions related with content negotiation normally need make use of HTTP Header and in that case we will need set the header in a Fake-request.
* Test services functionalities. I  won’t include anything related with HTTP request here. Just our own functionalities.

In an "Ideal Hello World Play Framework application" EVERY THINGS is very simple BUT the problems arise in Real World application with escenarios like that:

* Multimodule applications.
* Several route files.
* Several bindings class that are injected in Singleton Class.
* Class referencing configurations files.

**!!! revisar esto porque parece no concuerda mucho !!!\
And for example you have bindings with Class that inject other Singleton class and that Class at the same time have some references to configuration files in your Project. 
!!Fin de la revision!!**


I will talk here about testing problems that are not clearly treated in Play documentation and how tackle it. When you are in real life with play framework applications is when you understand **how tedious is mock a binding in Guice when you trying to create Fake apps for Testing**.
It is more easy MockitoSugar in this context and could be better if when you mock any component  in Guice we could have the MockitoSugar concept instead of having to create a Well Mocked Object and binding it when you try to make an App via Guice Builder.

**A design made for an easy/smart test**:

I will not tell here why is so important dependency injection, you can find that information in every where even in [playframework documentationh](https://www.playframework.com/documentation/2.6.x/ScalaDependencyInjection)

In the following example I will show a controller[^1] that manage the actions:

```scala
class FootballLeagueController @Inject()(action: DefaultControllerComponents, services: TDataServices) extends BaseController with ContentNegotiation {

  /**
    * Best way, using a wrapper for an action that process a POST with a JSON 
    * in the request.
    *
    * @see utilities.DefaultControllerComponents.jsonActionBuilder a reference 
    *      to JsonActionBuilder a builder of actions that process POST request
    * @return specific Result when every things is Ok so we send the status 
    *         and the comment with the specific json that show the result
    */

    def insertMatchWithCustomized = action.jsonActionBuilder{ implicit request =>
    val matchGame: Match = request.body.asJson.get.as[Match]
      processContentNegotiationForJson[Match](matchGame)
  }

// ..............

  def insertMatchGeneric = JsonAction[Match](matchReads){ implicit request =>
    val matchGame: Match = request.body.asJson.get.as[Match]
    processContentNegotiationForJson[Match](matchGame)
  }

// ..........

  def getMatchGame = action.defaultActionBuilder { implicit request =>
    val dataResults:Seq[Match] = services.modelOfMatchFootball("football.txt")
    proccessContentNegotiation[Match](dataResults)
  }

..............

```

Some of my recommendations for **the real life** Controllers:
* I mostly need a **custom** action so inject a component(line 1 in FootballLeagueController implementaion example[^1]) in the previous snapshot of code [action: DefaultControllerComponents]) and this custom action will simplify the processing of your requests in Play and at the same time this design will make easy your test, something very important in our TDD process as we will see later.
* Encapsulate the operations that must be done in your actions in services and inject it in your controller(line 1 in FootballLeagueController implementaion example[^1]) in the previous snapshot of code [services: TDataServices]), some tips here: Inject a trait and bind that trait to a concrete class both things are truly important.  If any service include for example any customizable object like a database connection do NOT inject at this level do it in the concrete class. 

In our ControllerSpec[^2] we are going to Test first the Controller:

```scala
....

package com.ldg.play.test


import com.ldg.basecontrollers.{DefaultActionBuilder, DefaultControllerComponents, JsonActionBuilder}
.......
import com.ldg.model.Match
import com.ldg.play.baseclass.UnitSpec
import controllers.FootballLeagueController
import org.mockito.Mockito._
import org.scalatest.mock.MockitoSugar
import play.api.test.FakeRequest
import play.api.test.Helpers._
.......
import services.TDataServices


class FootballControllerSpec extends UnitSpec with MockitoSugar {

  val mockDataServices = mock[TDataServices]
  val mockDefaultControllerComponents = mock[DefaultControllerComponents]
  val mockActionBuilder = new JsonActionBuilder()
  val mockDefaultActionBuilder = new DefaultActionBuilder()
........
  val TestDefaultControllerComponents: DefaultControllerComponents = DefaultControllerComponents(mockDefaultActionBuilder,mockActionBuilder,/*messagesApi,*/langs)
  val TestFootballleagueController = new FootballLeagueController(TestDefaultControllerComponents,mockDataServices)

  /**
    *
    * Testing right response for acceptance of application/header
    * Request: plain text
    * Response: a Json
    *
    */

  "Request /GET/ with Content-Type:text/plain and application/json" should
    "return a json file Response with a 200 Code" in {

      when(mockDataServices.modelOfMatchFootball("football.txt")) thenReturn Seq(matchGame)
      val request = FakeRequest(GET, "/football/matchs")
        .withHeaders(("Accept","application/json"),("Content-Type","text/plain"))
      val result = TestFootballleagueController.getMatchGame(request)
      val resultMatchGame: Match = (contentAsJson(result) \ "message" \ 0).as[Match]
      status(result) shouldBe OK
      resultMatchGame.homeTeam.name should equal(matchGame.homeTeam.name)
      resultMatchGame.homeTeam.goals should equal(matchGame.homeTeam.goals)

    }

  /**
    *
    * Testing right response for acceptance of application/header
    * Testing  template work fine: Result of a mocked template shoulBe equal to Res
    * Request: plain text
    * Response: a CSV file
    *
    */

  "Request /GET/ with Content-Type:text/plain and txt/csv" should "return a csv file Response with a 200 Code" in {
    when(mockDataServices.modelOfMatchFootball("football.txt")) thenReturn Seq(matchGame)
    val request = FakeRequest(GET, "/football/matchs").withHeaders(("Accept","text/csv"),("Content-Type","text/plain"))    val result = TestFootballleagueController.getMatchGame(request)
    val content = views.csv.football(Seq(matchGame))
    val templateContent: String = contentAsString(content)
    val resultContent: String = contentAsString(result)
    status(result) shouldBe OK
    templateContent should equal(resultContent)
  }

  "Request /GET with Content-Type:text/plain and WRONG Accept: image/jpeg" should
    "return a json file Response with a 406 Code" in {
    val result = TestFootballleagueController.getMatchGame      .apply(FakeRequest(GET, "/").withHeaders(("Accept","image/jpeg"),("Content-Type","text/plain")))
    status(result) shouldBe NOT_ACCEPTABLE
  }
}
```

So if I am planning to test my Controller and if at the same time it is being injected by several component, the first thing that I need to do is give value to those components and because our target here are NOT the components I will mock them! [ref. ControllerSpec[^2] **line 22 - 25**]:


```scala
  val mockDataServices = mock[TDataServices]
  val mockDefaultControllerComponents = mock[DefaultControllerComponents]
  val mockActionBuilder = new JsonActionBuilder()
  val mockDefaultActionBuilder = new DefaultActionBuilder()
 ``` 

**TDataServices** is a trait so we need binding it to a Concrete class,check the below code example[^3], **that is a very good practice**. As the play specification indicates: by default Play will load any class called Module that is defined in the root package (the "app" directory) or you can define them in any place and indicate where to find it in the play configuration file(**application.conf**) under **play.modules** configuration value.

```scala
..............
class Module extends AbstractModule {
   ...................
   bind(classOf[TDataServices]).to(classOf[DataServices])
      .asEagerSingleton()
  }
}
```

You can find more information in [Play Documentation about custom and eager bindings](https://www.playframework.com/documentation/2.6.x/ScalaDependencyInjection). You should pay attention about this, it is important for several practices and in our case we will see the benefits of it in Testing.

I recommend Test with Guice **only when we do not have any other choice**. That is why if I need to inject a data base connection, for example, I wouldn't do it in the Controller it is better do it in the service class indeed.

Our first Test Testing Controllers begin in [ref. ControllerSpec[^2] **line 38 - 48**] :
I need to simulate a behaviour so if any process call **services.modelOfMatchFootball**[ref.FootballLeagueController[^1] line 27] I will return Seq(matchGame):
**when(mockDataServices.modelOfMatchFootball("football.txt")) thenReturn Seq(matchGame)** in my FootballControllerSpec.
In the same way I will mock the controller because I want to inject all mocked component too and test the flow of the controller. Remember that we are in [**ref**. ControllerSpec[^2] **line 38 - 48**] so I have a Fake request , something important is that we can add to this request a header, body(json, text, xml), cookies and whatever that any request has. Test will execute Controller but with mocked components and will process everything and return a response in the same way:

```scala
.........................      
when(mockDataServices.modelOfMatchFootball("football.txt")) thenReturn Seq(matchGame)
      val request = FakeRequest(GET, "/football/matchs")
        .withHeaders(("Accept","application/json"),("Content-Type","text/plain"))
      val result = TestFootballleagueController.getMatchGame(request)
      val resultMatchGame: Match = (contentAsJson(result) \ "message" \ 0).as[Match]
      status(result) shouldBe OK
      resultMatchGame.homeTeam.name should equal(matchGame.homeTeam.name)
      resultMatchGame.homeTeam.goals should equal(matchGame.homeTeam.goals)
..........................

```

If **TestFootballleagueController.getMatchGame** method had any parameter then the statement would be **Test**Football**leagueController.getMatchGame(anyparam)(request)**[ref.  line 5 previous code]. In our example I have to deal with content-negotiation so I have added a header easily to my FakeRequest and then we  expect the right processing from my content-negotiation method and the right response.
It is important to know that **FakeRequest**(GET, "/football/matchs") is the same that FakeRequest(GET, "/"), it has nothing to do with the routes defined in the conf/routes file.

In our Controller [ref. FootballLeagueController[^1]] we are going to Test now the **Actions** in the code below [ref. FootballActionsSpec[^4]]:

```scala
package com.ldg.play.test

import akka.stream.Materializer
import com.ldg.basecontrollers.{BaseController, DefaultActionBuilder, JsonActionBuilder}
import com.ldg.implicitconversions.ImplicitConversions.matchReads
import com.ldg.model.Match
import com.ldg.play.baseclass.UnitSpec
import play.api.test.FakeRequest
import play.api.libs.json._
import play.api.test.Helpers._
import play.api.http.Status.OK
import play.api.inject.guice.GuiceApplicationBuilder
import play.api.mvc.Results.Status
import scala.io.Source

class FootballActionsSpec extends UnitSpec {

  val jsonActionBuilder = new JsonActionBuilder()
  val defaultActionBuilder = new DefaultActionBuilder()
  val jsonGenericAction = new BaseController().JsonAction[Match](matchReads)

  val rightMatchJson = Source.fromURL(getClass.getResource("/rightmatch.json")).getLines.mkString
  val wrongMatchJson = Source.fromURL(getClass.getResource("/wrongmatch.json")).getLines.mkString

  implicit lazy val app: play.api.Application = new GuiceApplicationBuilder().configure().build()
  implicit lazy val materializer: Materializer = app.materializer

  /**
    * Test JsonActionBuilder:
    *
    * validate: content-type
    *           jsonBody must be specific Model
    *
    * @see com.ldg.basecontrollers.JsonActionBuilder
    *
    * Request: application/json
    *
    */

  "JsonActionBuilder with Content-Type:application/json and a right Json body" should "return a 200 Code" in {
    val request = FakeRequest(POST, "/")
      .withJsonBody(Json.parse(rightMatchJson))
      .withHeaders(("Content-Type", "application/json"))

    def action = jsonActionBuilder{ implicit request =>
      new Status(OK)
    }
    val result = call(action, request)
    status(result) shouldBe OK
  }

.......

  "JsonAction with Content-Type:application/json and a wrong Json body" should "return a 400 Code" in {
    val request = FakeRequest(POST, "/")
      .withJsonBody(Json.parse(wrongMatchJson))
      .withHeaders(("Content-Type", "application/json"))

    def action = jsonGenericAction{ implicit request =>
      new Status(OK)
    }
    val result = call(action, request)
    status(result) shouldBe BAD_REQUEST
  }

.......

  "DefaultActionBuilder with Content-Type:text/plain and a right Json body" should "return a 200 Code" in {
    val request = FakeRequest(GET, "/").withHeaders(("Accept","application/json"),("Content-Type", "text/plain"))

    def action = defaultActionBuilder{ implicit request =>
      new Status(OK)
    }
    val result = call(action, request)
    status(result) shouldBe OK
  }

.......

}
```

We are going to highlight this import [ref. FootballActionsSpec[^4] **line 10**]:
* import play.api.test.Helpers._

The previous import will let **call** an **action** with an **specific request(GET/POST....)**. An example of this call in [ref. FootballActionsSpec[^4] line 48]:
* call(action, request)
But the previous call need an implicit value, an instance of a **Materializer** so the best way for build it if we can not get an instance of our application is through Guice see [ref. FootballActionsSpec[^4] line 25 - 26].
I am testing the Actions so in my case I will test only Action/ActionBuilder and some aspects related with the request. I won't test here the service that is executed under an specific action. Take a look about [ref. FootballActionsSpec[^4] line 42] about our JsonBody in our FakeRequest.
It is not the objective of this post but you could take a look to [JsonActionBuilder](https://github.com/ldipotetjob/restfulinplay/blob/master/modules/apirest/app/com/ldg/basecontrollers/ControllerComponents.scala) and [DefaultActionBuilder](https://github.com/ldipotetjob/restfulinplay/blob/master/modules/apirest/app/com/ldg/basecontrollers/ControllerComponents.scala) [ref. FootballActionsSpec[^4] line 19-20]

In our Controller [ref. FootballLeagueController[^1]] we are going to Test now the Routes[ref. FootballRoutesSpec[^5]]:

```scala
package com.ldg.play.test


import apirest.Routes
import com.ldg.play.baseclass.UnitSpec
import services.{MockTDataServices, TDataServices}

import play.api.inject.bind
import play.api.test.FakeRequest
import play.api.test.Helpers._
import play.api.inject.guice.GuiceApplicationBuilder

class FootballRoutesSpec extends UnitSpec {

  val app = new GuiceApplicationBuilder()
    .overrides(bind[TDataServices].to[MockTDataServices])
    .configure("play.http.router" -> classOf[Routes].getName)
    .build()

 // implicit lazy val materializer: Materializer = app.materializer

   "Request /GET/football/PRML/matchs with Content-Type:text/plain and application/json" should    "return a json file Response with a 200 Code" in {
    val Some(result) = route(app, FakeRequest(GET, "/football/matchs")      .withHeaders(("Accept", "application/json"),("Content-Type", "text/plain")))
    status(result) shouldBe OK
  }
}
```
This aspect is not clear in PlayFramework documentations. First of all I need to create a Fake application with the same skeleton that the real so I need to do the following:
1. Create a fake app.
2. Mock all service that I am going to use like in my real app.
3. Create my route file.

The previous 3 points are solved in [ref. code_example 5 line 15 - line 18]:
 * [ref. code_example 5 line 15]: We create a mock using a binding. It was one of the important thing that explain at the beginning. Mocks implemented in Guice are completely different at Mockito. In Guice you will need to mock the whole object. So what we are doing here is overriding our binding with bindings to my mocked object. You can take a look of my mocked object MockTDataServices but in general you have to give fixed values in your mocked object to any definitions in your concrete class.
* [ref. FootballRoutesSpec[^5] line 16]: I am indicating here to my Fake app the route file that I want to use. The way to indicate to my fake app the route file is overriding the route file in our case  play.http.router -> the router class indicate the route file of our app(the routes that we want to test) 
I have to import the auto-generated/compiled scala class Routes. PlayFramework constructs a Route class for each configuration route file. In our Routes class generated there is an important method:

```scala
def routes: PartialFunction[RequestHeader, Handler]
``

So in examples like the next snapshot of code in [framework documentation](https://www.playframework.com/documentation/2.6.x/ScalaFunctionalTestingWithScalaTest#testing-the-router)

```scala
"respond to the index Action" in new App(applicationWithRouter) {
  val Some(result) = route(app, FakeRequest(GET_REQUEST, "/Bob"))

  status(result) mustEqual OK
  contentType(result) mustEqual Some("text/html")
  charset(result) mustEqual Some("utf-8")
  contentAsString(result) must include("Hello Bob")
}
```
They never explain how get **applicationWithRouter** in line 1 in the above code for instance. The solution:

```scala
val applicationWithRouter = GuiceApplicationBuilder().appRoutes { app =>
      val Action = app.injector.instanceOf[DefaultActionBuilder]
      ({
        case ("GET", "/Bob") => Action {
          Ok("Hello Bob") as "text/html; charset=utf-8"
        }
      })
    }.build()
```

Testing is easy and should be an style of programming, perhaps in lightbend the make it just a bit more difficult. Anyway, in my opinion, this game framework is fantastic, but not its documentation.


[^1]: source code implementation of FootballLeagueController
[^2]: source code implementation of ControllerSpec
[^3]: source code implementation of bindings in Modules  
[^4]: source code implementation of FootballActionsSpec
[^5]: source code implementation of FootballRoutesSpec


refs:\
[ScalaFunctionalTestSpec](https://github.com/playframework/playframework/blob/main/documentation/manual/working/scalaGuide/main/tests/code/specs2/ScalaFunctionalTestSpec.scala)

