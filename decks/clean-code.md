# Clean Code

Principles from Robert C. Martin's "Clean Code" — writing code that is readable, maintainable, and professional.

---

## 1. Meaningful Names
**Advice:** Names should reveal intent. A name should tell you why something exists, what it does, and how it's used. If a name requires a comment, the name is wrong.
**Bad:**
```java
int d; // elapsed time in days
List<int[]> list1;
Map<String, Object> dataMap;
```
**Good:**
```java
int elapsedTimeInDays;
List<Cell> flaggedCells;
Map<String, Customer> customersByEmail;
```
**Commit check:** Can someone understand every variable and method name without additional context?

## 2. Functions Should Be Small
**Advice:** Functions should do one thing, do it well, and do it only. They should be short — ideally under 20 lines. If you need a comment to explain a section, extract it into a well-named method.
**Bad:** A 150-line method with nested ifs, multiple loops, and comments separating "sections".
**Good:** A method that reads like a story — each line at the same level of abstraction, calling well-named helper methods.
**Commit check:** Are any of my functions longer than a screen? Do they do more than one thing?

## 3. One Level of Abstraction per Function
**Advice:** All statements in a function should be at the same level of abstraction. Don't mix high-level business logic with low-level string manipulation in the same method.
**Bad:**
```java
public void generateReport(Order order) {
    String header = "Order #" + order.getId() + "\n";
    header += "Date: " + new SimpleDateFormat("yyyy-MM-dd").format(order.getDate());
    calculateTotals(order);
    emailService.send(order.getCustomer().getEmail(), header + body);
}
```
**Good:**
```java
public void generateReport(Order order) {
    String header = formatHeader(order);
    String body = formatBody(order);
    sendReport(order.getCustomer(), header + body);
}
```
**Commit check:** Are my functions mixing high-level orchestration with low-level details?

## 4. Command-Query Separation
**Advice:** A function should either do something (command) or answer something (query), but not both. Functions that change state should not return values that describe state.
**Bad:**
```java
public boolean set(String attribute, String value) {
    // sets the attribute AND returns whether it existed
    if (attributes.containsKey(attribute)) {
        attributes.put(attribute, value);
        return true;
    }
    return false;
}
// Leads to confusing usage: if (set("username", "bob")) { ... }
```
**Good:**
```java
public boolean hasAttribute(String attribute) { ... }
public void setAttribute(String attribute, String value) { ... }
```
**Commit check:** Do any of my methods both change state and return a value?

## 5. Don't Use Flag Arguments
**Advice:** Boolean arguments loudly declare a function does more than one thing. Split it into two well-named functions instead.
**Bad:**
```java
render(true);
booking.process(false);
report.generate(true, false);
```
**Good:**
```java
renderForPrint();
booking.processWithoutNotification();
report.generateSummary();
```
**Commit check:** Do any of my methods take boolean parameters that change behavior?

## 6. Error Handling Is One Thing
**Advice:** If a function handles errors, it should do that and nothing else. Extract try/catch bodies into separate methods. Don't use error codes — use exceptions.
**Bad:**
```java
public void processFile(String path) {
    try {
        File file = openFile(path);
        List<String> lines = readLines(file);
        for (String line : lines) {
            Record record = parseLine(line);
            if (record.isValid()) {
                repository.save(record);
            } else {
                errorLog.write("Invalid: " + line);
            }
        }
    } catch (IOException e) {
        logger.error("Failed", e);
    }
}
```
**Good:**
```java
public void processFile(String path) {
    try {
        processRecords(path);
    } catch (IOException e) {
        logger.error("Failed to process file: {}", path, e);
    }
}

private void processRecords(String path) throws IOException {
    List<String> lines = Files.readAllLines(Path.of(path));
    lines.stream().map(this::parseLine).filter(Record::isValid).forEach(repository::save);
}
```
**Commit check:** Are my try/catch blocks wrapping focused, single-purpose code?

## 7. Avoid Output Arguments
**Advice:** If your function must change state, have it change the state of its owning object. Don't pass objects in to be modified — it's confusing.
**Bad:**
```java
appendFooter(report); // does this append TO report or append report to something?
```
**Good:**
```java
report.appendFooter();
```
**Commit check:** Do any of my functions modify their arguments instead of returning results?

