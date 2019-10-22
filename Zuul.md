# Zuul Walkthrough

**Table of Contents:**

*   [Problem](#problem)
*   [Introducing Zuul](#introducing_zuul)
*   [How?](#how)
*   [Gotchas](#gotchas)
*	References:
	*	[Zuul wiki](https://github.com/Netflix/zuul/wiki)
	*	[Spring.io](https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html)
	*	[DZone.com](https://dzone.com/articles/microservices-communication-zuul-api-gateway-1)

## Problem

I’m writing a single page UI application and need to make a request to five different microservices. To make my application scalable, I need all the URLs of every deployed instance of those five services. That could mean dozens of URLs!

Also - every one of those instances needs to be configured to receive requests from external IP addresses. Not to mention CORS headers…​

## Introducing Zuul

Zuul is an API gateway built by Netflix. When configured properly, Zuul will forward requests to another microservice. It only takes a few lines of configuration in your `application.yml` to achieve this.

As put by the **[Zuul wiki](https://github.com/Netflix/zuul/wiki)**:

> Zuul is the front door for all requests from devices and web sites to the backend of the Netflix streaming application. As an edge service application, Zuul is built to enable dynamic routing, monitoring, resiliency and security.

Zuul allows us to load balance requests from our UI to our backend system. But it also mitigates some of the frustrations created by using a microservice architecture.

*   **Load balancing**: Since we use Eureka, Zuul will make round-robin calls to all instances that share a service ID.
    
*   **No CORS headaches**: All requests from your UI go to the same server that served your app. In other words, because the domains are the same, no CORS configuration is needed.
    
*   **No firewall headaches**: Since all requests from the UI are routed through Zuul, our UI microservice is the only one we need to expose to the public.

![image](/images/zuul.png)

## How?

*Zuul can be either enabled for the microservice hosting the UI or the Front-End microservice (FEMS), although it makes more sense to use Zuul for the UI service and Feign client for FEMS.*

1. The service need to be registered with Eureka.

> NB: Zuul can be implemented without Eureka server. In that case, you have to provide the exact URL of the service where it will be redirected.

1. Add Zuul dependecy:

	1. **MAVEN:** Add the following dependencies to your `pom.xml`:

			<dependency>
				<groupId>com.netflix.zuul</groupId>
				<artifactId>zuul-core</artifactId>
				<version>2.1.5</version>
			</dependency>

	1. **GRADLE:** Add the following dependencies to your `build.gradle`:

			compile "com.netflix.zuul:zuul-core:2.1.5"

1. Add the following annotations to your SpringBoot application:

		@EnableZuulProxy

1. Now Spring will search for Zuul configuration in our `application.yml`. We use this configuration to determine which requests will be routed to which microservices. Provided the Eureka dependency is in your classpath, the `@EnableDiscoveryClient` ensures Zuul will use Eureka for its service discovery.

	Here’s an `application.yml` Zuul example:

		zuul:
		  routes:
			my_users_path:
			  path: /myusers/**
			  url: https://example.com/users_service

	This sets up a route called `my_users_path`. Each request that your UI microservice receives under `/myusers/**` will be forwarded to `https://example.com/users_service/myusers/**`.

	For example, a request to `/myusers/xyz/123/abc` is forwarded to `https://example.com/users_service/myusers/xyz/123/abc`.

	Zuul can also use service IDs instead of hard-coded URLs. The URL will come from Eureka:

		zuul:
		  routes:
			my_users_path:
			  path: /myusers/**
			  serviceId: users-service
			  
	Or more simply:
	
		zuul:
		  routes:
			users-service: /myusers/**

## Gotchas

### 1\. Zuul will strip auth headers by default

To disable this, you need to add an extra line of configuration for each route:

    zuul:
      routes:
        my_users_path:
          path: /myusers/**
          sensitiveHeaders:
          serviceId: users-service

## 2\. Zuul will strip out your CORS headers by default

To disable this, you need to add an extra line of configuration for each route:

    zuul:
      routes:
        my_users_path:
          path: /myusers/**
          ignoredHeaders: 'Access-Control-Allow-Credentials, Access-Control-Allow-Origin'
          serviceId: users-service

## 3\. Setting timeout

With  `zuul.host.socket-timeout-millis=30000` we instruct Spring Boot to wait for the response for 30000 ms until Zuul's internal Hystrix timeout will kick off and show you the error.