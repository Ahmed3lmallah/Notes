# Mockito

## Verifying if methods were called

### `.verify` method

	//Alternative way to Verify die.roll() was called 1 time  
	Mockito.verify(die).roll();

	Mockito.verify(die, atLeast(1)).roll(); 

	//Verify die.roll() was never called  
	Mockito.verify(die, times(0)).roll();
	
	Mockito.verify(die, never).roll();  

### Another way - Similar to `.verify`

	then(serviceMock).should().method("");

	then(serviceMock).should(times(2)).method("");

	then(serviceMock).should(never()).method("");

## Capturing arguments

	//Declare argument
	ArgumentCaptor<String> stringArgumentCaptor = ArgumentCaptor.forClass(String.class);

	//Define Argument captor on specific method call

	then(serviceMock).should().method(stringArgumentCaptor.capture());

	//Capture the argument
	assertThat(stringArgumentCaptor.getValue(),is("someThing"));

	//if method is called more than once
	//Capture the argument
	assertThat(stringArgumentCaptor.getAllValues().size(),is(2));

## Mockito Junit Rules

	@Rule
	public MockitoRule mockitoRule = MockitoJUnit.rule();

Equivalent to `@RunWith(MockitoJUnitRunner.class)` which allows us to use other runners such as spring unit test.

## Mockito Spy

Mockito spies are rarely used. Mocks doesn't care about logic in the actual class since calling mock methods wouldn't affect class properties. They will always return default values when a method is mocked. However, to get class logic we can use spies.

	List arrayListSpy = spy(ArrayList.class);

This allows us to overwrite specific methods:

	stub(arrayListSpy.size()).toReturn(5);

To spy on logic:

	verify(arrayListSpy, never()).clear();


## PowerMock

Mockito doesn't allow us to mock private, static, or final classes. `PowerMock` is used to mock static, private, and constructor methods.

**Dependencies:**

		<dependency>
			<groupId>org.powermock</groupId>
			<artifactId>powermock-api-mockito</artifactId>
			<version>1.6.4</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.powermock</groupId>
			<artifactId>powermock-module-junit4</artifactId>
			<version>1.6.4</version>
			<scope>test</scope>
		</dependency>

### Mocking Static Methods

	@RunWith(PowerMockRunner.class)
	@PrepareForTest({ UtilityClass.class})

	PowerMockito.mockStatic(UtilityClass.class);
	when(UtilityClass.staticMethod(anyLong())).thenReturn(150);

	PowerMockito.verifyStatic();
	UtilityClass.staticMethod(1 + 2 + 3);

### Mocking Private Methods

	long value = (Long) Whitebox.invokeMethod(systemUnderTest, "privateMethodUnderTest");

### Mocking a constructor

	@PrepareForTest(SystemUnderTest.class) // the class where the constructor is called.

	PowerMockito.whenNew(ArrayList.class).withAnyArguments().thenReturn( mockList);

## Exercises and References

* [Unit Tests Are FIRST](https://pragprog.com/magazines/2012-01/unit-tests-are-first)
* [xUnit Patterns](http://xunitpatterns.com)
* [How to write good tests](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)