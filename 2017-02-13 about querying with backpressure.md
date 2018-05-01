When querying the database you don't always know in advance how many data the query will return. This is a problem when you receive more data than can fit in heap memory.  

  
  

Paging
------

One way to work around this is to use paging: first select the first million rows, then select the next million, etc... until all rows are queried. Your first query will look something like the following:  
  
SELECT * FROM myTable OFFSET 0 ROWS FETCH NEXT 1000000 ROWS ONLY;

  
while the next one will look something like the following:  
  
SELECT * FROM myTable OFFSET 1000000 ROWS FETCH NEXT 2000000 ROWS ONLY;

  
This is slow for 2 reasons:  

1.  you need to make sure all subsequent calls use a consistent ordering. As a result you have to add an order by clause to your query where you order by your primary key. you could also use something like rowId in Oracle or something similar in other databases.
2.  paging basically runs the entire query up till the page you requested. The higher the page you request, the longer your query will take.

  
  

Observables
-----------

Another way to avoid blowing up your heap size is to return [rxJava's Observable](https://github.com/ReactiveX/RxJava/wiki/Observable) instead of a list with all your results. This way you're streaming the results from your query to the subscriber of your Observable but you never keep all results in memory. This has two advantages:

1.  you don't need a lot of memory to keep all the results in memory
2.  you don't have to wait until all results from the database are fetched. You can already start processing items the moment they are read from the database and passed to the subscriber.

I'm using spring-jdbc's [JdbcTemplate](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) or [NamedParameterJdbcTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/namedparam/NamedParameterJdbcTemplate.html) for querying the database. It has [all sorts of methods](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/jdbc.html) for querying the database. The following code shows you how to return an Observable:

  
  

`Loading code....`

  

It takes a sql query and a [RowMapper](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/RowMapper.html) for converting the result to some type <T> and returns an [Observable](http://reactivex.io/documentation/observable.html). The observable will trigger the call to the database for every subscription using a [ResultSetExtractor](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/ResultSetExtractor.html). The ResultSetExtractor will iterate over the [ResultSet](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html?is-external=true) and will pass the data for each row to the subscriber until there are no more rows.  

`Loading code....`

  
I can call the observable, subscribe to it and handle each row:

  
  

`Loading code....`

  
  

This works great... until you start doing things on another thread and your subscriber can't keep up with the items that are being produced:  

`Loading code....`

  
The result is a MissingBackpressureException that is thrown when the consumer can't keep up.   
The problem is we didn't take [backpressure](https://github.com/ReactiveX/RxJava/wiki/Backpressure) into account. This is needed the moment you're not doing everything in 1 thread ([callstack blocking](https://github.com/ReactiveX/RxJava/wiki/Backpressure#callstack-blocking-as-a-flow-control-alternative-to-backpressure)) and have a consumer that is slower than your producer (the results from  the db query).  

  
  

Backpressure
------------

The solution is to refactor the above code into something that does know how to handle backpressure. Luckily rxJava helps us a lot by abstracting away most of the difficulties using [SyncOnSubscribe](http://reactivex.io/RxJava/javadoc/rx/observables/SyncOnSubscribe.html). The code is a bit more complicated because we let rxJava control the call. In return it needs to know how to keep state during the subscription so we need to tell it ...

1.  how to create a new state object
2.  what to do when a new item is requested
3.  how to clean up state

To start, we need to wrap the Connection, the DataSource, the PreparedStatement and the ResultSet in a state object called JdbcQueryState:  
  

`Loading code....`

  
Now, when a new item is requested, we get the next row from the ResultSet and pass it to the observer or signal the observer that it's complete when there are no rows left:  
  

`Loading code....`

  
Finally we need to cleanup all state when it's done:  
  

`Loading code....`

  
All this is called from methods that take the sql and return a backpressure aware Observable:  

`Loading code....`

  
To hook into Spring's statement handling I needed to extends from NamedParameterJdbcTemplate. You can [see the full code here](https://gist.github.com/janbols/9af3655233e851cc341c605747243909).   
  

Greetings  
Jan