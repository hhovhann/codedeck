# Design Patterns

Essential GoF (Gang of Four) design patterns every developer should recognize and apply when appropriate.

---

## 1. Strategy Pattern
**Advice:** When you have multiple algorithms for the same task and need to switch between them, encapsulate each in a class behind a common interface. Eliminates if/else chains based on type.
**Bad:**
```java
public double calculateDiscount(Order order) {
    if (order.getType().equals("VIP")) return order.getTotal() * 0.2;
    else if (order.getType().equals("REGULAR")) return order.getTotal() * 0.1;
    else if (order.getType().equals("NEW")) return order.getTotal() * 0.05;
}
```
**Good:**
```java
public interface DiscountStrategy { double calculate(Order order); }
public class VipDiscount implements DiscountStrategy { ... }
public class RegularDiscount implements DiscountStrategy { ... }
// Selected at runtime: order.applyDiscount(discountStrategy);
```
**Commit check:** Do I have if/else chains selecting behavior by type? Would Strategy pattern be cleaner?

## 2. Observer Pattern
**Advice:** When one object changes and multiple others need to know, use the Observer pattern. The subject maintains a list of observers and notifies them on state changes. Decouples the event source from handlers.
**Bad:**
```java
public void completeOrder(Order order) {
    orderRepo.save(order);
    emailService.sendConfirmation(order);   // tight coupling
    inventoryService.updateStock(order);    // tight coupling
    analyticsService.trackOrder(order);     // tight coupling
}
```
**Good:**
```java
public void completeOrder(Order order) {
    orderRepo.save(order);
    eventPublisher.publish(new OrderCompletedEvent(order));
    // Listeners handle email, inventory, analytics independently
}
```
**Commit check:** Am I directly calling multiple services after a state change? Would events be cleaner?

## 3. Factory Pattern
**Advice:** When object creation is complex or depends on runtime conditions, encapsulate it in a factory. Clients don't need to know which concrete class is instantiated.
**Bad:**
```java
Notification notification;
if (channel.equals("email")) notification = new EmailNotification(smtp, template);
else if (channel.equals("sms")) notification = new SmsNotification(twilioClient);
else if (channel.equals("push")) notification = new PushNotification(firebaseClient);
```
**Good:**
```java
public class NotificationFactory {
    public Notification create(String channel) {
        return switch (channel) {
            case "email" -> new EmailNotification(smtp, template);
            case "sms" -> new SmsNotification(twilioClient);
            case "push" -> new PushNotification(firebaseClient);
            default -> throw new IllegalArgumentException("Unknown: " + channel);
        };
    }
}
```
**Commit check:** Is my object creation logic complex? Would a factory make it clearer?

## 4. Builder Pattern
**Advice:** When constructing complex objects with many optional parameters, use a Builder. It makes construction readable, enforces required fields, and prevents telescoping constructors.
**Bad:**
```java
new Report("Q4 Sales", "2024-01-01", "2024-03-31", true, false, "PDF", null, 50, true);
```
**Good:**
```java
Report.builder()
    .title("Q4 Sales")
    .dateRange("2024-01-01", "2024-03-31")
    .includeCharts(true)
    .format("PDF")
    .maxPages(50)
    .build();
```
**Commit check:** Do I have constructors with more than 3-4 parameters? Would a builder be clearer?

## 5. Decorator Pattern
**Advice:** When you need to add behavior to an object dynamically without modifying its class, wrap it in a decorator that implements the same interface. Avoids explosion of subclasses.
**Bad:** `LoggingEncryptingCompressingFileWriter extends FileWriter` — combinatorial explosion.
**Good:**
```java
Writer writer = new FileWriter("output.txt");
writer = new BufferingWriter(writer);
writer = new EncryptingWriter(writer);
writer = new LoggingWriter(writer); // each wraps the previous
```
**Commit check:** Am I adding behavior through inheritance? Would wrapping/decorating be more flexible?

## 6. Template Method Pattern
**Advice:** When multiple classes share the same algorithm structure but differ in specific steps, define the skeleton in a base class and let subclasses override the varying steps.
**Bad:** Copy-pasting the same algorithm in multiple classes, changing only one or two steps.
**Good:**
```java
public abstract class DataProcessor {
    public final void process() {  // template method
        readData();
        transformData();   // override per subclass
        writeResults();
    }
    protected abstract void transformData();
}
```
**Commit check:** Do I have duplicated algorithm structures across classes? Would a template method reduce duplication?

## 7. Adapter Pattern
**Advice:** When you need to use a class whose interface doesn't match what your code expects, create an adapter that translates between the two interfaces. Essential for integrating third-party libraries.
**Bad:** Modifying your domain code to match a third-party library's interface.
**Good:**
```java
// Your interface
public interface NotificationSender { void send(Message msg); }

// Third-party library
public class SlackClient { public void postMessage(String channel, String text) { ... } }

// Adapter
public class SlackNotificationAdapter implements NotificationSender {
    private final SlackClient client;
    public void send(Message msg) { client.postMessage(msg.getChannel(), msg.getText()); }
}
```
**Commit check:** Am I coupling my code to a third-party interface? Would an adapter protect me from changes?

## 8. Singleton Pattern (Use With Caution)
**Advice:** Singleton ensures a class has only one instance. In Spring, beans are singletons by default. Avoid hand-rolled singletons — they're hard to test and create hidden global state. Let the DI container manage lifecycle instead.
**Bad:**
```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) instance = new DatabaseConnection();
        return instance;
    }
}
```
**Good:** Let Spring manage it as a `@Component` or `@Bean` — injectable, testable, lifecycle-managed.
**Commit check:** Am I creating a hand-rolled singleton? Would a Spring-managed bean be better?