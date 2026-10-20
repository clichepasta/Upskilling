# Class Relationships & UML — Low-Level Design Interview Prep

> **Goal:** Read and draw a clear UML class diagram, explain association, aggregation, and composition, and choose relationships that communicate real ownership and lifecycle rules.

## 1. Why Model Class Relationships?

A class diagram describes the static structure of a design: its classes, responsibilities, and relationships. It helps an interviewer understand not only which classes exist, but also how they collaborate and who owns what.

Use a diagram to make important design decisions visible—not to show every field, getter, and implementation detail. Start with the domain concepts and the relationships that affect behavior, then add multiplicities and navigability when they clarify requirements.

## 2. Relationship Types at a Glance

| Relationship | Meaning | UML notation | Lifecycle implication |
|---|---|---|---|
| Association | Objects are connected or collaborate | Solid line | None by itself |
| Aggregation | A whole groups parts that can exist independently | Solid line with a hollow diamond at the whole | Part lifecycle is independent |
| Composition | A whole owns parts whose lifecycle belongs to it | Solid line with a filled diamond at the owner | Part is owned by one composite at a time; deleting the whole removes its parts from that model |

Aggregation and composition are both whole-part relationships, but they communicate different ownership semantics. A plain association does not imply ownership.

## 3. Association

An **association** means instances of two classes are linked or communicate. It does not, on its own, say that one object owns the other or controls its lifetime.

### Example: Passenger and Flight

A passenger books or is associated with a flight. The passenger and flight have lifecycles independent of that booking relationship.

```text
Passenger "0..*" ───────── "0..*" Flight
                  books
```

The multiplicities describe the allowed number of linked instances at each end: a passenger may be associated with zero or more flights, and a flight may be associated with zero or more passengers.

If the relationship has its own data or behavior—such as seat assignment, booking time, or booking status—model it as an association class or an ordinary domain class:

```text
Passenger "1" ───── "0..*" Booking "0..*" ───── "1" Flight
```

In many application models, `Booking` is the clearest representation because it carries its own identity and lifecycle.

**Use association when:** Objects know about, reference, or interact with one another, but there is no meaningful whole-part ownership rule.

## 4. Aggregation

**Shared aggregation** is a whole-part relationship in which the part can exist independently of the whole. UML draws a hollow diamond on the whole side.

### Example: Team and Player

```text
Team "0..*" ◇──────── "0..*" Player
```

A player can exist without a particular team and may move between teams over time. The team groups players, but does not own their existence.

**Important:** UML defines shared aggregation loosely, and the hollow diamond is used inconsistently in practice. If the distinction adds no useful meaning, use a plain association instead. Do not use aggregation merely because one class stores a collection of another class.

**Use aggregation when:** The whole-part grouping matters to the domain, but parts are independently created, retained, or shared.

## 5. Composition

**Composition** is strong whole-part ownership. UML draws a filled diamond at the composite (owner) side. A part belongs to at most one composite at a time, and its lifecycle is governed by that composite in the model.

### Example: Order and OrderLine

```text
Order "1" ◆──────── "1..*" OrderLine
```

An `OrderLine` exists as part of an order. If an order is deleted from the domain, its order lines are removed with it. An order line should not be shared between different orders.

```java
public final class Order {
    private final List<OrderLine> lines = new ArrayList<>();

    public void addLine(Product product, int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("quantity must be positive");
        }
        lines.add(new OrderLine(product, quantity));
    }

    public List<OrderLine> lines() {
        return List.copyOf(lines);
    }
}

public final class OrderLine {
    private final Product product;
    private final int quantity;

    OrderLine(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}
```

The `Order` creates and controls its lines and exposes a read-only view. The referenced `Product` is not composed into the order: it exists independently and can appear on many order lines.

**Use composition when:** The part is conceptually owned, is not shared between composites, and should not outlive its owner in the domain model.

**Language caveat:** Composition is a domain/modeling constraint, not a guarantee provided by Java's garbage collector. A Java object can remain reachable through another reference after its owner is discarded. Enforce ownership through constructors, visibility, APIs, persistence rules, and domain operations.

## 6. Compare the Three Relationships

| Question | Association | Aggregation | Composition |
|---|---|---|---|
| Is it a whole-part relationship? | Not necessarily | Yes | Yes |
| Does the part have an independent lifecycle? | Not specified | Yes | Generally no, within the domain model |
| Can the part be shared between wholes? | Not specified | Often possible | No; at most one composite at a time |
| UML symbol | Solid line | Hollow diamond at whole | Filled diamond at owner |
| Example | Passenger—Flight | Team—Player | Order—OrderLine |

When unsure, use a plain association and explain the actual lifecycle rules. Do not infer composition just from a field, collection, or database foreign key.

## 7. UML Class Diagram Essentials

A UML class is commonly shown as a rectangle with up to three compartments:

