# DEPENDENCY INJECTION

**Table of Contents:**

* [Introduction](#introduction)
	* [What is Dependency Injection?](#what-is-dependency-injection)
		* [Limitations of Concrete Dependencies](#limitations-of-concrete-dependencies)
	* [How Dependency Injection Work?](#how-dependency-injection-work)
		* [Injection by Constructor](#1-injection-by-constructor)
		* [Injection by Setter](#2-injection-by-setter)
		* [Injection by Parameter](#3-injection-by-parameter)
		* [Injection by Reflection](#4-injection-by-reflection)
	* [Dependency Injection by Extension](#dependency-injection-by-extension)
	* [Dependency Injection with Interfaces](#dependency-injection-with-interfaces)
	* [What Not to Do](#what-not-to-do)
	* [Singleton Pattern](#singleton-pattern)
	* [Introducing Context](#introducing-context)
	* [Introducing IoC](#introducing-ioc)
		* [Dependency Injection & IoC](#dependency-injection--ioc)
* [Spring Dependency Injection](#spring-dependency-injection)
	* [What is SPRING?](#what-is-spring)
		* [Spring Beans](#spring-beans)
		* [Spring Annotations](#spring-annotations)
	* [SPRING Bean Injection](#spring-bean-injection)
	* [@Service, @Controller, and @Repository](#introducing-service-controller-and-repository)
	* [@Configuration](#configuration)
* [Angular Dependency Injection](#angular-dependency-injection)
	* [What is Angular?](#what-is-angular)
	* [Modules](#modules)
	* [Injectors](#injectors)
	* [Providers](#providers)
	* [Factories](#factories)
	
# Introduction

## WHAT IS DEPENDENCY INJECTION?

A type-safe way of supplying an external dependency to a software component.

*   When Class A uses some functionality of Class B, then it's said that Class A is dependant on Class B. Class B is a DEPENDENCY of Class A.

	*   Dependency Injection is an old concept that can be implemented with basic Java.
	*   There are multiple ways to implement Dependency Injection for a class.
	*   Dependency Injection allows you to swap dependencies in and out during runtime. Otherwise you must recompile code.
	*   Dependency Injection allows you to modify the behavior of dependant classes (usually for testing).

### LIMITATIONS OF CONCRETE DEPENDENCIES

Instantiating a dependency within a class or "hard-coding" a dependency in a class has multiple disadvantages.

*   You are now coupling one concrete class with another concrete class.
*   Changing a dependency requires recompiling the parent class instead of swapping it at runtime.
*   Difficult to test parent class

**Example of Concrete dependencies:**

	class Parent {
		public void methodThatUsesDependency(){
			Dependency dependency = new Dependency("Inside Parent");
			dependency.display();
		}
	}
	class Main {
			public static void main(String[] args){
					Parent parent = new Parent();
					parent.methodThatUsesDependency();
			}
	}

## How Dependency Injection Work?

To supply an external dependency to a class, you need a way to pass it into the Java class. This can be done in multiple ways.

1. Constructor Injection
1. Setter Injection
1. Passed as a parameter
1. Reflection

### 1. INJECTION BY CONSTRUCTOR

	class Parent {
		private Dependency dependency;
		public Parent(Dependency d){
			dependency = d;
		}
		public void methodThatUsesDependency(){
			dependency.display();
		}
	}
	class Main {
		public static void main(String[] args){
			Dependency dependency = new Dependency("Inside Main");
			Parent parent = new Parent(dependency);
			parent.methodThatUsesDependency();
		}
	}

### 2. INJECTION BY SETTER

	class Parent {
		private Dependency dependency;
		public void setDependency(Dependency d){
			dependency = d;
		}
		public void methodThatUsesDependency(){
			dependency.display();
		}
	}
	class Main {
		public static void main(String[] args){
			Dependency dependency = new Dependency("Inside Main");
			Parent parent = new Parent();
			parent.setDependency(dependency);
			parent.methodThatUsesDependency();
		}
	}

### 3. INJECTION BY PARAMETER

	class Parent {
	   public void methodThatUsesDependency(Dependency dependency){
			dependency.display();
		}
	}
	class Main {
		public static void main(String[] args){
			Dependency dependency = new Dependency("Inside Main");
			Parent parent = new Parent();
			parent.methodThatUsesDependency(dependency);
		}
	}
	
### 4. INJECTION BY REFLECTION

	class Parent {
		private Dependency dependency = new Dependency("In Parent");
		public void methodThatUsesDependency(){
			dependency.display();
		}
	}
	public class Main {
		public static void main(String[] args) throws Exception {
			Dependency dependency = new Dependency("In Main");
			Parent parent = new Parent();
			Class<?> c = parent.getClass();
			Field parentDependency = c.getDeclaredField("dependency");
			//This bypasses private access modifier
			parentDependency.setAccessible(true);
			parentDependency.set(parent, dependency);
			parent.methodThatUsesDependency();
		}
	}

## DEPENDENCY INJECTION BY EXTENSION

We can extend the class, then pass the child as the dependency, but Inheritance can bring unintended side-effects.

*   *WE DON’T RECOMMEND THIS FOR PRODUCTION! USE AN INTERFACE INSTEAD*

## DEPENDENCY INJECTION WITH INTERFACES

*   Most production code is written implementing interfaces.
	*   An interface specifies what a class must do, but not how. It lists methods that must be implemented by a class.

## WHAT NOT TO DO

### How should we not write our functions?

*   THE INFAMOUS "BIG FUNCTION":

		saveRecipe(id, recipe) {  
			 let db = postgres.open('database.com');  
			 // log the request  
			 ...  
			 // auth - is user allowed to do this?  
			 ...  
			 // call upsert recipe into the DB  
			 ...  
			 // error handling  
			 ...  
			 // send to RabbitMQ  
			 ...  
			 // finally, on line 250:  
			 db.close();  
			 return recipe;  
		}

#### Why not make one big function per operation (read, update, join…​) and call it a day?

Problems include…​

*   Bad Testability: The bigger the function, the more awkward and brittle the tests
*   Bad Modularity: How easy is it for two pairs of developers to work on closely related functionality at the same time if everything is in the same function? Not very.
*   Bad Reusability: Multiple endpoints may repeat similar behaviors but would be written from scratch each time. Client money and developer sanity are wasted.

## SINGLETON PATTERN

*   A singleton is a global, STATIC instance of a class
*   We can have a Singleton class for each feature. For example, for database code:

		constglobalDb = new CookbookDb();

		saveCookbook(...) {  
		 // do stuff with globalDb }  
		deleteCookbook(...) {  
		 // do stuff with globalDb }

But this has some problems, too.

Problems include: Singletons can be difficult to test and configure at runtime

*   Bad Testability: I want my code to use a fake CookbookDb instance when running tests. I don’t want to use the real database when testing.
*   Bad Modularity: What if I want to create a PostgresCookbookDb or a MongoCookbookDb based on some runtime configuration property?

## INTRODUCING CONTEXT

To address these problems, let’s use an abstraction that’s similar to the singleton pattern, but it’s more inclined towards runtime setup rather than a static declaration.

Context: a magical box that we can reach into at runtime and, if it exists, grab an instance of any type — including CookbookDb!

##### If you were making a rudimentary context from scratch, it could be as simple as:

	class Context {  
	 	constructor() {  
	   		this.map = {}  
	 	}  
	  
	 	get(type) {  
	   		return this.map[type];  
	 	}  
	  
	 	put(type, value) {  
	   		this.map[type] = value;  
	 	}  
	}  
	
	// setup:let context = new Context();  
	  
	if(config.mysql)  
		context.put(CookbookDb, new PostgresCookbookDb(config.mysql));  
	else
		context.put(CookbookDb, new InMemoryCookbookDb());  
	  
	// in our endpoint code:  
	let db = context.get(CookbookDb);  
	// easy to swap for a different database later!

##### In Java, generics add more complicated syntax, but it’s conceptually the same:

	class Context {  
	 	private HashMap<Class, Object> map = new HashMap<>();  
	  
		<T> T get(Class<T> clazz) {  
			return(T)map.get(clazz);  
		}  
	  	  
		<T> voidput(Class<T> valueType, T value) {  
			map.put(clazz, value);  
		}  
	}
	
### PROBLEMS WITH CONTEXT

*   One problem with the Cookbook context in the previous example is that it still could be made more modular and testable.

##### For example, everything that needs context must reference it manually:

	let cookbookDb = context.get(CookbookDb);  
	let userDb = context.get(UserDb);  
	let authService = context.get(AuthService);  
	let fluxCapacitor = context.get(FluxCapacitor);  
	...

*   If you change your context later, you need to rewrite every reference to it!
*   It’s also a little awkward to test — context must be mocked for every dependency!

## INTRODUCING IoC

*   Frameworks like Spring or Angular have considerably more complex contexts — for example, they might allow more than one instance of a certain type
*   We can think of these instances in context as dependencies in our code. For example, CookbookDb is a dependency of the business logic for saving Cookbooks.

In the previous example, our code has been calling our context API to retrieve dependencies. 

The concept of **Inversion of Control (IoC) says: instead, I’ll instead allow some framework to be in charge and call my code.**

*   Maybe my code doesn’t even need to know about context to receive its dependencies!

*   If my business logic is supplied its dependencies rather than retrieving them, how are they supplied?

### DEPENDENCY INJECTION & IoC

*   **Dependency Injection (DI) is one common form of IoC in which dependencies are passed via constructors, setters, or other service code.** Example:

		@Injectable()  
		classCookbookController {  
		 	constructor(cookbookDb, logger, authService, ...) {  
			 	this.cookbookDb = cookbookDb;  
			   	this.logger = logger;  
			   	this.authService = authService;  
			   	...  
		 }  
		  
			saveCookbook(...) {  
				// do stuff with this.cookbookDb, this.logger, this.authService  
			}  
		  
			deleteCookbook(...) {  
				// do stuff with this.cookbookDb, this.logger, this.authService  
			}  
		}

#### The magical DI framework does a couple of things:

*   Sees the `@Injectable` decorator and knows that dependencies should be injected into this class
*   Goes to the context and inserts the correct values into the constructor
*   After CookbookController has been created, it too is inserted into context — now it can be injected into other components!

# SPRING DEPENDENCY INJECTION

## WHAT IS SPRING?

*   Spring is a popular dependency injection and web framework for Java

*   Traditionally, Spring projects were mostly configured manually via XML files

*   SpringBoot is an effort by Pivotal to bring that configuration out of XML and into Java annotations

*   SpringBoot comes with "starter" build packs for web, databases, security, etc.

*   When writing tests, you should generally know how a framework works in order to effectively mock its features

### SPRING BEANS

In Spring, each dependency is unfortunately called a Bean, and they all live in ApplicationContext - Spring can inject beans into other beans at runtime

### SPRING ANNOTATIONS

You direct this injection via Spring-specific Java annotations:

*   `@Component`, `@Autowired`
*   `@Service`, `@Controller`, `@Repository`
*   `@Configuration`, `@Bean`

If you add the `@Component` annotation above a Java class, Spring will "know" about that class.

*   When your app starts, Spring will politely create an instance of that class for you
*   If your class has any dependencies, Spring will inject them in the instance (bean) for you
*   Terminology: the class is a Component, and the instance is a Bean

## SPRING BEAN INJECTION

##### Beans/Dependencies are injected via `@Autowired` annotations.

	@Component  
	class SomeClass {  
		@Autowired MyService service;  
		@Autowired MyDatabase database;  
	}

When the SomeClass bean is created at runtime, it will be provided its two dependencies only if those are also beans.

##### Dependencies may also be autowired into a class constructor.

	@Component  
	class SomeClass {  
		MyService service;  
	  
	    @Autowired  
	    public SomeClass(MyService service) {  
	        this.service = service;  
	    }  
	}

## INTRODUCING `@Service`, `@Controller`, and `@Repository`

The `@Service` annotation is an extension of `@Component`.
*   Typically, services are home to most of the business logic in your application
*   Of course, they can also autowire dependencies (and be autowired into other beans)

		@Service  
		classMyService {  
		    @Autowired Database db;  
		    @Autowired OtherService service;  
		  
		    publicintcompute() {  
		        ...  
		    }  
		}

Additional extensions of `@Component`:

*   `@Controller`: for beans that route incoming requests to business logic
*   `@Repository`: for beans that interface with data stores

##### FYI: If you `@Autowire` an interface, Spring will scan its context for services that satisfy the interface.

## CONFIGURATION

*   ##### `@Configuration` is an extension of `@Component`
    

Occasionally it’s necessary to create beans manually rather than through `@Component` et al. For example, you may want to create a bean with a third party class that doesn’t use Spring

### To do this, we use `@Configuration` beans with `@Bean` annotated methods

*   For each public method on a Configuration bean annotated with `@Bean`, Spring will call that method. Then, the return value of the method will become a bean.

##### One common case is configuring Jackson’s ObjectMapper class. Now the whole app can autowire ObjectMapper as a dependency:

	@Configuration  
	classJacksonConfig {  
	    @Bean  
	    public ObjectMapper objectMapper() {  
	        return new ObjectMapper()  
	            .setSerializationInclusion(Include.NON_NULL);  
	    }  
	  
	    @Bean  
	    public Jackson2HalModule hal() {  
	        ...  
	    }  
	}

# ANGULAR DEPENDENCY INJECTION

## WHAT IS ANGULAR?

*   Angular is a Typescript UI framework by Google with projects composed of re-usable units of logic and HTML called components.
*   Angular injects dependencies into these components via its own dependency injection system.
*   Injectable instances with business logic and without markup are called services.

## MODULES

*   Modules are used by the runtime and compiler to group components, services, and nested modules together
*   Angular generates a single module for your whole app on a fresh project

##### This is an example of an app’s root module

	@NgModule({  
		 declarations: [AppComponent],  
		 imports: [BrowserModule],  
		 providers: [],  
		 bootstrap: [AppComponent]  
	})  
	exportclass AppModule {}

In the previous example, the entire app’s functionality extends from the declarations in the AppModule:

*   Components are declared in declarations
*   Module imports are declared in imports
*   Service providers are declared in providers

In your tests, you create a custom module and injection context to provide your mock dependencies to components.
*   Modules also configure Injectors.

## INJECTORS

*   Injectors are the tool used by modules to manage dependency injection
*   Think of them as the context
*   It’s rare you need to make one yourself - the root module comes with a root injector

##### Here’s an example of creating an injectable service

	@Injectable({  
		providedIn: 'root',  
	})  
	exportclass SomeService {  
		constructor() { }  
	}

Unlike Spring, this **does not** create an instance on app-start automatically
*   For Angular to create an instance of the service for injection on app-start, it must be added to the @NgModule providers list:

		@NgModule({  
			declarations: [AppComponent],  
			imports: [BrowserModule],  
			providers: [SomeService],  
			bootstrap: [AppComponent]  
		})

## PROVIDERS

So why doesn’t Angular just ask for a list of services rather than providers?

*   Providers allow you to configure how services are created
*   It also involves some technicalities with Typescript compilation that are outside the scope of this workshop

		@NgModule({  
			 ...  
			 providers: [{ provide: SomeService, useClass: SomeService }]  
			 // equivalient to the shorthand notation:  
			 // providers: [SomeService]  
		})

*   Providers should have two keys: provide and what to use (useClass, useValue)
	*   The provide field should be an injection token
	*   Injection tokens are the key for what is being injected
		*   This can be the class itself

## FACTORIES

*   Factories are functions which take dependencies as input and output a new dependency
*   They are for the unusual case that you need to create a dependency value dynamically based on the information you don’t have until runtime

##### As an example:

	{  
	   provide: SomeService,  
	   useFactory: (svc: UserService) => new SomeService(svc.isAuthorized),  
	   deps: [UserService]  
	};