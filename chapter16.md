# Chapter 16: Memory Management and Booting Process

The booting process and memory management are fundamental aspects that determine the performance and stability of Harbour applications. This chapter explores how Harbour initializes its runtime environment and manages memory efficiently and safely.

---

## The booting process: from C `main` to the first PRG

The startup of a Harbour application is a complex process involving multiple initialization stages.

### Boot sequence

```mermaid
graph TD
    A[main() in C] --> B[hb_vmInit()]
    B --> C[VM Initialization]
    C --> D[Symbol Loading]
    D --> E[Memory Initialization]
    E --> F[GT Configuration]
    F --> G[Main() Execution]
    G --> H[hb_vmQuit()]
```

### Generated main() function

When you compile a Harbour program, a `main()` function in C is automatically generated:

```c
// Automatically generated C code
#include "hbvmopt.h"
#include "hbinit.h"

extern void _hb_MAIN(void);

int main(int argc, char *argv[])
{
   // Initialize the virtual machine
   hb_vmInit(HB_TRUE);
   
   // Configure command line arguments
   hb_cmdargInit(argc, argv);
   
   // Execute MAIN function of Harbour program
   hb_vmExecute(_hb_MAIN, NULL);
   
   // Clean up and terminate
   hb_vmQuit();
   
   return hb_vmExitCode();
}
```

### VM initialization (`hb_vmInit`)

The `hb_vmInit()` function performs the following operations:

1. **Stack initialization**: Configures the execution stack
2. **Symbol table**: Initializes global and local symbol tables
3. **Memory management**: Configures the heap and garbage collector
4. **Subsystems**: Initializes RDDs, GT, and other subsystems
5. **Environment variables**: Configures system variables

```c
void hb_vmInit(HB_BOOL fInit)
{
   if( fInit )
   {
      // Initialize memory heap
      hb_xinit();
      
      // Initialize symbol table
      hb_symTableInit();
      
      // Initialize VM stack
      hb_stackInit();
      
      // Initialize RDDs
      hb_rddInit();
      
      // Initialize GT (Graphic Terminal)
      hb_gtInit(NULL, NULL, NULL);
      
      // Initialize other subsystems
      hb_langInit();
      hb_cdpInit();
   }
}
```

### Loading libraries and modules

During startup, the following are automatically loaded:

```c
// Standard function symbols
extern HB_SYMB hb_vm_SymbolTable_ProgressName[];

// Runtime symbol initialization
HB_INIT_SYMBOLS_BEGIN(hb_vm_SymbolTable_ProgressName)
{
   // Register runtime functions
   hb_vmRegisterSymbols(hb_symbols_table, 
                       sizeof(hb_symbols_table)/sizeof(HB_SYMB));
}
HB_INIT_SYMBOLS_END()
```

---

## The memory manager: memory allocation and deallocation

Harbour implements its own memory manager that provides efficient allocation and memory leak detection.

### Memory management API

```c
// Main memory functions
void* hb_xgrab(HB_SIZE nSize);           // Allocate memory
void* hb_xrealloc(void* pMem, HB_SIZE nSize); // Resize
void  hb_xfree(void* pMem);              // Free memory
char* hb_xstrdup(const char* szText);    // Duplicate string
```

### Implementation with debug

When compiled with `HB_MEMORY_DEBUG`, the manager includes additional information:

```c
#ifdef HB_MEMORY_DEBUG
typedef struct _HB_MEMINFO
{
   HB_SIZE nSize;           // Block size
   const char* szFile;      // File where allocated
   int nLine;               // Line where allocated
   struct _HB_MEMINFO* pNext; // Next block in list
} HB_MEMINFO;
#endif

// Macro for allocation with debug
#define hb_xgrab(n)  hb_xgrab_debug(n, __FILE__, __LINE__)
```

### Memory pool for small objects

To optimize allocation of frequent small objects:

```c
// Pool for HB_ITEMs
#define HB_ITEM_POOL_SIZE 1024
static PHB_ITEM s_itemPool[HB_ITEM_POOL_SIZE];
static int s_itemPoolPos = 0;

PHB_ITEM hb_itemNew(PHB_ITEM pItem)
{
   if( pItem == NULL )
   {
      // Try to get from pool
      if( s_itemPoolPos > 0 )
      {
         pItem = s_itemPool[--s_itemPoolPos];
      }
      else
      {
         // Allocate new item if pool is empty
         pItem = (PHB_ITEM)hb_xgrab(sizeof(HB_ITEM));
      }
   }
   
   // Initialize item
   pItem->type = HB_IT_NIL;
   return pItem;
}
```

### String management with reference counting

```c
typedef struct _HB_STRING
{
   HB_SIZE nLen;      // String length
   HB_SIZE nAllocated; // Allocated size
   HB_COUNTER nRefs;   // Reference counter
   char* szText;       // String content
} HB_STRING;

// Increment reference
PHB_STRING hb_stringAddRef(PHB_STRING pString)
{
   if( pString )
   {
      HB_COUNTER_INC(&pString->nRefs);
   }
   return pString;
}

// Decrement reference and free if necessary
void hb_stringRelease(PHB_STRING pString)
{
   if( pString && HB_COUNTER_DEC(&pString->nRefs) == 0 )
   {
      hb_xfree(pString->szText);
      hb_xfree(pString);
   }
}
```

