# Capítulo 16: Gestión de memoria y arranque (`Booting Process`)

El proceso de arranque y la gestión de memoria son aspectos fundamentales que determinan el rendimiento y la estabilidad de las aplicaciones Harbour. Este capítulo explora cómo Harbour inicializa su entorno de ejecución y gestiona la memoria de manera eficiente y segura.

---

## El proceso de arranque: desde el `main` de C hasta el primer PRG

El arranque de una aplicación Harbour es un proceso complejo que involucra múltiples etapas de inicialización.

### Secuencia de arranque

```mermaid
graph TD
    A[main() en C] --> B[hb_vmInit()]
    B --> C[Inicialización VM]
    C --> D[Carga de símbolos]
    D --> E[Inicialización de memoria]
    E --> F[Configuración GT]
    F --> G[Ejecución de Main()]
    G --> H[hb_vmQuit()]
```

### Función main() generada

Cuando compilas un programa Harbour, se genera automáticamente una función `main()` en C:

```c
// Código C generado automáticamente
#include "hbvmopt.h"
#include "hbinit.h"

extern void _hb_MAIN(void);

int main(int argc, char *argv[])
{
   // Inicializar la máquina virtual
   hb_vmInit(HB_TRUE);
   
   // Configurar argumentos de línea de comandos
   hb_cmdargInit(argc, argv);
   
   // Ejecutar función MAIN del programa Harbour
   hb_vmExecute(_hb_MAIN, NULL);
   
   // Limpiar y terminar
   hb_vmQuit();
   
   return hb_vmExitCode();
}
```

### Inicialización de la VM (`hb_vmInit`)

La función `hb_vmInit()` realiza las siguientes operaciones:

1. **Inicialización de la pila**: Configura la pila de ejecución
2. **Tabla de símbolos**: Inicializa las tablas de símbolos globales y locales
3. **Gestión de memoria**: Configura el heap y el garbage collector
4. **Subsistemas**: Inicializa RDDs, GT, y otros subsistemas
5. **Variables de entorno**: Configura variables del sistema

```c
void hb_vmInit(HB_BOOL fInit)
{
   if( fInit )
   {
      // Inicializar heap de memoria
      hb_xinit();
      
      // Inicializar tabla de símbolos
      hb_symTableInit();
      
      // Inicializar pila de la VM
      hb_stackInit();
      
      // Inicializar RDDs
      hb_rddInit();
      
      // Inicializar GT (Graphic Terminal)
      hb_gtInit(NULL, NULL, NULL);
      
      // Inicializar otros subsistemas
      hb_langInit();
      hb_cdpInit();
   }
}
```

### Carga de bibliotecas y módulos

Durante el arranque, se cargan automáticamente:

```c
// Símbolos de funciones estándar
extern HB_SYMB hb_vm_SymbolTable_ProgressName[];

// Inicialización de símbolos de runtime
HB_INIT_SYMBOLS_BEGIN(hb_vm_SymbolTable_ProgressName)
{
   // Registrar funciones del runtime
   hb_vmRegisterSymbols(hb_symbols_table, 
                       sizeof(hb_symbols_table)/sizeof(HB_SYMB));
}
HB_INIT_SYMBOLS_END()
```

---

## El gestor de memoria: asignación y liberación de memoria

Harbour implementa su propio gestor de memoria que proporciona asignación eficiente y detección de memory leaks.

### API de gestión de memoria

```c
// Funciones principales de memoria
void* hb_xgrab(HB_SIZE nSize);           // Asignar memoria
void* hb_xrealloc(void* pMem, HB_SIZE nSize); // Redimensionar
void  hb_xfree(void* pMem);              // Liberar memoria
char* hb_xstrdup(const char* szText);    // Duplicar cadena
```

### Implementación con debug

Cuando se compila con `HB_MEMORY_DEBUG`, el gestor incluye información adicional:

