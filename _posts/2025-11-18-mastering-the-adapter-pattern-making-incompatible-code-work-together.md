---

title: "Mastering the Adapter Pattern: Making Incompatible Code Work Together"  
description: "An accessible introduction to the Adapter pattern, showing how to wrap existing classes so they can collaborate with unrelated ones"  
image: "./../assets/img/posts/2025-11-18-mastering-the-adapter-pattern-making-incompatible-code-work-together.png"  
date: 2025-11-18 00:00:00 +0300  
categories: [Design Patterns, Structural Patterns]  
tags: [ design patterns, software development, object-oriented programming, oop, java, clean code, software architecture, programming, best practices, developer tips, programming best practices, software engineering, structural patterns, java design patterns tutorial, spring boot, gang of four design patterns, adapter, adapter pattern, decoupling, wrapper ] 

---

> This article had its first draft written by AI, then refined by human editing to ensure clarity, correctness, and real-world applicability.
{: .prompt-info }

---

## **1. The Hook and Introduction**

Have you ever traveled to another country, only to find that your laptop charger won't fit into the wall socket? The plug shapes are completely different. Your charger works perfectly, and the wall socket works perfectly, but they just can't connect. The solution is simple: a travel adapter. It's a small device that "adapts" your plug to fit the foreign socket, bridging the gap without changing either end.  

In software, we face this problem all the time. You have a new, third-party analytics library you need to use, but its methods have completely different names and parameters than what your application expects. Or, you need to make a legacy class work with a modern system.  

The **Adapter Pattern** is a structural design pattern that acts as that travel adapter. It allows objects with incompatible interfaces to collaborate. It's a "wrapper" that translates calls from one interface into another, allowing systems to work together seamlessly.  

In this article, you'll learn:
* What the Adapter pattern is and why it's a structural pattern.  
* The key components and structure of the pattern.  
* A practical Java implementation for integrating a third-party service.  
* A classic example of where Spring Boot uses this pattern.  
* The pros, cons, and when you should (and should not) use it.

---

## **2. A Real-World Analogy: The Analytics Service**

Imagine your application has an AnalyticsService interface that all your code uses to track events:

```java
public interface AnalyticsService {  
    void logEvent(String eventName, String eventType);  
}
```

Your whole application is built around this. Now, your company buys a subscription to a new, high-end analytics platform, *SuperLog*. Its SDK is fantastic, but its main class looks like this:

```java
public class SuperLog {  
    public void track(String eventTitle, String category, Map<String, Object> metadata) {  
        // ... complex logging logic  
    }  
}
```

This is a problem. The new service's track method is incompatible with your app's logEvent method. You can't just replace the calls everywhere—that would be a massive, risky refactor.  

The **Adapter Pattern** solves this. You create a new class, SuperLogAdapter, that implements *your* AnalyticsService interface but secretly *wraps* the SuperLog object.  

---

## **3. Pattern Overview**

The Gang of Four define the Adapter's intent as:  

> *Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.*

**Core roles:**

* **Target** (AnalyticsService) → The interface that your application's client code expects to use.  
* **Adaptee** (SuperLog) → The incompatible class or interface that needs to be adapted.  
* **Adapter** (SuperLogAdapter) → The class that implements the Target interface and contains an instance of the Adaptee. It translates requests from the Target to the Adaptee.  
* **Client** (YourApplication) → The code that interacts with the Target interface.

---

## **4. UML Diagram**

```
+----------+      +-------------------+      +--------------+  
| Client   |----->| <Target>          |      | <Adaptee>    |  
|          |      | AnalyticsService  |      | SuperLog     |  
+----------+      +-------------------+      +--------------+  
                  | + logEvent(...)   |      | + track(...) |  
                  +-------------------+      +--------------+  
                            ^                  ^  
                            |                  |  
                          +----------------------+  
                          | <Adapter>            |  
                          | SuperLogAdapter      |  
                          +----------------------+  
                          | - adaptee: SuperLog  |  
                          +----------------------+  
                          | + logEvent(...)      |  
                          +----------------------+
```

---

## **5. Code Example in Java**

Let's turn our analytics analogy into code.

### Step 1 — The Target Interface (What your app uses)

This is the interface your client code is already coupled to.

```java
public interface AnalyticsService {  
    void logEvent(String eventName, String eventType);  
}
```

### Step 2 — The Adaptee (The incompatible class)

This is the new, third-party class you want to use.

```java
public class SuperLog {  
    public void track(String eventTitle, String category, Map<String, Object> metadata) {  
        System.out.println("SuperLog tracking: '" + eventTitle +   
                           "' in category '" + category + "'");  
    }  
}
```

### Step 3 — The Adapter

This is the magic. The `SuperLogAdapter` **implements** the Target and **wraps** the Adaptee.

```java
public class SuperLogAdapter implements AnalyticsService {  
    
    private final SuperLog superLog;
    private final String sourceName;

    public SuperLogAdapter(SuperLog superLog, String sourceName) {  
        this.superLog = superLog;
        this.sourceName = sourceName;  
    }

    @Override  
    public void logEvent(String eventName, String eventType) {  
        Map<String, Object> metadata = new HashMap<>();  
        metadata.put("source", this.sourceName);  

        superLog.track(eventName, eventType, metadata);  
    }  
}
```

