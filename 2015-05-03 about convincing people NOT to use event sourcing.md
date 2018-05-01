  
  

![](http://cdn.akamai.steamstatic.com/steamcommunity/public/images/avatars/2a/2a84d2f02e52beeb884fe1a472ee596d03db20f3_full.jpg)

I work at a company where they have embraced the idea of using CQRS with event sourcing (ES). It's seen as such a great pattern that they decided to use it for everything. It has become their favorite solution to every problem and [axon framework](http://www.axonframework.org/) has become the golden hammer in their toolbox to whack every nail.  
  
That's weird because for years, we've been trying to convince management what wonderful things we could do with CQRS+ES, if they only would let us... Now that they're finally convinced, it feels strange to do the opposite. To tell them what a terrible idea this is when used as the solution for the wrong problem.  
  
When using CQRS+ES, your events become the contract. Spatial and temporal. Your events define how other bounded contexts communicate with you and it's how your own _future_ bounded context will communicate with you. So it's super important to get those boundaries right and to get those events right. Sure, you can upcast an old obsolete event to one or more newer events and [axon has support for that](http://www.axonframework.org/docs/2.4/repositories-and-event-stores.html#event-upcasting), but that can become a real pain when you need to do too many of those.  
  
Other things that are super easy to do using a non ES solution, like correcting bad data, becomes hard using ES. In a non ES solution, customer support just executes some SQL update statements in your production database and the customer is happy. In an ES solution customer support needs to publish a _corrective event_ in order to let the view do the correct projection.  
  
So ES can become quite complex. But when I tell people ES is complex, they reply: "_No it's not complex. With axon framework it's easy. You just need to define the commands, aggregates, events and projections. Axon takes care of everything else."_ All the infrastructure, all the nitty gritty details of publishing, storing, replaying events is handled by this wonderful framework. However, they're only talking about development complexity and fail to see the operational complexity. It might be easy to program your CRUD-like application but once it's in production, things might become a mess.  
  
I'm not saying CQRS+ES doesn't solve some very hard problems in a very elegant way.  

*   For instance when facing a domain where the time factor is of great importance (what was the state of an aggregate 5 weeks ago).
*   Or where you will need to do a lot of projections in the future that you don't know now. 
*   Or where you have a high collaborative domain where your read model cannot keep up with the concurrent writes and you need to separate them in order to scale.

Traceability is not in that list! You don't NEED event sourcing to do traceability. You can do the same, simply by using a tracelog. Just write who did what in a separate table and you're done.   
Furthermore, when your aggregates are only created and never change afterwards, ES is probably not a good idea. Just store the state of the aggregate in the database and project your views based on that.  
Finally, when all you have is some CR(U)(D), don't bother doing event sourcing. Your events will only be called CreateXXXEvent, UpdateXXXEvent or DeleteXXXEvent. and won't contain any relevant business information.  
  
Greetings  
Jan