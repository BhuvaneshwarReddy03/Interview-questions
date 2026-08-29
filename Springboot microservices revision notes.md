# Spring Boot & Microservices Revision Notes — Mock Interview Session

Each topic: full explanation (the "why," so you actually understand it) → **Say this** (the tightened spoken answer for the interview) → common follow-up.

---

# PART 1 — SPRING BOOT

## 1. IoC vs Dependency Injection
**Explanation:** Before IoC, a class creates its own dependencies directly (`new StripeProcessor()` inside `OrderService`) — tight coupling, since the class now controls creation and lifecycle of things it depends on, and can't easily swap implementations. **Inversion of Control (IoC)** is the principle: control over creating and wiring dependencies is inverted — moved out of your class and into an external container. **Dependency Injection (DI)** is the mechanism Spring uses to actually do that — injecting a ready-made dependency into your class (via constructor, field, or setter) instead of the class instantiating it itself. Spring's **IoC container** scans for `@Component`/`@Service`/`@Repository` beans at startup, creates them, and wires the right implementations together.

**Say this:**
> "IoC is the principle introduced to remove tight coupling with dependencies — dependent classes never create the objects they need themselves. Instead, the Spring IoC container creates those objects and manages their lifecycle. Whenever a dependent class needs an object, Spring injects it — through constructor, field, or setter injection."

**Why it matters practically:** testability (inject a mock in unit tests), swappability (change implementation via config, not code), lifecycle management (Spring manages creation, singleton by default).

---

## 2. Constructor vs Field vs Setter Injection — Why Constructor Is Recommended
**Explanation:** Four real reasons:
1. **Immutability** — constructor injection lets the dependency field be `final`; field injection can't do this (Spring sets fields via reflection after construction).
2. **Mandatory dependencies enforced at creation** — with constructor injection, the object literally cannot exist without its dependencies. Field/setter injection can leave a half-constructed object with a `null` dependency until Spring finishes wiring, or in manually-constructed test objects — leading to hard-to-trace `NullPointerException`s.
3. **Easier unit testing** — `new OrderService(mockPaymentProcessor)` works with no Spring context needed. Field injection needs Spring's test context or reflection-based mocking.
4. **Circular dependencies fail fast at startup** — if `OrderService` needs `PaymentService` and vice versa, constructor injection makes this impossible to resolve, and Spring throws `BeanCurrentlyInCreationException` immediately at startup with a clear error. Field/setter injection can silently work around this via partial object construction — the app starts fine, but the underlying design smell (two services depending on each other) stays hidden.

**Say this:**
> "Constructor injection is recommended because it lets you declare dependencies as final, guaranteeing immutability. It also enforces that mandatory dependencies exist before the object can even be constructed — avoiding partially-initialized objects and NullPointerExceptions. It makes unit testing easier since you can just call the constructor directly without needing Spring's context. And it surfaces circular dependency problems immediately at startup as a BeanCurrentlyInCreationException, rather than letting Spring quietly work around them with field injection, which just hides a real design problem."

---

## 3. Bean Scopes — Singleton vs Prototype
**Explanation:** A bean scope controls how many instances Spring creates and how long each lives. **Singleton** (default) — exactly one instance for the whole application context, shared everywhere it's injected. This is why Spring beans should generally be **stateless** — one shared instance used concurrently means any mutable instance field is shared across every thread/request using it (same thread-safety concern as any shared heap object). **Prototype** — a brand-new instance created every time the bean is requested; used when a bean genuinely needs to hold state specific to one use.

**Say this:**
> "Singleton is the default — Spring creates exactly one instance of the bean for the whole application, shared everywhere it's injected. Prototype creates a new instance every time the bean is requested. Singleton is why Spring beans should generally be stateless — any mutable field on a singleton is shared across every thread using it, so it needs the same thread-safety care as any other shared object."

**Handling request-specific state in a singleton:** use a **local variable inside the method** (lives on that thread's own stack — automatically isolated). If state needs to persist across multiple calls within the same thread without being shared between threads, use `ThreadLocal<T>` — gives each thread its own independent copy of a variable.

---

## 4. @Component vs @Service vs @Repository
**Explanation:** `@Service` and `@Repository` are specialized versions of `@Component` — Spring treats all three identically for bean-scanning and DI purposes; functionally interchangeable. The value is semantic clarity plus one piece of real extra behavior: `@Repository` makes Spring automatically translate database-specific exceptions (like `SQLException`) into Spring's own unchecked `DataAccessException` hierarchy, so the rest of the app doesn't need to handle vendor-specific checked exceptions directly.

**Say this:**
> "@Component is the generic base annotation — any Spring-managed bean. @Service and @Repository are both specialized versions of it, so functionally they're all treated the same for dependency injection. The difference is mainly semantic and layer-specific: @Service marks the business-logic layer, purely for clarity. @Repository marks the data-access layer, and it actually does something extra — Spring automatically translates database-specific exceptions like SQLException into its own unchecked DataAccessException hierarchy, so the rest of the app doesn't need to handle vendor-specific checked exceptions directly."

---

## 5. @Transactional
**Explanation:** Wraps a method's database operations in a single transaction — all commit together or all roll back together, preventing half-updated, inconsistent data. Spring implements this via a **proxy** around the bean: calling a `@Transactional` method actually calls the proxy first, which starts the transaction, delegates to the real method, then commits or rolls back based on the outcome.

```java
@Transactional
void placeOrder(Order order) {
    inventoryRepository.decrementStock(order);
    paymentRepository.chargeCard(order);
    orderRepository.save(order);
}
```

**Two critical gotchas:**
1. **Rollback only happens on unchecked exceptions by default.** A checked exception won't trigger rollback unless you specify `@Transactional(rollbackFor = SomeCheckedException.class)`.
2. **Self-invocation doesn't work.** Calling a `@Transactional` method from another method in the *same class* bypasses the Spring proxy entirely (it's a direct Java call, not going through the proxy) — the transaction logic silently doesn't apply. Fix: move the method to a separate injected bean.

