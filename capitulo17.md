# Capítulo 17: El preprocesador y los macros

Este capítulo profundiza en el preprocesador de Harbour, explorando las directivas de compilación y la creación de macros avanzados que permiten extender y personalizar el lenguaje.

---

## Directivas de compilación: `#define`, `#include`, `#command`, `#xcommand`

El preprocesador de Harbour es una herramienta poderosa que procesa el código fuente antes de la compilación, permitiendo definir constantes, incluir archivos y crear comandos personalizados.

### `#define`: Definición de constantes y macros simples

La directiva `#define` permite crear constantes simbólicas y macros de sustitución simple.

#### Constantes básicas

```harbour
#define PI           3.14159265359
#define MAX_CLIENTES 1000
#define VERDADERO    .T.
#define FALSO        .F.

// Uso en el código
LOCAL nCircunferencia := 2 * PI * nRadio
IF nNumClientes > MAX_CLIENTES
   ? "Límite de clientes excedido"
ENDIF
```

#### Macros con parámetros

```harbour
#define MIN(a,b)     iif((a) < (b), (a), (b))
#define MAX(a,b)     iif((a) > (b), (a), (b))
#define SQUARE(x)    ((x) * (x))

// Uso
LOCAL nMenor := MIN(10, 20)      // Resultado: 10
LOCAL nCuadrado := SQUARE(5)     // Resultado: 25
```

#### Macros de cadenas

```harbour
#define MENSAJE(texto)  QOut("LOG: " + (texto))
#define ERROR_MSG(err)  Alert("Error: " + (err))

// Uso
MENSAJE("Aplicación iniciada")
ERROR_MSG("Archivo no encontrado")
```

### `#include`: Inclusión de archivos

La directiva `#include` permite incluir el contenido de otros archivos en el código fuente.

```harbour
// En archivo constantes.ch
#define VERSION_MAJOR 2
#define VERSION_MINOR 1
#define VERSION_BUILD 0

// En archivo principal.prg
#include "constantes.ch"
#include "hbapi.ch"        // Archivo del sistema
#include "common.ch"       // Definiciones comunes

FUNCTION Main()
   ? "Versión:", VERSION_MAJOR, VERSION_MINOR, VERSION_BUILD
   RETURN NIL
```

#### Búsqueda de archivos include

1. Directorio actual
2. Directorios especificados con `-I` en la línea de comandos
3. Directorios de include del sistema Harbour

### `#command`: Creación de comandos personalizados

La directiva `#command` permite crear nuevos comandos que se traduzcan a código Harbour estándar.

#### Sintaxis básica

```harbour
#command MOSTRAR <texto> => QOut(<texto>)

// Uso
MOSTRAR "Hola Mundo"
// Se traduce a: QOut("Hola Mundo")
```

#### Comandos con múltiples parámetros

```harbour
#command LOG <nivel> <mensaje> => ;
   WriteLog(<nivel>, <mensaje>, ProcName(), ProcLine())

// Uso
LOG "INFO" "Proceso completado"
// Se traduce a: WriteLog("INFO", "Proceso completado", ProcName(), ProcLine())
```

#### Comandos con opciones

```harbour
#command CONECTAR DATABASE <db> [HOST <host>] [PORT <puerto>] => ;
   ConectarBD(<db>, <host>, <puerto>)

// Uso
CONECTAR DATABASE "ventas" HOST "localhost" PORT 3306
// Se traduce a: ConectarBD("ventas", "localhost", 3306)

CONECTAR DATABASE "local"
// Se traduce a: ConectarBD("local", , )
```

### `#xcommand`: Comandos extendidos

`#xcommand` es una versión más potente de `#command` que permite patrones más complejos.

#### Comandos con repetición

```harbour
#xcommand SELECT <campos,...> FROM <tabla> [WHERE <condicion>] => ;
   local aResult := {}; ;
   UseTable(<tabla>); ;
   [if <condicion>; ]
   [   set filter to <condicion>; ]
   [endif; ]
   while !eof(); ;
      aadd(aResult, {<campos>}); ;
      skip; ;
   end; ;
   close

// Uso
SELECT nombre, edad FROM clientes WHERE edad > 25
```

#### Comandos con bloques anidados

```harbour
#xcommand TRY => BEGIN SEQUENCE WITH {|e| Break(e)}
#xcommand CATCH <error> => RECOVER [USING <error>]; <error> := ErrorNew()
#xcommand FINALLY => ALWAYS
#xcommand ENDTRY => END SEQUENCE

// Uso
TRY
   LOCAL nResult := 10 / 0
CATCH oError
   ? "Error:", oError:Description
FINALLY
   ? "Limpieza"
ENDTRY
```

---

## Creación de macros: sintaxis y uso avanzado

### Patrones de coincidencia avanzados

#### Wildcards y operadores especiales

