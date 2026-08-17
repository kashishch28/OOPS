# Java OOP — Interview Prep Notes
*Compiled from Kunal Kushwaha's DSA-Bootcamp-Java (`lectures/17-oop`) + extra core-concept sections (Stack/Heap, Encapsulation, Polymorphism, Instance vs Reference variables, Static Typing) written in simple language for quick revision.*

---

## Table of Contents
0. [Java Is a Statically Typed Language](#0-java-is-a-statically-typed-language)
1. [Classes & Objects](#1-classes--objects)
2. [Stack vs Heap — Where Objects & References Live](#2-stack-vs-heap--where-objects--references-live)
3. [Instance Variable vs Reference Variable](#3-instance-variable-vs-reference-variable)
4. [`this`, `final`, and `finalize()`](#4-this-final-and-finalize)
5. [Constructors](#5-constructors)
6. [Access Control](#6-access-control)
7. [Encapsulation](#7-encapsulation)
8. [`static`](#8-static)
9. [Inheritance](#9-inheritance)
10. [Polymorphism](#10-polymorphism)
11. [Overloading vs Overriding](#11-overloading-vs-overriding)
12. [Abstraction — `abstract` classes vs `interface`](#12-abstraction--abstract-classes-vs-interface)
13. [Packages](#13-packages)
14. [Enums](#14-enums)
15. [The Four Pillars of OOP — Summary](#15-the-four-pillars-of-oop--summary)
16. [Singleton Pattern](#16-singleton-pattern)
17. [Rapid-Fire Interview Q&A](#17-rapid-fire-interview-qa)

---

## 0. Java Is a Statically Typed Language

In simple words: **Java checks the type of every variable at compile time, before the program even runs.**

```java
int age = 20;
String name = "Kashish";

age = "Kashish";   // ❌ Compile-time error — age was declared as int, forever
```

Once you declare a variable's type, it can never change to a different type. Compare this to a **dynamically typed** language like Python, where the same variable name can hold an `int` and then a `String` later:

```python
x = 10
x = "Hello"   # ✅ totally fine in Python
```

> 💡 **Why it matters for interviews:** Static typing catches a whole class of bugs *before* the program runs (at compile time), rather than crashing mid-execution. It's also what makes Java's method overloading (see §11) possible — the compiler can tell which overload to call just by looking at the argument types.

---

## 1. Classes & Objects

- A **class** is a template/blueprint for an object; an **object** is an instance of a class. A class is a logical construct — an object has physical reality (it occupies memory).
- Every object has three properties: **state** (data), **identity** (its unique memory location), **behavior** (what its methods do).
- The `new` keyword dynamically allocates memory at runtime and returns a **reference** to the object — not an actual pointer (you can't do pointer arithmetic on it, which is part of Java's safety model).
- The `.` (dot) is technically called a **separator** in the Java spec, even though everyone calls it "the dot operator." It links an object's name to its member (field or method).

```java
Box mybox;          // declares a reference — doesn't point to an object yet
mybox = new Box();  // allocates the object, mybox now holds its address
```

- **Reference assignment copies the reference, not the object.**

```java
Box b1 = new Box();
Box b2 = b1;   // b1 and b2 point to the SAME object — no copy made
```

- Primitives (`int`, `char`, etc.) are **not** objects in Java — they're implemented as plain variables for efficiency, so `new` is never used for them.
- **Parameter vs Argument**: a parameter is the variable in the method signature (`int i` in `square(int i)`); an argument is the actual value passed at the call site (`square(100)`).
- `Bus bus = new Bus();` → the **left side (reference type)** is checked by the **compiler**, the **right side (object creation)** is handled by the **JVM**.

> 💡 **Interview angle:** "Is Java pass-by-value or pass-by-reference?" → Java is **always pass-by-value**. For objects, the *value of the reference* (i.e., the address) is passed, not the object itself — which is why mutating an object inside a method affects the caller, but reassigning the reference doesn't.

### Breaking down `Person p = new Person();` word by word

A handy one-liner to have ready if an interviewer asks you to explain object creation:

| Part | Meaning |
|---|---|
| `Person` | The **class/type** of the reference — tells Java what kind of object `p` can point to |
| `p` | The **reference variable** — a name that will hold the object's address |
| `=` | **Assignment** — stores the value on the right into the variable on the left |
| `new` | Tells Java to **create a new object**, allocated on the heap |
| `Person()` | Calls the **constructor**, with `()` meaning no arguments passed |
| `;` | Ends the statement |

Read as: *"Create a new `Person` object, and store its reference in the reference variable `p`."*

---

## 2. Stack vs Heap — Where Objects & References Live

This is a very common "explain how Java manages memory" interview question.

### Stack

The **Stack** stores method execution information and **local variables** — including local reference variables.

### Heap

The **Heap** is the area of memory where **objects are generally allocated**.

```java
Person p = new Person();
```

Here:
- `p` → a reference variable, living in the current stack frame
- `new Person()` → creates the actual object
- the object itself → generally stored in the heap

```
Stack                         Heap

p  ----------------------->  Person object
                              name = "Kashish"
                              age  = 20
```

### Reference Assignment — copies the reference, not the object

```java
Person p1 = new Person();
Person p2 = p1;
```

There is only **one** object here — both `p1` and `p2` point to it:

```
Stack                         Heap

p1  ----------------------\
                            → Person object
p2  ----------------------/
```

So if `p2.name = "John";`, then `p1.name` will **also** print `"John"`, because both references point to the same object.

### Two Different Objects

```java
Person p1 = new Person();
Person p2 = new Person();
```

This creates **two separate objects**. Changing one has zero effect on the other:

```
Stack                         Heap

p1  ----------------------->  Person object 1
p2  ----------------------->  Person object 2
```

### `null` Reference & Garbage Collection

```java
Person p = new Person();
p = null;
```

Now `p` no longer refers to the object. If an object is no longer reachable by **any** reference, it becomes eligible for **Garbage Collection**.

> 💡 **Interview note:** The stack/heap picture above is a very useful mental model and is what interviewers expect. But the real JVM memory model can be more nuanced (e.g., escape analysis can let the JIT allocate some objects on the stack) — it's fine to lead with this model, just don't claim it's an absolute, unbreakable rule of the JVM spec.

---

## 3. Instance Variable vs Reference Variable

These two terms sound similar but mean different things, and interviewers like to test whether you actually know the difference.

### Instance Variable

An **instance variable** is a variable declared **inside a class but outside any method, constructor, or block**. Each object gets **its own copy** of every instance variable.

```java
class Student {
    String name;   // instance variable
    int age;       // instance variable
}
```

```java
Student s1 = new Student();
Student s2 = new Student();

s1.name = "Kashish";
s2.name = "Rahul";
```

Each object has its own separate copy: `s1` has `name = "Kashish"`, `s2` has `name = "Rahul"`.

### Reference Variable

A **reference variable** stores a *reference* to an object — it is not the object itself.

```java
Student s = new Student();
```

Here `s` is the reference variable; `name` and `age` **inside** the `Student` object are the instance variables.

```
Student s = new Student();
s
↑
Reference variable

Student object
├── name
└── age
    ↑
    Instance variables
```

> 💡 **Interview tip:** "A reference variable *refers to* an object; an instance variable *represents part of that object's state*."

---

## 4. `this`, `final`, and `finalize()`

### `this`
Refers to the current object — the object on which the currently executing method was invoked. Commonly used to disambiguate instance variables from parameters with the same name.

### `final`
- On a variable → makes it a constant; must be initialized when declared. Convention: name it in `ALL_CAPS`, e.g. `final int FILE_OPEN = 2;`
- **Gotcha:** `final` on a reference-type variable only freezes the *reference* (it will always point to the same object) — the internal state of that object can still change. `final` does **not** make objects immutable.
- On a method → prevents overriding (see §11).
- On a class → prevents inheritance (implicitly makes all its methods `final` too). A class **cannot** be both `abstract` and `final`.

### `finalize()`
- Called by the JVM right before an object is garbage collected — used to release non-memory resources.
- **Interview note:** `finalize()` is deprecated since Java 9 and removed in later versions in favor of `try-with-resources` / `Cleaner`. Good to mention you know this if it comes up, even though the source material predates it.

---

## 5. Constructors

- Automatically invoked when an object is created via `new`, **before `new` finishes executing**.
- No return type — not even `void` — because the implicit return type is the class itself.
- If you don't define one, Java supplies a **default no-arg constructor**.

### Constructors + Inheritance
- The base class's no-arg constructor is called **automatically** before the derived class constructor body runs (constructors run **top-down**, base first).
- If the base class has **only a parameterized constructor** (no default one), the derived class **must** explicitly call it via `super(args)` — otherwise it's a compile error.

```java
class Base {
    Base() { System.out.println("Base Constructor"); }
}
class Derived extends Base {
    Derived() { System.out.println("Derived Constructor"); }
}
// new Derived() prints:
// Base Constructor
// Derived Constructor
```

> 💡 **Why constructors run base-first:** the superclass has no knowledge of the subclass, so any initialization it needs is a prerequisite for the subclass's own initialization.

### Default Constructor vs No-Argument Constructor — a subtle distinction

These two terms get used interchangeably, but technically they're not identical:

- **Default constructor** → the no-argument constructor that the **compiler automatically generates** *only if you don't declare any constructor yourself*.
- **No-argument constructor** → *any* constructor that takes zero arguments — whether you wrote it yourself or the compiler generated it.

```java
class Student {
    String name;
    // no constructor written by you
}
// The compiler conceptually generates:
// Student() { super(); }   ← this IS the "default constructor"
```

**Important rule:** the moment you write *any* constructor yourself (even a parameterized one), Java **stops** auto-generating the default constructor.

```java
class Student {
    Student(String name) { this.name = name; }
    String name;
}

Student s = new Student();   // ❌ compile-time error — no no-arg constructor exists anymore
```

To fix it, you must add a no-arg constructor explicitly:

```java
class Student {
    String name;
    Student() { }                          // explicitly written no-arg constructor
    Student(String name) { this.name = name; }
}

Student s1 = new Student();          // ✅ works
Student s2 = new Student("Kashish"); // ✅ works
```

> 💡 **Interview tip:** if asked "what is a default constructor?", mention this distinction — a hand-written no-arg constructor is *not* technically "the default constructor," even though it behaves the same way.

---

## 6. Access Control

How a member can be accessed is determined by the **access modifier** attached to its declaration. Java's access modifiers are `public`, `private`, `protected`, and **default** (no keyword at all).

| Modifier | Class | Package | Subclass (same pkg) | Subclass (diff pkg) | World |
|---|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ✅ | ❌ |
| *default* (no modifier) | ✅ | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ | ❌ |

*(Most restrictive → least restrictive, left to right: `private` → default → `protected` → `public`)*

> 📌 The same rules apply to **inner classes** too — an inner class is treated as just another member of its outer class, so its own access modifier follows this exact table.

- `protected` only becomes meaningful in the context of inheritance.
- **Classic gotcha (asked a lot):** in a subclass in a *different* package, you can access a `protected` member only **through a reference of the subclass type (or further subclass)** — not through a reference of the superclass type.

```java
// package packageTwo
public class Derived extends packageOne.Base {
    public void show() {
        new Base().display();     // ❌ NOT allowed
        new Derived().display();  // ✅ allowed
        display();                 // ✅ allowed (inherited)
    }
}
```
Reasoning: `Derived` only has the right to touch `protected` state it inherited itself — not `protected` state of an arbitrary `Base` instance (which might belong to some *other* subclass `Derived` knows nothing about). If you generalize this: for `obj.member` to be legal, `obj`'s (static) type must be the subclass itself (or a further subclass) — never just the superclass, even if the calling code happens to live inside a subclass.

---

## 7. Encapsulation

**Encapsulation** means bundling data (fields) and the methods that operate on that data inside a single class, while **controlling direct access** to that data from outside.

The most common way to achieve it in Java:
1. Make fields `private`.
2. Provide controlled access through public methods — usually **getters** and **setters**.

```java
class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age >= 0) {          // validation!
            this.age = age;
        }
    }
}
```

Now the fields can't be touched directly from outside the class:

```java
Student s = new Student();
// s.age = -5;    ❌ not allowed — age is private
s.setAge(20);     // ✅ controlled access, and invalid values get rejected
```

### Why bother with encapsulation?

- **Data hiding** — internal data isn't directly exposed to the outside world.
- **Controlled access** — a method decides *how* data is allowed to change.
- **Validation** — invalid values (like a negative age) can be rejected before they're ever stored.
- **Maintainability** — you can change the internal implementation later without breaking code that uses the class, as long as the public methods stay the same.

### Encapsulation vs Data Hiding

They're related, but not identical:
- **Encapsulation** → bundling data + behavior together *and* controlling access to it (the broader concept).
- **Data hiding** → specifically, restricting direct access to the internal implementation/data (one *outcome* of encapsulation).

`private` fields are the tool most commonly used to achieve data hiding.

> 💡 **Interview definition to memorize:** *"Encapsulation is the process of wrapping data and the methods that operate on that data into a single unit (a class), while restricting direct access to the internal state."*

---

## 8. `static`

- A `static` member belongs to the **class**, not an instance — accessible without creating an object.
- A `static` method can access only `static` data directly; to touch instance data it needs an explicit object reference.
- A `static` method **cannot** use `this` or `super`, and cannot be overridden in the true polymorphic sense — static methods are resolved at **compile time** (no dynamic dispatch), so "overriding" a static method is actually **method hiding**.
- `static` blocks run exactly once, when the class is first loaded — useful for static field initialization.
- Only **nested** classes can be declared `static` (a static nested class doesn't hold an implicit reference to an instance of the enclosing class).
- Static interface methods are **not inherited** by implementing classes or sub-interfaces, and must have a body (can't be abstract).

```java
class UseStatic {
    static int a = 3;
    static int b;
    static { b = a * 4; }   // static block: runs once at class load
}
```

---

## 9. Inheritance

- Use `extends`. Java supports only **single inheritance of classes** (no multiple superclasses) — this is exactly the gap interfaces fill.
- A subclass inherits all superclass members **except private ones**.
- **A superclass reference variable can point to a subclass object** — but it can only access members defined in the superclass (this is the basis of runtime polymorphism):

```java
SUPERCLASS ref = new SUBCLASS();   // ref can only call methods declared in SUPERCLASS
```

### `super`
Two uses:
1. **Call the superclass constructor** — must be the **first statement** in the subclass constructor:
```java
BoxWeight(double w, double h, double d, double m) {
    super(w, h, d);   // must be first line
    weight = m;
}
```
2. **Access a superclass member hidden by the subclass**: `super.member`.

- `super()` always refers to the constructor of the **immediate** superclass, even in multi-level hierarchies.
- If `super()` is not called explicitly, Java inserts a call to the superclass's no-arg constructor automatically.

### `final` + inheritance
- `final` method → cannot be overridden (enables **early binding** / compiler inlining, vs. the default **late/dynamic binding**).
- `final` class → cannot be extended.
- **Note:** Polymorphism does not apply to instance variables (field hiding, not overriding) — only to methods.

---

## 10. Polymorphism

**Polymorphism** literally means "many forms" — the same method call or reference can behave differently depending on context. In Java, it comes in two flavors:

| | Compile-time (static) Polymorphism | Runtime (dynamic) Polymorphism |
|---|---|---|
| Achieved via | **Method overloading** | **Method overriding** |
| Decided when | At compile time, based on the reference type / argument types | At runtime, based on the actual object type |
| Example | `test(int)` vs `test(double)` in the same class | `Animal a = new Dog(); a.makeSound();` → runs `Dog`'s version |

See §11 for the full breakdown of overloading vs overriding, and §9 for how a superclass reference can point to a subclass object (the setup that makes runtime polymorphism possible).

> 💡 One important limit to remember: **polymorphism does not apply to fields** — only to methods. If a subclass "hides" a field with the same name, which one you get is decided by the **reference type**, not the actual object type (unlike methods, which use the actual object type).

---

## 11. Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Where | Same class (or subclass adding new signatures) | Subclass redefining a superclass method |
| Signature | Must **differ** (params) | Must be **identical** (name + parameter types) |
| Return type | Can differ, but return type alone can't distinguish two overloads | Java 5+ allows a **covariant return type** (subclass return type) |
| Binding | **Compile-time** (static binding) | **Runtime** (dynamic binding) |
| Access modifier | No restriction | Overriding method's access must be **same or wider**, never narrower (e.g., `protected` → `public` is fine, `protected` → `private` is not) |

- When resolving an overload, Java tries an **exact match first**; only if none exists does it apply automatic widening conversions (e.g., `int` → `double`).

```java
class OverloadDemo {
    void test(double a) {
        System.out.println("Inside test(double) a: " + a);
    }
}
// test(int) is never defined here.
// ob.test(88) still compiles — Java widens the int 88 to a double
// and calls test(double), since no exact int match exists.
```

### Dynamic Method Dispatch (the mechanism behind runtime polymorphism)
- When an overridden method is called through a superclass reference, **the type of the object being pointed to at runtime** (not the type of the reference variable) determines which version executes.
- This is literally *how* Java implements runtime polymorphism.

```java
Animal a = new Dog();
a.makeSound();   // Dog's version runs — decided at RUNTIME based on actual object type
```

### Returning Objects

A method can return a newly created object — this is a common pattern worth being comfortable with:

```java
class Test {
    int a;
    Test(int i) { a = i; }

    Test incrByTen() {
        Test temp = new Test(a + 10);
        return temp;
    }
}

Test ob1 = new Test(2);
Test ob2 = ob1.incrByTen();
// ob1.a is still 2 — a brand-new object was returned, ob1 was untouched
// ob2.a is 12
```

Each call to `incrByTen()` creates a fresh object and returns a reference to it. Because Java objects are always allocated with `new`, you don't need to worry about an object "going out of scope" when the method that created it returns — the object keeps existing as long as *any* reference to it exists anywhere in the program. Once nothing references it anymore, it becomes eligible for garbage collection (see §2).

---

## 12. Abstraction — `abstract` classes vs `interface`

### Abstract classes
- Declared with `abstract`; may contain both abstract (no body) and concrete methods.
- **Cannot be instantiated**, but you *can* have a reference variable of an abstract type (used for runtime polymorphism).
- Cannot have `abstract` constructors or `abstract static` methods (doesn't make sense — nothing is ever instantiated to call them polymorphically).
- Can have `static` methods.
- Any subclass must implement **all** abstract methods, or be declared `abstract` itself.

### Interfaces
- All methods implicitly `public` and `abstract` by default (pre-Java 8).
- All variables implicitly `public static final` (i.e., constants) — must be initialized at declaration.
- Since Java 8: interfaces can have `default` and `static` methods (with bodies).
- A class can implement **multiple** interfaces — this is Java's answer to the diamond problem of multiple inheritance.
- Interfaces **cannot maintain state** (no instance variables) — this is the fundamental distinction from a class.

### Head-to-head comparison (frequently asked as-is)

| | Abstract Class | Interface |
|---|---|---|
| Methods | Abstract + concrete | Only abstract (default/static allowed from Java 8) |
| Variables | Final, non-final, static, non-static | Only `public static final` |
| Multiple inheritance | Extends only 1 class, implements many interfaces | Can extend multiple interfaces |
| Member access modifiers | private/protected/public allowed | Everything implicitly public |
| Keyword | `extends` | `implements` |
| Constructors | Can have one (called via subclass) | Cannot have one |

> 💡 **Why can't an interface have a constructor but an abstract class can?**
> An abstract class constructor exists to be called via `super()` from a concrete subclass during object construction. An interface has no state and isn't part of the single-inheritance chain, so there's no meaningful "construction" step for it.

### Default methods (Java 8+) — resolution rules
1. A **class's own implementation always wins** over any interface default.
2. If a class implements two interfaces with the **same default method** and doesn't override it → **compile error** (must resolve explicitly).
3. If interface `B extends A` and both define the same default method, **B's version wins** (more specific interface takes precedence).

### Nested (Member) Interfaces

An interface can be declared **inside a class or another interface** — this is called a **member interface** or **nested interface**. Unlike a top-level interface (which must be `public` or default-access), a nested interface can be `public`, `private`, or `protected`.

```java
class A {
    // this is a nested interface
    public interface NestedIF {
        boolean isNotNegative(int x);
    }
}

class B implements A.NestedIF {
    public boolean isNotNegative(int x) {
        return x >= 0;
    }
}

class NestedIFDemo {
    public static void main(String[] args) {
        A.NestedIF nif = new B();
        System.out.println(nif.isNotNegative(10));   // true
        System.out.println(nif.isNotNegative(-12));  // false
    }
}
```

---

## 13. Packages

- Packages are namespacing + access-control containers for classes.
- Package structure **must mirror the file system directory structure** exactly (case-sensitive) — e.g. `package java.awt.image;` must live under `java/awt/image` on disk.
- Java looks for packages in this order: current working directory → `CLASSPATH` env variable → `-classpath` compiler/runtime flag.
- Only members declared `public` are visible outside the package when imported.

---

## 14. Enums

- Created with `enum`; internally, Java converts each constant into a `public static final` instance of that type.
- All enums implicitly extend `java.lang.Enum` → since Java has single inheritance, **an enum cannot extend anything else**, and **cannot itself be a superclass**.
- An enum **can implement interfaces**.
- Enum constructors are implicitly `private` (or default) — never `public`/`protected`, because you should never be able to create additional instances beyond the declared constants.
- Enums can only contain **concrete** methods, never abstract ones.
- Useful built-ins (from `java.lang.Enum`): `values()` (all constants), `ordinal()` (index), `valueOf(String)` (constant by name).
- Compare enum constants with `==` (safe — same enum & same constant) or `.equals()`.

---

## 15. The Four Pillars of OOP — Summary

A quick recall map for interviews — every OOP topic above falls under one of these four pillars:

```
                    OOP
                     |
        -----------------------------
        |            |              |
  Encapsulation  Inheritance   Polymorphism
                                     |
                               Abstraction
```

| Pillar | One-line meaning | Java mechanism |
|---|---|---|
| **Encapsulation** | Protect and control access to data | `private` fields + public getters/setters (§7) |
| **Inheritance** | Reuse and extend existing classes | `extends`, `super` (§9) |
| **Polymorphism** | One interface/reference, many behaviors | Overloading (compile-time), Overriding (runtime) (§10, §11) |
| **Abstraction** | Hide implementation details, expose only essential behavior | `abstract` classes, `interface` (§12) |

---

## 16. Singleton Pattern (common OOP design-pattern interview question)

```java
public class Singleton {
    private Singleton() { }                 // private constructor — blocks external `new`
    private static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
- Guarantees only **one instance** exists across the JVM; all callers of `getInstance()` get the same reference.
- **Interview follow-ups to be ready for:**
  - This version isn't thread-safe — two threads could both pass the `null` check simultaneously. Fix with `synchronized`, double-checked locking, an `enum` singleton, or eager initialization.
  - Enum-based singleton is considered the most robust (handles serialization & reflection attacks automatically).

---

## 17. Rapid-Fire Interview Q&A

**Q: Why doesn't Java support multiple inheritance of classes?**
A: Ambiguity — the "diamond problem." If two parent classes define the same method differently, the compiler can't decide which to inherit. Interfaces sidestep this because (pre-Java 8) they carried no implementation, and even with default methods, Java forces explicit resolution on conflicts rather than silently picking one.

**Q: Can you override a static method?**
A: No — you can only **hide** it. Static methods are resolved at compile-time based on the reference type, not the runtime object type, so there's no dynamic dispatch involved.

**Q: What's the difference between method overloading and overriding, in one line?**
A: Overloading = same name, different parameters, resolved at compile-time; Overriding = same signature in a subclass, resolved at runtime based on actual object type.

**Q: Can an abstract class have a constructor?**
A: Yes — even though it can't be instantiated directly, its constructor runs when a concrete subclass is instantiated via `super()`.

**Q: What happens if you don't call `super()` explicitly in a constructor?**
A: The compiler inserts a call to the superclass's no-arg constructor automatically. If the superclass has no no-arg constructor, this is a compile error and you must call `super(args)` explicitly.

**Q: Is `final` on an object reference enough to make it immutable?**
A: No — `final` only prevents the reference from being reassigned. The object it points to can still have its internal state mutated (unless the class itself is designed to be immutable).

**Q: Why can interfaces not hold instance state?**
A: Interface variables are implicitly `public static final` — constants, not instance fields — which is exactly what keeps interfaces about *behavior contracts*, not state.

**Q: What determines which overridden method gets called — the reference type or the object type?**
A: The **actual object type at runtime** (dynamic method dispatch) — this is the mechanism behind runtime polymorphism.

**Q: Does polymorphism apply to fields/instance variables?**
A: No — field access is resolved at compile-time based on the **reference type**, not the object type. Only method calls get dynamic dispatch.

**Q: What's the difference between encapsulation and data hiding?**
A: Encapsulation is the broader idea — bundling data and behavior into a class and controlling access to it. Data hiding is one specific outcome of encapsulation: restricting direct access to internal fields (typically via `private`).

**Q: Where do objects live in memory — stack or heap?**
A: Objects themselves live on the **heap**; local reference variables that point to them live on the **stack**, as part of the current method's stack frame.

**Q: What's the real difference between a "default constructor" and a "no-argument constructor"?**
A: A default constructor is specifically the no-arg constructor the *compiler* auto-generates when you write no constructor at all. A no-argument constructor is any zero-parameter constructor — hand-written or compiler-generated. All default constructors are no-arg constructors, but not all no-arg constructors are "the default" one.

---

*Source: [kunal-kushwaha/DSA-Bootcamp-Java](https://github.com/kunal-kushwaha/DSA-Bootcamp-Java) — [`lectures/17-oop`](https://github.com/kunal-kushwaha/DSA-Bootcamp-Java/tree/main/lectures/17-oop) (notes & code) — plus supplementary sections on Stack/Heap, Encapsulation, Polymorphism, and Static Typing added for interview completeness.*
