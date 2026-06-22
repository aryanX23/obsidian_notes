
## Definition

The **Factory Method Pattern** a type of [[Creation Design Patterns]], is used when we want to encapsulate object creation, instantiation, and all related business logic in one place.

This approach offers an interface for creating objects without needing to specify their exact classes. It delegates the responsibility of object creation to subclasses, which implement specific functionalities. Instead of using the `new` keyword directly in your client code, you utilize a factory method that returns objects adhering to a common interface. The actual class that gets instantiated is determined at runtime.

**Key Benefits:** Encourages loose coupling and enhances extensibility.

## Understanding the Structure (Shape Example)

1. **Product (`Shape`)**
    
    - Defines the interface for objects that the factory method creates.
        
    - All concrete shapes must implement this interface.
        
2. **Concrete Products (`Circle`, `Square`, `Rectangle`, `Triangle`)**
    
    - Specific implementations of the `Shape` interface.
        
    - Each has its own specific business logic (e.g., drawing logic and area calculation).
        
3. **Creator (`ShapeFactory`)**
    
    - An abstract class or interface that declares the factory method.
        
    - May provide a default implementation or template methods.
        
4. **Concrete Creators (`CircleCreator`, `RectangleCreator`, etc.)**
    
    - Override the factory method to return specific product instances.
        
    - Each concrete factory knows exactly how to create one type of shape.
        

## Implementation (Java)

### Step 1: Define the Core Product Components

These classes are shared between both the Simple Factory and Factory Method approaches.

Java

```
// Define the Product interface 
public interface Shape { 
    void computeArea(); 
    void draw(); 
} 

// Concrete Product classes 
public class Circle implements Shape { 
    @Override 
    public void computeArea() { 
        System.out.println("Inside Circle::computeArea() method."); 
    } 
    @Override 
    public void draw() { 
        System.out.println("Inside Circle::draw() method."); 
    } 
} 

public class Rectangle implements Shape { 
    @Override 
    public void computeArea() { 
        System.out.println("Inside Rectangle::computeArea() method."); 
    } 
    @Override 
    public void draw() { 
        System.out.println("Inside Rectangle::draw() method."); 
    } 
} 

public class Square implements Shape { 
    @Override 
    public void computeArea() { 
        System.out.println("Inside Square::computeArea() method."); 
    } 
    @Override 
    public void draw() { 
        System.out.println("Inside Square::draw() method."); 
    } 
} 
```

### Approach A: The Simple Factory

_(Note: This is often considered a programming idiom rather than a strict GoF design pattern)._

Java

```
public enum ShapeType { 
    CIRCLE, RECTANGLE, SQUARE 
} 

public class SimpleShapeFactory { 
    public static Shape createShapeInstance(ShapeType shapeType) { 
        if (shapeType == null) { 
            return null; 
        } 
        return switch (shapeType) { 
            case CIRCLE -> new Circle(); 
            case RECTANGLE -> new Rectangle(); 
            case SQUARE -> new Square(); 
            default -> throw new IllegalStateException("ShapeType doesn't exist!"); 
        }; 
    } 
} 

// Client Demonstration (Bloated Design when scaling)
public class SimpleFactoryDemo { 
    public static void main(String[] args) { 
        System.out.println("======= Simple Factory ======"); 
        
        ShapeType shapeType = ShapeType.SQUARE; 
        Shape shape = SimpleShapeFactory.createShapeInstance(shapeType); 
        
        shape.draw(); 
        shape.computeArea(); 
    } 
} 
```

### Approach B: The Factory Method Pattern

This is the true design pattern approach, relying on inheritance and polymorphism.

Java

```
// Abstract Creator class 
public abstract class ShapeFactory { 
    // Factory method - to be implemented by subclasses 
    public abstract Shape createShape(); 
} 

// Concrete Creator classes 
public class CircleCreator extends ShapeFactory { 
    @Override 
    public Shape createShape() { 
        return new Circle(); 
    } 
} 

public class RectangleCreator extends ShapeFactory { 
    @Override 
    public Shape createShape() { 
        return new Rectangle(); 
    } 
} 

public class SquareCreator extends ShapeFactory { 
    @Override 
    public Shape createShape() { 
        return new Square(); 
    } 
} 

// Client Demonstration
public class FactoryMethodDemo { 
    public static void main(String[] args) { 
        System.out.println("======= Factory Method Design Pattern ======"); 
        
        ShapeType shapeType = ShapeType.SQUARE; 
        Shape shape = getShapeInstance(shapeType); 
        
        if (shape != null) {
            shape.draw(); 
            shape.computeArea(); 
        }
    } 
    
    private static Shape getShapeInstance(ShapeType shapeType) { 
        if (shapeType == null) return null; 
        
        ShapeFactory creator = switch (shapeType) {
            case CIRCLE -> new CircleCreator();
            case RECTANGLE -> new RectangleCreator();
            case SQUARE -> new SquareCreator();
            default -> throw new IllegalStateException("ShapeType doesn't exist."); 
        };
        
        return creator.createShape(); 
    } 
}
```

## Factory Method vs. Simple Factory

|**Feature**|**Simple Factory**|**Factory Method Pattern**|
|---|---|---|
|**Concept**|A programming idiom using a centralized, static method.|A true design pattern using inheritance and polymorphism.|
|**Object Creation**|Creates objects based on parameters (`if/else` or `switch`).|Each concrete creator subclass handles one specific product type.|
|**Extensibility**|**Violates** the Open/Closed Principle. Adding a new type requires modifying the core factory logic.|**Follows** the Open/Closed Principle. Adding a new type just requires a new Creator class.|
|**Best Use Case**|Small projects with a fixed, limited set of types.|Frameworks and libraries that require extension points for users to customize behavior.|


---
