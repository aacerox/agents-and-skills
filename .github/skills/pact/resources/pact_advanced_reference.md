# Pact.io Advanced Reference

## Advanced Features

### Advanced Matchers

```java
@Pact(consumer = "WebApp")
public RequestResponsePact advancedMatchersPact(PactDslWithProvider builder) {
    return builder
        .given("order exists")
        .uponReceiving("order with advanced matchers")
            .path("/orders/1")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                // Regex matchers
                .stringMatcher("status", "PENDING|CONFIRMED|SHIPPED")
                .stringMatcher("email", ".*@.*\\..*")
                
                // Numeric constraints
                .numberValue("quantity", 1, 100)
                .decimalValue("price", 0.01, 10000.0)
                
                // Min array like
                .minArrayLike("items", 1)
                    .stringType("productId")
                    .integerType("quantity")
                .closeObject()
                .asArray())
        .toPact();
}
```

### Filtering Pacts by Consumer Version

```java
@Provider("UserService")
@PactBroker(
    host = "pact-broker.example.com",
    consumerVersionSelectors = {
        @ConsumerVersionSelector(tag = "main", latest = "true"),
        @ConsumerVersionSelector(tag = "production", latest = "true")
    }
)
class UserProviderTest {
    // Tests only for specified consumer versions
}
```

### Publishing Pacts (Maven)

```xml
<plugin>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>maven</artifactId>
    <version>4.6.0</version>
    <configuration>
        <pactBrokerUrl>https://pact-broker.example.com</pactBrokerUrl>
        <pactBrokerToken>${env.PACT_BROKER_TOKEN}</pactBrokerToken>
        <projectVersion>${project.version}</projectVersion>
        <tags>
            <tag>main</tag>
            <tag>${env.BRANCH_NAME}</tag>
        </tags>
    </configuration>
</plugin>
```

### Publishing Pacts (Gradle)

```gradle
pact {
    broker {
        pactBrokerUrl = 'https://pact-broker.example.com'
        pactBrokerToken = System.getenv('PACT_BROKER_TOKEN')
    }
    publish {
        pactDirectory = 'build/pacts'
        version = project.version
        tags = ['main', System.getenv('BRANCH_NAME')]
    }
}
```

### Can-I-Deploy Checks

```bash
# Check if consumer can be deployed
pact-broker can-i-deploy \
  --pacticipant UserConsumer \
  --version 1.2.3 \
  --to-environment production \
  --broker-base-url https://pact-broker.example.com \
  --broker-token ${PACT_BROKER_TOKEN}

# Check if provider can be deployed
pact-broker can-i-deploy \
  --pacticipant UserService \
  --version 2.1.0 \
  --to-environment production \
  --broker-base-url https://pact-broker.example.com \
  --broker-token ${PACT_BROKER_TOKEN}
```

### Pending Pacts

```java
@Provider("UserService")
@PactBroker(
    host = "pact-broker.example.com",
    enablePendingPacts = "true",
    providerTags = "main"
)
class UserProviderTest {
    // New pacts won't fail provider build
}
```

### Message Metadata

```java
@Pact(consumer = "NotificationService")
public MessagePact createMessageWithMetadata(MessagePactBuilder builder) {
    return builder
        .given("event published")
        .expectsToReceive("event with metadata")
        .withMetadata(Map.of(
            "contentType", "application/json",
            "destination", "orders.created",
            "kafka_topic", "order-events"
        ))
        .withContent(new PactDslJsonBody()
            .uuid("orderId")
            .stringType("status"))
        .toPact();
}
```

### Multiple Interactions in One Pact

```java
@Pact(consumer = "WebApp")
public RequestResponsePact multipleInteractionsPact(PactDslWithProvider builder) {
    return builder
        // Interaction 1
        .given("order exists")
        .uponReceiving("get order request")
            .path("/orders/123")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(orderBody())
        
        // Interaction 2
        .given("products available")
        .uponReceiving("create order request")
            .path("/orders")
            .method("POST")
            .body(createOrderBody())
        .willRespondWith()
            .status(201)
            .body(createdOrderBody())
        
        .toPact();
}
```

### Complex State Setup

```java
@State("database has test data")
void setupTestData() {
    userRepository.deleteAll();
    orderRepository.deleteAll();
    
    User user = new User("test@example.com");
    userRepository.save(user);
    
    Order order = new Order(user.getId(), "PROD-001");
    orderRepository.save(order);
}

@State("no data exists")
void cleanDatabase() {
    userRepository.deleteAll();
    orderRepository.deleteAll();
}
```

### Custom Verification Settings

```java
@Provider("UserService")
@PactBroker(host = "pact-broker.example.com")
class UserProviderTest {
    
    @BeforeEach
    void setUp(PactVerificationContext context) {
        context.setTarget(HttpTestTarget.builder()
            .host("localhost")
            .port(port)
            .path("/api")
            .build());
    }
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction();
    }
}
```

### Pact V4 Features

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "UserService", pactVersion = PactSpecVersion.V4)
class UserConsumerV4Test {
    
    @Pact(consumer = "WebApp")
    public V4Pact createPact(PactBuilder builder) {
        return builder
            .usingLegacyDsl()
            .given("user exists")
            .uponReceiving("get user request")
                .path("/users/1")
                .method("GET")
            .willRespondWith()
                .status(200)
                .body(new PactDslJsonBody()
                    .integerType("id")
                    .stringType("name"))
            .toPact(V4Pact.class);
    }
}
```

### Provider State Cleanup

```java
@State(value = "user exists", action = StateChangeAction.SETUP)
void createUser() {
    User user = new User(1L, "John");
    userRepository.save(user);
}

@State(value = "user exists", action = StateChangeAction.TEARDOWN)
void cleanupUser() {
    userRepository.deleteById(1L);
}
```
