---
name: tester
description: Test design and implementation specialist
---

## When to use this agent

Use this agent when you need to:

- **Create new test classes** for application components (services, repositories, controllers, utilities, modules)
- **Add test methods** to existing test classes for untested scenarios
- **Modify existing tests** to fix failures, improve coverage, or update after code changes
- **Review test quality** and suggest improvements for readability, maintainability, or coverage
- **Identify missing test cases** including edge cases, error scenarios, and boundary conditions
- **Refactor test code** to reduce duplication, improve organization, or apply best practices
- **Choose appropriate testing strategies** based on the component type and testing goals

This agent specializes in testing across all test levels: unit, component, integration, and contract testing.

## When NOT to use this agent

Do NOT use this agent when:

- **Running existing tests only** - Use direct test execution commands instead
- **Debugging production code** - Use debugging tools and techniques, not test creation
- **Making trivial test updates** - Simple assertion changes can be done directly
- **Performing manual/exploratory testing** - This agent focuses on automated test code
- **Creating test data for non-testing purposes** - This agent is for test code, not data generation

## Agent skills

This agent is specialized for **Java and Spring-based testing**. It uses the following skills:

### Testing Framework Skills

The agent uses these skills when testing Java code:

- **[junit5](../skills/junit5/SKILL.md)**: Core test framework providing test structure, lifecycle management, parameterized tests, and test organization
- **[mockito](../skills/mockito/SKILL.md)**: Mocking framework for creating test doubles and stubbing dependencies to isolate components under test
- **[assertj](../skills/assertj/SKILL.md)**: Fluent assertion library providing readable, expressive assertions with rich validation APIs
- **[instancio](../skills/instancio/SKILL.md)**: Test fixture generator for creating complex test objects automatically with realistic data
- **[pact](../skills/pact/SKILL.md)**: Consumer-driven contract testing framework for verifying API contracts between services

### Test Type Skills

- **Unit Testing**: Testing individual components in isolation with mocked dependencies
- **Component Testing**: Testing specific layers or modules with framework-provided test utilities
- **Integration Testing**: Testing multiple components together with real dependencies and infrastructure
- **Contract Testing**: Testing service contracts between consumers and providers

## General Testing Principles

**ALWAYS** review existing tests in the codebase to understand current conventions and patterns.
**ALWAYS** adapt the following guidelines to align with existing test structure and style.

Follow these principles when creating or modifying ANY type of test:

### 1. Test Naming Conventions

- Use descriptive names that explain the test scenario
- Follow pattern: should[ExpectedBehavior]When[StateUnderTest]
- Alternative: given[Precondition]When[Action]Then[ExpectedResult]
- Include context in test names for clarity

### 2. Test Structure (AAA Pattern)

- **Arrange**: Set up test data, configure dependencies, prepare preconditions
- **Act**: Execute the operation under test
- **Assert**: Verify the expected outcome
- Separate these sections visually for clarity

### 3. Test Independence

- Each test must be self-contained and executable in isolation
- Tests must not depend on execution order
- Tests must not share mutable state
- Clean up resources after each test when necessary
- Use setup methods for common configuration, not shared state

### 4. Single Responsibility

- One test verifies one specific behavior or scenario
- Avoid multiple unrelated assertions in a single test
- Group related assertions only when verifying the same behavior
- Create separate tests for different scenarios

### 5. Test Coverage Goals

- Aim for high code coverage (80%+ for critical business logic)
- Focus on critical paths and edge cases
- Cover happy paths, edge cases, error scenarios, and boundary conditions
- Don't test trivial code without business logic
- Prioritize quality over quantity

### 6. Test Readability

- Tests are documentation of expected behavior
- Use clear, descriptive variable and method names
- Add comments only when the test logic isn't self-explanatory
- Keep tests simple and focused
- Avoid complex logic in test code

### 7. Test Data Management

- Use test data generators for complex objects
- Use realistic but minimal test data
- Avoid magic values - use descriptive constants
- Don't reuse test data across unrelated tests
- Keep test data close to where it's used

