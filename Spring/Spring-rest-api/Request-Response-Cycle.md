# Spring Boot Request & Response Lifecycle (JSON ⇄ Java Object)

> One of the biggest "magic" features of Spring Boot is how it automatically converts JSON into Java objects and Java objects back into JSON. This document explains the complete lifecycle, including every important Spring component involved.

---

# Table of Contents

1. High-Level Flow
2. Request Lifecycle (JSON → Java Object)
3. Components Involved
4. Deep Dive into Each Component
5. Response Lifecycle (Java Object → JSON)
6. What Happens When Returning an Entity?
7. How Jackson Performs Serialization & Deserialization
8. HttpMessageConverters
9. DispatcherServlet Internals
10. Bean Validation
11. Common Interview Questions
12. Summary

---

# 1. High-Level Flow

Whenever a client calls your REST API, two conversions happen.

```
Incoming Request

JSON
   │
   ▼
Java Object (DTO/Entity)
   │
Controller executes
   │
   ▼
Java Object (DTO/Entity)
   │
   ▼
JSON
```

Spring performs both conversions automatically.

---

# 2. Example

Suppose the client sends:

```http
POST /users
Content-Type: application/json

{
    "name":"Aryan",
    "age":25
}
```

Controller

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping
    public UserDto createUser(@RequestBody UserDto dto) {
        return dto;
    }
}
```

Question:

How did this JSON become a `UserDto`?

Answer:

Spring used:

```
MappingJackson2HttpMessageConverter
                │
                ▼
      Jackson ObjectMapper
```

---

# 3. Complete Request Lifecycle

```
Client

POST /users
(JSON)

        │
        ▼

DispatcherServlet

        │
        ▼

HandlerMapping

        │
        ▼

HandlerAdapter

        │
        ▼

@RequestBody detected

        │
        ▼

MappingJackson2HttpMessageConverter

        │
        ▼

Jackson ObjectMapper.readValue()

        │
        ▼

UserDto object

        │
        ▼

Controller Method
```

---

# 4. Step-by-Step Request Processing

## Step 1 - Client Sends HTTP Request

```
POST /users
```

with

```json
{
   "name":"Aryan",
   "age":25
}
```

The request reaches Spring Boot.

---

## Step 2 - DispatcherServlet

Every request first reaches

```
DispatcherServlet
```

It is called the **Front Controller**.

It performs:

- Receives every request
- Finds the correct controller
- Invokes the controller
- Sends the response back

Think of it as the manager of Spring MVC.

```
Client

    │

    ▼

DispatcherServlet
```

---

## Step 3 - HandlerMapping

DispatcherServlet asks

```
Who can handle this request?
```

HandlerMapping searches all controllers.

Example:

```
POST /users
```

matches

```java
@PostMapping("/users")
```

inside

```
UserController
```

HandlerMapping returns

```
UserController.createUser()
```

---

## Step 4 - HandlerAdapter

DispatcherServlet cannot invoke controller methods directly.

Instead it delegates to

```
HandlerAdapter
```

Responsibilities:

- Prepare method arguments
- Convert request body
- Resolve path variables
- Resolve request parameters
- Resolve headers
- Call the controller

---

## Step 5 - Detect @RequestBody

Spring inspects

```java
public UserDto createUser(@RequestBody UserDto dto)
```

It notices

```
@RequestBody
```

Meaning

> Read HTTP Body and convert it into UserDto.

---

## Step 6 - Choose HttpMessageConverter

Spring now asks

```
Which converter can read JSON?
```

It looks through registered converters.

Examples:

| Content-Type | Converter |
|--------------|-----------|
| JSON | MappingJackson2HttpMessageConverter |
| XML | MappingJackson2XmlHttpMessageConverter |
| String | StringHttpMessageConverter |
| Byte Array | ByteArrayHttpMessageConverter |

Since request is JSON,

Spring selects

```
MappingJackson2HttpMessageConverter
```

---

## Step 7 - Jackson Converts JSON

Internally Spring calls

```java
objectMapper.readValue(json, UserDto.class);
```

Jackson creates

```java
UserDto dto = new UserDto();

