# JUnit 5 Advanced Reference

## Advanced Features

### @EnumSource

```java
@ParameterizedTest
@EnumSource(Status.class)
void testAllStatuses(Status status) {
    assertNotNull(status);
}

@ParameterizedTest
@EnumSource(value = Status.class, names = {"ACTIVE", "PENDING"})
void testSpecificStatuses(Status status) {
    assertTrue(status.isProcessable());
}
```

### @CsvFileSource

```java
@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testWithCsvFile(String name, int age, String city) {
    assertNotNull(name);
    assertTrue(age > 0);
    assertNotNull(city);
}
```

### Conditional Test Execution

```java
@Test
@EnabledOnOs(OS.LINUX)
void testOnLinux() {
    // Runs only on Linux
}

@Test
@EnabledOnJre(JRE.JAVA_17)
void testOnJava17() {
    // Runs only on Java 17
}

@Test
@EnabledIfSystemProperty(named = "env", matches = "staging")
void testOnStaging() {
    // Runs only when system property env=staging
}

@Test
@EnabledIfEnvironmentVariable(named = "CI", matches = "true")
void testOnCI() {
    // Runs only in CI environment
}

@Test
@EnabledIf("customCondition")
void testWithCustomCondition() {
    // Runs only if customCondition() returns true
}

boolean customCondition() {
    return System.getenv("ENABLE_TEST") != null;
}
```

### Dynamic Tests

```java
@TestFactory
Stream<DynamicTest> dynamicTestsFromStream() {
    return Stream.of("apple", "banana", "orange")
        .map(fruit -> DynamicTest.dynamicTest(
            "Test for " + fruit,
            () -> assertNotNull(fruit)
        ));
}

@TestFactory
Collection<DynamicTest> dynamicTestsFromCollection() {
    return Arrays.asList(
        DynamicTest.dynamicTest("Test 1", () -> assertTrue(true)),
        DynamicTest.dynamicTest("Test 2", () -> assertEquals(4, 2 + 2))
    );
}
```

### Timeout Assertions

```java
@Test
void timeoutAssertion() {
    assertTimeout(Duration.ofSeconds(2), () -> {
        // Task that should complete in less than 2 seconds
        Thread.sleep(1000);
    });
}

@Test
void timeoutExceeded() {
    assertTimeoutPreemptively(Duration.ofSeconds(1), () -> {
        // Task that takes too long
        Thread.sleep(2000);
    });
}
```

### Test Instance Lifecycle

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class SharedInstanceTest {
    // Single instance for all tests
    
    private int counter = 0;
    
    @BeforeAll
    void setUp() {
        // Instance method (not static)
    }
    
    @Test
    void test1() {
        counter++;
    }
    
    @Test
    void test2() {
        counter++;
    }
}
```

### Test Execution Order

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTest {
    
    @Test
    @Order(1)
    void firstTest() {
        // Executed first
    }
    
    @Test
    @Order(2)
    void secondTest() {
        // Executed second
    }
}

@TestMethodOrder(MethodOrderer.MethodName.class)
class AlphabeticOrderTest {
    // Tests executed in alphabetical order
}
```

### Custom Extensions

```java
@ExtendWith(MockitoExtension.class)
class TestWithExtension {
    
    @Mock
    private UserRepository repository;
    
    @Test
    void test() {
        // Mockito extension handles mock initialization
    }
}

// Custom Extension
class TimingExtension implements BeforeEachCallback, AfterEachCallback {
    
    @Override
    public void beforeEach(ExtensionContext context) {
        context.getStore(Namespace.GLOBAL)
            .put("startTime", System.currentTimeMillis());
    }
    
    @Override
    public void afterEach(ExtensionContext context) {
        long startTime = context.getStore(Namespace.GLOBAL)
            .get("startTime", long.class);
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Test duration: " + duration + "ms");
    }
}
```

### Complex Parameterized Tests with Arguments

```java
@ParameterizedTest
@MethodSource("provideStringAndExpectedLength")
void testStringLength(String input, int expectedLength) {
    assertEquals(expectedLength, input.length());
}

static Stream<Arguments> provideStringAndExpectedLength() {
    return Stream.of(
        Arguments.of("hello", 5),
        Arguments.of("world", 5),
        Arguments.of("test", 4),
        Arguments.of(null, 0)
    );
}
```
