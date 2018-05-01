I've been using [groovy](http://groovy-lang.org/) for a couple of years now. I love it. It saved me from several nervous breakdowns coding in the pre-jdk8 world.  
  
However, there's one thing that starts to annoy me.  
  
It's closures. At least when passed as a parameter in a method, mimicking [higher order functions](http://en.wikipedia.org/wiki/Higher-order_function).  
  
In the following example, there's a CustomerNotifier that sends an email to a customer. It accepts a CustomerId and a closure creating the email. The closure is called with the Customer and a string indicating the environment ( we don't want to send emails to the real customers in a dev environment, do we):  
  

`Loading code....`

  
In the method signature, there's no way to express what input parameters the closure needs. I can express the result, but not the input parameters.  
  
That's annoying because now, the caller of that method, must look into the implementation to know what arguments will be passed to the closure. To make things worse, your IDE will not be able to assist you because it too won't have a clue of the parameters it expects. It's like typing blindfolded.  
  
I could use [@ClosureParams](http://docs.groovy-lang.org/docs/next/html/gapi/groovy/transform/stc/ClosureParams.html). This annotation provides extra information about the parameters a closure needs, assisting the IDE. However it just makes your code really ugly:  
  

`Loading code....`

  
Lately I started using a library called [functional java](http://www.functionaljava.org/). It facilitates programming functionally in java and can be used by all poor souls that aren't allowed to program in scala but do want to use a more functional approach.  
  
Among the basic stuff it contains interfaces for functions with [1 up till 8 input parameters](https://functionaljava.ci.cloudbees.com/job/master/javadoc/fj/package-summary.html). With these I can clearly define what input parameters my function needs.  
  
  

`Loading code....`

  
Together with groovy's [implicit closure coercion](http://mrhaki.blogspot.de/2013/11/groovy-goodness-implicit-closure.html), I can use closures as typed functions while my IDE will resolve the input parameters for me and warn me when using the wrong type.  
  

`Loading code....`

  

I'll definitely be using the functionalJava library more as it works great with groovy. There's also a groovy library building on top of functionalJava at [https://github.com/mperry/functionalgroovy](https://github.com/mperry/functionalgroovy). It makes working with groovy even smoother but up till now I haven't really felt the need to use it.  
  
You can [find a gist of the complete example here](https://gist.github.com/janbols/7d415011b43d71eb2269).

  

Greetings

Jan