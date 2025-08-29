# Chapter 14: The Harbour Data System (The Harbour item)

Harbour's data system is built upon a structure called `item` that acts as the universal container for all data types in the language. This flexible and powerful system is the heart of Harbour's virtual machine, enabling dynamic type handling and interoperability between Harbour code and C extensions.

---

## The `item`: Harbour's universal data container

In Harbour, all values - from simple numbers to complex objects - are stored internally in a data structure called `HB_ITEM`. This unified design allows Harbour to be a dynamically typed language while maintaining optimal performance.

### Internal structure of `HB_ITEM`

The `item` is a C structure that contains:

```c
typedef struct _HB_ITEM
{
   HB_TYPE type;     // Data type (HB_IT_STRING, HB_IT_NUMERIC, etc.)
   HB_ITEM_UNION;    // Union that contains the actual value
} HB_ITEM, * PHB_ITEM;
```

### Supported data types

The main data types that an `item` can contain are:

- **HB_IT_NIL**: Null value
- **HB_IT_LOGICAL**: Logical values (.T. or .F.)
- **HB_IT_INTEGER**: Integer numbers
- **HB_IT_LONG**: Long integers
- **HB_IT_DOUBLE**: Floating point numbers
- **HB_IT_DATE**: Dates
- **HB_IT_TIMESTAMP**: Timestamps
- **HB_IT_STRING**: Character strings
- **HB_IT_ARRAY**: Arrays (matrices)
- **HB_IT_HASH**: Hash tables
- **HB_IT_BLOCK**: Code blocks
- **HB_IT_SYMBOL**: Function symbols
- **HB_IT_POINTER**: Pointers to external data

---

## The `item` API: functions for manipulating values at the C level

To work with `items` from C extensions, Harbour provides a complete API of functions. These functions follow a consistent pattern for creating, accessing, and modifying values.

### Creation and assignment functions

```c
// Create a new item
PHB_ITEM hb_itemNew( PHB_ITEM pItem );

// Assign specific values
PHB_ITEM hb_itemPutNI( PHB_ITEM pItem, int iNumber );
PHB_ITEM hb_itemPutND( PHB_ITEM pItem, double dNumber );
PHB_ITEM hb_itemPutC( PHB_ITEM pItem, const char * szText );
PHB_ITEM hb_itemPutL( PHB_ITEM pItem, HB_BOOL fValue );
```

### Type query functions

```c
// Check the type of an item
HB_BOOL hb_itemIsString( PHB_ITEM pItem );
HB_BOOL hb_itemIsNumeric( PHB_ITEM pItem );
HB_BOOL hb_itemIsLogical( PHB_ITEM pItem );
HB_BOOL hb_itemIsArray( PHB_ITEM pItem );
```

### Value access functions

```c
// Get values from an item
const char * hb_itemGetCPtr( PHB_ITEM pItem );
double hb_itemGetND( PHB_ITEM pItem );
HB_BOOL hb_itemGetL( PHB_ITEM pItem );
```

---

## Scalar values: handling basic data types

Scalar types are those that contain a single simple value. Harbour optimizes the handling of these types to maximize performance.

### Numbers

Harbour automatically handles conversion between different numeric types:

```harbour
LOCAL nInteger := 42        // HB_IT_INTEGER
LOCAL nDecimal := 3.14159  // HB_IT_DOUBLE
LOCAL nCalculation := nInteger * nDecimal  // Automatic conversion
```

### Character strings

Strings in Harbour are immutable and use reference counting to optimize memory:

```harbour
LOCAL cText := "Hello Harbour"
LOCAL cCopy := cText  // String is not duplicated, only the reference
```

### Dates and timestamps

```harbour
LOCAL dDate := Date()         // HB_IT_DATE
LOCAL tNow := hb_DateTime()  // HB_IT_TIMESTAMP
```

---

## Handling `Arrays` and `Hashes`: structure and operations of complex collections

Arrays and hashes are complex data types that can contain multiple `items`.

### Arrays

Arrays in Harbour are dynamic and can contain elements of any type:

```harbour
LOCAL aList := Array(5)        // Create array of 5 elements
LOCAL aMixed := {"Text", 123, Date(), .T.}  // Mixed array
```

