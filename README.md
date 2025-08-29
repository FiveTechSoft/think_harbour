# think_harbour

## English Version

# Chapter 1: The Program's Path

* [What is a program?](chapter1.md#what-is-a-program)
* [The Harbour language: a compiler and xBase environment.](chapter1.md#the-harbour-language-a-compiler-and-xbase-environment)
* [The Harbour preprocessor: macros (`#define`), file inclusion (`#include`), and user-defined commands (`#command`, `#xcommand`).](chapter1.md#the-harbour-preprocessor-macros-define-file-inclusion-include-and-user-defined-commands-command-xcommand)
* [Your first program: "Hello, world!".](chapter1.md#your-first-program-hello-world)
* [Problem solving and debugging.](chapter1.md#problem-solving-and-debugging)

---

# Chapter 2: Variables, expressions and statements

* [Values and data types in Harbour: `String`, `Number`, `Logical`, `Date`, `CodeBlock`, `Symbol` and `Pointer`.](chapter2.md#values-and-data-types-in-harbour)
* [Variables and their scope (`LOCAL`, `STATIC`, `PRIVATE`, `PUBLIC`).](chapter2.md#variables-and-their-scope)
* [Assignment statements and operators.](chapter2.md#assignment-statements-and-operators)
* [Expressions.](chapter2.md#expressions)

---

# Chapter 3: Functions

* [Procedures (`PROCEDURE`) vs. Functions (`FUNCTION`).](chapter3.md#procedures-procedure-vs-functions-function)
* [Parameters and arguments.](chapter3.md#parameters-and-arguments)
* [Input/output (I/O) functions.](chapter3.md#inputoutput-io-functions)
* [Utility functions (`VALTYPE()`, `PROCNAME()`).](chapter3.md#utility-functions-valtype-procname)

---

# Chapter 4: Control structures

* [Conditional statements: `IF`, `ELSEIF`, `ELSE`, `ENDIF`, `SWITCH`.](chapter4.md#conditional-statements)
* [Loops: `DO WHILE`, `FOR EACH`, `FOR...NEXT`.](chapter4.md#loops)
* [Sequence handler: `BEGIN SEQUENCE...BREAK...END`.](chapter4.md#sequence-handler)

---

# Chapter 5: Complex data types

* [Arrays: ordered lists.](chapter5.md#arrays)
* [Hashes: key-value collections.](chapter5.md#hashes)
* [Differences and when to use each one.](chapter5.md#differences-and-when-to-use-each-one)

---

# Chapter 6: Files and databases

* [Low-level file handling (`FCREATE`, `FOPEN`, `FCLOSE`).](chapter6.md#low-level-file-handling)
* [The `.dbf` file format.](chapter6.md#the-dbf-file-format)
* [Database functions: `USE`, `APPEND BLANK`, `REPLACE`.](chapter6.md#database-functions)
* [Indexes: how to speed up searches.](chapter6.md#indexes-how-to-speed-up-searches)
* [Relational Database Management Systems (RDDs).](chapter6.md#relational-database-management-systems-rdds)

---

# Chapter 7: Classes and objects

* [OOP concepts: classes, objects, variables and methods.](chapter7.md#oop-concepts-classes-objects-variables-and-methods)
* [Creating a class in Harbour.](chapter7.md#creating-a-class-in-harbour)
* [Object instantiation.](chapter7.md#object-instantiation)
* [Inheritance.](chapter7.md#inheritance)

---

# Chapter 8: Error handling and debugging

* [Types of errors: syntax, logic and runtime errors.](chapter8.md#types-of-errors)
* [`ERROR HANDLER` and `ON ERROR` statements.](chapter8.md#traditional-error-handling-on-error)
* [Using the debugger and tracing tools.](chapter8.md#debugging-and-tracing-tools)

---

# Chapter 9: User interface development

* [Graphic Terminal (GT).](chapter9.md#graphic-terminal-gt-the-foundation-of-portability)
* [Command line interfaces (CLI).](chapter9.md#command-line-interfaces-cli)
* [`contrib` libraries for GUI: `GTWVG`, `HwGUI`, and `hbqt`.](chapter9.md#contributed-libraries-for-gui-gtwvg-hwgui-and-hbqt)
* [Event handling and form design.](chapter9.md#event-handling-and-form-design)

---

# Chapter 10: Connectivity and advanced databases

* [Connection to SQL databases (ODBC, MySQL, PostgreSQL, SQLite).](chapter10.md#1-connection-to-sql-databases)
* [Using `contrib` libraries `hbmysql`, `hbpgsql`, `hbsqlit3`.](chapter10.md#2-using-contrib-libraries)
* [SQL concepts: `SELECT`, `INSERT`, `UPDATE`, `DELETE` statements.](chapter10.md#3-fundamental-sql-concepts)

---

# Chapter 11: Libraries and external packages

* [The `hbmk2` build system.](chapter11.md#1-the-hbmk2-build-system)
* [Popular `contrib` libraries.](chapter11.md#2-popular-contrib-libraries)
* [XML and JSON file manipulation.](chapter11.md#3-xml-and-json-file-manipulation)

---

# Chapter 12: Advanced topics

* [Multithreading programming.](chapter12.md#1-multithreading-programming)
* [Web application development with `hbhttpd`.](chapter12.md#2-web-application-development-with-hbhttpd)
* [Using Harbour's Garbage Collector.](chapter12.md#3-using-harbours-garbage-collector-gc)
* [Integration of C code and external libraries.](chapter12.md#4-integration-of-c-code-and-external-libraries)

---

# Chapter 13: The core of Harbour's virtual machine

* [The compiler: the process of converting source code to P-Code.](chapter13.md#the-compiler-the-process-of-converting-source-code-to-p-code)
* [The virtual machine (VM): Harbour's execution engine.](chapter13.md#the-virtual-machine-vm-harbours-execution-engine)
* [P-Code: the optimized intermediate language.](chapter13.md#p-code-the-optimized-intermediate-language)
* [The stack: managing function calls and local variables.](chapter13.md#the-stack-managing-function-calls-and-local-variables)

---

## Versión en Español

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

# Capítulo 7: Clases y objetos

* [Conceptos de OOP: clases, objetos, variables y métodos.](capitulo7.md#conceptos-de-oop-clases-objetos-variables-y-métodos)
* [Creación de una clase en Harbour.](capitulo7.md#creación-de-una-clase-en-harbour)
* [Instanciación de objetos.](capitulo7.md#instanciación-de-objetos)
* [Herencia.](capitulo7.md#herencia)

---

# Capítulo 8: Manejo de errores y depuración

* [Tipos de errores: de sintaxis, lógicos y de tiempo de ejecución.](capitulo8.md#tipos-de-errores-de-sintaxis-lógicos-y-de-tiempo-de-ejecución)
* [Sentencias `ERROR HANDLER` y `ON ERROR`.](capitulo8.md#sentencias-error-handler-y-on-error)
* [Uso del depurador y herramientas de rastreo.](capitulo8.md#uso-del-depurador-y-herramientas-de-rastreo)

---

# Capítulo 9: Desarrollo de interfaces de usuario

* [Graphic Terminal (GT).](capitulo9.md#graphic-terminal-gt)
* [Interfaces de línea de comandos (CLI).](capitulo9.md#interfaces-de-línea-de-comandos-cli)
* [Librerías `contrib` para GUI: `GTWVG`, `HwGUI`, y `hbqt`.](capitulo9.md#librerías-contrib-para-gui-gtwvg-hwgui-y-hbqt)
* [Manejo de eventos y diseño de formularios.](capitulo9.md#manejo-de-eventos-y-diseño-de-formularios)

---

# Capítulo 10: Conectividad y bases de datos avanzadas

* [Conexión a bases de datos SQL (ODBC, MySQL, PostgreSQL, SQLite).](capitulo10.md#conexión-a-bases-de-datos-sql-odbc-mysql-postgresql-sqlite)
* [Uso de las librerías `contrib` `hbmysql`, `hbpgsql`, `hbsqlit3`.](capitulo10.md#uso-de-las-librerías-contrib-hbmysql-hbpgsql-hbsqlit3)
* [Conceptos de SQL: sentencias `SELECT`, `INSERT`, `UPDATE`, `DELETE`.](capitulo10.md#conceptos-de-sql-sentencias-select-insert-update-delete)

---

# Capítulo 11: Librerías y paquetes externos

* [El sistema de construcción `hbmk2`.](capitulo11.md#el-sistema-de-construcción-hbmk2)
* [Librerías `contrib` populares.](capitulo11.md#librerías-contrib-populares)
* [Manipulación de archivos XML y JSON.](capitulo11.md#manipulación-de-archivos-xml-y-json)

---

# Capítulo 12: Temas avanzados

* [Programación multihilo.](capitulo12.md#programación-multihilo)
* [Desarrollo de aplicaciones web con `hbhttpd`.](capitulo12.md#desarrollo-de-aplicaciones-web-con-hbhttpd)
* [Uso del Garbage Collector de Harbour.](capitulo12.md#uso-del-garbage-collector-de-harbour)
* [Integración de código C y librerías externas.](capitulo12.md#integración-de-código-c-y-librerías-externas)

---

# Capítulo 13: El núcleo de la máquina virtual de Harbour

* [El compilador: el proceso de conversión de código fuente a P-Code.](capitulo13.md#el-compilador-el-proceso-de-conversión-de-código-fuente-a-p-code)
* [La máquina virtual (VM): el motor de ejecución de Harbour.](capitulo13.md#la-máquina-virtual-vm-el-motor-de-ejecución-de-harbour)
* [P-Code: el lenguaje intermedio optimizado.](capitulo13.md#p-code-el-lenguaje-intermedio-optimizado)
* [La pila (`The stack`): gestión de llamadas a funciones y variables locales.](capitulo13.md#la-pila-the-stack-gestión-de-llamadas-a-funciones-y-variables-locales)

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
