# Capítulo 1: El camino del programa

Este capítulo introduce los conceptos fundamentales de la programación en Harbour, desde la definición de un programa hasta la creación de tu primera aplicación ejecutable.

---

### ¿Qué es un programa?

Un **programa** es una secuencia de instrucciones escritas en un lenguaje que el ordenador puede entender y ejecutar. En el contexto de Harbour, un programa es un archivo de texto plano, generalmente con la extensión `.prg`, que contiene código fuente.

El código fuente es legible por humanos y sigue las reglas sintácticas del lenguaje Harbour. Sin embargo, un ordenador no puede ejecutar directamente este código. Primero, debe ser traducido a un formato que el procesador pueda comprender. Este proceso se llama **compilación**.

El compilador de Harbour lee tu archivo `.prg` y lo convierte en un archivo **ejecutable** (por ejemplo, un `.exe` en Windows). Cuando ejecutas este archivo, el ordenador realiza las tareas que has programado.

---

### El lenguaje Harbour: un compilador y entorno xBase

**Harbour** es un lenguaje de programación de alto nivel, compilado y multiplataforma. Es conocido por ser un sucesor moderno y de código abierto del popular lenguaje **Clipper**, lo que lo enmarca dentro de la familia de lenguajes **xBase** (derivados de dBase).

*   **Compilador**: Harbour no es un intérprete. Toma tu código fuente y lo traduce a código C, que luego se compila en un binario nativo. Este proceso garantiza un rendimiento muy alto y la capacidad de crear aplicaciones independientes.
*   **Entorno xBase**: Harbour hereda la sintaxis y la filosofía de los lenguajes xBase, que están especialmente diseñados para aplicaciones de gestión de datos y negocios. Incluye un robusto sistema de bases de datos (RDDs) y funciones para manejar tablas (`.dbf`), índices y manipulación de registros de forma nativa.
*   **Características modernas**: Aunque su raíz es xBase, Harbour ha evolucionado para incluir características modernas como:
    *   Programación Orientada a Objetos (OOP).
    *   Soporte multiplataforma (Windows, Linux, macOS, etc.).
    *   Multihilo para tareas concurrentes.
    *   Extensibilidad a través de librerías en C/C++.
    *   Una gran colección de librerías `contrib` para GUI, conectividad web, etc.

---

### El preprocesador de Harbour: macros (`#define`), inclusión de archivos (`#include`), y comandos definidos por el usuario (`#command`, `#xcommand`)

El **preprocesador** es una herramienta que se ejecuta antes de la compilación. Su función es modificar el código fuente en función de directivas específicas.

*   **`#include`**: Inserta el contenido de otro archivo en la ubicación actual. Se usa comúnmente para incluir archivos de cabecera (`.h` o `.ch`) que contienen constantes, declaraciones de funciones o comandos.
    ```harbour
    // Incluye las constantes y comandos estándar de Harbour
    #include "hbclass.ch"
    ```

*   **`#define`**: Crea macros de sustitución. El preprocesador reemplazará todas las instancias del nombre de la macro con el valor definido.
    ```harbour
    #define MAX_ITEMS 100
    #define SALUDO "¡Hola!"

    // En el código, el preprocesador reemplazará MAX_ITEMS por 100
    aDatos := ARRAY(MAX_ITEMS)
    ```

*   **`#command` y `#xcommand`**: Permiten crear nuevos comandos en el lenguaje, simplificando la sintaxis y creando abstracciones de alto nivel. `#command` utiliza un sistema de coincidencias de patrones simple, mientras que `#xcommand` ofrece un motor de expresiones regulares más potente.
    ```harbour
    // Define un nuevo comando para mostrar un mensaje en una posición
    #command MSG <row>, <col> PROMPT <message> => QOut( <row>, <col>, <message> )

    // Uso del nuevo comando
    MSG 10, 5 PROMPT "Esto es un mensaje de prueba"
    ```

---

### Tu primer programa: "¡Hola, mundo!"

El "¡Hola, mundo!" es el programa más simple que se puede escribir. Demuestra que tu entorno de compilación está configurado correctamente.

1.  Crea un archivo llamado `hola.prg`.
2.  Escribe el siguiente código en él:

    ```harbour
    /*
     * Mi primer programa en Harbour
     * Muestra "¡Hola, mundo!" en la consola.
     */
    FUNCTION Main()

       // Limpia la pantalla
       CLS

       // Muestra el texto en la fila 10, columna 5
       @ 10, 5 SAY "¡Hola, mundo!"

       // Espera a que el usuario presione una tecla
       WAIT

    RETURN NIL
    ```

3.  **Compilación**: Para compilar este programa, necesitarás el compilador de Harbour y `hbmk2`, su herramienta de construcción. Abre una terminal o línea de comandos y ejecuta:
    ```bash
    hbmk2 hola.prg
    ```
    `hbmk2` se encargará de compilar y enlazar todo lo necesario, generando un archivo `hola.exe` (en Windows) o `hola` (en Linux/macOS).

4.  **Ejecución**: Ejecuta el programa desde la terminal:
    ```bash
    ./hola
    ```
    Verás el mensaje "¡Hola, mundo!" en la pantalla.

---

### Resolución de problemas y depuración

Cuando un programa no funciona como se espera, es hora de depurar.

*   **Errores de compilación**: Ocurren si el código viola las reglas sintácticas del lenguaje. El compilador de Harbour te informará del error y el número de línea para que puedas corregirlo.

*   **Errores en tiempo de ejecución**: Suceden mientras el programa se está ejecutando. Por ejemplo, intentar dividir por cero o acceder a un array fuera de sus límites. Harbour tiene un sistema de manejo de errores (`BEGIN SEQUENCE...END`) que te permite capturar estos fallos y manejarlos de forma controlada.

*   **Errores lógicos**: Son los más difíciles de encontrar. El programa se ejecuta sin fallar, pero el resultado no es el esperado. Las herramientas para solucionar estos errores incluyen:
    *   **Visualización de variables**: Usa `QOut()` o `Alert()` para mostrar el contenido de una variable en un punto específico del programa.
    *   **El depurador de Harbour**: Permite ejecutar el programa paso a paso, inspeccionar variables en tiempo real y analizar el flujo de ejecución. Para compilar con soporte de depuración, usa la opción `-b` con `hbmk2`:
      ```bash
      hbmk2 -b mi_programa.prg
      ```
    *   **Mensajes de registro (`logging`)**: Escribir información relevante en un archivo de texto (`.log`) puede ayudarte a rastrear lo que hizo el programa, especialmente en procesos largos o desatendidos.