## 8. The Boy Scout Rule
**Advice:** Leave the code cleaner than you found it. Every time you touch a file, make one small improvement — rename a variable, extract a method, remove dead code.
**Good:** While fixing a bug in `OrderService`, you also rename `proc()` to `processOrder()` and remove a commented-out line.
**Bad:** Ignoring the mess around your change because "it was already like that".
**Commit check:** Is the code around my change a little cleaner than before I touched it?

## 9. Don't Return Null
**Advice:** Returning null forces every caller to add null checks. Return empty collections, Optional, or throw exceptions instead. One missing null check = NullPointerException in production.
**Bad:**
```java
public List<Order> getOrders(User user) {
    if (user == null) return null;
    List<Order> orders = repository.findByUser(user);
    if (orders.isEmpty()) return null;
    return orders;
}
```
**Good:**
```java
public List<Order> getOrders(User user) {
    Objects.requireNonNull(user, "user must not be null");
    return repository.findByUser(user); // returns empty list, never null
}
```
**Commit check:** Do any of my methods return null where they could return Optional or an empty collection?

## 10. Don't Pass Null
**Advice:** Passing null into a method is worse than returning it. There's no good way to handle a null argument in most cases. Use `Objects.requireNonNull()` at method entry for public APIs.
**Bad:**
```java
calculator.calculateRate(null, null);
```
**Good:**
```java
public double calculateRate(Principal principal, Term term) {
    Objects.requireNonNull(principal);
    Objects.requireNonNull(term);
    ...
}
```
**Commit check:** Am I passing null to any method? Do my public methods validate non-null arguments?

## 11. Classes Should Be Small
**Advice:** Classes should have a small number of instance variables. Each method should use one or more of those variables. High cohesion means every variable is used by every method.
**Bad:** A 2000-line `UserManager` with 30 fields and 50 methods covering authentication, profile management, billing, and notifications.
**Good:** Small, focused classes: `AuthService`, `UserProfile`, `BillingService`, `NotificationService` — each under 200 lines.
**Commit check:** Does my class have too many responsibilities? Could it be split?

## 12. Newspaper Metaphor
**Advice:** Code should read like a newspaper article — the most important, high-level concepts at the top, details increasing as you go down. Public methods first, private helpers below.
**Good:** Public API methods at the top, each calling private methods defined immediately below them.
**Bad:** Random ordering of methods, private helpers scattered between public methods.
**Commit check:** If I read this file top-to-bottom, does the story make sense?

## 13. Avoid Mental Mapping
**Advice:** Readers shouldn't have to mentally translate your names into concepts they already know. Don't use single-letter variables (except in tiny loops), don't use abbreviations only you understand.
**Bad:**
```java
String r = request.getHeader("Referer");
Map<String, List<String>> hp = parseHeaders(req);
```
**Good:**
```java
String refererHeader = request.getHeader("Referer");
Map<String, List<String>> headersByName = parseHeaders(request);
```
**Commit check:** Would a new team member need a "translation guide" for my variable names?

## 14. Prefer Polymorphism to If/Else Chains
**Advice:** When you find yourself writing switch statements or long if/else chains based on type, consider replacing them with polymorphism.
**Bad:**
```java
public double calculatePay(Employee e) {
    switch (e.getType()) {
        case SALARIED: return e.getMonthlySalary();
        case HOURLY: return e.getHoursWorked() * e.getHourlyRate();
        case COMMISSIONED: return e.getBasePay() + e.getCommission();
    }
}
```
**Good:**
```java
public interface PayStrategy { double calculate(Employee e); }
// Each type has its own class implementing PayStrategy
```
**Commit check:** Do I have switch/if-else chains that dispatch on type? Would polymorphism be cleaner?

## 15. Tests Should Be F.I.R.S.T.
**Advice:** Tests should be Fast, Independent, Repeatable, Self-validating, and Timely (written before or with the production code).
**Bad:** Tests that depend on execution order, require network access, or need manual verification.
**Good:** Each test runs in milliseconds, sets up its own data, asserts its own results, and can run in any order on any machine.
**Commit check:** Are my tests fast, independent, and self-contained?