# Chapter 18: Debugging and Idle States

Effective debugging and idle state management are essential skills for professional development in Harbour. This chapter explores advanced tools and techniques for identifying, diagnosing, and resolving issues in Harbour applications, as well as managing background tasks.

---

## The debugger: tools for tracking and resolving errors

Harbour includes a sophisticated integrated debugger that allows real-time inspection of application state.

### Compilation with debugging information

To use the debugger, code must be compiled with debugging information:

```bash
# Compile with debug information
hbmk2 -b my_program.prg

# With extended debug information
hbmk2 -b -debug my_program.prg

# Complete debug with symbols
hbmk2 -b -debug -debugger my_program.prg
```

### Debugger activation

```harbour
FUNCTION Main()
   // Activate debugger programmatically
   AltD()  // Activates debugger at next step
   
   LOCAL nResult := CalculateResult(10, 20)
   ? nResult
   
   RETURN NIL

FUNCTION CalculateResult(nA, nB)
   LOCAL nTemp
   
   // Conditional breakpoint
   IF nA > nB
      AltD()  // Only activates if nA > nB
   ENDIF
   
   nTemp := nA * nB
   RETURN nTemp + (nA + nB)
```

### Debugger commands

When the debugger is active, available commands include:

```
F5 or Go      - Continue execution
F8 or Step    - Execute next line (step over)
F7 or Into    - Enter function (step into)
F6 or Out     - Exit current function (step out)
F9 or Toggle  - Toggle breakpoint
F10 or Quit   - Terminate debugging

// Inspection commands
? <variable>     - Show variable value
?? <expression>  - Evaluate expression
List             - Show current source code
Stack            - Show call stack
Locals           - Show local variables
Statics          - Show static variables
```

### Advanced programmatic debugger

```harbour
// Function for advanced conditional debugging
FUNCTION DebugIf(lCondition, cMessage)
   IF lCondition
      ? "DEBUG:", cMessage
      ? "Call stack:"
      ShowCallStack()
      AltD()
   ENDIF
   RETURN NIL

STATIC FUNCTION ShowCallStack()
   LOCAL i := 1
   LOCAL cProc
   
   WHILE !Empty(cProc := ProcName(i))
      ? Space(i * 2) + Str(i, 2) + ": " + cProc + "(" + ProcFile(i) + ":" + ;
        LTrim(Str(ProcLine(i))) + ")"
      i++
   ENDDO
   RETURN NIL

// Using the advanced debugger
FUNCTION ProcessData(aData)
   LOCAL i, result
   
   DebugIf(Empty(aData), "Data array is empty")
   DebugIf(Len(aData) > 1000, "Array too large: " + LTrim(Str(Len(aData))))
   
   FOR i := 1 TO Len(aData)
      result := ProcessItem(aData[i])
      DebugIf(result == NIL, "Error processing item " + LTrim(Str(i)))
   NEXT
   
   RETURN result
```

### Remote debugger

```harbour
// Configure remote debugging
FUNCTION ConfigureRemoteDebug()
   // Activate debug server
   hb_DebugServerStart(8080)
   
   // Connect remote client
   hb_DebugClientConnect("192.168.1.100", 8080)
   
   RETURN NIL

// Remote breakpoints
FUNCTION SetRemoteBreakpoint(cFile, nLine)
   hb_DebugSetBreakpoint(cFile, nLine, .T.)
   RETURN NIL
```

---

## Debugging statements: `PROCNAME()`, `VALTYPE()`, etc.

Harbour provides specialized intrinsic functions for debugging and diagnostics.

### Program introspection functions

```harbour
FUNCTION ExecutionInformation()
   ? "Current function:", ProcName()
   ? "Current file:", ProcFile()
   ? "Current line:", ProcLine()
   ? "Nesting level:", ProcLevel()
   
   // Show entire call stack
   LOCAL i := 0
   ? "=== CALL STACK ==="
   WHILE i <= ProcLevel()
      ? Str(i, 3) + ":", ProcName(i), "in", ProcFile(i), "line", ProcLine(i)
      i++
   ENDDO
   
   RETURN NIL

FUNCTION NestedFunction()
   ExecutionInformation()
   RETURN NIL

FUNCTION Main()
   NestedFunction()
   RETURN NIL
```

