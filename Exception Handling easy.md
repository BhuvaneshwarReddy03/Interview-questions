## What is an exception?
An exception is an abnormal event that occurs during program execution and disrupts the normal instruction flow.
In Java, an exception is represented as an object inheriting from Throwable

## What is the difference between checked and unchecked exceptions?
checked exceptions are the ones which are the direct subclasses of Exception class. And coming to unchecked exceptions they are the subclasses of runtime exception. checked exceptions are the one the general/common exceptions that may arise due to external factors like IO exceptions. And compiler will force us to handle these exceptions via try-catch block or declare via throws. And coming to runtime exceptions, These are like we get this expression at runtime due to logic flaws. These are not mandatory to be handled.

## What is the difference between throw and throws?
we use throw keyword in method or a block to trigger an exception. And coming to throws, we will add this in the method signature so that caller knows that the method they are calling might trigger the exception, so the caller will be forced to has to handle it.

## What is exception propagation?

> "**Exception propagation** is the mechanism where an unhandled exception travels backward up the **call stack** from the method where it occurred to the caller method.
> * **Stack Unwinding:** If the current method does not handle the exception with a `try-catch` block, its stack frame is discarded, and the exception is passed to its caller. This process repeats up the call hierarchy.
> * **Handling or Termination:** The propagation stops as soon as an enclosing method catches the exception. If it propagates unhandled all the way past `main()`, the JVM's **Default Exception Handler** terminates the thread and prints the stack trace.
> * **Checked vs. Unchecked:** Unchecked exceptions propagate automatically, whereas checked exceptions require each method along the propagation chain to explicitly declare them in its method signature using `throws`."
> 
>

## What is the try-catch-finally flow? Can finally be skipped?

> "The execution flow follows a strict order:
> 1. Code executes inside the **`try`** block.
> 2. If no exception occurs, all `catch` blocks are bypassed, and control jumps directly to **`finally`**.
> 3. If a matching exception occurs, control transfers to the **`catch`** block, and once handled, executes **`finally`**.
> 4. The `finally` block is guaranteed to execute even if there are `return`, `break`, or `continue` statements inside `try` or `catch`.
> 
> 
> **Can `finally` be skipped?** Yes, in four scenarios:
> * Calling **`System.exit()`**, which terminates the JVM process immediately.
> * If the OS terminates the process abruptly (e.g., `kill -9` or hardware/power failure).
> * An **infinite loop** or **deadlock** occurring inside `try` or `catch`.
> * If the thread is a **daemon thread**, and all **non-daemon (user) threads** have completed, causing the JVM to shut down without waiting."
> 
> 

---
## What is a custom exception? When to create one?
A custom exception is an application-specific class created by extending Exception or RuntimeException.

We typically build them to model business-domain events—for example, throwing a ResourceNotFoundException when a database query returns empty.

In modern architectures, this pairs directly with global exception handlers (like Spring's @ControllerAdvice and @ExceptionHandler), which intercept that specific exception type, prevent internal stack traces from leaking, and transform the failure into a structured, user-friendly response object with the proper HTTP status code.

## What is the difference between Error and Exception?

> "Both `Error` and `Exception` inherit directly from **`Throwable`**, but they serve fundamentally different purposes:
> * **`Exception`:**
> * Represents conditions caused by application logic, invalid user input, or external resource failures (like `IOException` or `NullPointerException`).
> * They are **recoverable**; applications are designed to catch and handle them gracefully using `try-catch` blocks without halting the program.
> 
> 
> * **`Error`:**
> * Represents severe, system-level issues within the JVM environment itself (like `OutOfMemoryError` or `StackOverflowError`).
> * They are **irrecoverable**; application code should never attempt to catch or handle errors, as they indicate that the JVM has run out of critical resources or is in an unstable state that necessitates process termination."
> 
> 
> 
>

## Should you catch Throwable?

> "In standard application code, **we should never catch `Throwable**`.
> * **Swallows Fatal Errors:** Because `Throwable` is the superclass of both `Exception` and `Error`, catching it intercepts severe JVM-level failures like `OutOfMemoryError` and `StackOverflowError`.
> * **Prevents Zombie Processes:** Errors indicate that JVM memory or resources are exhausted. Catching and suppressing them leaves the application in an unstable, corrupted state instead of letting it fail fast and restart cleanly.
> * **Only Valid Use Case:** It should only be caught at system boundaries—such as top-level thread pool loops or root application entry points—purely to log fatal diagnostics to monitoring systems before initiating an orderly shutdown."
> 
> 

---
