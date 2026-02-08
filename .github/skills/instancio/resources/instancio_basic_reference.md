# Instancio Basic Reference

## Most Commonly Used Features

### Simple Object Creation

```java
@Test
void testWithInstancio() {
    // Create a fully populated User object
    User user = Instancio.create(User.class);
    
    assertThat(user.getName()).isNotNull();
    assertThat(user.getEmail()).isNotNull();
    assertThat(user.getAge()).isGreaterThan(0);
}
```

### Creating Multiple Objects

```java
@Test
void testWithMultipleObjects() {
    // Create a list of 10 users
    List<User> users = Instancio.ofList(User.class).size(10).create();
    
    assertThat(users).hasSize(10);
    assertThat(users).allMatch(user -> user.getName() != null);
}

@Test
void testWithStream() {
    Stream<Product> products = Instancio.stream(Product.class).limit(5);
    assertThat(products).hasSize(5);
}
```

### Setting Specific Values

```java
@Test
void testWithCustomValues() {
    User user = Instancio.of(User.class)
        .set(field(User::getName), "John Doe")
        .set(field(User::getEmail), "john@example.com")
        .set(field(User::getAge), 30)
        .create();
    
    assertThat(user.getName()).isEqualTo("John Doe");
    assertThat(user.getEmail()).isEqualTo("john@example.com");
    assertThat(user.getAge()).isEqualTo(30);
}
```

### Generating Values

```java
@Test
void testWithGeneratedValues() {
    User user = Instancio.of(User.class)
        .generate(field(User::getEmail), gen -> gen.text().pattern("#a#a#a@example.com"))
        .generate(field(User::getAge), gen -> gen.ints().range(18, 65))
        .generate(field(User::getCreatedAt), gen -> gen.temporal().instant().past())
        .create();
    
    assertThat(user.getEmail()).matches("\\w{3}@example.com");
    assertThat(user.getAge()).isBetween(18, 65);
    assertThat(user.getCreatedAt()).isBefore(Instant.now());
}
```

### Supply Custom Values

```java
@Test
void testWithSupplier() {
    AtomicInteger counter = new AtomicInteger(1);
    
    List<User> users = Instancio.ofList(User.class)
        .size(5)
        .supply(field(User::getId), () -> (long) counter.getAndIncrement())
        .supply(field(User::getName), () -> "User-" + UUID.randomUUID())
        .create();
    
    assertThat(users).extracting(User::getId)
        .containsExactly(1L, 2L, 3L, 4L, 5L);
}
```

### Field Selectors

```java
@Test
void testWithFieldSelectors() {
    Order order = Instancio.of(Order.class)
        .set(field(Order::getStatus), OrderStatus.PENDING)
        .set(field("customerId"), "CUST-123")
        .ignore(field(Order::getProcessedAt))
        .create();
    
    assertThat(order.getStatus()).isEqualTo(OrderStatus.PENDING);
    assertThat(order.getProcessedAt()).isNull();
}
```

### Type Selectors

```java
@Test
void testWithTypeSelectors() {
    Order order = Instancio.of(Order.class)
        .generate(all(String.class), gen -> gen.string().length(10))
        .generate(all(BigDecimal.class), gen -> gen.bigDecimal().min(0).max(1000))
        .create();
    
    assertThat(order.getItems())
        .flatExtracting(Item::getName)
        .allMatch(name -> name.length() == 10);
}
```

### Collection Configuration

```java
@Test
void testWithListConfiguration() {
    Order order = Instancio.of(Order.class)
        .generate(field(Order::getItems), gen -> gen.collection().size(5))
        .create();
    
    assertThat(order.getItems()).hasSize(5);
}

@Test
void testWithListCustomization() {
    Order order = Instancio.of(Order.class)
        .generate(field(Order::getItems), gen -> gen.collection().minSize(3).maxSize(7))
        .create();
    
    assertThat(order.getItems()).hasSizeBetween(3, 7);
}
```

### Reusable Models

```java
class TestModels {
    static final Model<User> ADULT_USER = Instancio.of(User.class)
        .generate(field(User::getAge), gen -> gen.ints().range(18, 100))
        .set(field(User::getStatus), UserStatus.ACTIVE)
        .toModel();
}

@Test
void testWithModel() {
    User user = Instancio.of(TestModels.ADULT_USER)
        .set(field(User::getName), "John")
        .create();
    
    assertThat(user.getAge()).isGreaterThanOrEqualTo(18);
    assertThat(user.getName()).isEqualTo("John");
}
```

### Built-in Generators

```java
@Test
void testWithBuiltInGenerators() {
    User user = Instancio.of(User.class)
        // String generators
        .generate(field(User::getName), gen -> gen.string().length(5, 20))
        .generate(field(User::getEmail), gen -> gen.text().pattern("#a#a#a@#C#c#c.com"))
        
        // Numeric generators
        .generate(field(User::getAge), gen -> gen.ints().range(18, 65))
        .generate(field(User::getBalance), gen -> gen.doubles().range(0.0, 10000.0))
        
        // Temporal generators
        .generate(field(User::getBirthDate), gen -> gen.temporal().localDate().past())
        
        .create();
}
```