dto.setName("Aryan");
dto.setAge(25);
```

Finally,

Controller receives

```java
UserDto dto
```

instead of JSON.

---

# 5. Controller Executes

Example

```java
@PostMapping
public UserDto createUser(@RequestBody UserDto dto){

    dto.setAge(dto.getAge()+1);

    return dto;
}
```

Notice

Controller returns

```
Java Object

NOT JSON
```

---

# 6. Complete Response Lifecycle

Now Spring performs the reverse process.

```
Controller

        │
        ▼

UserDto

        │
        ▼

HandlerAdapter

        │
        ▼

@ResponseBody detected
(or @RestController)

        │
        ▼

MappingJackson2HttpMessageConverter

        │
        ▼

ObjectMapper.writeValueAsString()

        │
        ▼

JSON

        │
        ▼

DispatcherServlet

        │
        ▼

HTTP Response

        │
        ▼

Client
```

---

# 7. Step-by-Step Response Processing

## Step 1

Controller returns

```java
return dto;
```

Spring now has

```
Java Object
```

---

## Step 2

Because controller is

```java
@RestController
```

Spring knows

```
Return object as HTTP Response Body
```

instead of rendering a JSP page.

`@RestController`

is equivalent to

```java
@Controller
@ResponseBody
```

---

## Step 3

Spring again chooses

```
HttpMessageConverter
```

Request Accept header

```
application/json
```

So converter selected is

```
MappingJackson2HttpMessageConverter
```

---

## Step 4

Converter calls

```java
objectMapper.writeValueAsString(dto);
```

Jackson produces

```json
{
   "name":"Aryan",
   "age":26
}
```

---

## Step 5

DispatcherServlet writes

```
HTTP Status

Headers

Body(JSON)
```

back to client.

---

# 8. What Happens if We Return an Entity?

Example

```java
@Entity
class User{

    Long id;

    String name;
}
```

Controller

```java
@GetMapping("/{id}")
public User getUser(){

    return repository.findById(id).get();

}
```

Exactly the same process happens.

```
User Entity

      │

      ▼

Jackson

      │

      ▼

JSON
```

Result

```json
{
   "id":1,
   "name":"Aryan"
}
```

Jackson does not care whether object is

- DTO
- Entity
- POJO
- Record

Everything is just a Java object.

---

# 9. How Jackson Performs Serialization

Serialization means

```
Java Object

↓

JSON
```

Suppose

```java
class User{

    private Long id;

    private String name;
}
```

Jackson inspects class using Reflection.

Conceptually

```java
Field[] fields = User.class.getDeclaredFields();
```

Discovers

```
id

name
```

Reads values

```
id=1

name=Aryan
```

Produces

```json
{
    "id":1,
    "name":"Aryan"
}
```

Normally Jackson uses getters.

If configured,

it can also directly access fields.

---

# 10. How Jackson Performs Deserialization

Deserialization means

```
JSON

↓

Java Object
```

Incoming JSON

```json
{
   "name":"Aryan",
   "age":25
}
```

Jackson creates

```java
UserDto dto = new UserDto();
```

Calls

```java
dto.setName("Aryan");

dto.setAge(25);
```

Finished.

---

# 11. Serialization vs Deserialization

Serialization

```
Java Object

↓

JSON
```

Method used

```java
writeValueAsString()
```

---

Deserialization

```
JSON

↓

Java Object
```

Method used

```java
readValue()
```

---

# 12. What is HttpMessageConverter?

This is Spring's abstraction responsible for converting

```
HTTP Body

⇄

Java Object
```

Some implementations

| Converter | Handles |
|------------|---------|
| MappingJackson2HttpMessageConverter | JSON |
| StringHttpMessageConverter | String |
| ByteArrayHttpMessageConverter | byte[] |
| MappingJackson2XmlHttpMessageConverter | XML |

Spring automatically chooses the correct converter.

---

# 13. What if Extra Fields Exist?

Incoming JSON

```json
{
    "name":"Aryan",
    "age":25,
    "salary":50000
}
```

DTO

```java
class UserDto{

