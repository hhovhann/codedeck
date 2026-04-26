# Clean Architecture

Principles from Robert C. Martin's "Clean Architecture" — structuring systems so business logic is independent of frameworks, databases, and UI.

---

## 1. The Dependency Rule
**Advice:** Source code dependencies must point inward — from outer layers (frameworks, DB, UI) toward inner layers (business rules). Inner layers must never reference outer layers.
**Bad:**
```java
// Domain entity depends on Spring and JPA
@Entity @Table(name = "orders")
public class Order {
    @Id @GeneratedValue private Long id;
    @Autowired private EmailService emailService; // framework in domain!
}
```
**Good:**
```java
// Domain entity is a plain object
public class Order {
    private final OrderId id;
    private final List<LineItem> items;
    public Money calculateTotal() { ... } // pure business logic
}
```
**Commit check:** Do my domain/business classes import framework packages (Spring, JPA, HTTP)?

## 2. Entities
**Advice:** Entities encapsulate enterprise-wide business rules. They are the least likely to change when something external changes. An entity should work the same whether used in a web app, CLI, or batch job.
**Bad:** Business rules scattered across controllers, services, and utility classes.
**Good:** Core business rules live in entity objects with methods that enforce invariants. `Order.addItem()` validates quantity, `Account.withdraw()` checks balance.
**Commit check:** Are my business rules in domain entities, or scattered across the application?

## 3. Use Cases
**Advice:** Use cases contain application-specific business rules. They orchestrate data flow to and from entities and direct entities to use their business rules. One use case = one user intention.
**Bad:**
```java
@RestController
public class OrderController {
    // Controller does validation, business logic, persistence, and notification
    @PostMapping("/orders")
    public void createOrder(@RequestBody OrderDto dto) {
        validate(dto);
        Order order = convert(dto);
        order.applyDiscount(discountService.calculate(order));
        orderRepo.save(order);
        emailService.sendConfirmation(order);
    }
}
```
**Good:**
```java
public class CreateOrderUseCase {
    public OrderId execute(CreateOrderCommand command) {
        Order order = Order.create(command.items());
        order.applyDiscount(discountPolicy.calculate(order));
        orderRepository.save(order);
        notificationPort.orderCreated(order);
        return order.getId();
    }
}
```
**Commit check:** Is my business logic in a use case/service class, or buried in a controller?

## 4. Interface Adapters
**Advice:** Controllers, presenters, and gateways convert data between the format used by use cases and the format used by external agencies (web, DB, etc.). They are the translators.
**Bad:** JPA entities used directly as API responses. Database column names leaking into JSON.
**Good:** Separate DTOs for REST, separate mapper classes, separate repository interfaces. Each adapter translates between the domain model and the external format.
**Commit check:** Am I using domain objects directly in my API responses or database queries?

## 5. Frameworks Are Details
**Advice:** Don't marry a framework. Spring, Hibernate, MyBatis — they are tools, not your architecture. Your business logic should work without them. Wrap frameworks behind interfaces.
**Bad:** Business logic annotated with `@Transactional`, `@Cacheable`, `@Scheduled` — coupled to Spring.
**Good:** Business logic in plain classes. Spring annotations only on adapter/configuration classes that wire things together.
**Commit check:** Could I replace my framework without rewriting business logic?

## 6. The Database Is a Detail
**Advice:** Your application should not know or care whether it uses PostgreSQL, MongoDB, or flat files. The database is a plugin that the architecture uses, not the center around which everything revolves.
**Bad:** SQL queries in service classes. Business logic that assumes relational structure.
**Good:** Repository interfaces in the domain layer. Implementation details (SQL, MyBatis mappers) in the infrastructure layer.
**Commit check:** If I switched databases, would I need to change business logic?

## 7. Screaming Architecture
**Advice:** Your project structure should scream what the application does, not what framework it uses. Top-level directories should be `orders/`, `customers/`, `payments/` — not `controllers/`, `services/`, `repositories/`.
**Bad:**
```
src/main/java/com/app/
  controllers/    # what framework
  services/       # what framework
  repositories/   # what framework
```
**Good:**
```
src/main/java/com/app/
  order/          # what business domain
  customer/       # what business domain
  payment/        # what business domain
```
**Commit check:** Does my package structure communicate what the system does or what framework it uses?

## 8. Humble Objects
**Advice:** Make the hard-to-test parts of your system as humble (simple) as possible. Controllers should only translate HTTP to use case calls. Views should only display data. All logic should be in testable objects.
**Bad:** Complex business logic in a REST controller that requires MockMvc to test.
**Good:** Controller does nothing but parse the request and delegate to a use case. Use case is tested with simple unit tests, no HTTP needed.
**Commit check:** Can I test my business logic without starting a web server or database?

## 9. Boundaries
**Advice:** Draw clear boundaries between components. Each boundary has an interface that the inner side defines and the outer side implements. Cross boundary only through well-defined interfaces.
**Bad:** Service A directly calls Service B's internal methods, creating hidden coupling.
**Good:** Service A depends on an interface. Service B implements that interface. Neither knows about the other's internals.
**Commit check:** Are my component boundaries clearly defined with interfaces?

## 10. Testable Architecture
**Advice:** A good architecture allows you to test business rules without the UI, database, web server, or any external element. If you need to boot Spring to test a calculation, something is wrong.
**Bad:** Every test requires `@SpringBootTest` with a full application context.
**Good:** Business logic tested with plain JUnit. Integration tests only for adapter layers.
**Commit check:** Do my tests require the full application context, or can business rules be tested in isolation?