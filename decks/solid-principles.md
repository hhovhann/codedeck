# SOLID Principles

The five fundamental principles of object-oriented design by Robert C. Martin. These make software more understandable, flexible, and maintainable.

---

## 1. Single Responsibility Principle (SRP)
**Advice:** A class should have only one reason to change. If a class handles user validation, persistence, and email notification, it has three reasons to change — and three chances to break unrelated things.
**Bad:**
```java
public class UserService {
    public void registerUser(User user) {
        if (!user.getEmail().contains("@")) throw new ValidationException("Invalid email");
        jdbcTemplate.update("INSERT INTO users ...", user.getName());
        emailClient.send(user.getEmail(), "Welcome!");
    }
}
```
**Good:**
```java
public class UserValidator { public void validate(User user) { ... } }
public class UserRepository { public void save(User user) { ... } }
public class WelcomeEmailSender { public void send(User user) { ... } }
public class UserRegistrationService {
    public void register(User user) {
        validator.validate(user);
        repository.save(user);
        emailSender.send(user);
    }
}
```
**Commit check:** Does any class I changed have more than one reason to change?

## 2. Open/Closed Principle (OCP)
**Advice:** Software entities should be open for extension, but closed for modification. You should be able to add new behavior without changing existing, tested code.
**Bad:**
```java
public double calculateArea(Object shape) {
    if (shape instanceof Circle c) return Math.PI * c.radius * c.radius;
    if (shape instanceof Rectangle r) return r.width * r.height;
    // Adding Triangle means modifying this method
}
```
**Good:**
```java
public interface Shape { double area(); }
public record Circle(double radius) implements Shape {
    public double area() { return Math.PI * radius * radius; }
}
public record Rectangle(double width, double height) implements Shape {
    public double area() { return width * height; }
}
// Adding Triangle = new class, no existing code changes
```
**Commit check:** Did I modify existing code to add new behavior? Could I extend instead?

## 3. Liskov Substitution Principle (LSP)
**Advice:** Subtypes must be substitutable for their base types without altering the correctness of the program. If a subclass overrides a method in a way that breaks the parent's contract, callers will be surprised.
**Bad:**
```java
public class Rectangle {
    public void setWidth(int w) { this.width = w; }
    public void setHeight(int h) { this.height = h; }
}
public class Square extends Rectangle {
    public void setWidth(int w) { this.width = w; this.height = w; } // breaks contract
    public void setHeight(int h) { this.width = h; this.height = h; }
}
```
**Good:**
```java
public interface Shape { int area(); }
public record Rectangle(int width, int height) implements Shape {
    public int area() { return width * height; }
}
public record Square(int side) implements Shape {
    public int area() { return side * side; }
}
```
**Commit check:** If I replace a parent class with my subclass, does everything still work correctly?

## 4. Interface Segregation Principle (ISP)
**Advice:** No client should be forced to depend on methods it does not use. Fat interfaces force implementors to provide stubs for irrelevant methods.
**Bad:**
```java
public interface Worker {
    void work();
    void eat();
    void sleep();
}
public class Robot implements Worker {
    public void work() { /* OK */ }
    public void eat() { /* robots don't eat */ }
    public void sleep() { /* robots don't sleep */ }
}
```
**Good:**
```java
public interface Workable { void work(); }
public interface Feedable { void eat(); }
public interface Restable { void sleep(); }
public class Robot implements Workable { public void work() { ... } }
public class Human implements Workable, Feedable, Restable { ... }
```
**Commit check:** Do any of my interfaces force implementors to provide empty or meaningless methods?

## 5. Dependency Inversion Principle (DIP)
**Advice:** High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details.
**Bad:**
```java
public class OrderService {
    private MySQLOrderRepository repo = new MySQLOrderRepository();
    private SmtpEmailSender sender = new SmtpEmailSender();
}
```
**Good:**
```java
public class OrderService {
    private final OrderRepository repo;
    private final NotificationSender sender;
    public OrderService(OrderRepository repo, NotificationSender sender) {
        this.repo = repo;
        this.sender = sender;
    }
}
```
**Commit check:** Do my high-level classes instantiate low-level classes directly, or do they depend on abstractions?