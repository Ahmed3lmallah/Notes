# Cache

**What Is a Cache?**

A **cache is a copy of the original data** that is stored for future reference. Previously accessed data is stored in a cache to make future retrieval faster.

**Why Use a Cache?**

* It’s less expensive.
	* Saves money
	* Saves processing time
* Data transformation requires processing power. Once that data is transformed, caching the results saves our app from have to process it again.
* Improved user experience.

**How Does a Cache Work?**

1. The app attempts to fetch the data from the cache.
1. If the data exists in the cache, it is presented.
1. If the data is not in the cache, the data is requested from the database.
1. This data may then be cached for faster retrieval when future requests are made.

**Things to Consider When Using a Cache**

* Caches take up disk space.
* Caches are most useful with generic data; i.e., not user-specific.
* Cached data may not be the most recent, therefore not accurate.
	* If the accuracy of the data is critical, a cache is likely not the best solution.
* When data changes often, a cache is likely not very effective.

**Difference between Cache and buffer:**

**Caching** is accessing a copy of the data, while **Buffering** preloads data from the “original” source. Buffering is usually used with large file sizes to match the transmission speed of the sender and the receiver.

**Additional Resources:**

* 
* 
* 

## Tutorial: [Spring Data Caching Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-caching-tutorial.md)

## Tutorial Summary:

1. Add the `Spring cache abstraction` dependency to `pom.xml` file

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		
1. Annotate the `Main Application` Class with `@EnableCaching`
1. Annotate the Controller:

	We will use the following annotations:

	* class level annotation
		* @CacheConfig(cacheNames = {"cache"})
	* method level annotation.
		* @CachePut(key = "#result.getId()") 
		* @Cacheable
		* @CacheEvict(key = "#rsvp.getId()") 