```c
#ifdef HB_MEMORY_DEBUG
typedef struct _HB_MEMINFO
{
   HB_SIZE nSize;           // Tamaño del bloque
   const char* szFile;      // Archivo donde se asignó
   int nLine;               // Línea donde se asignó
   struct _HB_MEMINFO* pNext; // Siguiente bloque en la lista
} HB_MEMINFO;
#endif

// Macro para asignación con debug
#define hb_xgrab(n)  hb_xgrab_debug(n, __FILE__, __LINE__)
```

### Pool de memoria para objetos pequeños

Para optimizar la asignación de objetos pequeños frecuentes:

```c
// Pool para HB_ITEMs
#define HB_ITEM_POOL_SIZE 1024
static PHB_ITEM s_itemPool[HB_ITEM_POOL_SIZE];
static int s_itemPoolPos = 0;

PHB_ITEM hb_itemNew(PHB_ITEM pItem)
{
   if( pItem == NULL )
   {
      // Intentar obtener del pool
      if( s_itemPoolPos > 0 )
      {
         pItem = s_itemPool[--s_itemPoolPos];
      }
      else
      {
         // Asignar nuevo item si el pool está vacío
         pItem = (PHB_ITEM)hb_xgrab(sizeof(HB_ITEM));
      }
   }
   
   // Inicializar item
   pItem->type = HB_IT_NIL;
   return pItem;
}
```

### Gestión de cadenas con conteo de referencias

```c
typedef struct _HB_STRING
{
   HB_SIZE nLen;      // Longitud de la cadena
   HB_SIZE nAllocated; // Tamaño asignado
   HB_COUNTER nRefs;   // Contador de referencias
   char* szText;       // Contenido de la cadena
} HB_STRING;

// Incrementar referencia
PHB_STRING hb_stringAddRef(PHB_STRING pString)
{
   if( pString )
   {
      HB_COUNTER_INC(&pString->nRefs);
   }
   return pString;
}

// Decrementar referencia y liberar si es necesario
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

## El `Garbage Collector`: automatización de la liberación de memoria

El Garbage Collector (GC) de Harbour gestiona automáticamente la memoria de objetos complejos como arrays, hashes y objetos.

### Marcado y barrido (Mark & Sweep)

El GC utiliza el algoritmo clásico de marcado y barrido:

```c
typedef struct _HB_GCITEM
{
   HB_SIZE nAllocated;    // Tamaño del objeto
   HB_COUNTER nRefs;      // Referencias desde la pila/variables
   HB_BOOL fMarked;       // Marca para el GC
   HB_GARBAGE_FUNC pCleanupFunc; // Función de limpieza
   struct _HB_GCITEM* pNext;     // Siguiente en la lista
} HB_GCITEM;
```

### Ciclo del Garbage Collector

1. **Fase de marcado**: Marca todos los objetos accesibles desde la pila y variables globales
2. **Fase de barrido**: Libera los objetos no marcados
3. **Compactación opcional**: Reorganiza la memoria para reducir fragmentación

```c
void hb_gcCollectAll(void)
{
   // Fase 1: Marcar objetos accesibles
   hb_gcMark();
   
   // Fase 2: Barrer objetos no marcados
   hb_gcSweep();
   
   // Fase 3: Limpiar marcas para el siguiente ciclo
   hb_gcClearMarks();
}
```

### Registro de objetos en el GC

Los objetos complejos se registran automáticamente:

```c
// Crear un array y registrarlo en el GC
PHB_ITEM hb_arrayNew(HB_SIZE nLen)
{
   PHB_BASEARRAY pArray;
   PHB_ITEM pItem;
   
   // Asignar estructura del array
   pArray = (PHB_BASEARRAY)hb_gcAllocate(sizeof(HB_BASEARRAY),
                                        &hb_arrayGarbageCollector);
   
   // Configurar array
   pArray->nLen = nLen;
   pArray->nAllocated = nLen;
   pArray->pItems = (PHB_ITEM)hb_xgrab(nLen * sizeof(HB_ITEM));
   
   // Crear item contenedor
   pItem = hb_itemNew(NULL);
   pItem->type = HB_IT_ARRAY;
   pItem->item.asArray.value = pArray;
   
   return pItem;
}
```

### Función de limpieza personalizada

```c
// Función de limpieza para arrays
static HB_GARBAGE_FUNC(hb_arrayGarbageCollector)
{
   PHB_BASEARRAY pArray = (PHB_BASEARRAY)Cargo;
   
   if( pArray )
   {
      // Liberar elementos del array
      if( pArray->pItems )
      {
         for( HB_SIZE i = 0; i < pArray->nLen; i++ )
         {
            hb_itemClear(&pArray->pItems[i]);
         }
         hb_xfree(pArray->pItems);
      }
      
      // La estructura pArray se libera automáticamente
   }
}
```

### Control manual del GC

```harbour
// Desde código Harbour
FUNCTION ControlGC()
   // Forzar recolección de basura
   hb_gcAll()
   
   // Obtener estadísticas
   ? "Objetos en memoria:", hb_gcItemCount()
   ? "Memoria total usada:", hb_gcMemoryUsed()
   
   // Configurar umbral para activación automática
   hb_gcSetAutoThreshold(10000)  // Recolectar cada 10000 asignaciones
   
   RETURN NIL
