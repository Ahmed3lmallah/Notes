# JPA Cheatsheet

### Documentation

* JpaRepository JavaDoc

### Building a Repository/DAO

1. Create project with Spring Initializr. Include support for:
   1. JPA
   2. MySQL
   3. Web
2. Create Java class model for given scenario
3. Annotate Java classes with JPA annotations to map the classes to database tables.
   1. Mark the class with ```@Entity```
   2. Include ```@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})``` so that Hibernate properties don't get serialized by our REST API.
   3. Include ```@Id``` and ```@GeneratedValue(strategy = GenerationType.AUTO)``` on the Id field (primary key) of the class.
   4. Include any One to Many relationships if any exist.
      1. ```@OneToMany``` annotation on the Set of related objects (see Customer/Customer Note as an example)
   5. Include any Many to One relationships to match the One to Many relationships.
      1. See Note class below for an example.
      2. Include @JoinColumn

```java
@Entity
@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})
public class Customer implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer customerId = new Integer(0);
    private String name;
    private String email;
    private String phone;
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    private Set<Note> notes = new HashSet<>();

    getters, setters, equals, and hash
	.
	.
	.
}
```

```java
@Entity
@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})
public class Note implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer noteId;
    private String content;
    @ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "customer_id")
    private Customer customer;

    getters, setters, equals, and hash
	.
	.
	.

}
```

3. Create the Repository interface(s):
   1. Use JpaRepository and the @Repository annotation.
   2. Create any necessary custom query methods.
			
			List<Customer> findByLastNameAndCompany(String lastName, String company);
			
4. Create test suite
	1. Remember to add class-level annotations:
			
			@RunWith(SpringRunner.class)
			@SpringBootTest
			public class CrmApplicationTests {...
	
	1. Use `@Autowired` to connect the test with Repositories
	1. Design a test implementation for each (important) Repo method.

## Database Configuration

```
spring.datasource.url: jdbc:mysql://localhost:3306/YOUR_SCHEMA_NAME_HERE?useSSL=false
spring.datasource.username: root
spring.datasource.password: rootroot
spring.datasource.driver-class-name: com.mysql.jdbc.Driver

spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
```
- create the schema
- ```spring.jpa.hibernate.ddl-auto``` allows us to specify how we want Spring Data JPA to act when we start our application. Possible values include:
  - `none`—This is the default for MySQL databases. Spring Data JPA will not do anything to alter the database structure on startup.
  - `update`—Spring Data JPA will modify the database structure based on the annotations of the Java @Entity classes.
  - **`create`—Spring Data JPA creates the database every time the application is started, but it does not drop the tables when the application quits.**
  - `create-drop`—Spring Data JPA create the data every time the application is started and drops all the tables when the application quits.
  - **We use the `create` value because we don't have a database**. After the initial run, we could switch to `none` or `update` depending on our project requirements.
- `spring.jps.show-sql=true` allows us to see the SQL statements that Spring Data JPA is executing.

POM file changes:

```
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.46</version>
		<scope>runtime</scope>
	</dependency>

```

