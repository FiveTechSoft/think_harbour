# Capítulo 14: El sistema de datos de Harbour (`The Harbour item`)

Este capítulo explora el sistema de datos interno de Harbour, centrado en el concepto del "item" como contenedor universal de datos y su API para manipulación a nivel interno.

---

## El `item`: el contenedor de datos universal de Harbour

En Harbour, todos los datos se almacenan internamente en una estructura llamada "item" (elemento). Esta estructura unificada permite que Harbour maneje diferentes tipos de datos de manera consistente.

### Estructura del item

El item es una estructura en C que contiene:

```c
typedef struct _HB_ITEM
{
   HB_TYPE  type;        // Tipo del dato
   union
   {
      HB_LONG     asLong;     // Valor numérico entero
      double      asDouble;   // Valor numérico decimal
      struct                  // Cadena de caracteres
      {
         char *   value;
         HB_SIZE  length;
      } asString;
      // ... otros tipos
   } item;
} HB_ITEM;
```

### Ventajas del sistema de items

- **Unificación**: Un solo tipo de dato maneja todos los valores
- **Eficiencia**: Optimizaciones específicas para cada tipo
- **Flexibilidad**: Conversiones automáticas entre tipos cuando es necesario
- **Gestión de memoria**: Control centralizado de asignación y liberación

---

## La API del `item`: funciones para manipular valores a nivel de C

La API del item proporciona funciones para crear, manipular y destruir items desde código C.

### Funciones básicas de creación

```c
// Crear items vacíos
PHB_ITEM pItem = hb_itemNew( NULL );

// Crear items con valores específicos
PHB_ITEM pNumero = hb_itemPutNI( NULL, 42 );
PHB_ITEM pCadena = hb_itemPutC( NULL, "Hola Mundo" );
PHB_ITEM pLogico = hb_itemPutL( NULL, HB_TRUE );
```

### Funciones de consulta

```c
// Verificar tipos
if( hb_itemIsNumeric( pItem ) )
{
   // Es un número
}

if( hb_itemIsString( pItem ) )
{
   // Es una cadena
}

// Obtener valores
HB_LONG nValor = hb_itemGetNL( pItem );
const char* szTexto = hb_itemGetCPtr( pItem );
HB_BOOL bLogico = hb_itemGetL( pItem );
```

### Funciones de liberación

```c
// Liberar memoria del item
hb_itemRelease( pItem );

// Limpiar contenido sin liberar el item
hb_itemClear( pItem );
```

---

## Valores escalares: manejo de tipos de datos básicos

Los valores escalares son los tipos de datos fundamentales en Harbour.

### Tipos numéricos

#### Enteros (Integer)

```harbour
LOCAL nEntero := 42
// Internamente se almacena como HB_IT_INTEGER
```

```c
// Manipulación en C
PHB_ITEM pEntero = hb_itemPutNI( NULL, 42 );
HB_LONG nValor = hb_itemGetNL( pEntero );
```

#### Decimales (Double)

```harbour
LOCAL nDecimal := 3.14159
// Internamente se almacena como HB_IT_DOUBLE
```

```c
// Manipulación en C
PHB_ITEM pDecimal = hb_itemPutND( NULL, 3.14159 );
double dValor = hb_itemGetND( pDecimal );
```

### Tipos de cadena

```harbour
LOCAL cCadena := "Hola Mundo"
// Internamente se almacena como HB_IT_STRING
```

```c
// Manipulación en C
PHB_ITEM pCadena = hb_itemPutC( NULL, "Hola Mundo" );
const char* szValor = hb_itemGetCPtr( pCadena );
HB_SIZE nLongitud = hb_itemGetCLen( pCadena );
```

### Tipos lógicos

```harbour
LOCAL lVerdadero := .T.
LOCAL lFalso := .F.
// Internamente se almacena como HB_IT_LOGICAL
```

### Fechas

```harbour
LOCAL dFecha := Date()
// Internamente se almacena como HB_IT_DATE
```

---

## Manejo de `Arrays` y `Hashes`: estructura y operaciones de colecciones complejas

Los arrays y hashes son tipos de datos complejos que contienen múltiples items.

### Estructura interna de Arrays

