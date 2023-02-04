# Spring/Spring Boot Unit Testing, API endpoint, View template Mocking  
--------
## Requirements
For standard development, you might need the following specifications:

* `Spring Boot v2.0+`
* `JDK v1.8+`
* `JUnit 4+` - The most popular and widely used testing framework for Java.
* `Mockito` - General-purpose framework for mocking and stubbing services and objects.
* `MockMVC` - Spring's module for performing integration testing during unit testing.
* `Lombok` - Convenience library for reducing boilerplate code.
* Any IDE that supports Java and Spring Boot (IntelliJ, VSCode, etc...)

### Sample Setup needs
`pom.xml` Dependencies
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version> 
</dependency>
```
* `spring-boot-starter-test` Starter for testing Spring Boot applications with libraries including JUnit Jupiter, Hamcrest and Mockito.
* `junit` JUnit is a unit testing framework to write and run repeatable automated tests on Java. (Optional)


-------
## Contents
1. [Mocking with `RestController` API endpoints](#Mocking-with-RestController-API-endpoints)
    - [Create endpoint](#Create-endpoint)
    - [Read endpoint](#Read-endpoint)
    - [Update endpoint](#Update-endpoint)
    - [Delete endpoint](#Delete-endpoint)
    - [Endpoint with `@RequestParam`](#Endpoint-with-RequestParam)
    - [Endpoint with `@PathVariable`](#Endpoint-with-PathVariable)
    - [Endpoint with `@RequestBody`](#Endpoint-with-RequestBody)
2. [Mocking with `Controller` endpoints](#Mocking-with-Controller-endpoints)
    - [Create endpoint](#Create-endpoint)
    - [Read endpoint](#Read-endpoint)
    - [Update endpoint](#Update-endpoint)
    - [Delete endpoint](#Delete-endpoint)
    - [Endpoint with @RequestParam](#Endpoint-with-@RequestParam)
    - [Endpoint with @PathVariable](#Endpoint-with-@PathVariable)
    - [Endpoint with @RequestBody](#Endpoint-with-@RequestBody)
3. [How to mock authentication](#How-to-mock-authentication)
4. [Mocking secured API controller endpoint using `@WithMockUser`](#Mocking-secured-API-controller-endpoint-using-WithMockUser)
5. [Mocking `@AuthenticationPrincipal` with `CustomUserDetails` object](#Mocking-`@AuthenticationPrincipal`-with-`CustomUserDetails`-object)
6. [Database integration testing](#Database-integration-testing)
    -  [Setup](#Setup) 
    -  [Cleanup](#Cleanup)
---


## Mocking with `RestController` API endpoints

POJO entity `Todo.java` class
```java 
@Data
@Entity
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String name;
    private String description;
}
```

And it's rest controller `TodoController.java` class
```java
@RestController
@RequestMapping("/todos")
public class TodoController {

    @Autowired
    private TodoService todoService;

    @PostMapping
    public ResponseEntity<Todo> createTodo(@Valid @RequestBody Todo createRequest) {
        // business logic
    } 

    @GetMapping("{id}")
    public ResponseEntity<Todo> getTodo(@PathVariable Long id) {
        // business logic
    }

    @PutMapping("{id}")
    public ResponseEntity<Todo> updateTodo(@PathVariable Long id, @Valid @RequestBody Todo updateRequest) {
        // business logic
    }

    @DeleteMapping("{id}")
    public ResponseEntity<ApiResponse> deleteTodo(@PathVariable Long id) {
        // business logic
    }
}
```

### Create endpoint
```java
@Test
void createTodoHttpRequest_shouldPass() throws Exception {
     String name = "test title";
     String description = "Lorem ipsum dolor sit amet, consectetur adipiscing elit";

     Todo createRequest = new Todo();
     createRequest.setName(name);
     createRequest.setDescription(description);
     
     // make entity's instance being persited and managed
     entityManager.persist(createRequest);
     entityManager.flush();

     mockMvc.perform(MockMvcRequestBuilders.post("/api/todo")
                 .contentType(MediaType.APPLICATION_JSON)
                 .content(objectMapper.writeValueAsString(createRequest)))
             .andExpect(status().isCreated())
             .andExpect(content().contentType(APPLICATION_JSON_UTF8))
             .andExpect(jsonPath("$.title", is(createRequest.getName()))) 
             .andExpect(jsonPath("$.description", is(createRequest.getDescription())))
             .andDo(print());
    }
