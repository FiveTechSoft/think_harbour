# Capítulo 5: Tipos de datos complejos

En Harbour, además de los tipos de datos simples como numéricos, cadenas y lógicos, existen tipos de datos complejos que permiten estructurar y manejar la información de manera más eficiente. Los dos más importantes son los **Arrays** y los **Hashes**.

## Arrays

Un array (o arreglo) es una colección ordenada de elementos. Cada elemento en el array tiene una posición o índice numérico, que en Harbour comienza en 1. Los arrays pueden contener elementos de cualquier tipo de dato, ¡incluso otros arrays o hashes!

### Creación de Arrays

Puedes crear un array de varias formas:

```harbour
// Un array vacío
LOCAL aMiArray1 := {}

// Un array con elementos iniciales
LOCAL aMiArray2 := { "Manzana", "Banana", "Naranja" }

// Usando la función Array() para crear un array de un tamaño específico
LOCAL aMiArray3 := Array(10) // Crea un array con 10 elementos NIL
```

### Acceso y Modificación

Se accede a los elementos a través de su índice numérico entre corchetes `[]`.

```harbour
LOCAL aFrutas := { "Manzana", "Banana", "Naranja" }

? aFrutas[1] // Muestra "Manzana"
? aFrutas[3] // Muestra "Naranja"

// Modificar un elemento
aFrutas[2] := "Fresa"
? aFrutas[2] // Muestra "Fresa"
```

### Funciones comunes para Arrays

- **`len( <array> )`**: Devuelve el número de elementos en el array.
- **`aadd( <array>, <elemento> )`**: Añade un nuevo elemento al final del array.
- **`asize( <array>, <nuevo_tamaño> )`**: Cambia el tamaño del array. Si es más grande, añade elementos NIL. Si es más pequeño, los trunca.
- **`ascan( <array>, <valor_o_bloque_de_código> )`**: Busca un elemento y devuelve su índice.
- **`adel( <array>, <posición> )`**: Elimina un elemento en una posición específica.
- **`ains( <array>, <posición> )`**: Inserta un elemento NIL en una posición específica.

## Hashes

Un hash (también conocido como diccionario o mapa en otros lenguajes) es una colección de pares clave-valor. En lugar de usar un índice numérico, se utiliza una "clave" (generalmente una cadena de texto) para acceder a su "valor" asociado. Los hashes son extremadamente útiles para almacenar datos estructurados.

### Creación de Hashes

```harbour
// Un hash vacío
LOCAL hMiHash1 := {=>}

// Un hash con pares clave-valor iniciales
LOCAL hPersona := { "nombre" => "Juan", "edad" => 30, "ciudad" => "Madrid" }

// Usando la función HSet()
LOCAL hMiHash2 := {=>}
HSet( hMiHash2, "clave1", "valor1" )
```

### Acceso y Modificación

Se accede a los valores a través de su clave, usando la sintaxis `hMiHash["clave"]`.

```harbour
LOCAL hPersona := { "nombre" => "Juan", "edad" => 30 }

? hPersona["nombre"] // Muestra "Juan"

// Modificar un valor
hPersona["edad"] := 31

// Añadir un nuevo par clave-valor
hPersona["profesion"] := "Programador"

? hPersona // Muestra el contenido del hash
```

### Funciones y Bucles para Hashes

- **`haskey( <hash>, <clave> )`**: Comprueba si una clave existe en el hash. Devuelve `.T.` o `.F.`.
- **`hdel( <hash>, <clave> )`**: Elimina un par clave-valor del hash.
- **Iterar con `FOR EACH`**: La forma más común de recorrer todos los elementos de un hash.

```harbour
LOCAL hPersona := { "nombre" => "Ana", "edad" => 25, "ciudad" => "Barcelona" }
LOCAL xKey, xValue

FOR EACH xKey, xValue IN hPersona
   ? xKey + ":", xValue
NEXT
```

## Diferencias y Cuándo Usar Cada Uno

| Característica | Arrays | Hashes |
| :--- | :--- | :--- |
| **Acceso** | Por índice numérico (1, 2, 3...). | Por clave (ej: "nombre"). |
| **Orden** | **Ordenados**. Mantienen el orden de inserción. | **Generalmente desordenados**. No se debe confiar en el orden. |
| **Uso Típico** | Listas de elementos donde el orden importa o se necesita acceso secuencial. | Datos estructurados donde cada valor tiene un nombre o identificador. |
| **Velocidad** | Acceso por índice es muy rápido. | Acceso por clave es muy rápido y eficiente. |

### ¿Cuándo usar un Array?

- Para una lista de elementos del mismo tipo (ej: una lista de nombres de clientes).
- Cuando el orden de los elementos es importante (ej: los pasos en una secuencia).
- Para representar filas de una tabla leídas de una base de datos.

**Ejemplo:** Una lista de la compra.
```harbour
aCompra := { "Leche", "Pan", "Huevos" }
```

### ¿Cuándo usar un Hash?

- Para representar un objeto único con sus propiedades (ej: un cliente con su nombre, ID, y dirección).
- Para pasar un conjunto de parámetros con nombre a una función.
- Para almacenar configuraciones o ajustes de la aplicación.

**Ejemplo:** Los datos de un usuario.
```harbour
hUsuario := { "id" => 101, "usuario" => "admin", "nivel_acceso" => 5 }
```
