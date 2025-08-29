# Chapter 17: The Preprocessor and Macros

Harbour's preprocessor is an extremely powerful tool that allows modifying source code before compilation. It goes far beyond basic substitution capabilities, offering a complete code transformation system that enables creating DSLs (Domain Specific Languages) and simplifying complex syntax.

---

## Compilation directives: `#define`, `#include`, `#command`, `#xcommand`

### `#define` directive

The `#define` directive creates textual substitution macros that are processed before compilation.

#### Simple definitions

```harbour
// Basic constants
#define VERSION "1.0.0"
#define MAX_USERS 100
#define DEBUG .T.

// Macros with parameters
#define SQUARE(x) ((x) * (x))
#define MAX(a,b) (iif((a) > (b), (a), (b)))
#define LOG(msg) (iif(DEBUG, QOut("LOG: " + msg), NIL))

// Using the macros
LOCAL nResult := SQUARE(5)  // Expands to ((5) * (5))
LOCAL nMaximum := MAX(10, 20)  // Expands to (iif((10) > (20), (10), (20)))
LOG("Starting application")   // Conditional based on DEBUG
```

#### Conditional macros

```harbour
#ifdef __HARBOUR__
   #define COMPILER_NAME "Harbour"
#else
   #define COMPILER_NAME "Clipper"
#endif

#ifndef MAX_BUFFER_SIZE
   #define MAX_BUFFER_SIZE 4096
#endif

// Platform conditional compilation
#ifdef __PLATFORM__WINDOWS
   #define PATH_SEPARATOR "\"
   #define NULL_DEVICE "NUL"
#else
   #define PATH_SEPARATOR "/"
   #define NULL_DEVICE "/dev/null"
#endif
```

#### Nested and recursive macros

```harbour
#define INDENT "  "
#define DOUBLE_INDENT INDENT + INDENT
#define TRIPLE_INDENT DOUBLE_INDENT + INDENT

#define TRACE_LEVEL_1(msg) QOut(INDENT + msg)
#define TRACE_LEVEL_2(msg) QOut(DOUBLE_INDENT + msg)
#define TRACE_LEVEL_3(msg) QOut(TRIPLE_INDENT + msg)
```

### `#include` directive

Allows including content from other files in the current code.

```harbour
// File: constants.ch
#define APP_NAME "My Application"
#define APP_VERSION "2.1.0"
#define COPYRIGHT "© 2024 My Company"

// Main file
#include "constants.ch"
#include "hbclass.ch"   // OOP definitions
#include "fileio.ch"    // File handling constants

FUNCTION Main()
   ? APP_NAME, APP_VERSION
   ? COPYRIGHT
   RETURN NIL
```

#### Conditional includes

```harbour
#ifdef USE_GUI
   #include "hwgui.ch"
#else
   #include "console.ch"
#endif

// Include with existence check
#ifdef __FILE_EXISTS("local_config.ch")
   #include "local_config.ch"
#else
   #include "default_config.ch"
#endif
```

### `#command` directive

Allows creating new commands with custom syntax.

#### Basic commands

```harbour
// Define a simple command
#command SHOW <value> => QOut(<value>)

// Usage
SHOW "Hello world"  // Translates to: QOut("Hello world")
SHOW nVariable     // Translates to: QOut(nVariable)
```

#### Commands with multiple parameters

```harbour
// Command for validation
#command VALIDATE <var> TYPE <type> => ;
   if (ValType(<var>) != <type>) ; 
      Alert("Error: " + #<var> + " must be " + <type>) ; 
   endif

// Usage
LOCAL cName := "John"
LOCAL nAge := "30"  // Intentional error
VALIDATE cName TYPE "C"  // OK
VALIDATE nAge TYPE "N"    // Will show error
```

#### Commands with optional keywords

```harbour
// Command with SQL-like syntax
#command SELECT <fields> FROM <table> [WHERE <condition>] [ORDER BY <order>] => ;
   ProcessQuery(<table>, <fields> [, <condition>] [, <order>])

// Usage
SELECT "name, age" FROM "employees"
SELECT "name, age" FROM "employees" WHERE "age > 30"
SELECT "name, age" FROM "employees" WHERE "active = .T." ORDER BY "name"
```

### `#xcommand` directive

