---
name: "mockito"
description: "Mockito is a powerful mocking framework for Java that allows you to create test doubles, stub method behavior, and verify interactions in unit tests"
---

# Mockito Mocking Framework
Use this skill to create mock objects and stub behavior for dependencies in unit tests. ALWAYS use Mockito when you need to **isolate the class under test** from its dependencies (repositories, services, external APIs) to test business logic without requiring real implementations.

## Quick Reference

- **[Basic Reference](resources/mockito_basic_reference.md)**: Most commonly used features including creating mocks with @Mock and @InjectMocks, basic stubbing with when/thenReturn, throwing exceptions, void methods, argument matchers (any, eq), basic verification, argument captors, verification order, and BDD style
- **[Advanced Reference](resources/mockito_advanced_reference.md)**: Advanced features including custom matchers, combining matchers, multiple argument captures, spies, custom answers, answer shortcuts, deep stubs, mock settings, resetting mocks, consecutive calls, and callbacks

## Creating Test

When creating tests with Mockito:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard test scenarios:
  - Use `@Mock` to create mock objects
  - Use `@InjectMocks` to inject mocks into the class under test
  - Use `@ExtendWith(MockitoExtension.class)` for JUnit 5 integration
  - Use `when().thenReturn()` for stubbing method return values
  - Use `verify()` to verify method invocations
  - Use argument matchers: `any()`, `eq()`, `anyString()`, `anyInt()`
  - Use `ArgumentCaptor` to capture and inspect method arguments
  - Use `doThrow()` and `doNothing()` for void methods

- **Use advanced reference ONLY when tests require it** for complexity:
  - Custom argument matchers with `argThat()`
  - Spies with `@Spy` for partial mocking
  - Custom answers with `thenAnswer()`
  - Deep stubs for complex object chains (discouraged)
  - Resetting mocks mid-test
  - Consecutive calls with different return values

## Modifying Test

When modifying existing Mockito tests:

- **ALWAYS add the required imports** related to the framework.
- **NEVER use wildcard imports (*)** - always import specific classes and methods.

- **ALWAYS use basic reference** for standard modifications:
  - Add new mocks with `@Mock` annotation
  - Add stubbing for new scenarios with `when().thenReturn()`
  - Add verification for new interactions with `verify()`
  - Update argument matchers for broader scenarios
  - Add argument captors to inspect complex arguments
  - Add `InOrder` verification for ordered interactions

- **Use advanced reference ONLY when tests require it** for complexity:
  - Add custom matchers for complex validation logic
  - Convert mocks to spies for partial behavior
  - Add custom answers for dynamic return values
  - Add multiple captures for repeated invocations
  - Add consecutive stubbing for stateful behavior

## Best Practices

**ALWAYS follow these best practices when using Mockito:**

1. **Use @ExtendWith(MockitoExtension.class)** for JUnit 5
   - Enables automatic mock initialization
   - Cleaner than manual `MockitoAnnotations.openMocks()`

2. **Prefer @Mock over mock() method**
   - More readable and declarative
   - Automatic initialization with extension

3. **Use @InjectMocks wisely**
   - Only for the class under test
   - Ensures mocks are properly injected

4. **Don't mock value objects**
   - Mock only interfaces and complex dependencies
   - Use real objects for DTOs, entities, value objects

5. **Verify only what matters**
   - Don't over-verify trivial interactions
   - Focus on critical method calls

6. **Use argument matchers consistently**
   - Either all matchers or all concrete values
   - Mix only with `eq()`: `verify(service, times(1)).process(eq(1L), any())`

7. **Avoid deep stubs**
   - Indicates design problems
   - Refactor code instead of using `RETURNS_DEEP_STUBS`

8. **Use BDD style for readability**
   - `given().willReturn()` for setup
   - `then().should()` for verification
   - More readable in BDD-style tests

9. **Don't mock everything**
   - Only mock external dependencies
   - Test real business logic without mocks

10. **Reset mocks sparingly**
    - Usually indicates poor test design
    - Each test should have independent setup

## Common Patterns

