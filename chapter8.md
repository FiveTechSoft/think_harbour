# Chapter 8: Error Handling and Debugging

A fundamental part of robust software development is the ability to anticipate, handle, and correct errors. Harbour, as a descendant of Clipper, inherits a powerful error handling system that has been modernized with more structured language constructs.

## Types of Errors

In Harbour development, we can classify errors into three main categories:

### 1. Syntax Errors

These errors occur in the compilation phase. They are violations of the grammatical rules of the Harbour language. The Harbour compiler will detect these errors and stop the compilation process, reporting the error and the approximate line number where it is found.

**Common causes:**
- Misspelled keywords (`FUNCTON` instead of `FUNCTION`).
- Missing parentheses, quotes, or closing blocks (`ENDIF`, `NEXT`, `END`).
- Incorrect use of operators.

**Example:**
```harbour
// Syntax error: missing closing parenthesis in function call
? SUBSTR("Hello World"
```
The compiler will emit an error and will not generate an object file (.obj) for this code.

### 2. Logic Errors

These are the most difficult errors to detect, since the program compiles and runs without failing, but does not produce the expected result. The error lies in the logic or algorithm implemented by the programmer.

**Common causes:**
- Incorrect conditions in `IF` or `DO WHILE` statements.
- Incorrect mathematical calculations.
- Poorly designed program flow that omits important cases.

**Example:**
```harbour
// Logic error: the loop will run 10 times (from 1 to 10),
// but it might be expected to run 9 times.
FOR i := 1 TO 10
   IF i > 9  // This condition is only true in the last iteration
      ? "Nine"
   ENDIF
NEXT
```
The program doesn't fail, but its behavior is not correct if the intention was different. Debugging is the main tool to find these errors.

### 3. Runtime Errors

These errors occur while the program is running. They are exceptional situations that prevent the program from continuing its normal flow. Harbour has an error system to intercept and handle these failures.

**Common causes:**
- Trying to open a file that doesn't exist.
- Division by zero.
- Incorrect data type in an operation (e.g., adding a number to a string).
- Index outside the bounds of an array.
- Calling a function or method that doesn't exist.

If not handled, these errors usually terminate the application showing an error message on the screen.

## Structured Error Handling: `BEGIN SEQUENCE`

The modern and recommended way to handle runtime errors in Harbour is the `BEGIN SEQUENCE ... RECOVER ... END SEQUENCE` construct. It allows writing "protected" code and defining a specific code block to handle any error that occurs within it, similar to `try-catch` in other languages.

**Syntax:**
```harbour
BEGIN SEQUENCE
   // ... Error-prone code ...
   // If an error occurs here, control jumps to the RECOVER block
[BREAK] // Exits the sequence without error
RECOVER [USING oError]
   // ... Code to handle the error ...
   // The 'oError' object contains all error information
END SEQUENCE
```

- **`RECOVER [USING oError]`**: This block only executes if an error occurs in the upper block. The optional `USING oError` clause captures error details in an object (the "error object").
- **`BREAK`**: Can be used to exit the sequence prematurely, avoiding the `RECOVER` block.

**Practical Example:**
```harbour
FUNCTION Divide( nNum, nDen )
   LOCAL nResult

   BEGIN SEQUENCE
      IF nDen == 0
         // We can generate a custom error
         HB_DefaultError():New( "Division Error", ;
            "The denominator cannot be zero", ;
            NIL, NIL, NIL, NIL, NIL, NIL, NIL, 20 ):Throw()
      ENDIF
      nResult := nNum / nDen
   RECOVER USING oError
      ? "An error occurred while dividing:"
      ? "Description:", oError:Description
      ? "Operation:",   oError:Operation
      nResult := NIL // Assign a safe value
   END SEQUENCE

   RETURN nResult
```

### The Error Object

