# JAX-RS Client API

JAX-RS provides a client API for accessing REST resources from other Java applications.

**Table of Contents:**

* [Overview of the Client API](#overview-of-the-client-api)
    * [Required Dependencies](#required-maven-dependencies)
    * [Creating a Basic Client Request Using the Client API](#creating-a-basic-client-request-using-the-client-api)
        1. [Obtaining the Client Instance](#1-obtaining-the-client-instance)
        1. [Setting the Client Target](#2-setting-the-client-target)
        1. [Setting Path Parameters in Targets](#3-setting-path-parameters-in-targets)
        1. [Invoking the Request](#4-invoking-the-request)
            * [GET](#the-return-type-should-correspond-to-the-entity-returned-by-the-target-rest-resource)
            * [POST](#if-the-target-rest-resource-is-expecting-an-http-post-request-call-the-invocationbuilderpost-method)
            * [Collections](#if-the-return-type-is-a-collection-use-javaxwsrscoregenerictypet-as-the-response-type-parameter-where-t-is-the-collection-type)
         
* [Using the Client API in the JAX-RS Example Applications](#using-the-client-api-in-the-jax-rs-example-applications)
    1. [The Client API in the rsvp Example Application](#1-the-client-api-in-the-rsvp-example-application)
    1. [The Client API in the customer Example Application](#2-the-client-api-in-the-customer-example-application)

* [Advanced Features of the Client API](#advanced-features-of-the-client-api)
    1. [Configuring the Client Request](#i-configuring-the-client-request)
        1. [Setting Message Headers in the Client Request](#1-setting-message-headers-in-the-client-request) 
        1. [Setting Cookies in the Client Request](#2-setting-cookies-in-the-client-request)
        1. [Adding Filters to the Client](#3-adding-filters-to-the-client)
    1. [Asynchronous Invocations in the Client API](#ii-asynchronous-invocations-in-the-client-api)
        1. [Using Custom Callbacks in Asynchronous Invocations](#1-using-custom-callbacks-in-asynchronous-invocations)

* [References](#references)
* [Read More](#read-more)

# Overview of the Client API

The JAX-RS Client API provides a high-level API for accessing any REST resources, not just JAX-RS services. The Client API is defined in the `javax.ws.rs.client` package.

## Required Maven Dependencies

Let's begin by adding the required dependencies (for Jersey JAX-RS client) in the pom.xml:

    <dependency>
        <groupId>org.glassfish.jersey.core</groupId>
        <artifactId>jersey-client</artifactId>
        <version>2.25.1</version>
    </dependency>

To use Jackson 2.x as JSON provider:

    <dependency>
        <groupId>org.glassfish.jersey.media</groupId>
        <artifactId>jersey-media-json-jackson</artifactId>
        <version>2.25.1</version>
    </dependency>

The latest version of these dependencies can be found at [jersey-client](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.core%22%20AND%20a%3A%22jersey-client%22) and [jersey-media-json-jackson](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.media%22%20AND%20a%3A%22jersey-media-json-jackson%22).

## Creating a Basic Client Request Using the Client API

The following steps are needed to access a REST resource using the Client API:

1. Obtain an instance of the javax.ws.rs.client.Client interface.
1. Configure the Client instance with a target.
1. Create a request based on the target.
1. Invoke the request.

The Client API is designed to be fluent, with method invocations chained together to configure and submit a request to a REST resource in only a few lines of code.

    Client client = ClientBuilder.newClient();
    String name = client.target("http://example.com/webapi/hello")
            .request(MediaType.TEXT_PLAIN)
            .get(String.class);

In this example, the client instance is first created by calling the `javax.ws.rs.client.ClientBuilder`.newClient method. Then, the request is configured and invoked by chaining method calls together in one line of code. The `Client.target` method sets the target based on a URI. The `javax.ws.rs.client.WebTarget.request` method sets the media type for the returned entity. The `javax.ws.rs.client.Invocation.Builder.get` method invokes the service using an HTTP GET request, setting the type of the returned entity to String.

### 1. Obtaining the Client Instance

The Client interface defines the actions and infrastructure a REST client requires to consume a RESTful web service. Instances of Client are obtained by calling the `ClientBuilder.newClient` method.

    Client client = ClientBuilder.newClient();

Use the close method to close Client instances after all the invocations for the target resource have been performed:

    Client client = ClientBuilder.newClient();
    ...
    client.close();

**Client instances are heavyweight objects.** For performance reasons, limit the number of Client instances in your application, as the initialization and destruction of these instances may be expensive in your runtime environment.

### 2. Setting the Client Target

The target of a client, the REST resource at a particular URI, is represented by an instance of the `javax.ws.rs.client.WebTarget` interface. You obtain a WebTarget instance by calling the `Client.target` method and passing in the URI of the target REST resource.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi");

For complex REST resources, it may be beneficial to create several instances of WebTarget. In the following example, a base target is used to construct several other targets that represent different services provided by a REST resource.

    Client client = ClientBuilder.newClient();
    WebTarget base = client.target("http://example.com/webapi");
    // WebTarget at http://example.com/webapi/read
    WebTarget read = base.path("read");
    // WebTarget at http://example.com/webapi/write
    WebTarget write = base.path("write");

The `WebTarget.path` method **creates a new WebTarget instance** by appending the current target URI with the path that was passed in.

### 3. Setting Path Parameters in Targets

Path parameters in client requests can be specified as URI template parameters. Template parameters are specified by surrounding the template variable with braces ({}). Call the `resolveTemplate` method to substitute the {username}, and then call the queryParam method to add another variable to pass.

    // http://example.com/webapi/read/janedoe?chapter=1
    WebTarget myResource = client.target("http://example.com/webapi/read")
            .path("{userName}")
            .resolveTemplate("userName", "janedoe")        
            .queryParam("chapter", "1");
    Response response = myResource.request(...).get();

### 4. Invoking the Request

After setting and applying any configuration options to the target, call one of the `WebTarget.request` methods to begin creating the request. This is usually accomplished by passing to `WebTarget.request` **the accepted media response type for the request either as a string of the MIME type or using one of the constants in `javax.ws.rs.core.MediaType.`** The WebTarget.request method returns an instance of `javax.ws.rs.client.Invocation.Builder`, a helper object that provides methods for preparing the client request.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    Invocation.Builder builder = myResource.request(MediaType.TEXT_PLAIN);

Using a MediaType constant is equivalent to using the string defining the MIME type:

    Invocation.Builder builder = myResource.request("text/plain");

After setting the media type, invoke the request by calling one of the methods of the `Invocation.Builder` instance that corresponds to the type of HTTP request the target REST resource expects. These methods are:

* get()
* post()
* delete()
* put()
* head()
* options()

#### The return type should correspond to the entity returned by the target REST resource.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    String response = myResource.request(MediaType.TEXT_PLAIN)
        .get(String.class);

#### If the target REST resource is expecting an **HTTP POST request**, call the `Invocation.Builder.post` method.

    Client client = ClientBuilder.newClient();
    StoreOrder order = new StoreOrder(...);
    WebTarget myResource = client.target("http://example.com/webapi/write");
    TrackingNumber trackingNumber = myResource.request(MediaType.APPLICATION_XML)
        .post(Entity.xml(order), TrackingNumber.class);

In the preceding example, the return type is a custom class and is retrieved by setting the type in the `Invocation.Builder.post(Entity<?> entity, Class<T> responseType)` method as a parameter.

By default, if a return type wasn't specified, invoking the request would return a `Response` object. For example, the following invokation, `.post(Entity.entity(employee, MediaType.APPLICATION_JSON));` results in a Response object with methods such as `.getStatus`.
 
#### If the return type is a collection, use `javax.ws.rs.core.GenericType<T>` as the response type parameter, where T is the collection type:

    List<StoreOrder> orders = client.target("http://example.com/webapi/read")
            .path("allOrders")
            .request(MediaType.APPLICATION_XML)
            .get(new GenericType<List<StoreOrder>>() {});

This preceding example shows how methods are chained together in the Client API to simplify how requests are configured and invoked.

## Using the Client API in the JAX-RS Example Applications

This section describes how two example applications uses the Client API.

### 1. The Client API in the rsvp Example Application

The rsvp application allows users to respond to event invitations using JAX-RS resources. The web application uses the Client API in CDI backing beans to interact with the service resources, and the Facelets web interface displays the results.

The StatusManager CDI backing bean retrieves all the current events in the system. The client instance used in the backing bean is obtained in the constructor:

    public StatusManager() {
        this.client = ClientBuilder.newClient();
    }

The `StatusManager.getEvents` method returns a collection of all the current events in the system by calling the resource at `http://localhost:8080/rsvp/webapi/status/all`, which returns an XML document with entries for each event. The Client API automatically unmarshals the XML and creates a `List<Event>` instance.

    public List<Event> getEvents() {
            List<Event> returnedEvents = null;
            try {
                returnedEvents = client.target(baseUri)
                        .path("all")
                        .request(MediaType.APPLICATION_XML)
                        .get(new GenericType<List<Event>>() {
                });
                if (returnedEvents == null) {
                    logger.log(Level.SEVERE, "Returned events null.");
                } else {
                    logger.log(Level.INFO, "Events have been returned.");
                }
            } catch (WebApplicationException ex) {
                throw new WebApplicationException(Response.Status.NOT_FOUND);
            }
            ...
            return returnedEvents;
        }

The `StatusManager.changeStatus` method is used to update the attendee's response. It creates an HTTP POST request to the service with the new response. The body of the request is an XML document.

    public String changeStatus(ResponseEnum userResponse, Person person, Event event) {
            String navigation;
            try {
                logger.log(Level.INFO, 
                        "changing status to {0} for {1} {2} for event ID {3}.",
                        new Object[]{userResponse,
                            person.getFirstName(),
                            person.getLastName(),
                            event.getId().toString()});
                client.target(baseUri)
                        .path(event.getId().toString())
                        .path(person.getId().toString())
                        .request(MediaType.APPLICATION_XML)
                        .post(Entity.xml(userResponse.getLabel()));
                navigation = "changedStatus";
            } catch (ResponseProcessingException ex) {
                logger.log(Level.WARNING, "couldn''t change status for {0} {1}",
                        new Object[]{person.getFirstName(), person.getLastName()});
                logger.log(Level.WARNING, ex.getMessage());
                navigation = "error";
            }
            return navigation;
        }
### 2. The Client API in the customer Example Application

The customer example application stores customer data in a database and exposes the resource as XML. The service resource exposes methods that create customers and retrieve all the customers. A Facelets web application acts as a client for the service resource, with a form for creating customers and displaying the list of customers in a table.

The `CustomerBean` stateless session bean uses the JAX-RS Client API to interface with the service resource. The `CustomerBean.createCustomer` method takes the Customer entity instance created by the Facelets form and makes a POST call to the service URI.

    public String createCustomer(Customer customer) {
        if (customer == null) {
            logger.log(Level.WARNING, "customer is null.");
            return "customerError";
        }
        String navigation;
        Response response =
                client.target("http://localhost:8080/customer/webapi/Customer")
                .request(MediaType.APPLICATION_XML)
                .post(Entity.entity(customer, MediaType.APPLICATION_XML),
                        Response.class);
        if (response.getStatus() == Status.CREATED.getStatusCode()) {
            navigation = "customerCreated";
        } else {
            logger.log(Level.WARNING, 
                    "couldn''t create customer with id {0}. Status returned was {1}",
                    new Object[]{customer.getId(), response.getStatus()});
            FacesContext context = FacesContext.getCurrentInstance();
            context.addMessage(null, 
                    new FacesMessage("Could not create customer."));
            navigation = "customerError";
        }
        return navigation;
    }

The XML request entity is created by calling the `Invocation.Builder.post` method, passing in a new Entity instance from the Customer instance, and specifying the media type as `MediaType.APPLICATION_XML`.

The `CustomerBean.retrieveCustomer` method retrieves a Customer entity instance from the service by appending the customer's ID to the service URI.

    public String retrieveCustomer(String id) {
        String navigation;
        Customer customer =
                client.target("http://localhost:8080/customer/webapi/Customer")
                .path(id)
                .request(MediaType.APPLICATION_XML)
                .get(Customer.class);
        if (customer == null) {
            navigation = "customerError";
        } else {
            navigation = "customerRetrieved";
        }
        return navigation;
    }

The `CustomerBean.retrieveAllCustomers` method retrieves a collection of customers as a `List<Customer> `instance. This list is then displayed as a table in the Facelets web application.

    public List<Customer> retrieveAllCustomers() {
        List<Customer> customers =
                client.target("http://localhost:8080/customer/webapi/Customer")
                .path("all")
                .request(MediaType.APPLICATION_XML)
                .get(new GenericType<List<Customer>>() {
                });
        return customers;
    }

Because the response type is a collection, the Invocation.Builder.get method is called by passing in a new instance of `GenericType<List<Customer>>`.

## Advanced Features of the Client API

This section describes some of the advanced features of the JAX-RS Client API.

### I. Configuring the Client Request

Additional configuration options may be added to the client request after it is created but before it is invoked.

#### 1. Setting Message Headers in the Client Request

You can set HTTP headers on the request by calling the `Invocation.Builder.header` method.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    String response = myResource.request(MediaType.TEXT_PLAIN)
            .header("myHeader", "The header value")
            .get(String.class);

If you need to set multiple headers on the request, call the `Invocation.Builder.headers` method and pass in a `javax.ws.rs.core.MultivaluedMap` instance with the name-value pairs of the HTTP headers. Calling the headers method replaces all the existing headers with the headers supplied in the MultivaluedMap instance.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    MultivaluedMap<String, Object> myHeaders = 
        new MultivaluedMap<>("myHeader", "The header value");
    myHeaders.add(...);
    String response = myResource.request(MediaType.TEXT_PLAIN)
            .headers(myHeaders)
            .get(String.class);

The MultivaluedMap interface allows you to specify multiple values for a given key.

    MultivaluedMap<String, Object> myHeaders = 
        new MultivaluedMap<String, Object>();
    List<String> values = new ArrayList<>();
    values.add(...)
    myHeaders.add("myHeader", values

#### 2. Setting Cookies in the Client Request

You can add HTTP cookies to the request by calling the Invocation.Builder.cookie method, which takes a name-value pair as parameters.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    String response = myResource.request(MediaType.TEXT_PLAIN)
            .cookie("myCookie", "The cookie value")
            .get(String.class);

The `javax.ws.rs.core.Cookie` class encapsulates the attributes of an HTTP cookie, including the name, value, path, domain, and RFC specification version of the cookie. In the following example, the Cookie object is configured with a name-value pair, a path, and a domain.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    Cookie myCookie = new Cookie("myCookie", "The cookie value", 
        "/webapi/read", "example.com"); 
    String response = myResource.request(MediaType.TEXT_PLAIN)
            .cookie(myCookie)
            .get(String.class);

#### 3. Adding Filters to the Client

You can register custom filters with the client request or the response received from the target resource. To register filter classes when the Client instance is created, call the Client.register method.

    Client client = ClientBuilder.newClient().register(MyLoggingFilter.class);

In the preceding example, all invocations that use this Client instance have the MyLoggingFilter filter registered with them.

You can also register the filter classes on the target by calling WebTarget.register.

    Client client = ClientBuilder.newClient().register(MyLoggingFilter.class);
    WebTarget target = client.target("http://example.com/webapi/secure")
            .register(MyAuthenticationFilter.class);

In the preceding example, both the MyLoggingFilter and MyAuthenticationFilter filters are attached to the invocation.

Request and response filter classes implement the `javax.ws.rs.client.ClientRequestFilter` and `javax.ws.rs.client.ClientResponseFilter` interfaces, respectively. Both of these interfaces define a single method, filter. All filters must be annotated with `javax.ws.rs.ext.Provider`.

The following class is a logging filter for both client requests and client responses.

    @Provider
    public class MyLoggingFilter implements ClientRequestFilter, 
            ClientResponseFilter {
        static final Logger logger = Logger.getLogger(...);

        // implement the ClientRequestFilter.filter method
        @Override
        public void filter(ClientRequestContext requestContext) 
                throws IOException {
            logger.log(...);
            ...
        }

        // implement the ClientResponseFilter.filter method
        @Override
        public void filter(ClientRequestContext requestContext, 
            ClientResponseContext responseContext) throws IOException {
            logger.log(...);
            ...
        }
    }

If the invocation must be stopped while the filter is active, call the context object's abortWith method, and pass in a `javax.ws.rs.core.Response` instance from within the filter.

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        ...
        Response response = new Response();
        response.status(500);
        requestContext.abortWith(response);
    }

### II. Asynchronous Invocations in the Client API

In networked applications, network issues can affect the perceived performance of the application, particularly in long-running or complicated network calls. Asynchronous processing helps prevent blocking and makes better use of an application's resources.

In the JAX-RS Client API, the Invocation.Builder.async method is used when constructing a client request to indicate that the call to the service should be performed asynchronously. An asynchronous invocation returns control to the caller immediately, with a return type of `java.util.concurrent.Future<T>` (part of the Java SE concurrency API) and with the type set to the return type of the service call. `Future<T>` objects have methods to check if the asynchronous call has been completed, to retrieve the final result, to cancel the invocation, and to check if the invocation has been cancelled.

The following example shows how to invoke an asynchronous request on a resource.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    Future<String> response = myResource.request(MediaType.TEXT_PLAIN)
            .async()
            .get(String.class);

#### 1. Using Custom Callbacks in Asynchronous Invocations

The InvocationCallback interface defines two methods, completed and failed, that are called when an asynchronous invocation either completes successfully or fails, respectively. You may register an InvocationCallback instance on your request by creating a new instance when specifying the request method.

The following example shows how to register a callback object on an asynchronous invocation.

    Client client = ClientBuilder.newClient();
    WebTarget myResource = client.target("http://example.com/webapi/read");
    Future<Customer> fCustomer = myResource.request(MediaType.TEXT_PLAIN)
            .async()
            .get(new InvocationCallback<Customer>() {
                @Override
                public void completed(Customer customer) {
                // Do something with the customer object
                }
                @Override
                public void failed(Throwable throwable) {
                // handle the error
                }
        });

# References

* [The JAX-RS specification](https://github.com/jax-rs/spec)
* [ORACLE - Accessing REST Resources with the JAX-RS Client API](https://docs.oracle.com/javaee/7/tutorial/jaxrs-client.htm)
* [Jersey User Guide - Chapter 5. Client API](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/client.html)

# Read More

* [Reactive JAX-RS Client API]()
* [JAX-RS: Advanced Topics and an Example - ORACLE](https://docs.oracle.com/javaee/7/tutorial/jaxrs-advanced.htm)
* [JAX-RS Client with Jersey - baeldung](https://www.baeldung.com/jersey-jax-rs-client)
* [REST API with Jersey and Spring - baeldung](https://www.baeldung.com/jersey-rest-api-with-spring)

#### [GO TO TOP](#jax-rs-client-api)
