# Chapter 2: Variables, expressions and statements

In this chapter, we will explore the fundamental components of any program in Harbour: how data is stored, how it is manipulated, and how it is combined to form instructions that the compiler can understand.

---

### Values and data types in Harbour

Harbour is a dynamically typed language, which means that a variable can change type during program execution. However, each value has a specific data type.

*   **`String` (Character string)**: Represents text. It can be delimited with single quotes (`'`), double quotes (`"`) or square brackets (`[]`).
    ```harbour
    LOCAL cName := "John Doe"
    LOCAL cAddress := 'Main Street, 123'
    LOCAL cLongText := [This is text that can contain ' and " without problems.]
    ```

*   **`Number` (Number)**: Can be an integer or decimal number. Harbour handles precision automatically.
    ```harbour
    LOCAL nAge := 30
    LOCAL nPrice := 19.95
    LOCAL nNegative := -100
    ```

*   **`Logical` (Logical)**: Represents a boolean value, which can be true (`.T.`) or false (`.F.`).
    ```harbour
    LOCAL lIsCustomer := .T.
    LOCAL lActive := .F.
    ```

*   **`Date` (Date)**: Stores a date (day, month and year). The default format is `mm/dd/yyyy`, but can be changed with `SET DATE`.
    ```harbour
    LOCAL dBirth := CTOD("12/25/1990") // Convert string to date
    LOCAL dToday := DATE() // Get current date
    ```

*   **`CodeBlock` (Code block)**: Is a compiled code fragment that can be stored in a variable and executed later. It is one of Harbour's most powerful features. It is defined with `{|parameters| expression}`.
    ```harbour
    LOCAL bMyBlock := { |x, y| x + y } // A block that adds two numbers
    AEVAL( aMyArray, { |element| QOUT(element) } ) // Evaluate a block for each element of an array
    ```

*   **`Symbol` (Symbol)**: Is an internal reference to a function or variable. It is used for indirect calls.
    ```harbour
    LOCAL symFunc := @MyFunction()
    EVAL( symFunc ) // Calls MyFunction()
    ```

*   **`Pointer` (Pointer)**: Is a reference to the memory address of another variable. It is mainly used for integration with C code.

*   **`NIL`**: Represents the absence of value. An uninitialized variable has the value `NIL`.
    ```harbour
    LOCAL x // x is NIL by default
    IF x == NIL
       QOUT("The variable has no value")
    ENDIF
    ```
---

### Variables and their scope

The scope determines where a variable is visible and accessible.

*   **`LOCAL`**: The variable only exists within the function or procedure where it is declared. This is the most recommended and safe way to work.
    ```harbour
    FUNCTION MyFunction()
       LOCAL nCounter := 0 // Only visible here
       nCounter++
       RETURN nCounter
    ```

*   **`STATIC`**: Similar to `LOCAL`, but the variable retains its value between successive calls to the same function. It is initialized only the first time the function is executed.
    ```harbour
    FUNCTION CallCounter()
       STATIC nTimes := 0 // Initialized to 0 only the first time
       nTimes++
       QOUT("This function has been called", nTimes, "times.")
       RETURN NIL
    ```

*   **`PRIVATE`**: The variable is visible in the function where it is declared and in all functions called from it. It is a mechanism inherited from Clipper and its use can generate code that is difficult to maintain.
    ```harbour
    PROCEDURE Main()
       PRIVATE cMessage := "Hello from Main"
       Subroutine()
       RETURN

    PROCEDURE Subroutine()
       QOUT(cMessage) // Displays "Hello from Main"
       RETURN
    ```

*   **`PUBLIC`**: The variable is global and accessible from anywhere in the program. Its use is discouraged, as it can cause unexpected side effects. It is preferable to use functions to get or set global values in a controlled way.
    ```harbour
    PUBLIC g_cCurrentUser
    // ... somewhere in the code
    g_cCurrentUser := "admin"
    ```

---

### Assignment statements and operators

*   **Assignment**: Used to give a value to a variable. The inline assignment operator `:=` declares and initializes the variable as `LOCAL`. The `=` operator assigns a value to an existing variable.
    ```harbour
    LOCAL nNumber
    nNumber := 10 // Declaration and inline assignment (implies LOCAL)
    nNumber = 20  // Assignment to an existing variable
    ```

*   **Compound Assignment Operators**: These operators combine an arithmetic operation with assignment, providing a more concise way to modify a variable's value.
    ```harbour
    LOCAL nValue := 10
    nValue += 5   // Equivalent to: nValue = nValue + 5 (result: 15)
    nValue -= 3   // Equivalent to: nValue = nValue - 3 (result: 12)
    nValue *= 2   // Equivalent to: nValue = nValue * 2 (result: 24)
    nValue /= 4   // Equivalent to: nValue = nValue / 4 (result: 6)
    ```

*   **Arithmetic Operators**: `+` (addition), `-` (subtraction), `*` (multiplication), `/` (division), `%` (modulo), `^` or `**` (power).
*   **String Operators**: `+` (concatenate), `-` (concatenate removing trailing spaces).
*   **Date Operators**: `+` (add days), `-` (subtract days or calculate difference between dates).
*   **Relational Operators**: `==` (exactly equal), `=` (equal), `!=` or `<>` (different), `<` (less than), `>` (greater than), `<=` (less than or equal), `>=` (greater than or equal).
*   **Logical Operators**: `.AND.` (and), `.OR.` (or), `.NOT.` (not). They are written with dots around them.
*   **Increment/Decrement Operators**: `++` (increment by 1), `--` (decrement by 1). They can be prefix (`++n`) or suffix (`n++`).

---

### Expressions

An expression is a combination of values, variables, operators and function calls that the language can evaluate to produce a value.

*   **Simple expression**:
    ```harbour
    nAge + 1
    ```

*   **Complex expression**:
    ```harbour
    ( ( nSubTotal + nTaxes ) * ( 1 - nDiscount / 100 ) ) > 0 .AND. !EMPTY(cName)
    ```

*   **Expression in a statement**:
    ```harbour
    // The expression is nPrice * 1.21
    REPLACE price WITH nPrice * 1.21

    // The expression is lCondition .AND. nValue > 10
    IF lCondition .AND. nValue > 10
       // ...
    ENDIF
    ```