**Fundamental operations:**

```harbour
// Element access
? aList[1]           // First element (1-based)

// Modification
aList[2] := "New value"

// Resizing
ASize(aList, 10)     // Change size to 10 elements

// Insertion and deletion
AAdd(aList, "New element")   // Add to end
AIns(aList, 2)                  // Insert at position 2
ADel(aList, 3)                  // Delete element 3
```

### Hashes

Hashes provide associative access through keys:

```harbour
LOCAL hData := {=>}  // Empty hash
LOCAL hPerson := {"name" => "John", "age" => 30}

// Access by key
? hPerson["name"]

// Add new keys
hPerson["city"] := "Madrid"

// Iterate over keys
FOR EACH cKey IN hb_HKeys(hPerson)
   ? cKey, "=>", hPerson[cKey]
NEXT
```

---

## `Codeblocks`: the data type for executable code

Code blocks are a unique feature of Harbour that allows storing executable code as a first-class value.

### Creation and use

```harbour
LOCAL bBlock := {|x, y| x + y}  // Block that adds two parameters

// Block evaluation
LOCAL nResult := Eval(bBlock, 5, 3)  // Result: 8
```

### Blocks with local variables

```harbour
LOCAL nMultiplier := 10
LOCAL bScaler := {|x| x * nMultiplier}  // Captures the local variable

? Eval(bScaler, 5)  // Result: 50
```

### Practical applications

Code blocks are especially useful for:

- **Callbacks**: Functions that are passed as parameters
- **Filters and transformations**: Operations on arrays
- **Deferred evaluation**: Code that executes under certain conditions

```harbour
// Filter an array with a block
LOCAL aNumbers := {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
LOCAL aEvens := AEval(aNumbers, {|x| x % 2 == 0})
```

---

## `Classes and Objects`: the implementation of object-oriented programming

In Harbour, classes and objects are implemented on top of the `item` system, providing a flexible and powerful OOP model.

### Internal structure of a class

```harbour
// Class definition
CREATE CLASS Person
   VAR cName
   VAR nAge
   
   METHOD New(cNam, nAg) CONSTRUCTOR
   METHOD Greet()
   METHOD SetName(cNewName)
ENDCLASS

// Method implementation
METHOD New(cNam, nAg) CLASS Person
   ::cName := cNam
   ::nAge := nAg
   RETURN Self

METHOD Greet() CLASS Person
   RETURN "Hello, I'm " + ::cName
```

### Inheritance and polymorphism

```harbour
CREATE CLASS Employee FROM Person
   VAR cCompany
   VAR nSalary
   
   METHOD New(cNam, nAg, cComp, nSal) CONSTRUCTOR
   METHOD Work()
   METHOD Greet()  // Redefinition of parent method
ENDCLASS

METHOD Greet() CLASS Employee
   RETURN ::Super:Greet() + " and I work at " + ::cCompany
```

---

## Pointers and symbols: references to variables and functions

### Symbols

Symbols represent references to functions and variables in the global namespace:

```harbour
LOCAL sFunc := @MyFunction()  // Symbol pointing to MyFunction
LOCAL result := Eval(sFunc, parameters)  // Call through symbol
```

### Pointers

Pointers allow references to external data or C structures:

```harbour
// From a C extension
FUNCTION GetPointer()
   LOCAL pData := hb_PointerFromString("external data")
   RETURN pData

// Using the pointer
LOCAL pMyPointer := GetPointer()
? hb_PointerToString(pMyPointer)
```

---

## Optimizations of the `item` system

### Reference counting

Harbour uses reference counting to optimize memory usage, especially with strings and arrays:

```harbour
LOCAL cText1 := "Long string with lots of content"
LOCAL cText2 := cText1  // Only increments the reference counter
```

### Object pool

For frequently used types like small numbers and logical values, Harbour maintains a pool of reusable objects.

### Automatic conversions

The `item` system allows automatic conversions between compatible types:

```harbour
LOCAL cNumber := "123"
LOCAL nResult := cNumber + 45  // Automatic conversion: "123" -> 123
```

This unified design of the data system makes Harbour both flexible and efficient, enabling the development of robust applications with exceptional performance.