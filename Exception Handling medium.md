## When should you use checked exceptions vs unchecked exceptions?

> "It comes down to **who is at fault** and **whether the program can recover**:
> * **Use Unchecked Exceptions (like `IllegalArgumentException` or `IllegalStateException`):**
> When the failure is caused by a **developer mistake, logic bug, or bad input**. The caller broke the method contract. You do not force them to catch it; they need to fix the bug in their code.
> * **Use Checked Exceptions (like `IOException` or `SQLException`):**
> When the failure is caused by an **external, unavoidable condition** (like a missing file or a network drop) that a correct program still encounters. You use a checked exception so the compiler forces the caller to provide an alternate plan or fallback."
> 
>

## What is try-with-resources? What problem does AutoCloseable solve?

> "**`try-with-resources`** is a Java 7 feature that automates resource management, replacing boilerplate-heavy and error-prone `finally` blocks.
> * **Mechanism:** Any object declared in the `try(...)` statement must implement the **`AutoCloseable`** interface, which defines the `close()` method that the JVM automatically invokes upon exit.
> * **Handles Multiple Resources:** Resources are automatically closed in **reverse order of their creation**.
> * **Prevents Exception Masking:** If both the business logic and the `.close()` method fail, Java preserves the business failure as the primary exception and attaches the closing error via **suppressed exceptions** (`e.getSuppressed()`), ensuring root causes are never lost in logs."
> 
> 

## What is exception chaining?

> "**Exception chaining** is the practice of catching a lower-level exception (e.g., `SQLException`) and wrapping it inside a higher-level domain exception (e.g., `UserServiceException`) while passing the original exception into the constructor as the `cause`.
> * **Preserves Context:** It maintains the full causal stack trace in the logs via `Caused by:`, ensuring the original root cause and line numbers are never lost.
> * **Programmatic Access:** Upstream callers and centralized exception handlers can inspect the underlying problem programmatically at runtime using `e.getCause()`."
> 
> 

---

## What is the best way to handle exceptions in layered applications?

> "In a layered architecture:
> * **Lower/Data Layers:** Catch infrastructure-specific exceptions (like `SQLException`), wrap them into meaningful domain exceptions, and **preserve the root cause using exception chaining**.
> * **Service Layers:** Enforce business logic and throw unchecked custom exceptions (like `ResourceNotFoundException`).
> * **Controller Layers:** Remain clean with **zero `try-catch` blocks**, allowing exceptions to propagate freely to the application boundary.
> * **Global Exception Handler:** Intercepts these exceptions centrally. It has two responsibilities:
> 1. **Observability:** It logs the full chained stack trace internally for monitoring and debugging.
> 2. **Sanitization:** It maps the exception to an appropriate HTTP status code and returns a clean, structured error payload to the client without leaking internal system details."
> 
> 
> 
> 

---
## Can you have an empty catch block? Why is it bad practice?

> "Syntactically, Java allows empty catch blocks, but in production code it is a severe anti-pattern known as **exception swallowing**:
> * **Lost Diagnostics:** It destroys the stack trace and root cause, blinding logging and monitoring systems to runtime failures.
> * **State Corruption:** It allows the thread to continue running as if the operation succeeded, causing downstream operations to work on inconsistent or corrupt state.
> * **The Core Rule:** If you catch an exception, you must either:
> 1. Provide an alternate recovery or fallback path.
> 2. Wrap and rethrow it using exception chaining.
> 3. Log the full stack trace at the system boundary.
> 
> 
> 
> 


> *If an exception must truly be ignored in a rare edge case, best practice requires naming the exception variable `ignored` and adding an explicit comment explaining why.*
---

You have full command of `try-with-resources`. What question should we drill next?
