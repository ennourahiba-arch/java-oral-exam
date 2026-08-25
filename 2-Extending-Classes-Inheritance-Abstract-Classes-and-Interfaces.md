# 2. Extending Classes, Inheritance, Abstract Classes, and Interfaces

## Table of Contents
- [a) Superclass, subclass, and constructors](#a-superclass-subclass-and-constructors)
- [b) `super(...)` and parent constructors](#b-super-and-parent-constructors)
- [c) `private` vs `protected`](#c-private-vs-protected)
- [d) Method overloading and overriding](#d-method-overloading-and-overriding)
- [e) Abstract classes](#e-abstract-classes)
- [f) Interfaces](#f-interfaces)
- [g) Default methods in interfaces](#g-default-methods-in-interfaces)
- [h) Functional interfaces and lambdas](#h-functional-interfaces-and-lambdas)
- [i) Anonymous classes](#i-anonymous-classes)
- [j) Local classes](#j-local-classes)
- [k) Outer objects](#k-outer-objects)
- [l) Interface default-method collisions](#l-interface-default-method-collisions)
- [m) Polymorphism](#m-polymorphism)

---

## a) Superclass, subclass, and constructors

In Java, when a class **extends** another class, it becomes a **subclass** of that parent class.

The subclass automatically inherits all the **accessible** fields and methods of the parent class. That means the child class can use them without rewriting them, and it can also add new behavior of its own.

### Example

```java
class Animal {
    protected String name;

    Animal(String name) {
        this.name = name;
    }

    void eat() {
        System.out.println(name + " is eating.");
    }
}

class Dog extends Animal {
    Dog(String name) {
        super(name);
    }
}
```

Here:
- `Animal` is the **superclass**
- `Dog` is the **subclass**
- `Dog` inherits the accessible parts of `Animal`

### Important idea
A subclass does not rewrite inherited members automatically. It simply gets access to the ones that are visible to it.

### Oral exam questions and answers

**Q: What is a superclass?**  
A: A superclass is the parent class that another class extends.

**Q: What is a subclass?**  
A: A subclass is a child class that extends another class and inherits its accessible members.

**Q: Does a subclass inherit everything from its parent class?**  
A: No. It inherits only the accessible members, not private ones.

---

## b) `super(...)` and parent constructors

The keyword `super` refers to the **immediate parent class**.

`super(...)` is used to call the parent constructor, so the parent part of the object can be initialized correctly.

### Important rules
- `super(...)` must be the **first statement** in the subclass constructor
- only **one** constructor call is allowed there
- you can pass arguments to it, for example `super(name, age);`
- if you do not write it, Java automatically inserts `super()`
- if the parent class does not have a no-argument constructor, the code will not compile

### Why this rule exists
The parent part of the object must be initialized before the child part.
If the parent constructor has not run yet, inherited fields may still have their default values, such as:
- `null`
- `0`
- `false`

### Important correction
`super(name)` does **not** move the field from the parent to the child.  
It simply passes the value `name` to the parent constructor, so the parent class can initialize the field it owns.

### Example

```java
class Cat {
    String name;

    Cat(String name) {
        this.name = name;
    }
}

class Kitten extends Cat {
    Kitten(String name) {
        super(name);
    }
}
```

In this case:
- `Kitten` does not have its own `name` field unless it declares one
- it inherits the `name` field from `Cat`
- `super(name)` calls `Cat(String name)` so `Cat` can initialize `this.name`

### Oral exam questions and answers

**Q: What does `super(...)` do?**  
A: It calls the parent class constructor.

**Q: Why must `super(...)` be the first statement in a constructor?**  
A: Because the parent part of the object must be initialized before the child part.

**Q: What happens if the parent class has no no-argument constructor and you do not call `super(...)` explicitly?**  
A: The code does not compile.

**Q: Does `super(name)` create a field in the child class?**  
A: No. It only passes the value to the parent constructor so the parent can initialize its own field.

---

## c) `private` vs `protected`

- `private` fields are accessible only inside the same class
- `protected` fields are accessible inside the same class, in subclasses, and in the same package

### Oral exam questions and answers

**Q: What is the difference between `private` and `protected`?**  
A: `private` is visible only inside the class, while `protected` is visible inside the class, in subclasses, and in the same package.

**Q: Why would you use `protected`?**  
A: To allow subclasses to access a field or method while still restricting access from outside the package.

---

## d) Method overloading and overriding

These two concepts are different.

### Overloading
Overloading means having multiple methods with the **same name** in the same class, but with **different parameters**.

The difference can be:
- number of parameters
- type of parameters
- order of parameters

### Example

```java
class MathUtil {
    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Here, both methods are called `add`, but they are different because their parameter lists are different.

### Overriding
Overriding happens when a subclass gives its own version of a method that already exists in the superclass.

For overriding, the method must have:
- the same name
- the same parameters
- a compatible return type

### Example

```java
class Animal {
    void eat() {
        System.out.println("Animal eats.");
    }
}

class Dog extends Animal {
    @Override
    void eat() {
        System.out.println("Dog eats quickly.");
    }
}
```

### Role of `@Override`
`@Override` is optional, but very useful. It tells the compiler that you intend to override a parent method.

If the method signature does not match, the compiler will give an error.

### Main difference
- **Overloading** = same name, different parameters
- **Overriding** = same name, same parameters, different class, new implementation

### Oral exam questions and answers

**Q: What is method overloading?**  
A: Method overloading is when methods in the same class have the same name but different parameter lists.

**Q: What is method overriding?**  
A: Method overriding is when a subclass provides its own version of a parent method with the same name and parameters.

**Q: What is the difference between overloading and overriding?**  
A: Overloading changes the parameter list, while overriding keeps the same signature but changes the implementation in a subclass.

**Q: Is `@Override` required?**  
A: No, but it helps the compiler check that a method really overrides another one.

---

## e) Abstract classes

An abstract class is a **partial blueprint**.

It cannot be instantiated directly, but it can still contain:
- fields
- constructors
- normal methods
- abstract methods

### Abstract methods
An abstract method has no body. It only declares that a method must exist in the subclasses.

### Example

```java
abstract class Animal {
    String name;

    Animal(String name) {
        this.name = name;
    }

    abstract void eat();

    void sleep() {
        System.out.println(name + " is sleeping.");
    }
}
```

Here:
- `Animal` is abstract
- `eat()` is abstract, so subclasses must define it
- `sleep()` is a normal method with a body

### Important rules
- you cannot do `new Animal(...)`
- a concrete subclass must implement every abstract method it inherits
- if it does not, then that subclass must also be abstract

### Why abstract classes are useful
They let you define shared behavior in one place while forcing subclasses to complete the missing parts.

### Oral exam questions and answers

**Q: What is an abstract class?**  
A: An abstract class is a partial blueprint that cannot be instantiated directly.

**Q: Can an abstract class have a constructor?**  
A: Yes, it can.

**Q: Can an abstract class have normal methods?**  
A: Yes, it can have both normal methods and abstract methods.

**Q: Can you create an object from an abstract class?**  
A: No, you cannot use `new` with an abstract class.

**Q: What happens if a concrete subclass does not implement an inherited abstract method?**  
A: It will not compile unless the subclass is also declared abstract.

---

## f) Interfaces

An interface is a **contract** or a **checklist**.

It tells implementing classes:
> “You must provide these methods.”

By default, interface methods are abstract, so they usually have no body.

### Example

```java
interface Chewable {
    void chew();
}
```

Any class that implements `Chewable` must define `chew()`.

### Multiple inheritance with interfaces
A class can:
- extend only **one** class
- implement **many** interfaces

### Example

```java
interface Flyer {
    void fly();
}

interface Swimmer {
    void swim();
}

class Duck implements Flyer, Swimmer {
    public void fly() {
        System.out.println("Duck flies.");
    }

    public void swim() {
        System.out.println("Duck swims.");
    }
}
```

### Why this is allowed
Java does not allow multiple class inheritance because it can create ambiguity and conflicts.

Interfaces are safer because they traditionally describe behavior without providing conflicting full implementations.

### Oral exam questions and answers

**Q: What is an interface?**  
A: An interface is a contract that specifies methods a class must implement.

**Q: Can a class extend more than one class in Java?**  
A: No, a class can extend only one class.

**Q: Can a class implement more than one interface?**  
A: Yes, a class can implement many interfaces.

**Q: Why does Java not allow multiple class inheritance?**  
A: Because it could create ambiguity and method conflicts.

---

## g) Default methods in interfaces

Since Java 8, interfaces can contain methods with a body.

These are called **default methods**.

### Example

```java
interface Printer {
    default void print() {
        System.out.println("Printing...");
    }
}

class Document implements Printer {
}
```

In this case, `Document` automatically inherits `print()` even if it does not write it itself.

### Why default methods exist
They were added so that interfaces could evolve without breaking old code.

If Java adds a new method to a widely used interface, existing classes would normally fail to compile because they would be missing that method. Default methods solve that problem by providing a built-in implementation.

### Oral exam questions and answers

**Q: What is a default method in an interface?**  
A: It is a method in an interface that has a body and can be inherited by implementing classes.

**Q: Why were default methods added?**  
A: To allow interfaces to gain new methods without breaking existing code.

---

## h) Functional interfaces and lambdas

A **functional interface** is an interface with exactly **one abstract method**.

It may also contain:
- default methods
- static methods

### Example

```java
@FunctionalInterface
interface Calculator {
    int compute(int a, int b);
}
```

Because there is only one required method, Java allows you to use a **lambda expression** instead of writing a full class.

### Example

```java
Calculator c = (a, b) -> a + b;
```

This is a short way to provide the implementation of the single abstract method.

### Why the “one method” rule matters
If there were two abstract methods, Java would not know which one the lambda should implement.

### Oral exam questions and answers

**Q: What is a functional interface?**  
A: A functional interface is an interface with exactly one abstract method.

**Q: Can a functional interface have default methods?**  
A: Yes, it can.

**Q: Why are functional interfaces useful?**  
A: They are useful because they allow the use of lambdas and method references.

**Q: Why must a functional interface have only one abstract method?**  
A: Because a lambda can implement only one required method.

---

## i) Anonymous classes

An anonymous class is a class that is declared and instantiated at the same time, without a name.

It is useful when you need a class only once and do not want to create a separate named class.

There are several common ways to create anonymous classes:
- anonymous inner class
- anonymous subclass
- anonymous interface implementation

### Example idea

```java
Runnable r = new Runnable() {
    public void run() {
        System.out.println("Running");
    }
};
```

This creates an unnamed class that implements `Runnable`.

### Oral exam questions and answers

**Q: What is an anonymous class?**  
A: It is a class without a name that is declared and instantiated at the same time.

**Q: Why do we use anonymous classes?**  
A: We use them when we need a one-time class implementation.

---

## j) Local classes

A local class is a class declared inside a block, such as:
- a method
- a constructor
- another block

A local class is visible only inside that block.

It is useful when the class is only needed locally and does not belong elsewhere in the program.

### Example idea

```java
void test() {
    class Local {
        void message() {
            System.out.println("Inside local class");
        }
    }
}
```

### Oral exam questions and answers

**Q: What is a local class?**  
A: A local class is a class declared inside a block such as a method or constructor.

**Q: Where can a local class be used?**  
A: Only inside the block where it is declared.

---

## k) Outer objects

An outer object is the instance of the enclosing class when you use an inner class.

An inner class has access to the members of the outer class object.

### Example idea

```java
class Outer {
    int x = 10;

    class Inner {
        void show() {
            System.out.println(x);
        }
    }
}
```

Here, `Inner` can access `x` because it is connected to an outer object.

### Oral exam questions and answers

**Q: What is an outer object?**  
A: It is the instance of the enclosing class that an inner class belongs to.

**Q: Why can an inner class access outer class members?**  
A: Because it has a reference to the outer object.

---

## l) Interface default-method collisions

If a class implements two interfaces that both define a default method with the same signature, there is a conflict.

In that case, the class must override the method and provide its own implementation.

### Why
Java needs the class to decide which behavior should be used.

### Oral exam questions and answers

**Q: What happens if two interfaces define the same default method?**  
A: The implementing class gets a conflict and must override the method.

**Q: Why must the class override it?**  
A: To resolve the ambiguity and choose the behavior explicitly.

---

## m) Polymorphism

Polymorphism means **one interface, many forms**.

It means the same method call can behave differently depending on the actual object.

### Example idea

```java
Animal a = new Dog();
a.eat();
```

Even though the reference type is `Animal`, the actual object is `Dog`, so Java uses the `Dog` version of `eat()`.

### Key idea
- the **reference type** tells Java what it is allowed to call
- the **actual object type** tells Java which overridden method to execute

### Oral exam questions and answers

**Q: What is polymorphism?**  
A: Polymorphism means one reference type can work with many different object types.

**Q: Why does `Animal a = new Dog(); a.eat();` call the Dog version of `eat()`?**  
A: Because the actual object is a `Dog`, and overridden methods are chosen at runtime.

**Q: What is the difference between reference type and actual object type?**  
A: The reference type is the type of the variable, while the actual object type is the class of the object created with `new`.

---

## Short memorization version

- **Superclass**: parent class
- **Subclass**: child class
- **`super(...)`**: calls the parent constructor
- **`private`**: visible only in the same class
- **`protected`**: visible in subclasses and same package
- **Overloading**: same name, different parameters
- **Overriding**: same method signature, new implementation in subclass
- **Abstract class**: partial blueprint, cannot be instantiated
- **Interface**: contract of required methods
- **Default method**: interface method with a body
- **Functional interface**: interface with exactly one abstract method
- **Anonymous class**: unnamed class created once
- **Local class**: class declared inside a block
- **Outer object**: instance of the enclosing class
- **Polymorphism**: one reference, many possible behaviors
