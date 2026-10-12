# LLD Notes: Part 2. SOLID Principles & Clean Code (C++)
---

## Table of Contents

1. Why SOLID? (the problem it solves)
1. **S**: Single Responsibility Principle (SRP)
2. **O**: Open/Closed Principle (OCP)
3. **L**: Liskov Substitution Principle (LSP) ← *deep dive*
4. **I**: Interface Segregation Principle (ISP)
5. **D**: Dependency Inversion Principle (DIP)
6. How the five principles connect
7. DRY, KISS, YAGNI (plus Law of Demeter, Composition over Inheritance)
8. Over-engineering: when NOT to apply SOLID
9. Interview Q&A
10. One-page cheat sheet

---

## 1. Why SOLID?

SOLID is a set of 5 guidelines (popularized by Robert C. Martin, "Uncle Bob") to make OO code:
- **Easy to change** (new feature does not mean a rewrite)
- **Easy to test** (dependencies can be swapped for fakes)
- **Easy to understand** (each class has one clear job)

**Symptoms of bad design that SOLID fixes ("code smells")**
| Smell | Meaning |
|---|---|
| **Rigidity** | One change forces changes in many places |
| **Fragility** | A change in one place breaks unrelated places |
| **Immobility** | Can't reuse a class because it drags dependencies with it |
| **Viscosity** | Easier to hack than to do the right thing |

> ⚡ **One-liners**
> **S**RP: one reason to change · **O**CP: extend, don't edit · **L**SP: subtypes must be substitutable · **I**SP: small focused interfaces · **D**IP: depend on abstractions

---

## 1. S: Single Responsibility Principle (SRP)

### Definition
> *A class should have **only one reason to change**.*

"Reason to change" = one **actor / stakeholder** who might request a change. If the accounts team and the tech team could both force changes to the same class, it has 2 responsibilities.

**Analogy:** A chef cooks; a waiter serves; a cashier bills. If one person does all three, a change in billing rules disturbs the cooking.

### ❌ Violation

```cpp
class Invoice {
    double amount;
    string customer;
public:
    Invoice(string c, double a) : customer(c), amount(a) {}

    double calculateTotal() { return amount * 1.18; }        // 1) business logic

    void saveToDatabase() { /* SQL insert */ }               // 2) persistence

    void printInvoice() { cout << customer << ": " << calculateTotal(); }  // 3) presentation

    void sendEmail() { /* SMTP code */ }                     // 4) notification
};
```

**Why it breaks (4 reasons to change):**
- Tax rule changes → edit `calculateTotal`
- Switch MySQL → MongoDB → edit `saveToDatabase`
- New print format → edit `printInvoice`
- New email provider → edit `sendEmail`

All in one file → high merge-conflict risk, hard to test, one bug can break everything.

### ✅ Fix: split by responsibility

```cpp
class Invoice {                                   // only invoice data + business rule
    string customer; double amount;
public:
    Invoice(string c, double a) : customer(c), amount(a) {}
    double total() const { return amount * 1.18; }
    const string& getCustomer() const { return customer; }
};

class InvoiceRepository {                         // persistence only
public:
    void save(const Invoice& inv) { /* DB code */ }
};

class InvoicePrinter {                            // presentation only
public:
    void print(const Invoice& inv) { cout << inv.getCustomer() << ": " << inv.total() << "\n"; }
};

class EmailService {                              // notification only
public:
    void send(const Invoice& inv) { /* SMTP */ }
};
```

### Rules & gotchas
- **SRP ≠ "a class has one method".** A class can have many methods, all serving **one cohesive purpose**.
- **Test to apply:** describe the class in one sentence **without using "and"**. "Invoice calculates totals **and** saves **and** prints" → violation.
- **Don't over-split.** 20 tiny classes for a 10-line feature = over-engineering (see section 8).
- SRP applies at **class, function, and module** level. A function doing validate + save + log is also a violation.
- Ties to **cohesion** (Part 1): SRP = high cohesion.

### Interview points
- "Reason to change" = **actor**, not "task". Be ready to say this.
- A **Facade / orchestrator** class may coordinate the pieces (`InvoiceService`) without violating SRP; its one job is *coordination*.

---

## 2. O: Open/Closed Principle (OCP)

### Definition
> *Software entities should be **open for extension** but **closed for modification**.*

Add new behavior by **adding new code** (new classes), not by **editing** tested, working code.

**Analogy:** A power strip. You plug in new devices (extension) without rewiring the strip (modification).

### ❌ Violation