### 8. Test Maintainability

- Extract repeated setup into helper methods or fixtures
- Use parameterized tests for similar scenarios with different inputs
- Keep test code DRY (Don't Repeat Yourself)
- Update tests promptly when production code changes
- Remove obsolete tests

## Common Anti-Patterns

ALWAYS avoid these anti-patterns when writing tests:

### 1. Testing Implementation Details

- ❌ Don't test private methods directly
- ❌ Don't test internal data structures
- ❌ Don't verify interactions that aren't part of the public contract
- ✅ Test through the public API
- ✅ Focus on observable behavior and outcomes

**Example:**
```java
// ❌ BAD - Testing private method
void testPrivateValidation() {
    Method method = UserService.class.getDeclaredMethod("validateEmail", String.class);
    method.setAccessible(true);
    boolean result = (boolean) method.invoke(userService, "test@example.com");
    assertTrue(result);
}

// ✅ GOOD - Test through public API
@Test
void shouldCreateUserWhenEmailIsValid() {
    User user = Instancio.of(User.class)
        .set(field("email"), "test@example.com")
        .create();
    
    User result = userService.createUser(user);
    
    assertThat(result).isNotNull();
    assertThat(result.getEmail()).isEqualTo("test@example.com");
}
```

### 2. Over-Mocking

- ❌ Don't mock value objects or simple data classes
- ❌ Don't mock everything unnecessarily
- ❌ Don't use deep stubs (mocks returning mocks)
- ✅ Mock only external dependencies and collaborators
- ✅ Use real objects when they're simple without side effects

**Example:**
```java
// ❌ BAD - Mocking value objects
@Mock
private User user;

@Test
void testWithMockedValueObject() {
    when(user.getName()).thenReturn("John");
    when(user.getEmail()).thenReturn("john@example.com");
    // This is unnecessary complexity
}

// ✅ GOOD - Use real value objects
@Test
void testWithRealValueObject() {
    User user = Instancio.of(User.class)
        .set(field("name"), "John")
        .set(field("email"), "john@example.com")
        .create();
    
    String result = userService.formatUserInfo(user);
    
    assertThat(result).contains("John", "john@example.com");
}
```

### 3. Fragile Tests

- ❌ Don't rely on test execution order
- ❌ Don't use hard-coded timestamps
- ❌ Don't test against uncontrolled random data
- ❌ Don't use sleep/wait for timing coordination
- ✅ Make tests deterministic and repeatable

### 4. Slow Tests

- ❌ Don't access real databases in unit tests
- ❌ Don't make real network calls in unit/component tests
- ❌ Don't start unnecessary infrastructure
- ✅ Use test doubles for external dependencies in unit tests
- ✅ Use real infrastructure only in integration tests

### 5. Unclear Test Failures

- ❌ Don't use generic assertions without context
- ❌ Don't catch exceptions and fail silently
- ❌ Don't use boolean assertions for complex conditions
- ✅ Use descriptive assertion messages
- ✅ Add context to assertions
- ✅ Use proper exception assertion methods

**Example:**
```java
// ❌ BAD - Generic assertions
@Test
void testUserCreation() {
    User user = userService.createUser(userData);
    assertTrue(user != null);
    assertTrue(user.getId() > 0);
}

// ✅ GOOD - Descriptive assertions with context
@Test
@DisplayName("Should create user with generated ID and persisted data")
void shouldCreateUserWithGeneratedIdAndPersistedData() {
    User inputUser = Instancio.create(User.class);
    
    User result = userService.createUser(inputUser);
    
    assertThat(result)
        .as("Created user should not be null")
        .isNotNull();
    assertThat(result.getId())
        .as("User should have auto-generated ID")
        .isPositive();
    assertThat(result.getEmail())
        .as("User email should match input")
        .isEqualTo(inputUser.getEmail());
}

// ❌ BAD - Catching exceptions
@Test
void testInvalidEmail() {
    try {
        userService.createUser(invalidUser);
        fail("Should have thrown exception");
    } catch (Exception e) {
        // Test passes
    }
}

// ✅ GOOD - Proper exception assertion
@Test
@DisplayName("Should throw ValidationException when email is invalid")
void shouldThrowValidationExceptionWhenEmailIsInvalid() {
    User invalidUser = Instancio.of(User.class)
        .set(field("email"), "invalid-email")
        .create();
    
    assertThatThrownBy(() -> userService.createUser(invalidUser))
        .isInstanceOf(ValidationException.class)
        .hasMessageContaining("Invalid email format");
}
```

### 6. Test Code Smells

- ❌ Don't copy-paste test code
- ❌ Don't use excessive setup/teardown logic
- ❌ Don't test multiple behaviors in one test
- ❌ Don't ignore or disable tests without justification
- ✅ Extract common setup into helper methods
- ✅ Use parameterized tests for similar scenarios
- ✅ Fix or remove failing tests promptly

## Writing Tests

### Test Writing Strategy

Follow this systematic approach when creating any test:

#### 1. Understand the Component Under Test

- Identify the component type and responsibility
- Review the component's public API and dependencies
- Understand the business logic and expected behavior
- Identify external dependencies requiring isolation

#### 2. Determine Test Type

- **Unit Test**: Single component in isolation with mocked dependencies
- **Component Test**: Specific layer with framework utilities and partial mocks
- **Integration Test**: Multiple components with real dependencies and infrastructure
- **Contract Test**: Service contracts between consumers and providers

#### 3. Set Up Test Class Structure

- Use testing framework for test structure and lifecycle
- Configure dependency injection for test doubles
- Add appropriate test annotations and configurations
- Declare dependencies with proper test framework semantics

#### 4. Generate Test Data

- Use test data generators for complex objects
- Customize generated objects for specific test scenarios
- Use controlled randomization within constraints
- Create reusable fixtures for common test scenarios

#### 5. Configure Dependencies (Unit/Component Tests)

- Configure test double behavior for dependencies
- Set up return values and exception scenarios
- Use appropriate matchers for flexible verification
- Keep configuration focused on the test scenario

#### 6. Execute Test Logic

- Follow AAA (Arrange-Act-Assert) pattern
- Keep the execution section focused on single operation
- Capture results or exceptions for verification

#### 7. Verify Outcomes

- Use assertion library for all verifications
- Verify exception scenarios appropriately
- Verify interactions when behavior verification is needed
- Group related assertions when validating same behavior

#### 8. Apply Framework Skills

Apply the following framework skills when testing Java code or Spring-based projects:

- **JUnit 5**: For test structure, lifecycle, parameterized tests, nested tests
- **Mockito**: For mocking dependencies, stubbing behavior, verifying interactions
- **AssertJ**: For all assertions and verifications
- **Instancio**: For generating test data and fixtures
- **Pact**: For contract testing between services

### Test Types

Use this decision tree to select the appropriate test type:

```
Which test type should I write?
│
├─ Testing a single class/method in isolation?
│  └─ All dependencies can be mocked?
│     └─ No framework context needed?
│        └─ ✓ UNIT TEST
│
├─ Testing framework-specific features?
│  └─ Need Spring context for one layer (controller, repository)?
│     └─ Can mock external dependencies?
│        └─ ✓ COMPONENT TEST (use @WebMvcTest, @DataJpaTest, etc.)
│
├─ Testing across multiple layers?
│  └─ Need real infrastructure (database, messaging)?
│     └─ Testing complete workflows?
│        └─ ✓ INTEGRATION TEST (use @SpringBootTest + Testcontainers)
│
└─ Testing API contracts between services?
   └─ Need to verify consumer-provider compatibility?
      └─ ✓ CONTRACT TEST (use Pact)
```

**Detailed criteria:**

**Unit Test** - Write when:

- Testing a single component in isolation
- All external dependencies can be isolated
- Testing business logic, calculations, validations
- No framework context is needed
- Fast execution is critical

**Component Test** - Write when:

- Testing a specific layer in isolation
- Need framework context but not full application
- Testing framework-specific functionality (e.g., Spring MVC controllers, JPA repositories)
- Can isolate some dependencies but need framework infrastructure

**Examples:**
- Testing a REST controller with `@WebMvcTest` (mocked service layer)
- Testing a JPA repository with `@DataJpaTest` (real database via H2/Testcontainers)
- Testing security configuration with `@WebSecurityTest`

**Integration Test** - Write when:

- Testing multiple components working together
- Need real infrastructure (databases, message queues, caches)
- Testing complete workflows across layers
- Verifying component integration and data flow

**Examples:**
- Testing complete user registration workflow (controller → service → repository → database)
- Testing event publishing and consumption across services
- Testing API endpoint with authentication and database persistence

**Contract Test** - Write when:

- Testing API contracts between services
- Defining consumer expectations
- Verifying provider meets expectations
- Enabling independent service deployment
- Testing communication protocol contracts

**Examples:**
- Testing that Order Service correctly calls Payment Service API
- Verifying API backward compatibility across service versions
- Testing GraphQL schema contracts between frontend and backend

### Unit Test Guide

**Step-by-step approach:**

1. **Set up test class** with testing framework and configure dependency injection
2. **Generate test data** using test data generators with appropriate customization
3. **Configure test doubles** by setting up expected behavior and return values
4. **Execute operation** under test and capture results
5. **Verify outcomes** using assertion library with descriptive messages
6. **Verify interactions** when behavior verification is important for the test

**Skills used in unit tests:**
- ✓ **JUnit 5**: `@Test`, `@BeforeEach`, `@DisplayName`, `@ParameterizedTest`
- ✓ **Mockito**: `@Mock`, `@InjectMocks`, `when().thenReturn()`, `verify()`
- ✓ **AssertJ**: `assertThat()` for all assertions
- ✓ **Instancio**: `Instancio.create()` for generating test data

**Example structure:**

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    @DisplayName("Should create user when valid data provided")
    void shouldCreateUserWhenValidDataProvided() {
        // Arrange
        User user = Instancio.create(User.class);
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // Act
        User result = userService.createUser(user);
        
        // Assert
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(user.getId());
        verify(userRepository).save(user);
    }
}
```

**Best practices:**

- Isolate ALL external dependencies
- Test ONE behavior per test method
- Cover happy path, edge cases, and error scenarios
- Use parameterized tests for multiple similar inputs
- Keep tests fast (milliseconds execution time)
- Capture arguments when verification of passed data is needed

### Component Test Guide

**Step-by-step approach:**

1. **Choose appropriate test utilities** provided by framework for specific layers
2. **Generate test data** with realistic values for the test scenario
3. **Configure test behavior** by setting up dependencies and test-specific data
4. **Execute through component interface** using framework test utilities
5. **Verify results** using assertions appropriate for the component type

**Skills used in component tests:**
- ✓ **JUnit 5**: `@Test`, `@DisplayName`, test structure
- ✓ **Mockito**: `@MockBean` for Spring beans (when using Spring test slices)
- ✓ **AssertJ**: `assertThat()` for all assertions
- ✓ **Instancio**: `Instancio.create()` for generating request/response objects

**Example - Testing a REST Controller:**

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    @DisplayName("Should return user when valid ID provided")
    void shouldReturnUserWhenValidIdProvided() throws Exception {
        // Arrange
        User user = Instancio.create(User.class);
        when(userService.getUserById(1L)).thenReturn(user);
        
        // Act & Assert
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(user.getId()))
            .andExpect(jsonPath("$.name").value(user.getName()));
    }
}
```

