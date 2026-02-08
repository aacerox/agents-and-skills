---
name: "junit5"
description: "JUnit 5 (Jupiter) is the modern testing framework for Java, providing annotations, assertions, and lifecycle management for writing effective unit tests"
---

# JUnit 5 Testing Framework
Use this skill to create the structure and lifecycle of test classes and methods. ALWAYS use JUnit 5 as the **foundation for all Java tests**, providing test annotations (@Test, @BeforeEach, @AfterEach), parameterized tests, nested test organization, and test execution control.

## Quick Reference

- **[Basic Reference](resources/junit5_basic_reference.md)**: Most commonly used features including lifecycle annotations (@Test, @BeforeEach, @AfterEach), parameterized tests (@ValueSource, @CsvSource, @MethodSource), nested tests, basic assertions, exception testing, and repeated tests
- **[Advanced Reference](resources/junit5_advanced_reference.md)**: Advanced features including @EnumSource, @CsvFileSource, conditional test execution, dynamic tests, timeout assertions, test instance lifecycle, test execution order, custom extensions, and complex parameterized tests

## Creating Test

When creating tests with JUnit 5:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.
- **NEVER use fully qualified references for types used in tests**.
  
- **ALWAYS use basic reference** for standard test scenarios:
  - Use `@Test` annotation for test methods
  - Use `@BeforeEach` and `@AfterEach` for setup and teardown
  - Use `@DisplayName` for readable test names
  - Use `@ParameterizedTest` with `@ValueSource`, `@CsvSource`, or `@MethodSource` for data-driven tests
  - Use standard assertions: `assertEquals()`, `assertTrue()`, `assertThrows()`
  - Use `@Nested` classes for organizing related tests
  - Use `@RepeatedTest` for tests that need multiple executions

- **Use advanced reference ONLY when tests require it** for complexity:
  - Conditional execution based on OS, JRE, or environment variables
  - Dynamic test generation with `@TestFactory`
  - Custom timeout behaviors with `assertTimeoutPreemptively()`
  - Shared test instance lifecycle with `@TestInstance`
  - Ordered test execution with `@TestMethodOrder`
  - Custom extensions for cross-cutting concerns

## Modifying Test

When modifying existing JUnit 5 tests:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.
- **NEVER use fully qualified references for types used in tests**.
  
- **ALWAYS use basic reference** for standard modifications:
  - Add new test methods with `@Test`
  - Enhance parameterized tests with additional data sources
  - Add setup/teardown logic with lifecycle annotations
  - Improve assertions with `assertAll()` for grouped assertions
  - Add `@DisplayName` for clarity
  - Organize tests with `@Nested` classes

- **Use advanced reference ONLY when tests require it** for complexity:
  - Add conditional execution for platform-specific tests
  - Generate dynamic tests based on runtime data
  - Implement custom extensions for specialized behavior
  - Configure test execution order for integration scenarios
  - Add timeout constraints for performance tests

## Best Practices

**ALWAYS follow these best practices when using JUnit 5:**

1. **Use descriptive test names** with `@DisplayName` annotation
   - Format: "Should [expected behavior] when [condition]"
   - Example: `@DisplayName("Should return user when valid ID provided")`

2. **Follow AAA pattern** (Arrange-Act-Assert)
   - Separate setup, execution, and verification clearly
   - Use blank lines to separate sections

3. **Use parameterized tests** for multiple scenarios
   - Avoid duplicating test logic
   - Use `@ParameterizedTest` with appropriate source annotations

4. **Leverage nested tests** for organization
   - Group related tests with `@Nested` classes
   - Share setup code within nested classes

5. **Test one behavior per method**
   - Keep tests focused and simple
   - Use `assertAll()` only when assertions verify the same behavior

6. **Use appropriate assertions**
   - Use `assertEquals()` for value comparison
   - Use `assertThrows()` for exception testing
   - Use `assertAll()` for multiple related assertions

7. **Avoid test interdependencies**
   - Each test should be independent
   - Clean up resources in `@AfterEach`

8. **Use lifecycle annotations appropriately**
   - `@BeforeAll`/`@AfterAll` for expensive setup (static methods)
   - `@BeforeEach`/`@AfterEach` for test isolation

