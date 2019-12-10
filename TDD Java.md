# Java Test Driven Development

**Table of contents:**

* [Introduction](#introduction)
	* [Uncle BOB’S 3 Rules of TDD](#uncle-bobs-3-rules-of-tdd)
	* [Summary](#summary)
* [Test Driven Development with Java and JUnit](#test-driven-development-with-java-and-junit)
	* [Anatomy of a JUnit Test](#anatomy-of-a-junit-test)
	* [How to write a test](#how-to-write-a-test)
* [How To MOCK with MOCKITO in JAVA](#how-to-mock-with-mockito-in-java)
	* [Setting Up MOCKITO](#setting-up-mockito)
	* [How MOCKITO Works?](#how-mockito-works)
		* [MOCKITO.VERIFY Examples](#mockitoverify-examples)
	* [How to setup a mock?](#how-to-setup-a-mock)
	* [Spring Context and Mocking](#spring-context-and-mocking)
* [Testing Spring Boot Controller](#testing-spring-boot-controller)
	* [Testing Controllers - Unit Tests](#testing-controllers---unit-tests)
	* [Testing Controllers - Integration Tests](#testing-controllers---integration-tests)
* [Testing Spring Context](#testing-spring-context)
* [TDD Java Best Practices](./TDD%20Best%20Practices.md)


## Introduction

### WHAT IS TEST DRIVEN DEVELOPMENT?

TDD is a simple 3-step process.

1.  Red: write a test, watch it fail.
2.  Green: write just enough code to pass the test.
3.  Refactor: improve the code without changing the behavior.

### WHY DO WE FOLLOW TDD?

##### BECAUSE TESTS DO MORE THAN CATCH BUGS!

*   They allow you to refactor safely
*   They allow you to add features safely
*   They act as an API Specification
*   They act as documentation
*   They facilitate team development

### AUTOMATED TESTS SAVE TIME AND ARE CONSISTENT

All developers already test their code!

1.  Developer has a problem.
2.  Developer writes code.
3.  Developer manually tests code, or deploy and manually test functionality.
4.  Repeat

##### Imagine testing your edge-cases over and over and over again.

Although this process might not take long (couple of minutes), it adds up. Ideally, an automated test should take milliseconds to run. On bigger projects, the time saved from automated testing more than makes up for the time writing the tests. This increases productivity.

### TESTS PROTECT YOUR CODE

Tests are a contract for expected behavior

Writing Tests:

*   Broadcasts defined expected behavior to team.
*   Broadcasts defined error-handling for edge cases.
*   Warns other developers when they break your code.

Without Tests:

*   How would you know if code gets broken?

Without tests, broken code is discovered at runtime…​ a waste in time and resources.

## UNCLE BOB’S 3 RULES OF TDD

1.  Write no production code except to pass a test.
2.  Write only enough of a test to demonstrate a failure.
3.  Write only enough production code to pass a test.

### RULE 1: Write no production code except to write a test.

*   Any code that can’t be tested cannot be trusted.
*   All code should be tested.
*   This could lead to uncomfortable scenarios.

*   This is what makes TDD hard!

### RULE 2: Write only enough of a test to demonstrate a failure.

*   Tests should be small in scope
*   Tests should only test one edge case or scenario
*   This isolates bugs so you can easily see where they occur
*   Write and pass each test one at a time.

### RULE 3: Write only enough production code to pass a test.

*   Write the EASIEST solution to pass a test (don’t future proof).
*   If you want to write more production code, write another test!
*   Write just enough code to get the job done - code is EXPENSIVE to maintain.

## SUMMARY

### BENEFITS OF TDD

*   Automates the testing process
*   Fast Feedback
*   Manageable code
*   Adding new features is easier
*   Tests act as API Specification
*   Errors are quickly isolated
*   Facilitates team development

### DISADVANTAGES OF TDD

*   High Learning Curve (shift in thinking)
*   Increased Development Time

# Test Driven Development with Java and JUnit

## Anatomy of a JUnit Test

Every test can be clearly written like this:

*   `@Test` annotation
*   Descriptive Test Name
*   Test Body (3 Phases):
	*   ARRANGE - Initial Conditions
	*   ACT - Method to Test
	*   ASSERT - Verification of Result

##### Example:

	@Test <1>  
	public void getBalance_ReturnsZero_WhenNoTransactions() { <2>  
	       // ArrangeAccount account = new Account(); <3>  
	       // Act  
	       Long balance = account.getBalance(); <4>  
	       // Assert  
	       assertEquals(0L, balance); <5>  
	}

1.  Annotation - Informs the project that this is a test method.
2.  Descriptive Test Name - <method to test_expected result_condition> - For readability.
3.  Arrange - Instantiating a new account
4.  Act - Calling the method we want to test
5.  Assert - Verify the condition of the method we called we are testing. The value you expect to get back should be first, following with the actual result from act.

### Red / Green / Refactor

This the cycle that we use to make sure we are doing TDD properly.

* ##### Red

We initially want to make a test be red, or failing, to verify our test is not giving us a false positive. If our test was initially passing in the beginning, but are trying to make it pass with our implementation, how do we know if our implementation is even proper to begin with?

There are two types of Red stages people know, the compiler error with a red squiggly line and an Assertion Error where the test is failing as shown below. For this exercise and for CDE Studios, we agree that the Red stage is where the test fails.

* ##### Green

After we have a proper red stage, we now have TDD’s permission to code in the implementation class. We will not change the test code to make the test pass. Our implementation should make the test pass. We also want the simplest solution to pass our test. When we run the test, we should see a green arrow given to us as shown below. It is also very good practice to verify that all tests are passing, to fix anything, before continuing.

* ##### Refactor

This is only done when we want to or find a need to refactor our code. We find that some solutions can be done more simple or we need to redesign our implementation. Before refactoring, make sure that all tests are passing. During the refactoring process, you will naturally break some of the code. This is okay. We are officially done refactoring when all the tests are passing again.

# How to write a test

Tests should be written using only the same API that a user will have access to. **This is called black box testing.**

### Black box testing

*   Black box testing ensures the expected result is obtained from predetermined initial conditions. It doesn’t matter how the result is achieved!

*   As you add more tests, you further define the algorithm.

**TIP**

> Your tests define behavior, but they shouldn’t "be aware" of internal implementation. They only need to be aware of what you need to put in your method or what you expect to get out of that method.

### Coupled Tests

*   When a test calls on a private member or relies on internal logic not known to a user, you have *COUPLED A TEST TO IMPLEMENTATION CODE*.

**WARNING**

> Since coupled tests are tied to the implementation code, refactoring usually breaks those coupled tests.
>
> If you refactor a class to produce the same results, but it still breaks 40 tests, then the tests were most likely coupled to the implementation.

# How To MOCK with MOCKITO in JAVA

## INTRODUCING MOCKITO

Mockito is a library for Java that simplifies dependency injection and testing.

*   Mockito allows you to define behavior!
*   Mockito allows you to verify when MOCKED calls have been made.
*   Mockito allows you to spy on internal components.
*   All without extending or implementing!

It allows a developer to easily stub a dependency for testing purposes.

## SETTING UP MOCKITO

### Spring Initializr- Mockito automatically included.

### Gradle - include in build.gradle

> Verify version with your team

	testCompile group: 'org.mockito', name: 'mockito-core', version: '2.1.0'

### Maven - include in pom.xml

	<dependency>  
	   <groupId>org.mockito</groupId>  
	   <artifactId>mockito-core</artifactId>  
	   <!-- Verify version with your team -->  
	   <version>2.1.0</version>  
	   <scope>test</scope>  
	</dependency>

## HOW MOCKITO WORKS?

Mockito quickly allows you to define the behavior of a dependency.

1.  To mock a dependency you first have to declare and initialize it.
2.  Once you have initialized the mock, it can be treated like you would any other object.

*   All methods for a mock by default return either null, the default value for a primitive, or an empty collection, as appropriate.

3.  Use `Mockito.when(…​)` to define behavior.

*   Mockito.when only works with mocks! Attempting to use Mockito.when with a real instance will throw an error.

##### Example

	TwentySidedDie die = Mockito.mock(TwentySidedDie.class);  
	  
	System.out.println("Value before Mockito.when: " + die.roll());  
	  
	Mockito.when(die.roll()).thenReturn(20);  
	  
	System.out.println("Value after Mockito.when: " + die.roll());

##### OUTPUT

Value before `Mockito.when`: 0 Value after `Mockito.when`: 20

4.  After you setup the mock, you can pass or inject it to other classes.
5.  Another powerful feature of Mockito is Mockito.verify, which is used to determine if a method has been called a specified number of times.

*   If you try to verify a method belonging to a real instance, you will get an Exception.

### MOCKITO.VERIFY EXAMPLES

	//Verify die.roll() was called 1 time  
	Mockito.verify(die, times(1)).roll();

	//Alternative way to Verify die.roll() was called 1 time  
	Mockito.verify(die).roll();

	  
	//Verify die.roll() was never called  
	Mockito.verify(die, times(0)).roll();  
	  
	//Verify die.roll() was called twice  
	Mockito.verify(die, times(2)).roll();

## How to setup a mock?

To use a mock, you must set up all 4 phases in any combination:

1.  Declare a variable as a mock
2.  Initialize the mock
3.  Define the behavior
4.  Inject the mock

### PHASE 1: DECLARATION

Declaring a Mock

*   `@Mock`
*   `Object object = Mockito.mock(Object.class);`

### PHASE 2: INSTANTIATION

Instantiating a Mock: If you do not instantiate, you will get a NullPointerException

*   `Object object = Mockito.mock(Object.class);`
*   `MockitoAnnotations.init(this);`
*   `@RunWith(MockitoJUnitRunner.class)`

### PHASE 3: DEFINING BEHAVIOR

Defining Behavior for a Mock

*   `when(object.method()).thenReturn(value);`

### PHASE 4: INJECTION

Injecting a Mock into a parent class.

*   `@InjectMocks` - compiler injects with constructor, setter, or by reflection
*   Constructor or Setter

## SPRING CONTEXT AND MOCKING

When Spring `@autowired` dependencies:

*   Spring searches for defined beans and "puts them in a bag".
*   When Spring encounters a dependency, it "searches for a bean from the bag".
*   It then injects the bean into the parent class.

### THE PROBLEM: SPRING CONTROLS INJECTION!

*   The developer can’t inject a mock because Spring injects everything!
*   Spring injects only with the beans it has defined.
*   Developer doesn’t call constructor or setter.

### SOLUTION?

*   Use `@MockBean`

*   `@MockBean` tells Spring to setup a mock and inject it instead of a real instance.
*   `@MockBean` should be used in place of `@Autowired`
*   Mock can then be used as normal

##### Spring Class:

	@Service  
	public class ParentService{  
	  
	   @Autowired  
	   private DependencyService;  
	}

##### Spring Test:

	@RunWith(SpringRunner.class)  
	@SpringBootTest  
	public class ServiceTest(){  
	     
		 @Autowired FooService service;  
	     @MockBean DependencyService dependency;  
	         
	     @Test  
         public void test(){  
             //Define behavior like normal  
             when(dependency.method()).thenReturn(value);  
             //call method to test  
             service.methodToTest();  
         }  
	}

`@InjectMocks` and `@Mock` WILL NOT WORK.

# Testing Spring Boot Controller

## Testing Controllers - Unit Tests

A Controller’s methods can be tested in the same way that any other class can be using JUnit and method-focused TDD.

    @RestController
    @RequestMapping("/services")
    public class SomeEndpoint { (1)
    
      @GetMapping("/hello")
      public String hello() { (2)
        return "hello";
      }
    
    }

1.  Class Under Test
    
2.  Method Under Test
    

Below is the corresponding JUnit Test

    public class SomeEndpointTest {
    
      @Test
      public void hello() {
        SomeEndpoint someEndpoint = new SomeEndpoint(); (1)
        assertEquals("hello",someEndpoint.hello());(2)
      }
    
    }

1.  Instantiate Controller
    
2.  Invoke and check method
    

## Testing Controllers - Integration Tests

Controllers only become special when certain annotations are added supporting Spring in adding additional behaviors. To test that this configurations are working as expected, Spring Context Integration Tests can be used to verify the following;

*   exposure of endpoints is as expected
    
*   payloads are being processed correctly and handed to the correct methods
    
*   responses and exceptions are managed correctly
    

To perform Spring Context Integration Testing correctly, we invoke the exposed endpoints via a client. This allows us to simulate a remote client making requests of our service. The main library that supports doing this is Spring’s `MockMvc`.

    @RestController
    @RequestMapping("/services") (1)
    public class SomeEndpoint {
    
      @GetMapping("/hello")  (2) (3)
      public String hello() {
        return "hello"; (4)
      }
    
    }

1.  Root endpoint under test
    
2.  `GET` method under test
    
3.  URL under test
    
4.  Expected response
    

Below is an integration test for the above code

    @RunWith(SpringRunner.class) (1)
    @SpringBootTest (2)
    @AutoconfigureMockMvc (3)
    public class SomeEndpointIT { (4)
    
      @Autowired (5)
      MockMvc mockMvc;
    
      @Test
      public void testGetHello() throws Exception { (6)
        mockMvc.perform(get("/service/hello")) (7)
          .andExpect(status().isOk()) (8)
          .andExpect(content().string("hello")); (9)
      }
    
    }

1.  Annotation - tells JUnit to work with Spring
    
2.  Annotation - sets up the Spring Application Context
    
3.  Annotation - sets up a `MockMvc` object (server for our method)
    
4.  Class - Test class differently named to our Unit Tests
    
5.  Annotation - tells Spring to inject a copy of our server into the test
    
6.  Method - using `MockMvc` it will throw an exception on failure and we need to throw it up
    
7.  Method - `MockMvc` "performs" the `GET` operation against our URL just like a client would
    
8.  Test - `MockMvc` "expects" that the HTTP Status Code will be `200`/`OK`
    
9.  Test - `MockMvc` "expects" the reply from the server to have _only_ the words `hello` in it
    

`MockMvc` runs our class under test as a server and allows for invoking it as if we were a HTTP Client. `MockMvc` will throw an exception should it not get a result it was expecting.

# Testing Spring Context

Have `src/main/java/com/cognizant/studio/Application.java` to look like the following;

    package com.cognizant.studio;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication (1)
    public class Application {
    
        public static void main(String... args) {
            SpringApplication.run(Application.class,args);(2)
        }
    }

1.  Tells Spring that this is a Spring Application
    
2.  Sets up the Spring Application Context
    

This can be tested by running the Spring Integration Test:

	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class ApplicationIT {

		@Autowired
		ApplicationContext applicationContext;

		@Test
		public void main() {
			//test to see if the application context started up successfully
			assertTrue(applicationContext.getBeanDefinitionCount() >= 0);
		}

	}

# [TDD Java Best Practices](./TDD%20Best%20Practices.md)