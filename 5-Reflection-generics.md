# Java Oral Exam Notes — Lesson 5: Lambdas, Reflection, Generics

## Table of contents
- [Functional interfaces and lambdas (quick recap)](#functional-interfaces-and-lambdas-quick-recap)
- [Reflection](#reflection)
  - [What reflection actually is](#what-reflection-actually-is)
  - [Why reflection is needed](#why-reflection-is-needed)
  - [The four core reflection classes](#the-four-core-reflection-classes)
  - [Reflection and compiled classes (not raw files)](#reflection-and-compiled-classes-not-raw-files)
  - [The classpath](#the-classpath)
  - [Errors when things go wrong](#errors-when-things-go-wrong)
  - [Stack frames and reflection](#stack-frames-and-reflection)
- [Generics and generic classes](#generics-and-generic-classes)
  - [The problem generics solve](#the-problem-generics-solve)
  - [The Box<T> example](#the-boxt-example)
  - [How `put` and `get` work](#how-put-and-get-work)
  - [Same class definition, different specific versions](#same-class-definition-different-specific-versions)
  - [Type erasure](#type-erasure)
  - [Why `new T()` is not allowed](#why-new-t-is-not-allowed)
- [Professor questions and answers](#professor-questions-and-answers)

---

## Functional interfaces and lambdas (quick recap)

A **functional interface** has exactly one abstract method. That single method is what a **lambda expression** implements inline.

```java
@FunctionalInterface
interface Calculator {
    int compute(int a, int b); // exactly ONE abstract method
}

Calculator c = (a, b) -> a + b; // lambda — implicitly returns a + b
c.compute(3, 4); // 7
```

**Return type rule:** the lambda body must match what the interface method promises.
- `void` method → lambda just does something, no `return` with a value
- Method returning a real type (`int`, `String`, etc.) → lambda must produce that value, either with `return` inside `{ }`, or as a single expression (implicit return)

```java
interface Greeter {
    void greet(String name); // void — no return value
}
Greeter greet = (name) -> System.out.println("Hello, " + name);

interface Namer {
    String greet(String name); // returns a String
}
Namer namer = (name) -> "Hello, " + name; // must produce a String
```

**Helpful addition:**  
A lambda can only be used where Java expects a functional interface.  
That is why lambdas are often used with:
- `Runnable`
- `Comparator`
- `Consumer`
- `Supplier`
- `Function`
- `Predicate`

Example:
```java
Runnable r = () -> System.out.println("Running...");
r.run();
```

---

## Reflection

### What reflection actually is

Normally, code knows exactly what classes/methods it's using, decided at compile time:
```java
Cat myCat = new Cat("Whiskers");
myCat.eat();
```
The compiler checks, right when you compile, that `Cat` and `eat()` really exist.

**Reflection** lets a program inspect and use classes, methods, and fields **at runtime**, using their names as plain strings, instead of writing them directly into the code. It's like being handed a name on a piece of paper while the program is already running, and going to look up whatever matches that name.

**Helpful addition:**  
Reflection is often used when the program must be flexible and cannot know everything in advance. For example:
- loading classes from configuration
- testing frameworks
- annotation processing
- dependency injection
- serialization / deserialization

### Why reflection is needed

Sometimes the class or method name isn't known while writing the code — it's only decided later, while the program runs (e.g. from user input or a config file):

```java
String someClassName = getUserInput(); // e.g. "Cat" — unknown until runtime
```

You cannot write `new someClassName()` — `new` requires a literal class name the compiler can check immediately. A variable's value isn't something the compiler can resolve in advance.

Reflection is the workaround — look the class up **by name, as a string, at runtime**:

```java
Class<?> theClass = Class.forName(someClassName);       // find the class by name
Object obj = theClass.getConstructor().newInstance();    // create an object dynamically — like "new Cat()"
Method m = theClass.getMethod("eat");                     // find a method by name
m.invoke(obj);                                             // call it dynamically — like "obj.eat()"
```

**Trade-off:** reflection sacrifices compile-time safety. A typo in a class or method name (e.g. `"eatt"` instead of `"eat"`) won't be caught until the program actually runs and tries to resolve it — it would throw `NoSuchMethodException` at runtime, not a compile error.

**Helpful addition:**  
Reflection is powerful, but it is slower and more error-prone than direct method calls.  
That is why it should be used only when dynamic behavior is truly needed.

### The four core reflection classes

From `java.lang.reflect`:

| Class | Represents | What you can do with it |
|---|---|---|
| `Class` | The blueprint/structure of a class itself | Inspect its methods, fields, constructors, annotations |
| `Constructor` | One specific constructor | Create a new object dynamically |
| `Method` | One specific method | Invoke that method dynamically |
| `Field` | One specific field | Read or write its value directly — even if `private` |

**Helpful addition:**  
These are the main entry points for reflection.  
They let you inspect:
- the class itself
- how it can be constructed
- what methods it has
- what fields it contains

### Reflection and compiled classes (not raw files)

Reflection does **not** read `.java` source files or search folders like a text search. By the time reflection inspects something, the source has already been compiled into `.class` files (bytecode), and those compiled classes are what's loaded into the running program.

```java
Class<?> theClass = Class.forName("Cat");
```

This asks the JVM: "among the classes you can load, find one named `Cat`, and load it if it isn't already." It's inspecting an already-loaded compiled class's structure — not scanning file contents.

**Helpful addition:**  
A good way to remember this:
- source code = what the programmer writes
- bytecode/class files = what the JVM runs
- reflection = inspecting the running class metadata, not the source text

### The classpath

The **classpath** is the configured list of locations (folders, `.jar` files) the JVM searches when it needs to find and load a compiled class.

```
java -cp bin Main
```
`-cp bin` tells the JVM to look inside the `bin` folder for any needed `.class` files. Multiple locations can be listed at once (separated by `;` on Windows, `:` on Mac/Linux), including `.jar` files.

In the simplest case (running directly from one folder, no IDE/build tool), the classpath defaults to the current folder — so `.class` files sitting next to the `.java` files they came from will be found automatically. In real projects (IDEs, build tools), compiled output usually goes to a separate folder (e.g. `bin/`, `out/`), and the classpath is configured to point there instead.

When `Class.forName("Cat")` runs, the JVM searches every location on the classpath, in order, until it finds a match — or throws an exception if nothing matches anywhere.

**Helpful addition:**  
Classpath matters not only for reflection, but for normal class loading too.  
If the JVM cannot find a class on the classpath, the program cannot use it.

### Errors when things go wrong

- `Class.forName("SomeName")` with no matching compiled class anywhere on the classpath → throws `ClassNotFoundException` (checked exception, needs try-catch).
- `theClass.getMethod("wrongName")` with no matching method → throws `NoSuchMethodException` at runtime.

Both are only caught when the code actually runs — the compiler has no way to check a name that's just a string.

**Helpful addition:**  
That is one reason reflection code often looks like this:
```java
try {
    Class<?> c = Class.forName("Cat");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```

### Stack frames and reflection

The **call stack** is how Java keeps track of "where to return to" as methods call other methods. Each method call is *pushed* onto the stack; when it finishes, it's *popped* off, and control returns to whoever called it — last in, first out (LIFO).

```java
void methodA() { methodB(); }
void methodB() { methodC(); }
void methodC() { /* stack here: methodC, methodB, methodA, main */ }
```

Uncontrolled recursion (methods calling each other endlessly) can grow the stack until it runs out of space — this is what `StackOverflowError` means (a separate, unrelated risk from normal Java code, not specific to reflection).

Reflection can safely **inspect** the current call stack:
```java
StackTraceElement[] stack = Thread.currentThread().getStackTrace();
for (StackTraceElement frame : stack) {
    System.out.println(frame.getMethodName());
}
```

Actually **manipulating** stack frames isn't something Java's reflection API safely supports — doing so would require unsupported, low-level tricks that bypass the JVM's normal, automatic bookkeeping. Combined with reflection's general lack of compile-time checking, this could corrupt program state or cause crashes only visible at runtime — which is why it's considered unsafe and generally discouraged.

**Helpful addition:**  
The stack trace is very useful when debugging exceptions.  
It shows the path of method calls that led to the error.

---

## Generics and generic classes

### The problem generics solve

Without generics, a container class would need a separate, nearly-identical class written for every type it might hold — one for holding a String, another for an Integer, another for a Cat — even though the actual logic (put something in, get it back out) is identical every time. That's repetitive and wasteful.

Generics let you write **one** class with a placeholder standing in for "whatever type this particular instance will hold," decided later, each time the class is used.

**Helpful addition:**  
Before generics, people often used `Object` and casted manually.  
That was flexible, but unsafe because casting errors appeared only at runtime.

### The Box<T> example

`T` is not a real class — it's a placeholder name (short for "Type") for whatever type gets filled in later.

```java
class Box<T> {
    private T contents; // type is T — whatever T ends up being

    public void put(T item) {
        contents = item;
    }

    public T get() {
        return contents;
    }
}
```

Usage — `<String>` or `<Integer>` fills in the blank:
```java
Box<String> stringBox = new Box<>(); // T becomes String, for this box
stringBox.put("hello");
String s = stringBox.get(); // "hello"

Box<Integer> intBox = new Box<>(); // T becomes Integer, for this box
intBox.put(42);
```

**Helpful addition:**  
The angle brackets `<>` are the generic type arguments.  
They tell Java which concrete type should replace `T`.

### How `put` and `get` work

- `put(T item)` — takes a parameter `item` (of whatever type `T` is) and copies its value into the field `contents`.
- `get()` — returns whatever value is currently stored in `contents`.

**Helpful addition:**  
The `put` method stores data into the box.  
The `get` method retrieves data from the box.

### Same class definition, different specific versions

`Box<String>` and `Box<Integer>` come from the **same class definition** (the same source code / blueprint) — but the compiler treats them as different, incompatible specific types:

```java
Box<String> stringBox = new Box<>();
stringBox.put("hello"); // fine
stringBox.put(42);      // COMPILE ERROR — this box's type is locked to String
```

This is the same underlying idea as a reference type restricting what's allowed, seen earlier in polymorphism:
```java
Animal a = new Dog("Rex");
a.eat();   // fine — Animal declares eat()
a.bark();  // ERROR — Animal reference type doesn't expose bark(), even though it's really a Dog
```
Both are compile-time restrictions based on a **declared type**, catching mistakes before the program runs — this is generics' compile-time type-safety benefit, compared to the old pre-generics approach of storing everything as `Object` and manually casting (which only fails at runtime if the cast is wrong).

**Helpful addition:**  
So generics help you:
- avoid repeated class definitions
- keep code reusable
- keep type safety
- reduce casting

### Type erasure

Even though you write `Box<String>` and `Box<Integer>` in source code, this distinction doesn't exist anymore once compiled — Java "erases" the type information. At runtime, both are literally the same class:

```java
Box<String> stringBox = new Box<>();
Box<Integer> intBox = new Box<>();
stringBox.getClass() == intBox.getClass(); // true — same class at runtime
```

**Helpful addition:**  
This is why Java generics are mostly a compile-time feature.  
The JVM does not keep separate runtime classes for each generic type parameter.

### Why `new T()` is not allowed

```java
class Box<T> {
    public T createDefault() {
        return new T(); // COMPILE ERROR — not allowed
    }
}
```

`new` always requires a real, concrete class name that the compiler can verify exists. `T` is only ever a placeholder, replaced by *something different* depending on which specific box is being used — and because of type erasure, by the time the code runs, there's no `T` information left at all. There's nothing concrete for `new` to build.

**Helpful addition:**  
If you need to create an object of a generic type, common alternatives include:
- passing in a `Class<T>`
- passing in a factory/supplier
- using reflection carefully
- creating the object outside and storing it in the generic class

Example:
```java
class Box<T> {
    private T value;

    public Box(T value) {
        this.value = value;
    }
}
```

---

## Professor questions and answers

**Q: What is reflection in Java?**
A: A feature that lets a program inspect and use classes, methods, and fields at runtime, instead of everything being fixed at compile time.

**Q: How does normal Java code work, in contrast to reflection?**
A: If the class and method are known while writing the code (e.g. `new Cat().eat()`), the compiler checks they exist right at compile time — no reflection needed, and this works fine across different files.

**Q: Why is reflection needed at all?**
A: Sometimes a class or method name is only known at runtime (e.g. from user input or a config file), stored as a `String` — and normal syntax like `new` can't work with a name that's only available as runtime data.

**Q: Why can't you write `new someClassName()` if `someClassName` is a String variable?**
A: `new` requires a literal class name the compiler can check immediately. A variable's value might not even be decided until the program runs, so the compiler can't verify it in advance.

**Q: How do you create an object dynamically when the class name is only known at runtime?**
A: `Class<?> theClass = Class.forName(someClassName); Object obj = theClass.getConstructor().newInstance();`

**Q: How do you call a method dynamically?**
A: `Method m = theClass.getMethod("eat"); m.invoke(obj);`

**Q: How does the JVM find the class when `Class.forName(...)` runs?**
A: It searches the classpath — the configured list of folders/`.jar` files where compiled `.class` files can be found — not raw source files or arbitrary folders.

**Q: What happens if no matching class is found on the classpath?**
A: `Class.forName(...)` throws `ClassNotFoundException` at runtime.

**Q: What's the main downside of reflection?**
A: It sacrifices compile-time safety — typos or missing classes/methods are only caught when the program runs, not when it's compiled.

**Q: What are the four main classes in `java.lang.reflect`?**
A: `Class` (structure), `Constructor` (create instances), `Method` (invoke methods), `Field` (read/write field values).

**Q: What is a generic type, and why does it exist?**
A: A generic type (e.g. `Box<T>`) lets one class definition work with different types, decided when the class is used, instead of writing a separate class per type — while still keeping compile-time type checking.

**Q: Are `Box<String>` and `Box<Integer>` the same class or different classes?**
A: Same class definition/source code, but the compiler treats them as different, incompatible specific types — while at runtime, due to type erasure, they're actually the exact same class.

**Q: Why is it not possible to write `new T()` inside a generic class?**
A: `T` is just a placeholder, not a real class name — and due to type erasure, the actual type information is gone by runtime, leaving nothing concrete for `new` to construct.

**Q: What is the main benefit of generics?**
A: They provide reusable code with strong compile-time type checking.

**Q: What is the main benefit of reflection?**
A: It allows dynamic behavior when types, methods, or classes are not known until runtime.

**Q: What is the main drawback of both reflection and generics if misused?**
A: Reflection can become unsafe and error-prone at runtime, while generics can be misunderstood if people expect type information to still exist at runtime.
