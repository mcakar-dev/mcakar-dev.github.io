---

title: "Mastering the Prototype Pattern: The Art of the Perfect Copy"  
description: "An accessible look at the Prototype pattern, exploring the difference between shallow and deep copying to create new objects effectively."  
image: "./../assets/img/posts/2025-10-02-mastering-the-prototype-pattern-the-art-of-the-perfect-copy.png"  
date: 2025-10-02 00:00:00 +0300  
categories: [Design Patterns, Creational Patterns]  
tags: [ design patterns, software development, object-oriented programming, oop, java, clean code, software architecture, programming, best practices, developer tips, programming best practices, software engineering, creational patterns, java design patterns tutorial, spring boot, gang of four design patterns, prototype, prototype pattern, cloning, shallow copy, deep copy ]  

---

> This article had its first draft written by AI, then refined by human editing to ensure clarity, correctness, and real-world applicability.
{: .prompt-info }

---

## **1. The Hook and Introduction**

In biology, cells multiply through mitosis—they create nearly identical copies of themselves. This process is far more efficient than building a new cell from scratch every single time. Why assemble every component from basic molecules when you have a perfect template ready to go?  

In software development, we often face a similar situation. Sometimes, creating an object is an expensive operation, involving database queries, network calls, or heavy computation. If you need many similar objects, instantiating each one individually can be a major performance bottleneck.  

The **Prototype Pattern** is a creational design pattern that solves this by letting you create new objects by copying an existing one, known as a "prototype." Instead of building from scratch, you clone a pre-configured instance. It’s the software equivalent of having a photocopier for your objects.  

In this article, you'll learn:
* What the Prototype pattern is and when it's useful.
* The critical difference between a **shallow copy** and a **deep copy**.
* How to implement it in Java using a modern and safe copy constructor approach.
* The pros, cons, and common pitfalls of the pattern.

---

## **2. A Real-World Analogy: The Game Development Scenario**

Imagine you're building a game with hundreds of enemies. Each enemy object might be complex, with attributes like health, speed, AIBehavior, and a 3D model that needs to be loaded from disk.  

Without a good design, your code for spawning a new enemy might look like this:

```java
public Enemy createEnemy() {  
    Configuration config = loadConfigFromFile("enemy_config.json");  
    Model3D model = loadModelFromDisk("zombie.model");

    Enemy enemy = new Enemy();  
    enemy.setHealth(config.getHealth());  
    enemy.setSpeed(config.getSpeed());  
    enemy.setModel(model);  
    return enemy;  
}
```

Calling this method every time a new enemy appears is incredibly inefficient. The **Prototype Pattern** offers a much faster alternative: create one "master" enemy object at the start of the game, and whenever you need a new one, just create a copy of it.

---

## **3. Pattern Overview**

The Gang of Four define the Prototype's intent as:  

> *Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.*  

**Core roles:**

* **Prototype** → An interface or abstract class that declares a copy() method.
* **ConcretePrototype** → A class that implements the copy() method to create a copy of itself.
* **Client** → Creates a new object by asking a prototype to clone itself.

---

## **4. UML Diagram**

```
                                  +-----------------------+
                                  | <Prototype>           |
                                  | Enemy                 |
                                  +-----------------------+
                                  | - name: String        |
                                  | - type: String        | 
                                  +-----------------------+
                                  | + copy(): Enemy       |
                                  +-----------------------+
                                             ^
                                             |
                      +----------------------+----------------------+
                      |                                             |
+---------------------+-----------+                 +---------------------------------+
| <ConcretePrototype>             |                 | <ConcretePrototype>             |
| Zombie                          |                 | Monster                         |
+---------------------------------+                 +---------------------------------+
| + copy(): Enemy                 |                 | + copy(): Enemy                 |
+---------------------------------+                 +---------------------------------+
```

---

## **5. Code Example in Java**

Let’s model our game enemy scenario using a **modern** and **safe copy constructor** approach, which is preferable to Java's built-in `Cloneable` interface.

### Step 1 — The Abstract Prototype

