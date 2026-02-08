# AssertJ Advanced Reference

## Advanced Features

### Flat Extracting

```java
@Test
void testFlatExtracting() {
    List<Order> orders = Arrays.asList(
        new Order(List.of("item1", "item2")),
        new Order(List.of("item3"))
    );
    
    assertThat(orders)
        .flatExtracting(Order::getItems)
        .containsExactly("item1", "item2", "item3");
}
```

### Complex Filtering

```java
@Test
void testComplexFiltering() {
    List<User> users = getUsers();
    
    assertThat(users)
        .filteredOn(user -> user.isActive() && user.getAge() > 25)
        .hasSize(2)
        .extracting(User::getName)
        .contains("Alice", "Bob");
}
```

### Recursive Comparison

```java
@Test
void testRecursiveComparison() {
    User user1 = new User("John", 30, 
        new Address("123 Main St", "New York"));
    User user2 = new User("John", 30, 
        new Address("123 Main St", "New York"));
    
    assertThat(user1)
        .usingRecursiveComparison()
        .isEqualTo(user2);
}
```

### Ignoring Fields in Recursive Comparison

```java
@Test
void testRecursiveComparisonIgnoringFields() {
    User user1 = new User(1L, "John", 30);
    User user2 = new User(2L, "John", 30);
    
    assertThat(user1)
        .usingRecursiveComparison()
        .ignoringFields("id")
        .isEqualTo(user2);
}
```

### Custom Assertions

```java
class UserAssert extends AbstractAssert<UserAssert, User> {
    
    public UserAssert(User user) {
        super(user, UserAssert.class);
    }
    
    public static UserAssert assertThat(User actual) {
        return new UserAssert(actual);
    }
    
    public UserAssert hasName(String name) {
        isNotNull();
        if (!actual.getName().equals(name)) {
            failWithMessage("Expected user name to be <%s> but was <%s>", 
                name, actual.getName());
        }
        return this;
    }
    
    public UserAssert isAdult() {
        isNotNull();
        if (actual.getAge() < 18) {
            failWithMessage("Expected user to be adult but age was <%s>", 
                actual.getAge());
        }
        return this;
    }
}

@Test
void testCustomAssertion() {
    User user = new User("John", 30);
    
    UserAssert.assertThat(user)
        .hasName("John")
        .isAdult();
}
```

### Satisfying Conditions

```java
@Test
void testCondition() {
    User user = new User("john@example.com", 30);
    
    Condition<User> validEmail = new Condition<>(
        u -> u.getEmail().contains("@"),
        "valid email"
    );
    
    assertThat(user).is(validEmail);
    assertThat(user).has(validEmail);
}

@Test
void testMultipleConditions() {
    List<User> users = getUsers();
    
    assertThat(users)
        .areAtLeastOne(new Condition<>(u -> u.getAge() > 25, "older than 25"))
        .areAtMost(2, new Condition<>(u -> u.isAdmin(), "admin"))
        .haveExactly(1, new Condition<>(u -> u.getName().equals("Alice"), "named Alice"));
}
```

### Assumptions

```java
@Test
void testWithAssumptions() {
    String env = System.getenv("ENV");
    
    assumeThat(env).isEqualTo("production");
    
    // Test only runs in production environment
    User user = userService.getUser(1L);
    assertThat(user).isNotNull();
}
```

### Describing Assertions

```java
@Test
void testWithDescription() {
    User user = new User("John", 30);
    
    assertThat(user.getAge())
        .as("User age should be positive")
        .isGreaterThan(0);
    
    assertThat(user.getName())
        .as("check %s's name", "user")
        .isEqualTo("John");
}
```

### Exception with Cause

```java
@Test
void testExceptionWithCause() {
    assertThatThrownBy(() -> {
        throw new RuntimeException("Error", new IOException("File not found"));
    })
        .isInstanceOf(RuntimeException.class)
        .hasMessage("Error")
        .hasCauseInstanceOf(IOException.class)
        .hasRootCauseMessage("File not found");
}
```

### Instant/Timestamp Assertions

```java
@Test
void testInstantAssertions() {
    Instant now = Instant.now();
    Instant oneHourAgo = now.minus(1, ChronoUnit.HOURS);
    
    assertThat(now)
        .isAfter(oneHourAgo)
        .isBeforeOrEqualTo(Instant.now())
        .isCloseTo(Instant.now(), within(1, ChronoUnit.SECONDS));
}
```

### Set Assertions

```java
@Test
void testSetAssertions() {
    Set<String> tags = Set.of("java", "testing", "junit");
    
    assertThat(tags)
        .hasSize(3)
        .contains("java")
        .containsExactlyInAnyOrder("junit", "testing", "java")
        .doesNotContain("mockito");
}
```

### Satisfies (Lambda Assertions)

```java
@Test
void testSatisfies() {
    List<Order> orders = getOrders();
    
    assertThat(orders)
        .first()
        .satisfies(order -> {
            assertThat(order.getCustomerId()).isEqualTo("CUST-123");
            assertThat(order.getTotalAmount()).isGreaterThan(BigDecimal.ZERO);
            assertThat(order.getItems()).isNotEmpty();
        });
}
```
