# XDEV Commit Cards

Inspired by the [XDEV Commit Cards](https://commit-cards.xdev.software/latest/) by [XDEV Software](https://xdev.software).
32 cards for clean, sustainable code — one tip per commit.

> **Attribution:** The card topics are inspired by XDEV Software's Commit Cards. The original XDEV Commit Cards are copyright XDEV Software GmbH. All advice text and code examples in this file are original work — no content is copied from XDEV's materials. If you wish to use the original XDEV cards, visit [commit-cards.xdev.software](https://commit-cards.xdev.software/latest/) or purchase the [physical card set](https://xdev.software/en/about-us/xdev-commit-cards).

---

## 1. Automate Boring Tasks
**Advice:** If a change is worth doing manually more than twice, it deserves automation. Identify repetitive processes — building, testing, deploying, data manipulation — and automate them.
**Good:** Reusable methods, scripts, CI/CD pipelines, scheduled jobs.
**Bad:** Doing the same manual steps over and over.
**Commit check:** Can the changes you've done be automated? Is there a way to make this repeatable, reliable, and automatic?

## 2. Check Dependencies
**Advice:** Every dependency comes with costs — more code to maintain, possible security issues, slower builds. Projects collect libraries that nobody remembers why they exist.
**Good:** Regularly audit your `pom.xml` or `build.gradle`. Eliminate unused or redundant libraries. Keep versions current. Prefer well-maintained, widely-used libraries.
**Bad:** Legacy Apache Commons Lang 2.6 added solely for `StringUtils.isEmpty()` when `str == null || str.isBlank()` works natively.
**Commit check:** Did I modify dependencies? Is each one truly necessary? Are all versions current and secure?

## 3. Checkstyles and Auto-Formatters
**Advice:** Consistent code style is not just cosmetic. It makes code easier to read, reduces mistakes, and avoids unnecessary debates. Use checkstyle and auto-formatter tools.
**Good:** Checkstyle, Spotless, or IDE formatters enforced automatically during builds.
**Bad:** Relying on manual memory to maintain consistency across a team.
**Commit check:** Never commit code that violates style rules. Automate formatting so your team focuses on logic, not indentation.

## 4. Check Your Functions
**Advice:** Every function should have a clear, single purpose. If it does more than one thing, it's harder to read, test, and maintain. Functions should be small, focused, with minimal arguments (ideally 1-2) and no side effects.
**Bad:**
```java
public void processOrder(Order order, User user) {
    if(order.isValid()) {
        order.applyDiscount(user.getDiscount());
        repository.save(order);
        userNotification.notify(user);
    }
}
```
**Good:**
```java
public void processOrder(Order order, User user) {
    if (!order.isValid()) return;
    applyDiscount(order, user);
    saveOrder(order);
    notifyUser(user);
}
```
**Commit check:** Are your functions small, purposeful, and clean? Refactor if not.

## 5. Comments
**Advice:** Comments should explain *why* code exists, not what it does. Well-placed comments communicate intent, assumptions, and decisions that aren't obvious from the code.
**Bad:** `int total = a + b; // add a and b`
**Good:** `// Total includes base amount + surcharge for premium users`
**Commit check:** Would someone unfamiliar with this code understand why it exists?

## 6. Composition over Inheritance
**Advice:** Favor composition over inheritance whenever possible. Inheritance creates tight coupling. Composition lets you build complex behavior by combining small, focused objects.
**Bad:** `LoggingPrinter extends Printer` overriding `print()` to add logging.
**Good:** `LoggingPrinter` holds separate `Printer` and `Logger` instances, composing their behavior.
**Commit check:** Could composition replace inheritance here? Use inheritance only for genuine "is-a" relationships.

## 7. Credentials
**Advice:** Never commit credentials like API keys, passwords, or tokens into your repository. Once committed, they persist in git history forever.
**Bad:** `private static final String API_KEY = "12345-SECRET-KEY";`
**Good:** `@Value("${WEATHER_API_KEY}") private String apiKey;` — loaded from environment or secret store.
**Commit check:** If you are about to commit a string that looks like a secret, stop. Move it to configuration.

## 8. Coupled Classes
**Advice:** When classes are tightly coupled, changes to one force changes to others. Reduce dependencies by coding against interfaces, not concrete implementations.
**Bad:**
```java
private final EmailSender emailSender = new EmailSender();
```
**Good:**
```java
public class OrderService {
    private final Notifier notifier;
    public OrderService(Notifier notifier) {
        this.notifier = notifier;
    }
}
```
**Commit check:** If I change this class, how many others break with it?

## 9. Documentation
**Advice:** Code without documentation might work today, but it will confuse you or your teammates tomorrow. Leave precise hints as close to the code as possible.
**Bad:** `public void doIt(Order o) { // process }`
**Good:** Javadoc explaining intent and scope: what the method does and why, with clear naming.
**Commit check:** Would someone reviewing this code months later understand both what it does and why?

## 10. Don't Be Afraid of Long Names
**Advice:** A long, descriptive name saves more time than a short one that forces readers to guess. Names serve as documentation that stays in sync with code.
**Bad:** `int d;` `int t;` `double calc(int a, int b) { ... }`
**Good:** `int daysUntilExpiration;` `double calculateMonthlyInterest(int principalAmount, int numberOfMonths)`
**Commit check:** Would a developer unfamiliar with this code understand its purpose without comments? Rename anything ambiguous.

## 11. Don't Ignore Exceptions
**Advice:** Exceptions are signals that something went wrong. Ignoring them is like pretending a car's warning light isn't on. Always respond meaningfully — log, wrap, or rethrow.
**Bad:** Empty catch blocks that silently swallow exceptions.
**Good:** Log with context, throw custom exceptions, or document why it's safe to ignore.
**Commit check:** Am I ignoring an exception? What is the safe and meaningful way to handle it?

## 12. Don't Leave Broken Windows
**Advice:** Treat every code touch as a cleanup opportunity. Fix messy, confusing, or inconsistent code before committing. Leaving "broken windows" signals that shortcuts are acceptable.
**Bad:**
```java
public void process(Data d) {
    if(d != null) {
        // complicated nested logic
    }
}
```
**Good:**
```java
public void process(Data data) {
    if (data == null) return;
    validate(data);
    transform(data);
    persist(data);
}
```
**Commit check:** If someone read this in isolation, would it be clear, safe, and consistent?

## 13. Don't Reinvent the Wheel
**Advice:** Many problems have been solved, tested, and optimized by others. Writing your own versions wastes time and introduces bugs. Use proven libraries.
**Bad:** Custom password hashing: `Integer.toHexString(password.hashCode())`
**Good:** `BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();`
**Balance:** Don't add heavy dependencies for trivial methods, but adopt established libraries for complex problems.
**Commit check:** Did I just write code that already exists in a safe, tested library?

## 14. DRY (Don't Repeat Yourself)
**Advice:** Avoid duplicating logic. Duplicated code means future changes need updates in multiple locations, increasing error risk.
**Bad:** VAT calculation logic repeated in multiple methods.
**Good:** Centralize in a dedicated class with a constant. All calculations reference one source.
**Nuance:** Avoid premature abstraction. Minor duplication is sometimes more maintainable than complex solutions.
**Commit check:** Am I repeating logic that might need to change? Can I extract it safely without overengineering?

## 15. Enums Instead of Integer
**Advice:** Using raw integers to represent types, statuses, or categories creates confusion and bugs. Replace magic numbers with enums.
**Bad:** `if (status == 3)` — what does 3 mean?
**Good:** `if (status == OrderStatus.SHIPPED)` — self-documenting, type-safe.
**Commit check:** Replace any integer constants used as labels or identifiers with named enums.

## 16. ETC (Easier To Change)
**Advice:** Write code that adapts to future requirements without rigid, overengineered structures. Overengineering makes small changes painful.
**Bad:** A `FileProcessor` anticipating all file types with factories and unused methods for XML and JSON.
**Good:** Simple `FileProcessor` with only essential dependencies. Add methods as needed without touching factories.
**Commit check:** Did I make this code easier to extend or modify without adding unnecessary layers?

## 17. Fail Fast
**Advice:** A dead program does less damage than a crippled one. Detect problems immediately rather than allowing software to continue in an invalid state.
**Bad:**
```java
public void setAge(int age) {
    if (age < 0) { age = 0; } // silently corrects
    this.age = age;
}
```
**Good:**
```java
public void setAge(int age) {
    if (age < 0) throw new IllegalArgumentException("Age cannot be negative: " + age);
    this.age = age;
}
```
**Commit check:** Does my code detect invalid states right away, or allow problematic conditions to persist silently?

## 18. Finish What You Started
**Advice:** Always ensure every resource you open gets properly closed. Files, connections, streams, threads — if you opened it, close it.
**Good:** Try-with-resources for `AutoCloseable` classes. Finally blocks when try-with-resources isn't viable.
**Commit check:** Verify that every commit leaves code in a state where no resource can leak.

## 19. Generics
**Advice:** Generics are your safety net in Java. They enforce type safety at compile time so you don't get `ClassCastException` at runtime.
**Bad:** Raw types: `List rawList = new ArrayList()` with unsafe casts.
**Good:** `List<Burger>` — parameterized collections. Bounded wildcards `<T extends Burger>` for reusable methods.
**Commit check:** Replace raw types, favor collections over arrays, apply bounded wildcards for reusability.

## 20. KISS (Keep It Simple and Stupid)
**Advice:** Write code that prioritizes clarity over cleverness. Simplicity reduces bugs and improves maintainability.
**Bad:** Dense, compact logic sacrificing readability — nested conditions, complex boolean expressions.
**Good:** Step-by-step logic with early returns. Readable checks that tell the story.
**Commit check:** Is this code written for humans to understand, or for the compiler to execute?

## 21. Minimize Mutability
**Advice:** Immutable objects are your safest friend. When fields don't change after creation, code becomes predictable, thread-safe, and bug-resistant.
**Bad:** Mutable class with setters allowing post-creation modification.
**Good:** Java records: `public record User(String name, int age) {}` — immutable by design.
**Commit check:** Could this object be modified unexpectedly elsewhere? If yes, reduce mutability.

## 22. Principle of Least Surprise
**Advice:** Code should do what a reasonable developer expects. Predictability over cleverness. Explicit behavior over hidden magic.
**Bad:**
```java
public User findUser(String id) {
    return Optional.ofNullable(database.get(id))
        .orElseGet(() -> new User("default")); // surprise!
}
```
**Good:**
```java
public Optional<User> findUser(String id) {
    return Optional.ofNullable(database.get(id));
}
```
**Commit check:** Will another developer find this code does exactly what they expect?

## 23. Prefer Data Structures
**Advice:** Use named, structured data types instead of raw maps, lists, or generic objects. This makes code self-explanatory and safer.
**Bad:** `Map<String, Object> user = new HashMap<>(); user.put("id", 42);` — opaque structure.
**Good:** `public record User(int id, boolean active) {}` — explicit, immutable, type-safe.
**Commit check:** Did I use raw collections where a record or class would communicate intent better?

## 24. Refactoring
**Advice:** Refactoring improves code structure, readability, and safety without changing behavior. Can this be clearer, safer, or more maintainable?
**Bad:** `System.out.println()` for debugging, no null safety.
**Good:** SLF4J logging with parameterized messages, input validation.
**Commit check:** Does this commit leave the codebase safer and more maintainable?

## 25. Reusable Code
**Advice:** Code should be usable in multiple contexts without significant effort. When reuse is difficult, developers copy-paste, creating duplication.
**Bad:** Hardcoding file paths: `new PrintWriter("/tmp/report.txt")`
**Good:** Accept parameters: `generateReport(List<String> data, Path outputFile)`
**Commit check:** Could someone use this in a different context easily?

## 26. Reversibility
**Advice:** Design code to remain flexible as requirements evolve. Avoid locking into rigid approaches that are expensive to change.
**Bad:** Hard-coded payment logic tightly coupled to a service class.
**Good:** `PaymentProcessor` interface with injected implementations. New payment methods added without modifying existing code.
**Commit check:** Does this code allow straightforward modifications if requirements change?

## 27. Security
**Advice:** Security must be integrated into every commit. Handling sensitive data, validating inputs, and keeping dependencies current are non-negotiable.
**Bad:** `database.save(username, password);` — plain text passwords.
**Good:** `passwordEncoder.encode(password)` — hashed with strong algorithms.
**Commit check:** Could this expose sensitive data or create a vulnerability?

## 28. Single Purpose Class
**Advice:** A single-purpose class does exactly one thing and does it well. When a class tries to do too much, it becomes hard to understand, test, and maintain.
**Bad:** `UserManager` handling user creation, email sending, and activity logging.
**Good:** `UserService`, `EmailService`, `ActivityLogger` — each with one responsibility.
**Commit check:** Does each class handle only one responsibility? Are business logic and technical concerns separate?

## 29. Static Factory Methods
**Advice:** Static factory methods are an alternative to constructors that make code clearer, more flexible, and easier to maintain.
**Bad:** Overloaded constructors without descriptive meaning.
**Good:** `User.of(name, age)` and `User.anonymous()` — intent is clear from the name.
**Commit check:** Could this object creation be clearer or more flexible with a factory method?

## 30. Ubiquitous Language
**Advice:** Your code speaks the same language as the business. Every class, method, and variable should use terms everyone on the project understands.
**Bad:** `class TransactionProcessor { void doStuff(OrderWrapper ow) { ... } }`
**Good:** `class OrderService { void processOrder(Order order) { ... } }` with `enum InvoiceStatus { PENDING, PAID, CANCELLED }`
**Commit check:** Would someone from the business understand what this code is doing? If naming requires explanation, rename it.

## 31. Understanding Your Code
**Advice:** You should be able to explain every line to a colleague without hesitation. Writing code you don't fully grasp leads to bugs and technical debt.
**Bad:** `double result = x * 0.453592;` — magic number, unclear intent.
**Good:** `double poundsToKilograms(double pounds) { return pounds * 0.453592; }` — self-explanatory.
**Commit check:** Could a teammate understand this without asking me? If not, refactor.

## 32. Write Tests
**Advice:** Whenever possible, write tests. Even the most mundane tests can help. Three levels: unit (fast, isolated), integration (components together), end-to-end (real user behavior).
**Good:** Treat each commit as a milestone. If it changes behavior, a test should prove it works.
**Commit check:** Does this commit modify behavior? If yes, does an accompanying test validate the change?