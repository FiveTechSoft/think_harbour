# Capítulo 7: Clases y objetos

La Programación Orientada a Objetos (OOP) es un paradigma de programación que utiliza "objetos" para diseñar aplicaciones y programas de software. Harbour, aunque es un lenguaje con raíces en xBase, ha incorporado un potente sistema de OOP que es compatible con la sintaxis de Clipper Class(y).

Este capítulo te guiará a través de los conceptos fundamentales de la OOP y cómo aplicarlos en Harbour.

## Conceptos de OOP: clases, objetos, variables y métodos

Antes de sumergirnos en la sintaxis de Harbour, es crucial entender los pilares de la OOP:

*   **Clase (Class):** Una clase es una plantilla o un plano para crear objetos. Define un conjunto de atributos (variables) y comportamientos (métodos) que caracterizarán a los objetos creados a partir de ella. Por ejemplo, una clase `Coche` podría definir que todos los coches tienen una `marca`, un `modelo` y pueden `acelerar()` y `frenar()`.

*   **Objeto (Object):** Un objeto es una instancia de una clase. Si `Coche` es la clase, un objeto podría ser *miCoche* específico, con `marca = "Ford"` y `modelo = "Mustang"`. Puedes crear muchos objetos a partir de una misma clase, cada uno con sus propios valores de atributos.

*   **Variable de Instancia (o Atributo):** Son variables que pertenecen a un objeto específico. Cada instancia de una clase tiene su propia copia de estas variables. En Harbour, se declaran con la palabra clave `VAR`. A menudo se les llama "miembros de datos".

*   **Método (Method):** Son funciones que pertenecen a una clase y definen los comportamientos de sus objetos. Los métodos operan sobre los datos (variables de instancia) del objeto. Por ejemplo, un método `acelerar()` en la clase `Coche` podría modificar la variable `velocidadActual`. En Harbour, se declaran con `METHOD`.

## Creación de una clase en Harbour

En Harbour, las clases se definen usando la sintaxis `CREATE CLASS ... ENDCLASS`. La estructura básica es la siguiente:

```harbour
CREATE CLASS MiClase

   // Variables de instancia (atributos)
   VAR sNombre
   VAR nEdad

   // Métodos
   METHOD New( cNombre, nEdadParam ) CONSTRUCTOR
   METHOD Saludar()
   METHOD CumplirAnios()

ENDCLASS

/*
 * Implementación de los métodos
 */

// Constructor: se llama al crear un objeto.
METHOD New( cNombre, nEdadParam ) CLASS MiClase
   ::sNombre := cNombre
   ::nEdad   := nEdadParam
   RETURN Self // El constructor debe devolver Self

// Método para mostrar un saludo
METHOD Saludar() CLASS MiClase
   ? "Hola, mi nombre es", ::sNombre
   RETURN Nil

// Método para incrementar la edad
METHOD CumplirAnios() CLASS MiClase
   ::nEdad++
   ? "¡Feliz cumpleaños! Ahora tengo", ::nEdad, "años."
   RETURN Nil
```

**Puntos clave:**

*   `CREATE CLASS NombreClase ... ENDCLASS`: Define el inicio y el fin de la declaración de la clase.
*   `VAR nombreVariable`: Declara una variable de instancia. Por convención, se suele usar un prefijo como `s` para string, `n` para numérico, etc.
*   `METHOD nombreMetodo(...)`: Declara un método.
*   `CONSTRUCTOR`: Marca un método como el constructor de la clase. Este método se ejecuta automáticamente al crear una nueva instancia del objeto.
*   `METHOD ... CLASS NombreClase`: Implementación real del método. Se hace fuera del bloque `CREATE CLASS`.
*   `::`: Es el operador de ámbito de instancia. Se usa para acceder a las variables y métodos del objeto actual (similar a `this` en otros lenguajes).
*   `Self`: Es una referencia al propio objeto, que debe ser devuelta por el constructor.

## Instanciación de objetos

Una vez que tienes una clase, puedes crear objetos (instancias) a partir de ella. Esto se hace usando la sintaxis `NombreClase():New(...)` o `NombreClaseNew(...)`.

