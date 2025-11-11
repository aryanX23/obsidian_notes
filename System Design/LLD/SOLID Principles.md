
# What does SOLID Stands for?
- S - Single Responsibility Principle
- O - Open/Closed Principle
- L - Liskov Substitution Principle
- I - Interface Segregation Principle
- D - Dependency Inversion Principle

	Advantages of using SOLID Design Principle - 
	- Help us write better quality code in general
	- Avoid Duplicate Code
	- Easy to maintain code
	- Easy to Understand
	- Flexible Software
	- Reduce Complexity

# Single Responsibility Principle (SRP)

It states that *A class should have only one reason to change*, that is it should have `ONE AND ONLY` job or responsibility.

Lets take an example of a code that violates this principle - 

``` java
public class Invoice{
	private Snacks snack;
	private int quantity;
	
	public Invoice(Snacks snack, int quantity){
		this.snack = snack;
		this.quantity = quantity;
	}
	
	// Responsibility 1 -
	public int totalValue(){
		return this.snack.price * this.quantity;
	}
	
	// Responsibility 2 - 
	public void printInvoice(){
		// Assume Implemented Logic for printing invoice via Fax/Printer, etc
	}
	
	// Responsibility 3 - 
	public void saveToDatabase(){
		// Assume Implemented Logic for Saving to Database
	}
}
```

Problems with the Above Code - 
- Suppose, there is a feature ask by a stakeholder which goes as follows - "I want a functionality to send a copy of invoice to the customer via email or phone."
- Now to insert this functionality, one has to modify the above class which would mean re-testing the entire class functionality and bottlenecking the entire QA bandwidth.
- To avoid this, we can structure the code in such a way that each class only has once responsibility to implement so that a already tested classes need not be modified.

Let take an example of the same code but not obeying *Single Responsibility Principle* -

``` java
public class Invoice{
	private Snacks snack;
	private int quantity;
	
	public Invoice(Snacks snack, int quantity){
		this.snack = snack;
		this.quantity = quantity;
	}
}

public class SaveInvoiceToDatabase{
	private Invoice invoice;
	
	SaveInvoiceToDatabase(Invoice invoice){
		this.invoice = invoice;
	}
	
	public void saveToDatabase{
		// Save to Database logic
	}
}

public class SendInvoice{
	private Invoice invoice;
	
	SendInvoice(Invoice invoice){
		this.invoice = invoice;
	}
	
	public void sendInvoiceByEmail(String email){
		// Implements logic for sending invoice via email
	}
	
	public void sendInvoiceByPhone(String phone, String phoneCode){
		// Implements logic for sending invoice via sms
	}
}
```

- Now, each responsibility is segregated making sure an additional feature addition in the future does not disturb the code of the Invoice class.

## Key Benefits
- Single Responsibility ensures each class has only *one reason to change*
- Code Maintainability is further improved as *change* to one feature *does not affect* the other
- Testability is Improved as each class can now be *tested in isolation* from one other
- Reusability is Enhanced as classes *can be re-used* in *different contexts* without the *features being tightly coupled* in single unit 
---
# Open/Closed Principle (OCP)

It states that `A class should be open for extension but closed for modification`.

This means that new functionality can be added through *inheritance* rather than directly modifying existing code.

As existing code is already tested and being used in production, adding modifications would introduce an additional risk of breaking production which would in-turn require re-testing effort.

Lets take an example of a code violating this Principle - 

``` java
public class SendInvoice{
	private Invoice invoice;
	private String email;
	
	SendInvoice(Invoice invoice, String email){
		this.invoice = invoice;
		this.email = email;
	}
	
	public void sendInvoice(){
		// Invoice sending logic via email
	}
}
```

- Now in the above code, if we were to introduce a functionality to send invoice via SMS, we would need to do nomenclature changes and possibly changes to the pre-existing code structure.

Now Lets see a code correcting this issue with a simple change - 

