---
name: "assertj"
description: "AssertJ is a fluent assertion library for Java that provides a rich set of assertions and intuitive API for writing readable and maintainable test assertions"
---

# AssertJ Fluent Assertions
Use this skill to write fluent, readable assertions in tests. **ALWAYS prefer AssertJ assertions over JUnit assertions** when creating or modifying test assertions for better readability and more expressive error messages.

## Quick Reference

- **[Basic Reference](resources/assertj_basic_reference.md)**: Most commonly used features including object assertions, string assertions, numeric assertions, boolean assertions, list assertions, map assertions, exception assertions, extracting from collections, filtering collections, date/time assertions, and soft assertions
- **[Advanced Reference](resources/assertj_advanced_reference.md)**: Advanced features including flat extracting, complex filtering, recursive comparison, ignoring fields in comparisons, custom assertions, satisfying conditions, assumptions, describing assertions, exception with cause, instant/timestamp assertions, set assertions, and lambda assertions

## Creating Test

When creating tests with AssertJ:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard test scenarios:
  - Use `assertThat()` as the entry point for all assertions
  - Use string assertions: `contains()`, `startsWith()`, `endsWith()`, `isEqualTo()`
  - Use numeric assertions: `isGreaterThan()`, `isLessThan()`, `isBetween()`, `isCloseTo()`
  - Use collection assertions: `hasSize()`, `contains()`, `containsExactly()`, `isEmpty()`
  - Use exception assertions: `assertThatThrownBy()`, `isInstanceOf()`, `hasMessage()`
  - Use `extracting()` to pull fields from collections
  - Use `filteredOn()` to filter collection elements
  - Use `SoftAssertions` for multiple assertions without stopping on first failure

- **Use advanced reference ONLY when tests require it** for complexity:
  - Recursive comparison for deep object equality
  - Flat extracting for nested collections
  - Custom assertions for domain-specific validations
  - Condition-based assertions with `satisfies()`
  - Complex filtering with multiple predicates
  - Describing assertions for better error messages

## Modifying Test

When modifying existing AssertJ tests:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard modifications:
  - Replace standard JUnit assertions with fluent AssertJ equivalents
  - Add string matching assertions for better readability
  - Add collection assertions for list/set/map validation
  - Add `extracting()` to simplify collection element assertions
  - Add `SoftAssertions` to group related assertions
  - Add exception assertions with better error messages

- **Use advanced reference ONLY when tests require it** for complexity:
  - Add recursive comparison for complex object graphs
  - Add custom assertions for repeated validation patterns
  - Add flat extracting for nested collection scenarios
  - Add conditional assertions with `satisfies()`
  - Add field-ignoring comparison for partial equality

## Best Practices

**ALWAYS follow these best practices when using AssertJ:**

1. **Use assertThat() consistently**
   - Single entry point for all assertions
   - More readable than traditional assertions

2. **Chain assertions for readability**
   - Multiple assertions on same object: `assertThat(text).isNotNull().startsWith("Hello")`
   - Improves test readability

3. **Use extracting() for collections**
   - Extract fields before asserting
   - More concise than loops or streams

4. **Use specific assertions**
   - Prefer `hasSize(3)` over `isEqualTo(3)` for collections
   - Better error messages

5. **Use SoftAssertions for grouped checks**
   - Collect all failures before reporting
   - Better for validating multiple properties

6. **Use descriptive error messages with as()**
   - Add context to assertions
   - Example: `assertThat(age).as("User age should be positive").isGreaterThan(0)`

7. **Use extracting() with multiple fields**
   - Extract multiple properties at once
   - Use `tuple()` for verification

8. **Use filteredOn() instead of manual filtering**
   - Built-in filtering is more readable
   - Better error messages

9. **Use recursive comparison for deep equality**
   - Avoid manual field-by-field comparison
   - Simpler and more maintainable

10. **Avoid over-chaining**
    - Keep assertion chains readable
    - Split into multiple statements if too complex

## Common Patterns

### Basic Assertion Pattern
```java
@Test
void shouldValidateUser() {
    User user = userService.getUser(1L);
    
    assertThat(user)
        .isNotNull()
        .extracting(User::getName, User::getEmail, User::getAge)
        .containsExactly("John Doe", "john@example.com", 30);
}
```

