# Configuration Server

**What are the advantages of externalizing configuration settings?**

* Settings can be changed without having to rebuild code.
* Settings can be changed without having to restart the application.
* Settings are centralized.
* We can trace changes to settings.
* We can provide encryption and decryption of sensitive settings.


## [Config Server Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/config-server-tutorial.md)

## Tutorial Summary:

### How to setup configuration server?

1. Create a new (or use an existing) GitHub repository to store application configuration files.
1. Create a new IntelliJ project called `config-server` for the configuration server using Spring Initializr with the `Config Server` dependency.
1. Add the `@EnableConfigServer` annotation to the main project class.
1. Define the `server.port` and `spring.cloud.config.server.git.uri` for Github in the `application.properties` file.

		server.port=9999
		spring.cloud.config.server.git.uri=https://github.com/Ahmed3lmallah/configServer.git
		spring.cloud.config.server.git.username: username
		spring.cloud.config.server.git.password: password

1. Start the server and visit ```http://localhost:9999/properties-file-name/master```.

### How to utilize a configuration server?

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