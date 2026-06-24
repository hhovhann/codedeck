# API Design Best Practices

Principles for designing REST APIs and service interfaces that are intuitive, consistent, and maintainable.

---

## 1. Use Nouns for Resources, Verbs for Actions
**Advice:** URLs should represent resources (nouns), not actions. Use HTTP methods (GET, POST, PUT, DELETE) for actions. The URL tells you WHAT, the method tells you HOW.
**Bad:** `POST /createUser`, `GET /getOrderById?id=123`, `POST /deleteUser/5`
**Good:** `POST /users`, `GET /orders/123`, `DELETE /users/5`
**Commit check:** Do my endpoints use nouns for resources and HTTP methods for actions?

## 2. Use Proper HTTP Status Codes
**Advice:** Don't return 200 for everything. Use the right status code: 201 Created, 204 No Content, 400 Bad Request, 404 Not Found, 409 Conflict, 422 Unprocessable Entity.
**Bad:**
```java
@PostMapping("/users")
public ResponseEntity<Map<String, String>> create(@RequestBody User user) {
    if (exists(user)) return ResponseEntity.ok(Map.of("error", "User exists")); // 200 for error!
    save(user);
    return ResponseEntity.ok(Map.of("status", "created")); // 200 instead of 201
}
```
**Good:**
```java
@PostMapping("/users")
public ResponseEntity<User> create(@RequestBody User user) {
    if (exists(user)) return ResponseEntity.status(CONFLICT).build();
    User created = save(user);
    return ResponseEntity.created(URI.create("/users/" + created.getId())).body(created);
}
```
**Commit check:** Am I returning the correct HTTP status codes for success AND error cases?

## 3. Version Your API
**Advice:** Always version your API from day one. Breaking changes should go in a new version. Clients need time to migrate.
**Bad:** `/api/users` with no version — breaking changes break all clients simultaneously.
**Good:** `/api/v1/users` — version in URL path. Or use header versioning: `Accept: application/vnd.myapp.v1+json`.
**Commit check:** Does my new/changed endpoint have a version? Does my change break existing clients?

## 4. Consistent Naming Conventions
**Advice:** Pick one naming convention and stick to it across all endpoints. Use lowercase with hyphens for URLs, camelCase or snake_case for JSON (pick one, never mix).
**Bad:** `/api/userAccounts`, `/api/order-items`, `/api/Product_Categories` — three conventions.
**Good:** `/api/user-accounts`, `/api/order-items`, `/api/product-categories` — consistent kebab-case.
**Commit check:** Does my new endpoint follow the existing naming convention?

## 5. Pagination for Collections
**Advice:** Never return unbounded collections. Always paginate list endpoints. Include total count, page number, and page size in the response.
**Bad:**
```java
@GetMapping("/orders")
public List<Order> getAll() { return repository.findAll(); } // 1 million orders
```
**Good:**
```java
@GetMapping("/orders")
public Page<Order> getAll(@RequestParam(defaultValue = "0") int page,
                          @RequestParam(defaultValue = "20") int size) {
    return repository.findAll(PageRequest.of(page, size));
}
```
**Commit check:** Do my list endpoints support pagination? What happens with 100k records?

## 6. Validate Input at the Boundary
**Advice:** Validate all incoming data at the API layer. Use Bean Validation annotations (@NotNull, @Size, @Email). Return clear error messages telling the caller exactly what's wrong.
**Bad:**
```java
@PostMapping("/users")
public void create(@RequestBody User user) {
    repository.save(user); // NullPointerException when name is null
}
```
**Good:**
```java
@PostMapping("/users")
public ResponseEntity<User> create(@Valid @RequestBody CreateUserRequest request) { ... }

public record CreateUserRequest(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email
) {}
```
**Commit check:** Am I validating all input at the API boundary? Are error messages helpful?

## 7. Use DTOs, Not Entities
**Advice:** Never expose database entities directly in API responses. Use DTOs to control what's exposed, prevent over-fetching, and decouple your API from your data model.
**Bad:**
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id); // exposes password hash, internal IDs, etc.
}
```
**Good:**
```java
@GetMapping("/users/{id}")
public UserResponse getUser(@PathVariable Long id) {
    User user = userRepository.findById(id);
    return new UserResponse(user.getId(), user.getName(), user.getEmail());
}
```
**Commit check:** Am I exposing internal entity fields in my API response?

## 8. Idempotent Operations
**Advice:** GET, PUT, and DELETE should be idempotent — calling them multiple times produces the same result. This is critical for retry logic and network reliability.
**Bad:** `DELETE /orders/123` succeeds first time, throws 500 on retry because order is gone.
**Good:** `DELETE /orders/123` returns 204 first time, returns 204 (or 404) on retry — no error, no side effects.
**Commit check:** What happens if my endpoint is called twice? Does it produce the same result?