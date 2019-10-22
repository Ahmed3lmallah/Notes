# Microservices

### Table of Contents

  * [Configuration Server](#Configuration-Server) - [Tutorial](#config-server-tutorial)
  * [Service Registry](#Service-Registry) - [Tutorial](#service-registry-tutorial)
  * [Queues](#Queues) - [Tutorial](#tutorial-spring-rabbitmq-tutorial)
  * [Cache](#cache) - [Tutorial](#tutorial-spring-data-caching-tutorial)
  * [Edge Service](#edge-service) - [Tutorial](#tutorial-edge-service--feign-tutorial)
  * [Circuit Breaker](#circuit-breaker-pattern) - [Tutorial](#tutorial-1-rest-consumer-with-hystrix)

## Configuration Server

**What are the advantages of externalizing configuration settings?**

* Settings can be changed without having to rebuild code.
* Settings can be changed without having to restart the application.
* Settings are centralized.
* We can trace changes to settings.
* We can provide encryption and decryption of sensitive settings.


### [Config Server Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/config-server-tutorial.md)

### Tutorial Summary:

#### How to setup configuration server?

1. Create a new (or use an existing) GitHub repository to store application configuration files.
1. Create a new IntelliJ project called `config-server` for the configuration server using Spring Initializr with the `Config Server` dependency.
1. Add the `@EnableConfigServer` annotation to the main project class.
1. Define the `server.port` and `spring.cloud.config.server.git.uri` for Github in the `application.properties` file.

		server.port=9999
		spring.cloud.config.server.git.uri=https://github.com/Ahmed3lmallah/configServer.git
		spring.cloud.config.server.git.username: username
		spring.cloud.config.server.git.password: password

1. Start the server and visit ```http://localhost:9999/properties-file-name/master```.

#### How to utilize a configuration server?

1. Add the `Config client` and `Spring Boot Actuator` dependencies to the service.
		
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		
1. Add `app-name-service.properties` file to the GitHub repository.
	1. Include server port, refresh scope configuration, and any other properties we would need to configure the service

			server.port=8080
			# Allow for RefreshScope
			management.endpoints.web.exposure.include=*
 
1. Add `bootstrap.properties` file to the resources folder
	1. include the config server uri
	1. include the service name (**Must match the properties file name in Github**)
	
			spring.application.name=service-name-service
			spring.cloud.config.uri=http://localhost:9999

1. Add `@RefreshScope` to the controller class
1. To extract predefined variables from the properties file we can use: 
		
		@Value("${officialGreeting}")
		private String officialGreeting;

1. Start the service.

*To update the service Beans if changes were made to the .properties file we can make use of `@RefreshScope` by using `$ curl localhost:8080/actuator/refresh -d {} -H "Content-Type: application/json"`*

## Service Registry

**What Is a Service Registry?**

A service registry is a database of services, their status, their instances, and their locations. Applications can use the service registry to dynamically discover and call registered services. Clients use the service registry to know where to send their requests.

Similar to how DNS is used to find the IP address of a site, the service registry leads to the discovery of the service, aka **service discovery**.

**How Does a Service Registry Work?**

1. When the client registers itself as a service, it includes metadata including its host and port.
1. The service sends a "heartbeat" to the registry.
* If an instance of the service does not deliver a consistent “heartbeat,” the service registry will remove the instance from the registry.

**Advantages of Using a Service Registry:**

* Service instances are registered at startup and deregistered at shutdown.
* Allows the client to find available instances of a service.
* Can use the Health Check API to verify that the service is available to handle the request.

**Drawbacks of Using a Service Registry:**

* Is a critical system component—therefore, it needs to be highly available and up-to-date.

### [Service Registry Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/service-registry-tutorial.md)

### Tutorial Summary:

#### How to setup Service Registry?

1. Create a new IntelliJ project: `eureka-service-registry` using Spring Initializr include the `Eureka Server` dependency.
1. Add `@EnableEurekaServer` annotation to the main application class
1. Set the host name and port on which our Eureka Server will listen and configure some Eureka specific settings in the `application.properties` file.

		server.port=8761
		eureka.instance.hostname=localhost

		# Shut off the client functionality of the Eureka server (used for HA)
		eureka.client.registerWithEureka=false
		eureka.client.fetchRegistry=false
		eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka

	The ```registerWithEureka``` and ```fetchRegistry``` setting turn off the high availability/clustering features of Eureka because we are only running a single node.

1. Start your new Eureka Service Registry and visit ```http://localhost:8761```.

#### How to use Service Registry?

1. Add the `Eureka Discovery Client` dependency.

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		
1. Add the service name to `bootstrap.properties` if using a config server, or otherwise add it to the `application.properties`.

		spring.application.name = service-name
		
1. Add the `@EnbleDiscoveryClient` class level annotation to the main application class.
			
			
#### How to call an endpoint of a service registered with Eureka using DiscoveryClient?

*Both services must be registered with Eureka*

1. Add Information of the service to be called in the Configuration file of the calling service. 

		randomGreetingServiceName=service-name <- service to be called info
		serviceProtocol=http://
		servicePath=/greeting

1. Use RestTemplate and DiscoveryClient to call the service from any method.

			@Autowired
			private DiscoveryClient discoveryClient;

			private RestTemplate restTemplate = new RestTemplate();

			@Value("${randomGreetingServiceName}")
			private String randomGreetingServiceName;

			@Value("${serviceProtocol}")
			private String serviceProtocol;

			@Value("${servicePath}")
			private String servicePath;

			public String helloCloud() {

				List<ServiceInstance> instances = discoveryClient.getInstances(randomGreetingServiceName);

				String randomGreetingServiceUri = serviceProtocol + instances.get(0).getHost() + ":" + instances.get(0).getPort() + servicePath;

				String greeting = restTemplate.getForObject(randomGreetingServiceUri, String.class);

				return greeting;
			}
		}

## Queues 

**Asynchronous vs. synchronous processing:** 

Synchronous processing means that you can only execute one process at a time, while Asynchronous processing means multiple process can be executed at a time and you don't have to finish executing the current process in order to move on to next one.

**What is a Queue?**

**Queue** is a **linear data structure, or an ordered list of elements of similar data types**. in which the first element is inserted from one end called the **REAR (also called tail)**, and the removal of existing element takes place from the other end called as **FRONT (also called head)**. This makes queue as **FIFO (First in First Out) data structure**, which means that element inserted first will be removed first. The process to add an element into queue is called Enqueue and the process of removal of an element from queue is called Dequeue.

**What problems do Queues solve?**

1. Queues enable asynchronous communication, which means that the endpoints that are producing and consuming messages interact with the queue, not each other. Producers can add requests to the queue without waiting for them to be processed. Consumers process messages only when they are available. No component in the system is ever stalled waiting for another, optimizing data flow.

1. Queues make your service reliable, and reduce the errors that happen when different parts of your system go offline. By separating different components with message queues, you create more fault tolerance. If one part of the system is ever unreachable, the other can still continue to interact with the queue. The queue itself can also be mirrored for even more availability.

**How Does a Queue Work?**

1. Producer creates a new entry/message.
	* Entry/message includes the exchange name and a binding key (routing key).
1. Entry/message is sent to a message broker such as RabbitMQ.
1. Entry/message is routed to the appropriate queue(s) based on the binding key and distribution protocols.
1. Consumer listening to the Queue receives the message when available.

### Terminology

It's helpful to review queue-related terminlogy before we look at the design and begin implementation.

#### AMQP

Advanced Message Queuing Protocol (**AMQP**) is a messaging protocol that allows clients and messaging middleware to communicate in a standardized manner. RabbitMQ and the Spring client libraries conform to AMQP.

#### Producer

The **producer** is the application, process, or code that creates new entries and places them in the queue for processing.

#### Consumer

The **consumer** is the application, process, or code that processes, or consumes, the queue entries created by the producer.

#### Queue

A **queue** is a repository for messages. Producers place the messages in the queue. Consumers process the messages.

#### Exchange

An **exchange** is where producers send messages in an AMQP system. An exchange then routes the messages to one or more queues based on the type of exchange and routing rules (called bindings, explained below).

**Topic exchanges** route messages to one or more queues based on a routing key and the pattern used to bind the exchange to the queue or queues. Topic exchanges allow consumers to choose which types of messages they want to process and which ones they want to ignore.

#### Binding

A **binding** is a rule that an exchange uses to route messages to a queue. Routing keys and binding rules can be used to filter messages and only send certain messages to certain queues.

**Additional Resources:**

* [Message Queues - AWS](https://aws.amazon.com/message-queue/)

### Tutorial: [Spring RabbitMQ Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-rabbitmq-tutorial.md)

### Tutorial Summary:

1. Install RabbitMQ
1. Creare the Queue consumer (an independent service)
	1. Create the Consumer Application using the Spring Initializr with `Spring for RabbitMQ` dependency.
	1. Create the Message Class in `Queueconsumer.util.messages.msg.java`
		
		*Include getters, setters, custom constructors (for convenience), and toString methods to that class. If we decide to add a custom constructor, we must also include the default constructor. Jackson requires a default constructor to marshal and unmarshal the messages.*
		
	1. Add the Jackson Converter Libraries to `pom.xml` file:

			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-core</artifactId>
				<version>2.9.8</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-annotations</artifactId>
				<version>2.9.8</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-databind</artifactId>
				<version>2.9.8</version>
			</dependency>

	1. Configure the Exchange, Queue, Binding, and Converter in the `main application` class:

				public static final String TOPIC_EXCHANGE_NAME = "queue-demo-exchange";
				public static final String QUEUE_NAME = "email-list-add-queue";
				public static final String ROUTING_KEY = "email.list.add.#";

				@Bean
				Queue queue() {
					return new Queue(QUEUE_NAME, false);
				}

				@Bean
				TopicExchange exchange() {
					return new TopicExchange(TOPIC_EXCHANGE_NAME);
				}

				@Bean
				Binding binding(Queue queue, TopicExchange exchange) {
					return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY);
				}

				@Bean
				public Jackson2JsonMessageConverter jackson2JsonMessageConverter() {
					return new Jackson2JsonMessageConverter();
				}

	1. Create the Message Listener: `Queueconsumer.MessageListener.java`

			@Service
			public class MessageListener {

				@RabbitListener(queues = EmailListQueueConsumerApplication.QUEUE_NAME)
				public void receiveMessage(EmailListEntry msg) {
					System.out.println(msg.toString());
				}
			}
			
1. Configure the producer service to use the queue:
	1. Add the `Spring for RabbitMQ` dependency to the Producer Application.

			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-amqp</artifactId>
			</dependency>
			
	1. Create the Message Class - identical to the message class we created in the consumer application above. 
	1. Configure the `RabbitTemplate` and Message Converter in the `main application` class:

			@Bean
			public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
				RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
				rabbitTemplate.setMessageConverter(jackson2JsonMessageConverter());
				return rabbitTemplate;
			}

			@Bean
			public Jackson2JsonMessageConverter jackson2JsonMessageConverter() {
				return new Jackson2JsonMessageConverter();
			}

	1. Use `RabbitTemplate` to convert and send messages to the queue inside any method:

			public void sendMessage(Msg msg) {
				System.out.println("Sending message...");
				rabbitTemplate.convertAndSend(EXCHANGE, ROUTING_KEY, msg);
				System.out.println("Message Sent");
			}

**How to use RabbitMQ GUI?** 

1. Go to root directory. Default: C:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.16\sbin>
1. Use RabbitMQ CMD
1. `rabbitmq-plugins enable rabbitmq_management`
1. Restart the rappitmq server
1. visit localhost:15672/
1. Login with username: `guest` and Password: `guest`

## Cache

**What Is a Cache?**

A **cache is a copy of the original data** that is stored for future reference. Previously accessed data is stored in a cache to make future retrieval faster.

**Why Use a Cache?**

* It’s less expensive.
	* Saves money
	* Saves processing time
* Data transformation requires processing power. Once that data is transformed, caching the results saves our app from have to process it again.
* Improved user experience.

**How Does a Cache Work?**

1. The app attempts to fetch the data from the cache.
1. If the data exists in the cache, it is presented.
1. If the data is not in the cache, the data is requested from the database.
1. This data may then be cached for faster retrieval when future requests are made.

**Things to Consider When Using a Cache**

* Caches take up disk space.
* Caches are most useful with generic data; i.e., not user-specific.
* Cached data may not be the most recent, therefore not accurate.
	* If the accuracy of the data is critical, a cache is likely not the best solution.
* When data changes often, a cache is likely not very effective.

**Difference between Cache and buffer:**

**Caching** is accessing a copy of the data, while **Buffering** preloads data from the “original” source. Buffering is usually used with large file sizes to match the transmission speed of the sender and the receiver.

**Additional Resources:**

* 
* 
* 

### Tutorial: [Spring Data Caching Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-caching-tutorial.md)

### Tutorial Summary:

1. Add the `Spring cache abstraction` dependency to `pom.xml` file

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		
1. Annotate the `Main Application` Class with `@EnableCaching`
1. Annotate the Controller:

	We will use the following annotations:

	* class level annotation
		* @CacheConfig(cacheNames = {"cache"})
	* method level annotation.
		* @CachePut(key = "#result.getId()") 
		* @Cacheable
		* @CacheEvict(key = "#rsvp.getId()") 

## Edge Service

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

### Tutorial: [Edge Service / Feign Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/edge-service-feign-tutorial.md)

### Tutorial Summary:

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

## Circuit Breaker Pattern

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

### Tutorial 1: [REST Consumer with Hystrix](https://spring.io/guides/gs/circuit-breaker/)

### Tutorial 1 Summary:

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

### Tutorial 2: [REST Consumer with Hystrix & FEIGN](https://www.baeldung.com/spring-cloud-netflix-hystrix)

### Tutorial 2 Summary:

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

#### [Go Back...](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)