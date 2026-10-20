# OOP Fundamentals — Low-Level Design Interview Prep

> **Goal:** Explain the four pillars of object-oriented programming, demonstrate them in code, and choose the right trade-offs when modeling a design.

## 1. What Is Object-Oriented Programming?

Object-oriented programming organizes a program around **objects**: units that combine state (data) with behavior (operations on that data). A class defines the shape and behavior of its instances.

Good object-oriented design is not about making every noun a class. It is about assigning clear responsibilities, protecting invariants, and making collaboration between parts of a system understandable.

## 2. The Four Pillars at a Glance

| Pillar | Core idea | Main benefit |
|---|---|---|
| Encapsulation | Keep state and the operations that protect it together; control access to internal state | Preserves invariants and limits unintended changes |
| Abstraction | Expose the essential behavior through a simple contract while hiding implementation details | Reduces what a caller needs to know |
| Inheritance | Derive a specialized type from a more general type | Reuses and specializes behavior when the subtype truly is substitutable |
| Polymorphism | Use one contract to work with different implementations | Lets behavior vary without branching on concrete types |

These concepts support one another, but they are not interchangeable. For example, an interface is one way to express abstraction; using interface implementations polymorphically is a separate design choice.

## 3. Encapsulation

**Encapsulation** groups data with the methods that manage it and prevents callers from putting an object into an invalid state.

### Example: Protecting a Bank Account

```java
public final class BankAccount {
    private final String accountId;
    private long balanceInCents;

    public BankAccount(String accountId, long openingBalanceInCents) {
        if (accountId == null || accountId.isBlank()) {
            throw new IllegalArgumentException("accountId is required");
        }
        if (openingBalanceInCents < 0) {
            throw new IllegalArgumentException("opening balance cannot be negative");
        }
        this.accountId = accountId;
        this.balanceInCents = openingBalanceInCents;
    }

    public void deposit(long amountInCents) {
        if (amountInCents <= 0) {
            throw new IllegalArgumentException("deposit must be positive");
        }
        balanceInCents += amountInCents;
    }

    public void withdraw(long amountInCents) {
        if (amountInCents <= 0) {
            throw new IllegalArgumentException("withdrawal must be positive");
        }
        if (amountInCents > balanceInCents) {
            throw new IllegalStateException("insufficient funds");
        }
        balanceInCents -= amountInCents;
    }

    public long getBalanceInCents() {
        return balanceInCents;
    }
}
```

`balanceInCents` is private, and callers cannot assign it directly. Every change goes through operations that enforce the account's rules.

**Interview point:** Encapsulation is more than adding `private` fields and generating getters/setters. A public setter for every field can expose the representation without protecting any invariant.

**Use it when:** An object has rules that must remain true across updates—for example, a reservation cannot have a negative seat count or a payment cannot transition from refunded back to captured.

## 4. Abstraction

**Abstraction** presents a useful contract while hiding details that callers do not need. In Java, interfaces and abstract classes can define such contracts.

### Example: Payment Contract

```java
public interface PaymentProcessor {
    PaymentResult charge(Money amount, PaymentMethod method);
}
```

A checkout service can depend on the capability it needs without knowing how a provider communicates over the network:

```java
public final class CheckoutService {
    private final PaymentProcessor paymentProcessor;

    public CheckoutService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public PaymentResult checkout(Cart cart, PaymentMethod method) {
        return paymentProcessor.charge(cart.total(), method);
    }
}
```

Implementations can hide provider-specific APIs, credentials handling, retries, and response mapping behind `PaymentProcessor`.

| Interface | Abstract class |
|---|---|
| Defines a contract a class can implement alongside other interfaces | Provides a shared base and can hold instance state |
| Useful for capabilities and interchangeable implementations | Useful when closely related subclasses share implementation or state |
| A class can implement multiple interfaces | A class can extend only one class in Java |

**Interview point:** Abstraction is about the boundary and the concepts exposed to a caller—not about hiding every implementation detail or creating an interface for every class.

## 5. Inheritance

**Inheritance** models an “is-a” relationship: a subtype extends or specializes the behavior of a base type.

```java
public abstract class Notification {
    private final String recipient;

    protected Notification(String recipient) {
        this.recipient = recipient;
    }

    public String recipient() {
        return recipient;
    }

    public abstract void send(String message);
}

public final class EmailNotification extends Notification {
    public EmailNotification(String recipient) {
        super(recipient);
    }

    @Override
    public void send(String message) {
        // Deliver through an email provider.
    }
}
```

`EmailNotification` is a kind of `Notification`, and it supplies the behavior promised by the base contract.

### The Substitutability Test

If code expects a base type, can it use an instance of the subtype without breaking its expected behavior? This is the practical meaning of the **Liskov Substitution Principle (LSP)**.

A subtype is a poor fit if it needs to reject valid base-class operations, strengthens preconditions, or weakens the guarantees callers rely on. A classic warning sign is a `Square` subclass of a mutable `Rectangle`: changing width independently of height can violate the square's invariant.