Extended version of `#command` with more advanced pattern matching capabilities.

#### Patterns with repetition

```harbour
// Command that accepts multiple elements
#xcommand CREATE MENU <menu> ;
   [MENUITEM <item1> ACTION <action1>] ;
   [MENUITEM <item2> ACTION <action2>] ;
   [MENUITEM <item3> ACTION <action3>] ;
   => <menu> := BuildMenu({[<item1>], [<item2>], [<item3>]}, ;
                          {[<action1>], [<action2>], [<action3>]})

// Usage
CREATE MENU oMainMenu ;
   MENUITEM "File" ACTION FileMenu() ;
   MENUITEM "Edit" ACTION EditMenu() ;
   MENUITEM "Help" ACTION HelpMenu()
```

#### Patterns with variable lists

```harbour
#xcommand LOG TO <file> MESSAGE <msg1> [, <msgN>] => ;
   WriteLog(<file>, <msg1> [+ Chr(13) + Chr(10) + <msgN>])

// Usage
LOG TO "app.log" MESSAGE "Application startup"
LOG TO "error.log" MESSAGE "Critical error", "Details: " + cDetail
```

---

## Creating macros: advanced syntax and usage

### Macros with dynamic evaluation

```harbour
// Macro that generates code dynamically
#define PROPERTY(name, type) ;
   PROTECTED: name AS type ; ;
   METHOD Set##name(value) INLINE (::name := value, Self) ; ;
   METHOD Get##name() INLINE ::name

// Usage in a class
CREATE CLASS Person
   PROPERTY(Name, STRING)
   PROPERTY(Age, NUMERIC)
   PROPERTY(Active, LOGICAL)
ENDCLASS

// Automatically generates:
// METHOD SetName(value) INLINE (::Name := value, Self)
// METHOD GetName() INLINE ::Name
// etc.
```

### Advanced template system

```harbour
// Template system for code generation
#define CRUD_OPERATIONS(table, key) ;
   FUNCTION Create##table(oData) ; ;
      RETURN InsertRecord(#table, <key>, oData) ; ;
   ; ;
   FUNCTION Read##table(xKey) ; ;
      RETURN SelectRecord(#table, <key>, xKey) ; ;
   ; ;
   FUNCTION Update##table(xKey, oData) ; ;
      RETURN UpdateRecord(#table, <key>, xKey, oData) ; ;
   ; ;
   FUNCTION Delete##table(xKey) ; ;
      RETURN DeleteRecord(#table, <key>, xKey)

// Generate CRUD operations for multiple tables
CRUD_OPERATIONS(User, "user_id")
CRUD_OPERATIONS(Product, "product_id")
CRUD_OPERATIONS(Order, "order_number")
```

### Macros for validation and verification

```harbour
// Assertion system
#define ASSERT(condition, message) ;
   if (!(condition)) ; ;
      Alert("ASSERTION FAILED: " + message) ; ;
      QUIT ; ;
   endif

#define REQUIRE_PARAM(param, type) ;
   if (PCount() < param .OR. ValType(param) != type) ; ;
      Alert("Parameter " + #param + " is required and must be " + type) ; ;
      RETURN NIL ; ;
   endif

// Usage in functions
FUNCTION CalculateDiscount(nAmount, nPercentage)
   REQUIRE_PARAM(nAmount, "N")
   REQUIRE_PARAM(nPercentage, "N")
   ASSERT(nPercentage >= 0 .AND. nPercentage <= 100, "Percentage must be between 0 and 100")
   
   RETURN nAmount * (nPercentage / 100)
```

### Macros for error handling

```harbour
// Macro for simulated try-catch
#define TRY BEGIN SEQUENCE
#define CATCH(errorVar) RECOVER USING errorVar
#define FINALLY ; ALWAYS
#define END_TRY END SEQUENCE

#define THROW(error) BREAK(error)

// Usage
FUNCTION ProcessFile(cName)
   LOCAL oError
   LOCAL cContent
   
   TRY
      cContent := MemoRead(cName)
      IF Empty(cContent)
         THROW(ErrorNew("Empty file"))
      ENDIF
      RETURN ProcessContent(cContent)
   CATCH(oError)
      Alert("Error processing file: " + oError:Description)
      RETURN NIL
   FINALLY
      // Cleanup code
      IF File(cName + ".lock")
         FErase(cName + ".lock")
      ENDIF
   END_TRY
```

