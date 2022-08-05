# Repositories

## Learning Goals

- Define repositories.
- Define a `CrudRepository`.
- Create an entity with a repository that extends `CrudRepository`.

## Introduction

We have seen earlier how to use JPA and Hibernate to create entities, persist
them, query them, and create relationships between them. Everything we did
earlier can be done with Spring Data JPA but it does offer some helper methods
that streamlines how we work with entities and interact with the database.

In this lesson, we will learn more about repositories and how we can use the
`CrudRepository` to create entities faster. We will be creating the following
`Teacher` entity:

![Teacher database diagram](https://curriculum-content.s3.amazonaws.com/java-spring-1/db-diagram-teacher.png)

## Repositories

A repository in Spring is an abstraction that helps reduce duplicate code
required for entities. There are several repository interfaces and all of them
are database agnostic.

The base interface is called
`[Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)`:

```java
@Indexed
public interface Repository<T, ID> {

}
```

This is a marker interface which mainly exists to capture the entity type and
its primary key type. For example, if we create a `Teacher` entity with a
primary key of `int`, `T` would be `Teacher` and `ID` would be `Integer`.

The
`[@Indexed](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/stereotype/Indexed.html?is-external=true)`
annotation indicates that all interfaces that extend the `Repository` interface
should be treated as candidates for repository beans.

## CRUD Repository

Spring Data provides a `[CrudRepository`
interface](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)
which extends the base `Repository` interface. It provides methods for each CRUD
action.

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

The methods for creation and updating entities are the same since Spring Data
knows when an entity already exists in the database and performs the correct
operation. Any entity that implements this interface access to working
implementations of all of these methods. Let’s create a project and try out
these methods on an entity.

## Create Spring Data Project

Go to the [https://start.spring.io/](https://start.spring.io/) and generate a
project with the following properties and dependencies. Once you have downloaded
the archive, open it in an IDE.

![Spring Initializr settings for ](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-initalizr-spring-data-hibernate.png)

### Configure Database

The database configuration has to be added in the `application.properties` file
in the `resources` folder. The configuration property names and values are very
similar to the ones we have used with a Hibernate project. Here is a sample
`persistence.xml` file we used for a Hibernate project:

```xml
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
                      http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">

    <persistence-unit name="example" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <properties>
            <!-- connect to database -->
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test" />
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
            <property name="javax.persistence.jdbc.user" value="sa" />
            <property name="javax.persistence.jdbc.password" value="" />
            <!-- configure behavior -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```

Open the `[application.properties](http://application.properties)` file and add
the following code:

```bash
# connect to database
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver

# configure database
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

The properties int the `# connect to database` section define the URL, username,
password, and driver for connecting to a database. In this case, we are using an
H2 database.

The `spring.jpa.hibernate.ddl-auto` property defines the database initialization
behavior. Here are some of the common values:

- `create-drop`: This drops all databases and creates new ones from scratch. It
  drops the database schema when the entity manager is closed using the
  `entityManager.close()` method.
- `create`: It is similar to `create-drop` but it does not drop the database
  tables when the entity manager is closed.
- `validate`: Checks if the entity definitions match an existing table schema.
- `update`: Does not drop databases. Only updates the table schema.
- `none`: Does not make any changes to the database.

We will use the `create` value when we first create and persist `Teacher`
instances and then switch to `none` for the read, update, and delete operations.

## Create Entity and Repository

Create a `models` and `repositories` package in the
`com.example.springdatajpaexample` package (or whatever the main package is in
your project).

Next, create a `Teacher` class in the `models` package and add the following
code:

```java
package com.example.springdatajpaexample.models;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Teacher {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

Now that we have the entity, we can create the repository. Create a class called
`TeacherRepository` in the `repositories` package and add the following code:

```java
package com.example.springdatajpaexample.repositories;

import com.example.springdatajpaexample.models.Teacher;
import org.springframework.data.repository.CrudRepository;

public interface TeacherRepository extends CrudRepository<Teacher, Integer> {

}
```

Note that we don’t have to add any method implementations to this repository.
Spring will automatically create an object that implements the
`TeacherRepository` repository with all the CRUD methods and inject into the
Spring context.

Your directory structure should look like this:

```
~project-root/
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springdatajpaexample
    │   │               ├── SpringDataJpaExampleApplication.java
    │   │               ├── models
    │   │               │   └── Teacher.java
    │   │               └── repositories
    │   │                   └── TeacherRepository.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── com
                └── example
                    └── springdatajpaexample
                        └── SpringDataJpaExampleApplicationTests.java
```

## How to Run Database Example Code

Since Spring Boot manages application lifecycle we can’t simply put our code in
the `main` method of the `SpringDataJpaExampleApplication` class. Instead, we
have to create a class that implements the `CommandLineRunner` interface to run
our example code once Spring Boot has finished setting things up.

Open the `SpringDataJpaExampleApplication` class and add the following code:

```java
package com.example.springdatajpaexample;

import com.example.springdatajpaexample.models.Teacher;
import com.example.springdatajpaexample.repositories.TeacherRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Component;

@SpringBootApplication
public class SpringDataJpaExampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringDataJpaExampleApplication.class, args);
	}

	@Component
	public class ApplicationStartupRunner implements CommandLineRunner {
		private final TeacherRepository repository;

		public ApplicationStartupRunner(TeacherRepository repository) {
			this.repository = repository;
		}

		@Override
		public void run(String... args) {
			// example code here
		}
	}

}
```

We have created an `ApplicationStartupRunner` class and made it a component with
the `@Component` annotation. The constructor has a `TeacherRepository` parameter
which means Spring Boot will automatically create and inject a repository bean
and assign it to the `repository` field in the class. Once Spring Boot has
finished setting things up, it will call the `run` method and this is where we
will be testing out the repository methods.

## Create

Make sure the `spring.jpa.hibernate.ddl-auto` property in the
`application.properties` file is set to `create` before proceeding.

```java
spring.jpa.hibernate.ddl-auto=create
```

Here are the steps for creating an entity and persisting it in the database:

1. Create entity object.
2. Set object properties using setters.
3. Call `repository.save(entity)` to save the `entity` to the database.

Let’s try this with the `Teacher` entity. Open up the
`SpringDataJpaExampleApplication` class again and update the `run` method.

```java
// imports

