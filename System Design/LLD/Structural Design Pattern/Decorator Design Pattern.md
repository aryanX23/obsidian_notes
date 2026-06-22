## Definition

The **Decorator Pattern**  a type of [[Structural Design Patterns]] allows you to add new functionality to objects dynamically without altering their original structure.

## Use Cases

### Example 1: Pizza Shop

**What we have:**

- **Basic Pizza:** Margherita (Crust + Cheese)
    
- **Available Toppings:** Extra Cheese, Olives, Jalapenos, Pepperoni, Veggies, Spicy Red Pepper, etc.
    

**Pizza Combinations:**

- **Type 1:** Margherita + Extra Cheese
    
- **Type 2:** Margherita + Olives + Jalapenos
    
- **Type 3:** Margherita + Olives + Jalapenos + Veggies + Extra Cheese
    
- **Type 4:** Margherita + Pepperoni + Spicy Red Pepper
    

### Example 2: Coffee Cafe

**What we have:**

- **Espresso:** A strong shot of pure coffee.
    
- **Available add-ons:** Sugar, Hot water, Cold Water, Ice, Steamed Milk, Milk Foam, Cold Milk, Chocolate Syrup, Vanilla ice cream, etc.
    

**Coffee Beverages:**

- **Type 1: Doppio** → A double shot of espresso.
    
- **Type 2: Americano** → Espresso + Hot water.
    
- **Type 3: Cappuccino** → Espresso + Steamed Milk + Milk Foam.
    
- **Type 4: Mocha** → Espresso + Steamed Milk + Chocolate Syrup.
    
- **Type 5: Cold Coffee** → Espresso + Cold Water + Cold Milk + Ice.
    
- **Type 6: MakeYourOwnCoffee** → Espresso + Cold Water + Cold Milk + Ice + Vanilla Ice Cream + Chocolate Syrup.
    

## Why do we need the Decorator Pattern?

To grasp the necessity of the Decorator Pattern, we must first look into the challenges encountered when implementing the problem using a naive approach (without the pattern).

### The Issue: Class Explosion

**Class explosion** refers to a situation where the number of classes in your system grows rapidly and uncontrollably, making the codebase hard to navigate, maintain, and understand. This happens when every combination of additional behavior requires a new decorator class.

**Example Scenario (Coffee Cafe):**

You have a base component `Coffee`, which has decorators like `MilkDecorator`, `SugarDecorator`, `VanillaIceCreamDecorator`, `ChocolateSyrupDecorator`, etc.

To support all combinations using standard inheritance, you would need to create classes like:

- `MilkAndSugarCoffeeDecorator`
    
- `SugarAndVanillaDecorator`
    
- `MilkVanillaIceCreamDecorator`
    
- `ChocolateSyrupVanillaIceCreamDecorator`
    

That’s where class explosion kicks in. We begin to have too many classes (extreme subclassing) to represent all possible combinations of features/behaviors.

## Decorator Pattern as a Solution

### Understanding the Structure (Pizza Shop Example)

1. **Component Interface (`BasePizza`)**
    
    - Defines the contract that both the base object and the decorators must follow.
        
    - Contains methods like `getDescription()` and `getCost()`.
        
2. **Concrete Component (`PlainPizza`, `Farmhouse`, `TandooriPaneerDelight`, `ChickenDominator`)**
    
    - The base implementation that provides core functionality.
        
    - `PlainPizza` represents a simple pizza without any toppings.
        
    - Can contain a pre-formulated combination of features.
        
3. **Base Decorator (`ToppingDecorator`)**
    
    - An abstract class that implements the `Pizza` interface.
        
    - Contains a reference to a `Pizza` object—the component itself (**composition**).
        
    - Delegates calls to the wrapped pizza object.
        
4. **Concrete Decorators (`ExtraCheeseTopping`, `VeggiesTopping`, `MushroomTopping`, `PepperoniTopping`)**
    
    - Extend the base decorator to add specific functionality.
        
    - Override methods to add their behavior while calling the wrapped object.
        
    - Can modify both the description and the cost.
        

### Breakdown of Relationships

#### 1. IS-A Relationship (Inheritance-based)

Concrete Components (`PlainPizza`, `Farmhouse`, etc.) all implement the Component Interface (`BasePizza`). Hence, they are types of Pizza.

- `PlainPizza` is-a `BasePizza`.
    
- `ExtraCheeseTopping` is-a `ToppingDecorator`.
    
- `ToppingDecorator` is-a `BasePizza`.
    
- **Therefore:** `ExtraCheeseTopping` is-a `BasePizza`.
    

#### 2. HAS-A Relationship (Composition-based)

Decorators wrap around another `Pizza`, adding their specific behavior.

- `ToppingDecorator` has-a `BasePizza`.
    
- `ExtraCheeseTopping` has a `PlainPizza`, which is a `BasePizza` (e.g., Plain Pizza with extra cheese).
    
- `MushroomTopping` has a `PlainPizza` (e.g., Mushroom Pizza).
    

## Implementation (Java)

### Step 1: Define the Component Interface

Java

```
public interface BasePizza { 
    String getDescription(); 
    double getCost(); 
} 
```

### Step 2: Define the Concrete Components

Java

