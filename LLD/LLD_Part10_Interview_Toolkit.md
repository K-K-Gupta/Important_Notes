# LLD Notes: Part 10. Interview Toolkit — The Final Revision Guide

---

## Table of Contents

1. The 45-minute interview timeline
2. The universal workflow (one diagram)
3. Requirements clarification checklist
4. Trigger-word → pattern lookup table (all patterns, all parts)
5. Trigger-word → relationship lookup table
6. The "say this out loud" phrasebook
7. Common mistakes that cost points (master list)
8. What interviewers are actually scoring
9. If you get stuck — recovery moves
10. Follow-up questions to expect (and how to answer in one line)
11. Quick reference: every pattern in one table
12. Quick reference: every problem in this course in one table
13. The night-before checklist
14. Final condensed cheat sheet (all parts, one page)

---

## 1. The 45-Minute Interview Timeline

```
0-5 min    : Clarify requirements. Ask questions out loud. Write down functional + non-functional reqs.
5-10 min   : Identify entities (nouns) and relationships. Say the ownership test for each pair.
10-15 min  : Identify variation points. Name the pattern for each, and WHY (not just "I'll use Strategy").
15-20 min  : Draw the class diagram (whiteboard/doc). Keep it to core classes; skip trivial getters/setters.
20-35 min  : Write the code. Start with interfaces/abstract classes, then concrete classes, then orchestration.
35-40 min  : Walk through one flow out loud (mentally run a sequence diagram) to catch gaps.
40-45 min  : Discuss extensions/trade-offs the interviewer asks about. Don't over-engineer upfront —
             this is WHERE extensibility gets demonstrated, not before.
```

**If you only have 30 minutes:** compress clarification to 2-3 minutes, skip the formal diagram, go straight from entities to code, narrating relationships and patterns as you write.

---

## 2. The Universal Workflow (One Diagram)

This is Part 3 §19 and Part 7 §1, repeated here because it is the single most important thing to have memorized:

```
1. CLARIFY requirements out loud (functional + non-functional)
2. Extract NOUNS -> candidate classes. Extract VERBS -> candidate methods.
3. Assign ONE responsibility per class (SRP)
4. For every pair of classes, decide the RELATIONSHIP using the ownership test:
     "If I delete the container, does the part still make sense?"
     Yes, independent  -> Association
     Yes, but grouped   -> Aggregation
     No, dies with it   -> Composition
     Only used briefly  -> Dependency
     "is-a" + contract   -> Inheritance
     implements contract -> Realization
5. For every place the requirements say "different kinds of X" or "varies by Y" ->
     introduce an interface + name the PATTERN (usually Strategy; sometimes Factory/Observer/State)
6. Add multiplicity. Draw the class diagram.
7. Walk ONE scenario through a sequence diagram to validate methods exist.
8. Write the code: interfaces first, then concrete classes, then the orchestrating class last.
9. State your SOLID reasoning and trade-offs OUT LOUD as you go, not just at the end.
10. Answer extension questions by reaching for "introduce an interface + inject it" (Strategy + DIP)
    almost every time — it is the single most reusable answer in this entire course.
```

---

## 3. Requirements Clarification Checklist

Ask a version of these for **any** problem, before writing a single class:

| Category | Questions |
|---|---|
| **Scope** | What's in scope vs out of scope? (e.g., "should I handle payments, or just the booking flow?") |
| **Actors** | Who are the users/roles? (admin vs customer vs system) |
| **Scale** | Rough numbers — does this need to be thread-safe / highly concurrent, or is single-threaded fine for this discussion? |
| **Variability** | What parts of this are likely to have "multiple kinds of X" — this previews your pattern choices |
| **Core flow** | What is the ONE primary user journey I should design around first? |
| **Edge cases** | What happens on failure (payment declines, resource unavailable, concurrent conflict)? |

**Don't over-ask.** 3-5 sharp questions, then start designing. Endless clarification before writing anything reads as stalling, not thoroughness.

---

## 4. Trigger-Word → Pattern Lookup Table (All Patterns, All Parts)

This is the single most useful table in this toolkit — when a requirement uses one of these phrasings, it's signaling a specific pattern.