When an error is captured, Harbour creates an "error object" that contains valuable information for diagnosing the problem. Its main properties are:
- `oError:args` (Array): Arguments of the operation that failed.
- `oError:canDefault` (Logical): Can the operation be retried?
- `oError:canRetry` (Logical): Can the operation be retried?
- `oError:canSubstitute` (Logical): Can a result be substituted?
- `oError:description` (Character): Error description.
- `oError:filename` (Character): Name of the file where the error occurred.
- `oError:operation` (Character): Name of the function or operation that failed.
- `oError:osCode` (Numeric): Operating system error code.
- `oError:subCode` (Numeric): Harbour-specific error code.
- `oError:tries` (Numeric): Number of operation attempts.

## Traditional Error Handling: `ON ERROR`

This is the method inherited from Clipper. `ON ERROR` allows designating a global procedure that will be called automatically when a runtime error occurs anywhere in the application that is not covered by a `BEGIN SEQUENCE`.

**Syntax:**
`ON ERROR <procedure_name> | <code_block>`

**Example:**
```harbour
PROCEDURE Main()
   // Set the 'ErrorHandler' function as the global handler
   ON ERROR ErrorHandler( PCount(), ProcName( 1 ), ProcLine( 1 ) )

   // ... application code ...
   ? 1 / 0  // This will cause an error and call ErrorHandler

   RETURN

// Function that handles the error
PROCEDURE ErrorHandler( nParameters, cProcName, nLine )
   ALERT( "Error in: " + cProcName + " at line " + LTrim( Str( nLine ) ) )
   QUIT // Terminate the program
   RETURN
```
Although functional, `ON ERROR` is less flexible than `BEGIN SEQUENCE` because it is global and not structured, which can make it difficult to handle specific errors in local contexts.

## Debugging and Tracing Tools

Debugging is the process of finding and correcting logic errors. Harbour includes a very useful debugger.

### Using the Debugger

To use the debugger, you need to compile your program with the `-b` option.

`hbmk2 my_program.prg -b`

Once compiled, you can activate the debugger at any time during execution by pressing `Alt+D`. You can also start the program directly in debug mode.

**Main debugger features:**
- **Source Code Window:** Shows the code and highlights the current line.
- **Watch Window:** Allows monitoring the value of variables or expressions.
- **Call Stack Window:** Shows the sequence of functions that have been called to reach the current point.

**Common debugger commands:**
- `F8` (Step Over): Executes the current line and moves to the next. If the line is a function call, it executes it completely without entering it.
- `F7` (Step Into): Executes the current line. If it's a function call, it enters the function to debug it line by line.
- `F9` (Run): Executes the program until the next breakpoint or until it ends.
- `Ctrl+F8` (Run to Cursor): Executes the program until the line where the cursor is.
- `F4` (Go to Cursor): Similar to `Ctrl+F8`.
- `F5` (Zoom): Maximizes the active debugger window.

### Tracing Tools

Sometimes, instead of interactive debugging, it's useful to log the program flow or variable values to a text file (log).

**Useful functions:**
- `OutStd( ... )`, `OutErr( ... )`: Write to standard output or error output, which can be redirected to a file.
- `HB_TraceLog()`: A powerful function from the `hbdebug` library that can write formatted messages to a log file.

**Manual tracing example:**
```harbour
#include "fileio.ch"

PROCEDURE ComplexProcess( cData )
   LOCAL hFile

   // Open a log for this function
   hFile := FCreate( "process_log.txt" )
   FWrite( hFile, "Starting ComplexProcess at " + Time() + HB_EOL )
   FWrite( hFile, "Data received: " + cData + HB_EOL )

   // ... Complex logic ...
   IF Empty( cData )
      FWrite( hFile, "ERROR: Data is empty." + HB_EOL )
   ENDIF

   FWrite( hFile, "Finishing ComplexProcess." + HB_EOL )
   FClose( hFile )

   RETURN
```
This technique is invaluable for debugging applications on servers or in environments where it's not possible to run the interactive debugger.