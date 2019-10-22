# Project Checklist

This checklist is designed to help you create and build microservices using a TDD approach.

1. Create a new project with Spring Initializr (start.spring.io). Include support for:
   1. Web
   2. MySQL
   3. JPA
   4. Eureka Client
2. Review the requirements
3. Create the Java class model and map the model to the database (use the SQL create script for guidance if supplied)
4. Design the REST API for the edge service.
5. Implement a shell of the REST API (with all endpoints returning null)
6. Implement MockMvc tests for the REST API
7. Design the REST API for the CRUD service
8. Implement a shell of this REST API (with all endpoints returning null)
9. Implement MockMvc tests for this REST API
10. Go back to your edge service project and design the Service layer.
11. Implement the shell of the service layer
12. Implement JUnit/Mockito unit tests for the Service Layer
13. Go back to the CRUD service and design the DAO
14. Implement the shell of the DAO
15. Implement JUnit integration tests for your DAO
16. Now implement the endpoints of the edge web service one at a time. Include the implementation of all service layer code, communication with the CRUD web service, and CRUD web service DAO code for each enpoint.
17. Repeat until all the endpoints are implemented and all tests pass.