The following is a list of 10 **easy** things:  
  

1.  Caching results of a method using [JSR-107'](https://www.jcp.org/en/jsr/detail?id=107)s [@CacheResult](https://static.javadoc.io/javax.cache/cache-api/1.0.0/javax/cache/annotation/CacheResult.html) annotation.
2.  Automatic transaction handling with Spring's [@Transactional](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation.
3.  Using [JSR-303](https://jcp.org/en/jsr/detail?id=303)'s annotations for validating POJO's.
4.  Using [hibernate](http://hibernate.org/orm/) for inserting or loading data to/from the database
5.  Injecting fields in services using [JSR-330](https://www.jcp.org/en/jsr/detail?id=330)'s [@Inject](http://docs.oracle.com/javaee/6/api/javax/inject/Inject.html) or Spring's [@Autowired](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html) annotations.
6.  Automatically creating Spring beans using [component scanning](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html).
7.  Automatic permission checking using Spring Security's [@PreAuthorize](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/access/prepost/PreAuthorize.html).
8.  Implementing [service locators](https://en.wikipedia.org/wiki/Service_locator_pattern) using Spring's [ServiceLocatorFactoryBean](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/ServiceLocatorFactoryBean.html).
9.  Implementing repositories using [Spring data](http://projects.spring.io/spring-data/).
10.  Automatic project configuration using [Spring boot](https://projects.spring.io/spring-boot/).

  

The following is a list of 10 **simple** things:

1.  Keeping a cache using [Guava's cache](https://github.com/google/guava/wiki/CachesExplained). 
2.  Starting and committing transactions using Spring's [TransactionTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/support/TransactionTemplate.html).
3.  Validating POJO's by adding a validate method and implementing all the validation logic you need.
4.  Using Spring's [JdbcTemplate](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) to insert and select data to/from the database.
5.  Creating services using the new keyword and putting the dependencies in private final fields.
6.  Explicitly registering every Spring bean in [@Configuration](http://docs.spring.io/autorepo/docs/spring/3.1.x/javadoc-api/org/springframework/context/annotation/Configuration.html) config files.
7.  Permission checking by explicitly looking up the permission in Spring Security's [SecurityContext](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/context/SecurityContext.html), passed as an extra argument or by looking it up in the [SecurityContextHolder](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/context/SecurityContextHolder.html).
8.  Manually creating a service locator that keeps a registry of services.
9.  Implementing repositories by ... implementing them.
10.  Configuring your application by carefully choosing the dependencies your application needs and configuring them manually.

  

I like simple things. I dislike easy things. 

  

Easy things look simple but that's only in the beginning. It doesn't take long until they become complicated. 

  

Simple things require more effort - that's why they're not in the easy list - but they remain simple to read and understand. 

  

Aim for simple. Shy away from easy.

  

Greetings

Jan