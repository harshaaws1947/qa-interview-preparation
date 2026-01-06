# API Testing Interview Questions for QA/SDET

## REST API Basics

### 1. What is REST API?
**REST (Representational State Transfer)** is an architectural style for designing networked applications.

**Key Principles:**
- **Stateless**: Each request contains all information needed
- **Client-Server**: Separation of concerns
- **Cacheable**: Responses can be cached
- **Uniform Interface**: Consistent API design
- **Layered System**: Multiple layers between client and server

### 2. What are HTTP methods and when to use them?

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve data | Yes | Yes |
| POST | Create new resource | No | No |
| PUT | Update/Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

### 3. What are HTTP status codes?

| Range | Category | Common Codes |
|-------|----------|--------------|
| 1xx | Informational | 100 Continue |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

### 4. What is the difference between PUT and PATCH?

| PUT | PATCH |
|-----|-------|
| Replaces entire resource | Partial update |
| Must send complete data | Send only changed fields |
| Idempotent | Not necessarily idempotent |

```json
// PUT - Replace entire user
PUT /users/1
{
    "name": "John",
    "email": "john@example.com",
    "phone": "1234567890"
}

// PATCH - Update only email
PATCH /users/1
{
    "email": "newemail@example.com"
}
```

### 5. What is the difference between authentication and authorization?

| Authentication | Authorization |
|----------------|---------------|
| Who are you? | What can you access? |
| Verifies identity | Grants permissions |
| Login credentials | Access control |
| 401 Unauthorized | 403 Forbidden |

## REST Assured Framework

### 6. How do you set up REST Assured?
```java
// Maven dependency
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.4.0</version>
</dependency>

// Basic setup
RestAssured.baseURI = "https://api.example.com";
RestAssured.basePath = "/v1";
```

### 7. Explain GET request in REST Assured
```java
// Simple GET
Response response = given()
    .when()
    .get("/users/1")
    .then()
    .statusCode(200)
    .extract().response();

// GET with query parameters
given()
    .queryParam("page", 1)
    .queryParam("limit", 10)
    .when()
    .get("/users")
    .then()
    .statusCode(200);

// GET with headers
given()
    .header("Authorization", "Bearer token")
    .header("Content-Type", "application/json")
    .when()
    .get("/users");
```

### 8. Explain POST request in REST Assured
```java
// POST with JSON body
String requestBody = """
    {
        "name": "John Doe",
        "email": "john@example.com"
    }
    """;

given()
    .contentType(ContentType.JSON)
    .body(requestBody)
    .when()
    .post("/users")
    .then()
    .statusCode(201)
    .body("name", equalTo("John Doe"));

// POST with POJO
User user = new User("John", "john@example.com");
given()
    .contentType(ContentType.JSON)
    .body(user)
    .when()
    .post("/users");
```

### 9. How to validate JSON response?
```java
// Using Hamcrest matchers
given()
    .when()
    .get("/users/1")
    .then()
    .body("name", equalTo("John"))
    .body("email", containsString("@"))
    .body("age", greaterThan(18))
    .body("roles", hasSize(3))
    .body("roles", hasItem("admin"));

// Using JSONPath
Response response = get("/users/1");
String name = response.jsonPath().getString("name");
List<String> roles = response.jsonPath().getList("roles");
```

### 10. How to extract values from response?
```java
Response response = given()
    .when()
    .get("/users/1")
    .then()
    .extract().response();

// Extract values
String name = response.path("name");
int statusCode = response.getStatusCode();
String header = response.getHeader("Content-Type");
String body = response.getBody().asString();

// JSONPath extraction
JsonPath jsonPath = response.jsonPath();
String email = jsonPath.getString("data.email");
List<Integer> ids = jsonPath.getList("data.users.id");
```

### 11. How to handle authentication?
```java
// Basic Auth
given()
    .auth().basic("username", "password")
    .when()
    .get("/secure/endpoint");

// Bearer Token
given()
    .header("Authorization", "Bearer " + token)
    .when()
    .get("/secure/endpoint");

// OAuth 2.0
given()
    .auth().oauth2(accessToken)
    .when()
    .get("/secure/endpoint");

// API Key
given()
    .header("X-API-Key", apiKey)
    .when()
    .get("/endpoint");
```

### 12. How to use Request Specification?
```java
// Create reusable spec
RequestSpecification requestSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setContentType(ContentType.JSON)
    .addHeader("Authorization", "Bearer token")
    .build();

// Use in tests
given()
    .spec(requestSpec)
    .body(payload)
    .when()
    .post("/users");
```

### 13. How to use Response Specification?
```java
// Create response spec
ResponseSpecification responseSpec = new ResponseSpecBuilder()
    .expectStatusCode(200)
    .expectContentType(ContentType.JSON)
    .expectResponseTime(lessThan(3000L))
    .build();

// Use in tests
given()
    .when()
    .get("/users")
    .then()
    .spec(responseSpec);
```

## JSON Handling

