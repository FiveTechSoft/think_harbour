# Chapter 20: The Runtime Library

Harbour's runtime library is a comprehensive collection of functions, procedures, and classes that provide fundamental functionality for application development. This library includes everything from basic data manipulation operations to advanced connectivity and processing functions.

---

## Built-in standard functions

### String manipulation functions

Harbour includes a robust set of functions for text handling:

```harbour
FUNCTION StringExamples()
   LOCAL cText := "  Harbour Programming Language  "
   LOCAL cResult
   
   // Cleaning and formatting
   ? "Original:", "[" + cText + "]"
   ? "Trim:", "[" + Trim(cText) + "]"
   ? "LTrim:", "[" + LTrim(cText) + "]"
   ? "RTrim:", "[" + RTrim(cText) + "]"
   ? "AllTrim:", "[" + AllTrim(cText) + "]"
   
   // Transformations
   ? "Upper:", Upper(cText)
   ? "Lower:", Lower(cText)
   ? "Proper:", Proper(cText)
   
   // Search and replace
   cText := "Harbour Programming Language"
   ? "At('Program'):", At("Program", cText)
   ? "RAt('a'):", RAt("a", cText)
   ? "StrTran:", StrTran(cText, "Programming", "Development")
   
   // Substrings
   ? "Left(10):", Left(cText, 10)
   ? "Right(8):", Right(cText, 8)
   ? "SubStr(9,11):", SubStr(cText, 9, 11)
   
   // Comparisons
   ? "Like pattern:", "Harbour" $ cText
   ? "Starts with H:", Left(cText, 1) == "H"
   
   RETURN NIL

// Advanced string functions
FUNCTION AdvancedStrings()
   LOCAL aTokens, cString, i
   
   // Tokenization
   cString := "one,two,three,four,five"
   aTokens := StrTokenize(cString, ",")
   
   ? "Tokens found:", Len(aTokens)
   FOR i := 1 TO Len(aTokens)
      ? i, ":", aTokens[i]
   NEXT
   
   // Numeric formatting in strings
   LOCAL nNumber := 1234567.89
   ? "StrZero:", StrZero(nNumber, 12, 2)
   ? "Transform:", Transform(nNumber, "999,999.99")
   ? "PadL:", PadL(LTrim(Str(nNumber)), 15, "*")
   ? "PadR:", PadR("Text", 20, ".")
   ? "PadC:", PadC("Center", 20, "-")
   
   // Content checks
   ? "IsAlpha:", IsAlpha("ABC")
   ? "IsDigit:", IsDigit("123")
   ? "IsAlNum:", IsAlNum("ABC123")
   ? "IsSpace:", IsSpace("   ")
   
   RETURN NIL
```

### Numeric and mathematical functions

```harbour
FUNCTION NumericExamples()
   LOCAL nNum := -123.456
   
   // Basic functions
   ? "Abs:", Abs(nNum)
   ? "Int:", Int(nNum)
   ? "Round:", Round(nNum, 2)
   ? "Ceiling:", Ceiling(nNum)
   ? "Floor:", Floor(nNum)
   
   // Trigonometric functions
   LOCAL nAngle := 45
   LOCAL nRadians := nAngle * (Pi() / 180)
   
   ? "Sin(45°):", Sin(nRadians)
   ? "Cos(45°):", Cos(nRadians)
   ? "Tan(45°):", Tan(nRadians)
   ? "ATan(1):", ATan(1) * (180 / Pi())  // Should be 45°
   
   // Exponential and logarithmic functions
   ? "Sqrt(16):", Sqrt(16)
   ? "Power(2,8):", Power(2, 8)
   ? "Exp(1):", Exp(1)  // e^1
   ? "Log(Exp(1)):", Log(Exp(1))  // Natural logarithm
   
   // Basic statistical functions
   LOCAL aNumbers := {10, 20, 30, 40, 50}
   ? "Min:", Min(aNumbers)
   ? "Max:", Max(aNumbers)
   ? "Sum:", Sum(aNumbers)
   ? "Average:", Average(aNumbers)
   
   RETURN NIL

// Advanced functions for financial calculations
FUNCTION FinancialCalculations()
   LOCAL nPrincipal := 100000    // Initial capital
   LOCAL nRate := 0.05           // 5% annual
   LOCAL nPeriods := 12          // 12 months
   
   // Compound interest
   LOCAL nFinalAmount := nPrincipal * Power(1 + nRate/12, nPeriods)
   ? "Final amount (compound interest):", Transform(nFinalAmount, "999,999.99")
   
   // Present value
   LOCAL nFutureValue := 110000
   LOCAL nPresentValue := nFutureValue / Power(1 + nRate, 1)
   ? "Present value:", Transform(nPresentValue, "999,999.99")
   
   // Payment calculation (approximate)
   LOCAL nPayment := (nPrincipal * nRate/12) / (1 - Power(1 + nRate/12, -nPeriods))
   ? "Monthly payment:", Transform(nPayment, "999,999.99")
   
   RETURN NIL
```

