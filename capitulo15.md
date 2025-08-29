# Capítulo 15: El sistema extendido (`The Extend System`)

El sistema extendido de Harbour es una de sus características más poderosas, permitiendo la integración perfecta entre código Harbour y extensiones escritas en C/C++. Este sistema proporciona la infraestructura necesaria para extender las capacidades del lenguaje, acceder a APIs del sistema operativo y crear bibliotecas de alto rendimiento.

---

## Interoperabilidad de C y Harbour: creación de funciones externas

El sistema extendido permite que las funciones escritas en C aparezcan como funciones nativas de Harbour, manteniendo la compatibilidad de tipos y el manejo automático de memoria.

### Anatomía de una función extendida

Una función C que puede ser llamada desde Harbour debe seguir una estructura específica:

```c
#include "hbapi.h"

HB_FUNC( MI_FUNCION_C )
{
   // Obtener parámetros desde Harbour
   int nParam1 = hb_parni(1);        // Primer parámetro como entero
   const char* szParam2 = hb_parc(2); // Segundo parámetro como cadena
   
   // Procesar datos
   int nResultado = nParam1 * 2;
   
   // Retornar resultado a Harbour
   hb_retni(nResultado);
}
```

### Macros para definición de funciones

Harbour proporciona macros que simplifican la definición:

```c
// Función sin parámetros
HB_FUNC( OBTENER_VERSION )
{
   hb_retc("Mi Librería v1.0");
}

// Función con múltiples parámetros
HB_FUNC( CALCULAR_AREA )
{
   double dAncho = hb_parnd(1);
   double dAlto = hb_parnd(2);
   
   if( HB_ISNUM(1) && HB_ISNUM(2) )
   {
      hb_retnd(dAncho * dAlto);
   }
   else
   {
      hb_retnd(0.0);
   }
}
```

### Verificación de parámetros

Es crucial validar los parámetros recibidos:

```c
HB_FUNC( FUNCION_SEGURA )
{
   // Verificar número de parámetros
   if( hb_pcount() >= 2 )
   {
      // Verificar tipos específicos
      if( HB_ISCHAR(1) && HB_ISNUM(2) )
      {
         const char* szTexto = hb_parc(1);
         int nRepetir = hb_parni(2);
         
         // Procesar...
         hb_retl(HB_TRUE);  // Éxito
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

### Manejo de arrays desde C

```c
HB_FUNC( PROCESAR_ARRAY )
{
   PHB_ITEM pArray = hb_param(1, HB_IT_ARRAY);
   
   if( pArray )
   {
      HB_SIZE nLen = hb_arrayLen(pArray);
      double dSuma = 0.0;
      
      for( HB_SIZE i = 1; i <= nLen; i++ )
      {
         if( hb_arrayIsNumeric(pArray, i) )
         {
            dSuma += hb_arrayGetND(pArray, i);
         }
      }
      
      hb_retnd(dSuma);
   }
   else
   {
      hb_retnd(0.0);
   }
}
```

---

## La Tabla de Símbolos Globales: cómo se resuelven las llamadas a funciones

La Tabla de Símbolos Globales (GST - Global Symbol Table) es el mecanismo central para la resolución de nombres de funciones en Harbour.

### Estructura de la GST

```c
typedef struct _HB_SYMB
{
   const char* szName;        // Nombre de la función
   union
   {
      HB_PCODE pCodeFunc;     // Función P-Code
      HB_FUNC_PTR pFunPtr;    // Puntero a función C
   } value;
   HB_SYMBOLSCOPE scope;      // Ámbito del símbolo
} HB_SYMB, * PHB_SYMB;
```

### Registro de funciones C

Cuando se carga una librería, sus funciones se registran automáticamente:

```c
// Definición de la tabla de símbolos de la librería
static const HB_SYMB symbols_table[] = {
   { "MI_FUNCION_C",    {HB_FS_PUBLIC}, {HB_FUNCNAME(MI_FUNCION_C)},    NULL },
   { "CALCULAR_AREA",   {HB_FS_PUBLIC}, {HB_FUNCNAME(CALCULAR_AREA)},   NULL },
   { "OBTENER_VERSION", {HB_FS_PUBLIC}, {HB_FUNCNAME(OBTENER_VERSION)}, NULL }
};

// Función de inicialización de la librería
HB_INIT_SYMBOLS_BEGIN(mi_libreria)
{
   hb_vmRegisterSymbols(symbols_table, sizeof(symbols_table) / sizeof(HB_SYMB));
}
HB_INIT_SYMBOLS_END()
```

### Resolución de llamadas

El proceso de resolución sigue estos pasos:

1. **Búsqueda en tabla local**: Primero se busca en el contexto actual
2. **Búsqueda en GST**: Si no se encuentra, se busca en la tabla global
3. **Carga dinámica**: Si aún no se encuentra, se intenta cargar desde bibliotecas dinámicas
4. **Error**: Si no se puede resolver, se genera un error de función no encontrada

```harbour
// Ejemplo de resolución
FUNCTION TestResolucion()
   LOCAL resultado
   
   // Llamada a función estándar (siempre disponible)
   resultado := Len("test")
   
   // Llamada a función extendida (debe estar registrada)
   resultado := MI_FUNCION_C(42)
   
   // Llamada a función definida por usuario
   resultado := MiFuncionPersonalizada()
   
   RETURN resultado