### Type analysis functions

```harbour
FUNCTION AnalyzeVariable(xVariable, cName)
   LOCAL cType := ValType(xVariable)
   LOCAL cInfo := "Variable: " + cName + " "
   
   DO CASE
   CASE cType == "C"
      cInfo += "String(" + LTrim(Str(Len(xVariable))) + "): '" + xVariable + "'"
      
   CASE cType == "N"
      cInfo += "Numeric: " + LTrim(Str(xVariable))
      IF xVariable == Int(xVariable)
         cInfo += " (Integer)"
      ELSE
         cInfo += " (Decimal)"
      ENDIF
      
   CASE cType == "L"
      cInfo += "Logical: " + iif(xVariable, ".T.", ".F.")
      
   CASE cType == "D"
      cInfo += "Date: " + DToC(xVariable)
      
   CASE cType == "A"
      cInfo += "Array[" + LTrim(Str(Len(xVariable))) + "]"
      AnalyzeArray(xVariable, cName)
      
   CASE cType == "H"
      cInfo += "Hash{" + LTrim(Str(Len(xVariable))) + "}"
      AnalyzeHash(xVariable, cName)
      
   CASE cType == "B"
      cInfo += "CodeBlock"
      
   CASE cType == "O"
      cInfo += "Object: " + xVariable:ClassName()
      AnalyzeObject(xVariable, cName)
      
   OTHERWISE
      cInfo += "Unknown type: " + cType
   ENDCASE
   
   ? cInfo
   RETURN NIL

STATIC FUNCTION AnalyzeArray(aArray, cName)
   LOCAL i
   ? "  Contents of array " + cName + ":"
   FOR i := 1 TO Min(Len(aArray), 10)  // Show only first 10 elements
      ?? "  [" + LTrim(Str(i)) + "] "
      AnalyzeVariable(aArray[i], cName + "[" + LTrim(Str(i)) + "]")
   NEXT
   IF Len(aArray) > 10
      ? "  ... (" + LTrim(Str(Len(aArray) - 10)) + " more elements)"
   ENDIF
   RETURN NIL

STATIC FUNCTION AnalyzeHash(hHash, cName)
   LOCAL aKeys := hb_HKeys(hHash)
   LOCAL i
   ? "  Contents of hash " + cName + ":"
   FOR i := 1 TO Min(Len(aKeys), 10)
      ? "  [" + ValToChar(aKeys[i]) + "] =>",
      AnalyzeVariable(hHash[aKeys[i]], cName + "[" + ValToChar(aKeys[i]) + "]")
   NEXT
   RETURN NIL
```

### Advanced logging system