    private String name;

    private int age;
}
```

Default behaviour

```
salary

↓

Ignored
```

because Spring Boot configures Jackson to ignore unknown properties by default.

---

# 14. What if Fields are Missing?

JSON

```json
{
    "name":"Aryan"
}
```

DTO

```java
private int age;
```

Result

```
age = 0
```

If

```java
Integer age;
```

Result

```
null
```

---

# 15. Bean Validation

Example

```java
public class UserDto{

    @NotBlank
    private String name;

    @NotNull
    private Integer age;

}
```

Controller

```java
@PostMapping
public UserDto save(
        @Valid @RequestBody UserDto dto){
    return dto;
}
```

Validation happens

```
Before entering controller
```

If validation fails

```
400 Bad Request
```

is returned automatically.

---

# 16. Where Does Jackson Come From?

Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot automatically configures

- ObjectMapper
- HttpMessageConverters
- DispatcherServlet
- Spring MVC

No manual configuration required.

---

# 17. Why Return DTO Instead of Entity?

Recommended

```
Controller

↓

DTO

↓

JSON
```

Reasons

- Hide sensitive fields
- Avoid exposing database model
- Prevent LazyInitializationException
- Avoid circular references
- Return only required data
- Stable API contract

---

# 18. End-to-End Diagram

```
                    CLIENT

                       │

          POST /users (JSON)

                       │

                       ▼

              DispatcherServlet

                       │

                       ▼

               HandlerMapping

                       │

                       ▼

               HandlerAdapter

                       │

                       ▼

             @RequestBody detected

                       │

                       ▼

       MappingJackson2HttpMessageConverter

                       │

                       ▼

       Jackson ObjectMapper.readValue()

                       │

                       ▼

                 UserDto Object

                       │

                       ▼

                Controller Method

                       │

                 Business Logic

                       │

                       ▼

            Return UserDto / Entity

                       │

                       ▼

             @ResponseBody detected

                       │

                       ▼

       MappingJackson2HttpMessageConverter

                       │

                       ▼

      Jackson ObjectMapper.writeValueAsString()

                       │

                       ▼

                     JSON

                       │

                       ▼

              DispatcherServlet

                       │

                       ▼

               HTTP Response

                       │

                       ▼

                    CLIENT
```

---

# 19. Key Classes to Remember

| Class | Responsibility |
|--------|----------------|
| DispatcherServlet | Front Controller; receives every request |
| HandlerMapping | Finds the correct controller method |
| HandlerAdapter | Invokes controller and prepares arguments |
| HttpMessageConverter | Converts HTTP body ↔ Java object |
| MappingJackson2HttpMessageConverter | JSON converter |
| ObjectMapper | Performs JSON serialization/deserialization |
| @RequestBody | Converts JSON → Java Object |
| @ResponseBody | Converts Java Object → JSON |
| @RestController | @Controller + @ResponseBody |
| @Valid | Triggers bean validation |

---

# 20. Interview Summary

If an interviewer asks:

> **How does Spring convert JSON into a Java object?**

You can answer:

> Every HTTP request first reaches the `DispatcherServlet`, which acts as the Front Controller. It uses `HandlerMapping` to locate the correct controller method and `HandlerAdapter` to invoke it. When it sees `@RequestBody`, it delegates the conversion of the HTTP request body to an appropriate `HttpMessageConverter`. For JSON requests, Spring uses `MappingJackson2HttpMessageConverter`, which internally uses Jackson's `ObjectMapper.readValue()` to deserialize JSON into a Java object before invoking the controller.

If they ask:

> **How does Spring convert the response back into JSON?**

You can answer:

> After the controller returns a Java object, `HandlerAdapter` processes the return value. Because the controller is annotated with `@RestController` (or the method with `@ResponseBody`), Spring treats the returned object as the HTTP response body. It again selects `MappingJackson2HttpMessageConverter`, which uses `ObjectMapper.writeValueAsString()` to serialize the Java object into JSON. The `DispatcherServlet` then writes this JSON into the HTTP response sent back to the client.