```cpp
enum class ShapeType { CIRCLE, RECTANGLE };

struct Shape { ShapeType type; double a, b; };

class AreaCalculator {
public:
    double area(const Shape& s) {
        if (s.type == ShapeType::CIRCLE)         return 3.14 * s.a * s.a;
        else if (s.type == ShapeType::RECTANGLE) return s.a * s.b;
        // Adding TRIANGLE → must EDIT this function (and every other switch on type)
        return 0;
    }
};
```

**Why it breaks:** every new shape edits `AreaCalculator`. Risk of breaking existing cases; every `if/switch` over `type` scattered across the codebase must be found and updated.

### ✅ Fix: polymorphism (abstraction + new subclasses)

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14 * r * r; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area() const override { return w * h; }
};

// NEW requirement: Triangle → just ADD a class. Nothing else changes.
class Triangle : public Shape {
    double b, h;
public:
    Triangle(double b, double h) : b(b), h(h) {}
    double area() const override { return 0.5 * b * h; }
};

class AreaCalculator {                          // CLOSED for modification
public:
    double total(const vector<unique_ptr<Shape>>& shapes) const {
        double sum = 0;
        for (auto& s : shapes) sum += s->area();
        return sum;
    }
};
```

### Second example: Discount strategy (very common in interviews)

```cpp
class DiscountStrategy {
public:
    virtual double apply(double price) const = 0;
    virtual ~DiscountStrategy() = default;
};
class NoDiscount    : public DiscountStrategy { public: double apply(double p) const override { return p; } };
class FestiveOffer  : public DiscountStrategy { public: double apply(double p) const override { return p * 0.80; } };
class LoyaltyOffer  : public DiscountStrategy { public: double apply(double p) const override { return p * 0.90; } };
// New "BlackFriday" offer? Add a class. Checkout code untouched.
```
This is the **Strategy pattern** (Part 4+) and is the standard way to satisfy OCP.

### Rules & gotchas
- **How OCP is achieved:** abstraction (interface/abstract class) + polymorphism. Also via Strategy, Decorator, Template Method, Observer patterns.
- **Can't be 100% closed.** You must decide *which axis of change* to protect. Predict the likely variation (new payment types? new shapes?) and abstract **that**. Don't abstract everything.
- **Rule of thumb ("fool me once"):** write it simply first; when the **first** change request arrives, refactor to make that axis extensible.
- A **factory** somewhere still has a `switch` to create objects. That's acceptable; it's confined to one place (the composition root).

### Interview points
- OCP = **the goal**; polymorphism/abstraction = **the mechanism**.
- Red flag phrase: long `if-else` / `switch` on a **type code** → OCP violation.

---

## 3. L: Liskov Substitution Principle (LSP) 🔥 DEEP DIVE

### 3.1 Definition

**Informal**
> *Objects of a derived class must be usable **anywhere** the base class is expected, **without the program breaking or behaving incorrectly**.*

**Formal (Barbara Liskov, 1987)**
> *If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program.*

**Key insight:** LSP is about **behavior**, not syntax. Code can **compile perfectly** and still violate LSP. Inheritance must model **"behaves-as"**, not just "is-a" in English.

**Analogy:** A "duck" toy that needs batteries is-a duck (looks like one, quacks) but **cannot** be substituted for a real duck in a pond simulation. If your `Pond` code calls `duck.swim()` and the electronic duck throws "no water!", the substitution failed.

---

### 3.2 The classic violation: Rectangle / Square

In math, a square **is a** rectangle. In OOP behavior, it isn't.

```cpp
class Rectangle {
protected:
    int width, height;
public:
    virtual void setWidth(int w)  { width = w; }
    virtual void setHeight(int h) { height = h; }
    int area() const { return width * height; }
    virtual ~Rectangle() = default;
};

class Square : public Rectangle {
public:
    void setWidth(int w) override  { width = height = w; }   // keeps it square
    void setHeight(int h) override { width = height = h; }
};

// Client code written against Rectangle
void resizeAndCheck(Rectangle& r) {
    r.setWidth(5);
    r.setHeight(4);
    assert(r.area() == 20);       // ✅ true for Rectangle
}                                 // ❌ FAILS for Square (area = 16)
```

**Why it breaks:** The `Rectangle` **contract** says *width and height are independent*. `Square` **strengthens** the rule (they're linked), so a caller who relies on the base contract gets a surprise.

### ✅ Fixes
```cpp
// Fix A: Don't make Square inherit Rectangle. Use a common abstraction with no setters.
class Shape {
public:
    virtual int area() const = 0;
    virtual ~Shape() = default;
};
class Rectangle : public Shape {
    int w, h;
public:
    Rectangle(int w, int h) : w(w), h(h) {}
    int area() const override { return w * h; }
};
class Square : public Shape {
    int side;
public:
    Square(int s) : side(s) {}
    int area() const override { return side * side; }
};