**Say this:**
> "@Transactional wraps a method's database operations in a single transaction, so they either all commit or all roll back together, keeping data consistent. Spring implements this using a proxy around the bean — the proxy starts the transaction, delegates to the actual method, then commits or rolls back based on the outcome. Two things worth knowing: by default, rollback only triggers on unchecked exceptions, not checked ones — you need rollbackFor to force it for a checked exception. And self-invocation doesn't work — calling a @Transactional method from another method in the same class bypasses the Spring proxy entirely, so the transaction logic silently doesn't apply."

---

## 6. Global Exception Handling — @ControllerAdvice + @ExceptionHandler
**Explanation:** `@ControllerAdvice` marks a class as a global handler applying across every controller in the app, instead of try-catch in each controller method. Inside it, `@ExceptionHandler` methods each target one specific exception type, converting it into a clean, structured HTTP response with the right status code — instead of leaking a raw stack trace / generic 500 to the client.

```java
@ControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(InsufficientInventoryException.class)
    public ResponseEntity<String> handleInventory(InsufficientInventoryException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body(ex.getMessage());
    }

    @ExceptionHandler(InvalidOrderRequestException.class)
    public ResponseEntity<String> handleInvalidRequest(InvalidOrderRequestException ex) {
        return ResponseEntity.badRequest().body(ex.getMessage());
    }
}
```

**Say this:**
> "We create a class annotated with @ControllerAdvice, which acts as a centralized, global exception handler across every controller — instead of writing try-catch blocks in each one. Inside that class, we write separate methods annotated with @ExceptionHandler, each targeting a specific exception type — so one method handles InsufficientInventoryException, another handles InvalidOrderRequestException. Each method returns a clean, structured HTTP response with the right status code, instead of the client seeing a raw stack trace."

---

## 7. Spring Boot Auto-Configuration
**Explanation:** Lets you add a dependency (e.g. `spring-boot-starter-data-jpa`) and have Spring automatically configure the relevant beans (`DataSource`, `EntityManagerFactory`) without manual XML/config wiring. Mechanically: `@SpringBootApplication` includes `@EnableAutoConfiguration`, which at startup scans the classpath for a list of predefined `@Configuration` classes. Each is guarded by conditional annotations — most importantly `@ConditionalOnClass`, meaning a configuration only activates if a specific dependency is actually present on the classpath (e.g. `KafkaAutoConfiguration` only activates if `spring-kafka` is present). If you define your own `@Bean` of the same type, auto-configuration backs off via `@ConditionalOnMissingBean` — your explicit bean always wins.

**Say this:**
> "Spring Boot's auto-configuration scans the classpath at startup for a set of predefined configuration classes, each guarded by conditional annotations like @ConditionalOnClass — meaning a configuration only activates if the relevant dependency is actually present on the classpath. For example, if spring-kafka is on the classpath, KafkaAutoConfiguration kicks in and creates a KafkaTemplate bean automatically, using properties from application.yml. If I define my own bean of the same type explicitly, Spring Boot's auto-configuration backs off through @ConditionalOnMissingBean, so my explicit bean always takes priority."

**Follow-up:** disable a specific auto-config via `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` or `spring.autoconfigure.exclude` in properties.

---

# PART 2 — MICROSERVICES

## 8. Synchronous (REST) vs Asynchronous (Kafka) Communication
**Explanation:** **Synchronous** — simple, immediate response, but creates tight temporal coupling: if the downstream service is slow, the calling service gets slow too; if it's down, the caller must explicitly handle the failure. Can cascade across every service calling it. **Asynchronous (Kafka)** — the caller publishes an event and moves on immediately without waiting; the downstream service consumes it whenever ready. A temporary downstream outage doesn't cascade — the event just waits in Kafka. Trade-off: you give up **strong consistency** for **eventual consistency** — a brief window where the system isn't fully in sync, but guaranteed to converge.

