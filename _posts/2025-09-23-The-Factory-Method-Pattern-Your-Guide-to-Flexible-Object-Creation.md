---

title: "Mastering the Factory Method: Beyond the new Keyword"
description: "A hands-on guide to the Factory Method pattern, explaining how to decouple object creation for building flexible and maintainable software."
image: "./../assets/img/posts/2025-09-23-mastering-the-factory-method-beyond-the-new-keyword.png"
date: 2025-09-23 00:00:00 +0300
categories: [Design Patterns, Creational Patterns]
tags: [ design patterns, software development, object-oriented programming, oop, java, clean code, software architecture, programming, best practices, developer tips, programming best practices, software engineering, creational patterns, java design patterns tutorial, spring boot, gang of four design patterns, factory method, factory pattern, decoupling ]

---

> This article had its first draft written by AI, then refined by human editing to ensure clarity, correctness, and real-world applicability.
> {: .prompt-info }

---

## **1. The Hook and Introduction**

Have you ever been to a fast-food franchise which create burgers? The parent company has a standard menu and a core process for making a *burger.* But a franchise in
India might create a "McAloo Tikki," while one in Germany offers a "Nürnberger." The parent company defines the interface for creating a burger, but it’s the
regional franchises (the subclasses) that decide *which* burger to make.

That, in a nutshell, is the **Factory Method Pattern**. It's a creational design pattern that provides an interface for creating objects in a superclass but allows
subclasses to alter the type of objects created.

It might sound abstract, but it’s one of the most practical patterns you’ll encounter. It’s all about **delegating object creation**. Let’s break it down.

In this post, you’ll learn:

- What the Factory Method Pattern is, using simple analogies.
- The key components and structure of the pattern.
- How to implement it in Java with a real example.
- The pros, cons, and when you should (and should not) use it.

---

## **2. A Real-World Analogy: The Logistics Company**

Imagine building software for a logistics company. You need to ship goods, but the mode of transport isn’t known upfront: road or sea?

Without good design, you might end up with:

```java
if("road".equals(transportType)){
  new

Truck();
}else if("sea".

equals(transportType)){
  new

Ship();
}
```

Messy, right? What if you add **air freight** or **rail**? That central block keeps changing—a recipe for bugs.

The **Factory Method Pattern** solves this. Instead of hardcoding conditions, you:

- Define an abstract `Logistics` class with a factory method: `createTransport()`.
- Create subclasses (`RoadLogistics`, `SeaLogistics`) that implement `createTransport()`.

Your main app just calls `createTransport()`, without caring if it’s a **Truck** or a **Ship**.

---

## **3. Pattern Overview**

The **Gang of Four** define Factory Method as:

> *Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.*

Key components:

- **Product** → Interface or abstract class (e.g., `Transport`).
- **ConcreteProduct** → Implementations (e.g., `Truck`, `Ship`).
- **Creator** → Abstract class declaring the factory method (e.g., `Logistics`).
- **ConcreteCreator** → Subclasses overriding the factory method (e.g., `RoadLogistics`).

---

## **4. UML Diagram**

```
+---------------------+               +--------------------+
| <Creator>           |               | <Product>          |
| Logistics           |<>------------>| Transport          |
+---------------------+               +--------------------+
| + planDelivery()    |               | + deliver()        |
| + createTransport() |               +--------------------+
+---------------------+                        / \ 
         ^                                    /   \ 
         |                                   /     \
         |                                  /       \
+---------------------+     +-------------------+-------------------+
| <ConcreteCreator>   |     | <ConcreteProduct> | <ConcreteProduct> |
| RoadLogistics       |     | Truck             | Ship              |
+---------------------+     +-------------------+-------------------+
| + createTransport() |     | + deliver()       | + deliver()       |
+---------------------+     +-------------------+-------------------+
```

---

## **5. Code Example in Java**

Let’s turn the logistics analogy into working Java.

### Step 1 — Product Interface

```java
public interface Transport {
  void deliver();
}
```

### Step 2 — Concrete Products

```java
public class Truck implements Transport {
  @Override
  public void deliver() {
    System.out.println("Delivering by land in a truck.");
  }
}

public class Ship implements Transport {
  @Override
  public void deliver() {
    System.out.println("Delivering by sea in a container ship.");
  }
}
```

### Step 3 — Creator

```java
public abstract class Logistics {
  public void planDelivery() {
    Transport t = createTransport();
    System.out.println("Logistics plan confirmed. Starting delivery...");
    t.deliver();
  }

  public abstract Transport createTransport();
}
```

### Step 4 — Concrete Creators

```java
public class RoadLogistics extends Logistics {
  @Override
  public Transport createTransport() {
    return new Truck();
  }
}

public class SeaLogistics extends Logistics {
  @Override
  public Transport createTransport() {
    return new Ship();
  }
}
```

### Step 5 — **(BONUS)** — A Helper Simple Factory to Select the Creator

