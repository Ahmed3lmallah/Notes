# Java TDD Java Best Practices

**Table of Contents:**

*   [Summary](#summary)
    *   [Guidelines](#guidelines)
    *   [Common Code Smells](#common-code-smells)
*   [Naming Conventions](#naming-conventions)
    *   [Physical Separation of Code](#physical-separation-of-code)
    *   [Logical Separation of Code](#logical-separation-of-code)
    *   [Separation of Implementation and Test Code](#separation-of-implementation-and-test-code)
    *   [Test Method Names](#test-method-names)
*   [Processes](#processes)
    *   [Write a test before writing the implementation code](#write-a-test-before-writing-the-implementation-code)
    *   [Only write new code when a test is failing](#only-write-new-code-when-a-test-is-failing)
    *   [All tests should pass before a new test is written](#all-tests-should-pass-before-a-new-test-is-written)
    *   [Fixing Defects, Using TDD](#fixing-defects-using-tdd)
*   [Development Practices](#development-practices)
    *   [Write the simplest code to pass the test](#write-the-simplest-code-to-pass-the-test)
    *   [Write assertions first, act later](#write-assertions-first-act-later)
    *   [Minimize assertions in each test](#minimize-assertions-in-each-test)
*   [References](#references)

> Even within the specific guidelines that are prescribed by TDD, there’s still room for differences in interpretation and confusion. That said, we have a Java "_Best Practices_" that we can follow when exercising TDD.

## Summary

_Red-Green-Refactor_: Write a **failing** test, write the **minimal** code to make the test **pass**, then **refactor** if possible.

### Guidelines

From _Test-Driven Java Development (Farcic & Garcia, 2015)_

*   Naming Conventions
    
    *   **Separate the implementation from the test code**
        
        *   **Benefits:** It avoids accidentally packaging tests together with production code
            
        
    *   **Place unit test classes in the same package as the implementation**
        
        *   **Benefits:** It helps find the test for a particular class easier and faster
            
        
    *   **Name test classes in a similar fashion to the classes they test**
        
        *   **Benefits:** It helps find the test for a particular class easier and faster
            
        
    *   **Use descriptive names for test methods**
        
        *   **Benefits:** It helps the reader understand the purpose of the test
            
        
    
*   Processes
    
    *   **Write a test before writing the implementation code**
        
        *   **Benefits:** It ensures that testable code is written; ensures that every line of code gets tests written for it
            
        
    *   **Only write new code when a test is failing**
        
        *   **Benefits:** It confirms that the test does not work without the new implementation code
            
        
    *   **Rerun all tests every time the implementation code changes**
        
        *   **Benefits:** It ensures that there is no unexpected side effects caused by code changes
            
        
    *   **All tests should pass before a new test is written**
        
        *   **Benefits:** The focus is maintained on a small unit of work; implementation code is (almost) always in working condition
            
        
    *   **Refactor only after all tests are passing**
        
        *   **Benefits:** This type of refactoring is _safe_
            
        
    *   **Use TDD for fixing defects**
        
        *   **Benefits:** Each defect found and fixed strengthens the test suite coverage and quality
            
        
    
*   Development Practices
    
    *   **Write the simplest code to pass the test**
        
        *   **Benefits:** It ensures cleaner and clearer design; it avoids unnecessary features
            
        
    *   **Write _assertions_ first, _act_ later**
        
        *   **Benefits:** This clarifies the purpose of the requirements and tests early
            
        
    *   **Minimize assertions in each test**
        
        *   **Benefits:** This avoids [assertion roulette](http://xunitpatterns.com/Assertion%20Roulette.html); allow execution of more asserts
            
        
    *   **Don’t introduce dependencies between tests**
        
        *   **Benefits:** The tests work in any order independently, whether all or only a subset is run
            
        
    *   **Tests should run fast**
        
        *   **Benefits:** These tests are used often
            
        
    *   **Use mocks**
        
        *   **Benefits:** This reduces code dependency and test execution will be faster
            
        
    *   **Use set-up and tear-down methods** (`@Before`, `@BeforeClass`, `@After`, and `@AfterClass`)
        
        *   **Benefits:** This allows set-up and tear-down code to be executed before and after the class or each test method
            
        
    *   **Do not use base test classes**
        
        *   **Benefits:** It provides test clarity
            
        
    

### Common Code Smells

There are some characteristics of Java code that are generally considered to be [code smells](https://en.wikipedia.org/wiki/Code_smell). Additionally, these code smell guidelines can generally be extended to any language.

#### Public Methods Calling Other Public Methods

This scenario is usually a symptom of one of the following problems:

*   Both the caller and callee method have some shared logic that should be extracted to a private method, which both public methods call upon
    
*   One of the methods is not actually called by any code outside the owning class, and should be `private`. In general, methods should be `private` unless they are called by external code.
    

If the `public` method being called by another `public` method has some unintended state change, or if the calling `public` method returns the other `public` method’s return value, this is also usually a code smell.

All that said, if one `public` method uses the other `public` method in an appropriate way, there’s likely not an issue. For example:

    public void isEmpty() {
        return size == 0;
    }
    
    public void peek() {
        if (isEmpty()) {
            return null;
        }
        // ...
    }

The `peek` method is using the `isEmpty` method, and they are both `public`. This is not an issue because this relationship does not fall into any of the aforementioned scenarios, e.g.

*   The `isEmpty` and `peek` methods do not have shared logic that should be extracted to a third, `private` method
    
*   Both methods are to be called by consumers of the owning class (neither methods are _unused_)
    
*   The `isEmpty` method does not cause any state change (unintended or otherwise) that is outside the scope of `peek`
    

#### Unused Private Methods

When practicing TDD, it is inherent that `private` methods are called (either directly or transitively) by `public` methods, and therefore _used_. This is because the TDD **Red** and **Green** phases focus on testing the behavior of only `public` methods, and the **Refactor** phase is where `private` methods are extracted out of working, fully-tested `public` methods.

The presence of unused `private` methods is a symptom of:

*   Building those `private` methods outside the normal TDD cycle, or
    
*   Because all the public methods that call the `private` method in question have been removed or otherwise had their logic changed
    

The former problem is solved by continually striving to use TDD.

The latter problem can be mitigated by:

1.  Comment out all `private` methods in the target class
    
2.  Run the tests for that class
    
3.  Get the `public` methods to pass their tests **without** using any `private` methods, new or old _(if you get stuck, look at the methods you commented-out, but be careful not to pull in code not needed to pass tests)_
    
4.  Once all the tests are passing again, extract bits of cohesive or duplicated code from the `public` methods and name them accordingly _(look to the methods you commented-out for method name ideas)_
    
5.  Remove all commented-out code
    

One nuance to remember with identifying unused code is that _technically_, Spring Boot framework-annotated methods (those with `@Bean`, etc.) might be marked as 'unused' by your IDE. This is because none of your own project’s code calls those methods, even though Spring Boot will call upon them at runtime.

#### Unused Public Methods

This one can be a bit trickier to identify, because there are several ways to have **false-positives** as well as **false-negatives** of unused `public` methods. One form of unused method false-positives are methods that are provided for Spring Boot to call, and require no direct testing (see the above [section](#unused-private-methods), for one example).

An example of a unused method false-negative is a `public` method that your IDE identifies as used, but upon investigation, you see that only test code calls the method. These methods are generally called 'test convenience' methods, and they should be eliminated, if possible. Following TDD’s focus on YAGNI, "_You Ain’t Gonna Need It_", methods should not be declared unless we plan to use them in production.

#### Overwriting Values

Although there will be exceptions where the practice is absolutely needed, generally values should not be overwritten once defined. Overwriting values often leads to enigmatic errors and confusion for the future readers of the code.

The way to prevent _accidental_ overwriting of values is to declare every thing `final`. This includes method parameters, local variables within methods, and even class fields (when possible)

**Note:** Class fields that are Spring Injections (services, components, etc), and not simple value objects, don’t not need to be final, _because our code does not create new instances of those objects (dependency injection)_.

## Naming Conventions

In our projects, we separate our implementation code from our test code through the use of Gradle `sourceSets`. This means that our test files are physically separated from the implementation code that they test against.

Additionally, we separate different _kinds_ of test files from each other:

*   Unit Test
    
*   Integration Tests
    
*   Smoke Tests
    
*   Acceptance Tests
    

Each set of tests should be both physically and logically separated from each other. By logically, we mean that there should be an _easy_ way to run unit tests in isolation, to run integration tests in isolation, etc.

### Physical Separation of Code

An example of the physical separation of code is here:

    project-dir
    +--- gradle  // gradle wrapper
    +--- src
    |    +--- main            // implementation code and resources go in this sourceset
    |    |    +--- java
    |    |    \--- resources
    |    +--- test            // unit test code and resources go in this sourceset
    |    |    +--- java
    |    |    \--- resources
    |    \--- itest           // integration test code and resources go in this sourceset
    |         +--- java
    |         \--- resources
    +--- .gitignore
    +--- build.gradle
    +--- gradle.properties
    +--- gradlew
    +--- gradlew.bat
    +--- README.md
    \--- settings.gradle

### Logical Separation of Code

In Gradle we get a built-in task called `test`.

Because Gradle is (rightfully) opinionated about how we should build our projects, the `test` task assumes that all the **test** code and resources are in `src/test`. Similarly, the Gradle `test` task assumes that the implementation code that our **test** code depends on is in `src/main`. We acknowledge those two assumptions in the file structure above.

You may ask, "_How does Gradle know about the `sourceset` named `itest`?_" or "_How do I run the integration tests in `src/itest` if the `test` task only runs tests contained in `src/test`?_"

The answer is: Gradle does not know nor will those **integration** (or any other types of) tests run, unless you instruct Gradle to do so.

We do this by (replace `itest` with the actual name of the directory that contains your tests):

*   Add a new `sourceset` in our Gradle project, for the location of the tests
    

    // somewhere, near the top
    sourceSets {
      itest // this name should match the name of the dir in 'src' that contains our tests
    }

*   Add to the `itestImplementation` and `itestRuntime` configurations
    

    // after the 'sourceSets' closure
    configurations {
        itestImplementation.extendsFrom implementation
        itestRuntime.extendsFrom runtime
    }

*   Make the new ``sourceSet’s classpath extend the `main`` classpath, by adding the following to your `dependencies` closure
    
*   This is necessary so that the implentation code is accessable by the test code
    
*   **Note**: This should _not_ be done if the test code does not directly operate on the implmentation code (Smoke Tests, _sometimes_ Acceptance Tests, etc.)
    

    dependencies {
        // other dependencies …
    
        itestImplementation sourceSets.main.output
        itestRuntime configurations.runtime
    }

*   Add a new test task that will be used to run this type of test
    
*   This uses the built-in `Test` task archetype, which already knows how to run JUnit tests
    
*   We just need to tell this new task where our code is
    
*   Make the Gradle `check` task also run your new test task
    
*   `check` runs the `test` task (our unit tests), and by adding the new `itest` task to it, the `check` task continues to be a good way to run all verification tasks at once
    
*   (_recommended_) Add some configurations to the `idea` plugin for Gradle so that IntelliJ IDEA recognizes this new `sourceset`
    

    // near the top, after the 'sourceSets' and 'configurations' closures
    idea {
        module {
            testSourceDirs += project.sourceSets.itest.java.srcDirs
            testSourceDirs += project.sourceSets.itest.resources.srcDirs
    
            // put our custom test dependencies onto IDEA's TEST scope
            // Ref: https://docs.gradle.org/current/dsl/org.gradle.plugins.ide.idea.model.IdeaModule.html
            scopes.TEST.plus += [configurations.itestCompileClasspath]
        }
    }

### Separation of Implementation and Test Code

Not only do we separate our tests from our implementation code and separate our tests by type, we also name and package them differently, so that we can know the kind of test a class is simply from its package and name.

#### Unit Tests

For **Unit Tests**, which are created in a 1:1 ratio of unit test class to implementation _class_, we use the name and package path of the implementation _class_ and the `Test` suffix.

For example, if we have an implementation class with a fully-qualified name of `com.cognizant.example.app.service.MyService`, we will name the unit test class `com.cognizant.example.app.service.MyServiceTest`.

#### Integration Tests

For **Integration Tests**, which are created in a 1:1 ratio of integration test class to implementation _feature_, we use the package name of our `@SpringBootApplication` class and the name of the implementation _feature_ and the `IT` suffix.

For example, if we have an implementation feature that prescribes that `Person` and `Address` entities are related, we can name the integration test class `com.cognizant.example.app.PersonAddressIT`.

#### Smoke Tests

For **Smoke Tests**, which we usually have one per application, we use the name `SmokeTest`.

### Test Method Names

Consider the following two tests:

    @RunWith(MockitoJunitRunner.class)
    public class MyServiceTest {
    
        @InjectMocks
        private MyService service;
    
        @Test
        public void testAdd() {
            final int result = service.add(1, 1);
    
            assertThat(result, equalTo(2));
        }
    
        @Test
        public void whenAddTwoAndThreeThenReturnFive() {
            final int result = service.add(2, 3);
    
            assertThat(result, equalTo(5));
        }
    }

Both tests include a simple `Act` and `Assert` phase, but the name of the second test is clearly more informative to future developers.

Notice that it uses `when` and `then`. These [two words, along with `given`](https://en.wikipedia.org/wiki/Given-When-Then) are from the practice of [Behaviour-Driven Development (BDD)](https://en.wikipedia.org/wiki/Behavior-driven_development). The three words, `Given`, `When`, and `Then` distinctly map to the `Arrange`, `Act`, and `Assert` phases of test execution.

## Processes

### Write a test before writing the implementation code

By writing tests first, we focus on what requirements are being met by the code we are writing. This is in stark contrast with writing tests after writing the implementation, because the focus there is not on requirements or features, but on simply adding coverage or regression tests.

Additionally, when we write tests first, we ensure that quality is built-in to the code.

### Only write new code when a test is failing

If a new test passes before adding any new code, then the functionality being tested has **already been created** _or_ the test is **not** written correctly.

### All tests should pass before a new test is written

Sometimes we may want to write multiple tests before beginning to write the implementation. In other cases, we may cause existing tests to fail, when introducing new functionality.

In either case, the previous tests should be **passing** before working on new test cases.

### Fixing Defects, Using TDD

While using TDD for new development tasks can be very straightforward, fixing defects in code might be more difficult to do using TDD.

However, TDD is just as important for defect resolution, if not more important. Any time you are working a defect, it is critical that you write a **failing test** that creates the defective scenario and asserts that it works correctly.

Consider the following code:

    // MyService.java
    public class MyService {
        public void doSomething(HelperService helper) {
            helper.help();
        }
    }
    
    // MyServiceTest.java
    @RunWith(MockitoJunitRunner.class)
    public class MyServiceTest {
        @InjectMocks
        private MyService service;
    
        public void givenHelperWhenDoSomethingThenHelpIsCalled() {
            final HelperService helperService = mock(HelperService.class);
    
            service.doSomething(helperService);
    
            then(helperService).should().help();
        }
    }

The above `MyServiceTest` class and asserts that whenever `MyService#doSomething()` is called, the `HelperService#help()` method is also called.

Consider that we later have a bug that the `doSomething` method is throwing a `NullPointerException` in production. We realize that this is because the `doSomething` method does not check its parameter for `null` values.

In our example, we decide that we want the `doSomething` method to do _nothing_ when a `null` is passed in. Before we fix this by adding a `null` check, we must write a test that passes in a `null` to the `doSomething` method, and expects that _no_ exception is thrown.

Here’s our updated code:

    // MyService.java
    public class MyService {
        public void doSomething(HelperService helper) {
            if (helper != null) {
                helper.help();
            }
        }
    }
    
    // MyServiceTest.java
    @RunWith(MockitoJunitRunner.class)
    public class MyServiceTest {
        @InjectMocks
        private MyService service;
    
        public void givenHelperWhenDoSomethingThenHelpIsCalled() {
            final HelperService helperService = mock(HelperService.class);
    
            service.doSomething(helperService);
    
            then(helperService).should().help();
        }
    
        public void givenNoHelperWhenDoSomethingThenNothingHappens() {
            service.doSomething(null);
        }
    }

The second test will fail if an exception is thrown.

This is a trivial example, but it shows how to use TDD to work software defects.

By using TDD to fix defects, every defect is an opportunity to build a more comprehensive test suite.

## Development Practices

### Write the simplest code to pass the test

Simpler implementations beat complicated ones, assuming that they both pass all the tests.

Unnecessary complications should be avoided.

### Write _assertions_ first, _act_ later

When you write the assertion first, the purpose or goal of the test is clear.

### Minimize assertions in each test

Consider the following three tests:

    @RunWith(MockitoJunitRunner.class)
    public class MyServiceTest {
    
        @InjectMocks
        private MyService service;
    
        @Test
        public void whenAddTwoAndThreeThenReturnFive() {
            final int result = service.add(2, 3);
    
            assertThat(result, equalTo(5));
        }
    
        @Test
        public void whenAddNegativeNumberThenExceptionIsThrown() {
            Exception exception;
            try {
                final int result = service.add(2, -3);
            } catch (Exception e) {
                exception = e;
            }
    
            assertThat("Exception was not thrown", exception, notNullValue());
            assertThat("Exception message incorrect", exception.getMessage(), equalTo("Negatives not allowed"));
        }
    
        @Test
        public void whenAddTwoNumbersThenTheRightNumberIsReturned() {
            assertThat(service.add(2, 3), equalTo(2+3));
            assertThat(service.add(3, 6), equalTo(3*6));
            assertThat(service.add(6, 2), equalTo(6+2));
            assertThat(service.add(7, 32), equalTo(7/32));
            assertThat(service.add(7, 2), equalTo(7+2));
        }
    }

The first test is simple and understandable. It contains one step in the `Act` phase and one step in the `Assert` phase

The second test has _two_ assertions, but they are both testing the same logical unit, _throwing an exception when given a negative value_. Since there are two assertions in the same test, they make use of the `reason` string in the `assertThat` method. This allows future developers to have a better idea of why the test failed, given that there are more than one assertions in the test. In the first test, the lack of a `reason` string is not a problem, because the name of the method clearly identifies what the expected behavior is.

The third test contains a sequence of assertions, and it can be difficult to really understand what is going on there. Even if a test fails, you won’t immediately know if the remaining tests will also fail or pass. The name alone does not tell you what went wrong when this test fails.

### [TDD Java](./TDD%20Java.md)

## References

*   Farcic, V., Garcia, A. (2015). _Test-Driven Java Development_. Birmingham, UK: Packt Publishing Ltd.
    
*   Gradle, Inc. (2019). _IdeaModule_. Retrieved May 1, 2019, from Gradle DSL Version 5.4.1: [https://docs.gradle.org/current/dsl/org.gradle.plugins.ide.idea.model.IdeaModule.html](https://docs.gradle.org/current/dsl/org.gradle.plugins.ide.idea.model.IdeaModule.html)
    
*   Meszaros, G. (2011, February 9). _Assertion Roulette_. Retrieved May 1, 2019, from xUnit Patterns: [http://xunitpatterns.com/Assertion%20Roulette.html](http://xunitpatterns.com/Assertion%20Roulette.html)