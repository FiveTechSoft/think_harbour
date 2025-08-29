# Chapter 15: The Extend System

Harbour's extend system is one of its most powerful features, enabling seamless integration between Harbour code and extensions written in C/C++. This system provides the necessary infrastructure to extend the language's capabilities, access operating system APIs, and create high-performance libraries.

---

## C and Harbour interoperability: creating external functions

The extend system allows functions written in C to appear as native Harbour functions, maintaining type compatibility and automatic memory management.

### Anatomy of an extended function

A C function that can be called from Harbour must follow a specific structure:

```c
#include "hbapi.h"

HB_FUNC( MY_C_FUNCTION )
{
   // Get parameters from Harbour
   int nParam1 = hb_parni(1);        // First parameter as integer
   const char* szParam2 = hb_parc(2); // Second parameter as string
   
   // Process data
   int nResult = nParam1 * 2;
   
   // Return result to Harbour
   hb_retni(nResult);
}
```

### Macros for function definition

Harbour provides macros that simplify definition:

```c
// Function without parameters
HB_FUNC( GET_VERSION )
{
   hb_retc("My Library v1.0");
}

// Function with multiple parameters
HB_FUNC( CALCULATE_AREA )
{
   double dWidth = hb_parnd(1);
   double dHeight = hb_parnd(2);
   
   if( HB_ISNUM(1) && HB_ISNUM(2) )
   {
      hb_retnd(dWidth * dHeight);
   }
   else
   {
      hb_retnd(0.0);
   }
}
```

### Parameter validation

It's crucial to validate received parameters:

```c
HB_FUNC( SAFE_FUNCTION )
{
   // Check number of parameters
   if( hb_pcount() >= 2 )
   {
      // Check specific types
      if( HB_ISCHAR(1) && HB_ISNUM(2) )
      {
         const char* szText = hb_parc(1);
         int nRepeat = hb_parni(2);
         
         // Process...
         hb_retl(HB_TRUE);  // Success
      }
      else
      {
         hb_errRT_BASE_SubstR(EG_ARG, 0, NULL, HB_ERR_FUNCNAME, 2, 
                              hb_paramError(1), hb_paramError(2));
      }
   }
   else
   {
      hb_errRT_BASE_SubstR(EG_ARG, 0, NULL, HB_ERR_FUNCNAME, 0);
   }
}
```

### Handling arrays from C

```c
HB_FUNC( PROCESS_ARRAY )
{
   PHB_ITEM pArray = hb_param(1, HB_IT_ARRAY);
   
   if( pArray )
   {
      HB_SIZE nLen = hb_arrayLen(pArray);
      double dSum = 0.0;
      
      for( HB_SIZE i = 1; i <= nLen; i++ )
      {
         if( hb_arrayIsNumeric(pArray, i) )
         {
            dSum += hb_arrayGetND(pArray, i);
         }
      }
      
      hb_retnd(dSum);
   }
   else
   {
      hb_retnd(0.0);
   }
}
```

---

## The Global Symbol Table: how function calls are resolved

The Global Symbol Table (GST) is the central mechanism for function name resolution in Harbour.

### GST structure

```c
typedef struct _HB_SYMB
{
   const char* szName;        // Function name
   union
   {
      HB_PCODE pCodeFunc;     // P-Code function
      HB_FUNC_PTR pFunPtr;    // Pointer to C function
   } value;
   HB_SYMBOLSCOPE scope;      // Symbol scope
} HB_SYMB, * PHB_SYMB;
```

### C function registration

When a library is loaded, its functions are automatically registered:

```c
// Library symbol table definition
static const HB_SYMB symbols_table[] = {
   { "MY_C_FUNCTION",    {HB_FS_PUBLIC}, {HB_FUNCNAME(MY_C_FUNCTION)},    NULL },
   { "CALCULATE_AREA",   {HB_FS_PUBLIC}, {HB_FUNCNAME(CALCULATE_AREA)},   NULL },
   { "GET_VERSION",      {HB_FS_PUBLIC}, {HB_FUNCNAME(GET_VERSION)},      NULL }
};

// Library initialization function
HB_INIT_SYMBOLS_BEGIN(my_library)
{
   hb_vmRegisterSymbols(symbols_table, sizeof(symbols_table) / sizeof(HB_SYMB));
}
HB_INIT_SYMBOLS_END()
```

### Call resolution

The resolution process follows these steps:

1. **Local table search**: First searches in the current context
2. **GST search**: If not found, searches in the global table
3. **Dynamic loading**: If still not found, attempts to load from dynamic libraries
4. **Error**: If it cannot be resolved, generates a function not found error

```harbour
// Resolution example
FUNCTION TestResolution()
   LOCAL result
   
   // Call to standard function (always available)
   result := Len("test")
   
   // Call to extended function (must be registered)
   result := MY_C_FUNCTION(42)
   
   // Call to user-defined function
   result := MyCustomFunction()
   
   RETURN result
```

