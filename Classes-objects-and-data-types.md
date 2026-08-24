# 1. Classes, objects, and data types

## Table of Contents
- [1. Classes, Objects, and Data Types](#1-classes-objects-and-data-types)
  - [a) For loop types (3)](#a-for-loop-types-3)
  - [b) Constructors](#b-constructors)
  - [c) Iterable Interface](#c-iterable-interface)
  - [d) Encapsulation](#d-encapsulation)
  - [e) Static, non-static methods](#e-static-non-static-methods)
  - [f) Inheritance, overriding, and hiding](#f-inheritance-overriding-and-hiding)
- [2. Oral Exam Questions and Answers](#2-oral-exam-questions-and-answers)

---

## 1. Classes, Objects, and Data Types

### Class
A **class** is a blueprint or template used to create objects.

### Object
An **object** is an instance of a class. It is a real thing created from the class blueprint.

### Data types
A **data type** tells Java what kind of value a variable can store.

Examples:
- `int` for integers
- `double` for decimal numbers
- `boolean` for true/false values
- `char` for single characters
- `String` for text

Example:

```java
class Car {
    String brand;
    int speed;
}

Car c1 = new Car();
c1.brand = "Toyota";
c1.speed = 120;
```

---

### a) For loop types (3)

Java has three main loop types.

#### 1. `for` loop
Used when you know how many times the loop should repeat.

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

#### 2. `while` loop
Used when the number of repetitions is not known in advance.

```java
int i = 1;
while (i <= 5) {
    System.out.println(i);
    i++;
}
```

#### 3. `do-while` loop
Like `while`, but it always runs at least once.

```java
int i = 1;
do {
    System.out.println(i);
    i++;
} while (i <= 5);
```

#### Quick comparison

| Loop type | Runs at least once? | Main use |
|---|---:|---|
| `for` | No | known number of repetitions |
| `while` | No | condition-controlled repetition |
| `do-while` | Yes | execute once before checking condition |

---

### b) Constructors

A **constructor** is a special method used to initialize objects.

#### Main rules
- Same name as the class
- No return type, not even `void`
- Called automatically when using `new`

#### Example

```java
class Student {
    String name;

    Student() {
        name = "Unknown";
    }
}

Student s = new Student();
System.out.println(s.name);
```

#### Why constructors matter
Constructors give objects their initial values when they are created.

#### Types of constructors

##### Default constructor
If you do not write any constructor, Java provides one automatically.

##### Parameterized constructor
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

### c) Iterable Interface

`Iterable` is an interface that means an object can be iterated over.

If a class implements `Iterable`, its objects can be used in a for-each loop.

#### Main idea
- `Iterable` says: “my objects can be traversed”
- `Iterator` is the object that actually moves through the elements

#### Main method
- `iterator()` → returns an `Iterator`

#### Why `Iterable` matters
The enhanced for loop uses `Iterable` behind the scenes.

#### Iterator
An **Iterator** is an object used to traverse a collection one element at a time.

It must be an object because it keeps track of state, such as the current position in the collection, across multiple method calls.

#### Main methods of Iterator
- `hasNext()` → checks if there is another element
- `next()` → returns the next element
- `remove()` → removes the last returned element, in some cases

#### Example idea

```java
Iterator<Integer> it = list.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

#### Why Iterator is an object
An iterator needs memory of where it stopped. It is like a bookmark that remembers your position in a book.

#### What it does in practice
- moves element by element
- avoids direct access to collection internals
- works with the for-each loop through `Iterable`

---

### d) Encapsulation

Encapsulation means keeping data and methods together in one class while hiding internal details.

It protects data by using access modifiers such as `private` and `public`.

#### Example

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

#### Why it is useful
- protects data
- improves security
- makes code easier to maintain
- allows validation before changing values
- prevents direct unsafe access

#### Key idea
Instead of letting outside code change the data directly, you control it through methods.

---

### e) Static, non-static methods

#### Static methods
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

Usage:

```java
int sum = MathUtil.add(3, 4);
```

#### Non-static methods
- belong to an object
- need an instance to be called
- can access both static and non-static members

```java
class Dog {
    void bark() {
        System.out.println("Woof");
    }
}

Dog d = new Dog();
d.bark();
```

#### Important distinction
- `static` = class-level behavior
- non-static = object-level behavior

---

### f) Inheritance, overriding, and hiding

#### Inheritance
Inheritance means a subclass gets properties and behavior from a superclass.

If `B` extends `A`, then `B` inherits from `A`.

```java
class A {
    int name = 10;
    void m() {
        System.out.println("A");
    }
}

class B extends A {
    int name = 20;
    @Override
    void m() {
        System.out.println("B");
    }
}
```

#### `B x = new A();`
This is not valid.
A parent object cannot be stored in a child reference.

#### `A x = new B();`
This is valid.
A child object can be stored in a parent reference.

#### Overriding methods
A method is overridden when a subclass defines a method with the same name and same parameters as the superclass method, and provides its own implementation.

#### Can attributes be overridden?
No. Attributes are not overridden like methods. They are hidden or shadowed.

#### Methods are dynamic, attributes are static
- Methods are chosen at runtime based on the actual object
- Attributes are chosen based on the reference type

Example:

```java
A x = new B();
x.m();
System.out.println(x.name);
```

#### Explanation
- `x.m()` calls `B`'s method because methods are dynamically dispatched
- `x.name` accesses the field from `A` because fields depend on the reference type

#### Important exam idea
The object is really of type `B`, but the reference is of type `A`.
That is why method calls and field access behave differently.

---

## 2. Oral Exam Questions and Answers

### 1. What is a class?
A class is a blueprint used to create objects.

### 2. What is an object?
An object is an instance of a class.

### 3. What is a constructor?
A constructor is a special method that initializes an object.

### 4. What is the difference between a constructor and a normal method?
A constructor has the same name as the class, has no return type, and is called automatically when an object is created.

### 5. What is an iterator?
An iterator is an object used to traverse a collection one element at a time.

### 6. Why is an iterator an object?
Because it must keep state, such as the current position, across multiple calls like `hasNext()` and `next()`.

### 7. What does `hasNext()` do?
It checks whether there is another element in the collection.

### 8. What does `next()` do?
It returns the next element and moves the iterator forward.

### 9. What is the difference between `Iterable` and `Iterator`?
`Iterable` is the interface that allows iteration, while `Iterator` is the object that performs the traversal.

### 10. Why does `Iterable` matter for the for-each loop?
Because the for-each loop works on objects that provide an iterator.

### 11. What is encapsulation?
Encapsulation is the principle of keeping data and methods together while hiding internal details.

### 12. Why do we use encapsulation?
To protect data, control access, and improve code maintenance and safety.

### 13. What is the difference between static and non-static methods?
Static methods belong to the class, while non-static methods belong to an object.

### 14. Can a static method access instance variables directly?
No, it can directly access only static members.

### 15. What is inheritance?
Inheritance is when a subclass receives the properties and behavior of a superclass.

### 16. How do you override a method?
The subclass must define a method with the same name and parameters as the superclass method.

### 17. Can attributes be overridden?
No. Attributes are hidden or shadowed, not overridden.

### 18. What happens in `A x = new B();`?
A reference of type `A` points to an object of type `B`.

### 19. What happens in `x.m()` when `m()` is overridden in `B`?
The version in `B` is called, because methods are dynamic.

### 20. What happens in `System.out.println(x.name)` when both `A` and `B` define `name`?
The field from `A` is used, because fields are resolved by the reference type.

### 21. Is `B x = new A();` valid if `B` extends `A`?
No, it is not valid.

### 22. Is `A x = new B();` valid if `B` extends `A`?
Yes, it is valid.

### 23. What is the main idea behind polymorphism in this case?
The reference type is `A`, but the actual object is `B`, so method calls are decided at runtime.

### 24. Why can a variable with the same name in a subclass hide the parent field?
Because fields do not override; the subclass field simply shadows the inherited one.

---

## Short memorization version

- **Class**: blueprint
- **Object**: instance of a class
- **Data type**: kind of value stored in a variable
- **Constructor**: initializes object
- **for loop**: known repetitions
- **while loop**: unknown repetitions, tested before running
- **do-while loop**: runs at least once
- **Iterable**: can give an iterator
- **Iterator**: object that traverses a collection one by one
- **Encapsulation**: hide data, expose methods
- **Static method**: belongs to class
- **Non-static method**: belongs to object
- **Overriding**: subclass replaces superclass method
- **Attributes**: not overridden, they are hidden
- **`A x = new B()`**: valid
- **`B x = new A()`**: invalid
