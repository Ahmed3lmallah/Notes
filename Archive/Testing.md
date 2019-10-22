# Test Driven Development

### Table of Contents

* [Testing the Controller Layer](#testing-the-controller-layer)
* [Testing the Dao Layer](#testing-the-dao-layer)
* [Testing the Service Layer](#testing-the-service-layer) - [Mocks](#mocks)

### Introduction

**Different types of Testing:**
* Unit Tests
* Integration Tests
* System Tests
* Performance Tests
* User Acceptance Testing
* Maybe add security/penetration testing?

**Unit Testing:**

* Solitary testing: for that we need test doubles to simulate outside dependencies [Test doubles: Mocks [OR] Stubs]
* Sociable testing: for that we can involve outside dependencies in testing our code - at this point, this actually turns into integration testing.

**Test doubles:**

* Stubs: we code simple classes that acts like the dependencies we're trying to simulate. An example would be a class called "SomeServiceStub" that have methods that take test inputs and return the expected output using stubMethods. An obvious disadvantage is the stub being static - for each method we've limited test cases (we might endup needing to code many stubMethods). And, it's an extra code that we need manage.

* Mocks: are more dynamic. They use reflection to get information about the class we're trying to simulate and allow us to modify the behavior of that class methods to return the expected output.

*Any time we see someMethod(ClassName.class)* < **Reflection:** is an API which is used to examine or modify the behavior of methods, classes, interfaces at runtime.

## Testing the Controller Layer

1. There is some MockMvc setup and configuration required:
		
		@RunWith(SpringRunner.class)
		@WebMvcTest(RsvpController.class)
		public class RsvpControllerTest {
			.
			.

	1. Wire in the MockMvc object
	
			@Autowired
			private MockMvc mockMvc;
	
	1. Expose dependencies as `@MockBeans`
	
			@MockBean
			private RsvpDao dao;

	1. Instantiate an ObjectMapper to convert Java objects to JSON and vice versa
	
			private ObjectMapper mapper = new ObjectMapper();
			
		* To convert objects to Json Strings use the following inside the test:
		
				String outputJson = mapper.writeValueAsString(rsvp);

1. Look at the functionality of each endpoint and test for the following:

	*Using MockMvc to test controllers is similar to testing any component - we follow the Arrange-Act-Assert pattern*
	
	1. Happy Path - test that both the correct content and the correct HTTP status code are returned when valid input is given.
		
		* For GET requests:
		
				//Arrange
				.
				.

				//Objects to Json
				.
				.
				
				//Dao Mock
				doReturn(rsvpOut).when(dao).getRsvp(1);

				String response =  this.mock.perform(get("/rsvps/1")) 
						.andDo(print())
						.andExpect(status().isOk())
						.andExpect(content().json(responseJson))
				//      .andExpect(jsonPath("$.guestName", Matchers.is("Ahmed")))
				//      .andExpect(jsonPath("$.totalAttending", Matchers.is(15)))
						.andReturn().getResponse().getContentAsString();
				
				assertEquals(response, responseJson); <- This is Reduntant
			
		* In case of Requests with RequestBody, POST & PUT:
		
				this.mockMvc.perform(MockMvcRequestBuilders.post("/rsvps")
				.content(inputJson).contentType(MediaType.APPLICATION_JSON))
				.andDo(print())
				.andExpect(status().isCreated())
				.andExpect(content().json(mapper.writeValueAsString(outputRsvp)));
			
		* For lists, GET all:
		
				//Arrange
				List<Rsvp> listChecker = new ArrayList<>();
				listChecker.addAll(rsvpList);
				
				//To confirm the test is testing what we think it is... add another Rsvp.
				// Uncommenting the below line causes the test to fail. As expected!
				// listChecker.add(new Rsvp(10, "Donald Duck", 2));
				
				//List to Json
				String outputJson = mapper.writeValueAsString(listChecker);
				
				this.mockMvc.perform(get("/rsvps")).andDo(print()).andExpect(status().isOk())
						.andExpect(content()
								.json(outputJson));
							
		* When no content is expected, such as for DELETE:
		
				this.mockMvc.perform(MockMvcRequestBuilders.delete("/rsvps/8"))
					.andDo(print()).andExpect(status().isOk())
					.andExpect(content().string(""));
			
	1. Error Conditions - test that the correct HTTP status codes and error content are returned when invalid input is given.
	
			@Test
			public void getRsvpThatDoesNotExistReturns404() throws Exception {
				//no additional mocking necessary. 
				//By default, the mock will return null when it's not set up.
				//regardless, let's be explicit!
				int idForRsvpThatDoesNotExist = 100;
				
				//Dao Mock
				when(dao.getRsvp(idForRsvpThatDoesNotExist)).thenReturn(null);
				
				this.mockMvc.perform(get("/rsvps/" + idForRsvpThatDoesNotExist))
				.andDo(print())
				.andExpect(status().isNotFound());
			}
			
## Testing the Dao Layer

1. From the interface file, auto generate a test for all methods and include a `@Before` method
1. Add class-level annotations:
	1. `@RunWith(SpringJUnit4ClassRunner.class)`
	1. `@SpringBootTest`
1. Use `@Autowired` to connect the test with DAO: An example of Dependency Injection
		
		@Autowired
		protected CarLotDao dao;

1. Add method-level annotations: `@Before`, `@BeforeClass`, and `@Test`
1. Design a test implementation for each DAO method using AAA approach.

**Example:**
		
	@Before
	public void setUp() throws Exception {
		// clean out the test db
		List<Car> carList = dao.getAllCars();
		for (Car t : carList) {
			dao.deleteCar(t.getId());
		}
	}

	@Test
	public void addGetDeleteACar() {

		//Arrange
		.
		.
		
		//Act
		car = dao.addCar(car);

		Car car2 = dao.getCar(car.getId());

		//Assert
		assertEquals(car, car2);
	}

## Testing the Service Layer

1. No Class-level annotations required for testing the service layer.
1. Declare dependencies, call [setup methods for Mocks](#mocks) in the `@Before`, then inject mocks in service layer using the constructor:

		ServiceLayer service;
		AlbumDao albumDao;
		.
		.

		@Before
		public void setUp() throws Exception {
			setUpAlbumDaoMock();
			.
			.

			service = new ServiceLayer(albumDao, artistDao, labelDao, trackDao);
		}
		
1. Follow the Arrange, Act, Assert format to write tests.

		@Test
		public void saveAlbum() {
		
			// Arrange
			.
			.

			// Act
			AlbumViewModel fromService = service.saveAlbum(avm); 
				
			// Assert
			assertEquals(fromArrange, fromService);
		}
		
#### Alternative approach

1. Add `@RunWith(MockitoJUnitRunner.class)` to the class
1. Mark dependencies with `@Mock`
1. Continue as usual...
		
### Mocks 

#### How to mock the DAO?

1. Declare the mock `albumDao = mock(AlbumDaoJdbcTemplateImpl.class);` 
1. Hard-code the **output DTO**
1. Hard-code the **input DTO**
1. Let the mock know to return **output DTO** when **input DTO** is passed to the **DAO** 

	Final code:

		private void setUpAlbumDaoMock() {
			
			albumDao = mock(AlbumDaoJdbcTemplateImpl.class);
			
			// Output
			.
			.
			
			// Input
			.
			.
			
			doReturn(Output).when(albumDao).addAlbum(Input);
		}
		

#### How to mock an external microservice?

* Case (1): An external service that is called using RestTemplate and DiscoveryClient

	First, the setUp:
	
		private TaskerServiceLayer service;
		private TaskerDao taskerDao;
		private RestTemplate restTemplate;
		private DiscoveryClient discoveryClient;

		@Value("${adServerServiceName}")
		private String adServerServiceName;

		@Value("${serviceProtocol}")
		private String serviceProtocol;

		@Value("${servicePath}")
		private String servicePath;

		@Before
		public void setUp() throws Exception {
			
			// We call the mock methods here
			setUpTaskerDaoMock();
			setUpRestTemplateMock();
			setUpDiscoveryClientMock();

			// We use a custome made service layer constructor to pass on the mocked classes and properties
			service = new TaskerServiceLayer(taskerDao, discoveryClient, restTemplate, adServerServiceName, serviceProtocol, servicePath);

		}
		
	Second, we mock the RestTemplate:
	
		private void setUpRestTemplateMock() {
		
			restTemplate = mock(RestTemplate.class);

			doReturn("BOGO large 2 topping pizzas!").when(restTemplate).getForObject("http://localhost:6107/ad", String.class);
		}

	Finally, we mock the discoveryClient:
	
		private void setUpDiscoveryClientMock() {
			
			discoveryClient = mock(DiscoveryClient.class);
			
			// discoveryClient returns a LinkedList of DefaultServiceInstances with hostName and portNumber
			List<ServiceInstance> instances = new LinkedList<>();

			DefaultServiceInstance defaultServiceInstance = new DefaultServiceInstance("","","localhost",6107,true);

			instances.add(defaultServiceInstance);

			doReturn(instances).when(discoveryClient).getInstances("adserver-service");
		}

* Case (2): An external service that is called using Feign Client

		private void setUpLevelUpClientMock(){
			
			levelUpClient = mock(LevelUpClient.class);

			// output
			.
			.

			doReturn(output).when(feignClient).getMethod(1);
		}
	
* Case (3): A Queue service

		private void setUpRabbitTemplateMock(){
			rabbitTemplate = mock(RabbitTemplate.class);
		}
		