### Collection Assertion Pattern
```java
@Test
void shouldValidateUserList() {
    List<User> users = userService.getAllUsers();
    
    assertThat(users)
        .isNotEmpty()
        .hasSize(3)
        .extracting(User::getName)
        .containsExactly("Alice", "Bob", "Carol");
}
```

### Exception Assertion Pattern
```java
@Test
void shouldThrowExceptionForInvalidUser() {
    assertThatThrownBy(() -> userService.getUser(999L))
        .isInstanceOf(EntityNotFoundException.class)
        .hasMessage("User not found with id: 999")
        .hasMessageContaining("User not found")
        .hasNoCause();
}
```

### Extracting and Filtering Pattern
```java
@Test
void shouldFilterAndExtractUsers() {
    List<User> users = userService.getAllUsers();
    
    assertThat(users)
        .filteredOn(user -> user.getAge() > 25)
        .extracting(User::getName)
        .containsExactly("Bob", "Carol");
}
```

### Soft Assertions Pattern
```java
@Test
void shouldValidateAllUserProperties() {
    User user = userService.getUser(1L);
    
    SoftAssertions.assertSoftly(softly -> {
        softly.assertThat(user.getName()).isEqualTo("John Doe");
        softly.assertThat(user.getEmail()).isEqualTo("john@example.com");
        softly.assertThat(user.getAge()).isGreaterThan(18);
        softly.assertThat(user.isActive()).isTrue();
        softly.assertThat(user.getRoles()).contains("USER");
    });
}
```

### String Assertion Pattern
```java
@Test
void shouldValidateEmailFormat() {
    String email = user.getEmail();
    
    assertThat(email)
        .isNotBlank()
        .contains("@")
        .matches("^[\\w.-]+@[\\w.-]+\\.\\w+$")
        .endsWith(".com")
        .doesNotContain(" ");
}
```

### Map Assertion Pattern
```java
@Test
void shouldValidateUserMetadata() {
    Map<String, Object> metadata = user.getMetadata();
    
    assertThat(metadata)
        .isNotEmpty()
        .hasSize(3)
        .containsKeys("lastLogin", "loginCount", "preferences")
        .containsEntry("loginCount", 5)
        .doesNotContainKey("deleted");
}
```

### Tuple Extraction Pattern
```java
@Test
void shouldValidateMultipleFields() {
    List<Order> orders = orderService.getOrders();
    
    assertThat(orders)
        .extracting("orderId", "status", "totalAmount")
        .contains(
            tuple("ORD-001", OrderStatus.CONFIRMED, new BigDecimal("99.99")),
            tuple("ORD-002", OrderStatus.PENDING, new BigDecimal("49.99"))
        );
}
```

### Nested Property Extraction Pattern
```java
@Test
void shouldExtractNestedProperties() {
    List<Order> orders = orderService.getOrders();
    
    assertThat(orders)
        .flatExtracting(Order::getItems)
        .extracting(Item::getName)
        .contains("Product A", "Product B", "Product C");
}
```

### Numeric Assertion Pattern
```java
@Test
void shouldValidateOrderTotal() {
    BigDecimal total = order.calculateTotal();
    
    assertThat(total)
        .isNotNull()
        .isGreaterThan(BigDecimal.ZERO)
        .isLessThan(new BigDecimal("1000.00"))
        .isBetween(new BigDecimal("50.00"), new BigDecimal("500.00"));
}
```

### Date/Time Assertion Pattern
```java
@Test
void shouldValidateOrderTimestamp() {
    LocalDateTime orderTime = order.getCreatedAt();
    
    assertThat(orderTime)
        .isNotNull()
        .isBefore(LocalDateTime.now())
        .isAfter(LocalDateTime.now().minusDays(1))
        .hasYear(2026)
        .hasMonthValue(2);
}
```

## Resources

- [AssertJ Official Documentation](https://assertj.github.io/doc/)
- [AssertJ Core Documentation](https://assertj.github.io/doc/#assertj-core)
- [AssertJ GitHub Repository](https://github.com/assertj/assertj)
- [AssertJ Javadoc](https://javadoc.io/doc/org.assertj/assertj-core/latest/index.html)