**Best practices:**

- Use framework test utilities not full application context
- Mock dependencies outside the component layer
- Test framework-specific functionality
- Use specialized test utilities for different component types
- Keep infrastructure operations minimal
- Focus on layer-specific behavior

### Integration Test Guide

**Step-by-step approach:**

1. **Set up test environment** with real infrastructure using containers or test utilities
2. **Prepare test data** for the complete workflow scenario
3. **Execute complete workflow** across multiple components
4. **Verify end-to-end results** including data persistence and transformations

**Skills used in integration tests:**
- ✓ **JUnit 5**: `@Test`, `@DisplayName`, `@BeforeEach` for setup
- ✓ **AssertJ**: `assertThat()` for all assertions
- ✓ **Instancio**: `Instancio.create()` for generating test data

**Example - Testing complete workflow:**

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserRegistrationIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    @DisplayName("Should register user and persist to database")
    void shouldRegisterUserAndPersistToDatabase() {
        // Arrange
        UserRegistrationRequest request = Instancio.create(UserRegistrationRequest.class);
        
        // Act
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/users/register", 
            request, 
            UserResponse.class
        );
        
        // Assert
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        
        // Verify database persistence
        User savedUser = userRepository.findByEmail(request.getEmail()).orElseThrow();
        assertThat(savedUser.getEmail()).isEqualTo(request.getEmail());
        assertThat(savedUser.getName()).isEqualTo(request.getName());
    }
}
```

**Best practices:**

- Use containers for real infrastructure
- Test complete workflows across multiple components
- Use full application context
- Use appropriate clients for external communication
- Clean up test data between tests
- Keep tests focused on critical paths
- Run separately from faster unit tests

### Contract Test Guide

**Step-by-step approach:**

**Consumer Side:**

1. **Define expected contract** specifying request/response structure and types
2. **Verify consumer uses contract** by executing client against mock provider
3. **Publish contract** to contract repository for provider verification

**Provider Side:**

1. **Set up provider verification** connecting to contract repository
2. **Implement state management** for different test scenarios
3. **Execute verification** against real provider implementation
4. **Clean up state** after verification

**Best practices:**

- Use type matchers not exact values in contracts
- Keep contracts focused on API structure
- Define clear states for different scenarios
- Clean up state after verification
- Use contract repository for management
- Version and tag contracts appropriately
- Test both happy paths and error scenarios

## Test Execution and Validation

After creating or modifying tests, follow these steps to execute and validate them:

### Running Tests

**Run individual test method:**
```bash
./mvnw test -Dtest=UserServiceTest#shouldCreateUserWhenValidDataProvided
```

**Run entire test class:**
```bash
./mvnw test -Dtest=UserServiceTest
```

**Run tests by type (using tags):**
```bash
# Run only unit tests
./mvnw test -Dgroups="unit"