Create an abstract `Enemy` class that defines a copy constructor and an abstract `copy()` method.

```java
public abstract class Enemy {
    private String name;
    protected String type;
    
    public Enemy() {}
    
    public Enemy(Enemy source) {
        this.name = source.name;
        this.type = source.type;
    }

    public abstract void attack();
    public abstract Enemy copy();
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getType() { return type; }
    
    @Override
    public String toString() {
        return String.format("%s [name=%s]", type, name);
    }
}
```

### Step 2 — Concrete Prototypes

These are the specific types of enemies that can be cloned.

```java
public class Zombie extends Enemy {
    public Zombie() {
        this.type = "Zombie";
    }
    
    public Zombie(Zombie source) {
        super(source);
    }
  
    @Override
    public Enemy copy() {
        return new Zombie(this);
    }
  
    @Override
    public void attack() {
        System.out.println("Zombie attacks with a bite!");
    }
}

public class Ghoul extends Enemy {
    public Ghoul() {
        this.type = "Ghoul";
    }
    
    public Ghoul(Ghoul source) {
        super(source);
    }
    
    @Override
    public Enemy copy() {
        return new Ghoul(this);
    }
    
    @Override
    public void attack() {
        System.out.println("Ghoul attacks with its claws!");
    }
}
```

### Step 3 — **(BONUS)** — The Prototype Registry (Modern Enum Approach)

A registry is a great way to manage your prototypes. It acts like a factory that returns clones.

```java
public enum EnemyType {
    ZOMBIE(new Zombie()),
    GHOUL(new Ghoul());
  
    private final Enemy prototype;
  
    EnemyType(Enemy prototype) {
        this.prototype = prototype;
    }
  
    public Enemy createInstance() {
        return this.prototype.copy();
    }
}
```

### Step 4 — Client and Usage

```java
public class Game {
    public static void main(String[] args) {
        Enemy zombie1 = EnemyType.ZOMBIE.createInstance();
        zombie1.setName("Zombie X");
    
        Enemy zombie2 = EnemyType.ZOMBIE.createInstance();
        zombie2.setName("Zombie Y");
    
        Enemy ghoul1 = EnemyType.GHOUL.createInstance();
        ghoul1.setName("Ghoul Z");
    
        System.out.println("Created: " + zombie1);
        System.out.println("Created: " + zombie2);
        System.out.println("Created: " + ghoul1);
    
        zombie1.attack();
        zombie2.attack();
        ghoul1.attack();
    }
}
```

---

## **6. The Big Problem: Shallow vs. Deep Copy**

Our current `copy()` method performs a **shallow copy**. This is fine for primitive types and immutable objects like String, but it's dangerous for mutable objects.

* **Shallow Copy**: Copies fields value by value. If a field is a reference to an object, it copies the **reference**, not the object itself. Both the original and the clone will point to the *same* nested object.
* **Deep Copy**: Creates a completely independent copy. It copies not only the object's fields but also recursively clones any objects referenced by those fields.

Let’s add a mutable `Weapon` object to our `Enemy` to see the problem.

```java
public class Weapon {
    private String name;
    public Weapon(String name) { this.name = name; }

    public Weapon(Weapon source) { this.name = source.name; }
  
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

public abstract class Enemy {
    private Weapon weapon;
    public Weapon getWeapon() { return weapon; }
    public void setWeapon(Weapon weapon) { this.weapon = weapon; }
  
    public Enemy() {}
  
    public Enemy(Enemy source) {
        this.name = source.name;
        this.type = source.type;
        this.weapon = source.weapon;
    }
}
```

**The Danger of Shallow Copy:**

```java
// Give the prototype a weapon
Zombie zombiePrototype = new Zombie();
zombiePrototype.setWeapon(new Weapon("Axe"));

// Create two copies
Enemy zombie1 = zombiePrototype.copy();
Enemy zombie2 = zombiePrototype.copy();

// Change the weapon of the first copy
zombie1.getWeapon().setName("Sword");

// What's the weapon of the second copy?
System.out.println(zombie2.getWeapon().getName()); // Output: Sword
```

