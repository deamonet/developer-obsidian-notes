Last modified: February 23, 2023

Written by: [Eugen Paraschiv](https://www.baeldung.com/author/eugen "Posts by Eugen Paraschiv")

-   [Spring Data](https://www.baeldung.com/category/persistence/spring-persistence/spring-data)

-   [JPA](https://www.baeldung.com/tag/jpa)
 -   [Spring Data JPA](https://www.baeldung.com/tag/spring-data-jpa)

![announcement - icon](media/announcement_-_icon.png)

The right tools can and will save a lot of time. As long as you are **using Hibernate and IntelliJ IDEA** you can boost your coding speed and quality [with JPA Buddy](https://www.baeldung.com/JPA-Buddy-NPI-KjUtw). It will help in a lot of the day-to-day work:

-   Creating JPA entities that follow best practices for efficient mapping
-   Creating DTOs from entities and MapStruct mappers using convenient visual tools
-   Generating entities from the existing database or Swagger-generated POJOs
-   Visually composing methods for Spring Data JPA repositories
-   Generating differential SQL to update your schema in accordance with your changes in entities
-   Autogenerating Flyway migrations and Liquibase changelogs comparing entities with the database or two databases
-   … and a lot more

Simply put, you'll learn and use the best practices of Hibernate and surrounding technology and become a lot more!

Definitely [visit the JPA Buddy site](https://www.baeldung.com/JPA-Buddy-NPI-KjUtw) to see its features in action closer.

## **1. Overview**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#overview)

This tutorial will focus on **introducing Spring Data JPA into a Spring project,** and fully configuring the persistence layer. For a step-by-step introduction to setting up the Spring context using Java-based configuration and the basic Maven pom for the project, see [this article](https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration "Bootstrapping a web application with Spring 3.1 and Java based Configuration, part 1").

## Further reading:

## [A Guide to JPA with Spring](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)

Setup JPA with Spring - how to set up the EntityManager factory and use the raw JPA APIs.

[Read more](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) →

## [CrudRepository, JpaRepository, and PagingAndSortingRepository in Spring Data](https://www.baeldung.com/spring-data-repositories)

Learn about the different flavours of repositories offered by Spring Data.

[Read more](https://www.baeldung.com/spring-data-repositories) →

## [Simplify the DAO with Spring and Java Generics](https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics)

Simplify the Data Access Layer by using a single, generified DAO, which will result in <strong>elegant data access</strong>, no unnecessary clutter.

[Read more](https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics) →

## **2. The Spring Data Generated DAO – No More DAO Implementations**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#springdatadao)

As we discussed in an earlier article, [the DAO layer](https://www.baeldung.com/simplifying-the-data-access-layer-with-spring-and-java-generics "Removing the boilerplate from the DAO") usually consists of a lot of boilerplate code that can and should be simplified. The advantages of such a simplification are many: a decrease in the number of artifacts that we need to define and maintain, consistency of data access patterns, and consistency of configuration.

Spring Data takes this simplification one step further and **makes it possible to remove the DAO implementations entirely**. The interface of the DAO is now the only artifact that we need to explicitly define.

In order to start leveraging the Spring Data programming model with JPA, a DAO interface needs to extend the JPA specific _Repository_ interface, _JpaRepository_. This will enable Spring Data to find this interface and automatically create an implementation for it.

By extending the interface, we get the most relevant CRUD methods for standard data access available in a standard DAO.

## **3. Custom Access Method and Queries**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#customquery)

As discussed, **by implementing one of the _Repository_ interfaces, the DAO will already have some basic CRUD methods (and queries) defined and implemented**.

To define more specific access methods, Spring JPA supports quite a few options:

-   simply **define a new method** in the interface
-   provide the actual **JPQL query** by using the _@Query_ annotation
-   use the more advanced **Specification and Querydsl support** in Spring Data
-   define **custom queries** via JPA Named Queries

The [third option](http://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/), Specifications and Querydsl support, is similar to JPA Criteria, but uses a more flexible and convenient API. This makes the whole operation much more readable and reusable. The advantages of this API will become more pronounced when dealing with a large number of fixed queries, as we could potentially express these more concisely through a smaller number of reusable blocks.

The last option has the disadvantage that it either involves XML or burdening the domain class with the queries.

### **3.1. Automatic Custom Queries**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#1-automatic-custom-queries)

When Spring Data creates a new _Repository_ implementation, it analyses all the methods defined by the interfaces and tries to **automatically generate queries from the method names**. While this has some limitations, it's a very powerful and elegant way of defining new custom access methods with very little effort.

Let's look at an example. If the entity has a _name_ field (and the Java Bean standard _getName_ and _setName_ methods), **we'll define the _findByName_ method in the DAO interface.** This will automatically generate the correct query:

```java
public interface IFooDAO extends JpaRepository<Foo, Long> {

    Foo findByName(String name);

}
```

This is a relatively simple example. The query creation mechanism supports [a much larger set of keywords](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.query-methods.query-creation "Spring Data JPA - Query creation").

In case the parser can't match the property with the domain object field, we'll see the following exception:

```bash
java.lang.IllegalArgumentException: No property nam found for type class com.baeldung.spring.data.persistence.model.Foo
```

### **3.2. Manual Custom Queries**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#2-manual-custom-queries)

Now let's look at a custom query that we'll define via the _@Query_ annotation:

```java
@Query("SELECT f FROM Foo f WHERE LOWER(f.name) = LOWER(:name)")
Foo retrieveByName(@Param("name") String name);
```

For even more fine-grained control over the creation of queries, such as using named parameters or modifying existing queries, [the reference](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.named-parameters "Spring Data JPA - Query creation additional options") is a good place to start.

## **4. Transaction Configuration** [](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#transactions)

The actual implementation of the Spring-managed DAO is indeed hidden since we don't work with it directly. However, it's a simple enough implementation, **the _SimpleJpaRepository,_ which defines transaction semantics using annotations**.

More explicitly, this uses a read-only _@Transactional_ annotation at the class level, which is then overridden for the non-read-only methods. The rest of the transaction semantics are default, but these can be easily overridden manually per method.

### **4.1. Exception Translation Is Alive and Well** [](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#1-exception-translation-is-alive-and-well)

The question now becomes: since Spring Data JPA doesn't depend on the old ORM templates (_JpaTemplate_, _HibernateTemplate_), and they have been removed since Spring 5, are we still going to get our JPA exceptions translated to Spring's _DataAccessException_ hierarchy?

The answer is, of course, we are. **Exception translation is still enabled by the use of the _@Repository_ annotation on the DAO**. This annotation enables a Spring bean postprocessor to advise all _@Repository_ beans with all the _PersistenceExceptionTranslator_ instances found in the container, and provide exception translation just as before.

Let's verify exception translation with an integration test:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenInvalidEntityIsCreated_thenDataException() {
    service.create(new Foo());
}
```

Keep in mind that **exception translation is done through proxies.** In order for Spring to be able to create proxies around the DAO classes, these must not be declared _final_.

## **5. Spring Data JPA Repository Configuration**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#config)

To activate the Spring JPA repository support, we can use the _@EnableJpaRepositories_ annotation and specify the package that contains the DAO interfaces:

```java
@EnableJpaRepositories(basePackages = "com.baeldung.spring.data.persistence.repository") 
public class PersistenceConfig { 
    ...
}
```

We can do the same with an XML configuration:

```xml
<jpa:repositories base-package="com.baeldung.spring.data.persistence.repository" />
```

## **6. Java or XML Configuration**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#javaxml)

We already discussed in great detail how to [configure JPA in Spring](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) in a previous article. Spring Data also takes advantage of Spring's support for the JPA _@PersistenceContext_ annotation. It uses this to wire the _EntityManager_ into the Spring factory bean responsible for creating the actual DAO implementations, _JpaRepositoryFactoryBean_.

In addition to the already discussed configuration, we also need to include the Spring Data XML Config if we are using XML:

```java
@Configuration
@EnableTransactionManagement
@ImportResource("classpath*:*springDataConfig.xml")
public class PersistenceJPAConfig {
    ...
}
```

## **7. Maven Dependency**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#maven)

In addition to the Maven configuration for JPA, like in a [previous article](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa), we'll add [the _spring-data-jpa_ dependency](https://search.maven.org/search?q=g:org.springframework.data%20a:spring-data-jpa):

```xml
<dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-jpa</artifactId>
   <version>2.4.0</version>
</dependency>
```

## 8. Using Spring Boot[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#using-spring-boot)

**We can also use the [Spring Boot Starter Data JPA](https://search.maven.org/search?q=a:spring-boot-starter-data-jpa) dependency that will automatically configure the _DataSource_ for us.**

We need to make sure that the database we want to use is present in the classpath. In our example, we've added the H2 in-memory database:

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
   <version>2.7.2</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
```

As a result, just by doing these dependencies, our application is up and running and we can use it for other database operations.

**The explicit configuration for a standard Spring application is now included as part of Spring Boot auto-configuration.**

We can, of course, modify the auto-configuration by adding our customized explicit configuration.

Spring Boot provides an easy way to do this using properties in the _application.properties_ file. Let's see an example of changing the connection URL and credentials:

```java
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

## 9. Useful Tools for Spring Data JPA[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#useful-tools-for-spring-data-jpa)

Spring Data JPA is supported in all major Java IDEs. Let's see what useful tools are available for Eclipse and IntelliJ IDEA.

**If you use Eclipse as your IDE, you can install the [Dali Java Persistence Tools](https://www.eclipse.org/webtools/dali/downloads.php) plugin.** This provides ER diagrams for JPA entities, DDL generation to initialize schema, and basic reverse engineering capabilities. Also, you can use Eclipse Spring Tool Suite (STS). It will help to validate query method names in Spring Data JPA repositories.

In case you use IntelliJ IDEA, there are two options.

IntelliJ IDEA Ultimate enables ER diagrams, a JPA console for testing JPQL statements, and valuable inspections. However, these features are not available in the Community Edition.

**To boost productivity in IntelliJ, you can install the [JPA Buddy](https://plugins.jetbrains.com/plugin/15075-jpa-buddy) plugin,** which provides many features, including generation of JPA entities, Spring Data JPA repositories, DTOs, initialization DDL scripts, Flyway versioned migrations, Liquibase changelogs, etc. Also, JPA Buddy provides an advanced tool for reverse engineering.

Finally, the JPA Buddy plugin works with both Community and Ultimate editions.

## **10. Conclusion**[](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#conclusion)

In this article, we covered the configuration and implementation of the persistence layer with Spring 5, JPA 2, and Spring Data JPA (part of the Spring Data umbrella project) using both XML and Java-based configuration.

We discussed ways to define more **advanced custom queries**, as well as **transactional semantics**, and a **configuration with the new _jpa_ namespace**. The final result is a new and elegant take on data access with Spring, with almost no actual implementation work.

The implementation of this Spring Data JPA tutorial can be found in [the GitHub project](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo-2 "Spring Data JPA example project").