9. **Disable tests properly**
   - Use `@Disabled` with a reason: `@Disabled("Temporarily disabled due to bug #123")`
   - Don't comment out tests

10. **Set timeouts for long-running tests**
    - Use `@Timeout` annotation to prevent hanging tests
    - Example: `@Timeout(5)` for 5-second limit

## Common Patterns

### Standard Test Structure
```java
@DisplayName("User Service Tests")
class UserServiceTest {
    
    private UserService userService;
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository = mock(UserRepository.class);
        userService = new UserService(userRepository);
    }
    
    @Test
    @DisplayName("Should create user when valid data provided")
    void shouldCreateUserWhenValidDataProvided() {
        // Arrange
        User user = new User("john@example.com", "John Doe");
        when(userRepository.save(any())).thenReturn(user);
        
        // Act
        User created = userService.createUser(user);
        
        // Assert
        assertThat(created).isNotNull();
        verify(userRepository).save(user);
    }
}
```

### Parameterized Test Pattern
```java
@ParameterizedTest
@CsvSource({
    "john@example.com, true",
    "invalid-email, false",
    "test@test, false"
})
@DisplayName("Should validate email format")
void shouldValidateEmailFormat(String email, boolean expected) {
    boolean result = validator.isValidEmail(email);
    assertEquals(expected, result);
}
```

### Nested Test Pattern
```java
@DisplayName("Shopping Cart Tests")
class ShoppingCartTest {
    
    private ShoppingCart cart;
    
    @BeforeEach
    void setUp() {
        cart = new ShoppingCart();
    }
    
    @Nested
    @DisplayName("When cart is empty")
    class WhenEmpty {
        
        @Test
        @DisplayName("Should have zero total")
        void shouldHaveZeroTotal() {
            assertEquals(0, cart.getTotal());
        }
        
        @Test
        @DisplayName("Should throw exception when checking out")
        void shouldThrowWhenCheckingOut() {
            assertThrows(EmptyCartException.class, () -> cart.checkout());
        }
    }
    
    @Nested
    @DisplayName("When cart has items")
    class WhenHasItems {
        
        @BeforeEach
        void addItems() {
            cart.addItem(new Item("Product", 10.0));
        }
        
        @Test
        @DisplayName("Should calculate correct total")
        void shouldCalculateTotal() {
            assertEquals(10.0, cart.getTotal());
        }
    }
}
```

### Exception Testing Pattern
```java
@Test
@DisplayName("Should throw IllegalArgumentException when email is invalid")
void shouldThrowExceptionWhenEmailInvalid() {
    String invalidEmail = "not-an-email";
    
    IllegalArgumentException exception = assertThrows(
        IllegalArgumentException.class,
        () -> userService.createUser(new User(invalidEmail, "Name"))
    );
    
    assertThat(exception.getMessage()).contains("Invalid email format");
}
```

### Method Source Pattern
```java
@ParameterizedTest
@MethodSource("provideUserScenarios")
@DisplayName("Should validate user based on various scenarios")
void shouldValidateUser(User user, boolean expectedValid) {
    boolean isValid = validator.validate(user);
    assertEquals(expectedValid, isValid);
}

static Stream<Arguments> provideUserScenarios() {
    return Stream.of(
        Arguments.of(new User("john@example.com", 25), true),
        Arguments.of(new User("invalid", 25), false),
        Arguments.of(new User("john@example.com", -1), false)
    );
}
```

### Grouped Assertions Pattern
```java
@Test
@DisplayName("Should create user with correct attributes")
void shouldCreateUserWithCorrectAttributes() {
    User user = userService.createUser("john@example.com", "John", 30);
    
    assertAll("user attributes",
        () -> assertNotNull(user.getId(), "ID should not be null"),
        () -> assertEquals("john@example.com", user.getEmail(), "Email should match"),
        () -> assertEquals("John", user.getName(), "Name should match"),
        () -> assertEquals(30, user.getAge(), "Age should match"),
        () -> assertNotNull(user.getCreatedAt(), "Created date should be set")
    );
}
```

## Resources

- [JUnit 5 Official Documentation](https://junit.org/junit5/docs/current/user-guide/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [JUnit 5 GitHub Repository](https://github.com/junit-team/junit5)
- [JUnit 5 API Documentation](https://junit.org/junit5/docs/current/api/)
