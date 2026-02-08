# Before & After Comparison

## Visual Summary of Transformation

### Skills Section

#### BEFORE (Java-specific)
```markdown
## Agent skills

This agent is specialized for **Java and Spring-based testing**. It uses the following skills:

### Testing Framework Skills

The agent uses these skills when testing Java code:

- **[junit5](../skills/junit5/SKILL.md)**: Core test framework...
- **[mockito](../skills/mockito/SKILL.md)**: Mocking framework...
- **[assertj](../skills/assertj/SKILL.md)**: Fluent assertion library...
- **[instancio](../skills/instancio/SKILL.md)**: Test fixture generator...
- **[pact](../skills/pact/SKILL.md)**: Consumer-driven contract testing...
```

#### AFTER (Language-agnostic)
```markdown
## How this agent works

This agent acts as an **orchestrator of testing skills**.

### Required Skill Categories

#### 1. Test Framework Skill
**Examples by language:**
- Java: JUnit 5, TestNG
- JavaScript/TypeScript: Jest, Mocha, Vitest
- Python: pytest, unittest
- C#: NUnit, xUnit, MSTest
- Go: testing package
- Ruby: RSpec, Minitest

#### 2. Mocking/Test Doubles Skill
**Examples by language:**
- Java: Mockito, EasyMock
- JavaScript/TypeScript: Jest mocks, Sinon
- Python: unittest.mock, pytest-mock
- C#: Moq, NSubstitute
- Go: gomock, testify/mock
- Ruby: RSpec mocks, Mocha

[... 3 more skill categories ...]
```

---

### Unit Test Example

#### BEFORE (Java-specific)
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

#### AFTER (Language-agnostic pseudocode)
```
test_class UserServiceTest:
    // Dependencies (mocked)
    mock userRepository
    
    // System under test (with injected mocks)
    userService = new UserService(userRepository)
    
    test_should_create_user_when_valid_data_provided():
        // Arrange - using Test Data Generation Skill
        user = generate_test_user()
        configure_mock(userRepository.save, returns=user)
        
        // Act
        result = userService.createUser(user)
        
        // Assert - using Assertion Skill
        assert_not_null(result)
        assert_equal(result.id, user.id)
        
        // Verify - using Mocking Skill
        verify_called(userRepository.save, with=user)
```

---

### Test Execution

#### BEFORE (Maven only)
```bash
# Run individual test
./mvnw test -Dtest=UserServiceTest#shouldCreateUser

# Run test class
./mvnw test -Dtest=UserServiceTest

# Run by category
./mvnw test -Dgroups="unit"

# Run all tests
./mvnw test
```

#### AFTER (Multi-language)
```bash
# Run individual test - Examples by language/tool:
# Java (Maven): ./mvnw test -Dtest=ClassName#methodName
# Java (Gradle): ./gradlew test --tests ClassName.methodName
# JavaScript (npm): npm test -- --testNamePattern="test name"
# Python (pytest): pytest path/to/test_file.py::test_function_name
# C# (dotnet): dotnet test --filter "FullyQualifiedName=..."
# Go: go test -run TestFunctionName
# Ruby (RSpec): rspec spec/path/to/spec_file.rb:line_number

# Run test class - Examples by language/tool:
# Java (Maven): ./mvnw test -Dtest=ClassName
# JavaScript (npm): npm test path/to/test.file.js
# Python (pytest): pytest path/to/test_file.py
# C# (dotnet): dotnet test --filter "FullyQualifiedName~..."
# Go: go test -run "TestClassName"
# Ruby (RSpec): rspec spec/path/to/spec_file.rb
```

---

### Component Test Guide

#### BEFORE (Spring-specific)
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
            .andExpect(jsonPath("$.id").value(user.getId()));
    }
}
```

#### AFTER (Framework-agnostic)
```
test_class UserControllerTest:
    // Framework-provided test utilities
    http_client = framework_test_client()
    
    // Mocked dependencies
    mock userService
    
    test_should_return_user_when_valid_id_provided():
        // Arrange - using Test Data Generation Skill
        user = generate_test_user(id=1)
        configure_mock(userService.getUserById, with_arg=1, returns=user)
        
        // Act - using framework test client
        response = http_client.get("/api/users/1")
        
        // Assert - using Assertion Skill
        assert_status_code(response, 200)
        assert_equal(response.body.id, user.id)
        assert_equal(response.body.name, user.name)
```

---

### Test Organization

#### BEFORE (Java only)
```
src/main/java/com/example/service/UserService.java
src/test/java/com/example/service/UserServiceTest.java

