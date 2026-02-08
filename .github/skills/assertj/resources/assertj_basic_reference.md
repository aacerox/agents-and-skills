# AssertJ Basic Reference

## Most Commonly Used Features

### Object Assertions

```java
@Test
void testObjectAssertions() {
    User user = new User("John", 30);
    
    assertThat(user).isNotNull();
    assertThat(user).isInstanceOf(User.class);
    assertThat(user).isExactlyInstanceOf(User.class);
    
    User user2 = user;
    assertThat(user).isSameAs(user2);
    assertThat(user).isEqualTo(new User("John", 30));
}
```

### String Assertions

```java
@Test
void testStringAssertions() {
    String text = "Hello World";
    
    assertThat(text)
        .isNotNull()
        .isNotEmpty()
        .isNotBlank()
        .startsWith("Hello")
        .endsWith("World")
        .contains("lo Wo")
        .containsIgnoringCase("HELLO")
        .doesNotContain("Goodbye")
        .hasSize(11)
        .matches("^Hello.*")
        .isEqualToIgnoringCase("hello world");
}
```

### Numeric Assertions

```java
@Test
void testNumericAssertions() {
    int value = 42;
    
    assertThat(value)
        .isEqualTo(42)
        .isNotZero()
        .isPositive()
        .isGreaterThan(40)
        .isGreaterThanOrEqualTo(42)
        .isLessThan(50)
        .isLessThanOrEqualTo(42)
        .isBetween(40, 50);
    
    double price = 99.99;
    assertThat(price)
        .isCloseTo(100.0, within(0.1))
        .isCloseTo(100.0, withinPercentage(1));
}
```

### Boolean Assertions

```java
@Test
void testBooleanAssertions() {
    boolean isActive = true;
    
    assertThat(isActive).isTrue();
    assertThat(!isActive).isFalse();
}
```

### List Assertions

```java
@Test
void testListAssertions() {
    List<String> fruits = Arrays.asList("apple", "banana", "orange");
    
    assertThat(fruits)
        .isNotEmpty()
        .hasSize(3)
        .contains("apple")
        .containsExactly("apple", "banana", "orange")
        .containsExactlyInAnyOrder("orange", "apple", "banana")
        .doesNotContain("grape")
        .startsWith("apple")
        .endsWith("orange")
        .doesNotHaveDuplicates();
}
```

### Map Assertions

```java
@Test
void testMapAssertions() {
    Map<String, Integer> scores = Map.of(
        "Alice", 95,
        "Bob", 87
    );
    
    assertThat(scores)
        .isNotEmpty()
        .hasSize(2)
        .containsKey("Alice")
        .containsKeys("Alice", "Bob")
        .containsValue(95)
        .containsEntry("Alice", 95)
        .doesNotContainKey("Dave");
}
```

### Exception Assertions

```java
@Test
void testExceptionAssertions() {
    assertThatThrownBy(() -> {
        throw new IllegalArgumentException("Invalid input");
    })
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("Invalid input")
        .hasMessageContaining("Invalid")
        .hasMessageStartingWith("Invalid")
        .hasMessageEndingWith("input");
}

@Test
void testNoException() {
    assertThatCode(() -> {
        service.performOperation();
    }).doesNotThrowAnyException();
}
```

### Extracting from Collections

```java
@Test
void testExtracting() {
    List<User> users = Arrays.asList(
        new User("Alice", 25),
        new User("Bob", 30)
    );
    
    assertThat(users)
        .extracting(User::getName)
        .containsExactly("Alice", "Bob");
    
    assertThat(users)
        .extracting("name", "age")
        .contains(
            tuple("Alice", 25),
            tuple("Bob", 30)
        );
}
```

### Filtering Collections

```java
@Test
void testFiltering() {
    List<User> users = Arrays.asList(
        new User("Alice", 25),
        new User("Bob", 30),
        new User("Carol", 35)
    );
    
    assertThat(users)
        .filteredOn(user -> user.getAge() > 28)
        .extracting(User::getName)
        .containsExactly("Bob", "Carol");
    
    assertThat(users)
        .filteredOn("age", 30)
        .extracting("name")
        .containsExactly("Bob");
}
```

### Date/Time Assertions

```java
@Test
void testDateAssertions() {
    LocalDate date = LocalDate.of(2026, 2, 8);
    
    assertThat(date)
        .isAfter(LocalDate.of(2026, 1, 1))
        .isBefore(LocalDate.of(2027, 1, 1))
        .isBetween(
            LocalDate.of(2026, 1, 1),
            LocalDate.of(2026, 12, 31))
        .hasYear(2026)
        .hasMonthValue(2)
        .hasDayOfMonth(8);
}
```

### Soft Assertions

```java
@Test
void testSoftAssertions() {
    User user = new User("John", 30);
    
    SoftAssertions.assertSoftly(softly -> {
        softly.assertThat(user.getName()).isEqualTo("John");
        softly.assertThat(user.getAge()).isEqualTo(30);
        softly.assertThat(user.isActive()).isTrue();
    });
}
```
