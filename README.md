# Java Oral Exam Notes

## Table of Contents
- [1. Classes, Objects, and Data Types](#1-classes-objects-and-data-types)
- [2. Loops in Java](#2-loops-in-java)
- [3. Constructors](#3-constructors)
- [4. Iterable and Iterator](#4-iterable-and-iterator)
- [5. Encapsulation](#5-encapsulation)
- [6. Static vs Non-Static Methods](#6-static-vs-non-static-methods)
- [7. Inheritance, Overriding, and Hiding](#7-inheritance-overriding-and-hiding)
- [8. Oral Exam Questions and Answers](#8-oral-exam-questions-and-answers)

---

## 1. Classes, Objects, and Data Types

### Class
A **class** is a blueprint or template used to create objects.

Example:

```java
class Car {
    String brand;
    int speed;
}
```

### Object
An **object** is an instance of a class, meaning a real thing created from the blueprint.

Example:

```java
Car c1 = new Car();
c1.brand = "Toyota";
c1.speed = 120;
```

### Data types
A **data type** tells Java what kind of value a variable can store.

Examples:
- `int` for integers
- `double` for decimal numbers
- `boolean` for true/false
- `char` for single characters
- `String` for text

---

## 2. Loops in Java

Java has three main loop types.

### `for` loop
Used when you know how many times the loop should repeat.

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

### `while` loop
Used when the number of repetitions is not known in advance.

```java
int i = 1;
while (i <= 5) {
    System.out.println(i);
    i++;
}
```

### `do-while` loop
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

## 3. Constructors

A **constructor** is a special method used to initialize objects.

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

## 4. Iterable and Iterator

### Iterable
`Iterable` is an interface that means an object can be iterated over.

If a class implements `Iterable`, it can be used in a for-each loop.

### Iterator
An **Iterator** is an object used to traverse a collection one element at a time.

It keeps track of the current position in the collection, so it must be an object and not just a single method.

### What an Iterator does
- checks whether another element exists
- returns the next element
- remembers where it left off

### Main methods
- `hasNext()` → checks if there is another element
- `next()` → returns the next element
- `remove()` → removes the last returned element, in some cases

### Example of iteration

```java
Iterator<Integer> it = list.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

### Why Iterator is an object
It needs to remember state, such as the current position in the collection, across multiple calls.

### Easy analogy
An iterator is like a bookmark that remembers where you stopped reading.

---

## 5. Encapsulation

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

## 6. Static vs Non-Static Methods

### Static methods
- Belong to the class
- Can be called without creating an object
- Access static members directly

Example:

```java
class MathUtil {
    static int add(int a, int b) {
        return a + b;
    }
}
```

Usage:

```java
int sum = MathUtil.add(3, 4);
```

### Non-static methods
- Belong to an object
- Need an instance to be called
- Can access both static and non-static members

Example:

```java
class Dog {
    void bark() {
        System.out.println("Woof");
    }
}
```

Usage:

```java
Dog d = new Dog();
d.bark();
```

---

## 7. Inheritance, Overriding, and Hiding

### Inheritance
Inheritance means that a subclass gets the properties and behavior of a superclass.

If `B` extends `A`, then `B` inherits from `A`.

```java
class A {
    int name = 10;
}

class B extends A {
}
```

### Object creation and assignment

#### Not possible
```java
B x = new A();
```
This is not allowed because an object of type `A` cannot be assigned to a variable of type `B`.

#### Possible
```java
A x = new B();
```
This is allowed because a subclass object can be stored in a superclass reference.

---

### Overriding methods
A method is overridden when a subclass defines a method with the same name and same parameters as the superclass method, and provides its own implementation.

Example:

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

Example:

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

The attribute from `A` is accessed, because attributes are resolved by the reference type, not dynamically like methods.

---

### Methods are dynamic, attributes are static
- Methods are chosen at runtime based on the actual object
- Attributes are chosen based on the reference type

Example:

```java
A x = new B();
x.m();
System.out.println(x.name);
```

If `m()` is overridden in `B`, then `x.m()` calls `B`'s version.
But `x.name` uses the `name` defined in `A`.

---

## 8. Oral Exam Questions and Answers

### 1. What is a class?
A class is a blueprint used to create objects.

### 2. What is an object?
An object is an instance of a class.

### 3. What is an iterator?
An iterator is an object that lets us traverse a collection one element at a time.

### 4. Why is an iterator an object?
Because it must store state, such as the current position in the collection, across multiple method calls.

### 5. What does `hasNext()` do?
It checks whether there is another element in the collection.

### 6. What does `next()` do?
It returns the next element and moves the iterator forward.

### 7. What is encapsulation?
Encapsulation is the idea of keeping data and methods together while hiding internal details.

### 8. What is a constructor?
A constructor is a special method that initializes an object.

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

### 16. What is the role of `Iterable`?
`Iterable` allows an object to be used in a for-each loop by providing an iterator.

---

## Short memorization version

- **Class**: blueprint
- **Object**: instance of a class
- **Constructor**: initializes object
- **Iterator**: object that traverses a collection
- **Iterable**: can provide an iterator
- **Encapsulation**: hide data, expose methods
- **Static method**: belongs to class
- **Non-static method**: belongs to object
- **Overriding**: subclass replaces superclass method
- **Attributes**: hidden, not overridden
- **`A x = new B()`**: valid
- **`B x = new A()`**: invalid
