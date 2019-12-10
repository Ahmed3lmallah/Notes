# Angular

version 1.0.0-SNAPSHOT

Angular is a typescript based framework for developing component based applications, it is NOT the same as AngularJS.

## Basics

The basic building block of an angular app is a component, which is comprised of 4 files.

*   An .html file which is commonly referred to as a template.
    
*   A .ts file which is commonly referred to as the component.
    
*   A .css (or other type of cascading style sheet file such as .scss) which is used for styling.
    
*   A .spec file which is used for testing.
    

Other notable files are module.ts and service.ts files.

## Getting Started

Angular has 2 prerequisites needed before it can be used, node.js and an npm package manager. For more thorough getting started information [Angular’s getting started page](https://angular.io/guide/quickstart)

Some knowledge of typescript, html and css is highly encouraged.

## Terms

1.  directive: [Angular’s directive description](https://angular.io/api/core/Directive#directive)
    
2.  service: [Angular’s service information](https://angular.io/tutorial/toh-pt4)
    
3.  module: [Angular’s module information](https://angular.io/guide/architecture-modules)
    
4.  router: Angular uses routing navigation to establish a single page application, [Angular’s routing information](https://angular.io/guide/router)
    
5.  bootstrap: the first component loaded by your application
    

## Commonly Used Aspects

1.  [Angular Material](https://material.angular.io/)
    
2.  [Template Driven Forms](https://angular.io/guide/forms)
    
3.  [Reactive Forms](https://angular.io/guide/reactive-forms)
    
4.  [Tables](https://material.angular.io/components/table/overview)
    

### Useful Resources

1.  [Angular’s Homepage](https://angular.io/)
    
2.  [Alligator.io extensive resource list](https://alligator.io/resources/)
    
3.  [Pluralsight’s Angular Fundamentals Course - PAID](https://app.pluralsight.com/library/courses/angular-fundamentals/table-of-contents)
    
4.  [Scrimba Angular Tutorial](https://scrimba.com/g/gyourfirstangularapp)
    
5.  [Udemy’s Angular 2 Course - PAID](https://www.udemy.com/the-complete-guide-to-angular-2/)
    
6.  [Coursera Angular Course](https://www.coursera.org/learn/angular)
    
7.  [Dzone angular getting started](https://dzone.com/articles/getting-started-with-angular-70)
    
8.  [Tutorialspoint Angular 4 tutorial](https://www.tutorialspoint.com/angular4/index.htm)
    
9.  [A youtube tutorial](https://www.youtube.com/watch?v=WWQZCDegWHg&list=PL6n9fhu94yhWqGD8BuKuX-VTKqlNBj-m6)
    
10.  [A youtube tutorial](https://www.youtube.com/watch?v=htPYk6QxacQ)
    

Version 1.0.0-SNAPSHOT  
Last updated 2019-09-24 17:36:55 UTC

# TDD - Angular 6 FAQ

version 1.0.0-SNAPSHOT

## Introduction

This page will act as a reference for writing and executing unit tests in Angular 6.

When we write unit tests in Angular, we’re actually just using the following libraries:

*   Karma - test execution (and code coverage measurement)
    
*   Jasmine - test suite organization using the `describe`, `it`, and `expect` functions
    

## FAQ / Common Problems / Gotchas

### I’m in dependency hell. How do I get out?

Or, rephrased: every time I add a service or component dependency to one of my components, _it breaks all of my tests!_

Every time you run a test, **app.module.ts is ignored.** Your spec’s `TestBed` should provide 100% of everything your component or service needs to run. So when you just added that dependency, now that dependency is missing from your spec.

For an immediate fix to the broken tests, look for the `TestBed` at the top of your spec file:

*   If you added a service dependency, it should be in the **providers** list
    
*   If you added a component dependency, it should be in the **declarations** list
    
*   If you added a module dependency, it should be in the **imports** list
    

**For a long term fix**, consider creating a [module](https://angular.io/guide/architecture-modules) for the component or service you just added a dependency to. Anyone who imports the module will be "protected" from more dependencies being added to the module, because unlike declaring a component or providing a service, importing a module means importing all its dependencies too!

So when you’re testing a component that uses another component - if the child component has a module, add the module to your spec’s import list. **Don’t** add the component to the declarations list. It already comes with the module. The same is true for services - they don’t need to be in the providers list because they’re already in the module.

You can create a module using the following command _in the directory you want the file to be created in:_

    $ ng generate module <NAME OF SERVICE OR COMPONENT HERE>

### Do I need to unit test my HTML template?

In most cases, writing tests only for your Typescript code is sufficient. If it’s not, you might consider Angular’s `e2e` feature.

If you have complex logic in your template, you’re encouraged to move as much of that logic to your class file as possible. If this is hard to do, then it’s good idea to start testing your template. Use your best judgment.

SonarQube excludes template files from coverage analysis.

### Why are there two `beforeEach` functions in my spec files?

One of your `beforeEach` functions is probably calling `compileComponents`, which is asynchronous. Asynchronous calls in a `beforeEach` require wrapping the function in an `async(…​)` call so that `beforeEach` will wait until the function is finished.

### How do I generate a code coverage file (e.g. for SonarQube)?

You need to add the `--code-coverage` parameter when you call `ng test`.

### What are spies?

Imagine you’re writing a test for a component that calls some HTTP service:

    ngOnInit() {
        this.myService.getStuff().subscribe(result => {
            this.stuff = result;
        });
    }

Our test should ensure that `getStuff()` is called, **but we don’t want to make a network request every time we run the test.** To do this, we should make sure `this.myService` is a mock/fake service when we run our test.

While we could mock with any old javascript object, Jasmine has a feature called spying that allows us to do this easily. Spies can do the following: 1. Stub a function on an existing object ("stub" = fake or useless function)

    spyOn(myObject, 'myFunctionName').and.callThrough(); // Track calls but call real code
    spyOn(myObject, 'myFunctionName2').and.returnValue(5); // Track calls, return 5
    spyOn(myObject, 'myFunctionName3').and.callFake(() => console.log("Fake function!"));
    spyOn(myObject, 'myFunctionName4').and.throwError(new Error("Uh oh"));
    spyOn(myObject, 'myFunctionName5').and.stub() // myFunctionName5() now does nothing

1.  Create a "bare stub," or a stub function from scratch. They don’t replace anything
    

const fakeFunc = jasmine.createSpy('fakeFunc');
fakeFunc(1, 2, 3, 4, 5); // nothing happens
expect(fakeFunc).toHaveBeenCalled();

1.  Track calls to a function
    

spyOn(myObject, 'doSomething').and.stub();

myObject.doSomething();
expect(myObject.doSomething).toHaveBeenCalled();

myObject.doSomething("Hello", 100);
expect(myObject.doSomething).toHaveBeenCalledWith("Hello", 100);

# Running tests

To run unit tests in Angular 6, execute the following command:

    $ ng test

Your repository’s `package.json` file may have implemented a script to do this for you. In that case, you might prefer to run:

    $ npm run test

## Etc.

### Contribution

If you see a way these pages could be improved, could do any of the following:

*   Contact the Enablement team
    
*   Raise an issue on this repository
    
*   Make a pull request with your suggested changes
    

Thank you!

### Useful Resources

1.  [https://angular.io/guide/testing](https://angular.io/guide/testing)
    
2.  [https://medium.com/frontend-fun/angular-unit-testing-jasmine-karma-step-by-step-e3376d110ab4](https://medium.com/frontend-fun/angular-unit-testing-jasmine-karma-step-by-step-e3376d110ab4)
    

Version 1.0.0-SNAPSHOT  
Last updated 2019-09-24 17:36:55 UTC

# TDD - Angular 6 Common Cases (Tutorial)

version 1.0.0-SNAPSHOT

Table of Contents

*   [Introduction](#_introduction)
*   [I want to call another component or service from my component. How do I TDD that?](#_i_want_to_call_another_component_or_service_from_my_component_how_do_i_tdd_that)
    *   [1\. Mock the Service](#_1_mock_the_service)
    *   [2\. Import the Service](#_2_import_the_service)
    *   [3\. Pull the service out of the TestBed](#_3_pull_the_service_out_of_the_testbed)
    *   [4\. Write the test](#_4_write_the_test)
    *   [5\. Write the actual function](#_5_write_the_actual_function)
*   [I want to call my API from my service and return the value. How do I TDD that?](#_i_want_to_call_my_api_from_my_service_and_return_the_value_how_do_i_tdd_that)
    *   [1\. Import HttpClientTestingModule](#_1_import_httpclienttestingmodule)
    *   [2\. Write the test](#_2_write_the_test)
    *   [3\. Write the function](#_3_write_the_function)
*   [Etc.](#_etc)
    *   [Contribution](#_contribution)
    *   [Useful Resources](#_useful_resources)

## Introduction

This page will act as a reference for some common scenarios we need to test in our Angular code.

Remember that to run a test, you might run either of the following commands:

    $ ng test
    OR
    $ npm run test

Angular looks for files that match the **\\**.spec.ts\* pattern.

If you need to generate code coverage (e.g. for SonarQube in your Jenkinsfile), consider adding the `--code-coverage` parameter to `ng test`.

## I want to call another component or service from my component. How do I TDD that?

### 1\. Mock the Service

Rather than getting Angular to construct a live instance of the other component or service, Typescript’s loose-typing allows you to create an anonymous class that will stand in for your service.

Imagine that you have a service class that looks like this:

    import { Injectable } from '@angular/core';
    import { Observable } from 'rxjs';
    import { HttpClient } from '@angular/common/http';
    
    @Injectable({
      providedIn: 'root'
    })
    export class MySpecialService {
    
      constructor(private http: HttpClient) { }
    
      getStuff(): Observable<any> {
        // does something complicated that we don't want to happen in other classes' unit tests
      }
    }

In your own component’s `spec.ts` file, you can create the anonymous class like this:

    describe('MySpecialComponent', () => {
        // ...
        let fakeReturnValue = {
          // some value that mimics the actual return type of the real MySpecialService getStuff method
        }
    
        const mockMySpecialService = {
          getStuff : jasmine.createSpy('getStuff').and.returnValue(of(fakeReturnValue))
        }
    }

Above, we create a "spy" for the `getStuff()` function. When we use `createSpy`, it creates a spy function that tracks when it’s called.

### 2\. Import the Service

In the TestBed of your component’s spec file, add the service to the providers list:

    TestBed.configureTestingModule({
        declarations: [
            // ... other component declarations
        ],
        imports: [
            // ... other module imports
            HttpClientTestingModule
        ],
        providers: [
            // ... other service providers
            { provide: MySpecialService, useValue: mockMySpecialService }  // This tells TestBed to use our mocked object whenever it constructs anything that needs a MySpecialService
        ]
    }).compileComponents();

**This makes it so that your component’s dependency on the other component or service is satisfied, without the need to create an actual instance of that component or service (along with its own dependencies).**

_In other words, a real `MySpecialService` instance is not involved in the testing of your component._

### 3\. Pull the service out of the TestBed

To use that instance in your test, you need to retrieve its reference from the TestBed. First, declare the variable in your suite so all your tests can access it:

    describe('MySpecialComponent', () => {
        // ...
        let myService: MySpecialService;
        // ...
    })

Next, assign the variable at the end of your `beforeEach`:

    beforeEach(async(() => {
        TestBed.configureTestingModule({
            imports: [...],
            ...
        }).compileComponents();
    
        myService = TestBed.get(MySpecialService);
    }));

Now all of the tests inside of `describe` can access the `myService` variable.

Getting the service instance from the TestBed ensures that we are using the same instance of `MySpecialService` that was provided to the component we are testing (e.g. the dependency that was injected)

### 4\. Write the test

In this example, we’ll assume that the service call takes place in the `ngOnInit` function.

The first line of our test is the test definition where we write a short description of what we are trying to test.

    it('should call MySpecialService.getStuff() on ngOnInit and save the results to a "stuff" variable', () => {
        // Test goes in here
    })

We can override the behavior of the spy function:

    const fakeReturnValue = { key: 'a', key2: 'b' };
    myService.getStuff.and.returnValue(of(fakeReturnValue));

There are many alternatives to `and.returnValue(…​)`, but in this example, we want to change our `getStuff` spy function to return a different dummy object.

The `of` function, imported from `rxjs`, creates an Observable with the value you pass it. This observable will resolve instantly when it’s subscribed to, so it actually behaves synchronously.

Next, we can call our `ngOnInit` function:

    component.ngOnInit();

Our component’s real function is called, but the service method is fake.

Lastly, we can actually assert on our expected behavior. Remember that all we need to verify is that our component calls the service and saves the value:

    expect(myService.getStuff).toHaveBeenCalled();
    expect(component.stuff).toBe(fakeReturnValue);

**All together:**

    it('should call MySpecialService.getStuff() on ngOnInit and save the results to a "stuff" variable', () => {
        const fakeReturnValue = { key: 'a', key2: 'b' };
        myService.getStuff.and.returnValue(of(fakeReturnValue));
        component.ngOnInit();
    
        expect(myService.getStuff).toHaveBeenCalled();
        expect(component.stuff).toBe(fakeReturnValue);
    });

Of course, when we run this test, it will fail. That’s good - now we can be confident our real function works when it’s done!

### 5\. Write the actual function

Writing the actual function turns out to be a lot simpler. We call our service and "subscribe" to the result because it’s an asynchronous call. When we get the result, we save it to a variable in the component.

    ngOnInit() {
        this.myService.getStuff().subscribe(result => {
            this.stuff = result;
        });
    }

You’ll notice that most tests follow the pattern outlined above.

1.  Sort out your dependencies in the spec
    
2.  Pull out the reference of the thing you’re mocking from the TestBed
    
3.  Mock it
    
4.  Assert that your component interacts with your mocks in the way that you expect
    

## I want to call my API from my service and return the value. How do I TDD that?

When we test API calls, we mock Angular’s `httpClient` to make dummy requests. We assert that our function calls the mock and that it behaves correctly when it gets the result.

### 1\. Import HttpClientTestingModule

At the top of your test suite definition, make sure you declare a variable to hold your `httpMock` object. This object defines how you want your dummy requests to behave.

    describe('MySpecialService', () => {
        let httpMock;
        let myService: MySpecialService;
        // ...
    })

Next, we import `HttpClientTestingModule` in the TestBed and get the reference to our `httpMock`.

    beforeEach(() => {
        TestBed.configureTestingModule({
            imports: [
                HttpClientTestingModule
            ],
            providers: [
                // ...
            ]
        });
        myService = TestBed.get(MySpecialService);
        httpMock = TestBed.get(HttpTestingController);
    });

### 2\. Write the test

The first line of a test is always the test definition. Describe the purpose of your test in a clear and easy-to-understand way.

    it('should call GET /api/stuff in getStuff() and return an Observable of the result', () => {
        // test goes in here
    });

Interestingly, **we write the expect statements first.** This is because they’re wrapped in a `subscribe(…​)` call, and the arrow function is not executed until the end of our test:

    myService.getStuff().subscribe(result => {
        expect(result.name).toBe('Bob');
        expect(result.age).toBe(100);
    });

To be clear: we’re calling `getStuff` which **does** run our service code, and the service makes the request. The network response, however, does not exist yet.

To verify the URL and method of the request, the following lines are included:

    const mockRequest = httpMock.expectOne('/api/stuff');
    expect(mockRequest.request.method).toBe('GET');

The Observable returned from `getStuff()` will resolve when `flush(…​)` is called. This simulates receiving the server’s response. **The value we put inside of `flush` is the payload of the response:**

    mockRequest.flush({
        name: 'Bob',
        age: 100
    })
    
    httpMock.verify();

**All together:**

    it('should call GET /api/stuff in getStuff() and return an Observable of the result', () => {
        myService.getStuff().subscribe(result => {
            expect(result.name).toBe('Bob');
            expect(result.age).toBe(100);
        });
    
        const mockRequest = httpMock.expectOne('/api/stuff');
        expect(mockRequest.request.method).toBe('GET');
    
        mockRequest.flush({
            name: 'Bob',
            age: 100
        })
    
        httpMock.verify();
    });

### 3\. Write the function

Angular provides an interface for us to make HTTP calls, so we need to use that:

    getStuff(): Observable<Stuff> {
        try {
            return this.http.get<Stuff>('/api/stuff');
        } catch(e) {
            console.error(e);
        }
    }

The model, request type, and URL in this function are all protected by the spec from being changed. Solid tests enable us to refactor confidently!

## Etc.

### Contribution

If you see a way these pages could be improved, could do any of the following:

*   Contact the Enablement team
    
*   Raise an issue on this repository
    
*   Make a pull request with your suggested changes
    

Thank you!

### Useful Resources

1.  [https://angular.io/guide/testing](https://angular.io/guide/testing)
    
2.  [https://medium.com/frontend-fun/angular-unit-testing-jasmine-karma-step-by-step-e3376d110ab4](https://medium.com/frontend-fun/angular-unit-testing-jasmine-karma-step-by-step-e3376d110ab4)
    

Version 1.0.0-SNAPSHOT  
Last updated 2019-09-24 17:36:55 UTC

# Angular Tips and Tricks

version 1.0.0-SNAPSHOT

## Return an Error from an Observable

This can be useful for simulating getting a 4XX or 5XX error from an http client.

    const errorResponse = Observable.create(observer => {
        observer.error(new Error());
        observer.complete();
      });
    
      myService.myMethod.and.returnValue(errorResponse);

Version 1.0.0-SNAPSHOT  
Last updated 2019-10-24 16:57:50 UTC