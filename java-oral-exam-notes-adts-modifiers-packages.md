# Java Oral Exam Notes — Lesson: ADTs, Access Modifiers, Packages, Scoping

## Table of contents
- [Abstract Data Types: Lists](#abstract-data-types-lists)
- [Access modifiers](#access-modifiers)
- [Non-access modifiers](#non-access-modifiers)
  - [final](#final)
  - [static](#static)
  - [transient](#transient)
- [Package structure](#package-structure)
- [Scoping rules](#scoping-rules)
- [Professor questions and answers](#professor-questions-and-answers)

---

## Abstract Data Types: Lists

An **ADT (Abstract Data Type)** is defined by *what it does* (its operations/behavior), not *how* it's implemented internally — this connects directly to abstraction: `List` is the interface (the "what"), and each concrete class is a different "how."

```
        List (interface)
       /    |      \
ArrayList LinkedList Vector, Stack, CopyOnWriteArrayList...
```

**ArrayList** — backed by a resizable array. Fast `get(index)` (direct array access). Slower to insert/remove in the middle (elements have to shift).

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.get(0); // fast, direct index lookup
```

**LinkedList** — backed by a chain of nodes, each pointing to the next. Fast to insert/remove at the start or middle (just relink pointers). Slower `get(index)` (must walk the chain from the start each time).

**Vector** — like an older `ArrayList`, but every method is `synchronized` — thread-safe, but slower due to locking overhead even when only one thread is involved.

**Stack** — LIFO (Last-In-First-Out), same principle as the call stack. In Java, `Stack` is a class that extends `Vector`, adding `push()`, `pop()`, `peek()`.

```java
Stack<Integer> stack = new Stack<>();
stack.push(1);
stack.push(2);
stack.pop(); // removes and returns 2
```

**CopyOnWriteArrayList** — thread-safe, optimized for many reads and rare writes. Every modification (add/remove) creates a whole new copy of the underlying array internally. Writes are expensive, but reads are fast and need no locking, since a reading thread always sees a complete, unchanging snapshot.

**Core exam point:** code can be written entirely against the `List` interface, and the actual implementation can be swapped (`ArrayList` ↔ `LinkedList` ↔ `Vector`) without changing the code that uses it — the same reference-type-vs-actual-object pattern as polymorphism, applied to collections.

```java
List<String> names = new ArrayList<>(); // reference type List, actual object ArrayList
```

**Choosing between them:** lots of `get(index)` calls, few middle insertions → `ArrayList`. Lots of insertions/removals in the middle → `LinkedList`.

---

## Access modifiers

Four levels of access, from most to least restrictive:

| Modifier | Same class | Same package | Subclass (different package) | Everywhere |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| *(default, no keyword)* | Yes | Yes | No | No |
| `protected` | Yes | Yes | **Yes** | No |
| `public` | Yes | Yes | Yes | Yes |

**Default (package-private):** no modifier at all → visible only within the same package, full stop, no exceptions (not even for subclasses in other packages).

```java
class Cat { // package-private
    String name; // package-private
}
```

**`protected` — the key nuance:** broader than default. It grants access to the same class, the same package, **and** subclasses anywhere, even in a completely different package. Inheritance itself is what unlocks the access across the package boundary.

```java
// package zoo;
public class Animal {
    protected String name;
}

// package farm; (different package)
import zoo.Animal;
public class Dog extends Animal {
    void show() {
        System.out.println(name); // WORKS — Dog is a subclass, protected reaches across packages
    }
}

// package farm; same package as Dog, but NOT a subclass
class Groomer {
    void groom(Animal a) {
        System.out.println(a.name); // ERROR — not a subclass, and not in Animal's package
    }
}
```

The distinguishing rule: default = same package only, no exceptions. `protected` = same package, plus subclasses anywhere regardless of package.

---

## Non-access modifiers

### final

Means "cannot be changed again" — meaning depends on what it's applied to:

```java
final int MAX_SIZE = 100;
MAX_SIZE = 200; // COMPILE ERROR — final variable can't be reassigned

final class Cat { }
class Kitten extends Cat { } // COMPILE ERROR — final class can't be extended

class Animal {
    final void eat() { }
}
class Dog extends Animal {
    @Override
    void eat() { } // COMPILE ERROR — final method can't be overridden
}
```

**Why mark a method `final`:** to protect trusted/critical behavior from being altered or broken by subclassing — similar in spirit to how `private` protects data from uncontrolled modification. Example: a `calculateInterest()` method using a carefully-tested formula — marking it `final` guarantees no subclass can silently swap in broken or malicious logic.

### static

Means "belongs to the class itself, not to any individual object" — there is only ever ONE shared copy, no matter how many objects exist.

```java
class Counter {
    static int count = 0; // ONE shared copy, belongs to the class

    Counter() {
        count++;
    }
}

Counter c1 = new Counter(); // count = 1
Counter c2 = new Counter(); // count = 2 — same shared variable
Counter c3 = new Counter(); // count = 3

System.out.println(Counter.count); // 3 — accessed via the class name
```

Contrast with a normal (instance) field: `c1.name` and `c2.name` are genuinely separate, independent copies. `static` fields are shared by every object of that class.

### transient

Relevant to **serialization** (converting an object to a byte stream, e.g. to save to a file or send over a network). A `transient` field is skipped during that process — its value is not saved.

```java
class User implements Serializable {
    String username;           // saved when serialized
    transient String password; // NOT saved — skipped
}
```

Used for sensitive data (passwords) or data that doesn't make sense to persist (live connections, caches). When deserialized, a `transient` field comes back as its default value (`null`, `0`, etc.), since it was never actually saved.

*(`abstract`, `synchronized`, `volatile` — already covered in earlier lessons: abstract classes/methods, thread synchronization, and memory visibility, respectively.)*

---

## Package structure

A package groups related classes together, similar to folders organizing files. A package name directly corresponds to a folder structure on disk.

```java
// File: zoo/Animal.java
package zoo;
public class Animal { }
```

```
src/
└── zoo/
    ├── Animal.java        (package zoo;)
    └── mammals/
        └── Dog.java        (package zoo.mammals;)
```

**Using a class from another package requires `import`:**

```java
// File: farm/Dog.java
package farm;
import zoo.Animal; // required to use a class from a different package

public class Dog extends Animal { }
```

Without the `import`, `extends Animal` wouldn't compile — Java has no way to know which `Animal` is meant.

**Connection to access modifiers:** package-private (default) access exists as a middle ground between `private` (only this exact class) and `public` (everyone) — letting classes grouped in the same package freely access each other's internals while hiding those internals from unrelated outside code.

---

## Scoping rules

Scope = where in the code a name (variable, field, etc.) is actually visible and usable. A variable's scope is the block `{ }` it's declared inside — nothing outside that block can see it.

```java
void test() {
    int x = 10; // x's scope starts here, extends to the end of test()

    if (true) {
        int y = 20; // y's scope is ONLY inside this if-block
        System.out.println(x); // fine — x is visible here (outer scope reaches into inner block)
    }

    System.out.println(y); // COMPILE ERROR — y is out of scope here
}
```

**Key rule:** inner blocks can see variables from outer blocks that contain them, but not the reverse — an outer block can never see variables declared inside a nested inner block.

**Field scope vs. local variable scope:**

```java
class Cat {
    String name; // FIELD — scope is the entire class, visible in every method

    void greet() {
        String greeting = "Hi"; // LOCAL VARIABLE — scope is only inside greet()
        System.out.println(greeting + " " + name);
    }

    void purr() {
        System.out.println(greeting); // COMPILE ERROR — greeting's scope ended when greet() ended
    }
}
```

A `for` loop's header variable (`for (int i = 0; ...)`) is scoped to the loop's block — referencing `i` after the loop's closing `}` is a compile error.

---

## Professor questions and answers

**Q: What is an ADT, and how does `List` relate to it?**
A: An ADT is defined by its behavior/operations, not its implementation. `List` is the interface (the "what"); `ArrayList`, `LinkedList`, `Vector`, etc. are different concrete implementations (the "how").

**Q: When would you choose ArrayList over LinkedList?**
A: When the program does a lot of `get(index)` lookups and few middle insertions/removals — `ArrayList` gives fast direct index access, while `LinkedList` would need to walk its chain of nodes from the start each time.

**Q: What's the difference between Vector and ArrayList?**
A: They work similarly, but `Vector`'s methods are all `synchronized` (thread-safe, with locking overhead), while `ArrayList` is not synchronized and generally faster in single-threaded use.

**Q: What is Stack, and how does it relate to Vector?**
A: `Stack` is a LIFO (Last-In-First-Out) structure; in Java, `Stack` is a class that extends `Vector`, adding `push()`, `pop()`, `peek()`.

**Q: What is CopyOnWriteArrayList optimized for?**
A: Many reads, rare writes. Every write creates a full copy of the underlying array; reads are fast and need no locking since they always see a stable snapshot.

**Q: What are the four access modifier levels, from most to least restrictive?**
A: `private` (same class only) → default/package-private (same package only) → `protected` (same package, plus subclasses anywhere) → `public` (everywhere).

**Q: What's the key difference between default (package-private) and protected access?**
A: Default access never extends beyond the same package, with no exceptions. `protected` also reaches subclasses in different packages — inheritance itself unlocks that cross-package access.

**Q: If a class in the same package as `Animal`, but not a subclass of it, tries to access a `protected` field — does it work?**
A: Yes — `protected` grants access to the same package regardless of inheritance, in addition to subclasses anywhere.

**Q: What does `final` mean when applied to a variable, a class, and a method?**
A: Variable — cannot be reassigned after its initial value. Class — cannot be extended/subclassed. Method — cannot be overridden by a subclass.

**Q: Why would a class designer mark a method `final`?**
A: To protect trusted or critical behavior from being altered or broken by an incorrect override in a subclass — similar in spirit to how `private` protects data.

**Q: What does `static` mean on a field?**
A: The field belongs to the class itself, not to any individual object — there is exactly one shared copy, referenced by all instances.

**Q: What does `transient` do?**
A: Marks a field to be excluded from serialization; its value isn't saved when the object is converted to a byte stream, and it returns as its default value when deserialized.

**Q: How does a package relate to folder structure?**
A: A package name corresponds directly to a folder path on disk — `package zoo.mammals;` means the file must live inside a `zoo/mammals/` folder.

**Q: Why is `import` needed to use a class from another package?**
A: Without it, Java has no way to resolve which class is meant, since a class with the same name could theoretically exist in multiple packages.

**Q: What determines a variable's scope?**
A: The block `{ }` it's declared inside — it's visible from its declaration to the end of that block, and invisible outside it. Inner blocks can see outer variables, but not the reverse.