```java
public class LogisticsFactory {

  public static Logistics createLogistics(String type) {
    if (type == null || type.isEmpty()) {
      throw new IllegalArgumentException("Transport type cannot be null or empty.");
    }

    switch (type.toLowerCase()) {
      case "road":
        return new RoadLogistics();
      case "sea":
        return new SeaLogistics();
      // If you add AirLogistics, you only need to add a new case here.
      // case "air":
      //     return new AirLogistics();
      default:
        throw new IllegalArgumentException("Unknown transport type: " + type);
    }
  }
}
```

### Step 6 — Usage

```java
public class Application {
  public static void main(String[] args) {
    // The decision can come from a config file, user input, etc.
    String transportType = "sea";

    try {
      Logistics logistics = LogisticsFactory.createLogistics(transportType);
      logistics.planDelivery();
    } catch (IllegalArgumentException e) {
      System.err.println("Error: " + e.getMessage());
    }
  }
}
```

##### **A Quick Note:** Two Patterns at Play

It's important to recognize that this example beautifully demonstrates two distinct patterns working together to create a clean and scalable solution:

- ***The Factory Method Pattern:*** This is represented by the `Logistics` abstract class and its subclasses (`RoadLogistics`, `SeaLogistics`). Its purpose is to let
  a subclass decide which product (`Truck` or `Ship`) to create. The `planDelivery` method works with any product, but the subclass provides the specific one.

- ***The Simple Factory Idiom:*** This is represented by the `LogisticsFactory` class. We use it as a simple, centralized helper to hide the logic of choosing which
  creator (`RoadLogistics` or `SeaLogistics`) to use.

By combining them, the `Application` is completely decoupled. It doesn't know about concrete products or concrete creators. It simply asks the `LogisticsFactory` for
the right tool for the job and then uses it. This is a perfect example of applying the Single Responsibility Principle.


---

## **Spring Boot Context**

In **Spring Boot**, you rarely hand-roll factories. The IoC container already acts as one. Example:

```java

@Configuration
public class TransportConfig {

  @Bean
  @ConditionalOnProperty(name = "shipping.type", havingValue = "road")
  public Logistics roadLogistics() {
    return new RoadLogistics();
  }

  @Bean
  @ConditionalOnProperty(name = "shipping.type", havingValue = "sea")
  public Logistics seaLogistics() {
    return new SeaLogistics();
  }
}
```

Here, Spring decides which bean to create, based on properties.

---

## **6. Advantages & Disadvantages**

**Advantages**

- Loose Coupling → Client is decoupled from concrete implementations.
- Open/Closed Principle (OCP) → Add new product types without touching existing client code.
- Single Responsibility Principle (SRP) → Moves creation logic to dedicated subclasses, keeping the creator focused on its primary responsibility.

**Disadvantages**

- Extra Complexity → More classes and hierarchies.
- Subclassing Overhead → Sometimes feels heavy for simple cases.

---

## **7. When to Use or Avoid**

**Use it when:**

- The class can’t anticipate the exact type of objects it needs.
- You want subclasses to decide object creation.
- Building extensible frameworks/libraries.

**Avoid it when:**

- You only need a simple constructor.
- You end up with parallel hierarchies that don’t add real value.

---

## **8. Conclusion**

The **Factory Method Pattern** is a cornerstone of clean, extensible OOP design. By pushing object creation into subclasses, you decouple client code and make
systems easier to extend.

**Key Takeaways:**

- *Core Idea*: Let subclasses decide which object to create.
- *Main Benefit*: Loose coupling & flexibility.
- *Signs*: Abstract classes with methods returning abstract types.

---

***Pro Tip***: Next time you see a chain of `if/else` deciding object creation, ask: *Should this be a Factory Method?*

---

**Next Up:**  A single factory is useful for our **logistics company**, but how do we create entire kits of compatible equipment—like a specific ship and its
matching container? We'll use the **Abstract Factory Pattern** to ensure all our created objects belong to the same family.

---

## **GitHub Example**

You can find the complete, working code example for design patterns in my public GitHub repository. Feel free to clone it and experiment with the code.

- GitHub Repository - Design Patterns: [mcakar-dev/design-patterns](https://github.com/mcakar-dev/design-patterns)
- GitHub Repository - Factory
  Method: [mcakar-dev/design-patterns - Factory Method](https://github.com/mcakar-dev/design-patterns/tree/main/src/main/java/io/github/mcakardev/design/patterns/creational/factory)

---

## **References & Further Reading**

- Refactoring.Guru: [Factory Method](https://refactoring.guru/design-patterns/factory-method)
- GeeksforGeeks: [Factory method Design Pattern](https://www.geeksforgeeks.org/system-design/factory-method-for-designing-pattern/)
- *Head First Design Patterns* by Eric Freeman & Elisabeth Robson
- *Design Patterns: Elements of Reusable Object-Oriented Software* by Gamma, Helm, Johnson, Vlissides

