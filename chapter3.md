# Chapter 3: Functions

Functions and procedures are the fundamental building blocks of any program in Harbour. They allow organizing code into logical, reusable, and easy-to-maintain units.

---

### Procedures (`PROCEDURE`) vs. Functions (`FUNCTION`)

In Harbour, both `PROCEDURE` and `FUNCTION` are used to define subroutines. Although they are very similar, the main conceptual difference lies in whether they return a value or not.

*   **`FUNCTION`**: A function is a subroutine designed to perform a specific task and **return a value** to the code that called it. The value is returned using the `RETURN` statement. If no value is explicitly returned, the function returns `NIL`.

    ```harbour
    // Define a function that adds two numbers and returns the result
    FUNCTION Add( n1, n2 )
       LOCAL nResult
       nResult := n1 + n2
    RETURN nResult

    // Call the function and store the returned value
    LOCAL nMySum
    nMySum := Add( 5, 10 ) // nMySum will contain 15
    ? nMySum
    ```

*   **`PROCEDURE`**: A procedure is a subroutine that performs a series of actions but **does not return an explicit value**. Its purpose is to execute a process, such as printing to screen or modifying data.

    ```harbour
    // Define a procedure to display a greeting
    PROCEDURE Greet( cName )
       ? "Hello, " + cName + "!"
    RETURN

    // Call the procedure
    Greet( "World" ) // Will print "Hello, World!"
    ```

**Key conclusion:** In modern Harbour, the distinction is mainly semantic. You can use `FUNCTION` for everything and simply omit the `RETURN` value if it's not necessary. However, using the correct term makes the code's intent clearer.

---

### Parameters and Arguments

Parameters are local variables that receive the values (arguments) passed to a function or procedure.

*   **Declaration**: Parameters are declared in a comma-separated list in the function definition.
*   **Pass by value (default)**: By default, Harbour passes arguments by value. This means the function receives a copy of the original data. If the function modifies the parameter, the original variable remains unchanged.
*   **Pass by reference**: To modify the original variable, you can pass it by reference using the `@` operator before the variable name when calling the function. Inside the function, the parameter will be a reference to the original variable.

```harbour
FUNCTION ModifyValue( nValueByReference, nValueByCopy )
   nValueByReference += 100 // Modifies the original
   nValueByCopy      += 100 // Modifies only the local copy
RETURN NIL

LOCAL nA := 10, nB := 10

? "Original values:", nA, nB // Prints: 10, 10

ModifyValue( @nA, nB ) // nA is passed by reference, nB by value

? "Modified values:", nA, nB // Prints: 110, 10
```

*   **Optional parameters**: Harbour allows calling a function with fewer arguments than declared. Unreceived parameters will contain `NIL`. The `PCOUNT()` function can be used to know how many parameters were actually passed.

```harbour
FUNCTION Message( cText, cTitle )
   IF PCOUNT() < 2
      cTitle := "Notice" // Assign a default value if no title was passed
   ENDIF
   ? cTitle + ": " + cText
RETURN NIL

Message( "Operation completed." ) // Prints: "Notice: Operation completed."
Message( "Fatal error.", "Error" )  // Prints: "Error: Fatal error."
```

---

### Input/Output (I/O) Functions

I/O functions are essential for user interaction. Harbour provides a set of simple functions to handle the console.

*   `? <expression>`: Prints one or more expressions to the console, followed by a line break.
*   `QOUT(<expression>)` or `?? <expression>`: Prints an expression without the final line break.
*   `ACCEPT "<prompt>" TO <variable>`: Displays a message and waits for the user to enter a character string, which will be saved in the variable.
*   `INPUT "<prompt>" TO <variable>`: Similar to `ACCEPT`, but evaluates the input as a Harbour expression. It is less secure and its use is not recommended.
*   `INKEY(0)`: Waits for the user to press a key and returns its ASCII numeric code. It is useful for menus or pauses.
*   `WAIT "<prompt>"`: Displays a message and pauses execution until a key is pressed.

```harbour
LOCAL cName

// Ask the user for their name
ACCEPT "Please enter your name: " TO cName

// Greet the user
? "Welcome,", cName

// Wait for the user to press a key to exit
WAIT "Press any key to continue..."
```

---

### Utility Functions (`VALTYPE()`, `PROCNAME()`)

Harbour includes a rich set of built-in functions that provide information about the program state and the data being handled.

*   **`VALTYPE( <expression> )`**: Returns a single-character string indicating the data type of the expression. It is fundamental for parameter or variable validation.
    *   `"C"`: String (Character)
    *   `"N"`: Number
    *   `"L"`: Logical
    *   `"D"`: Date
    *   `"A"`: Array
    *   `"H"`: Hash
    *   `"B"`: Codeblock
    *   `"O"`: Object
    *   `"S"`: Symbol
    *   `"P"`: Pointer
    *   `"U"`: Undefined (`NIL`)

*   **`PROCNAME( [nLevel] )`**: Returns the name of the current function or procedure. If a level is provided, it can inspect the call stack to know which function called the current one.

```harbour
FUNCTION MyRoutine( xParameter )
   LOCAL cDataType

   // Get the current function name
   ? "Executing function:", PROCNAME()

   // Validate the parameter type
   cDataType := VALTYPE( xParameter )

   IF cDataType == "C"
      ? "The parameter is a string:", xParameter
   ELSEIF cDataType == "N"
      ? "The parameter is a number:", xParameter
   ELSE
      ? "The parameter is of an unexpected type:", cDataType
   ENDIF
RETURN NIL

// Example calls
MyRoutine( "Hello" )
MyRoutine( 123.45 )
MyRoutine( .T. )
```

Below are examples for the `Symbol` and `Pointer` types, which are more advanced.

```harbour
// Example of VALTYPE() with Symbol and Pointer

FUNCTION ReturnNumber()
RETURN 42

PROCEDURE Main()
   LOCAL pFunc // Function pointer
   LOCAL symVar // Symbol

   // --- Example with Pointer ---
   // Assign a reference (pointer) to the ReturnNumber() function
   pFunc := @ReturnNumber()

   ? "The type of pFunc is:", VALTYPE( pFunc ) // Prints: P
   ? "Executing the pointer:", EVAL( pFunc )   // Prints: 42

   ?
   
   // --- Example with Symbol ---
   // A symbol represents an identifier. Used in advanced functions.
   // Here we create a symbol from a variable name.
   symVar := hb_SymbolGet( "pFunc" )
   
   ? "The type of symVar is:", VALTYPE( symVar ) // Prints: S
   ? "Symbol name:", hb_SymbolName( symVar ) // Prints: PFUNC
   
RETURN
```