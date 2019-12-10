# Java Tips and Tricks

Table of Contents

*   [Core Java](#core-java)
*   [Spring Boot](#spring-boot)
    *   [Extracting Entity IDs from REST/HATEOAS](#extracting-entity-ids-from-resthateoas)
    *   [Linking Entities Using Feign Client](#linking-entities-using-feign-client)
    *   [Returning Correct HTTP Error Responses](#returning-correct-http-error-responses)

## Core Java

## Spring Boot

### Extracting Entity IDs from REST/HATEOAS

**Scenario:** My UI would like entity `id` values, but my CRUD backend is using REST/HATEOAS, so it is not returning `id` values for the entities.

How can we have `id` values provided to our UI, without changing how our CRUD works?

By having [Jackson](https://github.com/FasterXML/jackson-databind) unpack the `id` values from the HATEOAS data we are already receiving.

The full solution involves some Unit Test-level pieces, as well as some Spring-level pieces (Jackson)

#### Unit Test

PersonTest.java

    import com.cognizant.example.Person;
    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.mockito.junit.MockitoJUnitRunner;
    
    import java.util.HashMap;
    import java.util.Map;
    
    import static org.hamcrest.MatcherAssert.assertThat;
    import static org.hamcrest.core.IsEqual.equalTo;
    
    @RunWith(MockitoJUnitRunner.class)
    public class PersonTest {
    private Person person;
    
        @Before
        public void setup() {
            person = new Person();
        }
    
        @Test
        public void givenLinksWhenUnpackIdFromLinksThenIdIsSet() {
            final Map<String, Object> links = new HashMap() {{
                final Map<String, String> self = new HashMap() {{
                    put("href", "/uri/blah/with/blah/id/123");
                }};
                put("self", self);
            }};
    
            person.unpackIdFromLinks(links);
    
            assertThat(person.getId(), equalTo(123L));
        }
    }

#### Unit Test Solution

Person.java

    import lombok.Data;
    
    import java.util.List;
    import java.util.Map;
    
    @Data
    public class Person {
        private Long id;
        private String name;
    
        public void unpackIdFromLinks(final Map<String, Object> links) {
            final String uri = ((Map<String, String>) links.get("self")).get("href");
            final int idStartIndex = uri.lastIndexOf('/') + 1;
            final String idStr = uri.substring(idStartIndex);
            this.id = Long.parseLong(idStr);
        }
    }

#### Integration (Feign) Test

get_people_response.json

      {
      "_embedded": {
        "people": [
          {
            "name": "Jane Doe",
            "_links": {
              "self": {
                "href": "https://example-person-crud.cfapps.io/people/1"
              },
              "person": {
                "href": "https://example-person-crud.cfapps.io/people/1"
              },
              "address": {
                "href": "https://example-person-crud.cfapps.io/people/1/address"
              }
            }
          },
          {
            "name": "John Smith",
            "_links": {
              "self": {
                "href": "https://example-person-crud.cfapps.io/people/2"
              },
              "person": {
                "href": "https://example-person-crud.cfapps.io/people/2"
              },
              "address": {
                "href": "https://example-person-crud.cfapps.io/people/2/address"
              }
            }
          },
          {
            "name": "John Doe",
            "_links": {
              "self": {
                "href": "https://example-person-crud.cfapps.io/people/3"
              },
              "person": {
                "href": "https://example-person-crud.cfapps.io/people/3"
              },
              "address": {
                "href": "https://example-person-crud.cfapps.io/people/3/address"
              }
            }
          },
          {
            "name": "Jane Smith",
            "_links": {
              "self": {
                "href": "https://example-person-crud.cfapps.io/people/4"
              },
              "person": {
                "href": "https://example-person-crud.cfapps.io/people/4"
              },
              "address": {
                "href": "https://example-person-crud.cfapps.io/people/4/address"
              }
            }
          }
        ]
      },
      "_links": {
        "self": {
          "href": "https://example-person-crud.cfapps.io/people"
        },
        "profile": {
          "href": "https://example-person-crud.cfapps.io/profile/people"
        },
        "search": {
          "href": "https://example-person-crud.cfapps.io/people/search"
        }
      }
    }
    

PersonClientIT.java

    import com.cognizant.example.Person;
    import com.github.tomakehurst.wiremock.junit.WireMockRule;
    import com.netflix.loadbalancer.Server;
    import com.netflix.loadbalancer.ServerList;
    import org.hamcrest.core.IsEqual;
    import org.junit.ClassRule;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.context.TestConfiguration;
    import org.springframework.cloud.netflix.ribbon.StaticServerList;
    import org.springframework.context.annotation.Bean;
    import org.springframework.hateoas.Resources;
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
    import static org.hamcrest.MatcherAssert.assertThat;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class PersonClientIT {
    
        @ClassRule
        public static WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort());
    
        @Autowired
        private PersonClient client;
    
        @Test
        public void peopleGetIdsInjected() throws IOException, URISyntaxException {
            final URL resource = Thread.currentThread().getContextClassLoader().getResource("get_people_response.json");
            final String peopleResponseJson = Files.lines(Paths.get(resource.toURI()), StandardCharsets.UTF_8)
                    .collect(Collectors.joining(System.lineSeparator()));
            final String expectedBearerToken = "abcd";
    
            stubFor(get(urlMatching("/people"))
                    .withHeader("Authorization", matching(expectedBearerToken))
                    .willReturn(aResponse()
                            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                            .withBody(peopleResponseJson)));
    
            final Resources<Person> people = client.getPeople(expectedBearerToken);
    
            long expectedId = 1L;
            for (final Person person : people) {
                assertThat(person.getId(), IsEqual.equalTo(expectedId++));
            }
        }
    
        @TestConfiguration
        public static class LocalRibbonClientConfiguration {
            @Bean
            public ServerList<Server> ribbonServerList() {
                return new StaticServerList<>(new Server("localhost", wireMockRule.port()));
            }
        }
    
    }

#### Integration (Feign) Test Solution

Person.java

    import com.fasterxml.jackson.annotation.JsonProperty;
    import lombok.Data;
    import lombok.NonNull;
    
    import java.util.List;
    import java.util.Map;
    
    @Data
    public class Person {
        private Long id;
        private String name;
    
        @JsonProperty("_links")
        public void unpackIdFromLinks(final Map<String, Object> links) {
            final String uri = ((Map<String, String>) links.get("self")).get("href");
            final int idStartIndex = uri.lastIndexOf('/') + 1;
            final String idStr = uri.substring(idStartIndex);
            this.id = Long.parseLong(idStr);
        }
    }

Adding the `@JsonProperty("_links")` annotation to the method tells Jackson to call the method and use the entire `_links` object as the parameter:

get_people_response.json (excerpt)

      {
      "name": "Jane Doe",
      "_links": {
        "self": {
          "href": "https://example-person-crud.cfapps.io/people/1"
        },
        "person": {
          "href": "https://example-person-crud.cfapps.io/people/1"
        }
      }
    }
    

### Linking Entities Using Feign Client

#### Integration Test

AddressClientIT.java

    import com.github.tomakehurst.wiremock.junit.WireMockRule;
    import com.netflix.loadbalancer.Server;
    import com.netflix.loadbalancer.ServerList;
    import org.eclipse.jetty.http.HttpStatus;
    import org.junit.ClassRule;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.context.TestConfiguration;
    import org.springframework.cloud.netflix.ribbon.StaticServerList;
    import org.springframework.context.annotation.Bean;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import static com.github.tomakehurst.wiremock.client.WireMock.*;
    import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.wireMockConfig;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class AddressClientIT {
    
        @ClassRule
        public static WireMockRule wireMockRule = new WireMockRule(wireMockConfig().dynamicPort());
    
        @Autowired
        private AddressClient client;
    
        @Test
        public void setParentPersonMakesPutRequestWithValidData() {
            final String expectedBearerToken = "abcd";
            final String expectedUri = "/blah/blah/uri/123";
    
            stubFor(put(urlPathEqualTo("/address/1/person"))
                    .withHeader("Authorization", matching(expectedBearerToken))
                    .withRequestBody(equalTo(expectedUri))
                    .willReturn(aResponse()
                            .withStatus(HttpStatus.NO_CONTENT_204)));
    
            client.setParentPerson(expectedBearerToken, 2, expectedUri);
    
            wireMockRule.verify(putRequestedFor(urlPathEqualTo("/address/1/person"))
                    .withHeader("Authorization", matching(expectedBearerToken))
                    .withHeader("Content-Type", equalTo("text/uri-list"))
                    .withRequestBody(equalTo(expectedUri)));
        }
    
        @TestConfiguration
        public static class LocalRibbonClientConfiguration {
            @Bean
            public ServerList<Server> ribbonServerList() {
                return new StaticServerList<>(new Server("localhost", wireMockRule.port()));
            }
        }
    
    }

#### Example Solution

AddressClient.java

    import org.springframework.cloud.openfeign.FeignClient;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.PutMapping;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestHeader;
    
    @FeignClient(name = "example-person-crud", path = "/addresses")
    public interface StoryClient {
    
        @PutMapping(path = "/{addressId}/person", consumes = "text/uri-list")
        void setParentPerson(@RequestHeader("Authorization") final String bearerToken,
                              @PathVariable final long addressId,
                              @RequestBody final String personUri);
    }

### Returning Correct HTTP Error Responses

Before reading this section, check out the [Return 403 When Wrong Role](java/spring-integration-best-practices.html#_return_403_when_wrong_role) section of the [Spring Integration Best Practices](java/spring-integration-best-practices.html) guide.

Tests should be failing because the _wrong_ HTTP response is being returned for a particular scenario, before trying these solutions.

#### Forbidden

This should be returned when a user that is _properly authenticated_ attempts to access a resource they are not allowed to access.

    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.security.access.AccessDeniedException;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.Collection;
    
    @RestController
    public class PersonController {
    
        // mapping methods
        // ...
    
        @ExceptionHandler(AccessDeniedException.class)
        public ResponseEntity<String> handleAccessDenied(final AccessDeniedException ex) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).body(ex.getMessage());
        }
    }

#### Bad Request

This should be returned when a user that is _properly authenticated_ attempts to send an incomplete or incorrect request.

    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.HttpMessageConversionException;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.Collection;
    
    @RestController
    public class PersonController {
    
        // mapping methods
        // ...
    
        @ExceptionHandler({HttpMessageConversionException.class, IllegalArgumentException.class})
        public ResponseEntity<String> handleHttpMessageConversionException(final RuntimeException ex) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(ex.getMessage());
        }
    }