Changing one clone affected the other! This is because both clones share the same `Weapon` object.  

**Solution: Implement Deep Copy**  
To fix this, we must also create a new copy of the `Weapon` object inside the `Enemy` copy constructor.

```java
...
public Enemy(Enemy source) {
    this.name = source.name;
    this.type = source.type;
    if (source.weapon != null) {
        this.weapon = new Weapon(source.weapon); 
    }
}
...
```

With this change, each cloned `Enemy` gets its own independent `Weapon` object, and modifying one will not affect the others.

---

## **7. Spring Boot Context**

In Spring, the term "prototype" has a slightly different meaning. When you define a bean with `@Scope("prototype")`, the Spring IoC container creates a **new instance** of that bean every time it's requested.

```java
@Component  
@Scope("prototype")  
public class GameCharacter {  
// ...  
}
```

This is different from the GoF Prototype pattern because Spring isn't *cloning* a pre-existing instance; it's calling the constructor to create a fresh one. It solves a similar problem—avoiding a shared singleton instance—but through instantiation, not copying.

---

## **8. Advantages & Disadvantages**

**Advantages**

* **Performance**: Cloning an existing object is often much faster than creating a new one from scratch.
* **Simplicity**: Hides the complexity of object creation from the client.
* **Dynamic Configuration**: You can add and remove prototype objects to a registry at runtime.

**Disadvantages**

* **Complexity of Cloning**: Implementing a deep copy can be tricky. You have to be careful about circular references and whether you need a shallow or deep copy.
* **Requires Careful Design**: Every class in the hierarchy you want to clone must support the cloning process.

---

## **9. When to Use or Avoid**

**Use it when:**

* Object creation is expensive (e.g., requires database/network calls).
* You have a set of objects that only differ slightly in their state.
* You want to avoid a class hierarchy of factories that mirrors the class hierarchy of products.

**Avoid it when:**

* Your classes are simple, and object creation isn't a performance concern.
* The logic for deep copying a complex object with many nested mutable objects becomes too difficult to manage.

---

## **10. Conclusion**

The Prototype pattern is a powerful tool for optimizing object creation. By creating new objects from a template copy, you can significantly improve performance and simplify your code. However, its power comes with responsibility—you must master the distinction between shallow and deep copying to avoid subtle and frustrating bugs.  

**Key Takeaways:**

* **Core Idea:** Create new objects by cloning an existing prototype instance.
* **Main Benefit:** Boosts performance by avoiding expensive object construction.
* **Critical Pitfall:** A shallow copy can lead to unintended side effects; use a deep copy for objects with mutable fields.
* **Modern Implementation:** Prefer copy constructors over Java's `Cloneable` interface.
* **Spring's @Scope("prototype"):** Creates a new instance, not a clone.

**Final Thought:**  
Don't reinvent the wheel if you don't have to. Sometimes, the best way to create something new is to just clone it.

---

**Next Up:** We've now covered the major creational patterns! Next, we'll shift our focus to how objects and classes can be composed to form larger structures. We'll start with the **Adapter Pattern**, a structural pattern that helps incompatible interfaces work together.

---

## **GitHub Example**

You can find the complete, working code example for design patterns in my public GitHub repository. Feel free to clone it and experiment with the code.

- GitHub Repository - Design Patterns: [mcakar-dev/design-patterns](https://github.com/mcakar-dev/design-patterns)
- GitHub Repository - Prototype: [mcakar-dev/design-patterns - Prototype](https://github.com/mcakar-dev/design-patterns/tree/main/src/main/java/io/github/mcakardev/design/patterns/creational/prototype)

---

## **References & Further Reading**

* *Effective Java* by Joshua Bloch
* *Head First Design Patterns* by Eric Freeman & Elisabeth Robson
* *Design Patterns: Elements of Reusable Object-Oriented Software* by Gamma, Helm, Johnson, Vlissides
* Baeldung: [Prototype Pattern in Java](https://www.baeldung.com/java-pattern-prototype)
* Refactoring.Guru: [Prototype](https://refactoring.guru/design-patterns/prototype)
