# Integration Tests

**Table of Contents:**

*   [Introduction](#introduction)
*   [Best Practices](#spring-integration-best-practices)
	*   [Summary](#summary)
	*   [Security](#security)
		*   [Controllers](#controllers)
		*   [Testing Repository Security](#testing-repository-security)
	*   [Path Mapping](#path-mapping)
		*   [Testing Controller mappings](#testing-controller-mappings)
		*   [Testing Feign Client mappings](#testing-feign-client-mappings)
	*   [Data Validation](#data-validation)
		*   [Validate Request Data, Controllers](#validate-request-data-controllers)
		*   [Validate Request Data, SDR](#validate-request-data-sdr)
	*   [Misc](#misc)
		*   [Validate Autowires](#validate-autowires)

# Introduction

## What is an integration test?

An integration test is a type of test that validates the interaction of two or more independent systems/components.

*   Unlike a unit test, you do not mock in an integration test.

Let’s say you have a team that creates an authentication system and another team that creates a banking system that uses the authentication system to determine who’s banking details to display. An integration test, in this example, would be to test that the authentication system and the banking system successfully work together by verifying that after logging in the banking information displayed belongs to the authenticated user.

## Why should you do integration tests?

The primary reason for integration tests are to expose defects in the integration of two or more systems/components. You do not want a situation like this:

![Integration Test Gif](https://media.giphy.com/media/3o7rbPDRHIHwbmcOBy/giphy.gif)

In the image above, both doors work independently (as far as being able to open and close) but together they clearly do not. An integration test, in this scenario, could have caught this snafu because you would have validated that both doors, when brought together, are spaced out enough to open and close.

In a software environment where there are many teams working on independent systems of an application, it is very easy to get fixated on the fact that your system works on its own. However, holistically it is crucial to test that these systems work together so the entire application performs as intended.

## How do you write an integration test in Spring Boot?

Below is an example of testing the interaction between a GET REST Endpoint & JPA Repository. An H2 in-memory database is used:

Necessary Dependencies in build.gradle: `build.gradle`

    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'com.h2database:h2'
    
    testImplementation('org.springframework.boot:spring-boot-starter-test')

Book Application Integration Test: `BookControllerTest.java`

    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = BookApplication.class)
    @AutoConfigureMockMvc
    public class BookControllerTest {
    
        @Autowired
        private MockMvc mockMvc;
    
        @Autowired
        private BookRepository bookRepository;
    
        @Before
        public void setup() {
            Book book = new Book();
            book.setId("1L");
            book.setTitle("A New Book Example");
            book.setAuthor("Doctor Java");
    
            bookRepository.save(book);
        }
    
        @Test
        public void getBookByIdTest_returnsBookAndOKStatus() {
            mockMvc.perform(get("/books/1"))
                    .andExpect(jsonPath("$.id").value("1"))
                    .andExpect(jsonPath("$.title").value("A New Book Example"))
                    .andExpect(jsonPath("$.author").value("Doctor Java"))
                    .andExpect(status().isOk());
        }
    }

`@SpringBootTest` - spins up the full spring application context

`@AutoConfigureMockMvc` - tells spring to inject a MockMvc & handle incoming https requests and route to your controller

# Spring Integration Best Practices

## Summary

In this '_Best Practices_' document, we will not have examples of _implementation_ code to look at. Instead, we will have examples of how to set up tests that are effective and cohesive means of testing different aspects of a Spring Boot application.

Also, even though each section contains a complete test class example, this doesn’t mean that each test necessarily has to live in its own test class. Related tests can go in the same test class (such as multiple security tests involving the same resource), since that still demonstrates test cohesiveness.

## Security

Security-focused integration tests are a great way to ensure that security holes and backdoors aren’t created (whether accidentally or intentionally). \[[1](#_footnotedef_1 "View footnote.")\]

### Controllers

The behavior of Controller methods is covered by their Unit Tests, but we still need Spring integration tests to verify users' access to those methods. \[[2](#_footnotedef_2 "View footnote.")\]

#### Return 401 When No Token

In this test, we are verifying not only that we have security "_turned on_", but also that no exceptions have been made for accessing our resource. We verify that if the request does not come from an authenticated user, we get an `Unauthorized` response. In this example, the resource is `/donuts`.

DonutSecurityIT.java

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.mockito.MockitoAnnotations;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest(DonutController.class)
    public class DonutSecurityIT {
        @MockBean
        private DonutService donutService;
    
        @Autowired
        private DonutController donutController;
    
        private MockMvc mockMvc;
    
        @Before
        public void setup() {
            MockitoAnnotations.initMocks(this);
            mockMvc = MockMvcBuilders.standaloneSetup(donutController).build();
        }
    
        @Test
        public void givenUserNotAuthenticatedWhenGetUsersThenUnauthorized() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept(MediaType.APPLICATION_JSON))
                    .andDo(print())
                    .andExpect(status().isUnauthorized());
        }
    
    }

#### Return 403 When Wrong Role

In this test, we are verifying that a user with any role other than the _valid_ role (`DONUT_CONNOISSEUR`) will receive a `Forbidden` response when attempting to access the resource.

In this example, the resource is `/donuts`, and users that are not `DONUT_CONNOISSEUR` cannot access the resource.

DonutSecurityIT.java

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.mockito.MockitoAnnotations;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.http.MediaType;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest(DonutController.class)
    public class DonutSecurityIT {
        @MockBean
        private DonutService donutService;
    
        @Autowired
        private DonutController donutController;
    
        private MockMvc mockMvc;
    
        @Before
        public void setup() {
            MockitoAnnotations.initMocks(this);
            mockMvc = MockMvcBuilders.standaloneSetup(donutController).build();
        }
    
        @Test
        @WithMockUser(roles = {"DONUT_CHEF", "DONUT_ADMIRER"})
        public void givenUserIsNotDonutConnoisseurWhenGetDonutsThenForbidden() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept(MediaType.APPLICATION_JSON))
                    .andDo(print())
                    .andExpect(status().isForbidden());
        }
    
    }

#### Return 2XX When Correct Role

In this test, we are verifying that a user with the _valid_ role will receive a successful response that is appropriate for the request being invoked.

In this case, the successful response to a `GET` is `OK`, the resource is `/donuts`, and users that are `DONUT_CONNOISSEUR` can access the resource.

DonutSecurityIT.java

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.mockito.MockitoAnnotations;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.http.MediaType;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest(DonutController.class)
    public class DonutSecurityIT {
        @MockBean
        private DonutService donutService;
    
        @Autowired
        private DonutController donutController;
    
        private MockMvc mockMvc;
    
        @Before
        public void setup() {
            MockitoAnnotations.initMocks(this);
            mockMvc = MockMvcBuilders.standaloneSetup(donutController).build();
        }
    
        @Test
        @WithMockUser(roles = "DONUT_CONNOISSEUR")
        public void givenUserIsDonutConnoisseurWhenGetDonutsThenOk() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept(MediaType.APPLICATION_JSON))
                    .andDo(print())
                    .andExpect(status().isOk());
        }
    
    }

### Testing Repository Security

The need for testing exists even at the CRUD repository level; we can’t just add security to our FEMS (front-end microservices) and hope for the best.

In this case the resource is `/donuts`, and users that are `DONUT_CONNOISSEUR` can access the resource. All other users should not be able to get donuts _(try some veggies?)_.

We write these tests at the `MockMvc` and REST level, instead of autowiring the `Repository` class and testing that. \[[3](#_footnotedef_3 "View footnote.")\]

DonutSecurityIT.java

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @AutoConfigureMockMvc
    @SpringBootTest
    public class DonutSecurityIT {
    
        @Autowired
        MockMvc mockMvc;
    
        @Test
        public void givenUserNotAuthenticatedWhenGetUsersThenUnauthorized() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept("application/json"))
                    .andExpect(status().isUnauthorized());
        }
    
        @Test
        @WithMockUser(roles = {"DONUT_CHEF", "DONUT_ADMIRER"})
        public void givenUserIsNotDonutConnoisseurWhenGetDonutsThenForbidden() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept("application/json"))
                    .andExpect(status().isForbidden());
        }
    
        @Test
        @WithMockUser(roles = "DONUT_CONNOISSEUR")
        public void givenUserIsDonutConnoisseurWhenGetDonutsThenOk() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept("application/json"))
                    .andExpect(status().isOk());
        }
    
    }

Before tests can properly validate the role-based security in a SDR (Spring Data REST) project, you must add the following configuration file to your **test classes** (`src/test/java`), in the package or child-package of your `@SpringApplication`:

TestConfiguration.java

    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
    import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
    
    @Configuration
    public class TestConfiguration extends ResourceServerConfigurerAdapter {
        @Override
        public void configure(ResourceServerSecurityConfigurer security) {
            security.stateless(false);
        }
    }

Without this, all your SDR endpoints will return `Unauthorized` during the integration tests in the previous example.

## Path Mapping

### Testing Controller mappings

The behavior of Controller methods is covered by their Unit Tests, but we still need Spring integration tests to verify that we have properly mapped REST requests to those methods.

DonutControllerIT.java

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import static org.mockito.Mockito.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest
    public class DonutControllerIT {
    
        @MockBean
        private DonutController DonutController;
    
        @Autowired
        private MockMvc mockMvc;
    
        @Test
        public void getDonutsIsMapped() throws Exception {
            mockMvc.perform(get("/donuts")
                    .accept(MediaType.APPLICATION_JSON));
    
            verify(DonutController).getDonuts();
        }
    
    }

**Note:** If you are using security (_why not…​?_), this test will not work without the inclusion of an `@WithMockUser`.

### Testing Feign Client mappings

We don’t test that Feign _itself_ works, but that we are _using_ Feign **correctly**.

DonutClientIT.java

    import com.fasterxml.jackson.databind.ObjectMapper;
    import com.github.tomakehurst.wiremock.junit.WireMockRule;
    import com.netflix.loadbalancer.Server;
    import com.netflix.loadbalancer.ServerList;
    import org.junit.ClassRule;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.context.TestConfiguration;
    import org.springframework.cloud.netflix.ribbon.StaticServerList;
    import org.springframework.context.annotation.Bean;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.net.URL;
    import java.nio.charset.StandardCharsets;
    import java.nio.file.Files;
    import java.nio.file.Paths;
    import java.util.stream.Collectors;
    
    import static com.github.tomakehurst.wiremock.client.WireMock.*;
    import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.wireMockConfig;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class DonutClientIT {
    
        @ClassRule
        public static WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort());
    
        @Autowired
        private DonutClient client;
    
        @Autowired
        private ObjectMapper mapper;
    
        @Test
        public void createDonutMakesPostRequestWithBearerToken() throws IOException, URISyntaxException {
            final URL resource = Thread.currentThread().getContextClassLoader().getResource("create_donut_response.json");
            final String donutResponseJson = Files.lines(Paths.get(resource.toURI()), StandardCharsets.UTF_8)
                    .collect(Collectors.joining(System.lineSeparator()));
            final String expectedBearerToken = "abcd";
            final Donut donut = new Donut();
    
            stubFor(post(urlMatching("/donuts"))
                    .withHeader("Authorization", matching(expectedBearerToken))
                    .withRequestBody(equalTo(mapper.writeValueAsString(donut)))
                    .willReturn(aResponse()
                            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                            .withBody(donutResponseJson)
                            .withStatus(HttpStatus.CREATED_201)
                    ));
    
            client.createDonut(expectedBearerToken, donut);
    
            wireMockRule.verify(postRequestedFor(urlMatching("/donuts")));
        }
    
        @Test
        public void getDonutsMakesGetRequestWithBearerToken() throws IOException, URISyntaxException {
            final URL resource = Thread.currentThread().getContextClassLoader().getResource("get_donuts_response.json");
            final String donutsResponseJson = Files.lines(Paths.get(resource.toURI()), StandardCharsets.UTF_8)
                    .collect(Collectors.joining(System.lineSeparator()));
            final String expectedBearerToken = "abcd";
    
            stubFor(get(urlMatching("/donuts"))
                    .withHeader("Authorization", matching(expectedBearerToken))
                    .willReturn(aResponse()
                            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                            .withBody(donutsResponseJson)));
    
            client.getDonuts(expectedBearerToken);
    
            wireMockRule.verify(getRequestedFor(urlMatching("/donuts")));
        }
    
        @TestConfiguration
        public static class LocalRibbonClientConfiguration {
            @Bean
            public ServerList<Server> ribbonServerList() {
                return new StaticServerList<>(new Server("localhost", wireMockRule.port()));
            }
        }
    }

These tests require us to have two JSON files, which each represent a **real** response from the server that our Feign client should be communicating with. They should be located in our `test/resources` directory.

create\_donut\_response.json

    {
      "name": "Sprinkle Donut",
      "description": "Tasty and sweet, maybe too sweet...",
      "_links": {
        "self": {
          "href": "https://donuts.cognizant.com/donuts/1"
        },
        "donut": {
          "href": "https://donuts.cognizant.com/donuts/1"
        }
      }
    }

get\_donuts\_response.json

    {
      "_embedded": {
        "donuts": [
          {
            "name": "Sprinkle Donut",
            "description": "Tasty and sweet, maybe too sweet...",
            "_links": {
              "self": {
                "href": "https://donuts.cognizant.com/donuts/1"
              },
              "donut": {
                "href": "https://donuts.cognizant.com/donuts/1"
              }
            }
          },
          {
            "name": "Frosted Donut",
            "description": "Classic",
            "_links": {
              "self": {
                "href": "https://donuts.cognizant.com/donuts/2"
              },
              "donut": {
                "href": "https://donuts.cognizant.com/donuts/2"
              }
            }
          },
          {
            "name": "Chocolate Glazed Donut",
            "description": "Classic, but with chocolate",
            "_links": {
              "self": {
                "href": "https://donuts.cognizant.com/donuts/3"
              },
              "donut": {
                "href": "https://donuts.cognizant.com/donuts/3"
              }
            }
          },
          {
            "name": "Vanilla Creme-Filled Donut",
            "description": "Tasty, though very messy",
            "_links": {
              "self": {
                "href": "https://donuts.cognizant.com/donuts/4"
              },
              "donut": {
                "href": "https://donuts.cognizant.com/donuts/4"
              }
            }
          }
        ]
      },
      "_links": {
        "self": {
          "href": "https://donuts.cognizant.com/donuts"
        },
        "profile": {
          "href": "https://donuts.cognizant.com/profile/donuts"
        }
      }
    }

## Data Validation

### Validate Request Data, Controllers

Our Controller class unit tests can do a great job verifying that only requests with valid data are processed.

However, just like the [Validate Request Data, SDR](#_validate_request_data_sdr) section, below, it is still important to verify that bad requests are handled correctly at the _REST level_; they should result in a `Bad Request` response, and not contain stack trace information about an unintended side-effect of the bad request.

DonutIT.java

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.mockito.MockitoAnnotations;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.http.MediaType;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    
    import static org.mockito.BDDMockito.then;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest(DonutController.class)
    public class DonutIT {
    
        @MockBean
        private DonutService donutService;
    
        @Autowired
        private DonutController donutController;
    
        private MockMvc mockMvc;
    
        @Before
        public void setup() {
            MockitoAnnotations.initMocks(this);
            mockMvc = MockMvcBuilders.standaloneSetup(donutController).build();
        }
    
        @Test
        @WithMockUser(roles = {"DONUT_CONNOISSEUR"})
        public void createDonut_missingName_shouldReturn400BadRequest() throws Exception {
            mockMvc.perform(post("/donuts")
                    .header("Authorization", "Bearer asdfasf")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content("{\"description\": \"Mystery Donut\"}"))
                    .andDo(print())
                    .andExpect(status().isBadRequest());
        }
    
    }

### Validate Request Data, SDR

Just as with the [Validate Request Data, Controllers](#_validate_request_data_controllers) section, above, it isn’t enough to simply _stop_ bad requests from being processed; we must ensure the generated response reflects a `Bad Request`.

DonutIT.java

    import com.fasterxml.jackson.databind.ObjectMapper;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.http.MediaType;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import java.util.ArrayList;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @SpringBootTest
    @RunWith(SpringRunner.class)
    @AutoConfigureMockMvc
    public class DonutIT {
    
        @Autowired
        private MockMvc mockMvc;
    
        @Autowired
        private ObjectMapper mapper;
    
        @Test
        @WithMockUser(roles = "DONUT_CONNOISSEUR")
        public void nameCantBeEmpty() throws Exception {
            final Donut donut = new Donut();
            donut.setDescription("Another Mystery Donut");
    
            final String donutJson = mapper.writeValueAsString(donut);
    
            mockMvc.perform(post("/donuts")
                    .contentType(MediaType.APPLICATION_PROBLEM_JSON)
                    .content(donutJson))
                    .andExpect(status().isBadRequest());
        }
    
    }

## Misc

### Validate Autowires

We use TDD for everything that’s reasonably possible, but do we use TDD for applying `@Autowire` annotations to our dependencies? _We should!_ \[[4](#_footnotedef_4 "View footnote.")\]

The simplest way to validate that our dependencies are autowired into a target component is to write a test that will fail with a `NullPointerException` unless the dependency is autowired in.

As with any good, non-brittle, test, we don’t actually care if the target class has `@Autowired` on individual fields, `@Autowired` on an all-args constructor, or really if it has `@Autowired` at all.

All we really care about is that the target classes dependencies are injected and not created internally.

DonutControllerIT.java

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.hateoas.Resources;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import static org.mockito.ArgumentMatchers.anyString;
    import static org.mockito.BDDMockito.given;
    import static org.mockito.BDDMockito.then;
    import static org.mockito.Mockito.mock;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest(DonutController.class)
    public class DonutControllerIT {
    
        @Autowired
        private DonutController controller;
    
        @MockBean
        private DonutService service;
    
        @Test
        public void donutServiceIsAutowiredIntoController() {
            given(service.getDonuts(anyString())).willReturn(mock(Resources.class));
            final String bearerToken = "abcd123";
    
            controller.getDonuts(bearerToken);
    
            then(service).should().getDonuts(bearerToken);
        }
    }

The above example ensures that our `DonutController` is getting its dependency, `DonutService`, through _dependency-injection_ (and not creating a `DonutService` instance internally).

This pattern can be used to ensure that our `DonutService` is getting its `DonutClient` dependency through dependency-injection, as well.

* * *

[1](#_footnoteref_1). Built-in security integration tests are faster and easier to maintain than external penetration tests

[2](#_footnoteref_2). See [TDD Java Best Practices](TDD%20Best%20Practices.md)

[3](#_footnoteref_3). Security at the `Repository` level is far less important than at the REST level. Testing security RESTfully creates less coupling to the internal structure of the microservice

[4](#_footnoteref_4). _"If it’s worth building, it’s worth testing. If it’s not worth testing, why are you wasting your time working on it?"_ — Scott Ambler, Enterprise Agile Coach