```

---

## Tablas de símbolos locales: gestión del alcance de las variables

Además de la tabla global, Harbour mantiene tablas de símbolos locales para gestionar el alcance de variables y funciones privadas.

### Jerarquía de alcance

```
Tabla Global (GST)
├── Módulo 1
│   ├── Función A
│   │   ├── Variables LOCAL
│   │   └── Variables STATIC
│   └── Función B
└── Módulo 2
    └── Función C
```

### Variables STATIC

Las variables STATIC tienen su propia tabla de símbolos por módulo:

```harbour
// archivo1.prg
STATIC s_nContador := 0

FUNCTION IncrementarContador()
   RETURN ++s_nContador  // Acceso a variable STATIC local del módulo

// archivo2.prg
STATIC s_nContador := 100  // Diferente variable STATIC

FUNCTION ObtenerContador()
   RETURN s_nContador  // Acceso a su propia variable STATIC
```

### Implementación en C

```c
// Definición de variables STATIC en extensiones C
static int s_nContadorC = 0;

HB_FUNC( INCREMENTAR_CONTADOR_C )
{
   hb_retni(++s_nContadorC);
}

HB_FUNC( OBTENER_CONTADOR_C )
{
   hb_retni(s_nContadorC);
}
```

---

## Soporte `HRB`: la ejecución de binarios de Harbour independientes

Los archivos HRB (Harbour Binary) permiten compilar código Harbour en módulos ejecutables independientes que pueden cargarse dinámicamente.

### Creación de archivos HRB

```bash
# Compilar a HRB
harbour -gh mi_modulo.prg

# O usando hbmk2
hbmk2 -hbhrb mi_modulo.prg
```

### Carga dinámica de HRB

```harbour
FUNCTION CargarModuloDinamico()
   LOCAL pHRB
   LOCAL resultado
   
   // Cargar archivo HRB
   pHRB := hb_hrbLoad("mi_modulo.hrb")
   
   IF !Empty(pHRB)
      // Ejecutar función del módulo cargado
      resultado := hb_hrbCall(pHRB, "FUNCION_DEL_MODULO", "parametro1", 123)
      
      // Liberar recursos
      hb_hrbUnload(pHRB)
   ENDIF
   
   RETURN resultado
```

### Módulo HRB de ejemplo

```harbour
// mi_modulo.prg (para compilar a HRB)

FUNCTION FUNCION_DEL_MODULO(cParam, nParam)
   LOCAL cResultado
   
   cResultado := "Procesado: " + cParam + " con número: " + LTrim(Str(nParam))
   
   RETURN cResultado

FUNCTION OTRA_FUNCION_MODULO()
   RETURN "Función adicional del módulo HRB"

// Las funciones STATIC no son accesibles desde fuera del HRB
STATIC FUNCTION FuncionPrivada()
   RETURN "Esta función no es accesible externamente"
```

### Ventajas de los archivos HRB

1. **Modularidad**: Separación lógica del código
2. **Carga bajo demanda**: Los módulos se cargan solo cuando se necesitan
3. **Actualizaciones independientes**: Los módulos pueden actualizarse sin recompilar toda la aplicación
4. **Distribución**: Facilita la distribución de extensiones y plugins

### API completa para HRB

```harbour
// Funciones principales para manejo de HRB
pHRB := hb_hrbLoad(cFileName)           // Cargar archivo HRB
resultado := hb_hrbCall(pHRB, cFunc, ...)  // Llamar función del HRB
hb_hrbUnload(pHRB)                      // Liberar HRB de memoria

// Funciones de información
aFunctions := hb_hrbGetFunList(pHRB)    // Lista de funciones disponibles
lExists := hb_hrbGetFunSym(pHRB, cFunc) // Verificar si existe una función
```

---

## Integración avanzada con bibliotecas externas

### Wrapping de bibliotecas C

Para integrar bibliotecas C existentes:

```c
// Wrapper para función de librería externa
#include "external_lib.h"

HB_FUNC( USAR_LIBRERIA_EXTERNA )
{
   const char* szInput = hb_parc(1);
   char szBuffer[256];
   
   // Llamar función de librería externa
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

### Manejo de estructuras complejas

```c
// Definir estructura
typedef struct
{
   char szNombre[50];
   int nEdad;
   double dSalario;
} PERSONA;

HB_FUNC( CREAR_PERSONA )
{
   PERSONA* pPersona = (PERSONA*)hb_xgrab(sizeof(PERSONA));
   
   hb_strncpy(pPersona->szNombre, hb_parc(1), sizeof(pPersona->szNombre) - 1);
   pPersona->nEdad = hb_parni(2);
   pPersona->dSalario = hb_parnd(3);
   
   // Retornar como puntero
   hb_retptr(pPersona);
}

HB_FUNC( LIBERAR_PERSONA )
{
   PERSONA* pPersona = (PERSONA*)hb_parptr(1);
   if( pPersona )
   {
      hb_xfree(pPersona);
   }
}
```

El sistema extendido de Harbour proporciona una base sólida para crear aplicaciones híbridas que combinan la flexibilidad de Harbour con el rendimiento de C, manteniendo siempre la seguridad de tipos y la gestión automática de memoria.