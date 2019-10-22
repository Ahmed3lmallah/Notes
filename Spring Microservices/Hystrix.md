# Circuit Breaker Pattern

**Why Use the Circuit Breaker Pattern?**

* Remote calls can fail or “hang” until a timeout is reached. 
* Running out of resources can lead to cascading failures.

**How Circuit Breakers Work?** - based on Decorator pattern.

You wrap a protected function call in a circuit breaker object, which monitors for failures. Once the failures reach a certain threshold, the circuit breaker trips, and all further calls to the circuit breaker return with an error, without the protected call being made at all. Usually you'll also want some kind of monitor alert if the circuit breaker trips.

![image](https://martinfowler.com/bliki/images/circuitBreaker/sketch.png)

Circuit breaker status should be monitored and logged, and we must consider what to do when the breaker fails!

For software circuit breakers, we can have the breaker itself detect if the underlying calls are working again. We can implement this self-resetting behavior by trying the protected call again after a suitable interval, and resetting the breaker should it succeed.

![image](https://martinfowler.com/bliki/images/circuitBreaker/state.png)

Circuit breakers can also be used for asynchronous communications:
* Put all requests on a queue
* Open the circuit when the queue is full

**Netflix Hystrix:**

“Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.”

**Additional Resources:**

* [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
* [Fault Tolerance in a High Volume, Distributed System](https://medium.com/netflix-techblog/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a)
* [Netflix/Hystrix Git Repo](https://github.com/Netflix/Hystrix)

## Tutorial 1: [REST Consumer with Hystrix](https://spring.io/guides/gs/circuit-breaker/)

## Tutorial 1 Summary:

1. Add the `Netflix/Hystrix` dependency to the consuming service.
	
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
		
1. Add the `@EnableCircuitBreaker` annotation to the `main application` class.
1. Wrap the consuming method with the circuit breaker, and create the fallback method.

		@Service
		public class GreetingService {
			@HystrixCommand(fallbackMethod = "defaultGreeting")
			public String getGreeting(String username) {
				return new RestTemplate()
						.getForObject("http://localhost:9090/greeting/{username}",
								String.class, username);
			}

			private String defaultGreeting(String username) {
				return "Hello User!";
			}
		}

## Tutorial 2: [REST Consumer with Hystrix & FEIGN](https://www.baeldung.com/spring-cloud-netflix-hystrix)

## Tutorial 2 Summary:

1. Add the `Netflix/Hystrix` dependency to the consuming service.
	
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
		
1. Add the `@EnableCircuitBreaker` annotation to the `main application` class. 
1. Add the following configuration to `application.properties` file.
		
		feign.hystrix.enabled=true

1. Create a Fallback class that implements the FeignClient interface.

		@Component
		public class GreetingClientFallback implements GreetingClient {

			@Override
			public String greeting(@PathVariable("username") String username) {
				return "Hello User!";
			}
		}

1. Specify the Fallback class name in the `@FeignClient` annotation.

		@FeignClient(name = "rest-producer", fallback = GreetingClientFallback.class)