| Phrase in the requirements | Pattern | Part |
|---|---|---|
| "exactly one X coordinates everything" | Singleton | 4 |
| "different ways to do X, interchangeable" | Strategy | 6A |
| "notify / subscribe / listeners / react to changes" | Observer | 6A |
| "behaves differently depending on its mode/status" | State | 6A |
| "undo / redo / queue the action / log requests" | Command | 6A |
| "same steps every time, a few details vary" | Template Method | 6B |
| "passes through multiple handlers/levels/approvers" | Chain of Responsibility | 6B |
| "traverse without exposing internal structure" | Iterator | 6B |
| "many objects need to coordinate with each other" | Mediator | 6B |
| "save/restore state, checkpoints" | Memento | 6B |
| "new operations on a fixed/stable set of types" | Visitor | 6B |
| "integrate a third-party/legacy API with a different interface" | Adapter | 5 |
| "supports multiple X across multiple Y" (two dimensions) | Bridge | 5 |
| "folders and files / menus and submenus / org hierarchy" (a tree) | Composite | 5 |
| "optional add-ons / toppings / wrap with extra behavior" | Decorator | 5 |
| "one simple entry point to a complex subsystem" | Facade | 5 |
| "thousands/millions of similar objects, memory matters" | Flyweight | 5 |
| "lazy-load / access control / caching / remote access" | Proxy | 5 |
| "object is complex to build, many optional fields" | Builder | 4 |
| "which class to instantiate should be deferred to a subclass" | Factory Method | 4 |
| "create matching SETS of related objects" | Abstract Factory | 4 |
| "clone/copy an existing configured object" | Prototype | 4 |
| "expensive objects, reused across requests" | Object Pool | 4 |
| "prevent double-booking / race condition / concurrent conflict" | **Not a pattern — mutex/lock** | 9 |

> ⚡ **If you remember only one row:** *"Different ways to do X, interchangeable"* → **Strategy**. It is by far the most common pattern across every problem in Parts 7-9 (pricing, payment, scheduling, split calculation, eviction policy). When in doubt, ask whether the requirement describes an interchangeable algorithm — if yes, it's Strategy.

---

## 5. Trigger-Word → Relationship Lookup Table