// Fix B: make objects immutable, so no setters can violate the contract.
```

---

### 3.3 The formal LSP rules (memorize these)

LSP is enforced through **Design by Contract**. A contract = **preconditions** (what the caller must guarantee) + **postconditions** (what the method guarantees) + **invariants** (always true about the object).

#### A) Rules about the METHOD contract

| # | Rule | Meaning | Mnemonic |
|---|---|---|---|
| 1 | **Preconditions cannot be strengthened** in a subtype | Subclass can't demand *more* from the caller than the base did | "Don't be pickier than your parent" |
| 2 | **Postconditions cannot be weakened** in a subtype | Subclass must promise *at least* what the base promised | "Deliver at least as much as your parent" |
| 3 | **Invariants must be preserved** | Anything always-true in the base must stay true in the derived | "Don't break parent's guarantees" |
| 4 | **History constraint** | Subclass can't allow state changes the base forbade (e.g., base is immutable → subclass can't be mutable) | "Don't rewrite history" |

(Preconditions may be **weakened** and postconditions may be **strengthened**. That's allowed and good.)

#### B) Rules about SIGNATURES (type-level, enforced partly by the compiler)

| # | Rule | Meaning |
|---|---|---|
| 5 | **Contravariance of arguments** | Overriding method's parameter types may be **wider** (more general), never narrower |
| 6 | **Covariance of return types** | Overriding method's return type may be **narrower** (more specific), never wider |
| 7 | **Exception rule** | Subclass method must **not throw new exceptions** that the base method's contract doesn't allow (unless they're subtypes of exceptions the base already throws) |

**C++ note on rule 6:** C++ **supports covariant return types** for pointers/references in virtual overrides:
```cpp
class Animal { public: virtual Animal* clone() const { return new Animal(*this); } virtual ~Animal() = default; };
class Dog : public Animal {
public:
    Dog* clone() const override { return new Dog(*this); }   // ✅ covariant: Dog* is narrower than Animal*
};
```
C++ does **not** support contravariant parameters. A different parameter type = a **new overload**, not an override. (That's why `override` keyword is so valuable: it catches this.)

---

### 3.4 Each rule with a violation and fix

#### Rule 1: Preconditions must not be strengthened

```cpp
class PaymentProcessor {
public:
    // Contract: accepts any amount > 0
    virtual void pay(double amount) {
        if (amount <= 0) throw invalid_argument("amount must be > 0");
        cout << "Paid " << amount << "\n";
    }
    virtual ~PaymentProcessor() = default;
};

class PremiumProcessor : public PaymentProcessor {
public:
    void pay(double amount) override {
        if (amount < 1000) throw invalid_argument("min 1000");   // ❌ STRONGER precondition
        cout << "Premium paid " << amount << "\n";
    }
};

void checkout(PaymentProcessor& p) { p.pay(50); }   // fine for base, EXPLODES for PremiumProcessor
```
**Fix:** subclass must accept everything the base accepts (it may accept *more*). If it truly has a minimum, it isn't a valid substitute. Model it as a separate type or a separate interface.

#### Rule 2: Postconditions must not be weakened

```cpp
class FileReader {
public:
    // Contract: ALWAYS returns non-empty content or throws
    virtual string read(const string& path) {
        /* reads file */ return "content";
    }
    virtual ~FileReader() = default;
};

class LazyReader : public FileReader {
public:
    string read(const string& path) override {
        return "";                         // ❌ weakened: may return empty silently
    }
};
```
Callers of `FileReader::read` assume non-empty output. Returning empty breaks them.

#### Rule 3: Invariants must be preserved

```cpp
class BankAccount {
protected:
    double balance = 0;                    // INVARIANT: balance >= 0
public:
    virtual void withdraw(double amt) {
        if (amt > balance) throw runtime_error("insufficient");
        balance -= amt;
    }
    virtual ~BankAccount() = default;
};

class OverdraftAccount : public BankAccount {
public:
    void withdraw(double amt) override {
        balance -= amt;                    // ❌ balance can go negative → invariant broken
    }
};
```
**Fix:** if overdraft is a real feature, the base must not promise `balance >= 0`. Redefine the base contract (e.g., `balance >= -limit`) or don't inherit; use composition/strategy for overdraft rules.

#### Rule 4: History constraint

Base is **immutable** (e.g., `Point` with only getters). Derived adds `setX()`, letting state change in ways clients of the base never expected (e.g., someone used a `Point` as a map key, then it mutated). **Don't add mutators to a subtype of an immutable type.**

#### Rule 5/6: Signature variance in C++

```cpp
class Animal {};
class Dog : public Animal {};

