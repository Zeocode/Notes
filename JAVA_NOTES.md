# Java Notes

## Classes & Objects
- Class = blueprint (variables + methods); Object = physical reality in memory
- `new` allocates memory at runtime and returns a reference
- `Box b2 = b1;` → both refer to the **same** object
- Inside methods, instance variables need no dot operator

## Constructors
- Same name as class, implicit return type (no explicit return type allowed)
- Cannot be called like a regular method
- **Default constructor** auto-created if none exists → initializes `int/long` to `0`, `char` to `\u0000`, references to `null`
- **Parameterized constructor** → custom initialization

## Method Overloading
- Same name, different number or type of parameters
- Return type plays no role
- Java performs auto type conversion: `byte → int → long → float → double`

## Call by Value vs Reference
- Primitives → passed by value (original unaffected)
- Objects → reference is passed (changes affect original)

## Access Modifiers
| Modifier | Accessible From |
|---|---|
| `public` | Everywhere |
| `protected` | Same package + subclasses |
| default | Same package only |
| `private` | Same class only |

> Variables are generally `private`; use getters for controlled access.

## Static
- Member independent of any object instance
- Static methods: can only directly call other static methods/data
- Cannot use `this` or `super`
- Initialize via: static block, direct declaration, or private static method
- Outer class can never be static

## final
- `final` variable → constant (use CAPS)
- `final` method → prevents overriding; resolved at compile time (early binding)
- `final` class → prevents inheritance

## Garbage Collection
- Object with no reference → eligible for GC, destroyed automatically
- `finalize()` called before destruction; used to release resources
  ```java
  protected void finalize() { }
  ```

## Nested & Inner Classes
- Inner class scope is bounded by its enclosing class
- Nested class can access outer members; outer cannot access nested members
- Inner object exists only in context of the outer class
- **Static nested class** → seldom used
- **Non-static inner class** → requires outer class object
- Inner classes inside method blocks → useful for event handling

## String Class
- Immutable; string literals stored in the **constant pool** (not heap)
- Modification creates a new object
- Use `StringBuilder` / `StringBuffer` for mutable concatenation
- Common methods: `equals()`, `length()`, `charAt()`
- `+` is the concatenation operator

## Command Line Arguments
```java
main(String args[])
// args.length → count; args[0], args[1]… → access each
```

## Inheritance (is-a)
- Subclass inherits all non-private superclass members
- No multiple inheritance (a class can extend only one class)
- Superclass reference can hold a subclass object (polymorphism)
- `super()` → calls superclass constructor (auto-inserted if omitted)
- `super.member` → accesses hidden/overridden superclass member

## Method Overriding
- Subclass defines a method with the **same name and signature**
- Subclass version executes; use `super.method()` to call parent version explicitly

## Dynamic Method Dispatch
- Overridden method calls resolved at **runtime**
- Superclass reference → subclass object → correct version called based on actual type
- Enables **runtime polymorphism**

## Abstract Classes
- Marked with `abstract`; cannot be instantiated directly
- Can have both abstract and concrete methods
- No abstract constructors or abstract static methods (regular constructors are allowed)
- Subclass must implement all abstract methods, or declare itself abstract
- Can be used to create reference variables

## Object Class
- All classes implicitly extend `java.lang.Object`
- Key inherited methods: `getClass()`, `clone()`, `equals()`, `hashCode()`, `toString()`

## Packages
```java
package pkg1[.pkg2][.pkg3];
import pkg1.ClassName;
import pkg1.*;
```
- Package = container for classes and sub-packages

## Interfaces
- Pure abstraction; blueprint of shared functionality
- Variables: implicitly `public static final`
- Methods: implicitly `public abstract` (before Java 8)
- Supports multiple inheritance: `class A implements I1, I2 { }`
- Partial implementation → class must be declared abstract
- Enables runtime polymorphism via dynamic method lookup

### Nested Interfaces
- Declared inside a class or another interface
- Can be `public`, `private`, or `protected`
- Access: `A.InterfaceName`; Implement: `class B implements A.InterfaceName`

### JDK 8 Interface Features
- **Default methods** → provide a default implementation; prevent breakage on interface evolution
- **Static methods** → utility methods; called as `InterfaceName.staticMethod()`

#### Default Method Conflict Rules
| Situation | Result |
|---|---|
| Class has its own method | Class wins |
| Neither interface has a default | No conflict |
| Both interfaces have same default | Class **must** override; resolve with `A.super.method()` |

## Binding
- **Early binding** → `final` methods, resolved at compile time
- **Late binding** → overridden methods, resolved at runtime
- Loose coupling achieved via abstract classes or interfaces