### Prefer Composition When the Relationship Is “Has-A”

Inheritance couples a subtype to its parent. If an object merely uses a capability, compose it instead:

```java
public final class NotificationService {
    private final MessageSender sender;

    public NotificationService(MessageSender sender) {
        this.sender = sender;
    }

    public void notifyUser(String recipient, String message) {
        sender.send(recipient, message);
    }
}
```

For example, a notification service **has a** sender; it is not itself necessarily an email sender. Composition makes it easier to replace or combine behavior without building a deep class hierarchy.

**Interview point:** Inheritance is useful for a genuine subtype relationship with shared behavior and substitutability. Avoid using it solely to reuse a few lines of code.

## 6. Polymorphism

**Polymorphism** lets code work with a shared type while the concrete object determines which implementation runs.

```java
public interface MessageSender {
    void send(String recipient, String message);
}

public final class EmailSender implements MessageSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending email to " + recipient);
    }
}

public final class SmsSender implements MessageSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending SMS to " + recipient);
    }
}

public final class NotificationService {
    private final MessageSender sender;

    public NotificationService(MessageSender sender) {
        this.sender = sender;
    }

    public void notifyUser(String recipient, String message) {
        sender.send(recipient, message);
    }
}
```

`NotificationService` does not need `if (type == EMAIL)` / `else if (type == SMS)` logic. At runtime, Java dispatches `send` to the implementation of the actual object.

### Compile-Time and Runtime Polymorphism

- **Runtime polymorphism (overriding):** A subclass or implementation supplies a method with the same signature; the concrete object's method is selected at runtime. This is the central form used for interchangeable strategies.
- **Compile-time polymorphism (overloading):** Methods share a name but differ in parameter lists; the compiler selects the method based on the declared argument types.

In Java, static methods are hidden rather than overridden, and fields are not dynamically dispatched. Use `@Override` to have the compiler verify intended overrides.

**Interview point:** The benefit is not merely calling the same method name. It is being able to add or substitute behavior behind a stable contract without spreading type checks throughout the caller.

## 7. Putting the Pillars Together

Consider a notification workflow:

1. `NotificationService` depends on the `MessageSender` abstraction.
2. `EmailSender` and `SmsSender` implement that contract and provide polymorphic behavior.
3. Each sender encapsulates its provider-specific configuration and delivery logic.
4. Inheritance can be used for a shared base implementation only if the implementations are true substitutable kinds of the base type; otherwise, use composition.

The design can be tested by injecting a fake `MessageSender`, without invoking a real email or SMS provider.

## 8. Common Interview Questions

### “What is the difference between abstraction and encapsulation?”

**Abstraction** focuses on what a caller can do through a simple contract. **Encapsulation** focuses on controlling access to state and protecting the object's invariants. An interface can provide abstraction; private state plus validated operations can provide encapsulation.

### “Is inheritance always better than composition?”

No. Inheritance is appropriate for a stable “is-a” relationship where subtypes honor the base contract. Composition is usually more flexible for assembling behavior and avoiding tight coupling.

### “Can you give an example of runtime polymorphism?”

Calling `sender.send(...)` through a `MessageSender` reference invokes `EmailSender.send` or `SmsSender.send` according to the actual object at runtime.

### “Why not just use a switch on the notification type?”

A switch can be perfectly reasonable for a small, closed set of behavior. When implementations vary or grow independently, a shared contract and polymorphic dispatch keep selection logic from being duplicated across callers. Do not add polymorphism just to avoid every conditional.

### “Does a getter violate encapsulation?”

Not necessarily. A getter can expose a safe, stable view. The concern is exposing mutable internals or providing unrestricted setters that let callers violate invariants.

## 9. Common Mistakes

- Treating encapsulation as private fields plus public setters.
- Creating inheritance hierarchies without checking substitutability.
- Using inheritance for a “has-a” relationship.
- Creating interfaces with only one implementation and no meaningful variation or boundary.
- Replacing every conditional with a class hierarchy, even when the behavior is small and stable.
- Confusing method overloading with runtime overriding.
- Exposing mutable collections or internal objects that callers can change behind the class's back.

## 10. A Concise Interview Answer

> “The four OOP pillars are encapsulation, abstraction, inheritance, and polymorphism. Encapsulation protects an object's state and invariants; abstraction exposes a simple contract; inheritance models a genuine subtype relationship; and polymorphism lets callers use that contract while implementations vary. For example, a notification service can depend on a `MessageSender` interface and use email or SMS implementations without knowing provider details. I would use inheritance only when subtypes are substitutable, and prefer composition for assembling capabilities.”

## 11. One-Minute Revision

| Ask yourself | Design check |
|---|---|
| Who owns this state? | Keep it with the behavior that maintains its invariants |
| What should callers know? | Expose a small, stable contract |
| Is this really an “is-a” relationship? | Use inheritance only if every subtype honors the base contract |
| Could behavior vary independently? | Consider an interface and polymorphic implementation |
| Is this only code reuse? | Prefer composition over a questionable base class |