class Owner {
public:
    virtual void adopt(Animal& a) { }
};
class DogOwner : public Owner {
public:
    void adopt(Dog& d) { }     // ❌ NOT an override (narrower param) → just hides/overloads.
                               //    With `override` keyword the compiler REJECTS this. Always write `override`.
};
```
Narrowing parameters would break substitution (`Owner& o = dogOwner; o.adopt(cat);` must be valid for the base contract).

#### Rule 7: Exception rule (as requested)

**Rule:** A derived method must not throw exceptions the base method's callers don't expect. It may throw **fewer** or **more specific** (subtype) exceptions.

```cpp
class DataSource {
public:
    // Contract: may throw IOError only
    virtual string fetch() { throw IOError("disk"); }   // (or returns data)
    virtual ~DataSource() = default;
};

class NetworkSource : public DataSource {
public:
    string fetch() override {
        throw AuthenticationError("token expired");     // ❌ NEW, unrelated exception type
    }
};

void load(DataSource& ds) {
    try { ds.fetch(); }
    catch (const IOError&) { /* handled */ }            // AuthenticationError escapes → crash
}
```

**Allowed:**
```cpp
class NetworkIOError : public IOError {};               // ✅ subtype of what base already throws
string fetch() override { throw NetworkIOError("timeout"); }   // caller's catch(IOError&) still works
```

**Extra C++ specifics**
- Dynamic exception specifications (`throw(IOError)`) are **removed in C++17**. Use `noexcept` instead.
- An overriding function **may not be less restrictive** on `noexcept`: if the base virtual is `noexcept`, the override **must also be `noexcept`**. (The compiler enforces this, so it's LSP rule 7 built into the language.)
- The reverse is fine: base can throw, derived can be `noexcept` (stronger promise, allowed).

```cpp
class Base { public: virtual void run() noexcept {} virtual ~Base() = default; };
class Derived : public Base { public: void run() override { throw 1; } };   // ❌ compile error: looser than base
```

---

### 3.5 More classic violations (recognize them in interviews)

#### Violation A: Bird / Penguin (`fly()` problem)

```cpp
class Bird { public: virtual void fly() { cout << "flying\n"; } virtual ~Bird() = default; };
class Sparrow : public Bird {};
class Penguin : public Bird {
public:
    void fly() override { throw logic_error("penguins can't fly"); }   // ❌
};

void migrate(vector<Bird*>& flock) { for (auto* b : flock) b->fly(); }   // crashes on Penguin
```

**Fix: segregate capabilities (LSP + ISP together)**
```cpp
class Bird { public: virtual void eat() = 0; virtual ~Bird() = default; };
class FlyingBird : public Bird { public: virtual void fly() = 0; };

class Sparrow : public FlyingBird { public: void eat() override {} void fly() override { cout << "flap\n"; } };
class Penguin : public Bird       { public: void eat() override {} void swim() { cout << "swim\n"; } };