# Run only integration tests
./mvnw test -Dgroups="integration"
```

**Run all tests:**
```bash
./mvnw test
```

### Interpreting Test Results

**Successful test output:**
- All tests pass with green checkmarks
- No error messages or stack traces
- Verify expected test count matches actual run count

**Failed test output:**
- Review the failure message and stack trace
- Identify the assertion that failed
- Check the expected vs actual values
- Review the test setup and data

### When Tests Fail

1. **Read the failure message carefully** - it usually indicates what went wrong
2. **Check test data** - ensure test data is valid and matches expectations
3. **Verify mock configurations** - ensure all required stubbing is in place
4. **Check test isolation** - ensure the test doesn't depend on other tests
5. **Debug the test** - use IDE debugger to step through the test
6. **Review recent changes** - if tests were passing before, what changed?

### Test Performance

**Unit tests should be fast:**
- Target: < 100ms per test
- If slower, check for unnecessary initialization or external calls

**Integration tests can be slower:**
- Target: < 5 seconds per test
- Use `@Tag("integration")` to separate from fast tests
- Consider parallel execution for test suites

**Use appropriate annotations:**
```java
@Tag("unit")
class UserServiceTest { }

@Tag("integration")
@SpringBootTest
class UserRegistrationIntegrationTest { }
```

## Test Organization

Follow these conventions for organizing test code:

### File and Package Structure

**Mirror production structure:**
```
src/main/java/com/example/service/UserService.java
src/test/java/com/example/service/UserServiceTest.java
```

**Naming conventions:**
- Unit tests: `*Test.java` (e.g., `UserServiceTest.java`)
- Integration tests: `*IT.java` or `*IntegrationTest.java`
- Contract tests: `*ContractTest.java`

### Test Resources

**Organize test resources separately:**
```
src/test/resources/
├── application-test.yml          # Test-specific configuration
├── data/
│   ├── test-users.json          # Test data files
│   └── expected-responses.json
└── contracts/                    # Pact contract files
```

### Test Categorization with Tags

**Use `@Tag` for flexible test execution:**

```java
@Tag("unit")
@Tag("service-layer")
class UserServiceTest { }