```

### Optimizaciones del GC

1. **Recolección incremental**: El GC puede ejecutarse en pequeñas porciones
2. **Generacional**: Objetos jóvenes vs. objetos antiguos
3. **Umbrales adaptativos**: El GC se ajusta según el patrón de uso

```c
// Configuración del GC
typedef struct _HB_GC_CONFIG
{
   HB_SIZE nAutoThreshold;    // Umbral para recolección automática
   HB_SIZE nIncrementalStep;  // Tamaño del paso incremental
   HB_BOOL fGenerational;     // Usar recolección generacional
   double dGrowthFactor;      // Factor de crecimiento del heap
} HB_GC_CONFIG;
```

### Detección de referencias circulares

El GC puede detectar y resolver referencias circulares:

```harbour
// Ejemplo de referencia circular
LOCAL oObj1 := TMiClase():New()
LOCAL oObj2 := TMiClase():New()

oObj1:referencia := oObj2
oObj2:referencia := oObj1

// El GC puede liberar estos objetos aunque tengan referencias circulares
oObj1 := NIL
oObj2 := NIL
hb_gcAll()  // Liberará ambos objetos
```

---

## Configuración avanzada del entorno

### Variables de entorno importantes

```bash
# Configurar el tamaño inicial del heap
export HB_HEAPSIZE=8192

# Activar modo debug de memoria
export HB_MEMORY_DEBUG=1

# Configurar umbral del GC
export HB_GC_THRESHOLD=1000

# Directorio de bibliotecas dinámicas
export HB_LIB_PATH=/usr/local/harbour/lib
```

### Hooks de inicialización

```c
// Función llamada al inicio de la aplicación
HB_FUNC( HB_ATINIT )
{
   static HB_BOOL fInit = HB_FALSE;
   
   if( !fInit )
   {
      fInit = HB_TRUE;
      
      // Configuraciones personalizadas
      hb_gcSetAutoThreshold(5000);
      
      // Registrar función de cleanup
      hb_vmAtExit(mi_funcion_cleanup, NULL);
   }
}

// Función de limpieza al terminar
static void mi_funcion_cleanup(void* cargo)
{
   // Limpiar recursos específicos de la aplicación
   HB_SYMBOL_UNUSED(cargo);
}
```

El sistema de gestión de memoria y arranque de Harbour está diseñado para ser robusto, eficiente y predecible, proporcionando las bases para aplicaciones de alta calidad y rendimiento.