---

## The `Garbage Collector`: automating memory deallocation

Harbour's Garbage Collector (GC) automatically manages memory for complex objects like arrays, hashes, and objects.

### Mark and Sweep

The GC uses the classic mark and sweep algorithm:

```c
typedef struct _HB_GCITEM
{
   HB_SIZE nAllocated;    // Object size
   HB_COUNTER nRefs;      // References from stack/variables
   HB_BOOL fMarked;       // Mark for GC
   HB_GARBAGE_FUNC pCleanupFunc; // Cleanup function
   struct _HB_GCITEM* pNext;     // Next in list
} HB_GCITEM;
```

### Garbage Collector cycle

1. **Mark phase**: Marks all objects accessible from stack and global variables
2. **Sweep phase**: Frees unmarked objects
3. **Optional compaction**: Reorganizes memory to reduce fragmentation

```c
void hb_gcCollectAll(void)
{
   // Phase 1: Mark accessible objects
   hb_gcMark();
   
   // Phase 2: Sweep unmarked objects
   hb_gcSweep();
   
   // Phase 3: Clear marks for next cycle
   hb_gcClearMarks();
}
```

### Registering objects in the GC

Complex objects are automatically registered:

```c
// Create an array and register it in the GC
PHB_ITEM hb_arrayNew(HB_SIZE nLen)
{
   PHB_BASEARRAY pArray;
   PHB_ITEM pItem;
   
   // Allocate array structure
   pArray = (PHB_BASEARRAY)hb_gcAllocate(sizeof(HB_BASEARRAY),
                                        &hb_arrayGarbageCollector);
   
   // Configure array
   pArray->nLen = nLen;
   pArray->nAllocated = nLen;
   pArray->pItems = (PHB_ITEM)hb_xgrab(nLen * sizeof(HB_ITEM));
   
   // Create container item
   pItem = hb_itemNew(NULL);
   pItem->type = HB_IT_ARRAY;
   pItem->item.asArray.value = pArray;
   
   return pItem;
}
```

### Custom cleanup function

```c
// Cleanup function for arrays
static HB_GARBAGE_FUNC(hb_arrayGarbageCollector)
{
   PHB_BASEARRAY pArray = (PHB_BASEARRAY)Cargo;
   
   if( pArray )
   {
      // Free array elements
      if( pArray->pItems )
      {
         for( HB_SIZE i = 0; i < pArray->nLen; i++ )
         {
            hb_itemClear(&pArray->pItems[i]);
         }
         hb_xfree(pArray->pItems);
      }
      
      // The pArray structure is freed automatically
   }
}
```

### Manual GC control

```harbour
// From Harbour code
FUNCTION ControlGC()
   // Force garbage collection
   hb_gcAll()
   
   // Get statistics
   ? "Objects in memory:", hb_gcItemCount()
   ? "Total memory used:", hb_gcMemoryUsed()
   
   // Configure threshold for automatic activation
   hb_gcSetAutoThreshold(10000)  // Collect every 10000 allocations
   
   RETURN NIL
```

### GC optimizations

1. **Incremental collection**: GC can run in small portions
2. **Generational**: Young objects vs. old objects
3. **Adaptive thresholds**: GC adjusts according to usage pattern

```c
// GC configuration
typedef struct _HB_GC_CONFIG
{
   HB_SIZE nAutoThreshold;    // Threshold for automatic collection
   HB_SIZE nIncrementalStep;  // Incremental step size
   HB_BOOL fGenerational;     // Use generational collection
   double dGrowthFactor;      // Heap growth factor
} HB_GC_CONFIG;
```

### Circular reference detection

The GC can detect and resolve circular references:

```harbour
// Circular reference example
LOCAL oObj1 := TMyClass():New()
LOCAL oObj2 := TMyClass():New()

oObj1:reference := oObj2
oObj2:reference := oObj1

// GC can free these objects even though they have circular references
oObj1 := NIL
oObj2 := NIL
hb_gcAll()  // Will free both objects
```

---

## Advanced environment configuration

### Important environment variables

```bash
# Configure initial heap size
export HB_HEAPSIZE=8192

# Activate memory debug mode
export HB_MEMORY_DEBUG=1

# Configure GC threshold
export HB_GC_THRESHOLD=1000

# Dynamic library directory
export HB_LIB_PATH=/usr/local/harbour/lib
```

### Initialization hooks

```c
// Function called at application startup
HB_FUNC( HB_ATINIT )
{
   static HB_BOOL fInit = HB_FALSE;
   
   if( !fInit )
   {
      fInit = HB_TRUE;
      
      // Custom configurations
      hb_gcSetAutoThreshold(5000);
      
      // Register cleanup function
      hb_vmAtExit(my_cleanup_function, NULL);
   }
}

// Cleanup function at termination
static void my_cleanup_function(void* cargo)
{
   // Clean up application-specific resources
   HB_SYMBOL_UNUSED(cargo);
}
```

Harbour's memory management and booting system is designed to be robust, efficient, and predictable, providing the foundation for high-quality, high-performance applications.