### Step 4 — Client and Usage

Your client code doesn't change at all. It still only knows about AnalyticsService.

```java
public class Application {  
    public static void main(String[] args) {  
        
        AnalyticsService analyticsService;

        // ---
        // Old implementation (if we had one)  
        // analyticsService = new OldAnalyticsImplementation();  
        // analyticsService.logEvent("UserClicked", "Button");
        // ---
        // Now, we can swap in the new implementation without the client ever knowing the difference.  
        // ---
        
        analyticsService = new SuperLogAdapter(new SuperLog(), "MainApp");
        analyticsService.logEvent("UserLoggedIn", "Authentication");  
        analyticsService.logEvent("ItemPurchased", "E-commerce");  
    }  
}
```

---

## **6. Spring Boot Context**

A classic example of the Adapter pattern in the Spring ecosystem is the **HandlerAdapter** in Spring MVC.  

In Spring MVC, a DispatcherServlet routes web requests. However, there are many ways to define a "handler" (the thing that processes the request):

* You might use @Controller with `@RequestMapping`.  
* You might use functional endpoints (`RouterFunction`).  
* In older Spring versions, you could implement the `Controller` interface.

The DispatcherServlet doesn't want to know about all these different types. It just wants to call a single handle method.  

The **HandlerAdapter** (like `RequestMappingHandlerAdapter`) acts as the adapter. It takes a specific type of handler (like your `@Controller` method) and adapts it to the common interface (`handle(...)`) that the DispatcherServlet expects. This makes the framework incredibly flexible, allowing it to support new types of handlers without changing the core DispatcherServlet.  

---

## **7. Advantages & Disadvantages**

**Advantages**

* **Decouples Client:** The client is isolated from the implementation details of the Adaptee. You can swap out adapters with different implementations.  
* **Single Responsibility Principle:** The adapter's sole job is translation, which keeps both the client and the adaptee code clean.  
* **Integrates Legacy Code:** The primary use case. You can make old, non-standard code work with new systems.  
* **Open/Closed Principle:** You can introduce new adapters without modifying the existing client or target interface.

**Disadvantages**

* **Increases Complexity:** You add an extra class (the adapter) for every class you need to integrate, which can add a layer of indirection.  
* **Potential for "Smell":** If you find yourself adapting *everything*, your Target interface might be poorly designed.

---

## **8. When to Use or Avoid**

**Use it when:**

* You need to use a third-party or legacy class, but its interface doesn't match what your client code expects.  
* You want to create a reusable class that cooperates with several unrelated classes that have different interfaces.  
* (Object Adapter variant) You need to use several subclasses of an adaptee, but it's not practical to subclass all of them. The adapter can hold the parent type and work with all children.

**Avoid it when:**

* You have the ability to change the source code of the class you want to use. If you can modify the class to implement your Target interface directly, do that instead.  
* The "adaptation" is simple. If it's just one method name change, a simple lambda or function reference might be cleaner than a whole new class.

---

## **9. Conclusion**

The Adapter pattern is an indispensable tool in any developer's arsenal. It's a simple, elegant pattern that solves a messy, common problem. Like a universal travel adapter, it handles the "translation" so your code components can plug in and work together, regardless of their different "shapes".  

**Key Takeaways:**

* **Core Idea:** Wraps an incompatible object to make it usable by a client that expects a different interface.  
* **Main Benefit:** Integrates new or legacy code without changing the existing client.  
* **Key Structure:** Client -> Target (Interface) <- Adapter (Class) -> Adaptee (Object)  
* **In Spring:** The HandlerAdapter in Spring MVC is a perfect real-world example.

**Final Thought:**
Don't refactor your entire application just because a new library has a weird API. Encapsulate the incompatibility. That's what a good adapter is for.  

---

**Next Up:** We've seen how to make two different interfaces work together. But what if you need to separate an abstraction from its implementation so that *both* can evolve independently? Next, we'll explore the **Bridge Pattern**, a powerful pattern for decoupling complex hierarchies.  

---

## **10. GitHub Example**

You can find the complete, working code example for design patterns in my public GitHub repository. Feel free to clone it and experiment with the code.

- GitHub Repository - Design Patterns: [mcakar-dev/design-patterns](https://github.com/mcakar-dev/design-patterns)
- GitHub Repository - Adapter: [mcakar-dev/design-patterns - Adapter](https://github.com/mcakar-dev/design-patterns/tree/main/src/main/java/io/github/mcakardev/design/patterns/structural/adapter)

---

## **11. References & Further Reading**

* *Head First Design Patterns* by Eric Freeman & Elisabeth Robson  
* *Design Patterns: Elements of Reusable Object-Oriented Software* by Gamma, Helm, Johnson, Vlissides  
* *Refactoring: Improving the Design of Existing Code* by Martin Fowler  
* Baeldung: [The Adapter Pattern in Java](https://www.baeldung.com/java-adapter-pattern)
* Refactoring.Guru: [Adapter](https://refactoring.guru/design-patterns/adapter)