```
class PlainPizza implements BasePizza { 
    @Override 
    public String getDescription() { 
        return "Plain Pizza"; 
    }
    
    @Override 
    public double getCost() { 
        return 200.00; 
    } 
} 

public class Farmhouse implements BasePizza { 
    @Override 
    public String getDescription() { 
        return "Farmhouse Pizza"; 
    } 
    
    @Override 
    public double getCost() { 
        return 300.0; 
    } 
} 

public class TandooriPaneerDelight implements BasePizza { 
    @Override 
    public String getDescription() { 
        return "Tandoori Paneer Delight Pizza"; 
    } 
    
    @Override 
    public double getCost() { 
        return 400.0; 
    } 
} 

public class ChickenDominator implements BasePizza { 
    @Override 
    public String getDescription() { 
        return "Chicken Dominator Pizza"; 
    } 
    
    @Override 
    public double getCost() { 
        return 500.0; 
    } 
} 
```

### Step 3: Define the Abstract Base Decorator

Java

```
public abstract class ToppingDecorator implements BasePizza { 
    BasePizza pizza; 
    
    public ToppingDecorator(BasePizza pizza) { 
        this.pizza = pizza; 
    } 
} 
```

### Step 4: Define the Concrete Decorators

Java

```
public class ExtraCheeseTopping extends ToppingDecorator { 
    public ExtraCheeseTopping(BasePizza pizza) { 
        super(pizza); 
    } 
    
    @Override 
    public String getDescription() { 
        return pizza.getDescription() + " + Extra Cheese"; 
    } 
    
    @Override 
    public double getCost() { 
        return pizza.getCost() + 20; 
    } 
} 

public class VeggiesTopping extends ToppingDecorator { 
    public VeggiesTopping(BasePizza pizza) { 
        super(pizza); 
    } 
    
    @Override 
    public String getDescription() { 
        return pizza.getDescription() + " + Veggies"; 
    } 
    
    @Override 
    public double getCost() { 
        return pizza.getCost() + 30; 
    } 
} 

public class MushroomTopping extends ToppingDecorator { 
    public MushroomTopping(BasePizza pizza) { 
        super(pizza); 
    } 
    
    @Override 
    public String getDescription() { 
        return pizza.getDescription() + " + Mushroom"; 
    } 
    
    @Override 
    public double getCost() { 
        return pizza.getCost() + 40; 
    } 
} 

public class PepperoniTopping extends ToppingDecorator { 
    public PepperoniTopping(BasePizza pizza) { 
        super(pizza); 
    } 
    
    @Override 
    public String getDescription() { 
        return pizza.getDescription() + " + Pepperoni"; 
    } 
    
    @Override 
    public double getCost() { 
        return pizza.getCost() + 50; 
    } 
} 
```

### Step 5: Client Demonstration

Java

```
public class PizzaShop { 
    public static void main(String[] args) { 
        System.out.println("======= Decorator Design Pattern ======"); 
        
        // Create a plain pizza 
        BasePizza pizza1 = new PlainPizza(); 
        System.out.println("Order 1: " + pizza1.getDescription() + " = Rs." + pizza1.getCost()); 
        
        // Add toppings to the PlainPizza - Extra Cheese Only 
        BasePizza pizza2 = new ExtraCheeseTopping(new PlainPizza()); 
        System.out.println("Order 2: " + pizza2.getDescription() + " = Rs." + pizza2.getCost()); 
        
        // Add toppings to the PlainPizza - Extra Cheese and Veggies 
        BasePizza pizza3 = new VeggiesTopping(new ExtraCheeseTopping(new PlainPizza())); 
        System.out.println("Order 3: " + pizza3.getDescription() + " = Rs." + pizza3.getCost()); 
        
        // Add toppings to the PlainPizza - Extra Cheese and Pepperoni 
        BasePizza pizza4 = new PepperoniTopping(new ExtraCheeseTopping(new PlainPizza())); 
        System.out.println("Order 4: " + pizza4.getDescription() + " = Rs." + pizza4.getCost()); 
        
        // Add toppings to the PlainPizza - Extra Cheese, Mushroom and Pepperoni 
        BasePizza pizza5 = new MushroomTopping(new PepperoniTopping(new ExtraCheeseTopping(new PlainPizza())));
        System.out.println("Order 5: " + pizza5.getDescription() + " = Rs." + pizza5.getCost()); 
        
        // Farmhouse Pizza 
        BasePizza pizza6 = new Farmhouse(); 
        System.out.println("Order 6: " + pizza6.getDescription() + " = Rs." + pizza6.getCost()); 
        
        // Farmhouse Pizza with Extra Cheese and Mushroom 
        BasePizza pizza7 = new MushroomTopping(new ExtraCheeseTopping(new Farmhouse())); 
        System.out.println("Order 7: " + pizza7.getDescription() + " = Rs." + pizza7.getCost()); 
        
        // Tandoori Paneer Delight Pizza 
        BasePizza pizza8 = new TandooriPaneerDelight(); 
        System.out.println("Order 8: " + pizza8.getDescription() + " = Rs." + pizza8.getCost()); 
        
        // Chicken Dominator 
        BasePizza pizza9 = new ChickenDominator(); 
        System.out.println("Order 9: " + pizza9.getDescription() + " = Rs." + pizza9.getCost()); 
        
        // Chicken Dominator with Mushroom 
        BasePizza pizza10 = new MushroomTopping(new ChickenDominator()); 
        System.out.println("Order 10: " + pizza10.getDescription() + " = Rs." + pizza10.getCost()); 
    } 
}
```

> **Key Takeaway:** The primary benefit of the Decorator pattern is that it provides a flexible alternative to subclassing for extending functionality, allowing you to mix and match behaviors as needed at runtime.

---