```
### Read endpoint
```java
@Test
void getTodoByIdHttpRequest_shouldPass() throws Exception {
     Long id = 1L;

     mockMvc.perform(MockMvcRequestBuilders.get("/api/todo/{id}", id))
             .andExpect(status().isOk())
             .andExpect(jsonPath("$.id", is(id)))
	     .andExpect(jsonPath("$.name", is("whatever you defined for todo with id is 1"))) 
             .andExpect(jsonPath("$.description", is("whatever you defined for todo with id is 1")))
             .andDo(print());
    }
```

### Update endpoint
```java
@Test
void updateTodoByIdHttpRequest_shouldPass() throws Exception {
     Long id = 1L;
	 
     Todo updateRequest = new Todo();
     updateRequest.setName("do this");
     updateRequest.setDescription("Lorem ipsum dolor sit amet, consectetur adipiscing elit");

     mockMvc.perform(MockMvcRequestBuilders.put("/api/todos/{id}", id)
                 .contentType(MediaType.APPLICATION_JSON)
                 .content(objectMapper.writeValueAsString(updateRequest)))
             .andExpect(status().isOk())
             .andExpect(content().contentType(APPLICATION_JSON_UTF8))
             .andExpect(jsonPath("$.name", is(updateRequest.getName()))) 
             .andExpect(jsonPath("$.description", is(updateRequest.getDescription())))
             .andDo(print());
    }
```

### Delete endpoint
```java
@Test
void deleteTodoByIdHttpRequest_shouldPass() throws Exception {
     Long id = 1L;

     mockMvc.perform(MockMvcRequestBuilders.delete("/api/todos/{id}", id))
             .andExpect(status().isOk())
             .andExpect(jsonPath("$.status", is(Boolean.TRUE)))
             .andExpect(jsonPath("$.message", is("todo has been deleted successfully")))
             .andDo(print());
}
```

### Endpoint with `RequestParam`
Assume you want to attach a `@RequestParam` annotation to get the resources that contain given parameter, For example
```java
@GetMapping("{id}")
public ResponseEntity<Todo> getTodos(@RequestParam(value = "text", required = false) String text) {
     Todo response = text == null ? todoService.getTodos() : todoService.getTodosWithGivenText(text);
     // business logic
}
```
For testing this, just attach `param(...)` with parameter and its value defined in the controller endpoint
```java
@Test
void getTodosWithGivenTextHttpRequest_shouldPass() throws Exception {
    String param = "text";
    String paramValue = "givenParam";
		
    mockMvc.perform(MockMvcRequestBuilders.get("/api/todos")
                .param(param, paramValue))
            .andExpect(status().isOk())
            // expectJsonPathsResult...
}
```

### Endpoint with `PathVariable`
[refer to `Read Endpoint`](#read-endpoint)

### Endpoint with `RequestBody`
[refer to `Create Endpoint`](#create-endpoint)

### Mocking with `Controller` endpoints
```java

```

### 
```java

```

###
```java

```

###
```java

```

## Mocking secured API controller endpoint using `WithMockUser`
```java

```

## Mocking @AuthenticationPrincipal with `CustomUserDetails` object
```java

```

## How to mock authentication in Spring
 ```java

```

## How to mock secured API controller endpoint in Spring 
```java

```
## Mocking `AuthenticationPrincipal` with custom `CustomUserDetails` object
```java

```
## [Database integration testing](#Database-integration-testing)
* When we are perform integration testing with database
    * each test should run from known state
* Before each test, perform initialization (`setup`)
    * insert sample data 
* After each test, perform `cleanup`
    * delete the sample data

### [Setup](#Setup)
```java
@Autowired
private JdbcTemplate jdbc;  

@BeforeEach
void setupDbBeforeTransactions() throws Exception {
     jdbc.execute("INSERT INTO todos (id, name, description) VALUES (11, 'do that', 'lorem ipsum dolor sit amet')");
}   

// tests...

// cleanup...
```
### [Cleanup](#Cleanup)
```java
@Autowired
private JdbcTemplate jdbc;  

// setup...

// tests...

@AfterEach
    void cleanupDbAfterTransactions() throws Exception {
        jdbc.execute("DELETE FROM todos");
}
```

### References

1. https://eliux.github.io/java/spring/testing/how-to-mock-authentication-in-spring/
2. https://stackabuse.com/guide-to-unit-testing-spring-boot-rest-apis/
3. https://stackabuse.com/how-to-test-a-spring-boot-application/
4. https://babarowski.com/blog/mock-authentication-with-custom-userdetails/
