# Instancio Advanced Reference

## Advanced Features

### Predicate Selectors

```java
@Test
void testWithPredicateSelectors() {
    User user = Instancio.of(User.class)
        .set(fields().matching(".*Email.*"), "test@example.com")
        .generate(fields().ofType(Integer.class), gen -> gen.ints().range(1, 100))
        .create();
    
    assertThat(user.getEmail()).isEqualTo("test@example.com");
}
```

### Null Handling

```java
@Test
void testWithNullableFields() {
    User user = Instancio.of(User.class)
        .withNullable(field(User::getMiddleName))
        .setNull(field(User::getPhoneNumber))
        .create();
    
    // middleName may or may not be null
    assertThat(user.getPhoneNumber()).isNull();
}
```

### Subtype Mapping

```java
@Test
void testWithSubtypes() {
    // Map interface to concrete implementation
    Payment payment = Instancio.of(Payment.class)
        .subtype(all(PaymentMethod.class), CreditCardPayment.class)
        .create();
    
    assertThat(payment.getPaymentMethod())
        .isInstanceOf(CreditCardPayment.class);
}
```

### Nested Object Customization

```java
@Test
void testNestedObjects() {
    Order order = Instancio.of(Order.class)
        .set(field(Order::getStatus), OrderStatus.CONFIRMED)
        .set(field(Address::getCountry).within(scope(Order::getShippingAddress)), "USA")
        .generate(field(Item::getPrice).within(scope(Order::getItems)), 
            gen -> gen.bigDecimal().min(10).max(100))
        .create();
    
    assertThat(order.getShippingAddress().getCountry()).isEqualTo("USA");
    assertThat(order.getItems())
        .extracting(Item::getPrice)
        .allMatch(price -> price.compareTo(BigDecimal.TEN) >= 0);
}
```

### Conditional Generation

```java
@Test
void testConditionalGeneration() {
    User user = Instancio.of(User.class)
        .generate(field(User::getAge), gen -> gen.ints().range(18, 100))
        .onComplete(field(User::getAge), (User u) -> {
            if (u.getAge() >= 65) {
                u.setDiscount(0.2);
            }
        })
        .create();
    
    if (user.getAge() >= 65) {
        assertThat(user.getDiscount()).isEqualTo(0.2);
    }
}
```

### Map Configuration

```java
@Test
void testWithMapConfiguration() {
    Configuration config = Instancio.of(Configuration.class)
        .generate(field(Configuration::getProperties), gen -> gen.map().size(10))
        .set(allStrings().within(scope(Configuration::getProperties)), "value")
        .create();
    
    assertThat(config.getProperties()).hasSize(10);
}
```

### Enum Generators

```java
@Test
void testWithEnumGenerators() {
    Order order = Instancio.of(Order.class)
        .generate(field(Order::getStatus), 
            gen -> gen.enumOf(OrderStatus.class)
                      .excluding(OrderStatus.CANCELLED))
        .create();
    
    assertThat(order.getStatus()).isNotEqualTo(OrderStatus.CANCELLED);
}
```

### JUnit 5 Integration

```java
@ExtendWith(InstancioExtension.class)
class UserServiceTest {
    
    @Test
    void testCreateUser(@Random User user) {
        // Instancio automatically creates user
        assertThat(user).isNotNull();
    }
    
    @Test
    void testWithSeed(@Seed(12345) @Random User user) {
        // Reproducible random data
        assertThat(user.getName()).isNotNull();
    }
}
```

### Parameterized Tests with Instancio

```java
@ParameterizedTest
@MethodSource("provideUsers")
void testUserValidation(User user) {
    assertTrue(validator.isValid(user));
}

static Stream<User> provideUsers() {
    return Instancio.stream(User.class)
        .generate(field(User::getAge), gen -> gen.ints().range(18, 100))
        .limit(10);
}
```

### Test Data Builder Pattern

```java
class UserTestData {
    static Model<User> activeUser() {
        return Instancio.of(User.class)
            .set(field(User::getStatus), UserStatus.ACTIVE)
            .generate(field(User::getEmail), 
                gen -> gen.text().pattern("#a#a#a#a@test.com"))
            .toModel();
    }
    
    static Model<User> adminUser() {
        return Instancio.of(activeUser())
            .set(field(User::getRole), Role.ADMIN)
            .toModel();
    }
}

@Test
void testUserService() {
    User user = Instancio.create(UserTestData.activeUser());
    User admin = Instancio.create(UserTestData.adminUser());
    
    assertThat(user.getStatus()).isEqualTo(UserStatus.ACTIVE);
    assertThat(admin.getRole()).isEqualTo(Role.ADMIN);
}
```

### Collection Size Constraints

```java
@Test
void testCollectionConstraints() {
    Order order = Instancio.of(Order.class)
        .generate(field(Order::getItems), 
            gen -> gen.collection()
                     .minSize(3)
                     .maxSize(10)
                     .nullableElements())
        .create();
    
    assertThat(order.getItems()).hasSizeBetween(3, 10);
}
```
