# Testing your application

Test source files must be placed in your application’s `test` folder. You can run them from the Play console using the ``test` (run all tests) and `test-only` (run one test class: `test-only my.namespace.MySpec`) tasks.

## Using specs2

The default way to test a Play application is by using [specs2](http://etorreborre.github.com/specs2/).

Unit specifications extend the `org.specs2.mutable.Specification` trait and are using the should/in format:

```scala
import org.specs2.mutable._

import play.api.test._
import play.api.test.Helpers._

class HelloWorldSpec extends Specification {

  "The 'Hello world' string" should {
    "contain 11 characters" in {
      "Hello world" must have size(11)
    }
    "start with 'Hello'" in {
      "Hello world" must startWith("Hello")
    }
    "end with 'world'" in {
      "Hello world" must endWith("world")
    }
  }
}
```

## Running multiple examples inside the same specification

In Unit specifications (see the first part of this page) you use ``should`` method to create groups of ``Example`` and the ``in`` method to create an ``Example`` , which contains a ``Result``. If you want to create a group of Examples where multiple examples needs a Play! application to be running, you cannot share the application and you have to provide a new one to each example like the following:

```scala
"Computer model" should {

  "be retrieved by id" in new WithApplication {
    // your test code
  }
  "be retrieved by email" in new WithApplication {
    // your test code
  }
}
```

In some cases, you want to run some operations with the application started before executing your example. Using Specs2
you can factor out your code by implementing your own ``org.specs2.specification.Around``, this can even extend one of
the built in arounds like in the following example:

```scala
abstract class WithDbData extends WithApplication {
  override def around[T](t: => T)(implicit evidence: (T) => Result) = super.around {
    prepareDbWithData() 
    t
  }
}

"Computer model" should {

  "be retrieved by id" in new WithDbData {
       // your test code
  }
  "be retrieved by email" in new WithDbData {
       // your test code
  }
}
```

## Unit Testing Controllers

Controllers are defined as objects in Play, and so can be trickier to unit test.  In Play this can be alleviated by [[dependency injection|ScalaDependencyInjection]]. Another way to finesse unit testing with a controller is to use a trait with an [explicitly typed self reference](http://www.naildrivin5.com/scalatour/wiki_pages/ExplcitlyTypedSelfReferences) to the controller:

```scala
trait ExampleController {
  this: Controller =>

  def index() = {
     ...
  }
}

object ExampleController extends Controller with ExampleController
```

and then test the trait:

```scala
object ExampleControllerSpec extends Specification {

  class TestController() extends Controller with ExampleController

  "Example Page#index" should {
    "should be valid" in {
          val controller = new TestController()          
          val result = controller.index()          
          result must not beNull
      }
    }
  }
}
```

This approach can be extended with your choice of dependency injection framework (Subcut/Spring/Guice/Cake Pattern) to set up and inject mocks into the trait.

> **Next:** [[Writing functional tests|ScalaFunctionalTest]]