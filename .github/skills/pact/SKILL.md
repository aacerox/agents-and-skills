---
name: "pact"
description: "Pact.io is a consumer-driven contract testing framework that enables independent deployment of microservices by verifying service contracts between consumers and providers"
---



# Pact.io Contract Testing
Use this skill to verify API contracts between microservices. ALWAYS use Pact.io when you need to **test consumer-provider interactions without integration environments**, enabling independent service deployment by ensuring consumers and providers agree on API contracts (HTTP endpoints, message formats, and payloads).

## Scope
This skill is applicable exclusively to Java projects and Java-based contract testing.

## Quick Reference

- **[Basic Reference](resources/pact_basic_reference.md)**: Most commonly used features including basic consumer tests, POST requests with body, query parameters, type matching, arrays with each like, basic provider tests, state with parameters, message pact consumer/provider, headers and authentication
- **[Advanced Reference](resources/pact_advanced_reference.md)**: Advanced features including advanced matchers, filtering pacts by consumer version, publishing pacts (Maven/Gradle), can-i-deploy checks, pending pacts, message metadata, multiple interactions, complex state setup, custom verification settings, Pact V4 features, and provider state cleanup

## Creating Test

When creating tests with Pact.io:

- **ALWAYS add the required imports** related to the framework:
  ```java
  // Consumer tests
  import au.com.dius.pact.consumer.dsl.*;
  import au.com.dius.pact.consumer.junit5.*;
  import au.com.dius.pact.core.model.annotations.*;
  import au.com.dius.pact.core.model.*;
  
  // Provider tests
  import au.com.dius.pact.provider.junit5.*;
  import au.com.dius.pact.provider.junitsupport.*;
  ```

- **ALWAYS use basic reference** for standard test scenarios:
  - **Consumer Tests:**
    - Use `@ExtendWith(PactConsumerTestExt.class)` for JUnit 5
    - Use `@Pact(consumer = "ConsumerName")` to define contracts
    - Use `PactDslJsonBody` for request/response body definitions
    - Use type matchers: `integerType()`, `stringType()`, `decimalType()`
    - Use `@PactTestFor(pactMethod = "methodName")` to run tests
    - Use `MockServer` to get the mock provider URL
  
  - **Provider Tests:**
    - Use `@Provider("ProviderName")` to identify provider
    - Use `@PactBroker` to fetch pacts from broker
    - Use `@State("state description")` for state setup
    - Use `PactVerificationContext` for verification
    - Use `@TestTemplate` with `PactVerificationInvocationContextProvider`

- **Use advanced reference ONLY when tests require it** for complexity:
  - Advanced matchers with regex and constraints
  - Consumer version selectors for filtering pacts
  - Publishing pacts to broker with tags
  - Can-i-deploy checks in CI/CD
  - Pending pacts for WIP contracts
  - Message metadata for event-driven systems
  - Multiple interactions in single pact
  - Custom verification settings
  - Pact V4 features

## Modifying Test

When modifying existing Pact tests:

- **ALWAYS add the required imports** related to the framework:
  ```java
  import au.com.dius.pact.consumer.dsl.*;
  import au.com.dius.pact.provider.junit5.*;
  ```

- **ALWAYS use basic reference** for standard modifications:
  - **Consumer Tests:**
    - Add new interactions with additional `@Pact` methods
    - Add query parameters with `.query()`
    - Add headers with `.headers()`
    - Update request/response bodies with new fields
    - Add type matchers for new fields
  
  - **Provider Tests:**
    - Add new `@State` methods for new scenarios
    - Update state setup to match new consumer expectations
    - Add verification for new pact files
    - Update test data in state methods

- **Use advanced reference ONLY when tests require it** for complexity:
  - Add regex matchers for complex validations
  - Add consumer version selectors for selective verification
  - Add pact publishing to CI/CD pipeline
  - Add message pact testing for async communication
  - Add pending pact support for new contracts
  - Add state cleanup with action parameters

## Best Practices

**ALWAYS follow these best practices when using Pact.io:**

1. **Consumer-driven approach**
   - Consumers define their expectations
   - Providers verify they meet those expectations
   - Enables independent deployment

2. **Use descriptive state names**
   - State describes provider's data state
   - Example: "user with id 123 exists"
   - Makes provider tests clear

3. **Use type matchers, not exact values**
   - Use `integerType()` instead of `integerValue()`
   - Use `stringType()` instead of exact strings
   - More flexible, less brittle tests

4. **Keep pacts focused**
   - One interaction per test method
   - Test specific scenarios separately
   - Easier to understand and maintain

5. **Use Pact Broker**
   - Central repository for pacts
   - Version management
   - Can-i-deploy workflow support

6. **Implement proper state management**
   - Clean up state after verification
   - Use `@State(action = TEARDOWN)` when needed
   - Prevents test pollution

7. **Don't test business logic in pacts**
   - Pacts verify contracts, not behavior
   - Keep interactions simple
   - Test business logic in unit tests

8. **Version your pacts**
   - Tag pacts with branch names
   - Tag production deployments
   - Enables safe deployment workflows

9. **Use can-i-deploy before deployment**
   - Verify all consumers are compatible
   - Verify all providers are compatible
   - Prevents breaking changes

10. **Keep provider states minimal**
    - Only set up data needed for interaction
    - Avoid complex business logic
    - Focus on contract verification

## Common Patterns

