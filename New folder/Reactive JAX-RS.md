# Reactive JAX-RS Client API

This article assumes the reader has knowledge of the [Jersey REST client API]().

Some familiarity with [reactive programming concepts](https://www.baeldung.com/rx-java) will be helpful but isn't necessary.

**Table of Contents:**

* [Required Dependencies](#required-maven-dependencies)
* [Why We Need Reactive JAX-RS Clients](#why-we-need-reactive-jax-rs-clients)
    1. [The Problem With Synchronous Jersey Client Invocation](#1-the-problem-with-synchronous-jersey-client-invocation)
    1. [The Problem With Asynchronous Jersey Client Invocation](#2-the-problem-with-asynchronous-jersey-client-invocation)
* [The Functional, Reactive Solution](#the-functional-reactive-solution)
    1. [CompletionStage in JAX-RS](#1-completionstage-in-jax-rs)
    1. [Observable in JAX-RS](#2-observable-in-jax-rs)
    1. [Flowable in JAX-RS](#3-flowable-in-jax-rs)
* [Reactive JAX-RS Client API Example Application](#reactive-jax-rs-client-api-example-application)
* [References](#references)
* [Read More](#read-more)

## Required Maven Dependencies

First, we need the standard Jersey client library dependencies:

    <dependency>
        <groupId>org.glassfish.jersey.core</groupId>
        <artifactId>jersey-client</artifactId>
        <version>2.27</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish.jersey.inject</groupId>
        <artifactId>jersey-hk2</artifactId>
        <version>2.27</version>
    </dependency>

These dependencies give us stock JAX-RS reactive programming support. The current versions of [jersey-client](https://search.maven.org/search?q=g:org.glassfish.jersey.core%20AND%20a:jersey-client&core=gav) and [jersey-hk2](https://search.maven.org/search?q=g:org.glassfish.jersey.inject%20AND%20a:jersey-hk2&core=gav) are available on Maven Central.

For third-party reactive framework support, we'll use these extensions:

    <dependency>
        <groupId>org.glassfish.jersey.ext.rx</groupId>
        <artifactId>jersey-rx-client-rxjava</artifactId>
        <version>2.27</version>
    </dependency>

The dependency above provides support for RxJava's [Observable](https://github.com/ReactiveX/RxJava/wiki/Observable); for the newer RxJava2's [Flowable](https://www.baeldung.com/rxjava-2-flowable), we use the following extension:

    <dependency>
        <groupId>org.glassfish.jersey.ext.rx</groupId>
        <artifactId>jersey-rx-client-rxjava2</artifactId>
        <version>2.27</version>
    </dependency>

The dependencies to [rxjava](https://search.maven.org/search?q=a:jersey-rx-client-rxjava%20AND%20g:org.glassfish.jersey.ext.rx) and [rxjava2](https://search.maven.org/search?q=a:jersey-rx-client-rxjava2%20AND%20g:org.glassfish.jersey.ext.rx) are also available on Maven Central.

## Why We Need Reactive JAX-RS Clients

Let's say we have three REST APIs to consume:

1. the id-service provides a list of Long user IDs
1. the name-service provides a username for a given user ID
1. the hash-service will return a hash of both the user ID and the username

We create a client for each of the services:

    Client client = ClientBuilder.newClient();
    WebTarget userIdService = client.target("http://localhost:8080/id-service/ids");
    WebTarget nameService = client.target("http://localhost:8080/name-service/users/{userId}/name");
    WebTarget hashService = client.target("http://localhost:8080/hash-service/{rawValue}");

The JAX-RS specification supports at least three approaches for consuming these services together:

1. Synchronous (blocking)
1. Asynchronous (non-blocking)
1. Reactive (functional, non-blocking)

### 1. The Problem With Synchronous Jersey Client Invocation

The vanilla approach to consuming these services will see us consuming the id-service to get the user IDs, and then calling the name-service and hash-service APIs sequentially for each ID returned.

With this approach, each call blocks the running thread until the request is fulfilled, spending a lot of time in total to fulfill the combined request. This is clearly less than satisfactory in any non-trivial use case.

### 2. The Problem With Asynchronous Jersey Client Invocation

A more sophisticated approach is to use the InvocationCallback mechanism supported by JAX-RS. At its most basic form, we pass a callback to the get method to define what happens when the given API call completes.

While we now get true asynchronous execution (with some limitations on thread efficiency), it's easy to see how this style of code can get unreadable and unwieldy in anything but trivial scenarios.

    final WebTarget destination = ...;
    final WebTarget forecast = ...;
    
    // Obtain recommended destinations. (does not depend on visited ones)
    destination.path("recommended").request()
        // Identify the user.
        .header("Rx-User", "Async")
        // Async invoker.
        .async()
        // Return a list of destinations.
        .get(new InvocationCallback<List<Destination>>() {
            @Override
            public void completed(final List<Destination> recommended) {
                final CountDownLatch innerLatch = new CountDownLatch(recommended.size());

                // Forecasts. (depend on recommended destinations)
                final Map<String, Forecast> forecasts =
                    Collections.synchronizedMap(new HashMap<>());

                for (final Destination dest : recommended) {
                    forecast.resolveTemplate("destination", dest.getDestination())
                        .request()
                        .async()
                        .get(new InvocationCallback<Forecast>() {
                            @Override
                            public void completed(final Forecast forecast) {
                                forecasts.put(dest.getDestination(), forecast);
                                innerLatch.countDown();
                            }

                            @Override
                            public void failed(final Throwable throwable) {
                                errors.offer("Forecast: " + throwable.getMessage());
                                innerLatch.countDown();
                            }
                        });
                }

                // Have to wait here for dependent requests ...
                try {
                    if (!innerLatch.await(10, TimeUnit.SECONDS)) {
                        errors.offer("Inner: Waiting for requests to complete has timed out.");
                    }
                } catch (final InterruptedException e) {
                    errors.offer("Inner: Waiting for requests to complete has been interrupted.");
                }

                // Continue with processing.
            }

            @Override
            public void failed(final Throwable throwable) {
                errors.offer("Recommended: " + throwable.getMessage());
            }
        });

So we've achieved asynchronous, time-efficient code, but:

* it's difficult to read
* each call spawns a new thread

## The Functional, Reactive Solution

A functional and reactive approach will give us:

* Great code readability
* Fluent coding style
* Effective thread management

JAX-RS supports these objectives in the following components:

1. `CompletionStageRxInvoker` supports the CompletionStage interface as the default reactive component
1. `RxObservableInvokerProvider` supports RxJava's Observable 
1. `RxFlowableInvokerProvider` support RxJava's Flowable

There is also an API for adding support for other Reactive libraries.

### 1. CompletionStage in JAX-RS

Using the CompletionStage and its concrete implementation – CompletableFuture – we can write an elegant, non-blocking and fluent service call orchestration.

    CompletionStage<List<Long>> userIdStage = userIdService.request()
    .accept(MediaType.APPLICATION_JSON)
    .rx()
    .get(new GenericType<List<Long>>() {
    }).exceptionally((throwable) -> {
        logger.warn("An error has occurred");
        return null;
    });

The `rx()` method call is the point from which the reactive handling kicks in. We use the exceptionally function to fluently define our exception handling scenario.

From here, we can cleanly orchestrate the calls to retrieve the username from the Name service and then hash the combination of both the name and user ID:

    List<String> expectedHashValues = ...;
    List<String> receivedHashValues = new ArrayList<>(); 
    
    // used to keep track of the progress of the subsequent calls 
    CountDownLatch completionTracker = new CountDownLatch(expectedHashValues.size()); 
    
    userIdStage.thenAcceptAsync(employeeIds -> {
    logger.info("id-service result: {}", employeeIds);
    employeeIds.forEach((Long id) -> {
        CompletableFuture completable = nameService.resolveTemplate("userId", id).request()
        .rx()
        .get(String.class)
        .toCompletableFuture();
    
        completable.thenAccept((String userName) -> {
            logger.info("name-service result: {}", userName);
            hashService.resolveTemplate("rawValue", userName + id).request()
            .rx()
            .get(String.class)
            .toCompletableFuture()
            .thenAcceptAsync(hashValue -> {
                logger.info("hash-service result: {}", hashValue);
                receivedHashValues.add(hashValue);
                completionTracker.countDown();
            }).exceptionally((throwable) -> {
                logger.warn("Hash computation failed for {}", id);
                return null;
            });
        });
    });
    });
    
    if (!completionTracker.await(10, TimeUnit.SECONDS)) {
        logger.warn("Some requests didn't complete within the timeout");
    }
    
The method `thenAcceptAsync` will execute the supplied function after the given `CompletionStage` has completed execution (or thrown an exception).

Each successive call is non-blocking, making judicious use of system resources.

The `CompletionStage` interface provides a wide variety of staging and orchestration methods that allow us to compose, order and asynchronously execute any number of steps in a multi-step orchestration (or a single service call).

### 2. Observable in JAX-RS

To use the Observable RxJava component, we must first register the `RxObservableInvokerProvider` provider (and not the “ObservableRxInvokerProvider” as is stated in the Jersey specification document) on the client:

    Client client = client.register(RxObservableInvokerProvider.class);

Then we override the default invoker:

    Observable<List<Long>> userIdObservable = userIdService
    .request()
    .rx(RxObservableInvoker.class)
    .get(new GenericType<List<Long>>(){});

From this point, we can use standard Observable semantics to orchestrate the processing flow:

    userIdObservable.subscribe((List<Long> listOfIds)-> { 
    /** define processing flow for each ID */
    });

### 3. Flowable in JAX-RS

The semantics for using RxJava Flowable is similar to that of Observable. We register the appropriate provider:

    client.register(RxFlowableInvokerProvider.class);

Then we supply the RxFlowableInvoker:

    Flowable<List<Long>> userIdFlowable = userIdService
    .request()
    .rx(RxFlowableInvoker.class)
    .get(new GenericType<List<Long>>(){});

Following that, we can use the normal Flowable API.

The CompletionStage interface, in particular, provides a robust set of methods that cover a variety of service orchestration scenarios, as well as opportunities to supply custom Executors for more fine-grained control of the thread management.

## Reactive JAX-RS Client API Example Application

### Travel Agency

The task is to create a publicly available feature that would, for an authenticated user, display a list of 10 last visited places and also display a list of 10 new recommended destinations including weather forecast and price calculations for the user. Notice that some of the requests (to retrieve data) depend on results of previous requests. E.g. getting recommended destinations depends on obtaining information about the authenticated user first. Obtaining weather forecast depends on destination information, etc.

#### Reactive Approach

Reactive approach is a way out of the so-called Callback Hell which you can encounter when dealing with Java’s `Future`s or invocation callbacks. Reactive approach is based on a data-flow concept and the execution model propagate changes through the flow. When the JAX-RS request finishes then the next item (or the user code) in the data-flow chain is notified about the continuation, completion or error in the chain. You’re more describing what should be done next than how the next action in the chain should be triggered. The other important part here is that the data-flows are composable. You can compose/transform multiple flows into the resulting one and apply more operations on the result.

    final WebTarget destination = ...;
    final WebTarget forecast = ...;
    
    // Recommended places.
    final Observable<Destination> recommended = RxObservable.from(destination)
            .path("recommended")
            .request()
            // Identify the user.
            .header("Rx-User", "RxJava")
            // Reactive invoker.
            .rx()
            // Return a list of destinations.
            .get(new GenericType<List<Destination>>() {})
            // Handle Errors.
            .onErrorReturn(throwable -> {
                errors.offer("Recommended: " + throwable.getMessage());
                return Collections.emptyList();
            })
            // Emit destinations one-by-one.
            .flatMap(Observable::from)
            // Remember emitted items for dependant requests.
            .cache();
    
    // Forecasts. (depend on recommended destinations)
    final RxWebTarget<RxObservableInvoker> rxForecast = RxObservable.from(forecast);
    final Observable<Forecast> forecasts = recommended.flatMap(destination ->
            rxForecast
                    .resolveTemplate("destination", destination.getDestination())
                    .request()
                    .rx()
                    .get(Forecast.class)
                    .onErrorReturn(throwable -> {
                        errors.offer("Forecast: " + throwable.getMessage());
                        return new Forecast(destination.getDestination(), "N/A");
                    }));
    
    final Observable<Recommendation> recommendations = Observable
            .zip(recommended, forecasts, Recommendation::new);

# References

* [Jersey User Guide - Chapter 6. Reactive JAX-RS Client API](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/rx-client.html)
* [Reactive JAX-RS Client API - baeldung](https://www.baeldung.com/jax-rs-reactive-client)

# Read More

* [Introduction to RxJava](https://www.baeldung.com/rx-java)

#### [GO TO TOP](#reactive-jax-rs-client-api)