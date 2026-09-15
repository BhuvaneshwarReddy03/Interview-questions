## What happens if an exception is thrown inside a finally block?

> "If an exception is thrown inside a `finally` block:
> * **Immediate Propagation:** Execution aborts, skipping any subsequent code and propagating the exception directly to the caller.
> * **Overwrites Return Values:** Any pending `return` statement executed in the `try` or `catch` block is canceled.
> * **Swallows the Root Cause:** If the `try` or `catch` block had already thrown an exception, the exception from `finally` **overwrites it entirely**. The original error is lost, which is why cleanup in `finally` must be guarded or replaced with `try-with-resources`."
> 
> 

---

## What happens if you put a return statement inside a finally block?

> "Placing a `return` statement inside a `finally` block is a severe anti-pattern in Java:
> * **Overrides Return Values:** It unconditionally overwrites any previous `return` value executed inside the `try` or `catch` blocks.
> * **Swallows Exceptions Silently:** If an exception was thrown in the `try` or `catch` block, a `return` in `finally` **completely discards and swallows the exception**, allowing the method to exit normally and masking critical errors.
> * **Best Practice:** A `finally` block should be used exclusively for cleanup operations (like closing streams or resetting state), never for control-flow statements like `return` or `break`."
> 
>

## Multi-catch, rethrowing with precise types

> "**Multi-Catch and Precise Rethrow** were both introduced in Java 7 to clean up exception handling:
> * **Multi-Catch:** Allows combining multiple exceptions into a single catch block using the pipe operator (`catch (IOException | SQLException e)`). The types must be **disjoint** (no parent-child relationship), and the parameter `e` is implicitly `final`.
> * **Precise Rethrow:** When you catch a general `Exception` purely to execute cross-cutting logic (like logging) and rethrow it (`throw e`), the compiler performs flow analysis. As long as `e` is not reassigned, the method signature can declare the **exact checked exceptions** actually thrown inside the `try` block (`throws IOException, SQLException`) instead of forcing a leaky `throws Exception` declaration."
> 
> 

---
## Exception wrapping / translation pattern for microservices

> "The **Exception Translation pattern** in microservices solves the problem of boundary leakage and tight coupling across distributed systems:
> * **Boundary Translation:** We never let raw transport errors (like `FeignException`, SQL, or WebClient errors) leak into the business logic. Downstream decoders intercept them and wrap them into clean domain exceptions using **exception chaining** to preserve root diagnostics.
> * **Separation of Faults:** We classify translated errors into **transient/retryable** (timeouts, 503s) and **permanent** (400, 404), allowing Resilience4j or circuit breakers to act intelligently.
> * **Sanitized Public Contracts:** A centralized edge handler (`@RestControllerAdvice`) maps internal domain exceptions into a standardized API contract—typically **RFC 7807 Problem Details**—enriching the response with the distributed `traceId` while keeping internal system traces secure."
> 
>
