# Repositories

## Learning Goals

- Define what a repository is and how we will use it.
- Elaborate on the purpose of the `CrudRepository` interface.
- Create a Spring Boot Project that implements the Spring Data dependency.
- Add a `ModelMapper` dependency for entity to DTO conversion.
- Add a repository, an entity, and connect the database to the application.
- Run the application to show all four CRUD operations working as expected.
- Mention how to add custom native queries to the repository class.

## Introduction

Our football application is almost complete! But there's one big problem. When we
restart the application, none of the data is persisted. Which means we'd have to
POST all of our data again to the application in order to perform any kind of
useful GET request. This isn't practical in the real world. So how do we make our
data persist?

You probably guessed it - and the answer is using a database!

We have seen earlier how to use JPA and Postgres to create entities, persist
them, query them, and create relationships between them. Everything we did
earlier can be done with Spring Data JPA, but it does offer some helper methods
that streamlines how we work with entities and interact with the database.

In this lesson, we will learn more about repositories and how we can use the
`CrudRepository` to create entities faster. We will continue with our football
team example and using the sports database we created in the last lesson.
Consider the following table again:

![create-football-team-table](https://curriculum-content.s3.amazonaws.com/spring-mod-1/application-properties/create-football-team-table.png)

## Repository Interfaces

A **repository** in Spring is an abstraction that helps reduce duplicate code
required for entities. Repositories are responsible for performing persistent
operations on data objects. We can specify a class as a repository by using the
`@Repository` annotation. The `@Repository` annotation is a specialization of the
`@Component`. It will also catch any persistence-specific exceptions if it
encounters any while performing database operations. There are several repository
interfaces and all of them are database agnostic.

The base repository interface looks like this:

```java
@Indexed
public interface Repository<T, ID> {

}
```

This is a marker interface which mainly exists to capture the entity type and
its primary key type. For example, if we create a `FootballTeam` entity with a
primary key of `int`, `T` would be `FootballTeam` and `ID` would be `Integer`.

The
[@Indexed](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/stereotype/Indexed.html?is-external=true)
annotation indicates that all interfaces that extend the `Repository` interface
should be treated as candidates for repository beans.

## CRUD Repository

The repository interface we'll mostly focus on is the
[CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html).
The `CrudRepository` interface will specifically focus on handling the CRUD
operations: create, read, update, and delete. Consider the following methods
included in the `CrudRepository` interface:

```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    // CREATE/UPDATE methods
    <S extends T> S save(S entity);
    <S extends T> Iterable<S> saveAll(Iterable<S> entities);

    // READ methods
    Optional<T> findById(ID id);
    boolean existsById(ID id);
    Iterable<T> findAll();
    Iterable<T> findAllById(Iterable<ID> ids);
    long count();

    // DELETE methods
    void deleteById(ID id);
    void delete(T entity);
    void deleteAllById(Iterable<? extends ID> ids);
    void deleteAll(Iterable<? extends T> entities);
    void deleteAll();
}
```

The methods for creating and updating entities are the same since Spring Data
knows when an entity already exists in the database and performs the correct
operation. Any entity that implements this interface has access to working
implementations of all of these methods.

Let's delve into each of these methods a little more:

| Method            | Parameters                                         | Returns                                               | Description                                                                                                                                                           |
|-------------------|----------------------------------------------------|-------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `save()`          | A single entity                                    | The saved entity                                      | This method will attempt to POST a non-null entity to the database. Will throw an `IllegalArgumentException` if the entity is `null`.                                 |
| `saveAll()`       | Iterable of entities, such as a `List` of entities | The saved entities                                    | This method will attempt to POST all given entities to the database given that it is not `null` nor contains any `null` elements.                                     |
| `findById()`      | ID of an entity                                    | The entity with the given ID or an `Optional.empty()` | This method will attempt to GET an entity based on the given ID passed in. The ID must not be `null`.                                                                 |
| `existsById()`    | ID of an entity                                    | `boolean`                                             | This method will look to see if an entity exists. If it does, it will return true; otherwise it will return false.                                                    |
| `findAll()`       | None                                               | Iterable of entities, such as a `List` of entities    | This method will GET and return all entities of the type.                                                                                                             |
| `findAllById()`   | Iterable of IDs, such as a `List` of IDs           | Iterable of entities, such as a `List` of entities    | This method will GET and return all instances entities with the given ID. The IDs must not be `null` nor contain any `null` IDs. Order of elements is not guaranteed. |
| `count()`         | None                                               | `long`                                                | This method will return the number of entities available.                                                                                                             |
| `deleteById()`    | ID of an entity                                    | void                                                  | If the entity is found with the given ID, this method will DELETE the entity from the database. The ID must not be `null`.                                            |
| `delete()`        | A single entity                                    | void                                                  | This method will attempt to DELETE the given entity. The entity must not be `null`.                                                                                   |
| `deleteAllById()` | Iterable of IDs, such as a `List` of IDs           | void                                                  | This method will attempt to DELETE all entities with the given ID. The IDs must not be `null` nor contain any `null` IDs.                                             |
| `deleteAll()`     | Iterable of entities, such as a `List` of entities | void                                                  | This method will attempt ot DELETE all of the given entities. The entities must not be `null` nor contain any `null` elements.                                        |
| `deleteAll()`     | None                                               | void                                                  | This method will DELETE all entities managed by the repository.                                                                                                       |

## Create a Spring Data Project

1. Go to [https://start.spring.io/](https://start.spring.io/).
2. Select the project properties.
    1. Select "Maven Project", as we will use Maven as the build tool.
    2. Select "Java" as the language.
    3. Select the most recent version of Spring Boot 2. (Make sure it does not have
       "SNAPSHOT" listed after it.)
    4. Change the "Artifact" metadata field to "spring-data-demo". (This will update
       the "Name" and "Package name" metadata fields too).
    5. Change the "Description" metadata field to "Demo project for Spring Data".
    6. Select the appropriate Java JDK version.
3. Add dependencies.
    1. Let's add the Spring Web dependency to create a Spring web application.
    2. Click "ADD DEPENDENCIES".
    3. Search for "spring web".
    4. Select "Spring Web" from the list.
    5. Click on "ADD DEPENDENCIES" again and add  "Lombok" from the list.
    6. Click on "ADD DEPENDENCIES" again and add "Spring Data JPA" from the list.
    7. Click on "ADD DEPENDENCIES" again and add "PostgreSQL Driver" from the list.
4. Click on the "Generate" button on the bottom. This will download a zip file
   containing the Spring Boot Project.
5. Unzip the archive and open it in IntelliJ or a preferred code editor.

![spring-data-initializr-configurations](https://curriculum-content.s3.amazonaws.com/spring-mod-1/spring-data/spring-initializr-data.png)

Let's go ahead and add the code we already have regarding our football team
example! We'll also go ahead and create a `repository` directory and an `entity`
directory under the `com.example.springdatademo` package as well, so we have
somewhere to put our repository class and entity classes.

Consider the following project structure and ensure yours looks the same:

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springdatademo
    │   │               ├── SpringDataDemoApplication.java
    │   │               ├── controller
    │   │               │   └── FootballController.java
    │   │               ├── dto
    │   │               │   └── FootballTeamDTO.java
    │   │               ├── entity
    │   │               ├── repository
    │   │               └── service
    │   │                   └── FootballService.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springdatademo
                        └── SpringDataDemoApplicationTests.java
```

```java
// FootballTeamDTO.java

package com.example.springdatademo.dto;

import lombok.Data;

@Data
public class FootballTeamDTO {
    private String teamName;
    private int wins;
    private int losses;
    private boolean currentSuperBowlChampion;
}
```

```java
// FootballService.java

package com.example.springdatademo.service;

import com.example.springdatademo.dto.FootballTeamDTO;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class FootballService {
    List<FootballTeamDTO> footballTeams;

    public FootballService() {
        footballTeams = new ArrayList<>();
    }

    public String addFootballTeam(FootballTeamDTO footballTeam) {
        footballTeams.add(footballTeam);
        return String.format("%s has been added!", footballTeam.getTeamName());
    }

    public FootballTeamDTO getFootballTeam(String teamName) {
        Optional<FootballTeamDTO> footballTeamDTO = footballTeams.stream()
                .filter(footballTeam -> footballTeam.getTeamName().equals(teamName))
                .findAny();

        // If the team does not exist, throw a NoSuchElementException
        return footballTeamDTO.orElseThrow();
    }
}
```

```java
// FootballController.java

package com.example.springdatademo.controller;

import com.example.springdatademo.dto.FootballTeamDTO;
import com.example.springdatademo.service.FootballService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RestController
public class FootballController {

    private final FootballService footballService;

    @Autowired
    public FootballController(FootballService footballService) {
        this.footballService = footballService;
    }

    @PostMapping("/football-team")
    public String addFootballTeam(@RequestBody FootballTeamDTO footballTeam) {
        return footballService.addFootballTeam(footballTeam);
    }

    @GetMapping("/football-team/{teamName}")
    public FootballTeamDTO getFootballTeam(@PathVariable String teamName) {
        return footballService.getFootballTeam(teamName);
    }
}
```

### Configure Database

As we saw in the last lesson, we'll need to add some properties to our
`application.properties` file. Go ahead and open up the `application.properties`
file and copy-paste the properties we discussed from the last lesson:

```properties
spring.datasource.url= jdbc:postgresql://localhost:5432/sports
spring.datasource.username= postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate ddl auto (create, create-drop, validate, update)
spring.jpa.hibernate.ddl-auto=create
spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

We mentioned in the last lesson we didn't have the proper dependencies added to
a project yet. In order to properly run these properties, we would need both the
Spring Data JPA dependency and the PostgreSQL Driver dependency.

Now that we have added both those dependencies, if we were to run our application
we would see an output similar to this:

![connect-to-database](https://curriculum-content.s3.amazonaws.com/spring-mod-1/repository/connect-to-postgresql-sport-db.png)

The lines in this log we want to look at have the word "hibernate" in it. These
show us that the application is connecting to the database. If there were any
issues with connecting to the database, we might see a stack trace of errors.

## Create the Entity Class

In the `entity` package, go ahead and create a `FootballTeam` class with the
following code:

```java
package com.example.springdatademo.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table(name = "football_team")
public class FootballTeam {

    @Id
    @GeneratedValue
    private int id;

    @Column(name = "team_name", nullable = false)
    private String teamName;

    private int wins;

    private int losses;

    @Column(name = "current_super_bowl_champion")
    private boolean currentSuperBowlChampion;
}

```

## Create the Repository Class

Now that we have the entity, we can create the repository. Create an interface
called `FootballRepository` in the `repository` package and add the following
code:

```java
package com.example.springdatademo.repository;

import com.example.springdatademo.entity.FootballTeam;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface FootballRepository extends CrudRepository<FootballTeam, Integer> {

    Optional<FootballTeam> findFootballTeamByTeamName(String teamName);
}

```

Note that we don’t have to add any method implementations to this repository.
Spring will automatically create an object that implements the
`FootballRepository` repository with all the CRUD methods and inject then into
the Spring context. But since we are going to use the `teamName` field to GET a
team, we have added a method to look up an entity by team name.

Notice that we don't have to include any implementation to this method. That is
because of the query builder mechanism built into the Spring Data repository
infrastructure. For more information, please see
[Query Creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation).

The directory structure should now look like this:

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springdatademo
    │   │               ├── SpringDataDemoApplication.java
    │   │               ├── controller
    │   │               │   └── FootballController.java
    │   │               ├── dto
    │   │               │   └── FootballTeamDTO.java
    │   │               ├── entity
    │   │               │   └── FootballTeam.java
    │   │               ├── repository
    │   │               │   └── FootballRepository.java
    │   │               └── service
    │   │                   └── FootballService.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springdatademo
                        └── SpringDataDemoApplicationTests.java
```

## Using the Entity and Repository Classes

Now that we have our entity and repository classes defined, let's put them to use!
To do so, we'll modify our service class, `FootballService`, along with the
`SpringDataDemoApplication`.

One of the first things we have to do is figure out a way to map the entity class,
`FootballTeam`, with the DTO class we created, `FootballTeamDTO`. To do so, we
will make use of a `ModelMapper`.

### ModelMapper

 The ModelMapper class is used to perform entity to DTO conversions. To utilize
this class, we'll need to add a dependency to our `pom.xml` file. Go ahead and
open up the `pom.xml` and add this under the `<dependencies>` section:

```xml
        <dependency>
            <groupId>org.modelmapper</groupId>
            <artifactId>modelmapper</artifactId>
            <version>3.0.0</version>
        </dependency>
```

Re-load the maven changes after this dependency is added.

Now that we have the dependency loaded in, we'll need to add a `ModelMapper` bean
to our configuration. Therefore, we'll make the following changes to the
`SpringDataDemoApplication`:

```java
package com.example.springdatademo;

import org.modelmapper.ModelMapper;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class SpringDataDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringDataDemoApplication.class, args);
    }

    // Add this bean method
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }

}
```

Since we will be using the `ModelMapper` to do the entity to DTO (and vice versa)
conversions, we'll add this to our `Service` class, as this falls under business
logic. We will also make use of the `ModelMapper` instance method, `map()`. The
`map()` method is what we use to perform the conversions and map the two
objects. When we call the `map()` method, we will pass in two arguments: the
_source_ and the _destination_, where the source is the object we want to
convert and the destination is the Type we want the source object to become.
For example, if we want to convert our `FootballTeam` entity to a
`FootballTeamDTO`, then we'd pass in the entity object along with the type
`FootballTeamDTO.class`:

```java
import com.example.springdatademo.dto.FootballTeamDTO;
import com.example.springdatademo.entity.FootballTeam;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class FootballService {
    
    private final ModelMapper modelMapper;
    
    @Autowired
    public FootballService(ModelMapper modelMapper) {
        this.modelMapper = modelMapper;
    }
    
    // Don't add this method - example only
    public FootballTeamDTO convertEntity(FootballTeam footballTeamEntity) {
        return modelMapper.map(footballTeamEntity, FootballTeamDTO.class);
    }
}
```

The `map()` method will then look at these arguments "to determine which
properties implicitly match according to a matching strategy and other
configuration."

For further information on the `ModelMapper`, see
[ModelMapper Getting Started Documentation](http://modelmapper.org/getting-started/#how-it-works).

### Modify the Service Class

Now that we know how to use the `ModelMapper` to map our entity-DTO objects, let
us see how to add the `FootballRepository` to our `FootballService` class. Make
the following changes to the service class:

```java
package com.example.springdatademo.service;

