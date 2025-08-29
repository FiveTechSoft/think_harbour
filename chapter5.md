# Chapter 5: Complex data types

In Harbour, in addition to simple data types like numeric, strings and logical, there are complex data types that allow structuring and handling information more efficiently. The two most important are **Arrays** and **Hashes**.

## Arrays

An array is an ordered collection of elements. Each element in the array has a position or numeric index, which in Harbour starts at 1. Arrays can contain elements of any data type, even other arrays or hashes!

### Creating Arrays

You can create an array in several ways:

```harbour
// An empty array
LOCAL aMyArray1 := {}

// An array with initial elements
LOCAL aMyArray2 := { "Apple", "Banana", "Orange" }

// Using the Array() function to create an array of a specific size
LOCAL aMyArray3 := Array(10) // Creates an array with 10 NIL elements
```

### Access and Modification

Elements are accessed through their numeric index in square brackets `[]`.

```harbour
LOCAL aFruits := { "Apple", "Banana", "Orange" }

? aFruits[1] // Shows "Apple"
? aFruits[3] // Shows "Orange"

// Modify an element
aFruits[2] := "Strawberry"
? aFruits[2] // Shows "Strawberry"
```

### Common functions for Arrays

- **`len( <array> )`**: Returns the number of elements in the array.
- **`aadd( <array>, <element> )`**: Adds a new element to the end of the array.
- **`asize( <array>, <new_size> )`**: Changes the size of the array. If larger, adds NIL elements. If smaller, truncates them.
- **`ascan( <array>, <value_or_code_block> )`**: Searches for an element and returns its index.
- **`adel( <array>, <position> )`**: Deletes an element at a specific position.
- **`ains( <array>, <position> )`**: Inserts a NIL element at a specific position.

## Hashes

A hash (also known as dictionary or map in other languages) is a collection of key-value pairs. Instead of using a numeric index, a "key" (usually a text string) is used to access its associated "value". Hashes are extremely useful for storing structured data.

### Creating Hashes

```harbour
// An empty hash
LOCAL hMyHash1 := {=>}

// A hash with initial key-value pairs
LOCAL hPerson := { "name" => "John", "age" => 30, "city" => "Madrid" }

// Using the HSet() function
LOCAL hMyHash2 := {=>}
HSet( hMyHash2, "key1", "value1" )
```

### Access and Modification

Values are accessed through their key, using the syntax `hMyHash["key"]`.

```harbour
LOCAL hPerson := { "name" => "John", "age" => 30 }

? hPerson["name"] // Shows "John"

// Modify a value
hPerson["age"] := 31

// Add a new key-value pair
hPerson["profession"] := "Programmer"

? hPerson // Shows the hash content
```

### Functions and Loops for Hashes

- **`haskey( <hash>, <key> )`**: Checks if a key exists in the hash. Returns `.T.` or `.F.`.
- **`hdel( <hash>, <key> )`**: Removes a key-value pair from the hash.
- **Iterate with `FOR EACH`**: The most common way to traverse all elements of a hash.

```harbour
LOCAL hPerson := { "name" => "Ana", "age" => 25, "city" => "Barcelona" }
LOCAL xKey, xValue

FOR EACH xKey, xValue IN hPerson
   ? xKey + ":", xValue
NEXT
```

## Differences and When to Use Each One

| Feature | Arrays | Hashes |
| :--- | :--- | :--- |
| **Access** | By numeric index (1, 2, 3...). | By key (e.g: "name"). |
| **Order** | **Ordered**. Maintain insertion order. | **Generally unordered**. Should not rely on order. |
| **Typical Use** | Lists of elements where order matters or sequential access is needed. | Structured data where each value has a name or identifier. |
| **Speed** | Index access is very fast. | Key access is very fast and efficient. |

### When to use an Array?

- For a list of elements of the same type (e.g: a list of customer names).
- When the order of elements is important (e.g: steps in a sequence).
- To represent rows of a table read from a database.

**Example:** A shopping list.
```harbour
aShopping := { "Milk", "Bread", "Eggs" }
```

### When to use a Hash?

- To represent a unique object with its properties (e.g: a customer with their name, ID, and address).
- To pass a set of named parameters to a function.
- To store application configurations or settings.

**Example:** User data.
```harbour
hUser := { "id" => 101, "user" => "admin", "access_level" => 5 }
```