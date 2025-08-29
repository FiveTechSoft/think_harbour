# Chapter 4: Control structures

In Harbour, as in most programming languages, control structures are fundamental for directing the execution flow of a program. They allow making decisions, repeating actions, and handling code sequences in an orderly manner.

## Conditional statements

Conditional statements execute code blocks only if certain conditions are met.

### IF, ELSEIF, ELSE, ENDIF

This is the most common conditional structure. It allows executing different code blocks based on one or more logical conditions.

**Syntax:**
```harbour
IF <logicalCondition1>
   // Code block if logicalCondition1 is true (.T.)
[ELSEIF <logicalCondition2>]
   // Code block if logicalCondition2 is true (.T.)
[ELSE]
   // Code block if none of the previous conditions is true
ENDIF
```

**Example:**
```harbour
PROCEDURE CheckAge( nAge )
   IF nAge >= 18
      ? "Is of legal age."
   ELSE
      ? "Is a minor."
   ENDIF
RETURN
```

### SWITCH

The `SWITCH` structure is useful when you need to compare a single variable or expression with a series of possible values. It is a more readable alternative to a series of `IF...ELSEIF`.

**Syntax:**
```harbour
SWITCH <expression>
   CASE <value1>
      // Code block if the expression equals value1
      [EXIT] // Optional, exits the SWITCH
   CASE <value2>
      // Code block if the expression equals value2
      [EXIT]
   [OTHERWISE]
      // Code block if the expression doesn't match any CASE
ENDSWITCH
```
*Note: Unlike other languages, if `EXIT` is not used, execution continues to the next `CASE`.*

**Example:**
```harbour
FUNCTION DayOfTheWeek( nDay )
   LOCAL cDay := ""
   SWITCH nDay
      CASE 1
         cDay := "Monday"
      CASE 2
         cDay := "Tuesday"
      CASE 3
         cDay := "Wednesday"
      // ... other days
      OTHERWISE
         cDay := "Invalid day"
   ENDSWITCH
RETURN cDay
```

## Loops

Loops allow repeating the execution of a code block multiple times.

### DO WHILE

This loop repeats a code block while a condition is true. The condition is evaluated at the beginning of the loop.

**Syntax:**
```harbour
DO WHILE <logicalCondition>
   // Code block to repeat
   [LOOP]   // Returns to the beginning of DO WHILE
   [EXIT]   // Exits the loop
ENDDO
```

**Example:**
```harbour
PROCEDURE CountToFive()
   LOCAL nCounter := 1
   DO WHILE nCounter <= 5
      ? nCounter
      nCounter++
   ENDDO
RETURN
```

### FOR...NEXT

This loop is used to repeat a code block a specific number of times. It is ideal for iterating with a counter.

**Syntax:**
```harbour
FOR <variable> := <start> TO <end> [STEP <increment>]
   // Code block to repeat
   [LOOP]   // Skips to the next iteration
   [EXIT]   // Exits the loop
NEXT
```

**Example:**
```harbour
PROCEDURE TwoTimesTable()
   LOCAL nNumber
   FOR nNumber := 1 TO 10
      ? "2 x", nNumber, "=", 2 * nNumber
   NEXT
RETURN
```

### FOR EACH

This loop is designed to iterate over the elements of a collection, such as an array or a hash.

**Syntax:**
```harbour
FOR EACH <variable> IN <collection>
   // Code block to execute for each element
   // The <variable> takes the value of each element in each iteration
   [LOOP]   // Skips to the next iteration
   [EXIT]   // Exits the loop
NEXT
```

**Example:**
```harbour
PROCEDURE ShowColors()
   LOCAL aColors := { "Red", "Green", "Blue" }
   LOCAL cColor

   FOR EACH cColor IN aColors
      ? cColor
   NEXT
RETURN
```

## Sequence handler

### BEGIN SEQUENCE...END

This structure provides a mechanism for error handling or for exiting a nested code sequence in a controlled way, similar to `try...catch` blocks in other languages.

- `BREAK`: Interrupts the sequence and transfers control to the `RECOVER` clause.
- `RECOVER`: Executes if a `BREAK` occurs in the sequence.

**Syntax:**
```harbour
BEGIN SEQUENCE
   // Main sequence code
   [BREAK] // Interrupts the sequence
[RECOVER [USING <errorVariable>]]
   // Recovery code that executes after a BREAK
END SEQUENCE
```

**Error handling example:**
```harbour
FUNCTION Divide( nNum, nDen )
   LOCAL nResult

   BEGIN SEQUENCE
      IF nDen == 0
         // Cannot divide by zero, break the sequence
         BREAK
      ENDIF
      nResult := nNum / nDen
   RECOVER
      // If there was a BREAK, assign a default value
      nResult := 0
      ? "Error: Division by zero."
   END SEQUENCE

RETURN nResult
```