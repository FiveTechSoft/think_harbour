# Capítulo 15: El sistema extendido (`The Extend System`)

Este capítulo explora el sistema de extensión de Harbour, que permite la interoperabilidad entre código C y Harbour, así como la gestión de símbolos y el soporte para binarios independientes.

---

## Interoperabilidad de C y Harbour: creación de funciones externas

El sistema Extend de Harbour permite integrar código C nativo con aplicaciones Harbour, proporcionando acceso a librerías del sistema y optimizaciones de rendimiento.

### Estructura básica de una función C para Harbour

```c
#include "hbapi.h"

// Definición de función que puede ser llamada desde Harbour
HB_FUNC( MI_FUNCION_SUMA )
{
   // Obtener parámetros desde la pila de Harbour
   int nPrimero = hb_parni(1);    // Primer parámetro como entero
   int nSegundo = hb_parni(2);    // Segundo parámetro como entero
   
   // Realizar operación
   int nResultado = nPrimero + nSegundo;
   
   // Retornar resultado a Harbour
   hb_retni( nResultado );
}
```

### Llamada desde código Harbour

```harbour
// Una vez compilada e integrada la función C
LOCAL nResultado := MI_FUNCION_SUMA( 15, 27 )
? "El resultado es:", nResultado  // Imprime: El resultado es: 42
```

### Manejo de diferentes tipos de datos

```c
HB_FUNC( EJEMPLO_TIPOS )
{
   // Obtener parámetros de diferentes tipos
   const char* szTexto = hb_parc(1);           // String
   double dNumero = hb_parnd(2);               // Número decimal
   HB_BOOL bLogico = hb_parl(3);               // Lógico
   long nEntero = hb_parnl(4);                 // Entero largo
   
   // Verificar si el parámetro existe y es del tipo esperado
   if( hb_pcount() >= 1 && hb_parclen(1) > 0 )
   {
      // Procesar string...
   }
   
   // Retornar diferentes tipos
   if( bLogico )
   {
      hb_retc( "Verdadero" );                  // Retornar string
   }
   else
   {
      hb_retnd( dNumero * 2.0 );               // Retornar decimal
   }
}
```

### Gestión de arrays desde C

```c
HB_FUNC( PROCESAR_ARRAY )
{
   PHB_ITEM pArray = hb_param( 1, HB_IT_ARRAY );
   
   if( pArray )
   {
      HB_SIZE nLen = hb_arrayLen( pArray );
      HB_SIZE i;
      
      // Procesar cada elemento del array
      for( i = 1; i <= nLen; i++ )
      {
         PHB_ITEM pItem = hb_arrayGetItemPtr( pArray, i );
         
         if( hb_itemIsNumeric( pItem ) )
         {
            // Duplicar valores numéricos
            hb_arraySetNI( pArray, i, hb_itemGetNI( pItem ) * 2 );
         }
      }
      
      // Retornar el array modificado
      hb_itemReturn( pArray );
   }
   else
   {
      hb_ret();  // Retornar NIL si no es un array
   }
}
```

---

## La Tabla de Símbolos Globales: cómo se resuelven las llamadas a funciones

La tabla de símbolos globales es el mecanismo central para la resolución de nombres de funciones y variables públicas en Harbour.

### Estructura de la tabla de símbolos

```c
typedef struct _HB_SYMB
{
   const char *   szName;         // Nombre del símbolo
   union
   {
      HB_PCODEFUNC pFunPtr;       // Puntero a función nativa
      HB_BYTE *    pCodeFunc;     // Puntero a P-Code
   } value;
   HB_SYMBOLSCOPE scope;          // Alcance del símbolo
   HB_SYMTYPE     cType;          // Tipo de símbolo
} HB_SYMB, * PHB_SYMB;
```

### Proceso de resolución de símbolos

1. **Compilación**: El compilador registra todos los símbolos encontrados
2. **Enlazado**: Se resuelven las referencias entre módulos
3. **Ejecución**: La VM busca símbolos en la tabla global durante las llamadas

```harbour
// Cuando se hace una llamada a función
MiFuncion( param1, param2 )

// Harbour internamente:
// 1. Busca "MIFUNCION" en la tabla de símbolos
// 2. Verifica el tipo (función, procedimiento, etc.)
// 3. Ejecuta el código asociado
```

### Registro dinámico de símbolos

```c
// Registrar una función C en tiempo de ejecución
static HB_SYMB s_symbols[] = {
   { "MI_FUNCION_SUMA", HB_FS_PUBLIC, { HB_FUNCNAME(MI_FUNCION_SUMA) }, NULL }
};

// En la función de inicialización del módulo
void hb_mi_modulo_init( void )
{
   hb_vmRegisterSymbols( s_symbols, sizeof(s_symbols) / sizeof(HB_SYMB) );
}
```