### Date and time functions

```harbour
FUNCTION DateExamples()
   LOCAL dToday := Date()
   LOCAL tNow := hb_DateTime()
   
   // Basic information
   ? "Current date:", DToC(dToday)
   ? "Day of week:", DOW(dToday), CDoW(dToday)
   ? "Day of month:", Day(dToday)
   ? "Month:", Month(dToday), CMonth(dToday)
   ? "Year:", Year(dToday)
   
   // Date manipulation
   ? "First day of month:", dToday - Day(dToday) + 1
   ? "Last day of month:", dToday + (45 - Day(dToday + 45))
   ? "A week ago:", dToday - 7
   ? "In 30 days:", dToday + 30
   
   // Age calculations
   LOCAL dBirth := CToD("06/15/1990")
   LOCAL nAge := Int((dToday - dBirth) / 365.25)
   ? "Calculated age:", nAge, "years"
   
   // Dates in different formats
   ? "ISO format:", Transform(dToday, "@D")
   ? "US format:", Transform(dToday, "99/99/9999")
   ? "European:", Transform(dToday, "99.99.9999")
   
   RETURN NIL

FUNCTION DateTimeExamples()
   LOCAL tTimestamp := hb_DateTime()
   LOCAL cTimeStamp
   
   // Date/time components
   ? "Full date:", hb_TToD(tTimestamp)
   ? "Time only:", hb_TToT(tTimestamp)
   ? "Year:", hb_Year(tTimestamp)
   ? "Month:", hb_Month(tTimestamp)
   ? "Day:", hb_Day(tTimestamp)
   ? "Hour:", hb_Hour(tTimestamp)
   ? "Minute:", hb_Minute(tTimestamp)
   ? "Second:", hb_Sec(tTimestamp)
   ? "Millisecond:", hb_MilliSec(tTimestamp)
   
   // Timestamp formatting
   cTimeStamp := hb_TToC(tTimestamp, "YYYY-MM-DD", "HH:MM:SS.FFF")
   ? "ISO format:", cTimeStamp
   
   // Operations with timestamps
   LOCAL tFuture := hb_DateTime() + 1.5  // 1.5 days in the future
   ? "Difference in days:", tFuture - tTimestamp
   ? "Difference in hours:", (tFuture - tTimestamp) * 24
   
   RETURN NIL
```

### Type conversion functions

```harbour
FUNCTION ConversionExamples()
   LOCAL xValue
   
   // Numeric conversions
   ? "Val('123.45'):", Val("123.45")
   ? "Val('  99  '):", Val("  99  ")
   ? "Val('ABC123'):", Val("ABC123")  // Returns 0
   
   // String conversions
   ? "Str(123):", "[" + Str(123) + "]"
   ? "Str(123, 8):", "[" + Str(123, 8) + "]"
   ? "Str(123.45, 8, 2):", "[" + Str(123.45, 8, 2) + "]"
   ? "LTrim(Str(123)):", "[" + LTrim(Str(123)) + "]"
   
   // Date conversions
   ? "DToS(Date()):", DToS(Date())
   ? "SToD('20240315'):", SToD("20240315")
   ? "CToD('03/15/2024'):", CToD("03/15/2024")
   ? "DToC(Date()):", DToC(Date())
   
   // Logical conversions
   ? "ValType(.T.):", ValType(.T.)
   ? "ValType(.F.):", ValType(.F.)
   
   // Universal ValToChar conversion
   xValue := {1, "two", .T., Date(), NIL}
   LOCAL i
   FOR i := 1 TO Len(xValue)
      ? i, ":", ValToChar(xValue[i])
   NEXT
   
   RETURN NIL
```

