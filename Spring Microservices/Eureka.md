# Service Registry

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

## [Service Registry Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/service-registry-tutorial.md)

## Tutorial Summary:

### How to setup Service Registry?

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

### How to use Service Registry?

1. Add the `Eureka Discovery Client` dependency.

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		
1. Add the service name to `bootstrap.properties` if using a config server, or otherwise add it to the `application.properties`.

		spring.application.name = service-name
		
1. Add the `@EnbleDiscoveryClient` class level annotation to the main application class.
			
			
### How to call an endpoint of a service registered with Eureka using DiscoveryClient?

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