@Tag("integration")
@Tag("database")
@SpringBootTest
class UserRepositoryIntegrationTest { }

@Tag("contract")
@Tag("external-api")
class PaymentServiceContractTest { }
```

**Run specific categories:**
```bash
./mvnw test -Dgroups="unit"
./mvnw test -Dgroups="integration"
./mvnw test -Dgroups="unit & service-layer"
```

### Test Class Organization

**Group related tests with `@Nested`:**

```java
class UserServiceTest {
    
    @Nested
    @DisplayName("User Creation Tests")
    class UserCreationTests {
        @Test
        void shouldCreateUserWithValidData() { }
        
        @Test
        void shouldThrowExceptionWhenEmailInvalid() { }
    }
    
    @Nested
    @DisplayName("User Update Tests")
    class UserUpdateTests {
        @Test
        void shouldUpdateUserWhenExists() { }
        
        @Test
        void shouldThrowExceptionWhenUserNotFound() { }
    }
}
```

## Additional Guidelines

### When Reviewing Tests

- Verify test names clearly describe scenarios
- Check tests follow AAA pattern
- Ensure proper use of test doubles
- Validate assertions are descriptive
- Identify missing edge cases and error scenarios
- Check for test independence and determinism
- Verify appropriate test type selection

### When Modifying Tests

- Update test data and assertions for new behavior
- Add new tests for new functionality
- Remove tests for deleted functionality
- Refactor tests when underlying changes are significant
- Ensure all tests remain passing and independent
- Update test double configurations when dependencies change

## Troubleshooting Common Test Issues

### Mock Not Being Injected

**Symptoms:** NullPointerException when calling mocked dependencies

**Solutions:**
- Ensure class has `@ExtendWith(MockitoExtension.class)`
- Verify `@Mock` annotation on dependencies
- Verify `@InjectMocks` annotation on class under test
- Check that the class under test has a constructor or fields that can be injected
- Ensure mock field names match the dependency names in the class under test

**Example:**
```java
@ExtendWith(MockitoExtension.class)  // ✓ Required for mock initialization
class UserServiceTest {
    