``` java
public interface SendInvoice{
	public void sendInvoice();
} 

public class SendInvoiceViaEmail implements SendInvoice{
	private Invoice invoice;
	private String email;
	
	SendInvoiceViaEmail(Invoice invoice, String email){
		this.invoice = invoice;
		this.email = email;
	}
	
	@override
	public void sendInvoice(){
		// Implement Logic for sending invoice via email
	}
}

public class SendInvoiceViaSms implements SendInvoice{
	private Invoice invoice;
	private String phone;
	private String phoneCode;
	
	SendInvoiceViaEmail(Invoice invoice, String phone, String phoneCode){
		this.invoice = invoice;
		this.phone = phone;
		this.phoneCode = phoneCode;
	}
	
	@override
	public void sendInvoice(){
		// Implement Logic for sending invoice via SMS
	}
}
```

- By using the above code, we can easily implement even more methods to send invoice via different mediums without making additions to existing code.

## Key Benefits -
- Reduced Risk of breaking existing code
- Better Maintainability as new features will not break the existing ones
- Improved overall code flexibility
- Enhanced testability of the code
- Supports polymorphism by enabling dynamic behavior through interfaces and inheritance.

--- 

# Liskov Substitution Principle (LSP)

It states that the `objects of a superclass(parent) should be replaceable with objects of its subclasses without breaking the applivation`.

- This means that if class B is subclass of parent class A, then we should be able to replace the object of class A with object of class B without breaking the behavior of the program.
- Subclass should *always* extend the capability of the parent class, not narrow it down.

Lets take an example of a code that breaks this Principle - 

``` java
public interface Bike {
	void turnOnEngine();
	
	void turnOffEngine();
	
	void accelerate();
	
	void applyBrakes();
}

public class Motorcycle implements Bike{
	private String company;
	private int maxSpeed;
	
	Motorcycle(String company, int maxSpeed){
		this.company = company;
		this.maxSpeed = maxSpeed;
	}
	
	void turnOnEngine(){
		System.out.println("Turning Engine On --");
	};
	
	void turnOffEngine(){
		System.out.println("Turning Engine Off --");
	};
	
	void accelerate(){
		System.out.println("accelerate --");
	};
	
	void applyBrakes(){
		System.out.println("Applying brakes --");
	};
}

public class Bicycle implements Bike{
	private String company;
	private int maxSpeed;
	
	Bicycle(String company, int maxSpeed){
		this.company = company;
		this.maxSpeed = maxSpeed;
	}
	
	void turnOnEngine(){
		throw new AssertionError("Bicycle does not have an engine!");
	};
	
	void turnOffEngine(){
		throw new AssertionError("Bicycle does not have an engine!");
	};
	
	void accelerate(){
		System.out.println("accelerate --");
	};
	
	void applyBrakes(){
		System.out.println("Applying brakes --");
	};
}

public class Main{
	public static void main(String[] args){
		Bike ob1 = new Motorcycle("BMW", 300);
		Bike ob2 = new Bicycle("B-twin", 50);
		
		ob1.turnOnEngine(); // No error - The code perform the intended function
		ob2.turnOnEngine(); // throw assertion error as bicycle does not have that functionality
	}
}
```

- In the above example since the functionality engine is not common in both bike types, it cannot implement the functions specific to engine.
- The above code can easily be corrected by separating out functions dependent on engine

Example for corrected code - 

``` java
abstract class Bike {
	public abstract void accelerate();
	
	public abstract void applyBrakes();
}

public interface Engine {
	void turnOnEngine();
	
	void turnOffEngine();
}

public class Motorcycle extends Bike implements Engine{
	private String company;
	private int maxSpeed;
	
	Motorcycle(String company, int maxSpeed){
		this.company = company;
		this.maxSpeed = maxSpeed;
	}
	
	@override
	void turnOnEngine(){
		System.out.println("Turning Engine On --");
	};
	
	@override
	void turnOffEngine(){
		System.out.println("Turning Engine Off --");
	};
	
	@override
	void accelerate(){
		System.out.println("accelerate --");
	};
	
	@override
	void applyBrakes(){
		System.out.println("Applying brakes --");
	};
}

public class Bicycle extends Bike{
	private String company;
	private int maxSpeed;
	
	Bicycle(String company, int maxSpeed){
		this.company = company;
		this.maxSpeed = maxSpeed;
	}
	
	@override
	void accelerate(){
		System.out.println("accelerate --");
	};
	
	@override
	void applyBrakes(){
		System.out.println("Applying brakes --");
	};
}

public class Main{
	public static void main(String[] args){
		Bike ob1 = new Motorcycle("BMW", 300);
		Bike ob2 = new Bicycle("B-twin", 50);
		
		ob1.turnOnEngine(); // No error - The code perform the intended function
		ob2.turnOnEngine(); // No such function exists as it is not implemented
	}
}
```

