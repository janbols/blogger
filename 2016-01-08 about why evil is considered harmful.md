In last year's [Build Stuff conference](https://buildstuff15lithuania.sched.org/event/4P0p/slidesuncle-bob-robert-martin-unclebobmartin-the-last-programming-language), Uncle Bob ranted that we, as software developers, don't adhere to any rules or standard. A few weeks later, he repeated that claim in a [blogpost](http://blog.cleancoder.com/uncle-bob/2015/11/27/OathDiscussion.html):  

> _And yet nothing binds us together as a profession. We share no ethics. We share no discipline. We share no standards. We are viewed, by our employers, as laborers. We are tools for others to command and use. We have no profession._

I like uncle Bob very much. He's the only one I know that has been warning and continues to warn us about the way we treat our own industry.  

Don'ts and don'ts
-----------------

I'm a java developer. Personally, I don't have the feeling that we share no standards. My world is full of them. Rules, principles, standards and things a developer cannot, should not do. They're not explicit, but implicit. Still, those rules exist. Here's a list of some of them:  
  

1.  [fat controllers are bad](http://codebetter.com/iancooper/2008/12/03/the-fat-controller/)
2.  [mutable data is evil](http://henrikeichenhardt.blogspot.be/2013/06/why-shared-mutable-state-is-root-of-all.html)
3.  [premature optimization is the root of all evil](http://dl.acm.org/citation.cfm?id=356640) 
4.  [goto considered harmful](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF) 
5.  [static is bad](https://dzone.com/articles/why-static-bad-and-how-avoid) 
6.  [duplicate code is evil](https://hethmonster.wordpress.com/2010/09/21/duplicate-code-is-evil/)
7.  [if statements are evil](http://www.ata.io/if-else-statements-are-evil)
8.  [null references are the billion dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)
9.  [getters and setters are evil](http://www.javaworld.com/article/2073723/core-java/why-getter-and-setter-methods-are-evil.html)
10.  [extends is evil](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)
11.  [dependency injection is evil](http://www.tonymarston.net/php-mysql/dependency-injection-is-evil.html)
12.  [frameworks are evil](http://tomasp.net/blog/2015/library-frameworks/)
13.  ...and the list goes on... 

All those rules are really just opinions and they are there for a very good reason. Those opinions are explained in detail and are stated in a certain context. The rule survives the passing of time, the reasons behind them, for most of us, don't.  

The heretic
-----------

Here's another one. It must have been about the first rule I learned when starting programming in java. A little while ago I was pairing with a colleague and created a class that was used as a data container, a [dto](https://en.wikipedia.org/wiki/Data_transfer_object). I created the class and made all fields public. It looked something like this:  

> public class Employee {  
>     public final EmployeeId id;  
>     public final String firstName;  
>     public final String lastName;  
>     public final Email email;  
>     public Employee(EmployeeId id, String firstName, String lastName, Email email){

>         ....  
>     }  
> }

[![](https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcQtI5RjnlT55hLtGSX9RJWZaSHiiDgNbMLjoQggD94zWJg8Lr_r)](https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcQtI5RjnlT55hLtGSX9RJWZaSHiiDgNbMLjoQggD94zWJg8Lr_r)

There you have it. I broke one of the oldest rules in java development: Never create public fields! Always keep your fields private and create public getters.   
  
My colleague looked at me with a strange look. He didn't have the energy to start a discussion with me and didn't say anything, but I could tell he was thinking I was doing a very, very bad thing.  
  
Nevertheless, there is no reason why I would add all those extra intellij-generated lines of useless getter code. The class is not part of any api nor will it ever be. There's no information hiding necessary. It's just a data structure that contains certain data in a strongly typed way. By leaving out the getters, the class remains concise, clean and maintainable.   
  
_But, public fields are evil! Evil, I say!_  
  
The point is... people forgot the reason why that rule exists. They only remember the rule. The fact that it's evil. Truth is (=my opinion), in a lot of our code, there's no reason to hide information. Especially when it's immutable (which is another one of my rules).  
  
Saying something is _evil_ or _considered harmful_ is a great way to create a meme, but it's detrimental in the long term when you forget the reason why.  
  
Greetings,  
Jan