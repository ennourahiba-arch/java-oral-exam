# 1. Classes, objects, and data types

## a) For loop types (3)

Java has three main loop types:

### 1. `for` loop
Used when you know how many times the loop should run.

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

### 2. `while` loop
Used when the number of iterations is not known in advance.

```java
int i = 1;
while (i <= 5) {
    System.out.println(i);
    i++;
}
```

### 3. `do-while` loop
Like `while`, but it always runs at least once.

```java
int i = 1;
do {
    System.out.println(i);
    i++;
} while (i <= 5);
```

### Quick comparison

| Loop type | Runs at least once? | Main use |
|---|---:|---|
| `for` | No | known number of repetitions |
| `while` | No | condition-controlled repetition |
| `do-while` | Yes | execute once before checking condition |

---

## b) Constructors

A constructor is a special method used to initialize objects.

### Main rules
- Same name as the class
- No return type
- Called automatically when using `new`

### Example

```java
class Student {
    String name;

    Student() {
        name = "Unknown";
    }
}
```

```java
Student s = new Student();
System.out.println(s.name);
```

### Types of constructors

#### Default constructor
If you do not write any constructor, Java provides one automatically.

#### Parameterized constructor
A constructor that receives values.

```java
class Student {
    String name;
    int age;

    Student(String n, int a) {
        name = n;
        age = a;
    }
}
```

---

## c) Iterable Interface

`Iterable` is an interface that means an object can be iterated over.

If a class implements `Iterable`, it can be used in a for-each loop.

### Main idea
- `Iterable` says: “my objects can be traversed”
- `Iterator` is the object that actually moves through the elements

### Main method
- `iterator()` → returns an `Iterator`

### What an Iterator does
An iterator is an object that moves through a collection one element at a time.

It keeps track of the current position in the collection, so it must be an object and not just a method.

### Main methods of Iterator
- `hasNext()` → checks if there is another element
- `next()` → returns the next element
- `remove()` → removes the last returned element, in some cases

### Example use

```java
Iterator<Integer> it = list.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

### Why Iterator is an object
It needs to remember state, such as the current position in the collection, across multiple calls.

### Analogy
An iterator is like a bookmark that remembers where you stopped reading.

---

## d) Encapsulation

Encapsulation means keeping data and methods together in one class while hiding internal details.

It protects the data by using access modifiers such as `private` and `public`.

### Example

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

### Why it is useful
- protects data
- improves security
- makes code easier to maintain
- allows validation before changing values

---

## e) Static, non-static methods

### Static methods
- belong to the class
- can be called without creating an object
- can access static members directly

```java
class MathUtil {
    static int add(int a, int b) {
        return a + b;
    }
}
```

### Non-static methods
- belong to an object
- need an instance to be called
- can access both static and non-static members

```java
class Dog {
    void bark() {
        System.out.println("Woof");
    }
}
```

---

## f) Inheritance, overriding, and hiding

### Inheritance
Inheritance means a subclass gets properties and behavior from a superclass.

If `B` extends `A`, then `B` inherits from `A`.

```java
class A {
    int name = 10;
}

class B extends A {
}
```

### `B x = new A();`
This is not valid.
A parent object cannot be stored in a child reference.

### `A x = new B();`
This is valid.
A child object can be stored in a parent reference.

---

### Overriding methods
A method is overridden when a subclass defines a method with the same name and same parameters as the superclass method, and provides its own implementation.

```java
class A {
    void m() {
        System.out.println("A");
    }
}

class B extends A {
    @Override
    void m() {
        System.out.println("B");
    }
}
```

---

### Attributes cannot be overridden
Attributes are not overridden like methods. They can be hidden or shadowed.

```java
class A {
    String name = "A";
}

class B extends A {
    String name = "B";
}
```

If you have:

```java
A x = new B();
System.out.println(x.name);
```

The attribute from `A` is accessed, because attributes are resolved by the reference type.

### Methods are dynamic, attributes are static
- Methods are chosen at runtime based on the actual object
- Attributes are chosen based on the reference type

```java
A x = new B();
x.m();
System.out.println(x.name);
```

If `m()` is overridden in `B`, then `x.m()` calls `B`'s version.
But `x.name` uses the `name` defined in `A`.

---

## g) Oral exam questions and answers

### 1. What is a class?
A class is a blueprint used to create objects.

### 2. What is an object?
An object is an instance of a class.

### 3. What is a constructor?
A constructor is a special method that initializes an object.

### 4. What is an iterator?
An iterator is an object that lets us traverse a collection one element at a time.

### 5. Why is an iterator an object?
Because it must store state, such as the current position in the collection, across multiple method calls.

### 6. What does `hasNext()` do?
It checks whether there is another element in the collection.

### 7. What does `next()` do?
It returns the next element and moves the iterator forward.

### 8. What is encapsulation?
Encapsulation is the idea of keeping data and methods together while hiding internal details.

### 9. What is the difference between `for`, `while`, and `do-while` loops?
`for` is used when the number of repetitions is known, `while` checks the condition before running, and `do-while` runs at least once.

### 10. What is the difference between static and non-static methods?
Static methods belong to the class, while non-static methods belong to an object.

### 11. Can methods be overridden?
Yes, instance methods can be overridden in a subclass.

### 12. Can attributes be overridden?
No, attributes are hidden or shadowed, not overridden.

### 13. Is `B x = new A();` valid if `B` extends `A`?
No, it is not valid.

### 14. Is `A x = new B();` valid if `B` extends `A`?
Yes, it is valid.

### 15. What is the difference between methods and attributes in inheritance?
Methods are dynamically dispatched, while attributes depend on the reference type.