import com.example.springdatademo.dto.FootballTeamDTO;
import com.example.springdatademo.entity.FootballTeam;
import com.example.springdatademo.repository.FootballRepository;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Optional;

@Service
public class FootballService {

    private final ModelMapper modelMapper;
    private final FootballRepository footballRepository;

    @Autowired
    public FootballService(ModelMapper modelMapper, FootballRepository footballRepository) {
        this.modelMapper = modelMapper;
        this.footballRepository = footballRepository;
    }

    public String addFootballTeam(FootballTeamDTO footballTeam) {
        FootballTeam footballTeamEntity = modelMapper.map(footballTeam, FootballTeam.class);
        footballRepository.save(footballTeamEntity);
        return String.format("%s has been added!", footballTeam.getTeamName());
    }

    public FootballTeamDTO getFootballTeam(String teamName) {
        Optional<FootballTeam> optionalFootballTeam = footballRepository.findFootballTeamByTeamName(teamName);
        FootballTeam footballTeamEntity = optionalFootballTeam.orElseThrow();
        return modelMapper.map(footballTeamEntity, FootballTeamDTO.class);
    }
}
```

Let's look a little closer at what we did here:

- We no longer need to use the `List` data structure since we will be persisting
  the data to the database, so we can remove any code that has to do with adding
  and retrieving data from a `List`.
- We'll add an `@Autowired` constructor to reference the `ModelMapper` and
  `FootballRepository` beans.
- We now can modify the `addFootballTeam()` method.
  - Since the `FootballRepository` interface extends the `CrudRepository`, we can
    use the `save()` method we discussed earlier. In order to use that method, we
    need to pass an entity to it to persist to the database.
  - Since the `addFootballTeam()` method takes in a `FootballTeamDTO` object, we
    can use our `ModelMapper` to map the DTO to the entity:
    `FootballTeam footballTeamEntity = modelMapper.map(footballTeam, FootballTeam.class);`
  - Now we can save the `footballTeamEntity`:
    `footballRepository.save(footballTeamEntity);`
- We'll update the `getFootballTeam()` method too.
  - First, we'll call our new method, `findFootballTeamByTeamName()`, we
    created in the `FootballRepository`. We pass in the `teamName` to see if
    it can be found in the repository. This will return an
    `Optional<FootballTeam>`.
  - We can make use of the `orElseThrow()` method now to return either the
    `FootballTeam` entity or throw a `NoSuchElementException` if the value is
    `null`. If the value is `null`, we would get a 500 error in Postman.
  - Now we can use the `ModelMapper` to map the entity to the DTO, so we can
    return the DTO:
    `return modelMapper.map(footballTeamEntity, FootballTeamDTO.class);`

### Running the API

Let's run the application and see what happens. We will first need to add a
football team to the application.

Open up Postman and under the "Headers" tab, add "Content-Type":"application/json"
as shown in the screenshot below:

![Content-Type-Headers](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/api-headers-content-type.png)

Then we can add a football team in JSON format. The `FootballTeamDTO` has the
fields `teamName`, `wins`, `losses`, and `currentSuperBowlChampion`. So we will
have to make sure to include those in the JSON we want to send to our application.
We will add the following JSON to our application:

```json
{
    "teamName":"Dallas-Cowboys", 
    "wins":7, 
    "losses":3, 
    "currentSuperBowlChampion":0
}
```

In Postman, navigate to the "Body" tab, choose the "raw" radio button, and then
copy-paste the JSON in like so:

![JSON-response-body](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-post-football-team.png)

This will be the response body that we send to our REST API! For the request URL,
make sure "POST" is selected and that the URL is
"http://localhost:8080/football-team". Then click "Send" to post.

![Post-Football-Team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-add-football-team.png)

Now let's try sending a GET request using the path
"http://localhost:8080/football-team/Dallas-Cowboys":

![Get-Football-Team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/dto/postman-get-footbal-team.png)

The big test will be what happens when we stop the application. Let's try
restarting it! But before we completely start up the application again, navigate
back to the `application.properties` file and set the property
`spring.jpa.hibernate.ddl-auto` to `update`:

```properties
spring.jpa.hibernate.ddl-auto=update
```

Now start the application back up again and try sending the GET request again.
We should still get the same result as we did last time! If we open up pgAdmin4
and navigate to our sports database, we can even see the data there still!

![data-persisted](https://curriculum-content.s3.amazonaws.com/spring-mod-1/repository/pgadmin-select-all-1.png)

## Updating and Deleting

So far, our example only performs a POST and a GET request. But what if we wanted
to perform an UPDATE or DELETE request as well?

Let's modify the code to do so!

In the `FootballService` class, add the following methods:

```java
    public String updateFootballTeam(Integer id, FootballTeamDTO footballTeamDTO) {
        Optional<FootballTeam> optionalFootballTeam = footballRepository.findById(id);
        if (optionalFootballTeam.isPresent()) {
            FootballTeam footballTeamEntity = optionalFootballTeam.get();
            modelMapper.map(footballTeamDTO, footballTeamEntity);
            footballRepository.save(footballTeamEntity);
            return String.format("Team with ID %d has been updated", id);
        } else {
            return String.format("Team with ID %d was not updated; ID may not exist.", id);
        }
    }

    public String deleteFootballTeam(Integer id) {
        footballRepository.deleteById(id);
        return String.format("Team with ID %d was deleted", id);
    }