- The above code removes the unnecessary functionality that is not needed for Bicycle class.

---

# Interface Segregation Principle (ISP)

It states that `Interfaces should be such that the client should NOT implement unnecessary functions that they do not need`.

Lets take an example of a code that breaks this Principle - 

``` java
public interface HotelEmployee{
	void prepareFood();
	
	void cleanRooms();
}

public class Chef implements HotelEmployee{
	@override
	public void prepareFood(){
		System.out.println("Prepares food...");
	}
	
	@override
	public void cleanRooms(){
		throw new AssertionError("Chef cant clean rooms");
	}
}

public class Janitor implements HotelEmployee{
	@override
	public void prepareFood(){
		throw new AssertionError("Janitor cant cook food");
	}
	
	@override
	public void cleanRooms(){
		System.out.println("Cleans Rooms...");
	}
}
```

- Both the Janitor and the Chef classes are forced to implement properties they do not support due to defining everything into a single FAT interface.

Now Lets take an example that solved this problem - 

``` java
public interface ChefTasks{
	void prepareFood();
}

public interface JanitorTasks{
	void cleanRooms();
}

public class Chef implements ChefTasks{
	@override
	public void prepareFood(){
		System.out.println("Prepares food...");
	}
}

public class Janitor implements JanitorTasks{
	@override
	public void cleanRooms(){
		System.out.println("Cleans Rooms...");
	}
}
```

- The above `segreagation` of interfaces for both Chef and Janitor helps to overcome this limitation.
- LSP is different from Interface Segregation Principle because LSP focuses on keep the behavior consistent so that child class using object of parent class do not break the program when called with a already existing functionality in parent. The functionality can be overridden but not left unimplemented.
- ISP on the other hand focuses on segregating responsibilities for different classes by segregating interfaces so that no class has to unnecessarily implement functions that they do not need.

---

# Dependency Inversion Principle (DIP)

It states that `high-level components should not depend upon low-level components directly rather, they should depend on abstraction`.

Lets understand what this means with the help of an example - 

``` java
public interface Keyboard{
	void getSpecs();
}

// Low Level Implementation 1
public class WiredKeyboard implements Keyboard{
	private final String connectionType;
	private final String company;
	
	WiredKeyboard(String company){
		this.connectionType = "Wired;
		this.company = company;
	}
	
	public void getSpecs(){
		System.out.println("THIS IS A WIRED KEYBOARD =====>");
		System.out.println("Connection Type => " + this.connectionType + " and company => " + this.company);
	}
}

// Low Level Implementation 1
public class WirelessKeyboard implements Keyboard{
	private final String connectionType;
	private final String company;
	
	WirelessKeyboard(String company){
		this.connectionType = "Wireless";
		this.company = company;
	}
	
	public void getSpecs(){
		System.out.println("THIS IS A WIRELESS KEYBOARD =====>");
		System.out.println("Connection Type => " + this.connectionType + " and company => " + this.company);
	}
}

public class Laptop{
	private final WiredKeyboard keyboard; // This code here creates tight coupling
	// In future if we want to introduce a wireless keyboard and accept its value via constructer, it will directly change the type and number of dependencies which will break the future code
	
	Laptop(WiredKeyboard keyboard){
		this.keyboard = keyboard;
	}
}

public class Main{
	public static void main(String[] args){
		Laptop HP = new Laptop(new WiredKeyboard("HP"));
		Laptop Apple = new Laptop(new WirelessKeyboard("Apple")); // Will throw error as it accepts only object of WiredKeyboard as input
	}
}
```

Solution ?? -

``` java
public class Laptop{
	private final Keyboard keyboard; // Since Keyboard is a higher level components, this can accept objects of both Wired as well as Wireless Keyboard
	
	Laptop(Keyboard keyboard){
		this.keyboard = keyboard;
	}
}

public class Main{
	public static void main(String[] args){
		Laptop HP = new Laptop(new WiredKeyboard("HP"));
		Laptop Apple = new Laptop(new WirelessKeyboard("Apple"));
		
		// Now both are possible using a single constructor call which is ideal
	}
}
```

---