---

## Tablas de símbolos locales: gestión del alcance de las variables

Las tablas de símbolos locales manejan variables con alcance limitado (LOCAL, STATIC, PRIVATE).

### Jerarquía de alcance

```harbour
FUNCTION Ejemplo()
   LOCAL nLocal := 10        // Símbolo local al función
   STATIC nStatic := 20      // Símbolo estático (persistente)
   PRIVATE nPrivate := 30    // Símbolo privado (visible en subfunciones)
   
   SubFuncion()
   
   RETURN NIL

FUNCTION SubFuncion()
   // nLocal no es visible aquí
   // nStatic es visible si se declara aquí también
   // nPrivate ES visible aquí (herencia dinámica)
   
   ? nPrivate  // Imprime: 30
   
   RETURN NIL
```

### Gestión de marcos de variables

```c
// Estructura simplificada para marcos de variables locales
typedef struct _HB_STACK_LOCAL
{
   PHB_ITEM *  pItems;       // Array de variables locales
   HB_SIZE     nBase;        // Base en la pila
   HB_SIZE     nLen;         // Número de variables
   struct _HB_STACK_LOCAL * pPrev;  // Marco anterior
} HB_STACK_LOCAL;
```

### Resolución de variables

1. **LOCAL**: Se busca primero en el marco local actual
2. **STATIC**: Se busca en la tabla de estáticas del módulo
3. **PRIVATE**: Se busca en la pila de marcos privados
4. **PUBLIC**: Se busca en la tabla global de públicas

---

## Soporte `HRB`: la ejecución de binarios de Harbour independientes

HRB (Harbour Runtime Binary) permite compilar y ejecutar código Harbour de forma dinámica.

### ¿Qué es un archivo HRB?

Un archivo HRB contiene:
- P-Code compilado
- Tabla de símbolos
- Metadatos del módulo
- Dependencias de librerías

### Compilación a HRB

```bash
# Compilar a archivo HRB
hbmk2 -hbhrb mi_modulo.prg

# Esto genera: mi_modulo.hrb
```

### Carga y ejecución dinámica

```harbour
FUNCTION CargarModuloDinamico()
   LOCAL pHRB
   LOCAL bResultado
   
   // Cargar archivo HRB
   pHRB := hb_hrbLoad( "mi_modulo.hrb" )
   
   IF pHRB != NIL
      // Obtener función del módulo cargado
      bResultado := hb_hrbGetFunSym( pHRB, "FUNCION_DEL_MODULO" )
      
      IF bResultado != NIL
         // Ejecutar la función
         LOCAL nResult := Eval( bResultado, param1, param2 )
         ? "Resultado:", nResult
      ENDIF
      
      // Descargar módulo cuando ya no se necesite
      hb_hrbUnload( pHRB )
   ENDIF
   
   RETURN NIL
```

### Ventajas del sistema HRB

#### Modularidad
- Cargar funcionalidad bajo demanda
- Reducir el tamaño del ejecutable principal
- Actualizar módulos sin recompilar la aplicación completa

#### Flexibilidad
```harbour
// Cargar diferentes implementaciones según configuración
LOCAL cModulo := iif( lModoDesarrollo, "debug.hrb", "release.hrb" )
LOCAL pHRB := hb_hrbLoad( cModulo )
```

#### Plugins y extensiones
```harbour
FUNCTION CargarPlugins()
   LOCAL aArchivos := Directory( "plugins/*.hrb" )
   LOCAL i
   
   FOR i := 1 TO Len( aArchivos )
      LOCAL pPlugin := hb_hrbLoad( "plugins/" + aArchivos[i][1] )
      
      IF pPlugin != NIL
         // Registrar funciones del plugin
         RegistrarPlugin( pPlugin )
      ENDIF
   NEXT
   
   RETURN NIL
```

### Gestión de memoria y recursos

- **Carga bajo demanda**: Los módulos HRB solo se cargan cuando se necesitan
- **Descarga automática**: El garbage collector puede descargar módulos no utilizados
- **Compartición de código**: Múltiples instancias pueden compartir el mismo código HRB
- **Aislamiento**: Los módulos HRB mantienen sus propios espacios de símbolos

### Casos de uso comunes

1. **Sistemas de plugins**: Permite extensibilidad sin modificar el núcleo
2. **Actualizaciones remotas**: Descargar y aplicar actualizaciones de funcionalidad
3. **Configuración dinámica**: Cargar diferentes comportamientos según configuración
4. **Optimización de memoria**: Cargar solo las funciones necesarias
5. **Desarrollo modular**: Facilita el trabajo en equipo con módulos independientes

El sistema Extend proporciona una base sólida para la extensibilidad y modularidad en aplicaciones Harbour, permitiendo tanto la integración con código nativo como la carga dinámica de funcionalidad Harbour.