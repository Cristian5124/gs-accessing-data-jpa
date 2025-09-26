# Getting Started with Spring Data JPA

This guide walks you through the process of building an application that uses Spring Data JPA to store and retrieve data in a relational database.

## What You Will Build

You will build an application that stores `Customer` POJOs (Plain Old Java Objects) in a memory-based database.

## What You Need

* A favorite text editor or IDE
* JDK 17 or later
* Maven or Gradle (this guide uses Maven)

For detailed setup instructions, refer to the [Spring Guides prerequisites](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/prereq_editor_jdk_buildtools.adoc).

To complete this guide, follow the instructions in the [Spring Guides how-to](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/how_to_complete_this_guide.adoc).

## Starting with Spring Initializr

You can use this [pre-initialized project](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.1.5&packaging=jar&jvmVersion=17&groupId=com.example&artifactId=accessing-data-jpa&name=accessing-data-jpa&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.accessing-data-jpa&dependencies=data-jpa,h2) and click **Generate** to download a ZIP file. This project is configured to fit the examples in this tutorial.

To manually initialize the project:

1. Navigate to [https://start.spring.io](https://start.spring.io).
   This service pulls in all the dependencies you need for an application and does most of the setup for you.
2. Choose either Gradle or Maven and the language you want to use. This guide assumes that you chose Java.
3. Click **Dependencies** and select **Spring Data JPA** and **H2 Database**.
4. Click **Generate**.
5. Download the resulting ZIP file, which is an archive of a web application that is configured with your choices.

> **Note**: If your IDE has the Spring Initializr integration, you can complete this process from your IDE.
>
> **Note**: You can also fork the project from GitHub and open it in your IDE or other editor.

## Define a Simple Entity

In this example, you store `Customer` objects, each annotated as a JPA entity. The following listing shows the `Customer` class (in `src/main/java/com/example/accessingdatajpa/Customer.java`):

```java
package com.example.accessingdatajpa;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Customer {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  private String firstName;
  private String lastName;

  protected Customer() {}

  public Customer(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return String.format(
        "Customer[id=%d, firstName='%s', lastName='%s']",
        id, firstName, lastName);
  }

  public Long getId() {
    return id;
  }

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }
}
```

Here you have a `Customer` class with three attributes: `id`, `firstName`, and `lastName`. You also have two constructors. The default constructor exists only for the sake of JPA. You do not use it directly, so it is designated as `protected`. The other constructor is the one you use to create instances of `Customer` to be saved to the database.

The `Customer` class is annotated with `@Entity`, indicating that it is a JPA entity. (Because no `@Table` annotation exists, it is assumed that this entity is mapped to a table named `Customer`.)

The `Customer` object's `id` property is annotated with `@Id` so that JPA recognizes it as the object's ID. The `id` property is also annotated with `@GeneratedValue` to indicate that the ID should be generated automatically.

The other two properties, `firstName` and `lastName`, are left unannotated. It is assumed that they are mapped to columns that share the same names as the properties themselves.

The convenient `toString()` method prints out the customer's properties.

## Create Simple Queries

Spring Data JPA focuses on using JPA to store data in a relational database. Its most compelling feature is the ability to create repository implementations automatically, at runtime, from a repository interface.

To see how this works, create a repository interface that works with `Customer` entities as the following listing (in `src/main/java/com/example/accessingdatajpa/CustomerRepository.java`) shows:

```java
package com.example.accessingdatajpa;

import java.util.List;

import org.springframework.data.repository.CrudRepository;

public interface CustomerRepository extends CrudRepository<Customer, Long> {

  List<Customer> findByLastName(String lastName);

  Customer findById(long id);
}
```

`CustomerRepository` extends the `CrudRepository` interface. The type of entity and ID that it works with, `Customer` and `Long`, are specified in the generic parameters on `CrudRepository`. By extending `CrudRepository`, `CustomerRepository` inherits several methods for working with `Customer` persistence, including methods for saving, deleting, and finding `Customer` entities.

Spring Data JPA also lets you define other query methods by declaring their method signature. For example, `CustomerRepository` includes the `findByLastName()` method.

In a typical Java application, you might expect to write a class that implements `CustomerRepository`. However, that is what makes Spring Data JPA so powerful: You need not write an implementation of the repository interface. Spring Data JPA creates an implementation when you run the application.

## Create an Application Class

Spring Initializr creates a simple class for the application. The following listing shows the class that Initializr created for this example (in `src/main/java/com/example/accessingdatajpa/AccessingDataJpaApplication.java`):

```java
package com.example.accessingdatajpa;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataJpaApplication {

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataJpaApplication.class, args);
  }
}
```

For details on creating a Spring Boot application, refer to the [Spring Boot application guide](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/spring-boot-application-new-path.adoc).

Now you need to modify the simple class that the Initializr created for you. To get output (to the console, in this example), you need to set up a logger. Then you need to set up some data and use it to generate output. The following listing shows the finished `AccessingDataJpaApplication` class (in `src/main/java/com/example/accessingdatajpa/AccessingDataJpaApplication.java`):

```java
package com.example.accessingdatajpa;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class AccessingDataJpaApplication {

  private static final Logger log = LoggerFactory.getLogger(AccessingDataJpaApplication.class);

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataJpaApplication.class, args);
  }

  @Bean
  public CommandLineRunner demo(CustomerRepository repository) {
    return (args) -> {
      // save a few customers
      repository.save(new Customer("Jack", "Bauer"));
      repository.save(new Customer("Chloe", "O'Brian"));
      repository.save(new Customer("Kim", "Bauer"));
      repository.save(new Customer("David", "Palmer"));
      repository.save(new Customer("Michelle", "Dessler"));

      // fetch all customers
      log.info("Customers found with findAll():");
      log.info("-------------------------------");
      for (Customer customer : repository.findAll()) {
        log.info(customer.toString());
      }
      log.info("");

      // fetch an individual customer by ID
      Customer customer = repository.findById(1L);
      log.info("Customer found with findById(1L):");
      log.info("--------------------------------");
      log.info(customer.toString());
      log.info("");

      // fetch customers by last name
      log.info("Customer found with findByLastName('Bauer'):");
      log.info("--------------------------------------------");
      repository.findByLastName("Bauer").forEach(bauer -> {
        log.info(bauer.toString());
      });
      log.info("");
    };
  }
}
```

The `AccessingDataJpaApplication` class includes a `demo()` method that puts the `CustomerRepository` through a few tests. First, it fetches the `CustomerRepository` from the Spring application context. Then it saves a handful of `Customer` objects, demonstrating the `save()` method and setting up some data to work with. Next, it calls `findAll()` to fetch all `Customer` objects from the database. Then it calls `findById()` to fetch a single `Customer` by its ID. Finally, it calls `findByLastName()` to find all customers whose last name is "Bauer". The `demo()` method returns a `CommandLineRunner` bean that automatically runs the code when the application launches.

> **Note**: By default, Spring Boot enables JPA repository support and looks in the package (and its subpackages) where `@SpringBootApplication` is located. If your configuration has JPA repository interface definitions located in a package that is not visible, you can point out alternate packages by using `@EnableJpaRepositories` and its type-safe `basePackageClasses=MyRepository.class` parameter.

For instructions on building an executable JAR, refer to the [Spring Guides build guide](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/build_an_executable_jar_mainhead.adoc) and [build with both](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/build_an_executable_jar_with_both.adoc).

When you run your application, you should see output similar to the following:

```
== Customers found with findAll():
Customer[id=1, firstName='Jack', lastName='Bauer']
Customer[id=2, firstName='Chloe', lastName='O'Brian']
Customer[id=3, firstName='Kim', lastName='Bauer']
Customer[id=4, firstName='David', lastName='Palmer']
Customer[id=5, firstName='Michelle', lastName='Dessler']

== Customer found with findById(1L):
Customer[id=1, firstName='Jack', lastName='Bauer']

== Customer found with findByLastName('Bauer'):
Customer[id=1, firstName='Jack', lastName='Bauer']
Customer[id=3, firstName='Kim', lastName='Bauer']
```

## Summary

Congratulations! You have written a simple application that uses Spring Data JPA to save objects to and fetch them from a database, all without writing a concrete repository implementation.

> **Note**: If you want to expose JPA repositories with a hypermedia-based RESTful front end with little effort, you might want to read [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest).

## See Also

The following guides may also be helpful:

* [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest)
* [Accessing Data with Gemfire](https://spring.io/guides/gs/accessing-data-gemfire)
* [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb)
* [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql)
* [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j)

For additional resources, refer to the [Spring Guides footer](https://raw.githubusercontent.com/spring-guides/getting-started-macros/main/footer.adoc).