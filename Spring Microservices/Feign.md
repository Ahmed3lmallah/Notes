# Edge Service

**What Is a Edge Service?**

**Edge Service** is an **entry point into a service that sits in front of an API.** In other words, it acts as a gateway to other services. Edge service is usually exposed to the public internet and is used to route HTTP requests to the appropriate service(s).

**Why Use an Edge Service?**

* Benefits
	* Allows us to request from multiple services with a single request
	* Encapsulates the internal structure of the app
	* May improve security through authentication
	* May improve performance through load balancing
	* Improves the user experience
	* Can be tailored to specific clients

* Concerns
	* Another component to manage and update

**How Does an Edge Service Work?**

1. A client makes a request
1. Looks for a microservice (or multiple microservices) to deliver the request
1. Multiple microservices are invoked and aggregated
1. Returns the response to the caller

**Additional Resources:**

* 
* 
* 

## Tutorial: [Edge Service / Feign Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/edge-service-feign-tutorial.md)

## Tutorial Summary:

*Services must be registred with Eureka to contact eachother.*

1. Add `OpenFeign` dependency.

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>


1. Enable Feign by adding `@EnableFeignClients` to the `main application` class.
1. Create the Feign Client

		@FeignClient(name = "random-greeting-service")
		public interface RandomGreetingClient {

			@RequestMapping(value = "/greeting", method = RequestMethod.GET)
			public String getRandomGreeting();
		}

1. Use FeignClient
	1. Autowire the interface
	1. Call it's methods as needed

			@Autowired
			private final RandomGreetingClient client;

			HelloCloudServiceController(RandomGreetingClient client) {
				this.client = client;
			}

			@RequestMapping(value="/hello", method = RequestMethod.GET)
			public String helloCloud() {

				return client.getRandomGreeting();
			}
