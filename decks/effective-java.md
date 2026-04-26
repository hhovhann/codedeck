# Effective Java

Key items from Joshua Bloch's "Effective Java" — Java-specific best practices that prevent common bugs and improve code quality.

---

## 1. Static Factory Methods over Constructors
**Advice:** Consider static factory methods instead of constructors. They have names, can return subtypes, don't require creating new objects, and can return cached instances.
**Bad:**
```java
Boolean flag = new Boolean("true"); // always creates new object
```
**Good:**
```java
Boolean flag = Boolean.valueOf("true"); // cached, descriptive
Optional<User> user = Optional.of(value);
List<String> names = List.of("Alice", "Bob");
```
**Commit check:** Could a static factory method make this object creation clearer or more efficient?

## 2. Builder Pattern for Many Parameters
**Advice:** When a constructor has more than 3-4 parameters, use the Builder pattern. Telescoping constructors are hard to read; JavaBeans (setters) allow inconsistent state.
**Bad:**
```java
new Pizza(2, true, false, true, false, true, "thin"); // what does each arg mean?
```
**Good:**
```java
Pizza.builder()
    .size(2).cheese(true).pepperoni(true)
    .mushrooms(true).crust("thin")
    .build();
```
**Commit check:** Do any of my constructors have more than 3 parameters? Would a builder be clearer?

## 3. Enforce Noninstantiability with Private Constructor
**Advice:** Utility classes (only static methods) should not be instantiable. Add a private constructor to prevent accidental instantiation and subclassing.
**Bad:**
```java
public class MathUtils {
    public static double square(double x) { return x * x; }
    // anyone can do: new MathUtils() — meaningless
}
```
**Good:**
```java
public class MathUtils {
    private MathUtils() { throw new AssertionError("No instances"); }
    public static double square(double x) { return x * x; }
}
```
**Commit check:** Do my utility classes have private constructors?

## 4. Prefer Try-with-Resources
**Advice:** Always use try-with-resources for objects that implement `AutoCloseable`. It's shorter, cleaner, and doesn't suppress exceptions from close().
**Bad:**
```java
InputStream in = new FileInputStream(src);
try {
    OutputStream out = new FileOutputStream(dst);
    try {
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
    } finally { out.close(); }
} finally { in.close(); }
```
**Good:**
```java
try (InputStream in = new FileInputStream(src);
     OutputStream out = new FileOutputStream(dst)) {
    in.transferTo(out);
}
```
**Commit check:** Am I using try-finally where try-with-resources would be cleaner?

## 5. Override equals and hashCode Together
**Advice:** If you override `equals()`, you must override `hashCode()`. Objects that are equal must have the same hash code. Violating this breaks HashMap, HashSet, and any hash-based collection.
**Bad:**
```java
public class PhoneNumber {
    @Override public boolean equals(Object o) { ... }
    // hashCode not overridden — broken in HashMaps!
}
```
**Good:**
```java
public class PhoneNumber {
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { return Objects.hash(areaCode, prefix, lineNum); }
}
// Or use: records, @Data (Lombok), @EqualsAndHashCode
```
**Commit check:** Did I override equals without hashCode (or vice versa)?

## 6. Minimize Scope of Local Variables
**Advice:** Declare variables where they are first used, not at the top of the method. Prefer for-each loops over indexed loops. Keep variables in the narrowest scope possible.
**Bad:**
```java
public void process() {
    String name;
    int count;
    List<Order> orders;
    // ... 20 lines later ...
    name = getCustomerName();
}
```
**Good:**
```java
public void process() {
    String name = getCustomerName(); // declared where used
    List<Order> orders = repository.findByCustomer(name);
    for (Order order : orders) { ... } // for-each, not indexed
}
```
**Commit check:** Are my variables declared close to where they're used?

## 7. Prefer Interfaces to Abstract Classes
**Advice:** Interfaces allow multiple implementations without class hierarchy constraints. Since Java 8, interfaces can have default methods. Use abstract classes only when you need shared state or non-public methods.
**Bad:** An abstract class hierarchy that forces single inheritance and tight coupling.
**Good:** Interface with default methods for shared behavior. Companion abstract class for implementors that want a head start (e.g., `AbstractList`).
**Commit check:** Am I using an abstract class where an interface would give more flexibility?

## 8. Use Enums Instead of int Constants
**Advice:** Java enums are full-fledged classes. They provide type safety, can have methods and fields, and work with switch expressions. Never use `public static final int` for a fixed set of values.
**Bad:**
```java
public static final int SEASON_SPRING = 0;
public static final int SEASON_SUMMER = 1;
// no type safety — can pass any int
```
**Good:**
```java
public enum Season { SPRING, SUMMER, FALL, WINTER }
// type-safe, iterable, serializable
```
**Commit check:** Am I using int/String constants where an enum would be safer?

## 9. Favor Composition over Inheritance
**Advice:** Inheritance breaks encapsulation — the subclass depends on the superclass implementation. Use composition (wrapper/decorator) unless you have a true "is-a" relationship.
**Bad:** `InstrumentedHashSet extends HashSet` — breaks when HashSet internals change.
**Good:** `InstrumentedSet` wraps a `Set` instance, delegating calls and adding instrumentation.
**Commit check:** Am I extending a class where wrapping it would be safer?

## 10. Return Empty Collections, Not Null
**Advice:** Never return null for a collection or array. Return `Collections.emptyList()`, `List.of()`, or an empty array. This eliminates null checks for every caller.
**Bad:**
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```
**Good:**
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? List.of() : new ArrayList<>(cheesesInStock);
}
```
**Commit check:** Do any of my methods return null where an empty collection would work?

## 11. Use Optional for Potentially Absent Return Values
**Advice:** Return `Optional<T>` when a method may legitimately have no result. Never return `Optional.of(null)`, never use Optional for fields or parameters.
**Bad:**
```java
public User findById(long id) { return map.get(id); } // returns null
```
**Good:**
```java
public Optional<User> findById(long id) { return Optional.ofNullable(map.get(id)); }
```
**Commit check:** Am I returning null from a method where Optional would be more explicit?

## 12. Minimize Mutability
**Advice:** Make classes immutable unless there's a compelling reason not to. Immutable classes are simpler, thread-safe, and can be shared freely. Use records for simple data carriers.
**Bad:**
```java
public class Complex { private double re; private double im; /* setters */ }
```
**Good:**
```java
public record Complex(double re, double im) {
    public Complex plus(Complex other) { return new Complex(re + other.re, im + other.im); }
}
```
**Commit check:** Can this class be made immutable? Could it be a record?