| Signal | Relationship | Part |
|---|---|---|
| "X has a list of Y, and Y can exist without X" | Aggregation | 3 |
| "X owns Y; Y has no meaning without X" | Composition | 3 |
| "X temporarily uses Y (parameter/local variable only)" | Dependency | 3 |
| "X is a kind of Y" (and X honors Y's full contract) | Inheritance | 3 |
| "X promises to do what interface Y says" | Realization | 3 |
| "X keeps a reference to Y, both have independent lifetimes" | Association | 3 |

**The one test that covers all three ownership relationships (Part 3 §12, reused constantly in Parts 7-9):**
> *"If I delete the container, does the contained thing still make sense?"*
> Yes, fully independent → **Association**. Yes, but they're grouped → **Aggregation**. No, it dies too → **Composition**.

---

## 6. The "Say This Out Loud" Phrasebook

Interviewers score **reasoning spoken aloud**, not just correct code. These are ready-made sentences for the moments that matter most:

**When choosing a pattern:**
> *"The requirements say [X varies / multiple kinds of Y], so I'll extract that behind an interface — that's a [Strategy/Factory/Observer] — which keeps [class] closed for modification when a new [variant] is added."*

**When choosing a relationship:**
> *"[A] owns [B] — if I delete the [A], the [B] has no reason to exist — so that's composition, and I'll hold it as [a member by value / a unique_ptr]."*

**When explicitly NOT using a pattern (shows restraint, Part 2 §8 / Part 8 §13):**
> *"There's no variation point here yet, so I'm keeping this as a plain [class/algorithm] rather than introducing an interface — I'd refactor to [pattern] if a second [variant] actually showed up."*

**When recognizing a concurrency problem instead of a design-pattern problem (Part 9 §9):**
> *"This isn't really a structural pattern question — two threads can both pass a check before either commits, so I need the check-and-commit to be one atomic critical section, guarded by a mutex."*

**When asked to extend the design:**
> *"I'd introduce an interface for [the varying part] and inject an implementation — that's the same Strategy/DIP move I used for [earlier thing in this design], so [orchestrating class] doesn't need to change."*

**When you catch your own earlier mistake (own it, don't backtrack silently — Part 6A/6B/8 modeled this explicitly):**
> *"Actually, I realize [X] should be [Y] instead — let me fix that before I continue, because [reason]."*

**When wrapping up:**
> *"To summarize: [class] is the core orchestrator, [pattern] handles the variation in [area], and the main trade-off I made was [X vs Y] — I'd reconsider that if [scale/requirement changed]."*

---

## 7. Common Mistakes That Cost Points (Master List)

Pulled from the "Common Mistakes" sections across every part — the ones that recur most:

| Mistake | Fix | Where explained |
|---|---|---|
| Giant `if/switch` on a type field scattered across methods | Polymorphism + Strategy/State | Part 2 (OCP), Part 6A |
| God class doing creation + validation + persistence + presentation | Split by single responsibility | Part 2 (SRP) |
| `new`-ing concrete dependencies inside business classes | Constructor-inject an interface | Part 2 (DIP) |
| Forgetting a virtual destructor on a polymorphic base | Always `virtual ~Base() = default;` | Part 1 §5 |
| Object slicing (assigning derived to base by value) | Use pointers/references, never pass polymorphic types by value | Part 1 §9 |
| Diamond drawn on the wrong side of a UML relationship | Diamond always sits on the **whole/owner** | Part 3 |
| Confusing aggregation and composition | Apply the "delete the container" test explicitly | Part 3 §12-14 |
| Using inheritance for "has-a" | Switch to composition | Part 1 §12 |
| `NotImplemented`/empty-override in a subclass | LSP violation — redesign the hierarchy or use composition | Part 2 §3 |
| A fat interface forcing unrelated methods on implementers | Split into role interfaces (ISP) | Part 2 §4 |
| Forgetting `override` keyword | Always write it — lets the compiler catch signature mistakes | Part 1 §10 |
| ID collisions across nested containers (e.g., per-floor spot IDs) | Use a single global counter/namespace | Part 7 |
| `make_unique<class X>()` inline inside another class, X not forward-declared at file scope | Forward-declare all mutually-referencing classes up top; define method bodies out-of-line | Part 8 (Elevator bug #1) |
| Conflating "transition" with "execute" in a State pattern | Separate `setState()` from an explicit driver loop (`step()`/`runUntilIdle()`) | Part 8 (Elevator bug #2) |
| Check-then-act race condition under concurrency | Make check + commit one atomic critical section under a single lock | Part 9 (BookMyShow) |
| Forcing a pattern onto a one-off algorithm just to "use a pattern" | Name it as plain logic; patterns are for genuine variation points (YAGNI) | Part 2 §8, Part 8 §13 |
| Claiming an algorithm is optimal without justification (e.g., debt simplification) | State the actual guarantee and the trade-off honestly | Part 8 §16 |

---

## 8. What Interviewers Are Actually Scoring

Across all the worked examples in Parts 7-9, the recurring evaluation axes are:

1. **Can you turn ambiguous requirements into concrete entities?** (Not "do you know all 23 patterns," but can you map a vague prompt to classes.)
2. **Do you apply SOLID *because the requirements justify it*, not by rote?** (Part 2 §8 and Part 8 §13's restraint moments are exactly this.)
3. **Is your code actually correct, not just pattern-shaped?** (Every code example in Parts 7-9 was compiled and tested — several had real bugs caught only by actually running them. Interviewers notice when code "looks right" but has an off-by-one or a race condition.)
4. **Can you recognize when a problem ISN'T a design-pattern problem?** (Seat locking is concurrency control, not a GoF pattern — Part 9 §9 makes this explicit.)
5. **Do you narrate your reasoning, or silently produce a diagram?** (The phrasebook in section 6 exists because spoken reasoning is scored separately from the artifact.)
6. **How do you respond to being challenged or finding your own bug?** (Calmly fix it and explain why — exactly how Part 8's two Elevator bugs were handled in these notes: found, explained, fixed, moved on.)

---

## 9. If You Get Stuck — Recovery Moves

| Situation | What to do |
|---|---|
| Can't decide between two patterns | Say both out loud with their trade-off, pick one, and note you'd reconsider if a specific condition changes — see the phrasebook |
| Not sure if something should be composition or aggregation | Apply the ownership test explicitly, out loud, even if you're unsure — the reasoning matters more than instant certainty |
| Realize your class diagram is missing a relationship once you start coding | Say so, fix the diagram, continue — this is normal design iteration, not a failure |
| Can't remember exact UML notation | Describe it in words ("composition, so I'll draw a filled diamond on the owner's side") — the interviewer cares about the concept, not pixel-perfect notation |
| Running low on time with incomplete code | Say what's left and how you'd structure it, rather than silently rushing sloppy code — a clear plan for the remainder is worth more than broken code |
| Interviewer challenges a design choice | Treat it as a real question, not an attack — explain your reasoning, and if they have a point, adjust rather than defend a weaker choice out of ego |

---

## 10. Follow-Up Questions to Expect (and How to Answer in One Line)

These recur across almost every problem in this course — the one-line answer is usually enough to start, then expand if asked:

| Follow-up | One-line answer |
|---|---|
| "How would you make this thread-safe?" | "Identify the critical section where shared state is read-then-written, and guard it with a single mutex — coarse-grained first, fine-grained only if profiling shows contention." |
| "How would you persist this to a database?" | "Introduce a Repository interface that the orchestrating class depends on, and inject a concrete implementation — same DIP move as everywhere else in this design." |
| "How would this scale to multiple servers?" | "Anything in-process (a mutex, an in-memory Singleton) needs a distributed equivalent — a distributed lock, or a shared cache/datastore, behind the same interface." |
| "How would you add [new variant of an existing Strategy]?" | "Add one new class implementing the existing interface — nothing else in the design changes." |
| "What's the time/space complexity of X?" | Know this cold for anything you designed as a data structure (LRU Cache, section 6 of Part 9) — don't guess live. |
| "What would you change if requirements doubled in scale?" | Name the specific bottleneck (a linear scan, a single mutex, an in-memory structure) and the specific fix — vague "I'd optimize it" answers are weak. |

---

## 11. Quick Reference: Every Pattern in One Table

| Pattern | Part | One-line intent | Example from this course |
|---|---|---|---|
| Singleton | 4 | Exactly one instance | ParkingLot, ElevatorController |
| Simple Factory | 4 | Centralize creation by type | (introductory, not GoF) |
| Factory Method | 4 | Subclass decides which class to instantiate | Logistics/Transport |
| Abstract Factory | 4 | Create a family of related objects | Cross-platform GUI |
| Builder | 4 | Step-by-step construction of a complex object | Pizza, Computer, HTTP request |
| Prototype | 4 | Clone an existing configured object | Shape duplication, game entities |
| Object Pool | 4 | Reuse expensive objects | Connection pool |
| Adapter | 5 | Convert an incompatible interface | Payment gateway SDK wrapping |
| Bridge | 5 | Decouple abstraction from implementation (2 dimensions) | Shape × Color |
| Composite | 5 | Uniform treatment of leaf and group in a tree | File system |
| Decorator | 5 | Add behavior dynamically by wrapping | Coffee toppings |
| Facade | 5 | Simplify a complex subsystem | Home theater, order placement |
| Flyweight | 5 | Share immutable state to save memory | Forest of trees |
| Proxy | 5 | Control access to an object | Lazy image loading |
| Strategy | 6A | Swap an algorithm | **Pricing/payment/split/eviction — everywhere in Parts 7-9** |
| Observer | 6A | Broadcast state changes | Weather station, display panels |
| State | 6A | Behavior follows internal mode | Vending machine, Elevator |
| Command | 6A | Wrap a request as an object | Remote control, undo/redo |
| Template Method | 6B | Fixed skeleton, varying steps | Caffeine beverage |
| Chain of Responsibility | 6B | Pass along handlers until one handles it | HTTP middleware |
| Iterator | 6B | Traverse without exposing internals | Custom playlist |
| Mediator | 6B | Centralize many-to-many communication | Dialog box coordination |
| Memento | 6B | Snapshot/restore state | Text editor undo |
| Visitor | 6B | New operations on a stable hierarchy | Shape area/export |

---

## 12. Quick Reference: Every Problem in This Course in One Table

| Problem | Part | Core patterns used | The one tricky bit |
|---|---|---|---|
| Parking Lot | 7 | Singleton, Strategy, Observer | Spot IDs must be unique across floors, not per-floor |
| Elevator System | 8 | State, Strategy, Observer, Singleton | Separating state *transition* from state *execution* to avoid broken simulation runs |
| Splitwise | 8 | Strategy (split types only) | Debt simplification is a plain greedy algorithm, not a pattern — and isn't provably optimal |
| LRU Cache | 9 | Strategy (for pluggable eviction) | O(1) requires hash map + doubly linked list together, not either alone |
| BookMyShow | 9 | Strategy (pricing/payment) | Seat locking is concurrency control (mutex), not a design pattern |

---

## 13. The Night-Before Checklist

- [ ] Can you state the four pillars of OOP and give a one-line example of each, cold? (Part 1)
- [ ] Can you explain LSP's exception rule and give a Rectangle/Square-style violation from memory? (Part 2)
- [ ] Can you draw (or describe) all six UML relationships and when to use each? (Part 3)
- [ ] Can you explain Meyers' Singleton and why it's thread-safe since C++11? (Part 4)
- [ ] Can you explain the difference between Decorator and Proxy, and Composite and Decorator? (Part 5)
- [ ] Can you explain State vs Strategy vs Bridge — same UML shape, different intent? (Part 6A)
- [ ] Can you explain double dispatch and why Visitor needs it? (Part 6B)
- [ ] Can you walk through the Parking Lot design end-to-end from memory, including why Strategy fits pricing/payment? (Part 7)
- [ ] Can you explain the two bugs from building the Elevator example, and why each happened? (Part 8)
- [ ] Can you explain why LRU Cache needs hash map + doubly linked list together? (Part 9)
- [ ] Can you explain why seat-locking in BookMyShow is a concurrency problem, not a pattern problem? (Part 9)
- [ ] Do you have 3-5 clarifying questions ready to ask for ANY unseen problem? (Section 3, this part)

If you can do all twelve without opening these notes, you're ready.

---

## 14. Final Condensed Cheat Sheet (All Parts, One Page)

```
WORKFLOW: clarify -> entities -> relationships (ownership test) -> variation points -> patterns ->
          UML -> code (interfaces first) -> narrate SOLID reasoning -> extensions

RELATIONSHIPS (Part 3): "delete the container, does the part still make sense?"
  Yes, independent -> Association | Yes, grouped -> Aggregation | No -> Composition
  Brief use only -> Dependency | is-a + honors contract -> Inheritance | implements -> Realization

SOLID (Part 2):
  S: one reason to change, one actor        O: extend via new classes, don't edit existing
  L: subtype honors base's FULL contract     I: small role interfaces, no forced unused methods
  D: depend on abstractions, inject concretes

PATTERN RECOGNITION (the ONE row to remember):
  "different/interchangeable ways to do X" -> STRATEGY (by far the most common pattern in real problems)
  one instance -> Singleton | notify on change -> Observer | mode-dependent behavior -> State
  undo/queue -> Command | fixed steps, some vary -> Template Method | handler chain -> CoR
  traverse hidden internals -> Iterator | many-to-many hub -> Mediator | save/restore -> Memento
  new ops, stable types -> Visitor | wrong interface -> Adapter | 2 dimensions -> Bridge
  tree, uniform -> Composite | wrap to add behavior -> Decorator | simplify subsystem -> Facade
  massive similar objects -> Flyweight | control access -> Proxy | complex multi-step build -> Builder
  subclass picks class -> Factory Method | family of products -> Abstract Factory | clone -> Prototype
  CONCURRENCY (check-then-act race) -> NOT a pattern -> mutex-guarded atomic critical section

C++ ESSENTIALS: virtual destructor on every polymorphic base | always write `override` |
  avoid object slicing (ptr/ref, not by-value) | Rule of 0/3/5 | unique_ptr = exclusive ownership
  (composition), shared_ptr = shared ownership, raw ptr/ref = non-owning (association)
  Forward-declare mutually-referencing classes at FILE SCOPE; define bodies out-of-line after.

BIGGEST RECURRING LESSONS FROM BUILDING REAL EXAMPLES (Parts 7-9):
  - Compile and run your logic mentally/on paper; "pattern-shaped" isn't the same as "correct"
    (real bugs were caught only by actually testing: ID collisions, state-machine stepping, races)
  - Not every design decision is a GoF pattern — recognize concurrency control, plain algorithms,
    and genuine "no pattern needed yet" moments (YAGNI) as valid, confident answers
  - Say your reasoning OUT LOUD at every step — it's scored separately from the final artifact
  - When you catch your own mistake mid-design, name it, fix it, move on — don't silently patch
    or defend a weaker earlier choice

INTERVIEW TIMELINE: 5 min clarify | 10 min entities+relationships | 10 min patterns+diagram |
  20 min code (interfaces -> concretes -> orchestrator) | 5-10 min walk a flow + discuss extensions
```

---