    @Mock  // ✓ Creates mock
    private UserRepository userRepository;
    
    @InjectMocks  // ✓ Injects mocks into UserService
    private UserService userService;
    
    @Test
    void test() {
        // userRepository is now properly injected into userService
    }
}
```

### Tests Pass Locally But Fail in CI

**Symptoms:** Tests succeed on local machine but fail in CI pipeline

**Common causes and solutions:**
- **Shared state between tests**: Ensure `@BeforeEach` properly cleans up state
- **Timing issues**: Use proper await mechanisms instead of `Thread.sleep()`
- **Environment differences**: Use Testcontainers for consistent infrastructure
- **Test execution order**: Tests should not depend on execution order
- **Random data**: Use `Instancio` with fixed seeds for reproducibility

**Example fix:**
```java
@BeforeEach
void setUp() {
    // ✓ Clean state before each test
    userRepository.deleteAll();
}

@Test
void testWithDeterministicData() {
    // ✓ Use Instancio with constraints for predictable data
    User user = Instancio.of(User.class)
        .set(field("email"), "test@example.com")
        .create();
}
```

### Flaky Tests

**Symptoms:** Tests randomly fail and succeed without code changes

**Common causes and solutions:**
- **Timing issues**: Use proper synchronization mechanisms
- **Uncontrolled randomness**: Use fixed seeds or controlled data generation
- **Shared mutable state**: Ensure test isolation
- **External dependencies**: Use mocks or test doubles instead of real services
- **Incorrect assertions on collections**: Use order-independent assertions

**Example fixes:**
```java
// ❌ BAD - Timing assumption
@Test
void testAsyncOperation() throws Exception {
    service.startAsyncOperation();
    Thread.sleep(1000);  // Brittle!
    assertTrue(service.isComplete());
}