void migrate(vector<FlyingBird*>& flock) { for (auto* b : flock) b->fly(); }   // only birds that CAN fly
```

#### Violation B: Empty / "do nothing" override

```cpp
class Document { public: virtual void save() { /* writes to disk */ } };
class ReadOnlyDocument : public Document {
public:
    void save() override { }        // ❌ silently ignores. Caller thinks it was saved.
};
```
An empty or no-op override that breaks the base's postcondition ("after `save()`, data is persisted") is a violation. Throwing is also a violation. **Redesign the hierarchy** (`Document` → `WritableDocument`).

#### Violation C: Type checking in client code (`dynamic_cast`/`typeid`)

```cpp
void process(Shape* s) {
    if (dynamic_cast<Square*>(s)) { /* special case */ }   // ❌ client had to "know" the subtype
    else { ... }
}
```
If callers must check the concrete type to behave correctly, substitution failed. **Smell:** `dynamic_cast`, `typeid`, or `instanceof`-style checks sprinkled across client code.

#### Violation D: Changing behavior semantics

```cpp
class Stack { public: virtual void push(int x); virtual int pop(); };        // LIFO
class WeirdStack : public Stack { public: int pop() override { /* returns oldest item */ } };  // ❌ FIFO
```
Same signature, different **meaning**. Compiler is happy, behavior is broken.

#### Violation E: Side effects the base didn't have

Derived `getBalance()` that also logs to a remote server, mutates a counter, or is slow/blocking when base was pure and fast. Callers assumed a cheap, side-effect-free call.

#### Violation F: `ElectricCar : Car` with `refuel()`

`Car::refuel()` exists; `ElectricCar::refuel()` throws/does nothing. Same pattern as Penguin. **Fix:** `Car` (drive) → `FuelCar` (refuel), `ElectricCar` (recharge), or use composition (`Engine` strategy).

---

### 3.6 Exceptions to LSP / when it's OK to "bend" (the nuance interviewers love)

LSP has **no exceptions in theory** (it's a correctness property), but there are legitimate situations where violations are **tolerated, restricted, or reframed**:

| Situation | Explanation |
|---|---|
| **Weakening preconditions** | ✅ *Allowed and good.* Derived accepts a **wider** input range than base. |
| **Strengthening postconditions** | ✅ *Allowed and good.* Derived gives **more** guarantees (e.g., returns sorted output when base said "any order"). |
| **Covariant return types** | ✅ *Allowed.* `Dog* clone()` overriding `Animal* clone()`. |
| **Throwing subtype exceptions** | ✅ *Allowed.* If it's a subtype of what base may throw. |
| **Adding new methods in the subclass** | ✅ *Allowed.* Extra behavior doesn't break base-typed clients (they never call it). |
| **Documented `NotSupported` optional operations** | ⚠️ *Contract-level exception.* If the **base contract explicitly says** "this operation may be unsupported" (like Java's `Collections.unmodifiableList().add()` throwing `UnsupportedOperationException`), then throwing is **within contract**. Better still to model separately (`ReadOnlyList` vs `MutableList`). |
| **Private / non-virtual inheritance** | ⚠️ LSP applies to **public** inheritance ("is-a"). With `private` inheritance (implemented-in-terms-of), the derived is *not* substitutable by design, so LSP is not the goal. |
| **Value semantics / slicing** | ⚠️ Passing by value slices the derived object. LSP is about substitution via **pointer/reference**. |
| **Test doubles (mocks/fakes)** | ⚠️ Mocks may behave differently on purpose, but a *good* mock still honors the interface contract, or tests give false confidence. |

**Golden rule to state in interviews:**
> *LSP violations are judged against the **contract** (documented preconditions/postconditions/invariants). If the base contract honestly allows "may be unsupported / may return empty", then the derived behavior is legal. The fix is usually to make the contract honest, or to redesign the hierarchy.*

---

### 3.7 Quick LSP detection checklist

Ask these when reviewing a class hierarchy:

- ☐ Does the subclass **throw `NotImplemented` / `NotSupported`** for an inherited method?
- ☐ Does it have an **empty override** for an inherited method?
- ☐ Does the client have to **check the concrete type** (`dynamic_cast`, `typeid`, `type` enum)?
- ☐ Does the subclass **add stricter validation** to inherited inputs?
- ☐ Does it **return less** / weaker results than the base promised?
- ☐ Does it **break an invariant** (negative balance, unsorted "sorted" list)?
- ☐ Does it **throw new exception types**?
- ☐ Would base-class **unit tests fail** if run against the subclass? *(Best practical test: **run the base class test-suite against every subclass.**)*

If any answer is yes → the hierarchy likely violates LSP.

### 3.8 How to fix LSP violations (toolbox)

1. **Redesign the hierarchy**: move the offending method to a narrower interface (Bird → FlyingBird).
2. **Prefer composition over inheritance** (ElectricCar *has-a* Engine; Square doesn't inherit Rectangle).
3. **Extract a common abstraction** with only what all subtypes truly share (`Shape::area`).
4. **Make types immutable** to remove setter-based contract clashes.
5. **Strengthen or clarify the base contract** so it honestly reflects what subtypes can do.
6. **Use ISP**: split fat interfaces so no class inherits methods it can't honor.

### 3.9 LSP interview points

- **"Is-a" in English ≠ "is-substitutable-for" in code.** Square/Rectangle, Penguin/Bird.
- LSP is the principle that **makes OCP possible**: OCP relies on polymorphism, which is only safe if subtypes are truly substitutable.
- **Definition in one line:** *"Subtypes must honor the base type's contract."*
- **Best detection signals:** `NotImplemented` exceptions, empty overrides, `dynamic_cast` in client code.
- **Design by contract** vocabulary (preconditions / postconditions / invariants) impresses interviewers.
- Mention the **C++ tools**: `override`, `final`, `noexcept` compatibility, covariant returns, virtual destructors, avoiding slicing.

---

## 4. I: Interface Segregation Principle (ISP)

### Definition
> *Clients should not be forced to depend on methods they do not use.* Prefer **many small, specific interfaces** over one big "fat" interface.

**Analogy:** A restaurant menu split into "Veg", "Non-veg", "Drinks" cards vs. one huge booklet. A vegetarian shouldn't have to carry the non-veg pages.

### ❌ Violation

```cpp
class IMachine {                      // fat interface
public:
    virtual void print() = 0;
    virtual void scan() = 0;
    virtual void fax() = 0;
    virtual ~IMachine() = default;
};

