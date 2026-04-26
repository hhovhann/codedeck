# Testing Best Practices

Principles for writing tests that are reliable, maintainable, and actually catch bugs.

---

## 1. Arrange-Act-Assert (AAA)
**Advice:** Structure every test in three clear sections: set up the data (Arrange), execute the action (Act), verify the result (Assert). This makes tests readable and consistent.
**Bad:**
```java
@Test void test() {
    assertEquals("John", new UserService(new MockRepo()).findUser(1).getName());
}
```
**Good:**
```java
@Test void findUser_returnsUserWithCorrectName() {
    // Arrange
    UserService service = new UserService(mockRepo);
    when(mockRepo.findById(1)).thenReturn(new User(1, "John"));

    // Act
    User result = service.findUser(1);

    // Assert
    assertThat(result.getName()).isEqualTo("John");
}
```
**Commit check:** Does each test clearly separate setup, action, and verification?

## 2. Test Behavior, Not Implementation
**Advice:** Tests should verify what the code does, not how it does it internally. Testing implementation details makes tests brittle — they break when you refactor even though behavior hasn't changed.
**Bad:**
```java
@Test void testProcessOrder() {
    service.processOrder(order);
    verify(repo, times(1)).save(any()); // testing HOW
    verify(cache).invalidate("orders"); // testing internal detail
}
```
**Good:**
```java
@Test void processOrder_persistsOrderAndReturnsConfirmation() {
    OrderConfirmation result = service.processOrder(order);
    assertThat(result.getStatus()).isEqualTo(CONFIRMED);
    assertThat(repo.findById(order.getId())).isPresent(); // testing WHAT
}
```
**Commit check:** Would my test break if I refactored the implementation without changing behavior?

## 3. One Assertion per Concept
**Advice:** Each test should verify one logical concept. Multiple assertions are fine if they test the same concept. But testing user creation AND email sending AND logging in one test method hides failures.
**Bad:**
```java
@Test void testUserRegistration() {
    service.register(user);
    assertNotNull(repo.findByEmail(user.getEmail())); // persistence
    verify(emailService).sendWelcome(user); // email
    verify(auditLog).log("USER_CREATED", user.getId()); // audit
}
```
**Good:**
```java
@Test void register_persistsUser() { ... }
@Test void register_sendsWelcomeEmail() { ... }
@Test void register_logsAuditEvent() { ... }
```
**Commit check:** If this test fails, will the name and assertions immediately tell me what broke?

## 4. Use Descriptive Test Names
**Advice:** Test names should describe the scenario and expected outcome. Use `methodName_scenario_expectedBehavior` or a natural language format. Never use `test1`, `test2`.
**Bad:** `testProcess()`, `test1()`, `testUserService()`
**Good:** `processOrder_withInvalidAddress_throwsValidationException()`, `findUser_whenNotFound_returnsEmpty()`
**Commit check:** Can someone understand what each test verifies just from its name?

## 5. Tests Should Be Independent
**Advice:** Each test must run independently — no shared mutable state, no execution order dependency. Use @BeforeEach to set up fresh state for every test.
**Bad:** Test B depends on data created by Test A. Tests pass in order but fail when run individually.
**Good:** Each test creates its own data, verifies its own results, and cleans up after itself.
**Commit check:** Can each of my tests run alone, in any order, and pass?

## 6. Don't Test Private Methods Directly
**Advice:** If you feel the need to test a private method, it's a sign the class has too many responsibilities. Extract the logic into a separate class and test that publicly.
**Bad:** Using reflection to access private methods in tests.
**Good:** Extract complex private logic into a separate, testable class with public methods.
**Commit check:** Am I trying to test private methods? Should I extract a class instead?

## 7. Use Test Fixtures Wisely
**Advice:** Share setup code through @BeforeEach or helper methods, not through inheritance hierarchies. Create builder/factory methods for test data to keep tests readable.
**Bad:** 5-level test class hierarchy with scattered setup across `@BeforeAll` and `@BeforeEach` at every level.
**Good:**
```java
private User createTestUser() { return new User("test@example.com", "Test User"); }
private Order createTestOrder(User user) { return new Order(user, List.of(item1, item2)); }
```
**Commit check:** Is my test setup clear and localized, or scattered across an inheritance chain?

## 8. Test Edge Cases
**Advice:** Don't just test the happy path. Test: null/empty inputs, boundary values (0, MAX_INT, empty strings), error cases, concurrent access, and timeout scenarios.
**Bad:** Only testing `processOrder()` with a valid order.
**Good:** Also testing with null order, empty items, negative quantities, duplicate items, cancelled orders, expired discounts.
**Commit check:** Have I tested the unhappy paths and boundary conditions?

## 9. Keep Tests Fast
**Advice:** Slow tests don't get run. Use in-memory databases (H2), mock external services, avoid Thread.sleep(). If a test takes more than a second, question whether it's testing at the right level.
**Bad:** Integration test that starts a real database, seeds data, and waits for async processing.
**Good:** Unit test with mocked dependencies for logic. Reserve integration tests for boundary verification only.
**Commit check:** Do my tests run in seconds, not minutes? Am I testing at the right level?

## 10. Test Data Should Be Minimal and Meaningful
**Advice:** Use the minimum data needed to verify behavior. Every piece of test data should have a reason. Avoid copying production data dumps into tests.
**Bad:**
```java
User user = new User("John", "Doe", "john@example.com", 35, "123 Main St",
    "Springfield", "IL", "62704", "555-1234", "Manager", "IT", true, ...);
```
**Good:**
```java
User user = TestUsers.withEmail("john@example.com"); // only what matters for this test
```
**Commit check:** Does my test data include only what's relevant to the scenario being tested?