---

## Array and hash functions

### Array manipulation

```harbour
FUNCTION ArrayExamples()
   LOCAL aOriginal := {10, 20, 30, 40, 50}
   LOCAL aNew, i
   
   // Basic information
   ? "Length:", Len(aOriginal)
   ? "First element:", aOriginal[1]
   ? "Last element:", ATail(aOriginal)
   
   // Size modification
   aNew := Array(10)
   AFill(aNew, 0)
   ? "Array filled with zeros:", ValToChar(aNew)
   
   // Copy arrays
   aNew := ACopy(aOriginal, aNew, 1, Len(aOriginal), 1)
   ? "Copied array:", ValToChar(aNew)
   
   // Insertion and deletion
   AIns(aNew, 3)          // Insert at position 3
   aNew[3] := 25          // Assign value
   ? "After inserting:", ValToChar(aNew)
   
   ADel(aNew, 7)          // Delete position 7
   ASize(aNew, Len(aNew) - 1)  // Reduce size
   ? "After deleting:", ValToChar(aNew)
   
   // Search
   ? "AScan(30):", AScan(aNew, 30)
   ? "AScan(99):", AScan(aNew, 99)  // Not found
   
   // Sorting
   LOCAL aTexts := {"Zebra", "Alpha", "Beta", "Gamma"}
   ASort(aTexts)
   ? "Sorted:", ValToChar(aTexts)
   
   RETURN NIL

FUNCTION AdvancedArrays()
   LOCAL aMatrix := Array(3, 4)  // 3x4 matrix
   LOCAL i, j
   
   // Fill matrix
   FOR i := 1 TO 3
      FOR j := 1 TO 4
         aMatrix[i][j] := i * 10 + j
      NEXT
   NEXT
   
   // Display matrix
   ? "3x4 Matrix:"
   FOR i := 1 TO 3
      FOR j := 1 TO 4
         ?? Str(aMatrix[i][j], 4)
      NEXT
      ?
   NEXT
   
   // Array evaluation
   LOCAL aNumbers := {1, 2, 3, 4, 5}
   LOCAL aSquares := Array(Len(aNumbers))
   
   AEval(aNumbers, {|n, i| aSquares[i] := n * n})
   ? "Numbers:", ValToChar(aNumbers)
   ? "Squares:", ValToChar(aSquares)
   
   // Filtering with AEval
   LOCAL aEvens := {}
   AEval(aNumbers, {|n| iif(n % 2 == 0, AAdd(aEvens, n), NIL)})
   ? "Even numbers:", ValToChar(aEvens)
   
   RETURN NIL
```

### Hash functions