class BasicPrinter : public IMachine {
public:
    void print() override { cout << "printing\n"; }
    void scan() override { throw logic_error("not supported"); }   // ❌ forced to implement
    void fax()  override { throw logic_error("not supported"); }   // ❌ (also an LSP violation)
};
```

**Why it breaks:**
- `BasicPrinter` depends on `scan` and `fax` it never uses.
- Changing `fax()` signature forces recompiling/changing classes that never fax.
- Leads directly to **LSP violations** (NotSupported exceptions).

### ✅ Fix: role interfaces

```cpp
class IPrinter { public: virtual void print() = 0; virtual ~IPrinter() = default; };
class IScanner { public: virtual void scan()  = 0; virtual ~IScanner() = default; };
class IFax     { public: virtual void fax()   = 0; virtual ~IFax() = default; };

class BasicPrinter : public IPrinter {
public: void print() override { cout << "printing\n"; }
};

class AllInOne : public IPrinter, public IScanner, public IFax {      // multiple inheritance of interfaces = safe
public:
    void print() override {}
    void scan()  override {}
    void fax()   override {}
};

// Client needing only printing depends only on IPrinter
void printReport(IPrinter& p) { p.print(); }
```

### Rules & gotchas
- Split by **client needs** (roles), not by arbitrary size. A 5-method interface used by one client together is fine.
- **Symptom:** implementations with empty bodies or `throw NotSupported`.
- ISP and **SRP** are cousins: SRP is about a class's reasons to change; ISP is about an interface's client-facing surface.
- C++ multiple inheritance of **pure interfaces** has no diamond data problems, so it's the standard way to combine roles.
- Fat interfaces also cause **needless recompilation** in C++ (header dependencies).

### Interview points
- ISP violation almost always **causes** an LSP violation (forced NotSupported).
- Fix pattern: split into **role interfaces**; the concrete class implements only the roles it truly has.

---

## 5. D: Dependency Inversion Principle (DIP)

### Definition (two parts)
> 1. **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
> 2. **Abstractions should not depend on details. Details should depend on abstractions.**

- **High-level module** = business logic (`OrderService`, `NotificationManager`).
- **Low-level module** = implementation details (`MySQLDatabase`, `GmailSender`).

**Analogy:** A wall socket (abstraction). The lamp, TV, and phone charger all depend on the **socket standard**, not on the specific power plant behind it. You can change the power plant without changing any appliance.

### ❌ Violation

```cpp
class EmailSender {                    // low-level detail
public:
    void send(const string& msg) { cout << "Email: " << msg << "\n"; }
};

class NotificationService {            // high-level policy
    EmailSender sender;                // ❌ hard-coded concrete dependency
public:
    void notify(const string& msg) { sender.send(msg); }
};
```

**Why it breaks:**
- Want SMS/Push? Must **edit** `NotificationService` (also violates OCP).
- Can't unit test without sending a real email (no way to inject a fake).
- High-level logic is chained to a low-level detail.

### ✅ Fix: depend on an abstraction + inject it

```cpp
class INotifier {                                 // abstraction (owned by high-level module)
public:
    virtual void send(const string& msg) = 0;
    virtual ~INotifier() = default;
};

class EmailSender : public INotifier { public: void send(const string& m) override { cout << "Email: " << m << "\n"; } };
class SMSSender   : public INotifier { public: void send(const string& m) override { cout << "SMS: "   << m << "\n"; } };
class PushSender  : public INotifier { public: void send(const string& m) override { cout << "Push: "  << m << "\n"; } };

class NotificationService {
    INotifier& notifier;                           // depends on abstraction
public:
    explicit NotificationService(INotifier& n) : notifier(n) {}     // constructor injection
    void notify(const string& msg) { notifier.send(msg); }
};

int main() {
    SMSSender sms;
    NotificationService svc(sms);       // wiring happens at the composition root (main)
    svc.notify("Order shipped");
}

