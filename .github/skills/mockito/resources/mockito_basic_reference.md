# Mockito Basic Reference

## Most Commonly Used Features

### Creating Mocks with Annotations

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testGetUser() {
        when(userRepository.findById(1L))
            .thenReturn(Optional.of(new User(1L, "John")));
        
        User user = userService.getUser(1L);
        
        assertThat(user.getName()).isEqualTo("John");
    }
}
```

### Manual Mock Creation

```java
@Test
void testWithManualMock() {
    UserRepository repository = mock(UserRepository.class);
    UserService service = new UserService(repository);
    
    when(repository.findById(1L))
        .thenReturn(Optional.of(new User(1L, "John")));
    
    User user = service.getUser(1L);
    assertThat(user.getName()).isEqualTo("John");
}
```

### Basic Stubbing

```java
@Test
void testStubbing() {
    // Return specific value
    when(repository.findById(1L))
        .thenReturn(Optional.of(new User(1L, "John")));
    
    // Return multiple values in sequence
    when(repository.count())
        .thenReturn(10L)
        .thenReturn(20L)
        .thenReturn(30L);
}
```

### Throwing Exceptions

```java
@Test
void testExceptionStubbing() {
    when(repository.findById(999L))
        .thenThrow(new EntityNotFoundException("User not found"));
    
    assertThatThrownBy(() -> service.getUser(999L))
        .isInstanceOf(EntityNotFoundException.class)
        .hasMessage("User not found");
}
```

### Void Methods

```java
@Test
void testVoidMethod() {
    // Void methods that should do nothing
    doNothing().when(emailService).sendEmail(any());
    
    // Void methods that should throw exception
    doThrow(new EmailException()).when(emailService).sendEmail(any());
}
```

### Argument Matchers

```java
@Test
void testArgumentMatchers() {
    // Any matcher
    when(repository.findById(any(Long.class)))
        .thenReturn(Optional.of(new User()));
    
    when(repository.findByEmail(anyString()))
        .thenReturn(Optional.of(new User()));
    
    // Specific value
    when(repository.findById(eq(1L)))
        .thenReturn(Optional.of(new User(1L, "John")));
    
    // Null/NotNull
    when(repository.findByEmail(isNull()))
        .thenReturn(Optional.empty());
    
    when(repository.findByEmail(notNull()))
        .thenReturn(Optional.of(new User()));
}
```

### Basic Verification

```java
@Test
void testVerification() {
    service.createUser(new User("john@example.com"));
    
    // Verify method was called
    verify(repository).save(any(User.class));
    
    // Verify method was never called
    verify(repository, never()).delete(any());
    
    // Verify number of invocations
    verify(repository, times(1)).save(any());
    verify(repository, atLeastOnce()).save(any());
    verify(repository, atLeast(2)).save(any());
    verify(repository, atMost(5)).save(any());
}
```

### Argument Captors

```java
@Test
void testArgumentCaptor() {
    ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
    
    service.createUser(new User("john@example.com"));
    
    verify(repository).save(userCaptor.capture());
    
    User capturedUser = userCaptor.getValue();
    assertThat(capturedUser.getEmail()).isEqualTo("john@example.com");
}
```

### Verification Order

```java
@Test
void testVerificationOrder() {
    InOrder inOrder = inOrder(repository, emailService);
    
    service.createUser(new User("john@example.com"));
    
    inOrder.verify(repository).save(any(User.class));
    inOrder.verify(emailService).sendWelcomeEmail(anyString());
}
```

### Verifying No Interactions

```java
@Test
void testNoInteractions() {
    // Verify no methods were called
    verifyNoInteractions(repository);
    
    // Verify no more interactions after specified ones
    verify(repository).findById(1L);
    verifyNoMoreInteractions(repository);
}
```

### BDD Style

```java
@Test
void testBDDStyle() {
    // Given
    given(repository.findById(1L))
        .willReturn(Optional.of(new User(1L, "John")));
    
    // When
    User user = service.getUser(1L);
    
    // Then
    then(repository).should().findById(1L);
    assertThat(user.getName()).isEqualTo("John");
}
```
