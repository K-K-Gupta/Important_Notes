# LLD Notes: Part 1. Foundation & OOPs (C++)

---

## Table of Contents

1. What is LLD? (LLD vs HLD)
2. Why OOP for LLD?
3. Classes and Objects
4. Access Specifiers
5. Constructors, Destructors, and the Rule of 3/5/0
6. The `this` pointer, `static`, `const`
7. Pillar 1: Encapsulation
8. Pillar 2: Abstraction
9. Pillar 3: Inheritance
10. Pillar 4: Polymorphism
11. Abstract Class vs Interface (C++)
12. Composition vs Inheritance ("Has-A" vs "Is-A")
13. Association, Aggregation, Composition (intro)
14. Coupling and Cohesion
15. Object lifetime and smart pointers (LLD essentials)
16. Common OOP interview Q&A
17. One-page cheat sheet

---

## 1. What is LLD? (LLD vs HLD)

**Low Level Design (LLD)** = designing the **classes, objects, interfaces, relationships, and interactions** inside a system so the code is clean, extensible, and maintainable.

**High Level Design (HLD)** = designing the **overall architecture**: services, databases, load balancers, caches, queues, and how they communicate.

| Aspect | HLD | LLD |
|---|---|---|
| Focus | System architecture | Code-level structure |
| Output | Architecture diagram, components, data flow | Class diagram, interfaces, code |
| Concerns | Scalability, availability, latency | Maintainability, extensibility, readability |
| Example | "Design Uber": services, DBs, geo-index | "Design Parking Lot": classes, methods, patterns |
| Language-specific? | No | Yes (you write real classes) |

**What an interviewer checks in an LLD round**
- Can you turn vague requirements into **entities (classes)**?
- Do you apply **SOLID** and **design patterns** where they fit (not everywhere)?
- Is your design **extensible** (a new requirement should not force rewrites)?
- Can you write **clean, compilable code**?

> ⚡ **Quick Recall**
> HLD = boxes and arrows between services. LLD = classes and methods inside one service.

---

## 2. Why OOP for LLD?

Procedural code = functions operating on shared data. As it grows, everything depends on everything.

OOP fixes this by:
- **Bundling data and behavior** together (class).
- **Hiding internals** (encapsulation).
- **Modelling real-world entities** (Car, User, Order).
- **Enabling reuse and extension** without modifying existing code (inheritance, polymorphism).

**Real-world analogy:** A TV remote. You press buttons (public interface). You don't touch the circuit board (private internals).

---

## 3. Classes and Objects

- **Class**: a blueprint. Defines *data members* (state) and *member functions* (behavior).
- **Object**: a concrete instance of a class, occupying memory.

```cpp
#include <iostream>
#include <string>
using namespace std;

class Car {
public:
    string brand;      // state
    int speed;

    void accelerate(int delta) {   // behavior
        speed += delta;
    }
    void show() {
        cout << brand << " running at " << speed << " km/h\n";
    }
};

int main() {
    Car c1;                 // object on stack
    c1.brand = "Tata";
    c1.speed = 40;
    c1.accelerate(20);
    c1.show();              // Tata running at 60 km/h

    Car* c2 = new Car();    // object on heap
    c2->brand = "Maruti";
    c2->speed = 30;
    c2->show();
    delete c2;              // heap objects must be freed
}
```

**Key points**
- **Stack object**: destroyed automatically at end of scope.
- **Heap object** (`new`): you must `delete` it (or use smart pointers, see section 15).
- In C++, `struct` and `class` are identical except **default access**: `struct` → public, `class` → private.

> ⚡ **Quick Recall**
> Class = blueprint (no memory). Object = instance (has memory). `struct` default public, `class` default private.

---

## 4. Access Specifiers

| Specifier | Same class | Derived class | Outside |
|---|---|---|---|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

```cpp
class BankAccount {
private:
    double balance;          // hidden from everyone outside
protected:
    string accountType;      // visible to derived classes
public:
    void deposit(double amt) { balance += amt; }
};
```