// Unit test: inject a fake
class FakeNotifier : public INotifier { public: vector<string> sent; void send(const string& m) override { sent.push_back(m); } };
```

### Dependency Inversion ≠ Dependency Injection ≠ IoC
| Term | What it is |
|---|---|
| **DIP** | The **principle**: depend on abstractions |
| **Dependency Injection (DI)** | A **technique** to give an object its dependencies from outside |
| **IoC (Inversion of Control)** | The broader idea: framework/caller controls flow/creation instead of your code |
| **IoC container** | Tool that automates DI (Spring, etc.; rarely used in C++, where you wire manually in `main`) |

### Types of DI (in C++)
```cpp
// 1) Constructor injection (BEST: required dependency, object always valid)
NotificationService(INotifier& n);

// 2) Setter injection (optional / changeable dependency)
void setNotifier(INotifier* n);

// 3) Method (parameter) injection (dependency used by one call only)
void notify(const string& msg, INotifier& n);
```

### Rules & gotchas
- **"Inversion"** = the source-code dependency direction is *inverted*: `NotificationService → INotifier ← EmailSender` (instead of `NotificationService → EmailSender`).
- The **interface belongs to the high-level module** (it defines what it *needs*), not the low-level one.
- Ownership: reference (`INotifier&`) = borrowed, caller owns. `unique_ptr<INotifier>` = service owns. `shared_ptr` = shared. Pick deliberately.
- **Don't inject everything.** Stable, simple things (`std::string`, `std::vector`, math functions) don't need abstractions.
- Object creation with `new` inside a business class is the **classic DIP smell**. Move creation to a factory/composition root.

### Interview points
- If a class `new`s its own collaborators → tightly coupled → DIP violation.
- DIP makes **testing** easy (mocks/fakes) and **OCP** possible.
- Be ready to distinguish **DIP vs DI vs IoC** (table above).

---

## 6. How the five principles connect

```
SRP  →  small focused classes (one reason to change)
ISP  →  small focused interfaces (only what clients need)
OCP  →  extend by adding classes behind abstractions
LSP  →  ...and those added classes must be truly substitutable
DIP  →  clients depend on the abstractions, and get implementations injected
```
- **OCP needs LSP** (polymorphism only works if subtypes are substitutable).
- **OCP needs DIP** (you can only extend if code depends on an abstraction).
- **ISP prevents LSP violations** (no forced "NotSupported").
- **SRP supports all of them** (small focused units are easier to abstract and substitute).

---

## 7. DRY, KISS, YAGNI (+ related rules)

### DRY: Don't Repeat Yourself
> *Every piece of **knowledge** should have a single, authoritative representation.*

```cpp
// ❌ Repeated logic
double priceForKids(double p)    { return p - p * 0.10; }
double priceForSeniors(double p) { return p - p * 0.10; }

