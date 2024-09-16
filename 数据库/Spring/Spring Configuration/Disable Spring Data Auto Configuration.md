Last modified: August 26, 2022

Written by: [baeldung](https://www.baeldung.com/author/baeldung "Posts by baeldung")

-   [Spring Boot](https://www.baeldung.com/category/spring/spring-boot)
-   [Spring Data](https://www.baeldung.com/category/persistence/spring-persistence/spring-data)

### **Get started with Spring Data JPA through the reference _Learn Spring Data JPA_ course:**

**[>> CHECK OUT THE COURSE](https://www.baeldung.com/learn-spring-data-jpa-course)**

## **1. Introduction**[](https://www.baeldung.com/spring-data-disable-auto-config#introduction)

## [](https://www.baeldung.com/spring-data-disable-auto-config#introduction)

In this quick tutorial, we'll explore two different ways to disable database auto-configuration in Spring Boot. This can come in handy [when testing](https://www.baeldung.com/spring-boot-exclude-auto-configuration-test).

We'll illustrate examples for Redis, MongoDB, and Spring Data JPA.

We'll start by looking at the annotation-based approach, and then we'll look at the property file approach.

## **2. Disable Using Annotation**[](https://www.baeldung.com/spring-data-disable-auto-config#disable-using-annotation)

## [](https://www.baeldung.com/spring-data-disable-auto-config#disable-using-annotation)

Let's start with the [MongoDB](https://www.baeldung.com/spring-data-mongodb-tutorial) example. We'll look at classes that need to be excluded:

```java
@SpringBootApplication(exclude = {
    MongoAutoConfiguration.class, 
    MongoDataAutoConfiguration.class
})
```

Similarly, we'll look at disabling auto-configuration for [Redis](https://www.baeldung.com/spring-data-redis-tutorial):

```java
@SpringBootApplication(exclude = {
    RedisAutoConfiguration.class, 
    RedisRepositoryAutoConfiguration.class
})
```

Finally, we'll look at disabling auto-configuration for [Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa):

```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class, 
    DataSourceTransactionManagerAutoConfiguration.class, 
    HibernateJpaAutoConfiguration.class
})
```

## **3. Disable Using Property File**[](https://www.baeldung.com/spring-data-disable-auto-config#disable-using-property-file)

## [](https://www.baeldung.com/spring-data-disable-auto-config#disable-using-property-file)

We can also disable auto-configuration using the property file.

We'll first explore it with MongoDB:

```java
spring.autoconfigure.exclude= \
  org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration, \
  org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration
```

Now we'll disable it for Redis:

```java
spring.autoconfigure.exclude= \
  org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration, \
  org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration
```

Finally, we'll disable it for Spring Data JPA:

```java
	spring.autoconfigure.exclude= \ 
	  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration, \
	  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration, \
	  org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration
```

## **4. Testing**[](https://www.baeldung.com/spring-data-disable-auto-config#testing)

## [](https://www.baeldung.com/spring-data-disable-auto-config#testing)

For testing, we'll check that the Spring beans for the auto-configured classes are absent in our [application context](https://www.baeldung.com/spring-web-contexts).

We'll start with the test for MongoDB. We'll verify if the _MongoTemplate_ bean is absent:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() { 
    context.getBean(MongoTemplate.class); 
}
```

Now we'll check for JPA. For JPA, the _DataSource_ bean will be absent:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() {
    context.getBean(DataSource.class);
}
```

Finally, for Redis, we'll check the _RedisTemplate_ bean in our application context:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() {
    context.getBean(RedisTemplate.class);
}
```

## **5. Conclusion**[](https://www.baeldung.com/spring-data-disable-auto-config#conclusion)

## [](https://www.baeldung.com/spring-data-disable-auto-config#conclusion)

In this brief article, we learned how to disable Spring Boot auto-configuration for different databases.

The source code for all the examples in this article is available over on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data).