```c
// Estructura simplificada de un array
typedef struct _HB_ARRAY
{
   HB_SIZE     nLen;        // Número de elementos
   PHB_ITEM *  pItems;      // Array de punteros a items
   // ... otros campos
} HB_ARRAY;
```

### Operaciones con Arrays

```c
// Crear array
PHB_ITEM pArray = hb_itemArrayNew( 10 );

// Establecer valores
hb_arraySetNI( pArray, 1, 42 );
hb_arraySetC( pArray, 2, "texto" );

// Obtener valores
HB_LONG nValor = hb_arrayGetNL( pArray, 1 );
const char* szTexto = hb_arrayGetCPtr( pArray, 2 );

// Tamaño del array
HB_SIZE nLen = hb_arrayLen( pArray );
```

### Estructura interna de Hashes

```c
// Los hashes utilizan una tabla hash interna
typedef struct _HB_HASH
{
   HB_SIZE     nSize;       // Número de elementos
   HB_SIZE     nCapacity;   // Capacidad total
   // Tabla hash con buckets para claves y valores
} HB_HASH;
```

### Operaciones con Hashes

```c
// Crear hash
PHB_ITEM pHash = hb_hashNew( NULL );

// Establecer valores
hb_hashAddChar( pHash, "clave1", "valor1" );
hb_hashAddInt( pHash, "clave2", 42 );

// Obtener valores
PHB_ITEM pValor = hb_hashGetItemPtr( pHash, hb_itemPutC( NULL, "clave1" ), HB_HASH_AUTOADD_ACCESS );
```

---

## `Codeblocks`: el tipo de dato para código ejecutable

Los codeblocks son fragmentos de código que pueden ser almacenados y ejecutados dinámicamente.

### Estructura interna

```c
typedef struct _HB_CODEBLOCK
{
   const HB_BYTE * pCode;     // P-Code del codeblock
   PHB_SYMB        pSymbols;  // Tabla de símbolos
   PHB_ITEM        pLocals;   // Variables locales detached
   // ... otros campos
} HB_CODEBLOCK;
```

### Creación y ejecución

```harbour
// Crear codeblock
LOCAL bBloque := {|| ? "Hola desde codeblock" }

// Codeblock con parámetros
LOCAL bSuma := {|a, b| a + b }

// Ejecutar
Eval( bBloque )
LOCAL nResultado := Eval( bSuma, 5, 3 )  // Resultado: 8
```

---

## `Classes and Objects`: la implementación de la programación orientada a objetos

Las clases y objetos en Harbour se implementan sobre el sistema de items.

### Estructura de objetos

```c
typedef struct _HB_OBJECT
{
   PHB_SYMB    pClass;       // Puntero a la clase
   PHB_ITEM *  pItems;       // Array de variables de instancia
   HB_SIZE     nVars;        // Número de variables
} HB_OBJECT;
```

### Acceso a miembros

```harbour
// Definir clase
CREATE CLASS MiClase
   VAR nNumero
   VAR cTexto
   METHOD Nuevo()
ENDCLASS

// Crear objeto
LOCAL oObjeto := MiClase():Nuevo()

// Acceder a variables
oObjeto:nNumero := 42
oObjeto:cTexto := "Hola"
```

---

## Punteros y símbolos: referencias a variables y funciones

### Símbolos

Los símbolos representan nombres de funciones y variables en la tabla de símbolos global.

```c
typedef struct _HB_SYMB
{
   const char *   szName;      // Nombre del símbolo
   union
   {
      HB_PCODEFUNC pFunPtr;    // Puntero a función
      PHB_ITEM     pVarPtr;    // Puntero a variable
   } value;
   HB_SYMBOLSCOPE scope;       // Alcance del símbolo
} HB_SYMB;
```

### Punteros

Los punteros permiten referencias a datos en memoria:

```harbour
// Puntero a función
LOCAL pFunc := @MiFuncion()

// Puntero a variable
LOCAL nVar := 42
LOCAL pVar := @nVar
```

### Gestión de memoria

- **Referencias**: Control de referencias para garbage collection
- **Conteo de referencias**: Evita liberación prematura de memoria
- **Detección de ciclos**: Identifica referencias circulares

El sistema de items proporciona la base sólida sobre la cual se construyen todas las características de alto nivel de Harbour, desde tipos simples hasta estructuras de datos complejas y objetos.