```harbour
PROCEDURE Main()

   // Crear una instancia de la clase MiClase
   LOCAL oPersona1 := MiClase():New( "Juan Perez", 30 )

   // Crear otra instancia
   LOCAL oPersona2 := MiClase():New( "Ana López", 25 )

   // Usar los métodos de los objetos
   oPersona1:Saludar()       // Salida: Hola, mi nombre es Juan Perez
   oPersona2:Saludar()       // Salida: Hola, mi nombre es Ana López

   oPersona1:CumplirAnios()  // Salida: ¡Feliz cumpleaños! Ahora tengo 31 años.

   // Acceder a una variable de instancia
   ? "Edad de Ana:", oPersona2:nEdad  // Salida: Edad de Ana: 25

   WAIT // Espera que el usuario presione una tecla

RETURN
```

En este ejemplo:

*   `MiClase():New(...)` crea un nuevo objeto. Los argumentos pasados a `New()` son los que recibe el método constructor.
*   `oPersona1` y `oPersona2` son dos objetos independientes, cada uno con sus propios datos (`sNombre` y `nEdad`).
*   `:` es el operador de envío de mensajes. Se usa para llamar a un método o acceder a una variable de un objeto (`oPersona1:Saludar()`).

## Herencia

La herencia es un mecanismo que permite a una clase (llamada clase hija o subclase) heredar atributos y métodos de otra clase (llamada clase padre o superclase). Esto fomenta la reutilización de código.

En Harbour, la herencia se especifica con la cláusula `FROM` o `INHERIT` en la declaración de la clase.

```harbour
// Clase Padre
CREATE CLASS Vehiculo
   VAR nRuedas
   VAR cColor

   METHOD New( nRuedas, cColor ) CONSTRUCTOR
   METHOD Describir()
ENDCLASS

METHOD New( nRuedas, cColor ) CLASS Vehiculo
   ::nRuedas := nRuedas
   ::cColor  := cColor
   RETURN Self

METHOD Describir() CLASS Vehiculo
   ? "Este es un vehículo de", ::nRuedas, "ruedas y color", ::cColor
   RETURN Nil

// Clase Hija que hereda de Vehiculo
CREATE CLASS Coche FROM Vehiculo
   VAR sMarca

   METHOD New( cColor, sMarca ) CONSTRUCTOR
   METHOD Describir() // Sobrescribiendo el método del padre
ENDCLASS

METHOD New( cColor, sMarca ) CLASS Coche
   // Llamar al constructor del padre usando Super:
   Super:New( 4, cColor )
   ::sMarca := sMarca
   RETURN Self

METHOD Describir() CLASS Coche
   // Se puede llamar al método original del padre
   Super:Describir()
   ? "   -> Es un coche de la marca:", ::sMarca
   RETURN Nil

/*
 * Uso de las clases con herencia
 */
PROCEDURE Main()
   LOCAL oMiCoche := Coche():New( "Rojo", "Ferrari" )

   oMiCoche:Describir()

   WAIT
RETURN
```

**Salida del programa:**
```
Este es un vehículo de 4 ruedas y color Rojo
   -> Es un coche de la marca: Ferrari
```

**Puntos clave de la herencia:**

*   `CREATE CLASS Coche FROM Vehiculo`: La clase `Coche` hereda de `Vehiculo`.
*   La clase `Coche` tiene acceso a `nRuedas` y `cColor` de `Vehiculo`, además de su propio atributo `sMarca`.
*   **Sobrescritura de métodos (Overriding):** La clase `Coche` redefine el método `Describir()`. Cuando se llama a `oMiCoche:Describir()`, se ejecuta la versión de `Coche`.
*   `Super:`: Es una palabra clave que permite llamar a métodos de la clase padre desde la clase hija. `Super:New(...)` invoca el constructor de `Vehiculo`, y `Super:Describir()` invoca el método `Describir()` original del padre. Esto es muy útil para extender la funcionalidad en lugar de reemplazarla por completo.

Este capítulo proporciona una base sólida para empezar a trabajar con la programación orientada a objetos en Harbour. Dominar estos conceptos te permitirá crear código más modular, reutilizable y fácil de mantener.
