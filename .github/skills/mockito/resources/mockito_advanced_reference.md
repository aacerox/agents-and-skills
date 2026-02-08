# Mockito Advanced Reference

## Advanced Features

### Custom Matchers

```java
@Test
void testCustomMatcher() {
    when(repository.save(argThat(user -> 
        user.getEmail().endsWith("@example.com"))))
        .thenReturn(new User());
    
    // Alternative with lambda
    when(repository.findByAgeGreaterThan(intThat(age -> age >= 18)))
        .thenReturn(List.of(new User()));
}
```

### Combining Matchers

```java
@Test
void testCombiningMatchers() {
    // Use all matchers or all concrete values
    when(repository.update(eq(1L), any(User.class)))
        .thenReturn(true);
    
    when(repository.findByNameAndAge(anyString(), gt(18)))
        .thenReturn(List.of(new User()));
}
```

### Multiple Argument Captures

```java
@Test
void testMultipleCaptures() {
    ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
    
    service.createUsers(List.of(
        new User("john@example.com"),
        new User("jane@example.com")
    ));
    
    verify(repository, times(2)).save(userCaptor.capture());
    
    List<User> capturedUsers = userCaptor.getAllValues();
    assertThat(capturedUsers).hasSize(2);
}
```

### Spies

```java
@Test
void testSpy() {
    UserService realService = new UserService(repository);
    UserService spyService = spy(realService);
    
    // Partial stubbing
    when(spyService.generateUserId()).thenReturn("USER-123");
    
    // Real method is called
    spyService.createUser(new User());
    
    verify(spyService).generateUserId();
}

@Test
void testSpyWithAnnotation() {
    @Spy
    private UserService userService = new UserService(repository);
    
    doReturn("USER-123").when(userService).generateUserId();
}
```

### Custom Answers

```java
@Test
void testCustomAnswer() {
    when(repository.save(any(User.class)))
        .thenAnswer(invocation -> {
            User user = invocation.getArgument(0);
            user.setId(1L);
            user.setCreatedAt(Instant.now());
            return user;
        });
    
    User user = service.createUser(new User("john@example.com"));
    
    assertThat(user.getId()).isEqualTo(1L);
    assertThat(user.getCreatedAt()).isNotNull();
}
```

### Answer Shortcuts

```java
@Test
void testAnswerShortcuts() {
    // Return first argument
    when(repository.save(any()))
        .thenAnswer(AdditionalAnswers.returnsFirstArg());
    
    // Return second argument
    when(repository.update(any(), any()))
        .thenAnswer(AdditionalAnswers.returnsSecondArg());
    
    // Call real method
    when(repository.validate(any()))
        .thenCallRealMethod();
}
```

### Deep Stubs

```java
@Test
void testDeepStubs() {
    UserRepository repository = mock(UserRepository.class, RETURNS_DEEP_STUBS);
    
    when(repository.findById(1L).get().getName())
        .thenReturn("John");
    
    // Avoid in production - indicates design issues
}
```

### Mock Settings

```java
@Test
void testMockSettings() {
    UserRepository repository = mock(UserRepository.class, 
        withSettings()
            .name("UserRepositoryMock")
            .verboseLogging()
            .strictness(Strictness.STRICT_STUBS));
}
```

### Resetting Mocks

```java
@Test
void testResetMock() {
    when(repository.findById(1L))
        .thenReturn(Optional.of(new User()));
    
    // Reset mock
    reset(repository);
    
    // Previous stubbing is cleared
    assertThat(repository.findById(1L)).isEmpty();
}
```

### Consecutive Calls

```java
@Test
void testConsecutiveCalls() {
    when(repository.findById(1L))
        .thenReturn(Optional.of(new User("John")))
        .thenReturn(Optional.of(new User("Jane")))
        .thenThrow(new RuntimeException());
    
    assertThat(repository.findById(1L).get().getName()).isEqualTo("John");
    assertThat(repository.findById(1L).get().getName()).isEqualTo("Jane");
    assertThatThrownBy(() -> repository.findById(1L))
        .isInstanceOf(RuntimeException.class);
}
```

### Answer with Callback

```java
@Test
void testAnswerWithCallback() {
    doAnswer(invocation -> {
        String email = invocation.getArgument(0);
        System.out.println("Sending email to: " + email);
        return null;
    }).when(emailService).sendEmail(anyString());
    
    emailService.sendEmail("test@example.com");
}
```