// ✅ Single source of truth
double applyDiscount(double price, double pct) { return price - price * pct; }
```
- DRY is about duplicated **knowledge/business rules**, not just duplicated **text**. Two code blocks that look the same but change for *different reasons* should **not** be merged (accidental duplication).
- **Over-DRY trap:** forcing unrelated code into one shared abstraction couples things that should evolve independently.
- Magic numbers → named constants (`constexpr double GST = 0.18;`).

### KISS: Keep It Simple, Stupid
> *The simplest design that works is the best design.*
- Avoid clever tricks, deep inheritance trees, or patterns "just because".
- If a plain function solves it, don't build a class hierarchy.
- Readable > clever. The next developer (or you in 6 months) must understand it.

### YAGNI: You Aren't Gonna Need It
> *Don't build features or abstractions for hypothetical future needs.*
- Don't add a `PaymentFactory` when you have exactly one payment method and no requirement for more.
- Conflict with OCP? Resolve via: **build simple now; refactor to abstraction when the second variation actually arrives.**

### Law of Demeter (Principle of Least Knowledge)
> *Talk only to your immediate friends; don't reach through objects.*
```cpp
// ❌ Train wreck
customer.getWallet().getMoney().deduct(50);
// ✅ Tell, don't ask
customer.pay(50);
```
Reduces coupling; promotes encapsulation.

### Composition over Inheritance
Covered in Part 1 (section 12). Prefer it whenever the relationship isn't a strict, contract-honoring "is-a".

### Tell, Don't Ask
Instead of pulling data out of an object to decide what to do, **tell the object to do it** (`account.withdraw(x)` instead of `if (account.getBalance() >= x) account.setBalance(...)`).

---

## 8. Over-engineering: when NOT to apply SOLID

Interviewers reward **judgment**, not maximal abstraction.

| Situation | Approach |
|---|---|
| Small script / throwaway prototype | KISS. Skip layers. |
| One implementation, no foreseeable variants | Don't add an interface yet (YAGNI). |
| Stable code that never changes | No need to restructure. |
| Interview with 45 minutes | Apply SOLID to the **2-3 axes most likely to change** and say so aloud. |

**How to talk about it:** *"I'm abstracting payment methods because the requirements mention multiple providers. I'm keeping the notification logic simple since there's only one channel; I'd refactor if a second appears."*

---

## 9. Interview Q&A

**Q1. Explain SOLID briefly.**
S: one reason to change. O: extend without modifying. L: subtypes substitutable for base. I: no forced unused methods. D: depend on abstractions.

**Q2. Difference between SRP and cohesion?**
Cohesion is the *measure* (how related a class's parts are). SRP is the *rule* that drives you to high cohesion.

**Q3. How do you achieve OCP in C++?**
Abstract base class + virtual functions (or templates/Strategy). New behavior = new derived class.

**Q4. Give an LSP violation example.**
Rectangle/Square, or Bird/Penguin with `fly()`. Explain: the derived class breaks the base contract (invariant / postcondition), so client code misbehaves.

**Q5. Does LSP mean "derived must behave identically"?**
No. Derived can **extend** behavior, weaken preconditions, strengthen postconditions. It just must not **break** what the base promised.

**Q6. Rules of LSP?**
Preconditions can't be strengthened; postconditions can't be weakened; invariants preserved; history constraint; contravariant params; covariant returns; no new unexpected exceptions.

**Q7. How to detect an LSP violation?**
`NotImplemented`/`NotSupported` overrides, empty overrides, `dynamic_cast` in clients, base-class tests failing on subclasses.

**Q8. How are LSP and ISP related?**
Fat interfaces force classes to implement methods they can't honor, producing LSP violations. Splitting interfaces (ISP) removes the cause.

**Q9. How are OCP and LSP related?**
OCP depends on polymorphism; polymorphism is only safe if LSP holds.

**Q10. DIP vs DI vs IoC?**
DIP = principle, DI = technique, IoC = broader control-flow inversion concept.

**Q11. What's the problem with `new` inside a class?**
Hard-wired dependency → tight coupling → can't swap or mock → DIP/OCP violation.

**Q12. Can SOLID be overdone?**
Yes. Too many tiny classes/interfaces hurt readability. Apply where change is *likely*.

**Q13. Which C++ features help enforce LSP?**
`override`, `final`, covariant returns, `noexcept` compatibility checks, virtual destructors, pointers/references (avoid slicing), pure-abstract interfaces.

**Q14. How do you test for LSP practically?**
Write contract tests for the base interface and run them against **every** implementation.

**Q15. Is `Stack` inheriting `vector` an LSP problem?**
It's mainly a **wrong "is-a"**: a Stack is not substitutable for a vector (it shouldn't expose insert-at-index). Use composition.

---

## 10. One-Page Cheat Sheet

```
S  Single Responsibility  : 1 class = 1 reason to change (1 actor). Test: describe w/o "and".
                            Violation: Invoice that calculates+saves+prints+emails.
O  Open/Closed            : open to extension, closed to modification.
                            Smell: if/switch on type code. Fix: abstract base + new subclasses (Strategy).
L  Liskov Substitution    : subtype must be usable wherever base is, w/o breaking behavior.
      Contract rules      : preconditions NOT stronger | postconditions NOT weaker
                            invariants kept | history constraint
      Signature rules     : params contravariant | returns covariant (C++ ok) | no new exception types
                            (C++: override noexcept must not be looser)
      Classic violations  : Rectangle/Square, Bird/Penguin(fly), read-only doc save(), overdraft account
      Smells              : NotSupported throw, empty override, dynamic_cast in client
      Allowed             : weaker preconditions, stronger postconditions, covariant return,
                            extra methods, subtype exceptions
      Fix                 : redesign hierarchy, composition, extract common abstraction,
                            immutability, ISP split
I  Interface Segregation  : no client forced to depend on unused methods. Split fat interface into
                            role interfaces (IPrinter, IScanner, IFax).
D  Dependency Inversion   : high-level & low-level both depend on abstractions.
                            Smell: `new ConcreteClass` inside business class.
                            Fix: interface + constructor injection. (DIP principle, DI technique, IoC concept)

DRY  : one source of truth for each piece of knowledge (beware accidental duplication)
KISS : simplest thing that works
YAGNI: don't build for imaginary future needs
LoD  : don't chain a.getB().getC().do()
Balance: apply SOLID on likely axes of change; say your reasoning aloud in interviews.
```

---
