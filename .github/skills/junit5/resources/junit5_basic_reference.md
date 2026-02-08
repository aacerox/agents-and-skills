# JUnit 5 Basic Reference

## Most Commonly Used Features

### Test Lifecycle Annotations

```java
@BeforeAll
static void initAll() {
    // Executed once before all tests
}

@BeforeEach
void init() {
    // Executed before each test
}

@Test
void testMethod() {
    // Test implementation
}

@AfterEach
void tearDown() {
    // Executed after each test
}

@AfterAll
static void tearDownAll() {
    // Executed once after all tests
}
```

### Basic Test Configuration

```java
@DisplayName("User Service Tests")
class UserServiceTest {
    
    @Test
    @DisplayName("Should return user when valid ID provided")
    void getUserById() {
        // Test implementation
    }
    
    @Test
    @Disabled("Temporarily disabled")
    void testDisabled() {
        // Test implementation
    }
    
    @Test
    @Timeout(5)
    void testWithTimeout() {
        // Must complete within 5 seconds
    }
}
```

### Parameterized Tests - @ValueSource

```java
@ParameterizedTest
@ValueSource(strings = {"racecar", "radar", "level"})
void testPalindromes(String word) {
    assertTrue(isPalindrome(word));
}

@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 5, 8})
void testNumbers(int number) {
    assertTrue(number > 0);
}
```

### Parameterized Tests - @CsvSource

```java
@ParameterizedTest
@CsvSource({
    "1, 1",
    "2, 4",
    "3, 9"
})
void testSquare(int input, int expected) {
    assertEquals(expected, input * input);
}
```

### Parameterized Tests - @MethodSource

```java
@ParameterizedTest
@MethodSource("provideUsers")
void testUserValidation(User user) {
    assertTrue(validator.isValid(user));
}

static Stream<User> provideUsers() {
    return Stream.of(
        new User("john@example.com", 25),
        new User("jane@example.com", 30)
    );
}
```

### Nested Tests

```java
@DisplayName("Calculator Tests")
class CalculatorTest {
    
    Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Nested
    @DisplayName("Addition Tests")
    class AdditionTests {
        
        @Test
        void addPositiveNumbers() {
            assertEquals(5, calculator.add(2, 3));
        }
    }
    
    @Nested
    @DisplayName("Division Tests")
    class DivisionTests {
        
        @Test
        void divideNumbers() {
            assertEquals(2, calculator.divide(6, 3));
        }
    }
}
```

### Basic Assertions

```java
@Test
void standardAssertions() {
    assertEquals(2, calculator.add(1, 1));
    assertNotEquals(3, calculator.add(1, 1));
    assertTrue(calculator.add(1, 1) > 0);
    assertFalse(calculator.add(1, 1) < 0);
    assertNull(null);
    assertNotNull("value");
}

@Test
void groupedAssertions() {
    Person person = new Person("John", "Doe");
    
    assertAll("person",
        () -> assertEquals("John", person.getFirstName()),
        () -> assertEquals("Doe", person.getLastName())
    );
}
```

### Exception Testing

```java
@Test
void exceptionTesting() {
    Exception exception = assertThrows(
        IllegalArgumentException.class,
        () -> calculator.divide(1, 0)
    );
    
    assertEquals("Cannot divide by zero", exception.getMessage());
}
```

### Repeated Tests

```java
@RepeatedTest(5)
void repeatedTest() {
    // Executed 5 times
}

@RepeatedTest(value = 3, name = "repetition {currentRepetition} of {totalRepetitions}")
void repeatedTestWithName(RepetitionInfo info) {
    assertEquals(3, info.getTotalRepetitions());
}
```
