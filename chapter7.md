# Chapter 7: Classes and objects

Object-Oriented Programming (OOP) is a programming paradigm that uses "objects" to design applications and software programs. Harbour, although it is a language with roots in xBase, has incorporated a powerful OOP system that is compatible with Clipper Class(y) syntax.

This chapter will guide you through the fundamental concepts of OOP and how to apply them in Harbour.

## OOP concepts: classes, objects, variables and methods

Before diving into Harbour syntax, it's crucial to understand the pillars of OOP:

*   **Class:** A class is a template or blueprint for creating objects. It defines a set of attributes (variables) and behaviors (methods) that will characterize objects created from it. For example, a `Car` class could define that all cars have a `brand`, a `model` and can `accelerate()` and `brake()`.

*   **Object:** An object is an instance of a class. If `Car` is the class, an object could be *myCar* specific, with `brand = "Ford"` and `model = "Mustang"`. You can create many objects from the same class, each with their own attribute values.

*   **Instance Variable (or Attribute):** These are variables that belong to a specific object. Each instance of a class has its own copy of these variables. In Harbour, they are declared with the keyword `VAR`. They are often called "data members".

*   **Method:** These are functions that belong to a class and define the behaviors of its objects. Methods operate on the data (instance variables) of the object. For example, an `accelerate()` method in the `Car` class could modify the `currentSpeed` variable. In Harbour, they are declared with `METHOD`.

## Creating a class in Harbour

In Harbour, classes are defined using the syntax `CREATE CLASS ... ENDCLASS`. The basic structure is as follows:

```harbour
CREATE CLASS MyClass

   // Instance variables (attributes)
   VAR sName
   VAR nAge

   // Methods
   METHOD New( cName, nAgeParam ) CONSTRUCTOR
   METHOD Greet()
   METHOD HaveBirthday()

ENDCLASS

/*
 * Implementation of methods
 */

// Constructor: called when creating an object.
METHOD New( cName, nAgeParam ) CLASS MyClass
   ::sName := cName
   ::nAge  := nAgeParam
   RETURN Self // The constructor must return Self

// Method to display a greeting
METHOD Greet() CLASS MyClass
   ? "Hello, my name is", ::sName
   RETURN Nil

// Method to increment age
METHOD HaveBirthday() CLASS MyClass
   ::nAge++
   ? "Happy birthday! I am now", ::nAge, "years old."
   RETURN Nil
```

**Key points:**

*   `CREATE CLASS ClassName ... ENDCLASS`: Defines the beginning and end of the class declaration.
*   `VAR variableName`: Declares an instance variable. By convention, a prefix like `s` for string, `n` for numeric, etc. is often used.
*   `METHOD methodName(...)`: Declares a method.
*   `CONSTRUCTOR`: Marks a method as the class constructor. This method is executed automatically when creating a new instance of the object.
*   `METHOD ... CLASS ClassName`: Actual implementation of the method. It is done outside the `CREATE CLASS` block.
*   `::`: Is the instance scope operator. It is used to access variables and methods of the current object (similar to `this` in other languages).
*   `Self`: Is a reference to the object itself, which must be returned by the constructor.

## Object instantiation

Once you have a class, you can create objects (instances) from it. This is done using the syntax `ClassName():New(...)` or `ClassNameNew(...)`.

```harbour
PROCEDURE Main()

   // Create an instance of the MyClass class
   LOCAL oPerson1 := MyClass():New( "John Doe", 30 )

   // Create another instance
   LOCAL oPerson2 := MyClass():New( "Ana Lopez", 25 )

   // Use the object methods
   oPerson1:Greet()       // Output: Hello, my name is John Doe
   oPerson2:Greet()       // Output: Hello, my name is Ana Lopez

   oPerson1:HaveBirthday()  // Output: Happy birthday! I am now 31 years old.

   // Access an instance variable
   ? "Ana's age:", oPerson2:nAge  // Output: Ana's age: 25

   WAIT // Wait for user to press a key

RETURN
```

In this example:

*   `MyClass():New(...)` creates a new object. The arguments passed to `New()` are those received by the constructor method.
*   `oPerson1` and `oPerson2` are two independent objects, each with their own data (`sName` and `nAge`).
*   `:` is the message sending operator. It is used to call a method or access a variable of an object (`oPerson1:Greet()`).

## Inheritance

Inheritance is a mechanism that allows a class (called child class or subclass) to inherit attributes and methods from another class (called parent class or superclass). This promotes code reuse.

In Harbour, inheritance is specified with the `FROM` or `INHERIT` clause in the class declaration.

```harbour
// Parent Class
CREATE CLASS Vehicle
   VAR nWheels
   VAR cColor

   METHOD New( nWheels, cColor ) CONSTRUCTOR
   METHOD Describe()
ENDCLASS

METHOD New( nWheels, cColor ) CLASS Vehicle
   ::nWheels := nWheels
   ::cColor  := cColor
   RETURN Self

METHOD Describe() CLASS Vehicle
   ? "This is a", ::nWheels, "wheel vehicle in", ::cColor, "color"
   RETURN Nil

// Child Class that inherits from Vehicle
CREATE CLASS Car FROM Vehicle
   VAR sBrand

   METHOD New( cColor, sBrand ) CONSTRUCTOR
   METHOD Describe() // Overriding the parent method
ENDCLASS

METHOD New( cColor, sBrand ) CLASS Car
   // Call the parent constructor using Super:
   Super:New( 4, cColor )
   ::sBrand := sBrand
   RETURN Self

METHOD Describe() CLASS Car
   // You can call the original parent method
   Super:Describe()
   ? "   -> It's a car of brand:", ::sBrand
   RETURN Nil

/*
 * Using classes with inheritance
 */
PROCEDURE Main()
   LOCAL oMyCar := Car():New( "Red", "Ferrari" )

   oMyCar:Describe()

   WAIT
RETURN
```

**Program output:**
```
This is a 4 wheel vehicle in Red color
   -> It's a car of brand: Ferrari
```

**Key inheritance points:**

*   `CREATE CLASS Car FROM Vehicle`: The `Car` class inherits from `Vehicle`.
*   The `Car` class has access to `nWheels` and `cColor` from `Vehicle`, in addition to its own `sBrand` attribute.
*   **Method overriding:** The `Car` class redefines the `Describe()` method. When `oMyCar:Describe()` is called, the `Car` version is executed.
*   `Super:`: Is a keyword that allows calling methods of the parent class from the child class. `Super:New(...)` invokes the constructor of `Vehicle`, and `Super:Describe()` invokes the original `Describe()` method of the parent. This is very useful for extending functionality instead of completely replacing it.

This chapter provides a solid foundation to start working with object-oriented programming in Harbour. Mastering these concepts will allow you to create more modular, reusable and easy-to-maintain code.