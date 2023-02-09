# Spring/Spring Boot Unit Testing, API endpoint, View template Mocking

### This is the Test-driven Development study notes and takeaway of mine after has been practicing by code as well as reading several articles and sources over the internet. JUST WANT TO NOTE IT BACK FOR FUTURE RE-LOOKING UP.

## Requirements

For standard development, you might need the following specifications:

-   `Spring Boot v2.0+`
-   `JDK v1.8+`
-   `JUnit 4+` - The most popular and widely used testing framework for Java.
-   `Mockito` - General-purpose framework for mocking and stubbing services and objects.
-   `MockMVC` - Spring's module for performing integration testing during unit testing.
-   `Lombok` - Convenience library for reducing boilerplate code.
-   Any IDE that supports Java and Spring Boot (IntelliJ, VSCode, etc...)

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

-   `spring-boot-starter-test` Starter for testing Spring Boot applications with libraries including JUnit Jupiter, Hamcrest and Mockito.
-   `junit` JUnit is a unit testing framework to write and run repeatable automated tests on Java. (Optional)

---

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
    - [Setup](#Setup)
    - [Cleanup](#Cleanup)
7. [Why Mocking?](#Why-Mocking)
8. [Responsibilities of a Web Controller](#Responsibilities-of-a-web-controller)
    - [Verifying HTTP Request Matching](#Verifying-HTTP-Request-Matching)
    - [Verifying Input Se/Deserialization](#Verifying-Input-Se/Deserialization)
    - [Verifying Input Validation](#Verifying-Input-Validation)
    - [Verifying Business Logic](#Verifying-Business-Logic)
    - [Verifying Output Serialization](#Verifying-Output-Serialization)
    - [Verifying Exception Handling](#Verifying-Exception-Handling)
9. [Unit or Integration Test?](#Unit-or-Integration-Test?)
10. [Why Web Controllers should Integration-Test?](#Why-Web-Controllers-should-Integration-Test)

---

## Mocking with `RestController` API endpoints

For the sake of brevity, just one POJO entity is used for the example.
`Todo.java` class

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
@PersistenceContext
private EntityManager entityManager

@Test
void createTodoHttpRequest_shouldPass() throws Exception {
     String name = "test title";
     String description = "Lorem ipsum dolor sit amet, consectetur adipiscing elit";

     Todo createRequest = new Todo();
     createRequest.setName(name);
     createRequest.setDescription(description);

     // make entity's instance being persisted and managed
     entityManager.persist(createRequest);
     entityManager.flush();

     mockMvc.perform(MockMvcRequestBuilders.post("/api/todo")
                 .contentType(MediaType.APPLICATION_JSON)
                 .content(objectMapper.writeValueAsString(createRequest)))
             .andExpect(status().isCreated())
             .andExpect(content().contentType(APPLICATION_JSON_UTF8))
             .andExpect(jsonPath("$.name", is(createRequest.getName())))
             .andExpect(jsonPath("$.description", is(createRequest.getDescription())))
             .andDo(print());
    }
```

The code mostly speaks for itself what it does. As the `POST` request, first we need a `RequestBody` (e.g. name, description),
then we need `EntityManager` to make it persisted and managed in the DB.

`MockMvc.perform()` accepts a `MockMvcRequest` and mocks the API call given the fields of the object. Here, we built a request via the `MockMvcRequestBuilders`, as the request is `POST` and it does accept the `RequestBody` with `contentType` as `APPLICATION_JSON`,
and the `content` should be mapped and set as a UTF-8 String.

After `perform()` is ran, `andExpect()` methods are subsequently chained to it and tests against the results returned by the method.
Depends on the request, the content(`resource`, `status code`) is expected returning based on business logic.

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

[refer to `Read Endpoint`or`Update Endpoint`](#read-endpoint)

### Endpoint with `RequestBody`

[refer to `Create Endpoint`or`Update Endpoint`](#create-endpoint)

### Mocking with `Controller` endpoints

> TODO:

## Mocking secured API controller endpoint using `WithMockUser`

Assuming you've integrated `Spring Security` framework to secure controller endpoints with role-based authorities,
and you want to test those secured endpoints with specified user. E.g `USER_ROLE`-based user or `ADMIN_ROLE`-based user.

```java
@Test
@WithMockUser(username = "username_value", roles = "user_role_value")
void deleteTodoByIdHttpRequest_shouldPass() throws Exception {
     Long id = 1L;

     mockMvc.perform(MockMvcRequestBuilders.delete("/api/todos/{id}", id))
             .andExpect(status().isOk())
             .andExpect(jsonPath("$.status", is(Boolean.TRUE)))
             .andExpect(jsonPath("$.message", is("todo has been deleted successfully")))
             .andDo(print());
}
```

By defining test with annotation `@WithMockUser(username = "username_value", roles = "user_roles")`, the test will be run as user
who has username as `username_value` and roles as `user_role`.

**Further read**:

-   https://docs.spring.io/spring-security/site/docs/4.2.x/reference/html/test-method.html

## Mocking @AuthenticationPrincipal with `CustomUserDetails` object

In case you need to access current logged in user http context to get user's information (e.g. `username`, `email` or `address`) to perform some business logic in particular controller endpoint. For example, through using `@AuthenticationPrincipal`

```java
@PutMapping("{id}")
public ResponseEntity<Todo> updateTodo(@PathVariable Long id, @Valid @RequestBody Todo updateRequest,
    @AuthenticationPrincipal YourCustomUserDetails currentUser) {
    // business logic
}
```

Note that, it's assumed that you have `YouCustomUserDetails` exposed as a bean already.
Therefore it's convenient to test the above endpoint using `@WithUserDetails`

```java
@Test
@WithUserDetails("user")
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

**Further read**:

-   https://docs.spring.io/spring-security/site/docs/4.2.x/reference/html/test-method.html

## [Database integration testing](#Database-integration-testing)

-   When we are perform integration testing with database
    -   each test should run from known state
-   Before each test, perform initialization (`setup`)
    -   insert sample data
-   After each test, perform `cleanup`
    -   delete the sample data

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

## [Why Mocking?](#why-Mocking)

Why should we use a mock instead of a real service object in a test?

Imagine the service implementation above has a dependency to a database or some other third-party system. We don’t want to have our test run against the database. The reason is _If the database isn’t available, the test will fail even though our system under test might be completely bug-free_. _The more dependencies we add in a test, the more reasons a test has to fail_. And most of those reasons will be the wrong ones. If we use a mock instead, we can mock all those potential failures away.

Aside from reducing failures, mocking also reduces our tests' complexity and thus saves us some effort. It takes a lot of boilerplate code to set up a whole network of correctly-initialized objects to be used in a test. Using mocks, we only have to “instantiate” one mock instead of a whole rat-tail of objects the real object might need to be instantiated.

In summary, Mocking helps to:

-   Isolate the component under test
-   Make tests faster and more reliable
-   Test edge cases and unusual scenarios
-   Make tests easier to write and maintain
<!--
So, in a test of our SendMoneyController above, instead of a real instance of SendMoneyUseCase, we want to use a mock with the same interface whose behavior we can control as needed in the test. -->

## [Responsibilities of a Web Controller](#Responsibilities-of-a-web-controller)

| #   | Responsibility         | Description                                                                                                                                                                                       |
| --- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0   | Perform HTTP Requests  | The controller should respond to certain URLs, HTTP methods and content types.                                                                                                                    |
| 1   | Deserialize Input      | The controller should parse the incoming HTTP request and create Java objects from variables in the URL, HTTP request parameters and the request body so that we can work with them in the code.3 |
| 2   | Validate Input         | The controller is the first line of defense against bad input, so it’s a place where we can validate the input. 3                                                                                 |
| 3   | Perform Business Logic | Having parsed the input, the controller must transform the input into the model expected by the business logic and pass it on to the business logic.                                              |
| 4   | Serialize Output       | The controller takes the output of the business logic and serializes it into an HTTP response.                                                                                                    |
| 5   | Exception Handling     | If an exception occurs somewhere on the way, the controller should translate it into a meaningful error message and HTTP status for the user.                                                     |

### [Verifying HTTP Request Matching](#Verifying-HTTP-Request-Matching)

Performing particular HTTP request is straightforward task for the controller endpoint. And of course, we need to verify it for proper response result.

```java
mockMvc.perform(MockMvcRequestBuilders.get("/api/todo/{id}", id))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id", is(id)))
            .andExpect(jsonPath("$.name", is("whatever you defined for todo with id is 1")))
            .andExpect(jsonPath("$.description", is("whatever you defined for todo with id is 1")))
            .andDo(print());
```

### [Verifying Input Se/Deserialization](#Verifying-Input-Se/Deserialization)

Based on business logic, some HTTP requests may need particular input such `@RequestParam` or `@RequestBody` to be performed. E.g: `POST`, `UPDATE`, or `GET` requests. Whatever the input is, we need to verify content type of it before it's being sent. Normally, `JSON` or `XMl` for the request body, `String`, `Int`, or `Long` for request parameters, etc...

```java
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
```

### [Verifying Input Validation](#Verifying-Input-Validation)

Let's say, for each newly inserted `Todo` item data, its `name` field must be present, and the length must be equal or less than 200. You can't validate the input by displaying some text to UI such `name field must be present and allowed maximum length is less than 200 characters` and expect users will act what you said. Instead, you must validate it like this:

```java
@Value
public class TodoRequest {

  @NotNull
  @Size(max = 200, message = "name must be maximum 200 characters")
  private final String name;

  // other fields
}
```

### [Verifying Business Logic](#Verifying-Business-Logic)

Either, the request is `UPDATE`, `POST`, or `DELETE`, you need to verify it to get expected response result.

```java
@Autowired
private TodoService todoService;

@PostMapping
public ResponseEntity<Todo> createTodo(@Valid @RequestBody Todo createRequest) {
    Todo response = todoService.createTodo(createRequest);
    // ....
}

@PutMapping("{id}")
public ResponseEntity<Todo> updateTodo(@PathVariable Long id, @Valid @RequestBody Todo updateRequest) {
    Todo response = todoService.updateTodo(updateRequest);
    // ....
}
```

### [Verifying Output Serialization](#Verifying-Output-Serialization)

Sometimes, after business logic has been performed, you want to check the output response from the test is matched with the one you expected.

```java
MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.delete("/api/todos/{id}", id))
             .andExpect(status().isOk())
             .andExpect(jsonPath("$.status", is(Boolean.TRUE)))
             .andExpect(jsonPath("$.message", is("todo has been deleted successfully")))
             .andReturn();

    // verify response body
    TodoResponseResource expectedResponseBody = ...;
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(actualResponseBody).isEqualToIgnoringWhitespace(
              objectMapper.writeValueAsString(expectedResponseBody));
```

### [Verifying Exception Handling](#Verifying-Exception-Handling)

No matter, how good your application is, you will not be able to predict what user does with the application's features, maybe the weird behaviors that you couldn't expect. So that's why exceptions should be handled thoroughly.

```java
Long id = 1L;

String nameAsNull = null;
Todo updateRequest = new Todo();
updateRequest.setName(nameAsNull);
updateRequest.setDescription("Lorem ipsum dolor sit amet, consectetur adipiscing elit");

MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.put("/api/todos/{id}", id)
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(updateRequest)))
        .andExpect(status().isBadRequest())
        .andExpect(content().contentType(APPLICATION_JSON_UTF8))
        // businessExpectedResponse
        .andReturn();

ErrorResult expectedErrorResponse = new ErrorResult("name", "must not be null");
String actualResponseBody = mvcResult.getResponse().getContentAsString();
String expectedResponseBody = objectMapper.writeValueAsString(expectedErrorResponse);

assertThat(actualResponseBody).isEqualToIgnoringWhitespace(expectedResponseBody);
```

In bonus summary, `Exception handling` helps to:

-   _Improve stability_: Exception handling helps to prevent the termination of the program when an unexpected error occurs.
-   _Enhance user experience_: Exception handling helps to provide meaningful error messages to users.
    Help to maintain control: Exception handling helps to maintain control over the program's flow, even in the face of unexpected errors. This can help to prevent the program from executing unintended or harmful code.
-   _Facilitate debugging_: By catching exceptions and logging error information, exception handling can make it easier to identify and debug problems in the program.

## [Unit or Integration Test?](#Unit-or-Integration-Test?)

| #   | Responsibility         | Covered in Unit-Test?                                                                                                                                                                                    |
| --- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0   | Perform HTTP Requests  | [x] No, because the unit test would not evaluate the @PostMapping annotation and similar annotations specifying the properties of a HTTP request.                                                        |
| 1   | Deserialize Input      | [x] No, because annotations like @RequestParam and @PathVariable would not be evaluated. Instead we would provide the input as Java objects, effectively skipping deserialization from an HTTP request.3 |
| 2   | Validate Input         | [x] Not when depending on bean validation, because the @Valid annotation would not be evaluated.                                                                                                         |
| 3   | Perform Business Logic | [v] Yes, because we can verify if the mocked business logic has been called with the expected arguments.                                                                                                 |
| 4   | Serialize Output       | [x] No, because we can only verify the Java version of the output, and not the HTTP response that would be generated.                                                                                    |
| 5   | Exception Handling     | [x] No. We could check if a certain exception was raised, but not that it was translated to a certain JSON response or HTTP status code.3                                                                |

## [Why Web Controllers should Integration-Test?](#Why-Web-Controllers-should-Integration-Test)

Testing a Spring Web Controller with a unit test like this only covers a fraction of the potential errors that can happen in production. The unit test above verifies that a certain response code is returned, but it does not integrate with Spring to check if the input parameters are parsed correctly from an HTTP request, or if the controller listens to the correct path, or if exceptions are transformed into the expected HTTP response, and so on.

----


### References

-   https://martinfowler.com/testing/
-   https://martinfowler.com/articles/practical-test-pyramid.html
-   https://martinfowler.com/bliki/TestDrivenDevelopment.html
-   https://eliux.github.io/java/spring/testing/how-to-mock-authentication-in-spring/
-   https://stackabuse.com/guide-to-unit-testing-spring-boot-rest-apis/
-   https://stackabuse.com/how-to-test-a-spring-boot-application/
-   https://reflectoring.io/spring-boot-mock/
-   https://reflectoring.io/spring-boot-web-controller-test/
-   https://docs.spring.io/spring-security/site/docs/4.2.x/reference/html/test-method.html
-   https://babarowski.com/blog/mock-authentication-with-custom-userdetails/
-   more...