```harbour
FUNCTION HashExamples()
   LOCAL hPerson := {=>}  // Empty hash
   LOCAL aKeys, aValues, i
   
   // Add elements
   hPerson["name"] := "John Doe"
   hPerson["age"] := 30
   hPerson["city"] := "Madrid"
   hPerson["active"] := .T.
   hPerson["salary"] := 45000.50
   
   ? "Hash created with", Len(hPerson), "elements"
   
   // Element access
   ? "Name:", hPerson["name"]
   ? "Age:", hPerson["age"]
   
   // Check key existence
   ? "Does 'phone' exist?:", "phone" $ hPerson
   ? "Does 'age' exist?:", "age" $ hPerson
   
   // Get keys and values
   aKeys := hb_HKeys(hPerson)
   aValues := hb_HValues(hPerson)
   
   ? "Keys:", ValToChar(aKeys)
   ? "Values:", ValToChar(aValues)
   
   // Iterate over hash
   ? "Complete hash content:"
   FOR i := 1 TO Len(aKeys)
      ? "  ", aKeys[i], "=>", ValToChar(aValues[i])
   NEXT
   
   // Delete element
   hb_HDel(hPerson, "salary")
   ? "After deleting 'salary':", Len(hPerson), "elements"
   
   RETURN NIL

FUNCTION AdvancedHashes()
   // Hash with different key types
   LOCAL hMixed := {=>}
   
   hMixed[1] := "Numeric key"
   hMixed["text"] := "Text key"
   hMixed[Date()] := "Date key"
   hMixed[.T.] := "Logical key"
   
   // Nested hash
   LOCAL hCompany := {=>}
   hCompany["name"] := "Tech Corp"
   hCompany["employees"] := {=>}
   hCompany["employees"]["john"] := {"age" => 30, "dept" => "IT"}
   hCompany["employees"]["mary"] := {"age" => 28, "dept" => "HR"}
   
   ? "Employee John, age:", hCompany["employees"]["john"]["age"]
   
   // Combine hashes
   LOCAL hBaseConfig := {"debug" => .F., "timeout" => 30}
   LOCAL hLocalConfig := {"debug" => .T., "host" => "localhost"}
   LOCAL hFinalConfig := hb_HMerge(hBaseConfig, hLocalConfig)
   
   ? "Final configuration:", ValToChar(hb_HKeys(hFinalConfig))
   
   RETURN NIL
```

---

## File I/O functions

### Low-level file manipulation

```harbour
FUNCTION FileExamples()
   LOCAL hFile, cBuffer, nBytes
   LOCAL cFileName := "test_file.txt"
   LOCAL cContent := "Line 1" + hb_eol() + ;
                     "Line 2" + hb_eol() + ;
                     "Line 3" + hb_eol()
   
   // Create and write file
   hFile := FCreate(cFileName)
   IF hFile >= 0
      nBytes := FWrite(hFile, cContent)
      ? "Bytes written:", nBytes
      FClose(hFile)
   ELSE
      ? "Error creating file:", FError()
      RETURN NIL
   ENDIF
   
   // Read file
   hFile := FOpen(cFileName, FO_READ)
   IF hFile >= 0
      cBuffer := Space(1024)
      nBytes := FRead(hFile, @cBuffer, Len(cBuffer))
      cBuffer := Left(cBuffer, nBytes)
      ? "Content read:"
      ? cBuffer
      FClose(hFile)
   ELSE
      ? "Error opening file for reading"
   ENDIF
   
   // File information
   ? "File size:", FSeek(hFile, 0, FS_END)
   ? "Does file exist?:", File(cFileName)
   ? "File date:", FileDate(cFileName)
   ? "File time:", FileTime(cFileName)
   
   // Clean up
   FErase(cFileName)
   
   RETURN NIL

FUNCTION HighLevelFiles()
   LOCAL cFile := "memo_test.txt"
   LOCAL cContent := "This is a test file" + hb_eol() + ;
                     "with multiple lines" + hb_eol() + ;
                     "to demonstrate high-level functions."
   
   // Write entire file
   MemoWrit(cFile, cContent)
   ? "File written:", cFile
   
   // Read entire file
   LOCAL cRead := MemoRead(cFile)
   ? "Content read:"
   ? cRead
   
   // Count lines
   LOCAL aLines := StrTokenize(cRead, hb_eol())
   ? "Number of lines:", Len(aLines)
   
   // Process line by line
   LOCAL i
   FOR i := 1 TO Len(aLines)
      ? "Line", i, ":", aLines[i]
   NEXT
   
   // Additional information
   ? "Size in bytes:", Len(cRead)
   ? "Approximate words:", Len(StrTokenize(cRead, " " + hb_eol()))
   
   // Clean up
   FErase(cFile)
   
   RETURN NIL
```