### 14. What is JSON Schema Validation?
```java
// Add dependency: json-schema-validator

// Validate against schema
given()
    .when()
    .get("/users/1")
    .then()
    .body(matchesJsonSchemaInClasspath("user-schema.json"));

// Schema file (user-schema.json)
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "required": ["id", "name", "email"],
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"}
    }
}
```

### 15. How to handle dynamic JSON?
```java
// Using Groovy path
given()
    .when()
    .get("/users")
    .then()
    .body("data.find { it.name == 'John' }.id", equalTo(1))
    .body("data.findAll { it.age > 30 }.size()", greaterThan(0))
    .body("data.collect { it.name }", hasItem("John"));
```

## API Test Design

### 16. What are API test types?

1. **Functional Testing**: Verify endpoints work correctly
2. **Integration Testing**: Test API with other systems
3. **Performance Testing**: Load, stress, spike tests
4. **Security Testing**: Authentication, authorization, injection
5. **Contract Testing**: Verify API contract compliance
6. **End-to-End Testing**: Complete workflow testing

### 17. What should you test in APIs?

**Positive Tests:**
- Valid inputs return correct response
- Correct status codes
- Response structure and data types
- Headers and cookies

**Negative Tests:**
- Invalid inputs (missing, wrong type, boundary)
- Authentication failures
- Authorization failures
- Rate limiting

**Edge Cases:**
- Empty values
- Special characters
- Large payloads
- Concurrent requests

### 18. How to test error responses?
```java
// Test 400 Bad Request
given()
    .contentType(ContentType.JSON)
    .body("{\"name\": \"\"}")  // Invalid - empty name
    .when()
    .post("/users")
    .then()
    .statusCode(400)
    .body("error", equalTo("Validation failed"))
    .body("message", containsString("name"));

// Test 404 Not Found
given()
    .when()
    .get("/users/99999")
    .then()
    .statusCode(404)
    .body("error", equalTo("User not found"));

// Test 401 Unauthorized
given()
    .when()
    .get("/secure/data")
    .then()
    .statusCode(401);
```

### 19. How to test file upload?
```java
given()
    .multiPart("file", new File("test.pdf"))
    .multiPart("description", "Test document")
    .when()
    .post("/upload")
    .then()
    .statusCode(200)
    .body("fileName", equalTo("test.pdf"));
```

### 20. How to test file download?
```java
byte[] fileContent = given()
    .when()
    .get("/download/report.pdf")
    .then()
    .statusCode(200)
    .contentType("application/pdf")
    .extract().asByteArray();

// Save to file
Files.write(Paths.get("downloaded.pdf"), fileContent);
```

## Advanced Topics

### 21. How to handle cookies?
```java
// Send cookies
given()
    .cookie("session_id", "abc123")
    .cookie("token", "xyz789")
    .when()
    .get("/dashboard");

// Extract cookies
Response response = get("/login");
String sessionId = response.getCookie("session_id");
Map<String, String> cookies = response.getCookies();
```

### 22. How to use filters and logging?
```java
// Log request and response
given()
    .filter(new RequestLoggingFilter())
    .filter(new ResponseLoggingFilter())
    .when()
    .get("/users");

// Log conditionally
given()
    .log().ifValidationFails()
    .when()
    .get("/users")
    .then()
    .log().ifError();

// Custom logging
RestAssured.filters(new AllureRestAssured());  // Allure reporting
```

### 23. What is API mocking and when to use it?

**When to use:**
- Third-party API is unavailable
- Testing error scenarios
- Parallel development (API not ready)
- Isolated unit testing

**Tools:**
- WireMock
- MockServer
- JSON Server

```java
// WireMock example
stubFor(get(urlEqualTo("/users/1"))
    .willReturn(aResponse()
        .withStatus(200)
        .withHeader("Content-Type", "application/json")
        .withBody("{\"id\": 1, \"name\": \"John\"}")));
```

### 24. How to test API performance?
```java
// Response time assertion
given()
    .when()
    .get("/users")
    .then()
    .time(lessThan(2000L));  // milliseconds

// For load testing, use tools like:
// - JMeter
// - Gatling
// - K6
// - Locust
```

### 25. What is Contract Testing?

**Consumer-Driven Contract Testing:**
- Consumer defines expected API behavior
- Provider verifies against contract
- Tools: Pact, Spring Cloud Contract

```java
// Pact example
@Pact(consumer = "ConsumerService")
public RequestResponsePact createPact(PactDslWithProvider builder) {
    return builder
        .given("User exists")
        .uponReceiving("Get user by ID")
        .path("/users/1")
        .method("GET")
        .willRespondWith()
        .status(200)
        .body(new PactDslJsonBody()
            .integerType("id")
            .stringType("name"))
        .toPact();
}
```

---

## Quick Tips for Interview

1. **Know HTTP fundamentals** - Methods, status codes, headers
2. **Master REST Assured syntax** - Given-When-Then pattern
3. **Understand JSON handling** - JSONPath, schema validation
4. **Practice authentication** - Basic, Bearer, OAuth
5. **Know test design** - What to test, positive/negative scenarios