```harbour
// Logging system with levels
#define LOG_ERROR   1
#define LOG_WARNING 2
#define LOG_INFO    3
#define LOG_DEBUG   4

STATIC s_nLogLevel := LOG_INFO
STATIC s_cLogFile := "app.log"

FUNCTION SetLogLevel(nLevel)
   s_nLogLevel := nLevel
   RETURN NIL

FUNCTION WriteLog(nLevel, cMessage, xData)
   LOCAL cLevel, cLogEntry, hFile
   LOCAL cTimeStamp := DToC(Date()) + " " + Time()
   
   IF nLevel > s_nLogLevel
      RETURN NIL  // Don't log if below level
   ENDIF
   
   DO CASE
   CASE nLevel == LOG_ERROR
      cLevel := "ERROR"
   CASE nLevel == LOG_WARNING
      cLevel := "WARN "
   CASE nLevel == LOG_INFO
      cLevel := "INFO "
   CASE nLevel == LOG_DEBUG
      cLevel := "DEBUG"
   OTHERWISE
      cLevel := "UNKN "
   ENDCASE
   
   cLogEntry := cTimeStamp + " [" + cLevel + "] " + ;
                ProcName(1) + "(" + LTrim(Str(ProcLine(1))) + "): " + cMessage
   
   IF xData != NIL
      cLogEntry += " | Data: " + ValToChar(xData)
   ENDIF
   
   // Write to file
   hFile := FOpen(s_cLogFile, FO_CREAT + FO_WRITE)
   IF hFile >= 0
      FSeek(hFile, 0, FS_END)
      FWrite(hFile, cLogEntry + hb_eol())
      FClose(hFile)
   ENDIF
   
   // Show on console if critical error
   IF nLevel == LOG_ERROR
      ? cLogEntry
   ENDIF
   
   RETURN NIL

// Macros to simplify usage
#command LOG_ERR <msg> [DATA <data>] => WriteLog(LOG_ERROR, <msg> [, <data>])
#command LOG_WARN <msg> [DATA <data>] => WriteLog(LOG_WARNING, <msg> [, <data>])
#command LOG_INFO <msg> [DATA <data>] => WriteLog(LOG_INFO, <msg> [, <data>])
#command LOG_DEBUG <msg> [DATA <data>] => WriteLog(LOG_DEBUG, <msg> [, <data>])

// Usage
FUNCTION ProcessOrder(nId)
   LOCAL oOrder
   
   LOG_INFO "Starting order processing" DATA nId
   
   oOrder := LoadOrder(nId)
   IF oOrder == NIL
      LOG_ERR "Could not load order" DATA nId
      RETURN .F.
   ENDIF
   
   LOG_DEBUG "Order loaded successfully" DATA oOrder
   
   IF !ValidateOrder(oOrder)
      LOG_WARN "Invalid order" DATA oOrder:ToHash()
      RETURN .F.
   ENDIF
   
   LOG_INFO "Order processed successfully" DATA nId
   RETURN .T.
```

---

## `Idle states`: managing background tasks

Idle states allow executing tasks when the application is not processing main events.

### Basic idle state configuration

```harbour
FUNCTION ConfigureIdleStates()
   // Configure function that runs on idle
   hb_IdleAdd({|| ProcessBackgroundTasks()})
   
   // Configure multiple idle functions
   hb_IdleAdd({|| CleanTempCache()})
   hb_IdleAdd({|| UpdateStatistics()})
   hb_IdleAdd({|| CheckConnections()})
   
   RETURN NIL

STATIC FUNCTION ProcessBackgroundTasks()
   STATIC s_nCounter := 0
   
   s_nCounter++
   
   // Execute every 100 idle cycles (approx.)
   IF s_nCounter % 100 == 0
      ? "Processing background tasks:", Time()
      GarbageCollect()
   ENDIF
   
   RETURN NIL
```

### Scheduled task system

```harbour
// Structure for scheduled tasks
STATIC s_aTasks := {}

FUNCTION ScheduleTask(cName, bTask, nInterval, tLastExecution)
   LOCAL hTask := {=>}
   
   hTask["name"] := cName
   hTask["task"] := bTask
   hTask["interval"] := nInterval  // In seconds
   hTask["last"] := iif(tLastExecution == NIL, hb_DateTime(), tLastExecution)
   hTask["active"] := .T.
   
   AAdd(s_aTasks, hTask)
   
   RETURN Len(s_aTasks)

FUNCTION ExecuteScheduledTasks()
   LOCAL i, hTask, tNow := hb_DateTime()
   
   FOR i := 1 TO Len(s_aTasks)
      hTask := s_aTasks[i]
      
      IF hTask["active"] .AND. ;
         (tNow - hTask["last"]) >= hTask["interval"]
         
         LOG_DEBUG "Executing scheduled task: " + hTask["name"]
         
         TRY
            Eval(hTask["task"])
            hTask["last"] := tNow
         CATCH
            LOG_ERR "Error in scheduled task: " + hTask["name"]
         END
      ENDIF
   NEXT
   
   RETURN NIL

// Configure task system
FUNCTION ConfigureTaskSystem()
   // Clean temp files every 5 minutes
   ScheduleTask("CleanTemp", {|| CleanTempFiles()}, 300)
   
   // Backup data every 30 minutes
   ScheduleTask("Backup", {|| CreateAutoBackup()}, 1800)
   
   // Check DB connection every minute
   ScheduleTask("CheckDB", {|| VerifyDBConnection()}, 60)
   
   // Configure idle to execute tasks
   hb_IdleAdd({|| ExecuteScheduledTasks()})
   
   RETURN NIL
```

