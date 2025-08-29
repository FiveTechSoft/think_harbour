# Capítulo 16: Gestión de memoria y arranque (`Booting Process`)

Este capítulo explora los procesos internos de Harbour relacionados con la gestión de memoria y el proceso de arranque del sistema, desde la inicialización hasta la ejecución del primer código Harbour.

---

## El proceso de arranque: desde el `main` de C hasta el primer PRG

El proceso de arranque de Harbour involucra múltiples etapas de inicialización que preparan el entorno de ejecución antes de que se ejecute el primer código Harbour.

### Secuencia de arranque

#### 1. Función main() de C

```c
// Punto de entrada principal (simplificado)
int main( int argc, char * argv[] )
{
   // 1. Inicializar la máquina virtual
   hb_vmInit( HB_TRUE );
   
   // 2. Registrar símbolos del sistema
   hb_vmRegisterSymbols( /* símbolos del núcleo */ );
   
   // 3. Inicializar subsistemas
   hb_gtInit( argc, argv );
   hb_fsInit();
   hb_dateTimeInit();
   
   // 4. Procesar argumentos de línea de comandos
   hb_cmdargInit( argc, argv );
   
   // 5. Ejecutar función MAIN de Harbour
   hb_vmExecute( hb_dynsymFindName( "MAIN" ), 0 );
   
   // 6. Limpieza y finalización
   hb_vmQuit();
   
   return 0;
}
```

#### 2. Inicialización de la VM

```c
void hb_vmInit( HB_BOOL fInit )
{
   if( fInit )
   {
      // Inicializar pila de ejecución
      hb_stackInit();
      
      // Inicializar tabla de símbolos
      hb_symbolInit();
      
      // Inicializar sistema de memoria
      hb_memoryInit();
      
      // Inicializar garbage collector
      hb_gcInit();
      
      // Establecer manejadores de error por defecto
      hb_errorInit();
   }
}
```

#### 3. Registro de símbolos del núcleo

```c
// Símbolos fundamentales que siempre están disponibles
static HB_SYMB s_coreSymbols[] = {
   { "MAIN",      HB_FS_PUBLIC, { HB_FUNCNAME(MAIN) }, NULL },
   { "QOUT",      HB_FS_PUBLIC, { HB_FUNCNAME(QOUT) }, NULL },
   { "EVAL",      HB_FS_PUBLIC, { HB_FUNCNAME(EVAL) }, NULL },
   { "TYPE",      HB_FS_PUBLIC, { HB_FUNCNAME(TYPE) }, NULL },
   // ... más símbolos del núcleo
};
```

#### 4. Inicialización de subsistemas

**Terminal (GT) System:**
```c
void hb_gtInit( int argc, char * argv[] )
{
   // Detectar y cargar driver GT apropiado
   // (GTWIN en Windows, GTXWC en Linux con X11, etc.)
   hb_gtStartupInit();
}
```

**File System:**
```c
void hb_fsInit( void )
{
   // Inicializar tabla de handles de archivos
   // Configurar rutas de búsqueda por defecto
   // Inicializar codificación de caracteres
}
```

### Función MAIN de Harbour

Una vez completada la inicialización del sistema, se ejecuta la función MAIN del programa Harbour:

```harbour
FUNCTION Main()
   // Este es el punto de entrada de tu programa Harbour
   
   // Procesar argumentos de línea de comandos
   LOCAL aArgs := hb_AParams()
   
   // Tu código principal aquí
   ? "¡Aplicación Harbour iniciada!"
   
   // Llamar a otras funciones según sea necesario
   InicializarAplicacion()
   ProcesarDatos()
   
   RETURN 0  // Código de salida
```

---

## El gestor de memoria: asignación y liberación de memoria

Harbour implementa un gestor de memoria sofisticado que optimiza la asignación y liberación de memoria.

### Arquitectura del gestor de memoria

#### Pools de memoria

```c
// Estructura simplificada para pools de memoria
typedef struct _HB_MEMPOOL
{
   void *      pMemory;      // Bloque de memoria base
   HB_SIZE     nSize;        // Tamaño total del pool
   HB_SIZE     nUsed;        // Memoria utilizada
   HB_SIZE     nBlocks;      // Número de bloques
   struct _HB_MEMPOOL * pNext;  // Siguiente pool
} HB_MEMPOOL;
```

#### Estrategias de asignación

**Small Object Allocation:**
- Objetos pequeños (< 1KB) se asignan desde pools pre-asignados
- Reducción de fragmentación
- Asignación y liberación muy rápidas

**Large Object Allocation:**
- Objetos grandes se asignan directamente del sistema
- Uso de malloc/free del sistema operativo
- Tracking especial para objetos grandes

### Funciones de gestión de memoria

#### Asignación de memoria

