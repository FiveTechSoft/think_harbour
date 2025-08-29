# think_harbour

# Capítulo 1: El camino del programa

* [¿Qué es un programa?](capitulo1.md#qué-es-un-programa)
* [El lenguaje Harbour: un compilador y entorno xBase.](capitulo1.md#el-lenguaje-harbour-un-compilador-y-entorno-xbase)
* [El preprocesador de Harbour: macros (`#define`), inclusión de archivos (`#include`), y comandos definidos por el usuario (`#command`, `#xcommand`).](capitulo1.md#el-preprocesador-de-harbour-macros-define-inclusión-de-archivos-include-y-comandos-definidos-por-el-usuario-command-xcommand)
* [Tu primer programa: "¡Hola, mundo!".](capitulo1.md#tu-primer-programa-hola-mundo)
* [Resolución de problemas y depuración.](capitulo1.md#resolución-de-problemas-y-depuración)

---

# Capítulo 2: Variables, expresiones y sentencias

* [Valores y tipos de datos en Harbour: `String`, `Number`, `Logical`, `Date`, `CodeBlock`, `Symbol` y `Pointer`.](capitulo2.md#valores-y-tipos-de-datos-en-harbour-string-number-logical-date-codeblock-symbol-y-pointer)
* [Variables y su alcance (`LOCAL`, `STATIC`, `PRIVATE`, `PUBLIC`).](capitulo2.md#variables-y-su-alcance-local-static-private-public)
* [Sentencias de asignación y operadores.](capitulo2.md#sentencias-de-asignación-y-operadores)
* [Expresiones.](capitulo2.md#expresiones)

---

# Capítulo 3: Funciones

* [Procedimientos (`PROCEDURE`) vs. Funciones (`FUNCTION`).](capitulo3.md#procedimientos-procedure-vs-funciones-function)
* [Parámetros y argumentos.](capitulo3.md#parámetros-y-argumentos)
* [Funciones de entrada/salida (E/S).](capitulo3.md#funciones-de-entradasalida-es)
* [Funciones de utilidad (`VALTYPE()`, `PROCNAME()`).](capitulo3.md#funciones-de-utilidad-valtype-procname)

---

# Capítulo 4: Estructuras de control

* [Sentencias condicionales: `IF`, `ELSEIF`, `ELSE`, `ENDIF`, `SWITCH`.](capitulo4.md#sentencias-condicionales-if-elseif-else-endif-switch)
* [Bucles: `DO WHILE`, `FOR EACH`, `FOR...NEXT`.](capitulo4.md#bucles-do-while-for-each-fornext)
* [Manejador de secuencias: `BEGIN SEQUENCE...BREAK...END`.](capitulo4.md#manejador-de-secuencias-begin-sequencebreakend)

---

# Capítulo 5: Tipos de datos complejos

* [Arrays: listas ordenadas.](capitulo5.md#arrays-listas-ordenadas)
* [Hashes: colecciones clave-valor.](capitulo5.md#hashes-colecciones-clave-valor)
* [Diferencias y cuándo usar cada uno.](capitulo5.md#diferencias-y-cuándo-usar-cada-uno)

---

# Capítulo 6: Archivos y bases de datos

* [Manejo de archivos de bajo nivel (`FCREATE`, `FOPEN`, `FCLOSE`).](capitulo6.md#manejo-de-archivos-de-bajo-nivel-fcreate-fopen-fclose)
* [El formato de archivo `.dbf`.](capitulo6.md#el-formato-de-archivo-dbf)
* [Funciones de base de datos: `USE`, `APPEND BLANK`, `REPLACE`.](capitulo6.md#funciones-de-base-de-datos-use-append-blank-replace)
* [Índices: cómo acelerar búsquedas.](capitulo6.md#índices-cómo-acelerar-búsquedas)
* [Sistemas de gestión de bases de datos relacionales (RDDs).](capitulo6.md#sistemas-de-gestión-de-bases-de-datos-relacionales-rdds)

---

[# Capítulo 7: Clases y objetos

* ### Conceptos de OOP: clases, objetos, variables y métodos.

* ### Creación de una clase en Harbour.

* ### Instanciación de objetos.

* ### Herencia.](https://github.com/FiveTechSoft/think_harbour/blob/main/README.md)

---

# Capítulo 8: Manejo de errores y depuración

* ### Tipos de errores: de sintaxis, lógicos y de tiempo de ejecución.

* ### Sentencias `ERROR HANDLER` y `ON ERROR`.

* ### Uso del depurador y herramientas de rastreo.

---

# Capítulo 9: Desarrollo de interfaces de usuario

* ### Graphic Terminal (GT).

* ### Interfaces de línea de comandos (CLI).

* ### Librerías `contrib` para GUI: `GTWVG`, `HwGUI`, y `hbqt`.

* ### Manejo de eventos y diseño de formularios.

---

# Capítulo 10: Conectividad y bases de datos avanzadas

* ### Conexión a bases de datos SQL (ODBC, MySQL, PostgreSQL, SQLite).

* ### Uso de las librerías `contrib` `hbmysql`, `hbpgsql`, `hbsqlit3`.

* ### Conceptos de SQL: sentencias `SELECT`, `INSERT`, `UPDATE`, `DELETE`.

---

# Capítulo 11: Librerías y paquetes externos

* ### El sistema de construcción `hbmk2`.

* ### Librerías `contrib` populares.

* ### Manipulación de archivos XML y JSON.

---

# Capítulo 12: Temas avanzados

* ### Programación multihilo.

* ### Desarrollo de aplicaciones web con `hbhttpd`.

* ### Uso del Garbage Collector de Harbour.

* ### Integración de código C y librerías externas.

---

# Capítulo 13: El núcleo de la máquina virtual de Harbour

* ### El compilador: el proceso de conversión de código fuente a P-Code.

* ### La máquina virtual (VM): el motor de ejecución de Harbour.

* ### P-Code: el lenguaje intermedio optimizado.

* ### La pila (`The stack`): gestión de llamadas a funciones y variables locales.

---

# Capítulo 14: El sistema de datos de Harbour (`The Harbour item`)

* ### El `item`: el contenedor de datos universal de Harbour.

* ### La API del `item`: funciones para manipular valores a nivel de C.

* ### Valores escalares: manejo de tipos de datos básicos.

* ### Manejo de `Arrays` y `Hashes`: estructura y operaciones de colecciones complejas.

* ### `Codeblocks`: el tipo de dato para código ejecutable.

* ### `Classes and Objects`: la implementación de la programación orientada a objetos.

* ### Punteros y símbolos: referencias a variables y funciones.

---

# Capítulo 15: El sistema extendido (`The Extend System`)

* ### Interoperabilidad de C y Harbour: creación de funciones externas.

* ### La Tabla de Símbolos Globales: cómo se resuelven las llamadas a funciones.

* ### Tablas de símbolos locales: gestión del alcance de las variables.

* ### Soporte `HRB`: la ejecución de binarios de Harbour independientes.

---

# Capítulo 16: Gestión de memoria y arranque (`Booting Process`)

* ### El proceso de arranque: desde el `main` de C hasta el primer PRG.

* ### El gestor de memoria: asignación y liberación de memoria.

* ### El `Garbage Collector`: automatización de la liberación de memoria.

---

# Capítulo 17: El preprocesador y los macros

* ### Directivas de compilación: `#define`, `#include`, `#command`, `#xcommand`.

* ### Creación de macros: sintaxis y uso avanzado.

---

# Capítulo 18: Depuración y estados inactivos

* ### El depurador: herramientas para rastrear y resolver errores.

* ### Sentencias de depuración: `PROCNAME()`, `VALTYPE()`, etc.

* ### `Idle states`: manejo de tareas en segundo plano.

---

# Capítulo 19: Los RDDs (`The RDDs`)

* ### Controladores de bases de datos relacionales (`The RDDs`): la arquitectura de datos de Harbour.

---

# Capítulo 20: La biblioteca de tiempo de ejecución (`The runtime library`)

* ### Funciones estándar integradas.
