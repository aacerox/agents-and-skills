---
name: "instancio"
description: "Instancio is a Java library for automatically generating test objects with realistic data, eliminating boilerplate test code and making tests more maintainable"
---

# Instancio Test Fixture Generator
Use this skill to automatically generate test data objects with realistic values. ALWAYS use Instancio when you need to **create complex test objects without writing boilerplate builders** or manual construction code, reducing test setup and improving maintainability.

## Quick Reference

- **[Basic Reference](resources/instancio_basic_reference.md)**: Most commonly used features including simple object creation, creating multiple objects, setting specific values, generating values, supplying custom values, field selectors, type selectors, collection configuration, reusable models, and built-in generators
- **[Advanced Reference](resources/instancio_advanced_reference.md)**: Advanced features including predicate selectors, null handling, subtype mapping, nested object customization, conditional generation, map configuration, enum generators, JUnit 5 integration, parameterized tests, test data builders, and collection size constraints

## Creating Test

When creating tests with Instancio:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard test scenarios:
  - Use `Instancio.create(Class)` for simple object creation
  - Use `Instancio.ofList(Class).size(n).create()` for collections
  - Use `.set()` for specific fixed values
  - Use `.generate()` for controlled random values
  - Use `.supply()` for custom value generation logic
  - Use `field()` selector for specific field targeting
  - Use `all()` selector for type-based targeting
  - Use `.toModel()` for reusable test data templates
  - Use built-in generators: `gen.string()`, `gen.ints()`, `gen.temporal()`

- **Use advanced reference ONLY when tests require it** for complexity:
  - Predicate selectors with `fields().matching()`
  - Null handling with `withNullable()` and `setNull()`
  - Subtype mapping with `.subtype()` for interfaces
  - Nested object customization with `.within(scope())`
  - Conditional generation with `.onComplete()`
  - JUnit 5 integration with `@Random` annotation
  - Complex parameterized tests with Instancio streams

## Modifying Test

When modifying existing Instancio tests:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard modifications:
  - Add customization with `.set()` or `.generate()`
  - Modify collection sizes with `.generate(field(), gen -> gen.collection().size())`
  - Add new fields to customize with field selectors
  - Extract common patterns into Models with `.toModel()`
  - Add type-level customization with `all()` selector
  - Add specific generators for better test data quality

- **Use advanced reference ONLY when tests require it** for complexity:
  - Add predicate selectors for pattern-based field matching
  - Add subtype mapping for polymorphic objects
  - Add nested customization with scoped selectors
  - Add conditional logic with `.onComplete()`
  - Add JUnit 5 integration for cleaner tests
  - Add reproducible tests with `@Seed` annotation

## Best Practices

**ALWAYS follow these best practices when using Instancio:**

1. **Create reusable Models for common scenarios**
   - Define Models as static constants
   - Compose Models from other Models
   - Reduces duplication across tests

2. **Prefer generate() over set() for flexibility**
   - `generate()` creates varied data across test runs
   - `set()` only for truly fixed values
   - Better test coverage with varied data

3. **Use specific selectors to avoid side effects**
   - Use `field()` for precise targeting
   - Avoid overly broad `all()` selectors
   - Scope selectors with `.within()` for nested objects

4. **Leverage built-in generators**
   - Use `gen.string().length()` for controlled strings
   - Use `gen.ints().range()` for bounded numbers
   - Use `gen.temporal().past()` for historical dates
   - Better data quality than random values

5. **Keep test data realistic**
   - Use appropriate generators for domain context
   - Example: `gen.text().pattern("#a#a#a@example.com")` for emails
   - Helps catch real-world bugs

6. **Don't over-customize**
   - Only customize what matters for the test
   - Let Instancio handle the rest
   - Keeps tests focused and readable

7. **Use Models for test data organization**
   - Create a `TestData` or `Fixtures` class
   - Group related Models together
   - Improves test maintainability

8. **Use @Seed for reproducible failing tests**
   - Add `@Seed` when debugging flaky tests
   - Document seed value in bug reports
   - Remove after fixing to restore randomness

9. **Combine with other frameworks effectively**
   - Use Instancio for test data generation
   - Use Mockito for behavior stubbing
   - Use AssertJ for assertions
   - Each tool for its strength

10. **Use streams for parameterized tests**
    - Generate multiple test cases with `Instancio.stream()`
    - Better coverage than hand-written cases
    - Reduces test code

## Common Patterns

### Test Data Builder Replacement Pattern
```java
class UserTestData {
    static final Model<User> ACTIVE_USER = Instancio.of(User.class)
        .set(field(User::getStatus), UserStatus.ACTIVE)
        .generate(field(User::getEmail), gen -> gen.text().pattern("#a#a#a#a@test.com"))
        .generate(field(User::getAge), gen -> gen.ints().range(18, 65))
        .toModel();
    
    static final Model<User> ADMIN_USER = Instancio.of(ACTIVE_USER)
        .set(field(User::getRole), Role.ADMIN)
        .toModel();
    
    static final Model<User> PREMIUM_USER = Instancio.of(ACTIVE_USER)
        .set(field(User::getSubscription), SubscriptionType.PREMIUM)
        .toModel();
}

@Test
void shouldCreateActiveUser() {
    User user = Instancio.create(UserTestData.ACTIVE_USER);
    
    assertThat(user.getStatus()).isEqualTo(UserStatus.ACTIVE);
    assertThat(user.getEmail()).matches("\\w{4}@test.com");
}
```