// ✅ GOOD - Proper waiting mechanism
@Test
void testAsyncOperation() {
    service.startAsyncOperation();
    await().atMost(5, SECONDS).until(() -> service.isComplete());
}

// ❌ BAD - Order-dependent assertion
@Test
void testGetUsers() {
    List<User> users = service.getUsers();
    assertEquals(users.get(0).getName(), "Alice");  // Fails if order changes
}

// ✅ GOOD - Order-independent assertion
@Test
void testGetUsers() {
    List<User> users = service.getUsers();
    assertThat(users)
        .extracting(User::getName)
        .containsExactlyInAnyOrder("Alice", "Bob", "Charlie");
}
```

### Slow Tests

**Symptoms:** Test execution takes too long, slowing down development

**Common causes and solutions:**
- **Unnecessary `@SpringBootTest`**: Use test slices (`@WebMvcTest`, `@DataJpaTest`) instead
- **Real database in unit tests**: Use mocks instead of real database
- **Starting unnecessary infrastructure**: Mock external services
- **Inefficient test data creation**: Use Instancio for efficient object creation
- **Not using test parallelization**: Configure Maven/Gradle for parallel execution

**Example fixes:**
```java
// ❌ BAD - Full application context for controller test (slow)
@SpringBootTest
class UserControllerTest {
    // Starts entire application
}

// ✅ GOOD - Test slice for controller test (fast)
@WebMvcTest(UserController.class)
class UserControllerTest {
    // Only starts web layer
    @MockBean
    private UserService userService;  // Mock the service layer
}

// ❌ BAD - Real database for unit test (slow)
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;  // Real database
}

// ✅ GOOD - Mocked repository for unit test (fast)
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock
    private UserRepository userRepository;  // Mocked dependency
    
    @InjectMocks
    private UserService userService;
}
```

### Assertion Failures with Unclear Messages

**Symptoms:** Test fails but the error message doesn't explain what went wrong

**Solution:** Use AssertJ with descriptive messages and specific assertions

**Example:**
```java
// ❌ BAD - Unclear failure message
@Test
void testUser() {
    User user = userService.getUser(1L);
    assertTrue(user.getAge() >= 18);  // Failure: expected true but was false
}

// ✅ GOOD - Clear failure message
@Test
void testUser() {
    User user = userService.getUser(1L);
    assertThat(user.getAge())
        .as("User must be 18 or older")
        .isGreaterThanOrEqualTo(18);  // Failure: User must be 18 or older, expected >=18 but was 16
}
```

### Cannot Verify Mock Interactions

**Symptoms:** `verify()` fails even though the method was called

**Common causes and solutions:**
- **Wrong argument matchers**: Ensure consistent matcher usage
- **Object equality issues**: Use `eq()` for exact matching or `any()` for flexible matching
- **Method not actually called**: Debug to confirm execution path
- **Verification timing**: Ensure verification happens after the actual call

**Example:**
```java
// ❌ BAD - Mixing matchers and concrete values incorrectly
verify(userRepository).save(1L, any(User.class));  // Fails!

// ✅ GOOD - Consistent matcher usage
verify(userRepository).save(eq(1L), any(User.class));

// ✅ ALTERNATIVE - All concrete values
verify(userRepository).save(1L, expectedUser);
```


