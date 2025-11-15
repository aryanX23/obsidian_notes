	Definition: Majorly Focuses on classes and objects within a system
Goal is to - 
- Write *Clean* Code.
- Code should be *flexible* and *maintainable*.
- Code should be *easy to test*.

All of these below design patterns aims to enforce [[SOLID Principles]] throughout the program.
# Low Level Design Pattern Categories

## Creational Design Patterns

- It *controls the object creation*, aka, how the objects of a class should be created.
- Various Types of [[Creation Patterns]] include - 
	- [[Singleton Design Pattern]]
	- [[Builder Design Pattern]]
	- [[Factory and Abstract Factory Design Pattern]]
	- [[Object Pool Design Pattern]]
	- [[Prototype Design Pattern]]

## Structural Design Pattern

- It focuses on, how different classes/objects are arranged so that larger problems can be solved in most flexible way. It gives your code a *Skeleton* for the code to be written upon.
- Various types of [[Structural Patterns]] include - 
	- [[Decorator Design Pattern]]
	- [[Proxy Design Pattern]]
	- [[Composite Design Pattern]]
	- [[Adapter Design Pattern]]
	- [[Bridge Design Pattern]]
	- [[Facade Design Pattern]]
	- [[Flyweight Design Pattern]]

## Behavioral Design Pattern

- It focuses on how different objects communicate or interact with each other.
- In other words, with the above [[Structural Patterns]] we created the *skeleton on the system* but how that skeleton behaves (co-ordination, responsibility, interaction) is all guided by [[Behavioral Patterns]].
- Various Types of Behavioral Design Patterns include - 
	- [[State Design Pattern]]
	- [[Strategy Design Pattern]]
	- [[Observer Design Pattern]]
	- [[Chain of Responsibility Design Pattern]]
	- [[Template Design Pattern]]
	- [[Iterator Design Pattern]]
	- [[Interpreter Design Pattern]]
	- [[Command Design Pattern]]
	- [[Visitor Design Pattern]]
	- [[Mediator Design Pattern]]
	- [[Memento Design Pattern]]
	- [[Null Object Design Pattern]]
---
# Types of Relationships

There are mainly two types of relationships between classes/objects - 
- IS-A Relationship
- HAS-A Relationship

## IS-A Relationship (Inheritance)

This type of Relationship means that there is some kind of *inheritance* between two classes - 

For Example - 

Suppose there is a class Vehicle and there is a class SUV - 

```
class Vehicle{
	private String type;
	private String color;
	
	Vehicle(String type, String color){
		this.type = type;
		this.color = color;
	}
	
	public String getType(){
		return this.type;
	}
	
	public String getColor(){
		return this.color;
	}
}

class SUV extends Vehicle{
	SUV(String color){
		super("SUV", color);
	}
}
```

In the above example, the SUV class `IS A` child of Vehicle class, that is - Vehicle is a parent of SUV.
This is called as a `IS A` Relationship.

The following can be denoted using - 

![[Inheritance.png]]
	A Straight Line followed by a *simple arrow* at the end, with the arrow pointing towards the parent from the child.
## HAS-A Relationship (Association)

![[has-a-relationship.png]]
This is a relationship which shows the *association/link* between different classes/objects. There are two types of Associations between classes/objects possible - 

### Weak Relationship (Aggregation)

- In this type of Relationship, the existence is not dependent upon the other, for example: Library has books but books can exist without a library.
- They are denoted using this symbol - 
  ![[AGGREAGTION.png]]
	A Straight Line followed by a *Empty Diamond* at the end.
### Strong Relationship (Composition)

- Here, Existence of one object is dependent on the other, for example - House has rooms. Here *rooms* cannot exist without a *house*, hence both have strong dependence on the other.
- It can be denoted as - 
  ![[Composition.png]]
	A Straight Line followed by a *Filled Diamond* at the end.

---