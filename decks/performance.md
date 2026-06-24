# Performance Best Practices

Principles for writing code that performs well without premature optimization. Measure first, optimize where it matters.

---

## 1. Don't Optimize Prematurely
**Advice:** "Premature optimization is the root of all evil" — Donald Knuth. Write clear, correct code first. Profile to find actual bottlenecks. Optimize only what measurements prove is slow.
**Bad:** Spending 2 days optimizing a method that's called once on startup and takes 50ms.
**Good:** Profiling reveals a method called 10,000 times per request takes 200ms total. Optimize that.
**Commit check:** Am I optimizing based on measurement, or based on assumption?

## 2. Use the Right Data Structure
**Advice:** Data structure choice has the biggest performance impact. Know the time complexity of operations you use most: HashMap O(1) lookup vs ArrayList O(n) search; LinkedList O(1) insert vs ArrayList O(n) insert.
**Bad:**
```java
List<User> users = new ArrayList<>();
// Then searching: users.stream().filter(u -> u.getId().equals(id)).findFirst();
// O(n) for every lookup
```
**Good:**
```java
Map<String, User> usersById = new HashMap<>();
// O(1) lookup: usersById.get(id);
```
**Commit check:** Am I using the right data structure for my access pattern?

## 3. Avoid N+1 Queries
**Advice:** The N+1 problem: loading a list (1 query) then loading related data for each item (N queries). This kills database performance. Use JOINs, batch fetching, or eager loading.
**Bad:**
```java
List<Order> orders = orderRepo.findAll(); // 1 query
for (Order order : orders) {
    List<Item> items = itemRepo.findByOrderId(order.getId()); // N queries
}
```
**Good:**
```java
List<Order> orders = orderRepo.findAllWithItems(); // 1 query with JOIN
// Or batch: itemRepo.findByOrderIds(orderIds); // 2 queries total
```
**Commit check:** Am I querying inside a loop? Could I batch or JOIN instead?

## 4. Cache Appropriately
**Advice:** Cache data that is expensive to compute and doesn't change often. But every cache is a source of bugs — stale data, memory pressure, cache invalidation complexity. Cache only when profiling proves it's needed.
**Bad:** Caching everything "just in case". No eviction policy. No invalidation strategy.
**Good:** Cache with TTL, bounded size, and clear invalidation rules. Monitor hit rates. Start without cache, add when measurements justify it.
**Commit check:** If I'm adding a cache, do I have: eviction policy, invalidation strategy, and metrics?

## 5. Lazy Loading vs Eager Loading
**Advice:** Load data when you need it (lazy) vs loading everything upfront (eager). Lazy reduces initial load time but may cause N+1. Eager increases initial load but prevents subsequent queries. Choose based on actual access patterns.
**Bad:** Eagerly loading a user's entire order history when you only need their name.
**Good:** Load user profile eagerly (always needed). Lazy-load order history (only on the orders page).
**Commit check:** Am I loading more data than this use case needs?

## 6. Mind String Concatenation
**Advice:** In loops, `+` for strings creates a new object each iteration. Use `StringBuilder` for loops, `String.format()` or text blocks for readability.
**Bad:**
```java
String result = "";
for (Item item : items) {
    result += item.getName() + ", "; // O(n^2) — new String each iteration
}
```
**Good:**
```java
String result = items.stream().map(Item::getName).collect(Collectors.joining(", "));
// Or: StringBuilder in a loop
```
**Commit check:** Am I concatenating strings in a loop? Should I use StringBuilder or Collectors.joining()?

## 7. Close Resources Promptly
**Advice:** Database connections, file handles, and HTTP connections are limited. Hold them for the shortest time possible. Use try-with-resources and connection pooling.
**Bad:** Opening a database connection at the start of a request and holding it through email sending, file processing, and external API calls.
**Good:** Get connection, execute query, return connection to pool. Don't hold connections during I/O to external services.
**Commit check:** Am I holding resources (connections, streams) longer than necessary?

## 8. Pagination for Large Data Sets
**Advice:** Never load an entire table into memory. Always paginate when processing or displaying large collections. Use LIMIT/OFFSET or cursor-based pagination.
**Bad:** `SELECT * FROM orders` followed by in-memory filtering and sorting.
**Good:** `SELECT * FROM orders WHERE status = ? ORDER BY created_at LIMIT 20 OFFSET 40` — filter and paginate at the database level.
**Commit check:** Could this query return millions of rows? Am I paginating?