[![](http://cdn.thegadgetflow.com/wp-content/uploads/2013/05/Jigsaw-Cookie-Cutter.jpg)](http://cdn.thegadgetflow.com/wp-content/uploads/2013/05/Jigsaw-Cookie-Cutter.jpg)Our architecture - sort of - always looks the same:  

1.  A web request is handled by a spring controller. 
2.  The controller dispatches a command that is handled by a command handler. 
3.  The command handler retrieves some sort of aggregate by calling a repository, 
4.  calls an intention revealing method on the aggregate and 
5.  saves the result in the database.

  
Code is divided into command handlers, aggregates, value objects, repositories and domain services. Everything has its place. We put the command handlers in the application layer, aggregates in the domain layer and repositories in the infrastructure layer. It's a cookie cutter pattern.We can do this with our eyes closed.  
  
  

Feauture creep
--------------

We grow our software by adding more and more features. Before implementing each new functionality we ask ourselves if we can put the logic into an existing class. If we can find a service or repository that already does similar things we put it there. If not, we create a new class. For example, if we need to retrieve information about a customer from the database and we already have a CustomerRepository, we add the new method there.    

  
Things start out really simple in the beginning but after each sprint iteration, more functionality is added. A service that starts with a dependency on one repository, ends up having tons of them. An extra repository here, an additional validation service there. As time goes by, complexity increases because the number of dependencies increases too.  
  
Take a service that validates your holiday plans, for instance. It starts simple by receiving an unvalidated holiday and returning errors. It only has a dependency on a hotelRepository to look up the hotel where the customer will stay during his holiday:  
  

`Loading code....`

  
But that's only the beginning. Within a few sprint iterations, our poor HolidayValidator also has a dependency on a car booking agency, a flight booking agency, a restaurant reservation service, a repository for looking up previous holidays and a customer repository:  
  
  

`Loading code....`

  
  

Hidden complexity
-----------------

The result still looks quiet elegant though. It's nicely split into small methods that try to do only 1 thing. You don't really notice the increased complexity. The public method signatures stay the same. The complexity however, appears in the constructor that the IOC calls in order to setup the service.  It remains unnoticed to the developer because everything's @Autowired, @Resource-d or @Inject-ed.  
  
The place where you do notice that things get out of hand, is in your test code. It becomes more complex because now, you need to create all those stubs or mocks the service depends on.  
  

`Loading code....`

  
In our test we don't stub-out all methods of those dependencies. Only the ones that will be called from within our service. This means our test needs to know what methods our service will call. That's annoying and prone to false-negatives when the implementation changes.  
  
  

What went wrong? 
-----------------

The problem is not the growing number of dependencies. That's just a result of added functionality. It's not something you can avoid. The real problem is in the way we use dependencies:  
  
  

### 1. the method signature doesn't reveal its dependencies:

When calling a method, you have no idea what dependencies are involved. You need to look at the implementation to know what's going on. The method validating the hotel only really needs the HotelRepository, not the other 5 dependencies. Some other method of the HotelValidator needs the other dependencies but this one only needs a HotelRepository. Yet you can't tell looking at the method signature. It just says ...  

> Optional<Error> validateHotel(Holiday holiday).

It would be nice if we could see what dependencies it needs.  
  
  

### 2\. services depend on more than just the functionality they need

Usually, a class with a dependency to a repository, is really only dependent on 1 or 2 methods of that repository. It doesn't care about the zillion other services that repository offers.   
  
The validateHotel-method validates the hotel of a holiday object. To do that it needs to find the hotel based on a HotelId. The HotelRepository is able to fulfill this need and that's the reason why it's added as a dependency.   
  
HotelRepository has tons of other methods. But,the validateHotel-method doesn't care about saving, updating or deleting hotels. It doesn't want to lookup hotels by address or name, it only wants a way to find a hotel based on an id. That's just a function from ...  

HotelId -> Hotel. 

Let's try to fix things.
------------------------

Instead of injecting entire containers of functionality in the form of a repository, we'll only inject the functionality we need to fulfill our task.   
  
  

`Loading code....`

  
Now, our validator as no dependency on HotelRepository but instead it depends on Function<HotelId, Hotel>. Like any DI-style application it gets injected and it doesn't care where it comes from. That's the job of the IOC container.  
  
Now, the real dependencies of the validator are shown. They are as lean as they could be and only show what they really depend on instead of masking it with a dependency to an entire range of functionalities.  
  
  

Getting functions
-----------------

There's different ways to get a function from HotelId to Hotel. We can transform all HotelRepository's methods into static ones by adding its dependency - the dataSource - as an extra argument. This would turn ...  
  
  

Hotel findBy(HotelId id)

... into ...

static Hotel findBy(HotelId id, DataSource dataSource)

  

We could then use [partial application](https://en.wikipedia.org/wiki/Partial_application), to turn the method into a Function<HotelId, Hotel>.   
  
There's something else we could do, though. We can turn ...  
  
  

Hotel findBy(HotelId id)

... into ...

static Function<HotelId, Hotel> findBy(DataSource dataSource)

  

This has the advantage of being very explicit about dependencies: methods return functions and dependencies are passed as arguments. This way, HotelRepository is turned into a collection of static methods that take dependencies as arguments and return functions that do the correct behavior:  
  

`Loading code....`

  
When doing the same for our validator, it becomes something like this:  
  

`Loading code....`

  

There's nothing object oriented about stateless services
--------------------------------------------------------

Our repository and validator have stopped becoming stateless services. All that is left is a collection of functions. We can do the same for other services. In fact, we could do the same for all stateless services, repositories, validators, factories, ... Each would become loose groups of functions that are there, because we put them together. Because we think they belong together. Not due to some technical reason. We can put 2 functions together because they deal with the same thing like persisting a hotel. Because we can easily find them back later. Because they often need to be changed together. In fact, we have total freedom to choose where to put those function-returning-static-methods.   
  
But there's another advantage. Tests become easier. Because dependencies now become as lean as it can possibly get, it's much easier to stub or mock them. What used to be a complex test setup, now becomes as simple as it can get:  
  

`Loading code....`

  
Removing stateless services in favor of static functions, doesn't mean we should not have any services defined as objects at all. Sometimes it does make sense to wrap functionality inside a stateless service. For example when the service should be called in a strict order following some sort of life-cycle. Or when the analogy of a repository as being a store for your aggregates seems important to you. Nevertheless, in a lot of other cases, services can easily be transformed into static functions without losing any of the useful abstractions.  
  

Naked dependencies
------------------

  
Finally, when your dependencies only consist of functions, it allows you to use a different kind of reasoning when implementing a service. Because all of our stateless services have disappeared, we're free to come up with any dependency we need. Instead of thinking in terms of what services and objects are already available, you can now just think about what function you need to fulfill your task. Because dependencies have become completely naked, it's easier to reason about why a service has certain dependencies.  
  
  

References
----------

Some more stuff that I found interesting:

  
  

*   Greg Young's [8 lines of code talk](http://www.infoq.com/presentations/8-lines-code-refactoring)
*   Runar Oli's [talk from a few years ago at Lanyrd](http://lanyrd.com/2012/nescala/sqygc/)
*   Mike Hadlow's [blogpost on static methods](http://mikehadlow.blogspot.com/2015/08/c-program-entirely-with-static-methods.html)

  
  
  
  
Greetings  
Jan