### Directory manipulation

```harbour
FUNCTION DirectoryExamples()
   LOCAL aFiles, i
   LOCAL cWorkDir := CurDir()
   
   ? "Current directory:", cWorkDir
   
   // Change directory
   IF !DirChange("temp")
      ? "Creating 'temp' directory..."
      MakeDir("temp")
      DirChange("temp")
   ENDIF
   
   ? "New directory:", CurDir()
   
   // Create some test files
   MemoWrit("file1.txt", "Content 1")
   MemoWrit("file2.log", "Content 2")
   MemoWrit("data.csv", "col1,col2,col3")
   
   // List files
   aFiles := Directory("*.*")
   ? "Files in directory:"
   FOR i := 1 TO Len(aFiles)
      ? "  ", aFiles[i][F_NAME], ;
            Str(aFiles[i][F_SIZE], 8), ;
            DToC(aFiles[i][F_DATE]), ;
            aFiles[i][F_TIME]
   NEXT
   
   // Search specific files
   aFiles := Directory("*.txt")
   ? ".txt files found:", Len(aFiles)
   
   // Return to original directory
   DirChange("..")
   
   // Clean up test files
   FErase("temp/file1.txt")
   FErase("temp/file2.log")
   FErase("temp/data.csv")
   RemoveDir("temp")
   
   RETURN NIL
```

---

## System functions

### Environment information

```harbour
FUNCTION SystemInformation()
   ? "=== SYSTEM INFORMATION ==="
   ? "Operating system:", OS()
   ? "Harbour version:", Version()
   ? "Current directory:", CurDir()
   ? "System date:", Date()
   ? "System time:", Time()
   
   // Environment variables
   ? "PATH:", GetEnv("PATH")
   ? "USER:", GetEnv("USER")
   ? "HOME:", GetEnv("HOME")
   ? "TEMP:", GetEnv("TEMP")
   
   // Memory information
   ? "Memory used:", hb_MemoryUsed(), "bytes"
   ? "Maximum memory:", hb_MemoryMax(), "bytes"
   
   // Processor information
   ? "Number of CPUs:", hb_NumCPU()
   
   RETURN NIL

FUNCTION ExecuteCommands()
   LOCAL nExitCode, cCommand
   
   // Execute system command
   #ifdef __PLATFORM__WINDOWS
      cCommand := "dir"
   #else
      cCommand := "ls -la"
   #endif
   
   ? "Executing command:", cCommand
   nExitCode := hb_Run(cCommand)
   ? "Exit code:", nExitCode
   
   // Execute command and capture output
   LOCAL cOutput := ""
   nExitCode := hb_ProcessRun(cCommand, NIL, @cOutput)
   ? "Captured output:"
   ? cOutput
   
   RETURN NIL
```

### Time and timing functions

```harbour
FUNCTION TimeExamples()
   LOCAL nStart, nEnd, nElapsed
   
   ? "=== TIME MEASUREMENT ==="
   
   // Measure execution time
   nStart := Seconds()
   
   // Simulate work
   LOCAL i, nResult := 0
   FOR i := 1 TO 1000000
      nResult += i
   NEXT
   
   nEnd := Seconds()
   nElapsed := nEnd - nStart
   
   ? "Calculation result:", nResult
   ? "Elapsed time:", nElapsed, "seconds"
   
   // High precision time
   LOCAL nStartHP := hb_MilliSeconds()
   
   // Another operation
   LOCAL aLarge := Array(100000)
   AFill(aLarge, Random(1000))
   ASort(aLarge)
   
   LOCAL nEndHP := hb_MilliSeconds()
   ? "Sorting time:", nEndHP - nStartHP, "milliseconds"
   
   // Programmed pauses
   ? "Waiting 2 seconds..."
   hb_IdleSleep(2000)  // 2000 milliseconds
   ? "Continuing..."
   
   RETURN NIL
```

Harbour's runtime library is extremely rich and powerful, providing all the necessary tools for professional application development. These functions are optimized for performance and are compatible across different platforms, making code portable and reliable.