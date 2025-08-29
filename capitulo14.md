# Capítulo 14: El sistema de datos de Harbour (`The Harbour item`)

El sistema de datos de Harbour está fundamentado en una estructura llamada `item` que actúa como el contenedor universal para todos los tipos de datos del lenguaje. Este sistema flexible y potente es el corazón de la máquina virtual de Harbour, permitiendo el manejo dinámico de tipos y la interoperabilidad entre el código Harbour y las extensiones en C.

---

## El `item`: el contenedor de datos universal de Harbour

En Harbour, todos los valores - desde números simples hasta objetos complejos - se almacenan internamente en una estructura de datos denominada `HB_ITEM`. Este diseño unificado permite que Harbour sea un lenguaje de tipado dinámico mientras mantiene un rendimiento óptimo.

### Estructura interna del `HB_ITEM`

El `item` es una estructura C que contiene:

```c
typedef struct _HB_ITEM
{
   HB_TYPE type;     // Tipo de dato (HB_IT_STRING, HB_IT_NUMERIC, etc.)
   HB_ITEM_UNION;    // Unión que contiene el valor actual
} HB_ITEM, * PHB_ITEM;
```

### Tipos de datos soportados

Los principales tipos de datos que puede contener un `item` son:

- **HB_IT_NIL**: Valor nulo
- **HB_IT_LOGICAL**: Valores lógicos (.T. o .F.)
- **HB_IT_INTEGER**: Números enteros
- **HB_IT_LONG**: Enteros largos
- **HB_IT_DOUBLE**: Números de punto flotante
- **HB_IT_DATE**: Fechas
- **HB_IT_TIMESTAMP**: Marcas de tiempo
- **HB_IT_STRING**: Cadenas de caracteres
- **HB_IT_ARRAY**: Arrays (matrices)
- **HB_IT_HASH**: Tablas hash
- **HB_IT_BLOCK**: Bloques de código
- **HB_IT_SYMBOL**: Símbolos de funciones
- **HB_IT_POINTER**: Punteros a datos externos

---

## La API del `item`: funciones para manipular valores a nivel de C

Para trabajar con `items` desde extensiones en C, Harbour proporciona una API completa de funciones. Estas funciones siguen un patrón consistente para crear, acceder y modificar valores.

### Funciones de creación y asignación

```c
// Crear un nuevo item
PHB_ITEM hb_itemNew( PHB_ITEM pItem );

// Asignar valores específicos
PHB_ITEM hb_itemPutNI( PHB_ITEM pItem, int iNumber );
PHB_ITEM hb_itemPutND( PHB_ITEM pItem, double dNumber );
PHB_ITEM hb_itemPutC( PHB_ITEM pItem, const char * szText );
PHB_ITEM hb_itemPutL( PHB_ITEM pItem, HB_BOOL fValue );
```

### Funciones de consulta de tipo

```c
// Verificar el tipo de un item
HB_BOOL hb_itemIsString( PHB_ITEM pItem );
HB_BOOL hb_itemIsNumeric( PHB_ITEM pItem );
HB_BOOL hb_itemIsLogical( PHB_ITEM pItem );
HB_BOOL hb_itemIsArray( PHB_ITEM pItem );
```

### Funciones de acceso a valores

```c
// Obtener valores desde un item
const char * hb_itemGetCPtr( PHB_ITEM pItem );
double hb_itemGetND( PHB_ITEM pItem );
HB_BOOL hb_itemGetL( PHB_ITEM pItem );
```

---

## Valores escalares: manejo de tipos de datos básicos

Los tipos escalares son aquellos que contienen un único valor simple. Harbour optimiza el manejo de estos tipos para maximizar el rendimiento.

### Números

Harbour maneja automáticamente la conversión entre diferentes tipos numéricos:

```harbour
LOCAL nEntero := 42        // HB_IT_INTEGER
LOCAL nDecimal := 3.14159  // HB_IT_DOUBLE
LOCAL nCalculo := nEntero * nDecimal  // Conversión automática
```

### Cadenas de caracteres

Las cadenas en Harbour son inmutables y utilizan conteo de referencias para optimizar la memoria:

```harbour
LOCAL cTexto := "Hola Harbour"
LOCAL cCopia := cTexto  // No se duplica la cadena, solo la referencia
```

### Fechas y marcas de tiempo

```harbour
LOCAL dFecha := Date()         // HB_IT_DATE
LOCAL tAhora := hb_DateTime()  // HB_IT_TIMESTAMP
```

---

## Manejo de `Arrays` y `Hashes`: estructura y operaciones de colecciones complejas

Los arrays y hashes son tipos de datos complejos que pueden contener múltiples `items`.

### Arrays

Los arrays en Harbour son dinámicos y pueden contener elementos de cualquier tipo:

```harbour
LOCAL aLista := Array(5)        // Crear array de 5 elementos
LOCAL aMixto := {"Texto", 123, Date(), .T.}  // Array mixto
```

**Operaciones fundamentales:**