```

In the code above, we are updating and deleting the entities in the repository
using their `id` field. Notice to update a record, we use the same `save()`
method we did before when we performed a POST request. For deleting a record, we
will make use of the `deleteById()` method.

In the `FootballController` class, add the following methods:

```java
    @PutMapping("/football-team/{footballId}")
    public String updateFootballTeam(@PathVariable Integer footballId, @RequestBody FootballTeamDTO footballTeam) {
        return footballService.updateFootballTeam(footballId, footballTeam);
    }

    @DeleteMapping("football-team/{footballId}")
    public String deleteFootballTeam(@PathVariable Integer footballId) {
        return footballService.deleteFootballTeam(footballId);
    }
```

Here we are introducing two new annotations: the `@PutMapping` and the
`@DeleteMapping` annotation.

The `@PutMapping` annotation is shorthand for
`@RequestMapping(value="/football-team/{footballId}", method=RequestMethod.PUT)`.
This annotation will handle mapping HTTP PUT requests for whenever we want to
update a record.

The `@DeleteMapping` annotation is used for when we want to remove a record and is
shorthand for
`@RequestMapping(value="football-team/{footballId}", method=RequestMethod.DELETE)`.
This annotation will handle HTTP DELETE requests.

Let's run our application again and try to update the record in our database!
In Postman, navigate to the "Body" tab, choose the "raw" radio button, and then
copy-paste the JSON:

```json
{
    "teamName":"Pittsburgh-Steelers", 
    "wins":4, 
    "losses":7, 
    "currentSuperBowlChampion":0
}
```

Select "PUT" and enter in "http://localhost:8080/football-team/1" as the
request URL. Then click "Send" to update the record with an ID of 1 with
the given request body.

![postman-update-football-team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/repository/postman-update-football-team.png)

We can verify that the record was updated by navigating back to pgAdmin4 and
performing a `SELECT * FROM football_team query`:

![data-updated](https://curriculum-content.s3.amazonaws.com/spring-mod-1/repository/pgadmin-select-all-2.png)

Now let's try sending a DELETE request to delete the record with ID 1 in the
database. To send this request, we'll enter the request URL:
"http://localhost:8080/football-team/1". Remember to select "DELETE" next to the
request URL before clicking "Send".

![postman-delete-football-team](https://curriculum-content.s3.amazonaws.com/spring-mod-1/repository/postman-delete-football-team.png)

## Relationships in Spring Data JPA

Since Spring Data JPA is a complete implementation of the JPA specification,
its relationship annotations are the same. We can use all the annotations we
learned in the JPA and Hibernate section such as `@OneToOne`, `@OneToMany`, and
`@ManyToMany`. These annotations can still be used to define relationships
between entities within a Spring Boot application. The repositories will be able
to fetch the required data with relationships.

## Custom Queries

As we saw before, the query builder is pretty handy! The derived queries make it
easy for us; however, the method name can get pretty messy. For example,
consider the derived query method name:

```java
    Optional<FootballTeam> findTopByWinsAfter(Integer lowerBound);
