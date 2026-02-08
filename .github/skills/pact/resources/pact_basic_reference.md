# Pact.io Basic Reference

## Most Commonly Used Features

### Basic Consumer Test

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "UserService")
class UserConsumerTest {
    
    @Pact(consumer = "WebApp")
    public RequestResponsePact createUserPact(PactDslWithProvider builder) {
        return builder
            .given("user exists")
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
    @PactTestFor(pactMethod = "createUserPact")
    void testGetUser(MockServer mockServer) {
        UserClient client = new UserClient(mockServer.getUrl());
        User user = client.getUser(1);
        
        assertThat(user.getId()).isEqualTo(1);
        assertThat(user.getName()).isNotNull();
    }
}
```

### POST Request with Body

```java
@Pact(consumer = "WebApp")
public RequestResponsePact createUserCreationPact(PactDslWithProvider builder) {
    return builder
        .given("service is available")
        .uponReceiving("a request to create user")
            .path("/users")
            .method("POST")
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .stringType("name")
                .stringType("email"))
        .willRespondWith()
            .status(201)
            .headers(Map.of(
                "Content-Type", "application/json",
                "Location", regex("/users/\\d+", "/users/123")))
            .body(new PactDslJsonBody()
                .integerType("id")
                .stringMatcher("name", ".*")
                .stringMatcher("email", ".*@.*\\..*"))
        .toPact();
}
```

### Query Parameters

```java
@Pact(consumer = "WebApp")
public RequestResponsePact searchUsersPact(PactDslWithProvider builder) {
    return builder
        .given("users exist")
        .uponReceiving("a search request")
            .path("/users")
            .method("GET")
            .query("name=John&age=30")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonArray()
                .object()
                    .integerType("id", 1)
                    .stringType("name", "John")
                    .integerType("age", 30)
                .closeObject())
        .toPact();
}
```

### Type Matching

```java
@Pact(consumer = "WebApp")
public RequestResponsePact orderPact(PactDslWithProvider builder) {
    return builder
        .given("order exists")
        .uponReceiving("order details request")
            .path("/orders/1")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .integerType("orderId")
                .stringType("customerName")
                .decimalType("totalAmount")
                .booleanType("isPaid")
                .uuid("transactionId")
                .date("orderDate", "yyyy-MM-dd")
                .datetime("createdAt", "yyyy-MM-dd'T'HH:mm:ss"))
        .toPact();
}
```

### Arrays with Each Like

```java
@Pact(consumer = "WebApp")
public RequestResponsePact productsPact(PactDslWithProvider builder) {
    return builder
        .given("products exist")
        .uponReceiving("products list request")
            .path("/products")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonArray()
                .eachLike()
                    .integerType("id")
                    .stringType("name")
                    .decimalType("price")
                .closeObject())
        .toPact();
}
```

### Basic Provider Test

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
    
    @State("user exists")
    void userExists() {
        User user = new User(1L, "John Doe", "john@example.com");
        userRepository.save(user);
    }
    
    @State("service is available")
    void serviceAvailable() {
        // Setup if needed
    }
}
```

### State with Parameters

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

### Message Pact Consumer

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "OrderEventProvider", pactVersion = PactSpecVersion.V3)
class OrderEventConsumerTest {
    
    @Pact(consumer = "NotificationService")
    public MessagePact createOrderEventPact(MessagePactBuilder builder) {
        return builder
            .given("order is created")
            .expectsToReceive("order created event")
            .withContent(new PactDslJsonBody()
                .uuid("orderId")
                .stringType("customerId")
                .decimalType("amount")
                .stringValue("status", "CREATED"))
            .toPact();
    }
    
    @Test
    @PactTestFor(pactMethod = "createOrderEventPact")
    void testOrderCreatedEvent(List<Message> messages) throws Exception {
        String messageBody = messages.get(0).getContents().valueAsString();
        OrderEvent event = objectMapper.readValue(messageBody, OrderEvent.class);
        
        eventHandler.handle(event);
        
        verify(notificationService).sendNotification(any());
    }
}
```

### Message Pact Provider

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
            "CREATED"
        );
        return objectMapper.writeValueAsString(event);
    }
}
```

### Headers and Authentication

```java
@Pact(consumer = "WebApp")
public RequestResponsePact authenticatedPact(PactDslWithProvider builder) {
    return builder
        .given("user is authenticated")
        .uponReceiving("authenticated request")
            .path("/api/protected")
            .method("GET")
            .headers(Map.of(
                "Authorization", "Bearer token123",
                "Accept", "application/json"))
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .stringType("data"))
        .toPact();
}
```