### DSL for configuration

```harbour
// Create a DSL for application configuration
#command CONFIG SECTION <section> => BeginConfigSection(<section>)
#command END CONFIG => EndConfigSection()
#command SET <key> TO <value> => SetConfigValue(<key>, <value>)
#command SET <key> TO <value> TYPE <type> => SetConfigValue(<key>, <value>, <type>)

// Using the DSL
CONFIG SECTION "Database"
   SET "Host" TO "localhost"
   SET "Port" TO 5432 TYPE "N"
   SET "Database" TO "myapp"
   SET "User" TO "admin"
   SET "Password" TO "secret123"
END CONFIG

CONFIG SECTION "Application"
   SET "Debug" TO .T. TYPE "L"
   SET "LogLevel" TO "INFO"
   SET "MaxUsers" TO 100 TYPE "N"
END CONFIG
```

### Macros for interface generation

```harbour
// DSL for creating user interfaces
#command DIALOG <name> SIZE <width>, <height> => ;
   <name> := CreateDialog(<width>, <height>)

#command @ <row>, <col> SAY <text> SIZE <w>, <h> => ;
   AddLabel(GetCurrentDialog(), <row>, <col>, <text>, <w>, <h>)

#command @ <row>, <col> GET <var> SIZE <w>, <h> => ;
   AddTextBox(GetCurrentDialog(), <row>, <col>, @<var>, <w>, <h>)

#command @ <row>, <col> BUTTON <text> ACTION <action> SIZE <w>, <h> => ;
   AddButton(GetCurrentDialog(), <row>, <col>, <text>, <action>, <w>, <h>)

// Usage
DIALOG dlgPerson SIZE 400, 300
   @ 10, 10 SAY "Name:" SIZE 80, 20
   @ 10, 100 GET cName SIZE 200, 20
   
   @ 40, 10 SAY "Age:" SIZE 80, 20
   @ 40, 100 GET nAge SIZE 100, 20
   
   @ 70, 150 BUTTON "Accept" ACTION Accept() SIZE 80, 30
   @ 70, 240 BUTTON "Cancel" ACTION Cancel() SIZE 80, 30
```

### Advanced macro techniques

#### Token concatenation

```harbour
// Generate function names dynamically
#define DEFINE_GETTER_SETTER(prop) ;
   METHOD Get##prop() INLINE ::prop ; ;
   METHOD Set##prop(value) INLINE (::prop := value, Self)

// Generate multiple versions of a function
#define DEFINE_CONVERTERS(type) ;
   FUNCTION To##type##String(value) ; ;
      RETURN type##ToString(value) ; ;
   ; ;
   FUNCTION From##type##String(str) ; ;
      RETURN StringTo##type(str)

DEFINE_CONVERTERS(Date)
DEFINE_CONVERTERS(Number)
DEFINE_CONVERTERS(Logical)
```

#### Recursive macros

```harbour
// Macro for generating nested structures
#define REPEAT(code, times) ;
   RepeatCode(code, times)

// Helper function to repeat code
STATIC FUNCTION RepeatCode(cCode, nTimes)
   LOCAL i, cResult := ""
   FOR i := 1 TO nTimes
      cResult += StrTran(cCode, "##i##", LTrim(Str(i))) + Chr(13) + Chr(10)
   NEXT
   RETURN cResult
```

### Macro debugging

```harbour
// Macro to show generated code
#define DEBUG_MACRO(code) ;
   QOut("Generated code: " + #code) ; ;
   code

// Macro for execution tracing
#define TRACE_ENTRY(funcname) ;
   IF s_lTrace ; ;
      QOut("ENTER: " + funcname + "(" + ProcFile() + ":" + LTrim(Str(ProcLine())) + ")") ; ;
   ENDIF

#define TRACE_EXIT(funcname, result) ;
   IF s_lTrace ; ;
      QOut("EXIT: " + funcname + " -> " + ValToChar(result)) ; ;
   ENDIF
```

Harbour's preprocessor is an incredibly versatile tool that allows creating powerful abstractions, simplifying repetitive code, and creating specialized DSLs for specific domains. Its masterful use can dramatically transform code productivity and readability.