```harbour
// <lista,...> - Lista separada por comas
#xcommand ARRAY <var> OF <lista,...> => ;
   <var> := {<lista>}

// Uso: ARRAY aNumeros OF 1, 2, 3, 4, 5

// [<opcional>] - Parámetro opcional
#xcommand OPEN FILE <archivo> [MODE <modo>] => ;
   hFile := FOpen(<archivo> [, <modo>])

// <!lista!> - Lista que puede estar vacía
#xcommand CALL <func> [WITH <!params!>] => ;
   <func>(<params>)
```

#### Operadores de transformación

```harbour
// & - Operador de macro expansión
#define CREAR_FUNCION(nombre) ;
   FUNCTION &nombre; ;
   RETURN "Función " + #<nombre>

// #<> - Conversión a string literal
#command ASSERT <expresion> => ;
   if !(<expresion>); ;
      Alert("Assertion failed: " + #<expresion>); ;
   endif

// Uso
ASSERT nValor > 0
// Se traduce a: if !(nValor > 0); Alert("Assertion failed: nValor > 0"); endif
```

### Macros condicionales

```harbour
// Compilación condicional
#ifdef __HARBOUR__
   #define HARBOUR_VERSION .T.
#else
   #define HARBOUR_VERSION .F.
#endif

#ifndef DEBUG
   #define DEBUG .F.
#endif

#if DEBUG
   #command DEBUG_PRINT <texto> => QOut("DEBUG: " + <texto>)
#else
   #command DEBUG_PRINT <texto> =>
#endif
```

### Macros anidados y recursivos

```harbour
// Macro que genera otros macros
#define CREAR_GETTER_SETTER(campo) ;
   #command GET_&campo => ::&campo ;
   #command SET_&campo <valor> => ::&campo := <valor>

// Uso
CREAR_GETTER_SETTER(NOMBRE)
// Genera:
// #command GET_NOMBRE => ::NOMBRE
// #command SET_NOMBRE <valor> => ::NOMBRE := <valor>
```

### Macros para DSL (Domain Specific Languages)

#### Ejemplo: Mini-lenguaje de configuración

```harbour
#xcommand CONFIG START => LOCAL hConfig := {=>}
#xcommand CONFIG END => RETURN hConfig
#xcommand SET <key> TO <value> => hConfig[<"key">] := <value>
#xcommand DATABASE <db> USER <user> PASSWORD <pass> => ;
   hConfig["database"] := <db>; ;
   hConfig["user"] := <user>; ;
   hConfig["password"] := <pass>

// Uso
FUNCTION CrearConfiguracion()
   CONFIG START
      SET "servidor" TO "localhost"
      SET "puerto" TO 8080
      DATABASE "ventas" USER "admin" PASSWORD "secreto"
   CONFIG END
```

#### Ejemplo: Builder pattern

```harbour
#xcommand BUILD <objeto> => LOCAL o&objeto := T&objeto():New()
#xcommand SET <prop> TO <valor> => o&objeto:&prop := <valor>
#xcommand WITH METHOD <metodo> PARAMS <params,...> => o&objeto:&metodo(<params>)
#xcommand RETURN OBJECT => RETURN o&objeto

// Uso
FUNCTION CrearPersona()
   BUILD Persona
      SET nombre TO "Juan"
      SET edad TO 30
      WITH METHOD SetEmail PARAMS "juan@email.com"
   RETURN OBJECT
```

### Debugging y troubleshooting de macros

#### Expansión de macros

```harbour
// Ver la expansión de macros con -p en hbmk2
// hbmk2 -p programa.prg

// Resultado mostrará el código después de la expansión del preprocesador
```

#### Macros de debug

```harbour
#define __FUNCTION__ ProcName()
#define __LINE__     ProcLine()
#define __FILE__     ProcFile()

#command TRACE <mensaje> => ;
   QOut("[" + __FILE__ + ":" + LTrim(Str(__LINE__)) + " " + __FUNCTION__ + "] " + <mensaje>)

// Uso
FUNCTION MiFuncion()
   TRACE "Iniciando función"
   // ... código ...
   TRACE "Función completada"
   RETURN NIL
```

### Mejores prácticas para macros

#### Convenciones de nombres

```harbour
// Usar prefijos para evitar conflictos
#define MY_APP_VERSION "1.0"
#define MY_APP_DEBUG   .T.

// Usar MAYÚSCULAS para constantes
#define PI 3.14159
#define MAX_BUFFER_SIZE 1024

// Usar snake_case o PascalCase para comandos
#command CREATE_OBJECT <clase> => <clase>():New()
```

#### Documentación de macros

```harbour
/*
 * Macro: LOG_MESSAGE
 * Propósito: Escribir mensaje en log con timestamp
 * Parámetros:
 *   level - Nivel del mensaje (INFO, WARN, ERROR)
 *   msg   - Mensaje a escribir
 * Ejemplo:
 *   LOG_MESSAGE "INFO" "Aplicación iniciada"
 */
#command LOG_MESSAGE <level> <msg> => ;
   WriteToLog(<level>, <msg>, DateTime())
```

Los macros y el preprocesador de Harbour proporcionan un mecanismo poderoso para extender el lenguaje y crear abstracciones de alto nivel que hacen el código más expresivo y mantenible.