# Java Exception Handling — Complete Study Guide

## 📑 Table of Contents
1. [Step 1: The Family Tree](#step-1--the-family-tree)
2. [Step 2: Exception vs. Error](#step-2--exception-vs-error-corrected)
3. [Step 3: Checked vs. Unchecked Exceptions](#step-3--exception-splits-again-checked-vs-unchecked)
4. [Step 4: throw vs. throws](#step-4--throw-vs-throws-easy-to-mix-up-in-an-oral-exam)
5. [Step 5: try / catch — Handling It Yourself](#step-5--try--catch--handling-it-yourself)
6. [Step 6: Declaring throws — Passing the Obligation Up](#step-6--declaring-throws--passing-the-obligation-up)
7. [Step 7: finally — Always Runs, No Matter What](#step-7--finally--always-runs-no-matter-what)
8. [Step 8: Printing Exception Details](#step-8--printing-exception-details)
9. [The One-Paragraph Summary](#one-line-version-for-the-exam)
10. [Deep Questions & Answers](#deep-questions--answers)

---

## Step 1 — The Family Tree

```
                Throwable
                /       \
           Exception    Error
           /       \
     Checked    RuntimeException
   Exceptions    (Unchecked)
```

**Hierarchy Explanation:**
- **Throwable** is the root of all exception and error classes in Java. It's the only type that can be thrown or caught.
- **Exception** represents problems that a well-written application should anticipate and handle.
- **Error** represents serious problems that are typically beyond the control of the application and should not be caught.
- **Checked Exceptions** are explicit subclasses of `Exception` that don't inherit from `RuntimeException`. They must be declared in a method signature or caught.
- **RuntimeException** (Unchecked) are exceptions that occur due to programming errors and don't require explicit handling, though they can still be caught.

---

## Step 2 — Exception vs. Error (Corrected)

Both happen at runtime — that's the accurate version, correcting the compile-time confusion from your notes.

| Aspect | Exception | Error |
|--------|-----------|-------|
| **When** | Runtime | Runtime |
| **Meant to be handled?** | Yes — caught and recovered from | No — usually unrecoverable |
| **Typical cause** | Application-level issues, bad user input, bad array index, etc. | System-level issues — JVM resource exhaustion (out of memory, stack overflow) |
| **Example** | `ArrayIndexOutOfBoundsException`, `NullPointerException` | `OutOfMemoryError`, `StackOverflowError` |

### Key Distinction

**Exception** is the contract between your application and the Java runtime: "I know this problem can happen, and I have a plan to handle it."

**Error** is the JVM saying: "The system itself is failing; there's nothing your application code can do about it."

---

## Step 3 — Exception Splits Again: Checked vs. Unchecked

| Aspect | Checked | Unchecked (RuntimeException) |
|--------|---------|------------------------------|
| **Compiler forces handling?** | Yes — must `try-catch` or declare `throws` | No — handling is optional |
| **Represents** | Conditions outside your control that you should anticipate (file missing, thread interrupted) | Usually programming bugs (null reference, bad array index) |
| **Example** | `IOException`, `InterruptedException` | `NullPointerException`, `ArrayIndexOutOfBoundsException` |

### The Compiler Rule (Precise)

If a method's signature is declared with `throws SomeException`, and `SomeException` is **not** a subclass of `RuntimeException`, the compiler forces every caller to either:
1. Catch it with a `try-catch` block, OR
2. Declare `throws` themselves, passing the obligation up the call stack

### Real Java Library Example

```java
// real Java library declaration
public static void sleep(long millis) throws InterruptedException;
```

`InterruptedException` extends `Exception` directly (not `RuntimeException`) → **checked** → the compiler won't let you call `Thread.sleep(...)` without handling it one way or another.

---

## Step 4 — throw vs. throws (Easy to Mix Up in an Oral Exam — Say This Distinction Out Loud)

### `throw` — Raising an exception (Statement)
Used **inside a method body** to actually raise/create an exception right now:

```java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("Age cannot be negative");
    }
    this.age = age;
}
```

**Key points:**
- `throw` is a statement — it appears in code where you execute logic
- It immediately stops the current method and jumps to the nearest `catch` block
- Only one exception is thrown per `throw` statement
- The exception object is created with `new`, just like any other object

### `throws` — Declaring an exception (Method Signature)
Goes in a method signature, declaring "this method might raise this exception, so callers need to deal with it":

```java
void sleep(long millis) throws InterruptedException { ... }

public void readFile(String filename) throws IOException, FileNotFoundException {
    // method body
}
```

**Key points:**
- `throws` is part of the method signature — it appears in the declaration
- It tells callers: "I might throw this, so YOU need to handle it"
- Multiple exceptions can be declared: `throws IOException, FileNotFoundException`
- Only applies to **checked exceptions** (unchecked exceptions don't need declaration)
- The compiler enforces that callers handle declared checked exceptions

---

## Step 5 — try / catch — Handling It Yourself

```java
try {
    // code that MIGHT throw an exception goes here
    int[] numbers = {1, 2, 3};
    int x = numbers[10]; // throws ArrayIndexOutOfBoundsException
    System.out.println("This line never runs — the exception jumped past it");
} catch (ArrayIndexOutOfBoundsException e) {
    // runs ONLY if that specific exception type was thrown above
    System.out.println("Caught it: " + e.getMessage());
}
```

### What Actually Happens, Step by Step

1. Java runs `try` block code line by line
2. The moment a line throws an exception, Java **immediately jumps out of try** — skipping every remaining line inside it
3. Java looks for a matching `catch` (matching the exception's type)
4. If found, that `catch` block runs
5. Program continues normally after the whole `try-catch`, as if nothing crashed

### Important Details

**Exception Type Matching:**
```java
try {
    int x = Integer.parseInt("abc"); // throws NumberFormatException
} catch (NumberFormatException e) {
    System.out.println("Invalid number");
} catch (Exception e) {
    // generic catch — catches ANY Exception type
}
```

**Multiple Catches (Order Matters):**
```java
try {
    // risky code
} catch (FileNotFoundException e) {
    // catches only FileNotFoundException
    System.out.println("File not found");
} catch (IOException e) {
    // catches IOException and its subclasses
    // but NOT FileNotFoundException (already caught above)
    System.out.println("IO error");
} catch (Exception e) {
    // catches anything that extends Exception
    // MUST be last — specific exceptions first, general last
}
```

**Why order matters:** If you put `catch (Exception e)` first, it will catch *everything* (because all exceptions extend `Exception`), and your specific catches below it will never run.

---

## Step 6 — Declaring throws — Passing the Obligation Up

### The Core Concept

Instead of handling an exception **in the current method**, you can declare it in the method's signature. This passes the responsibility to the **caller** of your method.

```java
// instead of catching it here, I pass responsibility to WHOEVER calls doSomething()
void doSomething() throws InterruptedException {
    Thread.sleep(1000); // no try-catch needed — I declared throws instead
}

// now the CALLER of doSomething() must handle it
void caller() {
    try {
        doSomething();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

### Why Would You Do This?

**Scenario 1: You can't reasonably handle it locally**
```java
public void downloadFile(String url) throws IOException {
    // We're reading from a URL — if it fails, there's nothing smart we can do here
    // The CALLER knows the context (maybe they want to retry, log, show user message)
    // So we pass it up
    URLConnection connection = new URL(url).openConnection();
    connection.getInputStream();
}
```

**Scenario 2: The caller is in a better position to handle it**
```java
public void processOrder(Order order) throws PaymentException {
    // Payment failed — but we're just the processor
    // The CALLER (maybe the web controller) knows whether to retry, refund, or show error UI
    // So we throw and let them decide
    if (!payment.process(order)) {
        throw new PaymentException("Payment failed");
    }
}
```

**Scenario 3: Multiple methods in a chain**
```java
public void main(String[] args) throws IOException {
    readConfigFile(); // throws IOException
}

public void readConfigFile() throws IOException {
    readFile("config.txt"); // throws IOException
}

public void readFile(String filename) throws IOException {
    // actual file reading — throws IOException
    FileReader fr = new FileReader(filename);
}
```

Each method declares `throws IOException`, so responsibility bubbles up the call stack. Eventually, `main()` declares it too.

### The Exception Propagation Chain

Think of it like passing a hot potato:

```
main() calls readConfig()
    ↓
readConfig() gets IOException from readFile()
    ↓
readConfig() has two choices:
    a) Catch it: try-catch (the potato stops here)
    b) Rethrow it: declare throws (pass it back to main)
    ↓
If choice (b), main() gets the IOException
    ↓
main() has two choices:
    a) Catch it: try-catch (the potato stops here)
    b) Rethrow it to JVM: declare throws main (JVM crashes the program)
```

### Checked vs. Unchecked — Why It Matters for `throws`

**For Checked Exceptions:**
```java
// REQUIRED: if InterruptedException is checked, you MUST either catch or declare throws
public void sleep() throws InterruptedException {
    Thread.sleep(1000); // declares throws in signature
}
```

**For Unchecked Exceptions:**
```java
// OPTIONAL: NullPointerException is unchecked, you CAN declare throws but don't have to
public void riskyMethod() throws NullPointerException {
    String s = null;
    s.length(); // throws NPE, but compiler doesn't force it
}

// This also compiles (doesn't declare NullPointerException):
public void riskyMethod() {
    String s = null;
    s.length(); // throws NPE at runtime
}
```

**Key insight:** The compiler only *forces* you to declare checked exceptions. You *can* declare unchecked exceptions, but the compiler won't complain if you don't.

### Real-World Pattern: The Delegation Chain

```java
public class WebController {
    public void handleRequest(Request req) throws ServletException {
        try {
            processOrder(req.getOrder());
        } catch (PaymentException e) {
            // We caught it and handle it at the web layer
            req.setAttribute("error", "Payment failed");
        }
    }
}

public class OrderService {
    public void processOrder(Order order) throws PaymentException {
        // We don't handle payment errors — we throw them to the caller
        // The caller (web layer) is responsible for user-facing responses
        paymentGateway.charge(order.getAmount());
    }
}

public class PaymentGateway {
    public void charge(double amount) throws PaymentException {
        // Deep in the system, we throw
        if (amount < 0) {
            throw new PaymentException("Invalid amount");
        }
    }
}
```

In this pattern:
- **PaymentGateway** throws the exception (it's the one that detected the problem)
- **OrderService** passes it up (it can't fix payment issues)
- **WebController** catches it and converts it to a user-friendly response (the presentation layer handles it)

---

## Step 7 — finally — Always Runs, No Matter What

```java
try {
    int x = numbers[10]; // throws ArrayIndexOutOfBoundsException
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Caught: " + e.getMessage());
} finally {
    // ALWAYS executes — exception or not, even if catch itself throws, even if there's a return
    System.out.println("Cleanup runs no matter what");
}
```

### When finally Executes

`finally` runs in these scenarios:

1. **Normal execution** (no exception):
   ```java
   try {
       System.out.println("Normal code");
   } finally {
       System.out.println("Finally runs"); // ✓ runs
   }
   // Output: "Normal code" then "Finally runs"
   ```

2. **Exception caught**:
   ```java
   try {
       int x = 1 / 0;
   } catch (ArithmeticException e) {
       System.out.println("Caught");
   } finally {
       System.out.println("Finally runs"); // ✓ runs
   }
   // Output: "Caught" then "Finally runs"
   ```

3. **Exception NOT caught** (still runs before crashing):
   ```java
   try {
       int x = 1 / 0;
   } finally {
       System.out.println("Finally runs"); // ✓ runs before crash
   }
   // Output: "Finally runs" then JVM crashes
   ```

4. **Return statement in try**:
   ```java
   public String example() {
       try {
           return "from try";
       } finally {
           System.out.println("Finally runs"); // ✓ runs before return
       }
   }
   // Output: "Finally runs" then returns "from try"
   ```

5. **Return statement in catch**:
   ```java
   try {
       int x = 1 / 0;
   } catch (ArithmeticException e) {
       return "from catch";
   } finally {
       System.out.println("Finally runs"); // ✓ runs before return
   }
   // Output: "Finally runs" then returns "from catch"
   ```

6. **Exception thrown in catch**:
   ```java
   try {
       int x = 1 / 0;
   } catch (ArithmeticException e) {
       throw new RuntimeException("New error");
   } finally {
       System.out.println("Finally runs"); // ✓ runs before new exception propagates
   }
   // Output: "Finally runs" then RuntimeException propagates
   ```

### Typical Use: Resource Cleanup

Used for guaranteed cleanup — closing files, releasing connections, releasing locks — anything that must happen regardless of success or failure.

```java
public void readFile(String filename) throws IOException {
    FileReader fr = null;
    try {
        fr = new FileReader(filename);
        char[] buffer = new char[1024];
        fr.read(buffer);
        System.out.println("File read successfully");
    } catch (FileNotFoundException e) {
        System.out.println("File not found");
    } finally {
        if (fr != null) {
            fr.close(); // ALWAYS closes, whether success or exception
        }
    }
}
```

### Modern Alternative: Try-With-Resources

Java 7+ introduced a cleaner way:

```java
public void readFile(String filename) throws IOException {
    try (FileReader fr = new FileReader(filename)) {
        char[] buffer = new char[1024];
        fr.read(buffer);
        System.out.println("File read successfully");
    } catch (FileNotFoundException e) {
        System.out.println("File not found");
    }
    // fr.close() is called automatically — no finally needed
}
```

The resource (`FileReader`) automatically implements `AutoCloseable` and is closed after the try block, even if an exception occurs.

---

## Step 8 — Printing Exception Details

```java
catch (ArrayIndexOutOfBoundsException e) {
    e.printStackTrace();          // full detail: exception type + message + exact line/call chain
    System.out.println(e.getMessage()); // JUST the message text, for a custom-formatted output
}
```

### Methods for Printing Exception Information

| Method | Output | Use Case |
|--------|--------|----------|
| `e.printStackTrace()` | Full stack trace with class names, method names, and line numbers | Debugging, detailed logging |
| `e.getMessage()` | Just the error message string | Custom formatted output, user-friendly messages |
| `e.getClass().getName()` | The exception class name | Logging the exception type |
| `e.toString()` | Class name + message (shorter than stack trace) | Quick error display |

### Practical Examples

```java
// Example 1: Logging
catch (IOException e) {
    System.err.println("Error reading file: " + e.getMessage());
    e.printStackTrace(); // for debugging
}

// Example 2: User-friendly error message
catch (SQLException e) {
    System.out.println("Database error occurred. Please try again later.");
    // Don't show raw exception to user — show getMessage() to admins
    logger.error("Database error: " + e.getMessage());
}

// Example 3: Re-throwing with new exception
catch (IOException e) {
    throw new ApplicationException("Failed to load config: " + e.getMessage(), e);
}

// Example 4: All exception details
catch (Exception e) {
    String errorReport = "Exception: " + e.getClass().getName() + 
                         "\nMessage: " + e.getMessage() +
                         "\nStack trace:\n";
    e.printStackTrace(System.out);
}
```

---

## One-Line Version for the Exam

**"Throwable splits into Exception and Error, both occurring at runtime. Exceptions are recoverable, application-level problems meant to be caught; Errors are severe, usually-unrecoverable system-level failures. Exceptions further split into checked and unchecked — checked exceptions, which don't extend RuntimeException, force the compiler to require a try-catch or a throws declaration on the caller. `throw` raises an exception at a specific point in code; `throws` declares in a method's signature that it might propagate one. `try` wraps risky code, `catch` handles a specific exception type if raised, and `finally` runs regardless of whether an exception occurred, typically used for cleanup."**

---

## Deep Questions & Answers

### Q1: Why does Java distinguish between checked and unchecked exceptions?

**Answer:**

Java makes this distinction based on **predictability and responsibility**:

- **Checked Exceptions** represent conditions that are **outside your program's control but predictable**. Examples: a file might not exist, a network connection might drop, a thread might be interrupted. The compiler forces you to acknowledge these risks and plan for them.

- **Unchecked Exceptions (RuntimeException)** represent **programming bugs** — mistakes you made as the developer. Examples: accessing a null reference, wrong array index, wrong type cast. The compiler doesn't force handling because these shouldn't happen if you write correct code.

**Philosophical reason:** Checked exceptions say, "This is a real-world problem you should anticipate." Unchecked exceptions say, "This is a logic error you should prevent by writing better code."

**Exam tip:** If an interviewer asks "Why does IOException have to be caught but NullPointerException doesn't?", you now have a complete answer rooted in the design philosophy.

---

### Q2: If I declare `throws` in a method, can I still use `try-catch` inside it?

**Answer:**

Yes, absolutely. You can mix both:

```java
public void processFile(String filename) throws IOException {
    try {
        FileReader fr = new FileReader(filename); // might throw FileNotFoundException
        // do stuff
    } catch (FileNotFoundException e) {
        System.out.println("File not found, using default");
        // handle this specific case
        // IOException (parent class) still propagates up if other IO errors occur
    }
    // IOException that wasn't caught above propagates to the caller
}
```

**Common pattern:**
- Catch exceptions you can *actually recover from* locally
- Declare `throws` for exceptions you *can't* or *shouldn't* handle locally
- Let the caller decide what to do

---

### Q3: What happens if an exception is thrown in a `catch` block?

**Answer:**

The new exception **immediately stops the catch block** and propagates upward, just like any other exception:

```java
try {
    int x = 1 / 0; // throws ArithmeticException
} catch (ArithmeticException e) {
    System.out.println("Caught division by zero");
    throw new RuntimeException("Critical error!"); // NEW exception thrown here
    System.out.println("This never runs");
} finally {
    System.out.println("Finally runs"); // ✓ this DOES run
}
// RuntimeException propagates to caller
```

**Output:**
```
Caught division by zero
Finally runs
Exception in thread "main": java.lang.RuntimeException: Critical error!
```

**Key insight:** Even though the new exception interrupts the catch block, `finally` still runs before the new exception propagates.

---

### Q4: What's the difference between `throws` on a method vs. catching inside the method?

**Answer:**

| Aspect | `throws` | `try-catch` |
|--------|--------|-----------|
| **Where handled** | By the *caller* | In the *current method* |
| **Control flow** | Exception jumps to caller's catch block | Exception is handled locally, program continues |
| **Caller's code** | Must declare `throws` or have their own `try-catch` | Can call the method without special handling (if no throws) |
| **Use case** | When caller is better positioned to handle | When you can recover locally |

**Example:**

```java
// OPTION 1: Using throws
public void method1() throws IOException {
    FileReader fr = new FileReader("file.txt");
}

// Caller MUST handle it:
try {
    method1();
} catch (IOException e) {
    // ...
}

// OPTION 2: Using try-catch
public void method2() {
    try {
        FileReader fr = new FileReader("file.txt");
    } catch (IOException e) {
        System.out.println("File not found, using default");
    }
}

// Caller doesn't need to do anything:
method2(); // clean, no exception handling needed
```

**When to choose:**
- Use `throws` if the caller has context to make a better decision
- Use `try-catch` if you can fix the problem locally

---

### Q5: Can a method throw an unchecked exception without declaring it?

**Answer:**

Yes. Unchecked exceptions (anything extending `RuntimeException`) don't need to be declared in the signature:

```java
public void riskyMethod() {
    String s = null;
    s.length(); // throws NullPointerException, but no declaration needed
}

// Caller doesn't need to handle it:
riskyMethod(); // compiles fine, crashes at runtime if s is null
```

However, you *can* declare it if you want to be explicit:

```java
public void riskyMethod() throws NullPointerException {
    String s = null;
    s.length();
}
```

**But there's a catch:** The compiler won't enforce handling for unchecked exceptions. The declaration is more of a *documentation* that this method *might* throw that exception.

**Exam tip:** Remember: **Only checked exceptions are compiler-enforced.** Unchecked exceptions are optional and serve as documentation.

---

### Q6: Why does `finally` run even if there's a `return` statement?

**Answer:**

Java guarantees that `finally` runs **no matter what**, because it's meant for cleanup. If `finally` didn't run on `return`, you could leak resources:

```java
public void processAndReturn() {
    InputStream in = null;
    try {
        in = new FileInputStream("data.txt");
        return; // early exit
    } finally {
        if (in != null) {
            in.close(); // ✓ MUST run to prevent resource leak
        }
    }
}
```

**The execution order:**
1. `return` statement is encountered
2. Return value is prepared
3. `finally` block executes
4. Method actually returns

It's like Java saying: "I know you want to return, but first, cleanup."

---

### Q7: Can you catch multiple exception types in one `catch` block?

**Answer:**

Yes, Java 7+ allows multi-catch:

```java
try {
    // code that might throw IOException or SQLException
} catch (IOException | SQLException e) {
    System.out.println("IO or database error: " + e.getMessage());
    // handle both types
}
```

**Rules:**
- Exception types are separated by `|` (pipe)
- The variable `e` can call methods common to both types (methods from `Exception`)
- If one exception type is a subclass of another, you get a compile error:
  ```java
  // This is WRONG (FileNotFoundException extends IOException):
  catch (IOException | FileNotFoundException e) { ... }
  // Compile error: "Exceptions in union must not be related by subclassing"
  ```

**When to use multi-catch:**
- When multiple unrelated exception types need the same handling
- Reduces code duplication
- Makes intent clearer than catching generic `Exception`

---

### Q8: What's the difference between throwing and rethrowing an exception?

**Answer:**

**Throwing:** Creating a new exception from scratch:
```java
if (age < 0) {
    throw new IllegalArgumentException("Age cannot be negative");
}
```

**Rethrowing:** Catching an exception and throwing it again (or a wrapped version):
```java
catch (IOException e) {
    // Rethrow the same exception
    throw e;
}

catch (IOException e) {
    // Throw a new exception, but include the original as cause
    throw new ApplicationException("Failed to load file", e);
}
```

**When to rethrow:**
- You need to log or inspect the exception
- You need to wrap it in a domain-specific exception
- You need to clean up before propagating

**Example pattern:**
```java
public void loadConfig() throws ConfigException {
    try {
        FileReader fr = new FileReader("config.xml");
        // parse config
    } catch (FileNotFoundException e) {
        throw new ConfigException("Configuration file not found", e);
    } catch (IOException e) {
        throw new ConfigException("Failed to read configuration", e);
    }
}
```

---

### Q9: When should I use a checked exception vs. an unchecked exception in my own code?

**Answer:**

**Use Checked Exceptions when:**
- The condition is outside the caller's control (file doesn't exist, network fails)
- The caller can reasonably recover from it
- It's a legitimate, expected scenario (not a programming error)

```java
public class FileLoader {
    public String load(String filename) throws FileNotFoundException, IOException {
        // Caller can't prevent files not existing
        // Caller might want to try another file or use a default
    }
}
```

**Use Unchecked Exceptions when:**
- It's a programming error (bad precondition, invalid argument)
- The caller can't reasonably recover from it
- It indicates a bug in the code

```java
public class BankAccount {
    public void withdraw(double amount) {
        if (amount < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
            // Caller made a mistake; this is a bug
        }
        if (amount > balance) {
            throw new InsufficientFundsException("Not enough balance");
            // Could go either way, but usually unchecked in financial systems
        }
    }
}
```

**Rule of thumb for exams:** "Checked exceptions for external factors, unchecked for internal bugs."

---

### Q10: Can you have a try block without a catch block?

**Answer:**

Yes, if you have `finally`:

```java
InputStream in = null;
try {
    in = new FileInputStream("file.txt");
    System.out.println("File opened");
} finally {
    if (in != null) {
        in.close(); // cleanup runs, but exceptions propagate
    }
}
// If IOException occurs, it propagates to caller
```

**Also, try-with-resources (Java 7+):**

```java
try (FileReader fr = new FileReader("file.txt")) {
    // use fr
} // Resource auto-closes, even without explicit finally
// If exception occurs, it propagates to caller
```

**Why useful:**
- You need cleanup (`finally`) but can't handle the exception
- You need to auto-close a resource but let exceptions propagate
- The caller is responsible for handling

---

### Q11: What happens if `finally` throws an exception?

**Answer:**

The exception from `finally` **replaces** the original exception:

```java
try {
    throw new IOException("Original");
} finally {
    throw new RuntimeException("From finally");
}
// Only RuntimeException propagates; IOException is lost!
```

**Output:**
```
Exception in thread "main": java.lang.RuntimeException: From finally
```

The `IOException` is **suppressed** (lost). This is a pitfall!

**Better approach:** If you must throw in `finally`, handle the original exception first:

```java
try {
    throw new IOException("Original");
} finally {
    try {
        resource.close();
    } catch (IOException e) {
        // handle close exception, don't replace original
        logger.warn("Close failed", e);
    }
}
// IOException from try still propagates
```

**Modern solution (Java 7+):** Try-with-resources handles this automatically:

```java
try (Resource r = new Resource()) {
    throw new IOException("Original");
} // If close() throws, it's added as "suppressed" to the original exception
// Original IOException still propagates with close exception attached
```

---

### Q12: What's the call stack, and why does `printStackTrace()` show it?

**Answer:**

The **call stack** is the sequence of method calls that led to the exception:

```java
public static void main(String[] args) {
    method1();
}

public static void method1() {
    method2();
}

public static void method2() {
    int x = 1 / 0; // throws ArithmeticException here
}
```

**Call stack at time of exception:**
```
method2() ← exception thrown here
  ↓
method1() ← called method2()
  ↓
main() ← called method1()
```

**`printStackTrace()` output:**
```
Exception in thread "main": java.lang.ArithmeticException: / by zero
    at StackTraceExample.method2(StackTraceExample.java:14)
    at StackTraceExample.method1(StackTraceExample.java:10)
    at StackTraceExample.main(StackTraceExample.java:6)
```

**Reading it:** Start from the top (where the error occurred), then work down to see how we got there.

**Why it's useful for debugging:**
- Shows exactly which line threw the exception (`method2:14`)
- Shows the complete call chain (how we got there)
- Identifies if the problem is in your code or a library

---

This comprehensive guide covers all the nuances you'll need for your oral exam. Good luck! 🚀