**Inheritance mode also matters** (`class D : public B`):
- `public` inheritance keeps base access levels as-is. **This is the one you use for "is-a".**
- `protected` / `private` inheritance turn base public members into protected/private in the derived class (rare, used for "implemented-in-terms-of").

**Interview trap:** `friend` functions/classes can access private members. It breaks encapsulation, so use it sparingly (e.g., `operator<<`).

---

## 5. Constructors, Destructors, and the Rule of 3/5/0

### Constructor types

```cpp
class Student {
    string name;
    int age;
public:
    Student() : name("NA"), age(0) {}                       // default
    Student(string n, int a) : name(n), age(a) {}           // parameterized (use init list)
    Student(const Student& other)                           // copy constructor
        : name(other.name), age(other.age) {}
    Student(Student&& other) noexcept                       // move constructor
        : name(std::move(other.name)), age(other.age) {}
    ~Student() {}                                           // destructor
};
```

**Initializer list** (`: name(n), age(a)`) is preferred because:
1. It **initializes** members directly (assignment in the body = default-construct first, then assign, which is slower).
2. It is **mandatory** for `const` members, reference members, and base-class constructors.

**Order of initialization** = order of **declaration in class**, not order in the initializer list.

### Shallow vs Deep copy

```cpp
class Buffer {
    int* data;
    int size;
public:
    Buffer(int n) : size(n), data(new int[n]) {}
    // Deep copy constructor
    Buffer(const Buffer& o) : size(o.size), data(new int[o.size]) {
        for (int i = 0; i < size; i++) data[i] = o.data[i];
    }
    // Copy assignment
    Buffer& operator=(const Buffer& o) {
        if (this == &o) return *this;   // self-assignment guard
        delete[] data;
        size = o.size;
        data = new int[size];
        for (int i = 0; i < size; i++) data[i] = o.data[i];
        return *this;
    }
    ~Buffer() { delete[] data; }
};
```
Without the custom copy, the default **shallow copy** copies the pointer, so two objects point to the same memory, leading to a **double-free** crash.

### Rule of 3 / 5 / 0
- **Rule of 3**: if you define any of *destructor, copy constructor, copy assignment*, you probably need all three.
- **Rule of 5**: add *move constructor* and *move assignment* (C++11).
- **Rule of 0**: best case. Use `std::vector`, `std::string`, smart pointers so you write **none** of them.

### Virtual destructor (very common question)
```cpp
class Base { public: virtual ~Base() {} };   // MUST be virtual if used polymorphically
class Derived : public Base { int* p = new int[10]; public: ~Derived() { delete[] p; } };

Base* b = new Derived();
delete b;   // without virtual ~Base(), Derived's destructor never runs → memory leak
```

> ⚡ **Quick Recall**
> Use initializer lists. Owning raw pointer → Rule of 5 (or better: Rule of 0 with smart pointers). Base class with virtual functions → **virtual destructor**.

---

## 6. `this` pointer, `static`, `const`

### `this`
Implicit pointer to the current object inside non-static member functions. Used to disambiguate names and for **method chaining**.

```cpp
class Pizza {
    int size; string topping;
public:
    Pizza& setSize(int size) { this->size = size; return *this; }
    Pizza& setTopping(string t) { topping = t; return *this; }
};
Pizza p;
p.setSize(12).setTopping("cheese");   // chaining (the idea behind the Builder pattern)
```

### `static`
- **Static data member**: one copy shared by **all** objects. Defined outside the class (pre-C++17) or with `inline static` (C++17).
- **Static member function**: no `this`, so it can only touch static members. Called via `ClassName::fn()`.

```cpp
class IdGenerator {
    static int counter;             // shared
public:
    static int next() { return ++counter; }
};
int IdGenerator::counter = 0;       // definition
// IdGenerator::next();
```
Real LLD use: **Singleton** instance, ID generators, counters.

### `const`
```cpp
class Circle {
    const double PI = 3.14159;      // const data member (initialize via init list or in-class)
    double r;
public:
    double area() const {           // const member function: promises NOT to modify object
        return PI * r * r;
    }
};
```
- A `const` object can only call `const` member functions.
- `mutable` members can change even inside a `const` function (e.g., caches, mutex).

