_tl;dr: stop scattering your codebase into maven modules divided by layers._  
[  
](http://adoptanewbeginning.org/assets/images/infant.jpg)[  
](http://adoptanewbeginning.org/assets/images/infant.jpg)[![](http://adoptanewbeginning.org/assets/images/infant.jpg)](http://adoptanewbeginning.org/assets/images/infant.jpg)  
  
  
  
I wish I could go back in time. Back to the time where I was still a junior in software development. The way things worked was to look at the existing code and copy it. When someone used an abstract factory I would try to do the same. When some of my colleagues used a visitor pattern to do double dispatch, I would use them as often as I could. I blindly accepted the patterns that were used, made them my own and tried to apply them everywhere.  
  
That's why I was called ... a junior developer.  
  
  
  
When I started working as a developer, aspects were the next big thing that would change everything. _Hibernate_ did some amazing things compared to _ejb2_. _Spring_ blew our minds with inversion of control._JSF_ was a better _Struts_ and _Seam_ was a better _JSF_. There were all these crazy new technologies that changed the way we would build software forever.  
[![](http://1.bp.blogspot.com/-uhCKOtw9OJc/T1xAyyAi2zI/AAAAAAAAAi0/Sa4gZyE-Wew/s320/inverted-bookshelf.jpg)](http://1.bp.blogspot.com/-uhCKOtw9OJc/T1xAyyAi2zI/AAAAAAAAAi0/Sa4gZyE-Wew/s1600/inverted-bookshelf.jpg)  
  
  

And then there was _Maven_. _Ant _was old school. _Maven _was hot. It gave us a way to have dependencies at runtime that were not available at compiletime allowing us to do some funky stuff with maven modules. 

  
  
  
It was a [DIP](http://en.wikipedia.org/wiki/Dependency_inversion_principle) dream come true. It felt good and it felt intelligent. We were able to protect ourselves (and those juniors) against violations made against the dependency inversion principle.   
  
  
Somehow, you could create maven modules containing services using other services that were implemented in another maven module. But those modules didn't depend on each other. Instead they depended on yet another module containing only interfaces.   
  
So we created a maven module per layer:  

  

*   one containing web controllers for the UI layer, 

*   one containing application services, 

*   one containing the domain (which we called 'model' because we didn't understand the difference), 

*   one containing the infrastructure

  
... and finally    

*   one to tie all modules together in a war
    

[![](http://www.comodotnet.com/Media/Default/Graphics/SingleResponsibilityPrinciple.jpg)](http://www.comodotnet.com/Media/Default/Graphics/SingleResponsibilityPrinciple.jpg)Each layer had its own responsibility. The controllers would only accept requests and turn them into calls to services in the application layer. The application service would call domain objects from the repositories. The interface would be in the domain layer while the implementation would reside in the infrastructure. The UI layer would not have access to the domain layer and the domain layer would not have access to any other layer. Nice. Clean. Separated.   
  
  
  

However. It didn't stay with those 4+1 modules. 

  

*   We had dto's that would transfer state from the domain layer all the way to the UI. So we created a module containing dto's.
*   We had converters that would convert the aggregate to/from the dto. These obviously needed to stay in their own maven module.
*   We had value objects that were accessible in the web layer. You can't put them in the domain layer because that's not accessible from the UI layer. The solution? ... add another maven module.
*   We had code that didn't belong anywhere but that was used everywhere. The infamous shared kernel maven module was born.
*   There was code that would handle exceptions. 
*   Code that would handle web requests. 
*   Code for handling queries.
*   Rule engines.
*   Email generators.
*   Logging code.
*   Hibernate user types.
*   Aspects. 
*   ...

  
  

Memory is failing me, but what I can remember from those days is we built a lot of maven modules just because we could. When we had a piece of code that didn't really belong anywhere, we would create a new maven module. Most modules only had 1 or 2 packages each with a small number of classes. Some didn't even contain code but only contained poms tying dependencies together to manage the dependency complexity.

  

The result was that every project had at least 60+ modules while the bigger projects would contain more than 140 modules.   
  

  

Now, there's 2 problems with this:  
  
1\. when writing a class, it becomes very difficult to know where it belongs to. Is a validation service part of the application layer or is it a domain service. In what package should we put an event listener. The result is we put it in the shared kernel module that grows and grows and contains all sorts of things.  
  
  
2\. sometimes we do want the domain layer to access a dto or a command directly. Sometimes an application service needs access to some infrastructure code. Sometimes we want to query something directly in the UI layer. Sometimes the strict dependency rules we set up don't allow us to do what we want. Instead of relaxing those rules, we put those classes in yet other maven modules that do have the needed dependencies to the dto's, infrastructure, ...  
  

Here's a law: Enforcing dependency rules by separating your application into maven modules always leads to more and more modules just to satisfy those strict rules. 

  
  

What's even more terrible is that in order to do something simple, like storing a key-value in a database, you need to go through all these layers. You need to start with a post arriving in the UI that translates the key-value into a dto that sends it to an application service that creates a some aggregate and tells a repository to save it. You can't just call some sql in the UI contollers because you just don't have a dependency on a dataSource in your web layer.

[![](http://www.sabisabi.com/images/DungBeetle-on-dung.JPG)](http://www.sabisabi.com/images/DungBeetle-on-dung.JPG)  
  

What you end up with, is an application that is totally scattered. Your application contains components or bounded contexts, but they're completely spread out over those 140+ modules. You cannot see what pieces belong together to form a component. What started as a noble intention to have a clean separation ends in a _big ball of modules_.

  
  

[![](http://raeganhuston.com/wp-content/uploads/2012/07/Zen-Dog1.jpg)](http://raeganhuston.com/wp-content/uploads/2012/07/Zen-Dog1.jpg)Over the last 8 years I grew older and hopefully a little wiser. I don't like to separate my code into different maven modules. Especially not by layers. I like to keep all the code together as much as I can.

  

When I do separate my code it's done by functionality, by component, by bounded context. Inside a maven module you would have code that belongs in the UI layer, the application layer, the domain layer and the infrastructure layer. I still have those layers, but it's indicated by putting them in different packages. 

  

Oooohh, but can't some junior developer do something stupid like directly use an implementation of a repository instead of the interface?  The answer is yes. Developers can do stupid things. But they can do stupid things in the 140+ module project too. Chances are they will do far more stupid things in a complicated project than in one that is separated by functionality.

  

Ok.

  

Here's the problem. 

  

[![](http://www.nextgenkids.com/images/kids.jpg)](http://www.nextgenkids.com/images/kids.jpg)

Those projects that I worked on when I was a junior, are still around. In the meantime a new generation of developers has arrived and they're working on that same codebase, taking the same patterns that were used back then as a reference of how things should be done, making the same mistakes and assuming having a lot of maven modules is the right thing to do when you're doing some serious programming.  
  

  

Here's another problem:

  

I don't seem to be able to sell the idea _-__maven modules per component instead of per layer - _to most of my colleagues. They seem so indoctrinated by separating software into maven modules by layers that they don't see the wreckage it does to the clarity of the program. The idea that we need to provide these strict rules in order to obey the dependency inversion principle is so strong that they don't understand that it's detrimental to the maintainability of the code. 

  

Truth be told, because of this doctrine, I never even worked an a software project that was not separated by layers. I was just never able to convince my colleagues to do otherwise. But I really hope to do so. Very soon. Because it's the best protection against the ever increasing entropy in our codebase.

  

Greetings

Jan