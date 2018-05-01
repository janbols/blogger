[RxJava](https://github.com/ReactiveX/RxJava) is a wonderful tool to do computation logic and stream processing. However, it sometimes behaves in unexpected ways. One of these will be discussed below:  
  
  

### first and single operator

According to the javadoc, [first](http://reactivex.io/RxJava/javadoc/rx/Observable.html#first()) turns an observable into one that only returns 1 element and then terminates.

[![](https://1.bp.blogspot.com/-_6TZbfAfmh4/WK1zUPIAvvI/AAAAAAABsKQ/VRZpfpx6-wAJIAK3rioENjNlM7S3RbVuQCLcB/s320/first.png)](https://1.bp.blogspot.com/-_6TZbfAfmh4/WK1zUPIAvvI/AAAAAAABsKQ/VRZpfpx6-wAJIAK3rioENjNlM7S3RbVuQCLcB/s1600/first.png)

  

There's a similar operator called [single](http://reactivex.io/RxJava/javadoc/rx/Observable.html#single()). The difference is that the latter expects the source observable to only emit 1 item:

[![](https://4.bp.blogspot.com/--zLFWdmyTQQ/WK1z86A8DlI/AAAAAAABsKU/s1BOShBZfMEAfN7r-gHMPbz2OAHd8KwzwCLcB/s320/single.png)](https://4.bp.blogspot.com/--zLFWdmyTQQ/WK1z86A8DlI/AAAAAAABsKU/s1BOShBZfMEAfN7r-gHMPbz2OAHd8KwzwCLcB/s1600/single.png)

So, use first when you don't care how many items the source observable will emit. Otherwise use single.  
  

### When abstractions leak

However, they behave different when you add [doOnCompleted](http://reactivex.io/RxJava/javadoc/rx/Observable.html#doOnCompleted(rx.functions.Action0)), [doOnTerminate](http://reactivex.io/RxJava/javadoc/rx/Observable.html#doOnTerminate(rx.functions.Action0)) and [doAfterTerminate](http://reactivex.io/RxJava/javadoc/rx/Observable.html#doAfterTerminate(rx.functions.Action0)) to the mix. These operators allow you to do some side effects after the last item has been emitted.  
  
The following code will print "onCompleted called":  

   Observable.just(1)
        .doOnCompleted(() -> System.out.println("onCompleted called") )
        .single()
        .subscribe()

  
The next code won't:  

   Observable.just(1)
        .doOnCompleted(() -> System.out.println("onCompleted called") )
        .first()
        .subscribe()

  
The next example will:  

   Observable.just(1)
        .first()
        .doOnCompleted(() -> System.out.println("onCompleted called") )
        .subscribe()

  
So, you will get different behavior depending on where you put doOnCompleted and if you use single or first.  
  
The reason why first behaves unexpectedly is because it unsubscribes from the source observable ones it receives the first element. As a consequence the source observable never has the opportunity to signal completion. As a result, an upstreamdoOnCompleted handler is never called while a downstreamdoOnCompleted handler is.  
  
By contrast, single will wait until it receives the completion signal from the source observable allowing upstream doOnCompleted handlers to be called.  
  
Greetings  
Jan