Naming conventions:
- Unit tests: *Test.java
- Integration tests: *IT.java or *IntegrationTest.java
```

#### AFTER (Multi-language)
```
# Examples by language:

# Java:
src/main/java/com/example/service/UserService.java
src/test/java/com/example/service/UserServiceTest.java

# JavaScript/TypeScript:
src/services/UserService.ts
src/services/UserService.test.ts

# Python:
src/services/user_service.py
tests/services/test_user_service.py

# C#:
src/Services/UserService.cs
tests/Services/UserServiceTests.cs

# Go:
pkg/services/user_service.go
pkg/services/user_service_test.go

# Ruby:
lib/services/user_service.rb
spec/services/user_service_spec.rb

Naming conventions (adapt to language idioms):
- Unit tests: *Test, *_test, *Spec, *.test.*
- Integration tests: *IT, *IntegrationTest, *_integration_test
```

---

### Anti-Pattern Example

#### BEFORE (Java-specific)
```java
// ❌ BAD - Mocking value objects
@Mock
private User user;

@Test
void testWithMockedValueObject() {
    when(user.getName()).thenReturn("John");
    when(user.getEmail()).thenReturn("john@example.com");
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

#### AFTER (Language-agnostic)
```
// ❌ BAD - Mocking simple value objects
test_with_mocked_value_object():
    user = create_mock(User)
    when(user.getName()).return("John")
    when(user.getEmail()).return("john@example.com")
    // Unnecessary complexity for simple data

// ✅ GOOD - Use real value objects
test_with_real_value_object():
    // Arrange
    user = create_test_user(
        name="John",
        email="john@example.com"
    )
    
    // Act
    result = userService.formatUserInfo(user)
    
    // Assert
    assert_contains(result, ["John", "john@example.com"])

**Principle:** Only mock complex dependencies with side effects, 
not simple value objects.
```

---

### Decision Tree

#### BEFORE (Framework-specific)
```
├─ Testing framework-specific features?
│  └─ Need Spring context for one layer?
│     └─ Can mock external dependencies?
│        └─ ✓ COMPONENT TEST (use @WebMvcTest, @DataJpaTest, etc.)
│
├─ Testing across multiple layers?
│  └─ Need real infrastructure?
│     └─ Testing complete workflows?
│        └─ ✓ INTEGRATION TEST (use @SpringBootTest + Testcontainers)
```

#### AFTER (Generic)
```
├─ Testing framework-specific features?
│  └─ Need framework context for one layer?
│     └─ Can mock external dependencies?
│        └─ ✓ COMPONENT TEST
│           Examples:
│           - Web layer tests (controllers, routes, handlers)
│           - Data access layer tests (repositories, DAOs)
│           - Security/auth layer tests
│
├─ Testing across multiple layers?
│  └─ Need real infrastructure (database, messaging, cache)?
│     └─ Testing complete workflows?
│        └─ ✓ INTEGRATION TEST
│           Examples:
│           - Full application with real database
│           - End-to-end API workflows
│           - Multi-service interactions
```

---

## Key Differences Summary

| Aspect | Before | After |
|--------|--------|-------|
| **Focus** | Java/Spring | All languages |
| **Skills** | Hardcoded list | Skill categories |
| **Examples** | Java code only | Pseudocode + multi-language |
| **Commands** | Maven only | 7 build tools |
| **File structure** | Java paths | 6 language conventions |
| **Agent role** | Java test guide | Skill orchestrator |
| **Maintainability** | Agent has Java knowledge | Skills have language knowledge |
| **Extensibility** | Add Java features | Add skills for new languages |

## The Transformation in One Sentence

**Before:** A comprehensive Java/Spring testing guide.

**After:** A universal testing orchestrator that coordinates language-specific skills.

---

## Agent Architecture

### BEFORE
```
Tester Agent
└─ Contains Java/Spring testing knowledge
   ├─ JUnit 5 examples
   ├─ Mockito patterns
   ├─ AssertJ assertions
   ├─ Instancio data generation
   └─ Spring Boot test slices
```

### AFTER
```
Tester Agent (Orchestrator)
├─ Universal testing principles
├─ Test type strategies
└─ Skill coordination

Skills (Pluggable)
├─ Test Framework Skill
│  ├─ JUnit 5 (Java)
│  ├─ Jest (JavaScript)
│  ├─ pytest (Python)
│  └─ [...other languages]
├─ Mocking Skill
│  ├─ Mockito (Java)
│  ├─ Sinon (JavaScript)
│  ├─ unittest.mock (Python)
│  └─ [...other languages]
└─ [... other skill categories]
```

The agent is now a **true orchestrator** 🎯