@SpringBootApplication
public class SpringDataJpaExampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringDataJpaExampleApplication.class, args);
	}

	@Component
	public class ApplicationStartupRunner implements CommandLineRunner {
		private final TeacherRepository repository;

		public ApplicationStartupRunner(TeacherRepository repository) {
			this.repository = repository;
		}

		@Override
		public void run(String... args) {
			Teacher teacher1 = new Teacher();
			Teacher teacher2 = new Teacher();

			teacher1.setName("Otha");
			teacher2.setName("Ian");

			repository.save(teacher1);
			repository.save(teacher2);
		}
	}

}
```

Now run the `main` method in the `SpringDataJpaExampleApplication` class to
start up Spring which will cause it to call the `run` method. You should have
the following table in the H2 database now:

| ID  | NAME |
| --- | ---- |
| 1   | Otha |
| 2   | Ian  |

Change the `spring.jpa.hibernate.ddl-auto` property in the
`application.properties` file to `none` before proceeding:

```java
spring.jpa.hibernate.ddl-auto=none
```

## Read

We will be using the `findById` and the `findAll` method. Update the `run`
method in the `SpringDataJpaExampleApplication` class:

```java
public void run(String... args) {
	Optional<Teacher> teacher1 = repository.findById(1);
	System.out.println("Teacher with ID 1:" + teacher1.orElse(null));

	Iterable<Teacher> teachers = repository.findAll();
	System.out.println("All Teachers:");
	teachers.forEach(System.out::println);
}
```

You should see the following output in the console for the `findById` call:

```
Hibernate: select teacher0_.id as id1_0_0_, teacher0_.name as name2_0_0_ from teacher teacher0_ where teacher0_.id=?
Teacher with ID 1:Teacher{id=1, name='Otha'}
```

For the `findAll` call the output should look similar to this:

```
Hibernate: select teacher0_.id as id1_0_, teacher0_.name as name2_0_ from teacher teacher0_
All Teachers:
Teacher{id=1, name='Otha'}
Teacher{id=2, name='Ian'}
```

We can see that Spring can automatically create the proper Hibernate queries and
fetch the desired resources from the database.

## Update

In order to update database records, we have to first fetch them using
`findById`, update the object instance, and call the the `repository.save`
method with the modified object instance.

Modify the `run` method to the following:

```java
@Override
public void run(String... args) {
	Optional<Teacher> teacher1Optional = repository.findById(1);

	Teacher teacher1 = teacher1Optional.orElseThrow(() -> new Error("object not found"));
	System.out.println("Teacher with ID 1:" + teacher1);

	teacher2.setName("Michelle");
	System.out.println("Teacher with ID 1 after update: " + teacher1);

	repository.save(teacher1);
}
```

```
Hibernate: select teacher0_.id as id1_0_0_, teacher0_.name as name2_0_0_ from teacher teacher0_ where teacher0_.id=?
Teacher with ID 1: Teacher{id=1, name='Otha'}
Teacher with ID 1 after update: Teacher{id=1, name='Michelle'}
Hibernate: select teacher0_.id as id1_0_0_, teacher0_.name as name2_0_0_ from teacher teacher0_ where teacher0_.id=?
Hibernate: update teacher set name=? where id=?
```

The database table should show the updated value now:

| ID  | NAME     |
| --- | -------- |
| 1   | Michelle |
| 2   | Ian      |

## Delete

We can use the `delete` or `deleteById` methods to delete an entity. We will be
using the `delete` method since we will be fetching a record from the database
and storing it as an object.

Open up the `SpringDataJpaExampleApplication` class and update the `run` method:

```java
@Override
public void run(String... args) {
	Optional<Teacher> teacher1Optional = repository.findById(1);
	Teacher teacher1 = teacher1Optional.orElseThrow(() -> new Error("object not found"));
	repository.delete(teacher1);
}
```

```java
Hibernate: select teacher0_.id as id1_0_0_, teacher0_.name as name2_0_0_ from teacher teacher0_ where teacher0_.id=?
Hibernate: select teacher0_.id as id1_0_0_, teacher0_.name as name2_0_0_ from teacher teacher0_ where teacher0_.id=?
Hibernate: delete from teacher where id=?
```

The first record in the database table should now be deleted:

| ID  | NAME |
| --- | ---- |
| 2   | Ian  |

## Relationships in Spring Data JPA

Since Spring Data JPA is a feature complete implementation of the JPA
specification, its relationship annotations are the same. We can use all of the
annotations we learned in the JPA and Hibernate section such as `@OneToOne`,
`OneToMany`, and `ManyToMany` to define relationships between entities. The
repositories will be able to fetch the required data with relationships.

## Conclusion

We have learned how to create CRUD repositories to make it easier to work with
entities and databases. We also created our first Spring Data JPA project.
Spring Data JPA provides all the benefits of JPA along with additional helpers
to streamline database interaction in our applications.