```text
+---------------------------+
| Order                     |  Class name
+---------------------------+
| - id: OrderId             |  Attributes
| - status: OrderStatus     |
+---------------------------+
| + addLine(Product, int)   |  Operations
| + submit(): void          |
+---------------------------+
```

Common visibility markers:

| Marker | Visibility |
|---|---|
| `+` | Public |
| `-` | Private |
| `#` | Protected |
| `~` | Package |

### Multiplicity

Multiplicity appears near an association end and describes how many instances at that end can be linked to one instance at the opposite end.

| Notation | Meaning |
|---|---|
| `1` | Exactly one |
| `0..1` | Optional; zero or one |
| `*` or `0..*` | Zero or more |
| `1..*` | One or more |
| `m..n` | Between `m` and `n`, inclusive |

Read multiplicities carefully from both ends. In `Order "1" ◆── "1..*" OrderLine`, every line belongs to one order, and each order has at least one line in this model. If draft orders may be empty, use `0..*` instead.

### Other Useful Class-Diagram Relationships

```text
Inheritance:       EmailNotification ─────▷ Notification
                   (solid line, hollow triangle points to the base)

Realization:       EmailSender - - - - -▷ MessageSender
                   (dashed line, hollow triangle points to the interface)

Dependency:        CheckoutService - - - -> PaymentProcessor
                   (dashed arrow points to the thing depended on)
```

Inheritance/generalization means “is a”; realization means a class implements an interface. A dependency indicates temporary use, such as a method parameter or local variable, and is weaker than a stored association.

## 8. How to Draw a Useful LLD Class Diagram

1. **Clarify scope:** Agree on the core use cases and what the diagram needs to explain.
2. **Find domain concepts:** Identify entities, value objects, services, and important interfaces.
3. **Assign responsibilities:** Write only key state and operations that help explain the design.
4. **Connect collaborators:** Use associations for links and dependencies for temporary use.
5. **Model ownership:** Choose aggregation or composition only when the lifecycle distinction matters.
6. **Add multiplicities:** Show optionality and cardinality required by the domain.
7. **Check consistency:** Ensure the diagram agrees with the API, invariants, and example flows.
8. **Keep it readable:** Omit implementation noise; split a large design into focused diagrams.

## 9. Common Interview Questions

### “What is the difference between association and aggregation?”

Association says objects are related or collaborate. Aggregation says there is a whole-part grouping and the part can still exist independently of that whole. A hollow diamond marks the whole side in UML, although many teams prefer a plain association when aggregation semantics are not useful.

### “What is the difference between aggregation and composition?”

Both model whole-part relationships. In aggregation, the part has an independent lifecycle. In composition, the whole owns the part: a part belongs to at most one composite at a time and its lifecycle is tied to that composite in the model.

### “A class has a `List<OtherClass>`. Is that composition?”

Not automatically. A collection only shows that one object refers to multiple objects. Composition requires ownership and lifecycle semantics; a team’s player list, for instance, does not mean the team owns the players.

### “Can a composed object be shared?”

Not as a part of multiple composites at the same time under the usual UML composition semantics. If the object is shared or independently managed, use an association or aggregation as appropriate.

### “Does composition mean Java destroys the part when the owner is garbage-collected?”

No. UML communicates a modeling constraint; garbage collection is based on reachability. Another reference can keep a Java object alive. The application must enforce its domain ownership rules.

### “When should I show multiplicity?”

Show it when cardinality or optionality affects the design—for example, an order requiring at least one line or a user having at most one active profile. Do not guess; confirm the domain rule.

## 10. Common Mistakes

- Treating every object reference or collection as composition.
- Confusing a UML relationship with a database foreign key or garbage-collection behavior.
- Putting a diamond on the part end instead of the whole/owner end.
- Drawing composition when parts are shared or independently managed.
- Adding hollow diamonds even when plain association communicates the design better.
- Omitting multiplicities that encode important requirements.
- Using inheritance when the relationship is “has-a” rather than “is-a.”
- Making a diagram so detailed that the core responsibilities and ownership are hard to see.

## 11. A Concise Interview Answer

> “Association means two objects are related or collaborate, with no ownership implied. Aggregation is a weak whole-part relationship: the part can live independently, shown by a hollow diamond on the whole. Composition is strong ownership: a part belongs to at most one composite at a time and its lifecycle is tied to that owner in the model, shown by a filled diamond. I choose based on domain lifecycle rules—not just because a class has a reference or collection—and add multiplicities to make cardinality explicit.”

## 12. One-Minute Revision

| Relationship | Ask |
|---|---|
| Association | Are the objects connected or collaborating? |
| Aggregation | Is there meaningful grouping, while the part remains independent? |
| Composition | Does the whole own the part and govern its lifecycle? |
| Multiplicity | How many linked instances are allowed at each end? |
| Dependency | Is this only a temporary use rather than a lasting link? |