### Standard Mock Setup Pattern
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser() {
        // Arrange
        User user = new User("john@example.com");
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // Act
        User created = userService.createUser(user);
        
        // Assert
        assertThat(created).isNotNull();
        verify(userRepository).save(user);
        verify(emailService).sendWelcomeEmail(user.getEmail());
    }
}
```

### Exception Stubbing Pattern
```java
@Test
void shouldHandleRepositoryException() {
    // Arrange
    when(userRepository.findById(999L))
        .thenThrow(new EntityNotFoundException("User not found"));
    
    // Act & Assert
    assertThatThrownBy(() -> userService.getUser(999L))
        .isInstanceOf(EntityNotFoundException.class)
        .hasMessageContaining("User not found");
    
    verify(userRepository).findById(999L);
}
```

### Argument Captor Pattern
```java
@Test
void shouldSaveUserWithCorrectData() {
    // Arrange
    ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
    UserRequest request = new UserRequest("john@example.com", "John Doe");
    
    // Act
    userService.createUser(request);
    
    // Assert
    verify(userRepository).save(userCaptor.capture());
    
    User capturedUser = userCaptor.getValue();
    assertThat(capturedUser.getEmail()).isEqualTo("john@example.com");
    assertThat(capturedUser.getName()).isEqualTo("John Doe");
    assertThat(capturedUser.getCreatedAt()).isNotNull();
}
```

### Verification Order Pattern
```java
@Test
void shouldExecuteStepsInOrder() {
    // Arrange
    User user = new User("john@example.com");
    InOrder inOrder = inOrder(userRepository, emailService, auditService);
    
    // Act
    userService.registerUser(user);
    
    // Assert
    inOrder.verify(userRepository).save(user);
    inOrder.verify(emailService).sendWelcomeEmail(user.getEmail());
    inOrder.verify(auditService).logRegistration(user.getId());
}
```

### BDD Style Pattern
```java
@Test
void shouldGetUserInBDDStyle() {
    // Given
    User user = new User(1L, "John Doe");
    given(userRepository.findById(1L)).willReturn(Optional.of(user));
    
    // When
    User result = userService.getUser(1L);
    
    // Then
    then(userRepository).should().findById(1L);
    then(userRepository).shouldHaveNoMoreInteractions();
    assertThat(result.getName()).isEqualTo("John Doe");
}
```

### Void Method Pattern
```java
@Test
void shouldHandleVoidMethod() {
    // Arrange - successful case
    doNothing().when(emailService).sendEmail(anyString());
    
    // Act
    userService.notifyUser("john@example.com");
    
    // Assert
    verify(emailService).sendEmail("john@example.com");
}

@Test
void shouldHandleVoidMethodException() {
    // Arrange - exception case
    doThrow(new EmailException("SMTP error"))
        .when(emailService).sendEmail(anyString());
    
    // Act & Assert
    assertThatThrownBy(() -> userService.notifyUser("john@example.com"))
        .isInstanceOf(EmailException.class);
}
```

### Argument Matcher Pattern
```java
@Test
void shouldUseArgumentMatchers() {
    // Arrange
    when(userRepository.findByEmail(anyString()))
        .thenReturn(Optional.of(new User()));
    
    when(userRepository.findByAgeGreaterThan(gt(18)))
        .thenReturn(List.of(new User()));
    
    when(userRepository.update(eq(1L), any(User.class)))
        .thenReturn(true);
    
    // Act
    userService.processUsers();
    
    // Assert
    verify(userRepository).findByEmail(anyString());
    verify(userRepository).findByAgeGreaterThan(anyInt());
}
```

### Multiple Verifications Pattern
```java
@Test
void shouldVerifyMultipleInteractions() {
    // Arrange
    List<User> users = List.of(
        new User("user1@example.com"),
        new User("user2@example.com")
    );
    
    // Act
    userService.notifyUsers(users);
    
    // Assert
    verify(emailService, times(2)).sendEmail(anyString());
    verify(emailService).sendEmail("user1@example.com");
    verify(emailService).sendEmail("user2@example.com");
    verify(emailService, never()).sendSMS(anyString());
    verifyNoMoreInteractions(emailService);
}
```

## Resources

- [Mockito Official Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Mockito GitHub Repository](https://github.com/mockito/mockito)
- [Mockito Wiki](https://github.com/mockito/mockito/wiki)
- [Mockito Javadoc](https://javadoc.io/doc/org.mockito/mockito-core/latest/index.html)