> ⚡ **Quick Recall**
> `static` = belongs to the class, not the object. `const` method = read-only on `this`. `mutable` = exception to that.

---

## 7. Pillar 1: Encapsulation

**Definition:** Bundle data and the functions that operate on it into one unit (class), and **restrict direct access** to the data.

**Why:** Protects invariants. Nobody can set `balance = -5000` directly; the class controls how state changes.

**Bad**
```cpp
class Account { public: double balance; };
Account a; a.balance = -9999;    // nothing stops this
```

**Good**
```cpp
class Account {
    double balance = 0;
public:
    bool deposit(double amt) {
        if (amt <= 0) return false;      // validation lives here
        balance += amt; return true;
    }
    bool withdraw(double amt) {
        if (amt <= 0 || amt > balance) return false;
        balance -= amt; return true;
    }
    double getBalance() const { return balance; }
};
```

**Interview points**
- Encapsulation = **data hiding + bundling**. Data hiding is achieved through access specifiers.
- Don't blindly generate getters/setters for everything. That is "encapsulation in name only". Expose **behavior** (`withdraw`), not raw state (`setBalance`).
- Benefit: you can change internal representation without breaking callers.

---

## 8. Pillar 2: Abstraction

**Definition:** Expose **what** an object does, hide **how** it does it. Show only essential features.

**Encapsulation vs Abstraction (asked constantly)**

| | Encapsulation | Abstraction |
|---|---|---|
| Question answered | *How do I protect data?* | *What should the user see?* |
| Level | Implementation level | Design level |
| Achieved by | Access specifiers | Abstract classes / interfaces |
| Analogy | Capsule that hides medicine inside | Car pedals: you don't need to know the engine |

**Code: abstraction via abstract class**
```cpp
class PaymentMethod {                    // abstraction: WHAT
public:
    virtual bool pay(double amount) = 0;
    virtual ~PaymentMethod() = default;
};

class UPI : public PaymentMethod {       // HOW is hidden here
public:
    bool pay(double amount) override {
        cout << "Paid " << amount << " via UPI\n"; return true;
    }
};
```
Client code depends only on `PaymentMethod`, never on `UPI`'s internals.

> ⚡ **Quick Recall**
> Abstraction = hide **complexity** (design). Encapsulation = hide **data** (implementation).

---

## 9. Pillar 3: Inheritance

**Definition:** A class (derived/child) acquires properties and behavior of another (base/parent). Models an **"is-a"** relationship.

```cpp
class Vehicle {
protected:
    string regNo;
public:
    Vehicle(string r) : regNo(r) {}
    virtual void start() { cout << "Vehicle starting\n"; }
    virtual ~Vehicle() = default;
};

class Car : public Vehicle {
public:
    Car(string r) : Vehicle(r) {}                    // call base constructor
    void start() override { cout << "Car engine on\n"; }
};
```

**Constructor / destructor order**
- Construction: **Base → Derived**.
- Destruction: **Derived → Base**.