### Real-time performance monitor

```harbour
STATIC s_hStatistics := {=>}

FUNCTION InitializeMonitor()
   s_hStatistics["start"] := hb_DateTime()
   s_hStatistics["idle_cycles"] := 0
   s_hStatistics["max_memory"] := 0
   s_hStatistics["db_queries"] := 0
   s_hStatistics["errors"] := 0
   
   // Monitor on idle
   hb_IdleAdd({|| UpdatePerformanceStatistics()})
   
   RETURN NIL

STATIC FUNCTION UpdatePerformanceStatistics()
   LOCAL nCurrentMemory
   
   s_hStatistics["idle_cycles"]++
   
   // Monitor memory usage
   nCurrentMemory := hb_MemoryUsed()
   IF nCurrentMemory > s_hStatistics["max_memory"]
      s_hStatistics["max_memory"] := nCurrentMemory
   ENDIF
   
   // Show statistics every 1000 cycles
   IF s_hStatistics["idle_cycles"] % 1000 == 0
      ShowStatistics()
   ENDIF
   
   RETURN NIL

FUNCTION ShowStatistics()
   LOCAL nTotalTime := hb_DateTime() - s_hStatistics["start"]
   
   ? "=== PERFORMANCE STATISTICS ==="
   ? "Total time:", Int(nTotalTime), "seconds"
   ? "Idle cycles:", s_hStatistics["idle_cycles"]
   ? "Max memory:", s_hStatistics["max_memory"], "bytes"
   ? "DB queries:", s_hStatistics["db_queries"]
   ? "Errors:", s_hStatistics["errors"]
   ? "Avg cycles/sec:", Int(s_hStatistics["idle_cycles"] / Max(nTotalTime, 1))
   
   RETURN NIL
```

### Advanced idle state management

```harbour
// Granular control of idle states
FUNCTION ControlIdleStates()
   LOCAL nIdleHandle1, nIdleHandle2
   
   // Add idle functions with identifiers
   nIdleHandle1 := hb_IdleAdd({|| CriticalTask()})
   nIdleHandle2 := hb_IdleAdd({|| OptionalTask()})
   
   // Temporarily pause specific task
   hb_IdleSleep(nIdleHandle2)
   
   // Reactivate task
   hb_IdleWake(nIdleHandle2)
   
   // Remove specific task
   hb_IdleDel(nIdleHandle1)
   
   RETURN NIL

// System load-aware idle state
STATIC FUNCTION AdaptiveIdle()
   STATIC s_nLoad := 0
   LOCAL nMemoryUsed := hb_MemoryUsed()
   LOCAL nCpuLoad := GetCpuLoad()  // Hypothetical function
   
   // Adjust frequency based on load
   IF nCpuLoad > 80
      s_nLoad++
      IF s_nLoad > 10
         hb_IdleSleep()  // Pause temporarily
         hb_RelIdle(1000)  // Wait 1 second
         s_nLoad := 0
      ENDIF
   ELSE
      s_nLoad := Max(0, s_nLoad - 1)
   ENDIF
   
   RETURN NIL
```

### Debugging idle states

```harbour
FUNCTION DebugIdleStates()
   // Global idle counter
   STATIC s_nIdleCount := 0
   
   s_nIdleCount++
   
   // Periodic log of idle activity
   IF s_nIdleCount % 500 == 0
      LOG_DEBUG "Idle state executed " + LTrim(Str(s_nIdleCount)) + " times"
      
      // Check for deadlocks or infinite loops
      IF s_nIdleCount > 100000
         LOG_ERR "Possible infinite loop in idle states"
         ? "ALERT: Too many idle cycles, press any key to continue"
         Inkey(0)
      ENDIF
   ENDIF
   
   RETURN NIL
```

Idle states are a powerful tool for creating responsive applications that can perform maintenance, monitoring, and update tasks in the background without blocking user interaction.