```c
// Asignar memoria inicializada a cero
void * hb_xgrab( HB_SIZE nSize )
{
   void * pMem = hb_memoryAlloc( nSize );
   if( pMem )
   {
      memset( pMem, 0, nSize );
   }
   return pMem;
}

// Asignar memoria sin inicializar
void * hb_xalloc( HB_SIZE nSize )
{
   return hb_memoryAlloc( nSize );
}

// Redimensionar bloque de memoria
void * hb_xrealloc( void * pMem, HB_SIZE nSize )
{
   return hb_memoryRealloc( pMem, nSize );
}
```

#### Liberación de memoria

```c
// Liberar memoria
void hb_xfree( void * pMem )
{
   if( pMem )
   {
      hb_memoryFree( pMem );
   }
}

// Liberar y establecer puntero a NULL
#define hb_xfreeNull( p ) do { \
   if( (p) ) { \
      hb_xfree( (p) ); \
      (p) = NULL; \
   } \
} while(0)
```

### Detección de memory leaks

```c
#ifdef HB_MEMORY_DEBUG
// En modo debug, todas las asignaciones se rastrean
typedef struct _HB_MEMINFO
{
   void *      pMemory;      // Dirección de memoria
   HB_SIZE     nSize;        // Tamaño asignado
   const char* szFile;       // Archivo fuente
   int         nLine;        // Línea en el archivo
   HB_ULONG    nSequence;    // Número de secuencia
} HB_MEMINFO;

// Estadísticas de memoria
void hb_memoryShowStats( void )
{
   // Mostrar bloques no liberados
   // Estadísticas de uso de memoria
   // Detectar memory leaks
}
#endif
```

---

## El `Garbage Collector`: automatización de la liberación de memoria

El Garbage Collector (GC) de Harbour automatiza la liberación de objetos que ya no están siendo referenciados.

### Algoritmo del GC

#### Mark and Sweep

```c
void hb_gcCollectAll( void )
{
   // Fase 1: Mark - Marcar objetos alcanzables
   hb_gcMarkPhase();
   
   // Fase 2: Sweep - Liberar objetos no marcados
   hb_gcSweepPhase();
   
   // Fase 3: Compactación (opcional)
   hb_gcCompactPhase();
}
```

#### Fase de marcado

```c
void hb_gcMarkPhase( void )
{
   // Marcar desde las raíces (stack, símbolos globales, etc.)
   hb_gcMarkFromStack();
   hb_gcMarkFromSymbols();
   hb_gcMarkFromStatics();
   
   // Marcar transitivamente todos los objetos alcanzables
   hb_gcMarkTransitive();
}
```

#### Fase de liberación

```c
void hb_gcSweepPhase( void )
{
   PHB_GCITEM pItem = s_pGCItemList;
   
   while( pItem )
   {
      PHB_GCITEM pNext = pItem->pNext;
      
      if( !pItem->bMarked )
      {
         // Objeto no marcado - puede ser liberado
         hb_gcItemFree( pItem );
      }
      else
      {
         // Limpiar marca para próximo ciclo
         pItem->bMarked = HB_FALSE;
      }
      
      pItem = pNext;
   }
}
```

### Tipos de objetos gestionados por el GC

#### Arrays
```harbour
LOCAL aArray := { 1, 2, 3 }
// El array se libera automáticamente cuando sale de scope
```

#### Hashes
```harbour
LOCAL hHash := { "clave" => "valor" }
// El hash se libera automáticamente
```

#### Objetos
```harbour
LOCAL oObjeto := MiClase():New()
// El objeto se libera cuando no hay más referencias
```

#### Codeblocks
```harbour
LOCAL bBloque := {|| ? "Hola" }
// El codeblock se libera automáticamente
```

### Control manual del GC

```harbour
// Forzar ciclo completo de GC
hb_gcAll()

// Obtener estadísticas de memoria
LOCAL nMemUsed := hb_gcMemUsed()
LOCAL nMemTotal := hb_gcMemTotal()

? "Memoria usada:", nMemUsed, "de", nMemTotal

// Configurar comportamiento del GC
hb_gcSetAuto( .T. )  // Activar GC automático
```

### Optimizaciones del GC

#### GC incremental
- Ejecuta en pequeños ciclos para evitar pausas largas
- Mantiene la responsividad de la aplicación

#### GC generacional
- Objetos jóvenes se recolectan más frecuentemente
- Objetos viejos se recolectan menos frecuentemente

#### GC concurrente
- En aplicaciones multihilo, puede ejecutar en paralelo
- Minimiza la interferencia con hilos de aplicación

### Manejo de referencias circulares

```harbour
// Ejemplo de referencia circular
LOCAL oA := ObjetoA():New()
LOCAL oB := ObjetoB():New()

oA:oB := oB  // A referencia a B
oB:oA := oA  // B referencia a A

// El GC puede detectar y resolver estas referencias circulares
// automáticamente durante el ciclo de marcado
```

El sistema de gestión de memoria de Harbour proporciona tanto control manual como automatización inteligente, permitiendo que las aplicaciones funcionen eficientemente sin la complejidad de gestión manual de memoria en la mayoría de casos.