### Types of inheritance
| Type | Shape |
|---|---|
| Single | A → B |
| Multilevel | A → B → C |
| Hierarchical | A → B, A → C |
| Multiple | A, B → C (C++ supports this; Java doesn't for classes) |
| Hybrid | Mix of the above |

### The Diamond Problem
```
      Animal
      /    \
   Dog      Cat
      \    /
      Hybrid          // two copies of Animal inside Hybrid → ambiguity
```
**Fix in C++: virtual inheritance**
```cpp
class Animal { public: int age; };
class Dog : virtual public Animal {};
class Cat : virtual public Animal {};
class Hybrid : public Dog, public Cat {};   // only ONE Animal subobject now
```

### Object slicing (classic C++ trap)
```cpp
Vehicle v = Car("MH12");   // copies only the Vehicle part; Car part is SLICED off
v.start();                  // prints "Vehicle starting", not "Car engine on"
```
**Fix:** use pointers/references (`Vehicle* v = new Car(...)`, or `unique_ptr<Vehicle>`).

### Name hiding
If Derived defines a function with the **same name** as Base (even a different signature), it **hides all** Base overloads. Fix with `using Base::fn;`.

**Interview points**
- Prefer **composition over inheritance** (section 12). Inheritance couples child tightly to parent.
- Inherit **publicly** only when the relationship is genuinely "is-a" **and** the child honors the parent's contract (LSP, covered in the SOLID part).

---

## 10. Pillar 4: Polymorphism

**Definition:** "Many forms". The same interface, different behavior depending on the object.

### A) Compile-time polymorphism (static binding)
1. **Function overloading**: same name, different parameter lists.
2. **Operator overloading**: give operators custom meaning for your types.
3. **Templates**: generic code resolved at compile time.

```cpp
class Calc {
public:
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }        // overload
    int add(int a, int b, int c) { return a + b + c; }      // overload
};

class Vec2 {
public:
    int x, y;
    Vec2 operator+(const Vec2& o) const { return {x + o.x, y + o.y}; }   // operator overloading
};
```

### B) Run-time polymorphism (dynamic binding)
Achieved by **virtual functions** + **base pointer/reference**. The call is resolved at **runtime** based on the actual object type.

```cpp
class Shape {
public:
    virtual double area() const = 0;      // pure virtual → Shape is abstract
    virtual ~Shape() = default;
};
class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
};
class Rect : public Shape {
    double w, h;
public:
    Rect(double w, double h) : w(w), h(h) {}
    double area() const override { return w * h; }
};

void printArea(const Shape& s) { cout << s.area() << "\n"; }   // works for ANY Shape, now or future
```

**How it works internally (interview favorite)**
- Every class with virtual functions gets a **vtable** (table of function pointers, one per class).
- Every object of such a class carries a hidden **vptr** pointing to its class's vtable.
- A virtual call = *follow vptr → look up slot → call*. That is why it costs a little more than a normal call.

### Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Binding | Compile-time | Run-time |
| Scope | Same class | Base ↔ Derived |
| Signature | Must differ | Must be **same** |
| Needs `virtual`? | No | Yes (on base) |

### Good habits
- Always write **`override`** on overriding functions. The compiler catches signature mismatches.
- **`final`** stops further overriding/inheriting.
- Never call virtual functions from a **constructor/destructor**. The derived part doesn't exist yet or is already gone, so the base version runs.

> ⚡ **Quick Recall**
> Compile-time = overloading, operators, templates. Run-time = `virtual` + base pointer/reference. Mechanism = vptr → vtable. Always `override`, always virtual destructor.

---

## 11. Abstract Class vs Interface (C++)

C++ has **no `interface` keyword**. Convention: a class with **only pure virtual functions** (and a virtual destructor) is an *interface*.

```cpp
// Interface (by convention)
class ILogger {
public:
    virtual void log(const string& msg) = 0;
    virtual ~ILogger() = default;
};

// Abstract class (mix of pure virtual + concrete + state)
class Employee {
protected:
    string name;
public:
    Employee(string n) : name(n) {}
    virtual double salary() const = 0;            // must be implemented
    void printName() const { cout << name; }      // shared implementation
    virtual ~Employee() = default;
};
```

| | Interface (pure abstract) | Abstract class |
|---|---|---|
| Data members | None (ideally) | Allowed |
| Concrete methods | No | Yes |
| Constructors | Not needed | Yes (called by derived) |
| Purpose | Pure **contract** ("can-do") | Partial implementation + contract ("is-a") |
| Multiple inheritance | Safe and common | Risky |

**Rules**
- A class with **≥ 1 pure virtual function** is abstract and **cannot be instantiated**.
- A pure virtual function *can* have a body (rare, but legal, and is required for pure virtual destructors).
- A derived class that doesn't override **all** pure virtuals remains abstract.

**When to use which**
- Need to share code/state among related classes → **abstract class**.
- Need to define a capability that unrelated classes can adopt (`Printable`, `Serializable`) → **interface**.

---

## 12. Composition vs Inheritance ("Has-A" vs "Is-A")

**Inheritance** = "Car **is a** Vehicle".
**Composition** = "Car **has an** Engine".

**Why "favor composition over inheritance"?**
- Inheritance is **compile-time, rigid**, and exposes parent internals to child (fragile base class problem).
- Composition is **flexible**: swap parts at runtime, no deep hierarchies, no diamond issues.

**Bad: inheritance for reuse**
```cpp
class Stack : public vector<int> {};   // Stack "is-a" vector? No. Exposes push_back at index, insert, etc.
```

**Good: composition**
```cpp
class Stack {
    vector<int> data;                  // has-a
public:
    void push(int x) { data.push_back(x); }
    void pop() { data.pop_back(); }
    int top() const { return data.back(); }
};
```

**Composition enabling runtime flexibility (preview of Strategy pattern)**
```cpp
class Engine { public: virtual void start() = 0; virtual ~Engine() = default; };
class Petrol : public Engine { public: void start() override { cout << "Vroom\n"; } };
class Electric : public Engine { public: void start() override { cout << "Hummm\n"; } };

class Car {
    unique_ptr<Engine> engine;         // composed, swappable
public:
    Car(unique_ptr<Engine> e) : engine(std::move(e)) {}
    void start() { engine->start(); }
};
```

**Rule of thumb:** Use inheritance only for a true "is-a" **with a stable contract**. Everything else: compose.

---

## 13. Association, Aggregation, Composition (Intro)

(Detailed UML notation comes in the UML part. This is the concept.)

| Relationship | Meaning | Lifetime | Example | C++ shape |
|---|---|---|---|---|
| **Association** | "uses / knows" | Independent | Teacher ↔ Student | pointer/reference |
| **Aggregation** | "has-a", **weak** ownership | Part outlives whole | Department has Teachers | `Teacher*` / `shared_ptr` |
| **Composition** | "has-a", **strong** ownership | Part dies with whole | House has Rooms | member object / `unique_ptr` |

```cpp
class Teacher {};
class Room {};

class Department {              // Aggregation: teachers exist independently
    vector<Teacher*> teachers;
};

class House {                   // Composition: rooms die with the house
    vector<Room> rooms;
};
```

> ⚡ **Quick Recall**
> Composition = part cannot exist without whole. Aggregation = part can. Association = just uses.

---

## 14. Coupling and Cohesion

**Goal of good design: LOW coupling, HIGH cohesion.**

### Coupling
How much one class **depends on** another.
- **Tight coupling**: class creates and depends on concrete classes. Changing one breaks the other.
- **Loose coupling**: class depends on **abstractions** (interfaces), so implementations can be swapped.

```cpp
// TIGHT
class OrderService {
    MySQLDatabase db;               // hard-wired to MySQL
};

// LOOSE (dependency injection)
class Database { public: virtual void save(string) = 0; virtual ~Database() = default; };
class OrderService {
    Database& db;                   // depends on abstraction
public:
    OrderService(Database& d) : db(d) {}
};
```

### Cohesion
How closely related the responsibilities **inside** one class are.
- **High cohesion**: `Invoice` handles only invoice data and totals. ✅
- **Low cohesion**: `Utils` class doing file IO, email, and math. ❌ ("God class")

> ⚡ **Quick Recall**
> Low coupling = depend on interfaces, inject dependencies. High cohesion = one class, one focused purpose. (This leads directly into **SRP** and **DIP** in the SOLID part.)

---

## 15. Object Lifetime and Smart Pointers (LLD Essentials)

Modern C++ interviews expect **no raw `new`/`delete`** in design code.

| Smart pointer | Ownership | Use when |
|---|---|---|
| `unique_ptr<T>` | Exclusive (move-only) | One clear owner (default choice) |
| `shared_ptr<T>` | Shared (ref-counted) | Multiple owners (aggregation) |
| `weak_ptr<T>` | Non-owning observer | Break **cyclic references**, observers |

```cpp
#include <memory>
auto a = make_unique<Car>("MH12");            // sole owner
auto b = make_shared<Car>("KA01");            // shared
shared_ptr<Car> c = b;                        // ref count = 2
weak_ptr<Car> w = b;                          // no count increase
if (auto locked = w.lock()) { /* safe use */ }
```

**Cyclic reference leak:** if `A` holds `shared_ptr<B>` and `B` holds `shared_ptr<A>`, neither is ever freed. Make one side a `weak_ptr`.

**RAII** (Resource Acquisition Is Initialization): tie resource lifetime to object lifetime. Acquire in constructor, release in destructor. Smart pointers, `lock_guard`, and `fstream` all follow it.

---

## 16. Common OOP Interview Q&A

**Q1. Four pillars of OOP?**
Encapsulation, Abstraction, Inheritance, Polymorphism, each with a one-line definition and an example from above.

**Q2. Difference between class and struct in C++?**
Only default access (private vs public) and default inheritance mode. Otherwise identical.

**Q3. Why virtual destructor?**
So `delete basePtr` calls the derived destructor, preventing leaks and undefined behavior.

**Q4. Can a constructor be virtual? Why not?**
No. The vptr is set up *during* construction, so the mechanism doesn't exist yet. (Use the Factory pattern for the same effect.)

**Q5. Can a static function be virtual?**
No. Static functions have no `this`, so there is no object to consult a vptr on.

**Q6. Abstract class vs interface in C++?**
See section 11. An interface is a class with only pure virtuals.

**Q7. What is object slicing?**
Assigning a derived object to a base object by value chops off the derived part. Avoid by using pointers/references.

**Q8. Overloading vs Overriding vs Hiding?**
Overloading: same scope, different params. Overriding: virtual, same signature, base↔derived. Hiding: derived name shadows base name.

**Q9. Shallow vs deep copy?**
Shallow copies pointers (shared memory, double-free risk). Deep copies the pointed-to data.

**Q10. Why prefer composition over inheritance?**
Less coupling, runtime flexibility, no fragile-base-class or diamond problem.

**Q11. What is the diamond problem and how does C++ solve it?**
Two paths to the same base give duplicate subobjects. Solved with `virtual` inheritance.

**Q12. What is RAII?**
Bind resource lifetime to object scope, so cleanup happens automatically even on exceptions.

**Q13. `const` member function, what does it mean?**
It won't modify the object's non-`mutable` state; callable on `const` objects.

**Q14. When would you use `protected`?**
When derived classes need access but the outside world must not. Use sparingly, since it still leaks internals to subclasses.

**Q15. Is C++ fully object-oriented?**
No, it's multi-paradigm (procedural, OOP, generic, functional). Free functions and primitives exist outside classes.

---

## 17. One-Page Cheat Sheet

```
LLD          → classes + relationships + interactions (vs HLD = architecture)
Class/Object → blueprint / instance   |  struct=public, class=private (default)
Ctor order   → Base → Derived         |  Dtor order → Derived → Base
Init list    → required for const, reference, base-class ctor
Rule 3/5/0   → own resource? define dtor+copy(+move); else use RAII/smart ptrs (Rule 0)

ENCAPSULATION → hide data (private + methods)          "capsule"
ABSTRACTION   → hide complexity (abstract class/iface)  "car pedals"
INHERITANCE   → is-a  (public inheritance)              diamond → virtual inheritance
POLYMORPHISM  → compile: overload/operator/template
                run-time: virtual + base ptr/ref  (vptr → vtable)

ALWAYS: virtual destructor in polymorphic base, `override` keyword,
        avoid slicing (use ptr/ref), no virtual calls in ctor/dtor.

Abstract class → partial impl + state    | Interface → pure virtual only
Association → uses | Aggregation → has (part outlives) | Composition → owns (part dies)
Composition over Inheritance.
Low Coupling (depend on abstractions) + High Cohesion (one purpose per class).
Smart ptrs: unique (default) > shared (multi-owner) > weak (break cycles).
```

---
