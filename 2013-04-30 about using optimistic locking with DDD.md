One of the main objectives when designing aggregate roots is to reduce concurrent modification from the same aggregate root by 2 users.

  
  
Take the example of [Vaughn Vernon's aggregate root design essay](http://dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_3.pdf):  

[![](http://4.bp.blogspot.com/-zUyOZ_Oauvc/UXizxVF1PPI/AAAAAAAACt8/DYmeyr4TMrc/s320/Capture.PNG)](http://4.bp.blogspot.com/-zUyOZ_Oauvc/UXizxVF1PPI/AAAAAAAACt8/DYmeyr4TMrc/s1600/Capture.PNG)

Taken from [http://dddcommunity.org/wp-content/uploads/files/pdf\_articles/Vernon\_2011_3.pdf](http://dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_3.pdf) by [@VaughnVernon ](https://twitter.com/VaughnVernon)

  

In there he writes about the design of an agile project management application where a BackLogItem contains a collection of Tasks each containing a collection of EstimatedLogEntries. When the hoursRemaining on the last open task becomes 0, the status field on the BackLogItem should become DONE. A relational database is used for persistence and hiberante as the ORM.

  

  

Lets see what happens when a Alberto would change a baklogitem in complete isolation. The backLogItem contains a number of tasks but only taskX and taskY have remaining hours left.

  

When Alberto sets the hoursRemaining of task X to 0 the following might happen:

[![](http://3.bp.blogspot.com/-47JAthhbQG0/UX-c7w_Xb3I/AAAAAAAACuU/w7jwt2W86Rw/s640/singleUser.png)](http://3.bp.blogspot.com/-47JAthhbQG0/UX-c7w_Xb3I/AAAAAAAACuU/w7jwt2W86Rw/s1600/singleUser.png)

  

1.  Alberto sends a SetHoursRemainingOfTaskXToZero command to the application service
2.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskX, 0)
3.  the backLogItem delegates the call to taskX . TaskX sets the remaining hours to 0. Finally the backLogItem checks if there are other tasks that have hoursRemaining. There still is some work left in taskY so the status of the backLogItem is kept as is.
4.  the application service method ends. Hibernate notices taskX's hoursRemaining is changed. The changes are flushed to the db and the transaction commits.

  

  

  

Now, what happens when Alberto sets the hoursRemaining of taskX to 0 and Yves does the same with taskY, both at the same time? The following might happen:

[![](http://1.bp.blogspot.com/-DvvCf-JWNS4/UX-kFJVTYdI/AAAAAAAACuk/iA6suXUhaBw/s640/two+users.png)](http://1.bp.blogspot.com/-DvvCf-JWNS4/UX-kFJVTYdI/AAAAAAAACuk/iA6suXUhaBw/s1600/two+users.png)

  

1.  Alberto sends a SetHoursRemainingOfTaskXToZero command to the application service
2.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskX, 0)
3.  Yves sends a SetHoursRemainingOfTaskYToZero command to the application service
4.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskY, 0)
5.  the backLogItem that Alberto is working on delegates the call to taskX . TaskX sets the remaining hours to 0. Finally the backLogItem checks if there are other tasks that have hoursRemaining. There still is some work left in taskY so the status of the backLogItem is kept as is.
6.  the backLogItem that Yves is working on delegates the call to taskY. TaskY sets the remaining hours to 0. Finally the backLogItem  checks if there are other tasks that have hoursRemaining. There still is some work left in taskX so the status of the backLogItem is kept as is.
7.  the application service method on the thread that executes the command sent from Alberto ends. Hibernate notices taskX's hoursRemaining is changed. The changes are flushed to the db and the transaction commits.
8.  the application service method on the thread that executes the command sent from Yves ends. Hibernate notices taskY's hoursRemaining is changed. The changes are flushed to the db and the transaction commits.

  

When the backLogItem would be loaded again, it would not contain any hours left, but still the status would not be set to 'DONE'. 

  

To improve this we need locks. When Alberto is busy changing the backLogItem, we need to make sure no-one else can change it. We might be in a pessimistic mood, but most of the time we would favor [optimistic locking](http://en.wikipedia.org/wiki/Optimistic_concurrency_control) because that would increase throughput.

  

Hibernate [support optimistic locking](http://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch05.html#d0e2225) by adding an extra column on the table mapped as an number field annotated with @Version. The field should be used by Hibernate only and increases every time someone changes the entity. During flush time Hibernate checks the version with the one currently stored in the database and throws an exception when it notices someone else increased it in the meantime.

  

So we can add an extra optLock fields on BackLogItem, Task and on EstimatedLogEntries. This way Hibernate will protect us from concurrent changes. Lets check this by replaying our previous scenario:

  

[![](http://1.bp.blogspot.com/-DvvCf-JWNS4/UX-kFJVTYdI/AAAAAAAACuk/iA6suXUhaBw/s640/two+users.png)](http://1.bp.blogspot.com/-DvvCf-JWNS4/UX-kFJVTYdI/AAAAAAAACuk/iA6suXUhaBw/s1600/two+users.png)

  

1.  Alberto sends a SetHoursRemainingOfTaskXToZero command to the application service
2.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskX, 0)
3.  Yves sends a SetHoursRemainingOfTaskYToZero command to the application service
4.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskY, 0)
5.  the backLogItem that Alberto is working on delegates the call to taskX . TaskX sets the remaining hours to 0. Finally the backLogItem checks if there are other tasks that have hoursRemaining. There still is some work left in taskY so the status of the backLogItem is kept as is.
6.  the backLogItem that Yves is working on delegates the call to taskY. TaskY sets the remaining hours to 0. Finally the backLogItem  checks if there are other tasks that have hoursRemaining. There still is some work left in taskX so the status of the backLogItem is kept as is.
7.  the application service method on the thread that executes the command sent from Alberto ends. Hibernate notices taskX's hoursRemaining is changed. Because taskX has become dirty, hibernate increases the optimistic lock field. It compares the optimistic lock field in the db and continues because that one did not change in the meantime. The changes are flushed to the db and the transaction commits. 
8.  the application service method on the thread that executes the command sent from Yves ends. Hibernate notices taskY's hoursRemaining is changed. Because taskY has become dirty, hibernate increases the optimistic lock field. It compares the optimistic lock field in the db and continues because that one did not change in the meantime. The changes are flushed to the db and the transaction commits. 

  

  

When the backLogItem would be loaded again, it would not contain any hours left, but still the status would not be set to 'DONE'. 

  

Why didn't hibernate protect us from concurrent modification?

  

This is because Alberto modified taskX while Yves modified taskY. Hibernate doesn't know that both belong to the same aggregate root. TaskY is totally unaware of modifications happening in taksX. It doesn't event know that taskY exists. 

  

The only one that does know is the backLogItem. By changing taskX, we not only want to protect taskX from concurrent modification, but also everything inside the aggregate root that holds taskX.

  

So how do we solve this? 

1.  When working with aggregate roots, we need to make sure we **place the optimistic lock field annotated with @Version field on the aggregate root**.
2.  Second we need to make sure we mark the aggregate root as dirty for every change happening inside the boundaries of the aggregate root. This can be a timestamp or you might just reuse the optLock field and increase it by one.  

  

Replaying our scenario with the BackLogItem marking the AR dirty would result in the following:

[![](http://4.bp.blogspot.com/-ezmesDdm3rs/UX-7W-116bI/AAAAAAAACu0/_P5nhE1SezI/s640/optLockingOnAR.png)](http://4.bp.blogspot.com/-ezmesDdm3rs/UX-7W-116bI/AAAAAAAACu0/_P5nhE1SezI/s1600/optLockingOnAR.png)

  

1.  Alberto sends a SetHoursRemainingOfTaskXToZero command to the application service
2.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskX, 0)
3.  Yves sends a SetHoursRemainingOfTaskYToZero command to the application service
4.  a transaction is started, the application service loads the backLogItem and calls backLogItem.hoursRemaining(taskY, 0)
5.  the backLogItem that Alberto is working on delegates the call to taskX . TaskX sets the remaining hours to 0 and notifies the caller that it's changed. This can be as simple as returning a boolean. The backLogItem now knows one of its tasks have changed. As a result it dirties itself. Finally the backLogItem checks if there are other tasks that have hoursRemaining. There still is some work left in taskY so the status of the backLogItem is kept as is.
6.  the copy of the backLogItem that Yves is working on delegates the call to taskY.  TaskY sets the remaining hours to 0 and notifies the caller that it's changed. The backLogItem dirties itself. Finally the backLogItem  checks if there are other tasks that have hoursRemaining. There still is some work left in taskX so the status of the backLogItem is kept as is.
7.  the application service method on the thread that executes the command sent from Alberto ends. Hibernate notices taskX's hoursRemaining is changed. It also notices that the backLogItem is changed and increases the optimistic lock field. It compares the optimistic lock field in the db and continues because that one did not change in the meantime. The changes are flushed to the db and the transaction commits.
8.  the application service method on the thread that executes the command sent from Yves ends. Hibernate notices taskY's hoursRemaining is changed. It also notices that the backLogItem is changed  and increases the optimistic lock field. It compares the optimistic lock field of the backLogItem to the one stored in the db and notices the latter changed. As a result it throws an exception and the transaction is rolled back.

  
  

  

By placing the optimistic lock field on the aggregate root and by explicitly marking the aggregate root as dirty when a change occurs in one of its containing entities, you can effectively guard your aggregate roots against concurrent modification. 

  

Greetings...

Jan