---

## Local symbol tables: managing variable scope

In addition to the global table, Harbour maintains local symbol tables to manage the scope of variables and private functions.

### Scope hierarchy

```
Global Table (GST)
├── Module 1
│   ├── Function A
│   │   ├── LOCAL Variables
│   │   └── STATIC Variables
│   └── Function B
└── Module 2
    └── Function C
```

### STATIC variables

STATIC variables have their own symbol table per module:

```harbour
// file1.prg
STATIC s_nCounter := 0

FUNCTION IncrementCounter()
   RETURN ++s_nCounter  // Access to module's local STATIC variable

// file2.prg
STATIC s_nCounter := 100  // Different STATIC variable

FUNCTION GetCounter()
   RETURN s_nCounter  // Access to its own STATIC variable
```

### Implementation in C

```c
// Definition of STATIC variables in C extensions
static int s_nCounterC = 0;

HB_FUNC( INCREMENT_COUNTER_C )
{
   hb_retni(++s_nCounterC);
}

HB_FUNC( GET_COUNTER_C )
{
   hb_retni(s_nCounterC);
}
```

---

## `HRB` support: execution of independent Harbour binaries

HRB (Harbour Binary) files allow compiling Harbour code into independent executable modules that can be dynamically loaded.

### Creating HRB files

```bash
# Compile to HRB
harbour -gh my_module.prg

# Or using hbmk2
hbmk2 -hbhrb my_module.prg
```

### Dynamic HRB loading

```harbour
FUNCTION LoadDynamicModule()
   LOCAL pHRB
   LOCAL result
   
   // Load HRB file
   pHRB := hb_hrbLoad("my_module.hrb")
   
   IF !Empty(pHRB)
      // Execute function from loaded module
      result := hb_hrbCall(pHRB, "MODULE_FUNCTION", "parameter1", 123)
      
      // Free resources
      hb_hrbUnload(pHRB)
   ENDIF
   
   RETURN result
```

### HRB module example

```harbour
// my_module.prg (to compile to HRB)

FUNCTION MODULE_FUNCTION(cParam, nParam)
   LOCAL cResult
   
   cResult := "Processed: " + cParam + " with number: " + LTrim(Str(nParam))
   
   RETURN cResult

FUNCTION ANOTHER_MODULE_FUNCTION()
   RETURN "Additional function from HRB module"

// STATIC functions are not accessible from outside the HRB
STATIC FUNCTION PrivateFunction()
   RETURN "This function is not externally accessible"
```

### Advantages of HRB files

1. **Modularity**: Logical code separation
2. **On-demand loading**: Modules are loaded only when needed
3. **Independent updates**: Modules can be updated without recompiling the entire application
4. **Distribution**: Facilitates distribution of extensions and plugins

### Complete HRB API

```harbour
// Main functions for HRB handling
pHRB := hb_hrbLoad(cFileName)           // Load HRB file
result := hb_hrbCall(pHRB, cFunc, ...)  // Call HRB function
hb_hrbUnload(pHRB)                      // Free HRB from memory

// Information functions
aFunctions := hb_hrbGetFunList(pHRB)    // List of available functions
lExists := hb_hrbGetFunSym(pHRB, cFunc) // Check if function exists
```

---

## Advanced integration with external libraries

### Wrapping C libraries

To integrate existing C libraries:

```c
// Wrapper for external library function
#include "external_lib.h"

HB_FUNC( USE_EXTERNAL_LIBRARY )
{
   const char* szInput = hb_parc(1);
   char szBuffer[256];
   
   // Call external library function
   int nResult = external_library_function(szInput, szBuffer, sizeof(szBuffer));
   
   if( nResult == 0 )
   {
      hb_retc(szBuffer);
   }
   else
   {
      hb_retc("");
   }
}
```

### Handling complex structures

```c
// Define structure
typedef struct
{
   char szName[50];
   int nAge;
   double dSalary;
} PERSON;

HB_FUNC( CREATE_PERSON )
{
   PERSON* pPerson = (PERSON*)hb_xgrab(sizeof(PERSON));
   
   hb_strncpy(pPerson->szName, hb_parc(1), sizeof(pPerson->szName) - 1);
   pPerson->nAge = hb_parni(2);
   pPerson->dSalary = hb_parnd(3);
   
   // Return as pointer
   hb_retptr(pPerson);
}

HB_FUNC( FREE_PERSON )
{
   PERSON* pPerson = (PERSON*)hb_parptr(1);
   if( pPerson )
   {
      hb_xfree(pPerson);
   }
}
```

Harbour's extend system provides a solid foundation for creating hybrid applications that combine Harbour's flexibility with C's performance, always maintaining type safety and automatic memory management.