### Simple Test Data Pattern
```java
@Test
void shouldProcessOrder() {
    // Arrange
    Order order = Instancio.of(Order.class)
        .set(field(Order::getStatus), OrderStatus.PENDING)
        .generate(field(Order::getItems), gen -> gen.collection().size(3))
        .create();
    
    // Act
    orderService.processOrder(order);
    
    // Assert
    assertThat(order.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
}
```

### Collection Test Data Pattern
```java
@Test
void shouldProcessMultipleOrders() {
    // Arrange
    List<Order> orders = Instancio.ofList(Order.class)
        .size(5)
        .generate(field(Order::getStatus), 
            gen -> gen.enumOf(OrderStatus.class).excluding(OrderStatus.CANCELLED))
        .create();
    
    // Act
    List<Order> processed = orderService.processOrders(orders);
    
    // Assert
    assertThat(processed).hasSize(5);
    assertThat(processed).allMatch(order -> order.getStatus() != OrderStatus.CANCELLED);
}
```

### Nested Object Customization Pattern
```java
@Test
void shouldCreateOrderWithShippingAddress() {
    Order order = Instancio.of(Order.class)
        .set(field(Address::getCountry).within(scope(Order::getShippingAddress)), "USA")
        .set(field(Address::getCity).within(scope(Order::getShippingAddress)), "New York")
        .generate(field(Item::getPrice).within(scope(Order::getItems)), 
            gen -> gen.bigDecimal().min(10).max(1000))
        .create();
    
    assertThat(order.getShippingAddress().getCountry()).isEqualTo("USA");
    assertThat(order.getItems())
        .allMatch(item -> item.getPrice().compareTo(BigDecimal.TEN) >= 0);
}
```

### Parameterized Test Pattern
```java
@ParameterizedTest
@MethodSource("provideUsers")
void shouldValidateUser(User user) {
    boolean isValid = userValidator.validate(user);
    assertTrue(isValid);
}

static Stream<User> provideUsers() {
    return Instancio.stream(User.class)
        .generate(field(User::getAge), gen -> gen.ints().range(18, 100))
        .generate(field(User::getEmail), gen -> gen.text().pattern("#a#a#a@test.com"))
        .limit(10);
}
```

### JUnit 5 Extension Pattern
```java
@ExtendWith(InstancioExtension.class)
class UserServiceTest {
    
    @Test
    void shouldCreateUser(@Random User user) {
        User created = userService.create(user);
        assertThat(created.getId()).isNotNull();
    }
    
    @Test
    void shouldUpdateUser(@Random User user, @Random UserUpdateRequest request) {
        userService.update(user.getId(), request);
        verify(userRepository).save(any(User.class));
    }
}
```

### Supplier Pattern for Sequential Data
```java
@Test
void shouldCreateUsersWithSequentialIds() {
    AtomicLong idGenerator = new AtomicLong(1);
    
    List<User> users = Instancio.ofList(User.class)
        .size(5)
        .supply(field(User::getId), () -> idGenerator.getAndIncrement())
        .create();
    
    assertThat(users)
        .extracting(User::getId)
        .containsExactly(1L, 2L, 3L, 4L, 5L);
}
```

### Realistic Email Generation Pattern
```java
@Test
void shouldCreateUsersWithRealisticEmails() {
    List<User> users = Instancio.ofList(User.class)
        .size(10)
        .generate(field(User::getEmail), 
            gen -> gen.text().pattern("#c#c#c#c.#c#c#c#c@example.com"))
        .create();
    
    assertThat(users)
        .extracting(User::getEmail)
        .allMatch(email -> email.matches("^[a-z]{4}\\.[a-z]{4}@example\\.com$"));
}
```

### Complex Domain Model Pattern
```java
@Test
void shouldCreateCompleteOrderWithAllRelations() {
    Order order = Instancio.of(Order.class)
        // Order level
        .set(field(Order::getStatus), OrderStatus.CONFIRMED)
        .generate(field(Order::getOrderDate), gen -> gen.temporal().localDate().past())
        
        // Customer level
        .set(field(Customer::getType).within(scope(Order::getCustomer)), CustomerType.PREMIUM)
        
        // Items level
        .generate(field(Order::getItems), gen -> gen.collection().minSize(2).maxSize(5))
        .generate(field(Item::getQuantity).within(scope(Order::getItems)), 
            gen -> gen.ints().range(1, 10))
        
        // Address level
        .set(field(Address::getCountry).within(scope(Order::getShippingAddress)), "USA")
        
        .create();
    
    assertThat(order.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    assertThat(order.getCustomer().getType()).isEqualTo(CustomerType.PREMIUM);
    assertThat(order.getItems()).hasSizeBetween(2, 5);
}
```

## Resources

- [Instancio Official Website](https://www.instancio.org/)
- [Instancio User Guide](https://www.instancio.org/user-guide/)
- [Instancio GitHub Repository](https://github.com/instancio/instancio)
- [Instancio Javadoc](https://javadoc.io/doc/org.instancio/instancio-core/latest/index.html)
