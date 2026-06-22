
## Definition

The **Abstract Factory Pattern** a type of [[Creation Design Patterns]], provides an interface for creating families of related or dependent objects without specifying their concrete classes.

Think of it as a "Factory of Factories" or a "Super Factory." It is more complex than the standard Factory pattern because it is responsible for creating families of _multiple_ related product types. This high level of abstraction enables the client to switch between different product families at runtime without needing to know the concrete implementation of how those products are instantiated.

## Understanding the Structure (Car Manufacturing Example)

1. **Abstract Product Interfaces (`CarInterior`, `CarExterior`)**
    
    - Define interfaces for a family of related products.
        
    - Clients depend on this interface to create objects without needing to know their specific types.
        
2. **Concrete Products (`EconomyCarInterior`, `LuxuryCarExterior`, etc.)**
    
    - Implement the abstract product interfaces.
        
    - Products from the same family are designed to work together seamlessly.
        
3. **Abstract Factory (`CarFactory`)**
    
    - Declares creation methods for each abstract product type.
        
    - Defines the contract for creating product families.
        
4. **Concrete Factories (`EconomyCarFactory`, `LuxuryCarFactory`)**
    
    - Implement the abstract factory interface.
        
    - Each factory produces products from a _single specific family_ (e.g., only Economy parts or only Luxury parts).
        
5. **Factory Provider**
    
    - Returns a concrete factory based on client input.
        
    - This concrete factory is then used to create families of concrete products as per the client’s request.
        
6. **Client (`Application`)**
    
    - Utilizes the Abstract Factory Interface to request concrete products.
        

## Implementation (Java)

### Step 1: Abstract Product Interfaces

Define the product families.

Java

```
public interface CarExterior { 
    void addExteriorComponents(); 
} 

public interface CarInterior { 
    void addInteriorComponents();
} 
```

### Step 2: Concrete Products

Implementations for the Economy and Luxury car families.

Java

```
// Concrete Products for Economy Car Family 
public class EconomyCarExterior implements CarExterior { 
    @Override 
    public void addExteriorComponents() { 
        System.out.println("Adding basic exterior components for Economy Car."); 
    } 
} 

public class EconomyCarInterior implements CarInterior { 
    @Override 
    public void addInteriorComponents() { 
        System.out.println("Adding basic interior components for Economy Car."); 
    } 
} 

// Concrete Products for Luxury Car Family 
public class LuxuryCarExterior implements CarExterior { 
    @Override 
    public void addExteriorComponents() { 
        System.out.println("Adding luxurious exterior components for Luxury Car."); 
    } 
} 

public class LuxuryCarInterior implements CarInterior { 
    @Override 
    public void addInteriorComponents() { 
        System.out.println("Adding luxurious interior components for Luxury Car."); 
    } 
} 
```

### Step 3: Abstract Factory Interface

Java

```
public interface CarFactory { 
    // Factory methods 
    CarInterior createInterior(); 
    CarExterior createExterior(); 
    
    // Template method that uses all factory methods 
    default void produceCompleteVehicle() { 
        System.out.println("Starting complete vehicle production..."); 
        
        // Create all components 
        CarInterior interior = createInterior(); 
        CarExterior exterior = createExterior(); 
        
        // Assemble the vehicle 
        interior.addInteriorComponents(); 
        exterior.addExteriorComponents(); 
        
        System.out.println("Vehicle production completed!\n"); 
    } 
} 
```

### Step 4: Concrete Factories

Java

```
public class EconomyCarFactory implements CarFactory { 
    private String brand; 
    
    public EconomyCarFactory(String brand) { 
        this.brand = brand;
    } 
    
    @Override 
    public CarInterior createInterior() { 
        return new EconomyCarInterior(); 
    } 
    
    @Override 
    public CarExterior createExterior() { 
        return new EconomyCarExterior(); 
    } 
} 

public class LuxuryCarFactory implements CarFactory { 
    private final String brand; 
    
    public LuxuryCarFactory(String brand) { 
        this.brand = brand; 
    } 
    
    @Override 
    public CarInterior createInterior() { 
        return new LuxuryCarInterior(); 
    } 
    
    @Override 
    public CarExterior createExterior() { 
        return new LuxuryCarExterior(); 
    } 
} 
```

### Step 5: Factory Provider

Java

```
public enum CarType {
    ECONOMY, PREMIUM, LUXURY
}

public class CarFactoryProvider { 
    public CarFactory getFactory(CarType type, String brand) { 
        switch (type) { 
            case ECONOMY: 
                return new EconomyCarFactory(brand); 
            case PREMIUM: 
            case LUXURY: 
                return new LuxuryCarFactory(brand); 
            default: 
                throw new IllegalArgumentException("Unknown car type: " + type); 
        } 
    } 
} 
```

### Step 6: Client Demonstration

Java

```
public class AbstractFactoryDemo { 
    public static void main(String[] args) { 
        System.out.println("===== Abstract Factory Design Pattern ====="); 
        
        // Get Factory Provider 
        CarFactoryProvider carFactoryProvider = new CarFactoryProvider(); 
        
        // Get Economy Car Factory 
        CarFactory economyCar = carFactoryProvider.getFactory(CarType.ECONOMY, "Honda"); 
        economyCar.produceCompleteVehicle(); 
        
        // Get Luxury Car Factory 
        CarFactory luxuryCar = carFactoryProvider.getFactory(CarType.LUXURY, "Mercedes"); 
        luxuryCar.produceCompleteVehicle(); 
        
        // Get Premium Car Factory 
        CarFactory premiumCar = carFactoryProvider.getFactory(CarType.PREMIUM, "Rolls Royce"); 
        premiumCar.produceCompleteVehicle(); 
    } 
}
```

> **Key Takeaway:** The Abstract Factory Pattern is highly effective when you need to ensure that related products work well together, making it especially suitable for complex systems with various product variants. It strongly supports extensibility and complies beautifully with the Open/Closed Principle.

## Factory Method vs. Abstract Factory

|**Scenario**|**Pattern to Use**|
|---|---|
|**One product, many variants**|**Factory Method**|
|**Many products, grouped by family** (theme, platform, brand, etc.)|**Abstract Factory**|

---