```harbour
// Acceso a elementos
? aLista[1]           // Primer elemento (base 1)

// Modificación
aLista[2] := "Nuevo valor"

// Redimensionamiento
ASize(aLista, 10)     // Cambiar tamaño a 10 elementos

// Inserción y eliminación
AAdd(aLista, "Nuevo elemento")   // Agregar al final
AIns(aLista, 2)                  // Insertar en posición 2
ADel(aLista, 3)                  // Eliminar elemento 3
```

### Hashes

Los hashes proporcionan acceso asociativo mediante claves:

```harbour
LOCAL hDatos := {=>}  // Hash vacío
LOCAL hPersona := {"nombre" => "Juan", "edad" => 30}

// Acceso por clave
? hPersona["nombre"]

// Agregar nuevas claves
hPersona["ciudad"] := "Madrid"

// Iterar sobre claves
FOR EACH cClave IN hb_HKeys(hPersona)
   ? cClave, "=>", hPersona[cClave]
NEXT
```

---

## `Codeblocks`: el tipo de dato para código ejecutable

Los bloques de código son una característica única de Harbour que permite almacenar código ejecutable como un valor de primera clase.

### Creación y uso

```harbour
LOCAL bBloque := {|x, y| x + y}  // Bloque que suma dos parámetros

// Evaluación del bloque
LOCAL nResultado := Eval(bBloque, 5, 3)  // Resultado: 8
```

### Bloques con variables locales

```harbour
LOCAL nMultiplicador := 10
LOCAL bEscalar := {|x| x * nMultiplicador}  // Captura la variable local

? Eval(bEscalar, 5)  // Resultado: 50
```

### Aplicaciones prácticas

Los bloques de código son especialmente útiles para:

- **Callbacks**: Funciones que se pasan como parámetros
- **Filtros y transformaciones**: Operaciones en arrays
- **Evaluación diferida**: Código que se ejecuta bajo ciertas condiciones

```harbour
// Filtrar un array con un bloque
LOCAL aNumeros := {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
LOCAL aPares := AEval(aNumeros, {|x| x % 2 == 0})
```

---

## `Classes and Objects`: la implementación de la programación orientada a objetos

En Harbour, las clases y objetos se implementan sobre el sistema de `items`, proporcionando un modelo de POO flexible y potente.

### Estructura interna de una clase

```harbour
// Definición de clase
CREATE CLASS Persona
   VAR cNombre
   VAR nEdad
   
   METHOD New(cNom, nEd) CONSTRUCTOR
   METHOD Saludar()
   METHOD SetNombre(cNuevoNombre)
ENDCLASS

// Implementación de métodos
METHOD New(cNom, nEd) CLASS Persona
   ::cNombre := cNom
   ::nEdad := nEd
   RETURN Self

METHOD Saludar() CLASS Persona
   RETURN "Hola, soy " + ::cNombre
```

### Herencia y polimorfismo

```harbour
CREATE CLASS Empleado FROM Persona
   VAR cEmpresa
   VAR nSalario
   
   METHOD New(cNom, nEd, cEmp, nSal) CONSTRUCTOR
   METHOD Trabajar()
   METHOD Saludar()  // Redefinición del método padre
ENDCLASS

METHOD Saludar() CLASS Empleado
   RETURN ::Super:Saludar() + " y trabajo en " + ::cEmpresa
```

---

## Punteros y símbolos: referencias a variables y funciones

### Símbolos

Los símbolos representan referencias a funciones y variables en el espacio de nombres global:

```harbour
LOCAL sFunc := @MiFuncion()  // Símbolo que apunta a MiFuncion
LOCAL resultado := Eval(sFunc, parametros)  // Llamada a través del símbolo
```

### Punteros

Los punteros permiten referencias a datos externos o estructuras C:

```harbour
// Desde una extensión C
FUNCTION ObtenerPuntero()
   LOCAL pDatos := hb_PointerFromString("datos externos")
   RETURN pDatos

// Uso del puntero
LOCAL pMiPuntero := ObtenerPuntero()
? hb_PointerToString(pMiPuntero)
```

---

## Optimizaciones del sistema de `items`

### Conteo de referencias

Harbour utiliza conteo de referencias para optimizar el uso de memoria, especialmente con cadenas y arrays:

```harbour
LOCAL cTexto1 := "Cadena larga con mucho contenido"
LOCAL cTexto2 := cTexto1  // Solo se incrementa el contador de referencias
```

### Pool de objetos

Para tipos frecuentemente utilizados como números pequeños y valores lógicos, Harbour mantiene un pool de objetos reutilizables.

### Conversiones automáticas

El sistema de `items` permite conversiones automáticas entre tipos compatibles:

```harbour
LOCAL cNumero := "123"
LOCAL nResultado := cNumero + 45  // Conversión automática: "123" -> 123
```

Este diseño unificado del sistema de datos hace que Harbour sea tanto flexible como eficiente, permitiendo el desarrollo de aplicaciones robustas con un rendimiento excepcional.