**Say this:**
> "The two broad approaches are synchronous REST calls, or asynchronous communication through something like Kafka. With synchronous calls, the trade-off is tight coupling — if the downstream service is slow, the caller gets slow too, since it's waiting on that response. With Kafka, services are decoupled — the caller publishes an event and moves on, and the downstream service consumes it whenever it's ready, so a temporary outage doesn't cascade. The trade-off there is giving up strong consistency for eventual consistency — there's a brief window where the system isn't fully in sync, but it's guaranteed to converge once the event is processed. In my own order platform, I used the async approach with the Saga pattern specifically for this reason — accepting eventual consistency in exchange for not tightly coupling services to each other's availability."

---

## 9. Circuit Breaker Pattern
**Explanation:** When a service calls a downstream service synchronously and that downstream service is failing/slow, every request blocks a thread waiting for a response (or timeout). Under load, this exhausts the thread pool — the calling service can no longer serve *any* requests, even unrelated ones — a **cascading failure**. Circuit breaker prevents this with three states:
1. **Closed** — normal operation, requests flow through, failures are tracked.
2. **Open** — once failures cross a threshold, the circuit trips; calls stop being attempted entirely (fail fast or return a fallback), so no threads are wasted on a service known to be broken.
3. **Half-Open** — after a cooldown, a few test requests check if the service recovered. Success → close again. Failure → reopen.

```java
@CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackInventoryCheck")
public InventoryStatus checkInventory(Order order) {
    return inventoryClient.check(order);
}
```
(Resilience4j is the standard Spring Boot library for this.)

**Say this:**
> "When one service calls another synchronously and the downstream service is failing or slow, every request blocks a thread waiting for a response. Under load, that exhausts your thread pool, and your own service can no longer serve any requests — a cascading failure. A circuit breaker prevents this by tracking failures and, once they cross a threshold, tripping to an Open state — where calls to the failing service stop being attempted entirely, failing fast or falling back to a default response instead of blocking a thread. After a cooldown, it moves to Half-Open and sends a few test requests to check if the service has recovered — if they succeed, the circuit closes again; if not, it reopens. In Spring Boot, this is typically implemented with Resilience4j's @CircuitBreaker annotation with a fallback method."

---

## 10. Retry with Exponential Backoff
**Explanation:** For **transient failures** — short-lived blips where trying again will likely succeed (unlike a circuit breaker, which is for *persistent* failures). Naive immediate retry can worsen an already-overloaded downstream service (a "retry storm"). **Exponential backoff** waits progressively longer between attempts (1s, 2s, 4s, 8s...), giving the service room to recover. **Jitter** — adding a small random offset to each wait — prevents many client instances from all retrying at the exact same synchronized moments ("thundering herd").

**Say this:**
> "Retry with backoff handles transient failures — short-lived blips where trying again will likely succeed. Instead of retrying immediately, which can pile more load onto an already-struggling service, you wait progressively longer between each attempt — exponential backoff, like 1 second, then 2, then 4. You'd also typically add jitter, a small random offset, so that many client instances retrying at once don't all hit the downstream service at exactly the same synchronized moments. Retry and circuit breaker usually work together — retry absorbs short blips, and if failures persist beyond that, the circuit breaker trips open and stops calling the service altogether."

---

## 11. Saga Pattern
**Explanation:** In a single database, multiple operations can be wrapped in one ACID transaction — automatic atomic rollback if anything fails. In microservices, each service (Order, Payment, Inventory) has its **own separate database** — there's no single transaction spanning all three. If Inventory decrements stock and Payment then fails, there's no automatic cross-service rollback. **Saga** solves this by breaking the operation into a sequence of local transactions, each in its own service, with an explicit **compensating transaction** to undo each step if a later step fails.

```
Step 1: Inventory decrements stock (commits locally)
Step 2: Payment charges the card → FAILS
Compensating step: Inventory increments stock back (explicitly undoes step 1)
```
You must write the compensating action yourself — the database has no idea a different service's step failed.

**Choreography vs Orchestration:**
- **Choreography** — no central coordinator; each service reacts to events and publishes its own events in turn. Fully decentralized.
- **Orchestration** — a central orchestrator explicitly tells each service what to do and calls compensating actions on failure.

**Say this:**
> "In a single database, you'd wrap multiple operations in one transaction, and the database guarantees atomic rollback if anything fails. But in microservices, each service has its own separate database — there's no single transaction that can span all of them. So if Inventory decrements stock and then Payment fails, there's no automatic rollback across services. The Saga pattern solves this by breaking the operation into a sequence of local transactions, each in its own service, with an explicit compensating transaction to undo each step if a later step fails — for example, if payment fails after inventory was already decremented, a compensating action restores the stock. I used Kafka events to drive this in my order platform — each service reacts to events and, if needed, triggers its own compensating action."

---

## Delivery reminders for tomorrow
- Don't conflate "async decoupling" with "Saga pattern" — Saga is specifically about compensating transactions across separate databases, not just about using Kafka.
- Land answers on a clear closing statement — several answers tonight trailed off mid-sentence.
- Have the standard vocabulary ready: Closed/Open/Half-Open for circuit breakers, Choreography/Orchestration for Saga — examples showed you understand the mechanism before you had the exact terms, so anchor the terms to what you already know.
- Structure every answer: definition → why it matters / what breaks without it → concrete example from your own project.
