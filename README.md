# Best Performance Practices For Hibernate 5 & Spring Boot 2

<a href="https://leanpub.com/java-persistence-performance-illustrated-guide"><p align="center"><img src="https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/Java%20Persistence%20Performance%20Illustrated%20Guide.jpg" height="410" width="350"/></p></a>

**Hibernate & Spring Boot Samples**

1. **[How To Store UTC Timezone In MySQL](https://github.com/AnghelLeonard/Hibernate/tree/master/HibernateSpringBootUTCTimezone)**

**Description:** How to store date, time, and timestamps in UTC time zone in MySQL

**Key points:**\
     - `spring.jpa.properties.hibernate.jdbc.time_zone=UTC`\
     - `spring.datasource.url=jdbc:mysql://localhost:3306/db_screenshot?useLegacyDatetimeCode=false`
     
-----------------------------------------------------------------------------------------------------------------------    

2. **[How To View Binding Params Via Log4J 2](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootLog4j2ViewBindingParameters)**

**See also recipe 52 for using log4jdbc and 53 for logging using TRACE**

**Description:** View the prepared statement binding parameters via Log4J 2 logger setting

**Key points:**\
     - in pom.xml, exclude Spring Boot's Default Logging\
     - in pom.xml, Add Log4j 2 Dependency\
     - in log4j2.xml add, `<Logger name="org.hibernate.type.descriptor.sql" level="trace"/>`
   
**Output example:**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootLog4j2ViewBindingParameters/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

3. **[How To View Query Details Via "datasource-proxy"](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDataSourceProxy)**

**Description:** View the query details (query type, binding parameters, batch size, etc) via **[datasource-proxy](https://github.com/ttddyy/datasource-proxy)**

**Key points:**\
     - add in pom.xml the datasource-proxy dependency\
     - create an bean post processor to intercept the `DataSource` bean\
     - wrap the `DataSource` bean via `ProxyFactory` and an implementation of `MethodInterceptor`
   
**Output example:**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootDataSourceProxy/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

4. **[How To Batch Inserts Via `saveAll(Iterable<S> entities)` In MySQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootBatchInsertsJpaRepository)**

**Description:** Batch inserts via `SimpleJpaRepository#saveAll(Iterable<S> entities)` method in MySQL

**Key points:**\
     - in application.properties set `spring.jpa.properties.hibernate.jdbc.batch_size`\
     - in application.properties set `spring.jpa.properties.hibernate.generate_statistics` (just to check that batching is working)\
     - in application.properties set JDBC URL with `rewriteBatchedStatements=true` (optimization for MySQL)\
     - in entity, use the [assigned generator](https://vladmihalcea.com/how-to-combine-the-hibernate-assigned-generator-with-a-sequence-or-an-identity-column/) since MySQL `IDENTITY` will cause batching to be disabled\
     - in entity, add `@Version` property of type `Long` to avoid extra-`SELECT`s fired before batching (also prevent lost updates in multi-request transactions). Extra-`SELECT`s are the effect of using `merge()` instead of `persist()`. Behind the scene, `saveAll()` uses `save()`, which in case of non-new entities (have IDs) will call `merge()`, which instruct Hibernate to fire a `SELECT` statement to make sure that there is no record in the database having the same identifier.\
     - pay attention on the amount of inserts passed to `saveAll()` to not "overwhelm" the persistence context. Normally the `EntityManager` should be flushed and cleared from time to time, but during the `saveAll()` execution you simply cannot do that, so if in `saveAll()` there is a list with a high amount of data, all that data will hit the persistence context (1st level cache) and will be in-memory until flush time. Using relatively small amount of data should be ok. For large amount of data, please check the next example.
  
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootBatchInsertsJpaRepository/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

5. **[How To Batch Inserts Via EntityManager In MySQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootBatchInsertsEntityManager)**

**Description:** Batch inserts via `EntityManager` in MySQL. This way you can easily control the `flush()` and `clear()` of the persistence context (1st level cache). This is not possible via SpringBoot, `saveAll(Iterable<S> entities)`. Another advantage is that you can call `persist()` instead of `merge()` - this is used behind the scene by the SpringBoot `saveAll(Iterable<S> entities)` and `save(S entity)`.

**Key points:**\
     - in application.properties set `spring.jpa.properties.hibernate.jdbc.batch_size`\
     - in application.properties set `spring.jpa.properties.hibernate.generate_statistics` (just to check that batching is working)\
     - in application.properties set JDBC URL with `rewriteBatchedStatements=true` (optimization for MySQL)\
     - in entity, use the [assigned generator](https://vladmihalcea.com/how-to-combine-the-hibernate-assigned-generator-with-a-sequence-or-an-identity-column/) since MySQL `IDENTITY` will cause batching to be disabled\
     - in DAO, flush and clear the persistence context from time to time. This way you avoid to "overwhelm" the persistence context. 
   
**Output example:**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootBatchInsertsEntityManager/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

6. **[How To Batch Inserts Via JpaContext/EntityManager In MySQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootBatchInsertsEntityManagerViaJpaContext)**

**Description:** Batch inserts via `JpaContext/EntityManager` in MySQL.

**Key points:**\
     - in application.properties set `spring.jpa.properties.hibernate.jdbc.batch_size`\
     - in application.properties set `spring.jpa.properties.hibernate.generate_statistics` (just to check that batching is working)\
     - in application.properties set JDBC URL with `rewriteBatchedStatements=true` (optimization for MySQL)\
     - in entity, use the [assigned generator](https://vladmihalcea.com/how-to-combine-the-hibernate-assigned-generator-with-a-sequence-or-an-identity-column/) since MySQL `IDENTITY` will cause batching to be disabled\
     - the `EntityManager` is obtain per entity type via, `JpaContext#getEntityManagerByManagedType(Class<?> entity)`\
     - in DAO, flush and clear the persistence context from time to time. This way you avoid to "overwhelm" the persistence context. 
   
**Output example:**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootBatchInsertsEntityManagerViaJpaContext/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

7. **[How To Exploit Session-Level Batching (Hibernate 5.2 Or Higher) In MySQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootBatchInsertsViaSession)**

**Description:** Batch inserts via Hibernate session-level batching (Hibernate 5.2 or higher) in MySQL.

**Key points:**\
     - in application.properties set `spring.jpa.properties.hibernate.generate_statistics` (just to check that batching is working)\
     - in application.properties set JDBC URL with `rewriteBatchedStatements=true` (optimization for MySQL)\
     - in entity, use the [assigned generator](https://vladmihalcea.com/how-to-combine-the-hibernate-assigned-generator-with-a-sequence-or-an-identity-column/) since MySQL `IDENTITY` will cause batching to be disabled\
     - the Hibernate `Session` is obtained by un-wrapping it via `EntityManager#unwrap(Session.class)`\
     - the batching size is set via `Session#setJdbcBatchSize(Integer size)` and get via `Session#getJdbcBatchSize()`\
     - in DAO, flush and clear the persistence context from time to time. This way you avoid to "overwhelm" the persistence context. 
   
**Output example:**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootBatchInsertsViaSession/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

8. **[How To Use Direct Fetching Via Spring Data/EntityManager/Session](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDirectFetching)**

**Description:** Direct fetching via Spring Data, `EntityManager` and Hibernate `Session` examples.

**Key points:**\
     - direct fetching via Spring Data uses `findById()`\
     - direct fetching via `EntityManager` uses `find()`\
     - direct fetching via Hibernate `Session` uses `get()`

-----------------------------------------------------------------------------------------------------------------------    

9. **[How To Create DTOs Via Spring Data Projections](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)**

**Description:** Fetch only the needed data from the database via Spring Data Projections (DTOs)

**Key points:**\
     - write an interface (projection) containing getters only for the columns that should be fetched from the database\
     - write the proper query returning a `List<projection>`\
     - if is applicable, limit the number of returned rows (e.g., via `LIMIT`) - here, we can use query builder mechanism built into Spring Data repository infrastructure
     
**Output example (select first 2 rows; select only "name" and "city"):**
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootDtoViaProjections/sample.png)  

-----------------------------------------------------------------------------------------------------------------------    

10. **[How To Load Attributes Lazily](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootAttributeLazyFetching)**

**Description:** By default, the attributes of an entity are loaded eager (all at once). We can load them **lazy** as well. This is useful for column types that store large amounts of data: `CLOB`, `BLOB`, `VARBINARY`, etc.

**Key points:**\
     - in pom.xml, activate Hibernate bytecode instrumentation (e.g. use Maven bytecode enhancement plugin as follows)\
     - mark the columns that should be loaded lazy with `@Basic(fetch = FetchType.LAZY)`
     
**Run the following requests:**\
     - create a new user: `localhost:8080/new`\
     - fetch the user without avatar (this is a picture, therefore a large amount of data): `localhost:8080/user`\
     - fetch the user with avatar (loaded lazy): `localhost:8080/avatar`

-----------------------------------------------------------------------------------------------------------------------    

11. **[How To Populate A Child-Side Parent Association Via Proxy](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootPopulatingChildViaProxy)**

**Description:** A `Proxy` can be useful when a child entity can be persisted with a reference to its parent. In such cases, fetching the parent entity from the database (execute the `SELECT` statement) is a performance penalty and a pointless action. Hibernate can set the underlying foreign key value for an uninitialized `Proxy`.

**Key points:**\
     - rely on `EntityManager#getReference()`\
     - in Spring, use `JpaRepository#getOne()` -> used in this example\
     - in Hibernate, use `load()`\
     - here, we have two entities, `Tournament` and `TennisPlayer`, and a tournament can have multiple players (`@OneToMany`)\
     - we fetch the tournament via a `Proxy` (this will not trigger a `SELECT`), we create a new tennis player, we set the `Proxy` as the tournament for this player and we save the player (this will trigger an `INSERT` in the tennis players table, `tennis_player`)
     
**Output example:**\
     - the console output will reveal that only an `INSERT` is triggered, not the `SELECT`
     
-----------------------------------------------------------------------------------------------------------------------    

12. **[How To Reproduce N+1 Performance Issue](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootSimulateNPlus1)**

**Description:** N+1 is an issue of lazy fetching (but, eager is not exempt). This application reproduce the N+1 behavior.

**Key points:**\
     - define two entities, `Category` and `Product` having a `@OneToMany` relationship\
     - fetch all `Product` lazy, so without `Category` (results in 1 query)\
     - loop the fetched `Product` collection and for each entry fetch the corresponding `Category` (results N queries)
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootSimulateNPlus1/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

13. **[How To Optimize Distinct SELECTs Via HINT_PASS_DISTINCT_THROUGH Hint](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootHintPassDistinctThrough)**

**Description:** Starting with Hibernate 5.2.2, we can optimize `SELECT DISTINCT` via `HINT_PASS_DISTINCT_THROUGH` hint

**Key points:**\
     - use `@QueryHints(value = @QueryHint(name = HINT_PASS_DISTINCT_THROUGH, value = "false"))`
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootHintPassDistinctThrough/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

14. **[How To Enable Dirty Tracking](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootEnableDirtyTracking)**

**Description:** Prior to Hibernate version 5, the dirty checking mechanism relies on Java Reflection API. Starting with Hibernate version 5, the dirty checking mechanism relies on bytecode enhancement. This approach sustain a better performance, especially when you have a relatively large number of entitites.

**Key points:**\
     - add the corresponding `plugin` in pom.xml (use Maven bytecode enhancement plugin)
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootEnableDirtyTracking/sample.png)

The bytecode enhancement effect can be seen on `User.class` [here](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootEnableDirtyTracking/Bytecode%20Enhancement%20User.class/User.java)

-----------------------------------------------------------------------------------------------------------------------    

15. **[How To Use Java 8 Optional In Entities And Queries](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootOptional)**

**Description:** This application is a proof of concept of how is correct to use the Java 8 `Optional` in entities and queries. 

**Key points:**\
     - use the Spring Data built-in query-methods that return `Optional` (e.g., `findById()`)\
     - write your own queries that return `Optional`\
     - use `Optional` in entities getters\
     - in order to run different scenarios check the file, data-mysql.sql

-----------------------------------------------------------------------------------------------------------------------    

16. **[How To Use OneToMany Bidirectional Correctly](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootOneToManyBidirectional)**

**Description:** This application is a proof of concept of how is correct to implement the bidirectional `@OneToMany` association. 

**Key points:**\
     - always cascade from parent to child\
     - use `mappedBy` on the parent\
     - use `orphanRemoval` on parent in order to remove children without references\
     - use helper methods on parent to keep both sides of the association in sync\
     - use lazy fetch\
     - use a natural/business key or use entity identifier and override `equlas()` and `hashCode()` as [here](https://vladmihalcea.com/the-best-way-to-implement-equals-hashcode-and-tostring-with-jpa-and-hibernate/)         

-----------------------------------------------------------------------------------------------------------------------    

17. **[How To Use Query Fetching](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootQueryFetching)**

**Description:** This application is a proof of concept of how to write a query via `JpaRepository`, `EntityManager` and `Session`.

**Key points:**\
     - for `JpaRepository` use `@Query` or Spring Data Query Creation\
     - for `EntityManager` and `Session` use the `createQuery()` method 
     
-----------------------------------------------------------------------------------------------------------------------    

18. **[How To Avoid MySQL & Hibernate 5 AUTO Generator Type](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootAutoGeneratorType)**

**Description:** In MySQL & Hibernate 5, the `GenerationType.AUTO` generator type will result in using the `TABLE` generator. This adds a significant performance penalty. Turning this behavior to `IDENTITY` generator can be obtained by using `GenerationType.IDENTITY` or the native generator.

**Key points:**\
     - use `GenerationType.IDENTITY` instead of `GenerationType.AUTO`\
     - use the native generator - exemplified in this application
   
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootAutoGeneratorType/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

19. **[How To Avoid The Redundant save() Call](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootRedundantSave)**

**Description:** This application is an example when calling `save()` for a managed entity is redundant.

**Key points:**\
     - Hibernate triggers `UPDATE` statements for managed entities without the need to explicitly call the `save()` method\
     - behind the scene, this redundancy implies a performance penalty as well

-----------------------------------------------------------------------------------------------------------------------    

20. **[How To Handle PostgreSQL (BIG)SERIAL And Batching Inserts](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootBatchingAndSerial)**

**Description:** In PostgreSQL, using `GenerationType.IDENTITY` will disable insert batching. The `(BIG)SERIAL` is acting "almost" like MySQL, `AUTO_INCREMENT`. In this application, we use the `GenerationType.SEQUENCE` which enable insert batching, and we optimize it via the `hi/lo` optimization algorithm.

**Key points:**\
     - use `GenerationType.SEQUENCE` instead of `GenerationType.IDENTITY`\
     - rely on the `hi/lo` algorithm to fetch multiple identifiers in a single database roundtrip (you can go even further and use the Hibernate `pooled` and `pooled-lo` identifier generators (these are optimizations of `hi/lo`))
   
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootBatchingAndSerial/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

21. **[How To Write JPA Inheritance - Single Table](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootSingleTableInheritance)**

**Description:** This application is a sample of JPA Single Table inheritance strategy (`SINGLE_TABLE`)

**Key points:**\
     - this is the default inheritance strategy (`@Inheritance(strategy=InheritanceType.SINGLE_TABLE)`)\
     - all the classes in a hierarchy are mapped to a single table in a the database
   
**Output example (below is a single table obtained from 4 entities):**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootSingleTableInheritance/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

22. **[How To Count And Assert SQL Statements](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootCountSQLStatements)**

**Description:** This application is a sample of counting and asserting SQL statements triggered "behind the scene". Is very useful to count the SQL statements in order to ensure that your code is not generating more SQLs that you may think (e.g., N+1 can be easily detected by asserting the number of expected statements).

**Key points:**\
     - in pom.xml add dependencies for `datasource-proxy` and Vlad Mihalcea's `db-util`\
     - create the `ProxyDataSourceBuilder` with `countQuery()`\
     - reset the counter via `SQLStatementCountValidator.reset()`\
     - assert `INSERT`, `UPDATE`, `DELETE` and `SELECT` via `assertInsert/Update/Delete/Select/Count(long expectedNumberOfSql)`
   
**Output example (when the number of expected SQLs is not equal with the reality an exception is thrown):**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootCountSQLStatements/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

23. **[How To Use JPA Callbacks](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootJpaCallbacks)**

**Description:** This application is a sample of enabling the JPA callbacks (`Pre/PostPersist`, `Pre/PostUpdate`, `Pre/PostRemove` and `PostLoad`).

**Key points:**\
     - in entity, write callback methods and use the proper annotations\
     - callback methods annotated on the bean class must return void and take no arguments
   
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootJpaCallbacks/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

24. **[How To Write @OneToOne Via @MapsId](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootOneToOneMapsId)**

**Description:** Instead of a bidirectional `@OneToOne` better rely on an unidirectional `@OneToOne` and `@MapsId`. This application is a proof of concept. 

**Key points:**\
     - use `@MapsId` on child side\
     - basically, for `@OneToOne` associations, this will share the Primary Key with the parent table
     
-----------------------------------------------------------------------------------------------------------------------    

25. **[How To Extract DTOs Via SqlResultSetMapping](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoSqlResultSetMapping)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on `SqlResultSetMapping` and `EntityManager`.

**Key points:**\
     - use `SqlResultSetMapping` and `EntityManager`\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)
     
-----------------------------------------------------------------------------------------------------------------------    

26. **[How To Extract DTOs Via SqlResultSetMapping & NamedNativeQuery](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoSqlResultSetMappingAndNamedNativeQuery)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on `SqlResultSetMapping`, `NamedNativeQuery` and `EntityManager`.

**Key points:**\
     - use `SqlResultSetMapping`, `NamedNativeQuery` and `EntityManager`\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)
     
-----------------------------------------------------------------------------------------------------------------------    

27. **[How To Extract DTOs  Via javax.persistence.Tuple And Native SQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoTupleAndSql)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on `javax.persistence.Tuple` and native SQL.

**Key points:**\
     - use `java.persistence.Tuple` and `EntityManager.createNativeQuery()`\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)     

-----------------------------------------------------------------------------------------------------------------------    

28. **[How To Extract DTOs Via javax.persistence.Tuple And JPQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoTupleAndJpql)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on `javax.persistence.Tuple` and JPQL.

-----------------------------------------------------------------------------------------------------------------------    

29. **[How To Extract DTOs Via Constructor Expression And JPQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoConstructorExpression)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on Constructor Expression and JPQL.

**Key points:**\
     - use a proper constructor in the DTO class and use a query as `select new com.jpa.CarDto(c.name, c.color) from Car c`\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections) 
     
-----------------------------------------------------------------------------------------------------------------------    

30. **[How To Extract DTOs Via ResultTransformer And Native SQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoResultTransformer)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on Hibernate, `ResultTransformer` and native SQL.

**Key points:**\
     - use `AliasToBeanConstructorResultTransformer` for DTOs without setters, with constructor\
     - use `Transformers.aliasToBean()` for DTOs with setters\
     - use `EntityManager.createNativeQuery()` and `unwrap(org.hibernate.query.NativeQuery.class)`\
     - starting with Hibernate 5.2, `ResultTransformer` is deprecated, but until a replacement will be available (in Hibernate 6.0) it can be used ([read further](https://discourse.hibernate.org/t/hibernate-resulttransformer-is-deprecated-what-to-use-instead/232))\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections) 

-----------------------------------------------------------------------------------------------------------------------    

31. **[How To Extract DTOs Via ResultTransformer And JPQL](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoResultTransformerJpql)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on Hibernate, `ResultTransformer` and JPQL.

**Key points:**\
     - use `AliasToBeanConstructorResultTransformer` for DTOs without setters, with constructor\
     - use `Transformers.aliasToBean()` for DTOs with setters\
     - use `EntityManager.createQuery()` and `unwrap(org.hibernate.query.Query.class)`\
     - starting with Hibernate 5.2, `ResultTransformer` is deprecated, but until a replacement will be available (in Hibernate 6.0) it can be used ([read further](https://discourse.hibernate.org/t/hibernate-resulttransformer-is-deprecated-what-to-use-instead/232))\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)
     
-----------------------------------------------------------------------------------------------------------------------    

32. **[How To Extract DTOs Via Blaze-Persistence Entity Views](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoBlazeEntityView)**

**Description:** Fetching more data than needed is prone to performance penalities. Using DTOs allows us to extract only the needed data. In this application we rely on [Blaze-Persistence](https://persistence.blazebit.com/) entity views.

**Key points:**\
     - add in pom.xml dependencies specific to Blaze-Persistence\
     - configure Blaze-Persistence, `CriteriaBuilderFactory` and `EntityViewManager`\
     - write an entity view via an interface in Blaze-Persistence fashion\
     - write a Spring-centric repository by extending `EntityViewRepository`\
     - call method of this repository such as, `findAll()`, `findOne()`, etc\
     - for using Spring Data Projections check this [recipe](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaProjections)

-----------------------------------------------------------------------------------------------------------------------    

33. **[How @ElementCollection Without @OrderColumn Works](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootElementCollectionNoOrderColumn)**

**Description:** This application reveals the possible performance penalties of using `@ElementCollection`. In this case, without `@OrderColumn`. As you can see in the next recipe (34) adding `@OrderColumn` can mitigate some performance penalties.

**Key points:**\
     - an `@ElementCollection` doesn't have a Primary Key\
     - an `@ElementCollection` is mapped in a separate table\
     - avoid `@ElementCollection` when you have a lot of inserts and deletes in/from it since the database has to delete all existing rows in order to add a new one or delete one\
     - the more items we have in this table the greater the performance penalty
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootElementCollectionNoOrderColumn/sample.png)  

-----------------------------------------------------------------------------------------------------------------------    

34. **[How @ElementCollection With @OrderColumn Works](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootElementCollectionWithOrderColumn)**

**Description:** This application reveals the performance penalties of using `@ElementCollection`. In this case, with `@OrderColumn`. But, as you can see in this application, by adding `@OrderColumn` can mitigate some performance penalties when operations take place near the collection tail (e.g., add/remove at/from the end of the collection). Mainly, all elements situated before the adding/removing entry are left untouched, so the performance penalty can be ignored if we affect rows close to the collection tail.

**Key points:**\
     - an `@ElementCollection` doesn't have a Primary Key\
     - an `@ElementCollection` is mapped in a separate table\
     - pefer `@ElementCollection` with `@OrderColumn` when you have a lot of inserts and deletes from the collection tail\
     - the more items are inserted/removed from the beginning of this table the greater the performance penalty
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootElementCollectionWithOrderColumn/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

35. **[How To Avoid Lazy Initialization Caused By Open Session In View Anti-Pattern (1 Session / 1 HTTP Request-Response)](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootSuppressLazyInitInOpenSessionInView)**

**Description:** The Open-Session in View anti-pattern is activated by default in SpringBoot. If you prefer to use it then it is recommended to mitigate its performance penalties as much as possible. One optimization consist in marking the `Connection` as read-only which would allow the database server to avoid writing to the transaction log. Another optimization consist in explicitly setting the lazy properties of the fetched entities when you don't want them to be lazy initialized.

**Key points:**\
     - fetch a entity and set the lazy properties explicitly\
     - you can do this is the service or controller layer, depending where it fits better to your case, but outside of an explicit transaction\
     - why is this working? why we can set the property of a managed entity and not trigger the flush? well, the answer can be found in the documentation of `OpenSessionInViewFilter` which specifies that:
     
 **NOTE:** This filter will by default not flush the Hibernate `Session`, with the flush mode set to `FlushMode.NEVER`. It assumes to be used in combination with service layer transactions that care for the flushing: The active transaction manager will temporarily change the flush mode to `FlushMode.AUTO` during a read-write transaction, with the flush mode reset to `FlushMode.NEVER` at the end of each transaction. If you intend to use this filter without transactions, consider changing the default flush mode (through the "flushMode" property).
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootSuppressLazyInitInOpenSessionInView/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

36. **[How To Use Spring Projections(DTOs) And Inner Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaInnerJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and inner joins written via JPQL and native SQL (for MySQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-mysql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`, `PlayerRankNameDto`, `TournamentIdNameDto`)\
     - write inner joins queries using JPQL/SQL, for example:\
     - Query the tournaments of all players (`localhost:8080/tournamentsOfPlayersNamesInnerJoinJpql`)\
     - Query all tournaments that have players with rank smaller or equal to "rank" (`localhost:8080/tournamentsIdNameByRankInnerJoinSql`)
     
-----------------------------------------------------------------------------------------------------------------------    

37. **[How To Use Spring Projections(DTOs) And Left Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaLeftJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and left joins written via JPQL and native SQL (for MySQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-mysql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write left joins queries using JPQL/SQL, for example:\
     - Query all players even if they are not in tournaments (`localhost:8080/allPlayersLeftJoinJpql`)\
     - Query all tournaments even if they don't have players (`localhost:8080/allTournamentsLeftJoinJpql`)
     
-----------------------------------------------------------------------------------------------------------------------    

38. **[How To Use Spring Projections(DTOs) And Right Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaRightJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and right joins written via JPQL and native SQL (for MySQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-mysql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write right joins queries using JPQL/SQL, for example:\
     - Query all tournaments even if they don't have players (`localhost:8080/allTournamentsRightJoinJpql`)\
     - Query all players even if they are not in tournaments (`localhost:8080/allPlayersRightJoinJpql`)
     
-----------------------------------------------------------------------------------------------------------------------    

39. **[How To Use Spring Projections(DTOs) And Full Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaFullJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and right joins written via JPQL and native SQL (for PostgreSQL; MySQL does not support `FULL JOINS`).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-postgresql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write right joins queries using JPQL/SQL, for example:\
     - Query all tournaments and players (`localhost:8080/allTournamentsAndPlayersFullJoinJpql`)   
     
-----------------------------------------------------------------------------------------------------------------------    

40. **[How To Use Spring Projections(DTOs) And Left Excluding Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaLeftExcludingJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and left excluding joins written via JPQL and native SQL (we use MySQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-postgresql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write left excluding joins queries using JPQL/SQL, for example:\
     - Query all players that are not in tournaments (`localhost:8080/allPlayersLeftExcludingJoinJpql`)\
     - Query all tournaments that don't have players (`localhost:8080/allTournamentsLeftExcludingJoinJpql`)
     
-----------------------------------------------------------------------------------------------------------------------    

41. **[How To Use Spring Projections(DTOs) And Right Excluding Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaRightExcludingJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and right excluding joins written via JPQL and native SQL (we use MySQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-postgresql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write right excluding joins queries using JPQL/SQL, for example:\
     - Query all players that are not in tournaments (`localhost:8080/allPlayersRightExcludingJoinJpql`)\
     - Query all tournaments that don't have players (`localhost:8080/allTournamentsRightExcludingJoinJpql`)
     
-----------------------------------------------------------------------------------------------------------------------    

42. **[How To Use Spring Projections(DTOs) And Outer Excluding Joins](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoViaOuterExcludingJoins)**

**Description:** This application is a proof of concept for using Spring Projections(DTOs) and outer excluding joins written via JPQL and native SQL (we use PostgreSQL).

**Key points:**\
     - define serveral entities (e.g., `Tournament` and `Player` in a bidirectional `@OneToMany` relationship)\
     - populate the database with some test data (e.g., check the file `resources/data-postgresql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `TournamentPlayerNameDto`)\
     - write outer excluding joins queries using JPQL/SQL, for example:\
     - Query all tournaments that don't have players and all players that don't participate in tournaments (`localhost:8080/allTournamentsWithoutPlayersAndViceversaOuterExcludingJoinJpql`)
     
-----------------------------------------------------------------------------------------------------------------------    

43. **[How To Use Use Spring Post Commits](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootPostCommit)**

**Description:** This application is a proof of concept for using Spring post-commit hooks.

**Key points:**\
     - avoid time-consuming task in post-commits since the database connection will remain open until this code finshes

-----------------------------------------------------------------------------------------------------------------------    

44. **[How To Exploit Spring Projections(DTOs) And Join Unrelated Entities in Hibernate 5.1+](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDtoUnrelatedEntities)**

**Description:** This application is a proof of concept for using Spring Projections (DTOs) and join unrelated entities. Hibernate 5.1 introduced explicit joins on unrelated entities and the syntax and behaviour are similar to `SQL JOIN` statements.

**Key points:**\
     - define serveral entities (e.g., `Patient` and `Clinic` unrelated entities)\
     - populate the database with some test data (e.g., check the file `resources/data-mysql.sql`)\
     - write interfaces (projections) that contains getters for the columns that should be fetched from the database (e.g., `PatientNameAndMedicalHistoryDto`)\
     - write joins queries using JPQL/SQL, for example:\
     - Query all patients names and medical history with no current treatment (`localhost:8080/allPatientsNameAndMedicalHistoryNoTreatmentInnerJoinJpql`)

-----------------------------------------------------------------------------------------------------------------------    

45. **[Why To Avoid Lombok @EqualsAndHashCode in Entities](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootLombokEqualsAndHashCode)**

**Description:** Entities should implement `equals()` and `hashCode()` as [here](https://vladmihalcea.com/the-best-way-to-implement-equals-hashcode-and-tostring-with-jpa-and-hibernate/). The main idea is that Hibernate requires that an entity is equal to itself across all its state transitions (*transient*, *attached*, *detached* and *removed*). Using Lombok `@EqualsAndHashCode` will not respect this requirment.

**Key points:**\
**AVOID THESE APPROACHES**\
     - Using Lombok default behavior of `@EqualsAndHashCode`\
     (entity: `LombokDefaultProduct`, test: `LombokDefaultEqualsAndHashCodeTest`)\
     - Using Lombok  `@EqualsAndHashCode` with primary key only\
     (entity: `LombokIdProduct`, test: `LombokEqualsAndHashCodeWithIdOnlyTest`)\
     - Rely on default `equals()` and `hashCode()`\
     (entity: `DefaultProduct`, test: `DefaultEqualsAndHashCodeTest`)\
     - Implement `equals()` and `hashCode()` based only on primary key\
     (entity: `IdProduct`, test: `IdEqualsAndHashCodeTest`)

**PREFER THESE APPROACHES**\
     - Rely on `@NaturalId` (entity: `NaturalIdProduct`, test: `NaturalIdEqualsAndHashCodeTest`)\
     - Rely on primary key (entity: `GoodProduct`, test: `GoodEqualsAndHashCodeTest`)
     
**Good implementation of equals() and hashCode():**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootLombokEqualsAndHashCode/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

46. **[How To Avoid LazyInitializationException Via JOIN FETCH](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootJoinFetch)**

**Description:** Typically, when we get a `LazyInitializationException` we tend to modify the relationship fetching type from `LAZY` to `EAGER`. That is bad! This is a [code smell](https://vladmihalcea.com/eager-fetching-is-a-code-smell/). Best way to avoid this exception is to rely on `JOIN FETCH` + DTOs (if needed). This application is a `JOIN FETCH` example with no DTOs. But, based on the DTOs examples from this repo, you can easily adapt it to use DTOs as well.

**Key points:**\
     - define two related entities (e.g., `Category` and `Product` in a one-to-many lazy bidirectional relationship)\
     - write a JPQL `JOIN FETCH` to fetch a category including products\
     - write a JPQL `JOIN FETCH` to fetch all products including categories

**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootJoinFetch/sample%201.png) 
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootJoinFetch/sample2.png)

-----------------------------------------------------------------------------------------------------------------------    

47. **[How To Merge Entity Collections](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootMergeCollections)**

**Description:** This is a Spring Boot example based on the following [article](https://vladmihalcea.com/merge-entity-collections-jpa-hibernate/). Is a functional implementation of the Vlad's example. It is highly recommended to read that article.

**Key points:**\
     - Remove the existing database rows that are no longer found in the incoming collection\
     - Update the existing database rows which can be found in the incoming collection\
     - Add the rows found in the incoming collection, which cannot be found in the current database snapshot
     
-----------------------------------------------------------------------------------------------------------------------    

48. **[How To Delay Connection Acquisition As Needed (Hibernate 5.2.10)](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootDelayConnection)**

**Description:** This is a Spring Boot example that exploits Hibernate 5.2.10 capability of delaying the connection acquisition as needed. Normally, a database connection is aquried immediately after calling a method annotated with `@Transactional`. If this method contains some time-consuming tasks before the first SQL statement then the connection is holded for nothing. But, Hibernate 5.2.10 allows us to delay the connection acquisition as needed. This example rely on HikariCP as the default connection pool for Spring Boot.

**Key points:**\
     - set `spring.datasource.hikari.auto-commit=false` in application.properties\
     - set `spring.jpa.properties.hibernate.connection.provider_disables_autocommit=true` in application.properties
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootDelayConnection/sample1.png)
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootDelayConnection/sample2.png)

-----------------------------------------------------------------------------------------------------------------------    

49. **[How To Fetch Primary Keys Via Hibernate hi/lo Algorithm](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootHiLo)**

**Description:** This is a Spring Boot example of using the hi/lo algorithm for fetching 1000 PKs in 10 database roundtrips for batching 1000 inserts in batches of 10 inserts. The hi/lo algorithm is an optimization algorithm for generating sequences of identifiers.

**Key points:**\
     - use the `SEQUENCE` generator type (e.g., in PostgreSQL)\
     - configure the hi/lo algorithm as in `Player.java` entity
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootHiLo/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

50. **[How To Correctly Write a Bidirectional @ManyToMany Association](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootManyToManyBidirectional)**

**Description:** This is a Spring Boot proof of concept for writting a bidirectional `@ManyToMany` association.

**Key points:**\
     - we use two entities, `Tournament` and `Player`, and a tournament can have multiple players and a player can participate to multiple tournaments
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootManyToManyBidirectional/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

51. **[Prefer Set Instead of List in @ManyToMany Relationships](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootManyToManyBidirectionalListVsSet)**

**Description:** This is a Spring Boot example of removing rows in case of a bidirectional `@ManyToMany` using a `List` and a `Set`. The conclusion is that `Set` is much better! This applies to unidirectional as well!

**Key points:**\
     - using `Set` is much more efficent than `List`      
     
**Output example:**\
![](https://github.com/AnghelLeonard/Hibernate-SpringBoot/blob/master/HibernateSpringBootManyToManyBidirectionalListVsSet/sample.png)

-----------------------------------------------------------------------------------------------------------------------    

52. **[How To View Binding Params Via log4jdbc](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootLog4JdbcViewBindingParameters)**

**Description:** View the prepared statement binding parameters via [log4jdbc](https://stackoverflow.com/questions/45346905/how-to-log-sql-queries-their-parameters-and-results-with-log4jdbc-in-spring-boo/45346996#45346996)

**Key points:**\
     - in pom.xml, Add `log4jdbc` Dependency\
     - in application.properties add: `logging.level.resultsettable=info, logging.level.sqltiming=info, logging.level.sqlonly=fatal, logging.level.audit=fatal, logging.level.resultset=fatal, logging.level.connection=fatal`   

-----------------------------------------------------------------------------------------------------------------------    

53. **[How To View Binding Params Via TRACE](https://github.com/AnghelLeonard/Hibernate-SpringBoot/tree/master/HibernateSpringBootLogTraceViewBindingParameters)**

**Description:** View the prepared statement binding parameters via `TRACE`

**Key points:**\
     - in application.properties add: `logging.level.org.hibernate.type.descriptor.sql: TRACE`  