```

We might be thinking "What does that method even do?" and that is because it may
not seem completely obvious or intuitive. The method above equates to the SQL
query:

```postgresql
SELECT *
FROM football_team
WHERE wins > lowerBound
LIMIT 1;
```

The query itself, in this case, is a little easier to understand. When the query
we are interested in gets a little too complex, it might be beneficial to use a
declared custom query with the `@Query` annotation. We could rewrite this method
to be:

```java
    @Query(value = "SELECT * FROM football_team WHERE wins > ? LIMIT 1", nativeQuery = true)
    Optional<FootballTeam> findTeamWithWinsMoreThan(Integer lowerBound);
```

This is a more hands-on approach when it comes to writing queries and allows us to
insert our pure native SQL queries. For more information on the `@Query`
annotation, see

There are pros and cons to this approach. A benefit is that it is a more
hands-on approach when it comes to writing queries and allows us to insert our
pure native SQL queries. The downfall to using native queries is that they are
not database agnostic. So if moving to a different database management system,
these native queries could cause a headache. For more information on the
`@Query` annotation, see
[Using @Query Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query).

## Conclusion

We have learned how to create CRUD repositories to make it easier to work with
entities and databases. We also created our first Spring Data JPA project.
Spring Data JPA provides all the benefits of JPA along with additional helpers
to streamline database interaction in our applications.

## References

- [Repository Annotation Documentation](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)
- [Indexed Annotation Documentation](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/stereotype/Indexed.html?is-external=true)
- [CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)
- [Query Creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation)
- [ModelMapper Getting Started Documentation](http://modelmapper.org/getting-started/#how-it-works)
- [PutMapping Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PutMapping.html)
- [DeleteMapping Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/DeleteMapping.html)
- [Using @Query Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query)