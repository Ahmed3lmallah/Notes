# Spring Boot Controller

## Agenda

*   Review the code structure for a Controller
    
*   Write and Run JUnit Tests for a Controller
    
*   Write and Run Integration Tests for a Controller
    
*   Modify and submit the Controller
    
*   Add a method to expose a Service
    

## Controller Code

Spring Boot allows us to readily develop and deploy HTTP Accessible controllers. Turning a Java class into a Controller is done via annotations such as `@Controller` or `@RestController`.

Once Spring has been told that a Java class is now going to receive HTTP requests, the next thing is to show _which_ method to action against which URL.

This is done by using the `@GetMapping`, `@PostMapping`, `@DeleteMapping`, `@PutMapping` and `@PatchMapping` annotations. These annotations, when placed on a method, help Spring know where to route URLs.

### Example

    @RestController (1)
    @RequestMapping("/services") (2)
    public class SomeEndpoint {
    
      @GetMapping("/hello") (3)
      public String hello() { (4)
        return "hello";
      }
    
    }

1.  Annotation - Inform Spring that this is a REST Controller
    
2.  Annotation - Means that all `/services` belong to this `class`
    
3.  Annotation - Means that `GET /services/hello` should be redirected to this method
    
4.  Method - normal Java method
    

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

## Spring Context

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