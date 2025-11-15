
# 🧑‍🤝‍🧑 Behavioral Design Patterns

Behavioral design patterns focus on the **interactions** and **communication** between objects. They help define how objects collaborate and distribute responsibility, simplifying complex control flow.

---

## 🔗 Types of Behavioral Design Patterns

### 1. [[Chain of Responsibility Design Pattern]]

- **Concept:** Allows an object to pass a request along a chain of handlers. Each handler decides whether to process the request or pass it to the next handler.
    
- **When to Use:**
    
    - When objects need to communicate in **complicated ways**.
        
    - When you want to easily change how things behave **while the program is running**.
        

### 2. [[Command Design Pattern]]

- **Concept:** Turns a request into a stand-alone object called a **command**. It captures all parts of a request: the object, the method, and the parameters.
    
- **When to Use:**
    
    - To **separate the requester (sender)** from the object executing the request.
        
    - If you need to support **undo and redo** operations.
        

### 3. [[Interpreter Design Pattern]]

- **Concept:** Defines a way to interpret and evaluate language **grammar or expressions**. It represents the grammar as a set of classes.
    
- **When to Use:**
    
    - If you need to interpret and execute expressions or commands in a **domain-specific language (DSL)**.
        
    - If your application frequently requires the addition of **new operations or commands**.
        

### 4. [[Mediator Design Pattern]]

- **Concept:** Defines an object (the **Mediator**) that encapsulates how a set of objects interact. It promotes **loose coupling** by preventing objects from referring to each other explicitly.
    
- **When to Use:**
    
    - When a set of objects needs to communicate in a **complex manner**.
        
    - When you need a **centralized mechanism** to coordinate and control interactions, ensuring a more organized system.
        

### 5. [[Memento Design Pattern]]

- **Concept:** Used to **capture and restore an object’s internal state** without violating encapsulation. Allows saving and restoring to a previous state.
    
- **When to Use:**
    
    - When you need to save the state of an object at various points in time for features like **versioning or checkpoints**.
        
    - When you need to **rollback changes** to an object's state (e.g., in database transactions).
        

### 6. [[Observer Design Pattern]] (Dependents, Publish-Subscribe)

- **Concept:** Defines a **one-to-many dependency** between objects. When one object (the **Subject**) changes state, all its dependents (the **Observers**) are notified and updated automatically.
    
- **When to Use:**
    
    - When a change to one object requires changing others, and you **don't know how many** objects need to be changed.
        
    - When an object should be able to notify others **without making assumptions** about who these objects are.
        

### 7. [[State Design Pattern]] (Objects for States)

- **Concept:** Allows an object to **alter its behavior** when its **internal state changes**. The object appears to change its class.
    
- **When to Use:**
    
    - When **conditional statements (if-else or switch-case)** become extensive and complex within your object.
        
    - If your object transitions between states frequently, providing a clear mechanism for managing these transitions.
        

### 8. [[Strategy Design Pattern]] (Policy)

- **Concept:** Defines a **family of algorithms**, encapsulates each one, and makes them **interchangeable**. It lets the algorithm vary independently from clients that use it.
    
- **When to Use:**
    
    - When you have **multiple algorithms** that can be used interchangeably based on different contexts (e.g., different sorting algorithms).
        
    - When you need to **dynamically select and switch** between different algorithms at runtime.
        

### 9. [[Template Design Pattern]]

- **Concept:** Defines the **skeleton of an algorithm** in an operation, deferring some steps to subclasses. It lets subclasses redefine certain steps without changing the algorithm's structure.
    
- **When to Use:**
    
    - If you have **similar tasks or processes** that need to be performed in different contexts.
        
    - When you want to **enforce a specific structure or sequence of steps** in an algorithm while allowing for flexibility in certain parts.
        

### 10. [[Visitor Design Pattern]]

- **Concept:** Allows you to **add new operations** to a group of related classes **without modifying their structures**. The operation (Visitor) is separate from the element classes.
    
- **When to Use:**
    
    - When you have a stable set of classes and need to perform **different operations** on them, enabling easy extension of functionality.
        
    - Use it when your object structure is **unlikely to change frequently**, but you anticipate needing to add new operations over time.

---