### Basic Consumer Test Pattern
```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "UserService")
class UserConsumerTest {
    
    @Pact(consumer = "WebApp")
    public RequestResponsePact getUserPact(PactDslWithProvider builder) {
        return builder
            .given("user with id 1 exists")
            .uponReceiving("a request to get user")
                .path("/users/1")
                .method("GET")
            .willRespondWith()
                .status(200)
                .headers(Map.of("Content-Type", "application/json"))
                .body(new PactDslJsonBody()
                    .integerType("id", 1)
                    .stringType("name", "John Doe")
                    .stringType("email", "john@example.com"))
            .toPact();
    }
    
    @Test
    @PactTestFor(pactMethod = "getUserPact")
    void testGetUser(MockServer mockServer) {
        UserClient client = new UserClient(mockServer.getUrl());
        User user = client.getUser(1);
        
        assertThat(user.getId()).isEqualTo(1);
        assertThat(user.getName()).isNotNull();
        assertThat(user.getEmail()).isNotNull();
    }
}
```

### Basic Provider Test Pattern
```java
@Provider("UserService")
@PactBroker(host = "pact-broker.example.com", port = "443", scheme = "https")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserProviderTest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", port));
    }
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }
    
    @State("user with id 1 exists")
    void userExists() {
        User user = new User(1L, "John Doe", "john@example.com");
        userRepository.save(user);
    }
    
    @State("no users exist")
    void noUsers() {
        userRepository.deleteAll();
    }
}
```

### POST Request Consumer Pattern
```java
@Pact(consumer = "WebApp")
public RequestResponsePact createUserPact(PactDslWithProvider builder) {
    return builder
        .given("service is available")
        .uponReceiving("a request to create user")
            .path("/users")
            .method("POST")
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .stringType("name")
                .stringMatcher("email", ".*@.*\\..*"))
        .willRespondWith()
            .status(201)
            .headers(Map.of(
                "Content-Type", "application/json",
                "Location", regex("/users/\\d+", "/users/123")))
            .body(new PactDslJsonBody()
                .integerType("id")
                .stringType("name")
                .stringType("email"))
        .toPact();
}
```

### Query Parameters Pattern
```java
@Pact(consumer = "WebApp")
public RequestResponsePact searchUsersPact(PactDslWithProvider builder) {
    return builder
        .given("users exist")
        .uponReceiving("a search request")
            .path("/users")
            .method("GET")
            .query("status=active&limit=10")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonArray()
                .eachLike()
                    .integerType("id")
                    .stringType("name")
                    .stringValue("status", "active")
                .closeObject())
        .toPact();
}
```

### Message Pact Consumer Pattern
```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "OrderEventProvider", pactVersion = PactSpecVersion.V3)
class OrderEventConsumerTest {
    
    @Pact(consumer = "NotificationService")
    public MessagePact orderCreatedEventPact(MessagePactBuilder builder) {
        return builder
            .given("order is created")
            .expectsToReceive("order created event")
            .withContent(new PactDslJsonBody()
                .uuid("orderId")
                .stringType("customerId")
                .decimalType("amount")
                .stringValue("status", "CREATED")
                .datetime("timestamp", "yyyy-MM-dd'T'HH:mm:ss"))
            .toPact();
    }
    
    @Test
    @PactTestFor(pactMethod = "orderCreatedEventPact")
    void testOrderCreatedEvent(List<Message> messages) throws Exception {
        String messageBody = messages.get(0).getContents().valueAsString();
        OrderEvent event = objectMapper.readValue(messageBody, OrderEvent.class);
        
        eventHandler.handle(event);
        
        verify(notificationService).sendNotification(any());
    }
}
```

### Message Pact Provider Pattern
```java
@Provider("OrderEventProvider")
@PactBroker(host = "pact-broker.example.com")
class OrderEventProviderTest {
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }
    
    @PactVerifyProvider("order created event")
    String verifyOrderCreatedEvent() throws JsonProcessingException {
        OrderEvent event = new OrderEvent(
            UUID.randomUUID(),
            "CUST-123",
            new BigDecimal("99.99"),
            "CREATED",
            Instant.now()
        );
        return objectMapper.writeValueAsString(event);
    }
}
```

### State with Parameters Pattern
```java
@State("user {id} exists with balance {balance}")
void userExistsWithBalance(Map<String, Object> params) {
    Long id = Long.parseLong(params.get("id").toString());
    BigDecimal balance = new BigDecimal(params.get("balance").toString());
    
    User user = new User(id, "Test User");
    user.setBalance(balance);
    userRepository.save(user);
}
```

### Authentication Pattern
```java
@Pact(consumer = "WebApp")
public RequestResponsePact authenticatedRequestPact(PactDslWithProvider builder) {
    return builder
        .given("user is authenticated")
        .uponReceiving("authenticated request to protected endpoint")
            .path("/api/protected")
            .method("GET")
            .headers(Map.of(
                "Authorization", "Bearer token123",
                "Accept", "application/json"))
        .willRespondWith()
            .status(200)
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .stringType("data"))
        .toPact();
}
```

### Array Matching Pattern
```java
@Pact(consumer = "WebApp")
public RequestResponsePact productListPact(PactDslWithProvider builder) {
    return builder
        .given("products exist")
        .uponReceiving("products list request")
            .path("/products")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonArray()
                .minArrayLike(1)
                    .integerType("id")
                    .stringType("name")
                    .decimalType("price")
                    .booleanType("inStock")
                .closeObject()
                .asArray())
        .toPact();
}
```

## Resources

- [Pact.io Official Documentation](https://docs.pact.io/)
- [Pact JVM Documentation](https://github.com/pact-foundation/pact-jvm)
- [Pact Broker](https://docs.pact.io/pact_broker)
- [Pact Workshop](https://docs.pact.io/implementation_guides/workshops)
- [Pact GitHub Repository](https://github.com/pact-foundation/pact-jvm)
