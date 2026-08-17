# Java OOP — Interview Prep Notes
*Compiled from Kunal Kushwaha's DSA-Bootcamp-Java (`lectures/17-oop`)*

---

## 1. Classes & Objects

- A **class** is a template/blueprint for an object; an **object** is an instance of a class. A class is a logical construct — an object has physical reality (it occupies memory).
- Every object has three properties: **state** (data), **identity** (its unique memory location), **behavior** (what its methods do).
- The `new` keyword dynamically allocates memory at runtime and returns a **reference** to the object — not an actual pointer (you can't do pointer arithmetic on it, which is part of Java's safety model).

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

---

## 2. `this`, `final`, and `finalize()`

### `this`
Refers to the current object — the object on which the currently executing method was invoked. Commonly used to disambiguate instance variables from parameters with the same name.

### `final`
- On a variable → makes it a constant; must be initialized when declared.
- **Gotcha:** `final` on a reference-type variable only freezes the *reference* (it will always point to the same object) — the internal state of that object can still change. `final` does **not** make objects immutable.
- On a method → prevents overriding (see §7).
- On a class → prevents inheritance (implicitly makes all its methods `final` too). A class **cannot** be both `abstract` and `final`.

### `finalize()`
- Called by the JVM right before an object is garbage collected — used to release non-memory resources.
- **Interview note:** `finalize()` is deprecated since Java 9 and removed in later versions in favor of `try-with-resources` / `Cleaner`. Good to mention you know this if it comes up, even though the source material predates it.

---

## 3. Constructors

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

---

## 4. Access Control

| Modifier | Class | Package | Subclass (same pkg) | Subclass (diff pkg) | World |
|---|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ✅ | ❌ |
| *default* (no modifier) | ✅ | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ | ❌ |

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
Reasoning: `Derived` only has the right to touch `protected` state it inherited itself — not `protected` state of an arbitrary `Base` instance (which might belong to some *other* subclass `Derived` knows nothing about).

---

## 5. `static`

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

## 6. Inheritance

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

## 7. Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Where | Same class (or subclass adding new signatures) | Subclass redefining a superclass method |
| Signature | Must **differ** (params) | Must be **identical** (name + parameter types) |
| Return type | Can differ, but return type alone can't distinguish two overloads | Java 5+ allows a **covariant return type** (subclass return type) |
| Binding | **Compile-time** (static binding) | **Runtime** (dynamic binding) |
| Access modifier | No restriction | Overriding method's access must be **same or wider**, never narrower (e.g., `protected` → `public` is fine, `protected` → `private` is not) |

- When resolving an overload, Java tries an **exact match first**; only if none exists does it apply automatic widening conversions (e.g., `int` → `double`).

### Dynamic Method Dispatch (the mechanism behind runtime polymorphism)
- When an overridden method is called through a superclass reference, **the type of the object being pointed to at runtime** (not the type of the reference variable) determines which version executes.
- This is literally *how* Java implements runtime polymorphism.

```java
Animal a = new Dog();
a.makeSound();   // Dog's version runs — decided at RUNTIME based on actual object type
```

---

## 8. Abstraction — `abstract` classes vs `interface`

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

---

## 9. Packages

- Packages are namespacing + access-control containers for classes.
- Package structure **must mirror the file system directory structure** exactly (case-sensitive).
- Classpath resolution order: current working directory → `CLASSPATH` env variable → `-classpath` compiler/runtime flag.
- Only members declared `public` are visible outside the package when imported.

---

## 10. Enums

- Created with `enum`; internally, Java converts each constant into a `public static final` instance of that type.
- All enums implicitly extend `java.lang.Enum` → since Java has single inheritance, **an enum cannot extend anything else**, and **cannot itself be a superclass**.
- An enum **can implement interfaces**.
- Enum constructors are implicitly `private` (or default) — never `public`/`protected`, because you should never be able to create additional instances beyond the declared constants.
- Enums can only contain **concrete** methods, never abstract ones.
- Useful built-ins (from `java.lang.Enum`): `values()` (all constants), `ordinal()` (index), `valueOf(String)` (constant by name).
- Compare enum constants with `==` (safe — same enum & same constant) or `.equals()`.

---

## 11. Singleton Pattern (common OOP design-pattern interview question)

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

## 12. Rapid-Fire Interview Q&A

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

---

*Source: [kunal-kushwaha/DSA-Bootcamp-Java](https://github.com/kunal-kushwaha/DSA-Bootcamp-Java) — `lectures/17-oop/notes/` and `lectures/17-oop/code/`*
