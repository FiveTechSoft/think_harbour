# Capítulo 17: El preprocesador y los macros

El preprocesador de Harbour es una herramienta extremadamente poderosa que permite modificar el código fuente antes de la compilación. Va mucho más allá de las capacidades básicas de substitución, ofreciendo un sistema completo de transformación de código que permite crear DSLs (Domain Specific Languages) y simplificar sintaxis complejas.

---

## Directivas de compilación: `#define`, `#include`, `#command`, `#xcommand`

### Directiva `#define`

La directiva `#define` crea macros de sustitución textual que se procesan antes de la compilación.

#### Definiciones simples

```harbour
// Constantes básicas
#define VERSION "1.0.0"
#define MAX_USERS 100
#define DEBUG .T.

// Macros con parámetros
#define SQUARE(x) ((x) * (x))
#define MAX(a,b) (iif((a) > (b), (a), (b)))
#define LOG(msg) (iif(DEBUG, QOut("LOG: " + msg), NIL))

// Uso de las macros
LOCAL nResult := SQUARE(5)  // Se expande a ((5) * (5))
LOCAL nMaximo := MAX(10, 20)  // Se expande a (iif((10) > (20), (10), (20)))
LOG("Iniciando aplicación")   // Condicional según DEBUG
```

#### Macros condicionales

```harbour
#ifdef __HARBOUR__
   #define COMPILER_NAME "Harbour"
#else
   #define COMPILER_NAME "Clipper"
#endif

#ifndef MAX_BUFFER_SIZE
   #define MAX_BUFFER_SIZE 4096
#endif

// Compilación condicional por plataforma
#ifdef __PLATFORM__WINDOWS
   #define PATH_SEPARATOR "\"
   #define NULL_DEVICE "NUL"
#else
   #define PATH_SEPARATOR "/"
   #define NULL_DEVICE "/dev/null"
#endif
```

#### Macros anidadas y recursivas

```harbour
#define INDENT "  "
#define DOUBLE_INDENT INDENT + INDENT
#define TRIPLE_INDENT DOUBLE_INDENT + INDENT

#define TRACE_LEVEL_1(msg) QOut(INDENT + msg)
#define TRACE_LEVEL_2(msg) QOut(DOUBLE_INDENT + msg)
#define TRACE_LEVEL_3(msg) QOut(TRIPLE_INDENT + msg)
```

### Directiva `#include`

Permite incluir contenido de otros archivos en el código actual.

```harbour
// Archivo: constantes.ch
#define APP_NAME "Mi Aplicación"
#define APP_VERSION "2.1.0"
#define COPYRIGHT "© 2024 Mi Empresa"

// Archivo principal
#include "constantes.ch"
#include "hbclass.ch"   // Definiciones para POO
#include "fileio.ch"    // Constantes para manejo de archivos

FUNCTION Main()
   ? APP_NAME, APP_VERSION
   ? COPYRIGHT
   RETURN NIL
```

#### Includes condicionales

```harbour
#ifdef USE_GUI
   #include "hwgui.ch"
#else
   #include "console.ch"
#endif

// Include con verificación de existencia
#ifdef __FILE_EXISTS("local_config.ch")
   #include "local_config.ch"
#else
   #include "default_config.ch"
#endif
```

### Directiva `#command`

Permite crear nuevos comandos con sintaxis personalizada.

#### Comandos básicos

```harbour
// Definir un comando simple
#command SHOW <value> => QOut(<value>)

// Uso
SHOW "Hola mundo"  // Se traduce a: QOut("Hola mundo")
SHOW nVariable     // Se traduce a: QOut(nVariable)
```

#### Comandos con múltiples parámetros

```harbour
// Comando para validación
#command VALIDATE <var> TYPE <type> => ;
   if (ValType(<var>) != <type>) ; 
      Alert("Error: " + #<var> + " debe ser " + <type>) ; 
   endif

// Uso
LOCAL cNombre := "Juan"
LOCAL nEdad := "30"  // Error intencional
VALIDATE cNombre TYPE "C"  // OK
VALIDATE nEdad TYPE "N"    // Mostrará error
```

#### Comandos con palabras clave opcionales

```harbour
// Comando con sintaxis SQL-like
#command SELECT <fields> FROM <table> [WHERE <condition>] [ORDER BY <order>] => ;
   ProcessQuery(<table>, <fields> [, <condition>] [, <order>])

// Uso
SELECT "nombre, edad" FROM "empleados"
SELECT "nombre, edad" FROM "empleados" WHERE "edad > 30"
SELECT "nombre, edad" FROM "empleados" WHERE "activo = .T." ORDER BY "nombre"
```

### Directiva `#xcommand`

Versión extendida de `#command` con capacidades de coincidencia de patrones más avanzadas.

#### Patrones con repetición

```harbour
// Comando que acepta múltiples elementos
#xcommand CREATE MENU <menu> ;
   [MENUITEM <item1> ACTION <action1>] ;
   [MENUITEM <item2> ACTION <action2>] ;
   [MENUITEM <item3> ACTION <action3>] ;
   => <menu> := BuildMenu({[<item1>], [<item2>], [<item3>]}, ;
                          {[<action1>], [<action2>], [<action3>]})

// Uso
CREATE MENU oMenuPrincipal ;
   MENUITEM "Archivo" ACTION MenuArchivo() ;
   MENUITEM "Editar" ACTION MenuEditar() ;
   MENUITEM "Ayuda" ACTION MenuAyuda()
```

#### Patrones con listas variables

```harbour
#xcommand LOG TO <file> MESSAGE <msg1> [, <msgN>] => ;
   WriteLog(<file>, <msg1> [+ Chr(13) + Chr(10) + <msgN>])

// Uso
LOG TO "app.log" MESSAGE "Inicio de aplicación"
LOG TO "error.log" MESSAGE "Error crítico", "Detalles: " + cDetalle
```

---

## Creación de macros: sintaxis y uso avanzado

### Macros con evaluación dinámica

```harbour
// Macro que genera código dinámicamente
#define PROPERTY(name, type) ;
   PROTECTED: name AS type ; ;
   METHOD Set##name(value) INLINE (::name := value, Self) ; ;
   METHOD Get##name() INLINE ::name

// Uso en una clase
CREATE CLASS Persona
   PROPERTY(Nombre, STRING)
   PROPERTY(Edad, NUMERIC)
   PROPERTY(Activo, LOGICAL)
ENDCLASS

// Genera automáticamente:
// METHOD SetNombre(value) INLINE (::Nombre := value, Self)
// METHOD GetNombre() INLINE ::Nombre
// etc.
```

### Sistema de plantillas avanzado

```harbour
// Sistema de plantillas para generación de código
#define CRUD_OPERATIONS(table, key) ;
   FUNCTION Create##table(oData) ; ;
      RETURN InsertRecord(#table, <key>, oData) ; ;
   ; ;
   FUNCTION Read##table(xKey) ; ;
      RETURN SelectRecord(#table, <key>, xKey) ; ;
   ; ;
   FUNCTION Update##table(xKey, oData) ; ;
      RETURN UpdateRecord(#table, <key>, xKey, oData) ; ;
   ; ;
   FUNCTION Delete##table(xKey) ; ;
      RETURN DeleteRecord(#table, <key>, xKey)

// Generar operaciones CRUD para múltiples tablas
CRUD_OPERATIONS(Usuario, "id_usuario")
CRUD_OPERATIONS(Producto, "id_producto")
CRUD_OPERATIONS(Pedido, "numero_pedido")
```

### Macros para validación y verificación

```harbour
// Sistema de assertions
#define ASSERT(condition, message) ;
   if (!(condition)) ; ;
      Alert("ASSERTION FAILED: " + message) ; ;
      QUIT ; ;
   endif

#define REQUIRE_PARAM(param, type) ;
   if (PCount() < param .OR. ValType(param) != type) ; ;
      Alert("Parameter " + #param + " is required and must be " + type) ; ;
      RETURN NIL ; ;
   endif

// Uso en funciones
FUNCTION CalcularDescuento(nMonto, nPorcentaje)
   REQUIRE_PARAM(nMonto, "N")
   REQUIRE_PARAM(nPorcentaje, "N")
   ASSERT(nPorcentaje >= 0 .AND. nPorcentaje <= 100, "Porcentaje debe estar entre 0 y 100")
   
   RETURN nMonto * (nPorcentaje / 100)
```

### Macros para manejo de errores

```harbour
// Macro para try-catch simulado
#define TRY BEGIN SEQUENCE
#define CATCH(errorVar) RECOVER USING errorVar
#define FINALLY ; ALWAYS
#define END_TRY END SEQUENCE

#define THROW(error) BREAK(error)

// Uso
FUNCTION ProcesarArchivo(cNombre)
   LOCAL oError
   LOCAL cContenido
   
   TRY
      cContenido := MemoRead(cNombre)
      IF Empty(cContenido)
         THROW(ErrorNew("Archivo vacío"))
      ENDIF
      RETURN ProcesarContenido(cContenido)
   CATCH(oError)
      Alert("Error procesando archivo: " + oError:Description)
      RETURN NIL
   FINALLY
      // Código de limpieza
      IF File(cNombre + ".lock")
         FErase(cNombre + ".lock")
      ENDIF
   END_TRY
```

### DSL para configuración

```harbour
// Crear un DSL para configuración de aplicación
#command CONFIG SECTION <section> => BeginConfigSection(<section>)
#command END CONFIG => EndConfigSection()
#command SET <key> TO <value> => SetConfigValue(<key>, <value>)
#command SET <key> TO <value> TYPE <type> => SetConfigValue(<key>, <value>, <type>)

// Uso del DSL
CONFIG SECTION "Database"
   SET "Host" TO "localhost"
   SET "Port" TO 5432 TYPE "N"
   SET "Database" TO "myapp"
   SET "User" TO "admin"
   SET "Password" TO "secret123"
END CONFIG

CONFIG SECTION "Application"
   SET "Debug" TO .T. TYPE "L"
   SET "LogLevel" TO "INFO"
   SET "MaxUsers" TO 100 TYPE "N"
END CONFIG
```

### Macros para generación de interfaces

```harbour
// DSL para crear interfaces de usuario
#command DIALOG <name> SIZE <width>, <height> => ;
   <name> := CreateDialog(<width>, <height>)

#command @ <row>, <col> SAY <text> SIZE <w>, <h> => ;
   AddLabel(GetCurrentDialog(), <row>, <col>, <text>, <w>, <h>)

#command @ <row>, <col> GET <var> SIZE <w>, <h> => ;
   AddTextBox(GetCurrentDialog(), <row>, <col>, @<var>, <w>, <h>)

#command @ <row>, <col> BUTTON <text> ACTION <action> SIZE <w>, <h> => ;
   AddButton(GetCurrentDialog(), <row>, <col>, <text>, <action>, <w>, <h>)

// Uso
DIALOG dlgPersona SIZE 400, 300
   @ 10, 10 SAY "Nombre:" SIZE 80, 20
   @ 10, 100 GET cNombre SIZE 200, 20
   
   @ 40, 10 SAY "Edad:" SIZE 80, 20
   @ 40, 100 GET nEdad SIZE 100, 20
   
   @ 70, 150 BUTTON "Aceptar" ACTION Aceptar() SIZE 80, 30
   @ 70, 240 BUTTON "Cancelar" ACTION Cancelar() SIZE 80, 30
```

### Técnicas avanzadas de macros

#### Concatenación de tokens

```harbour
// Generar nombres de funciones dinámicamente
#define DEFINE_GETTER_SETTER(prop) ;
   METHOD Get##prop() INLINE ::prop ; ;
   METHOD Set##prop(value) INLINE (::prop := value, Self)

// Generar múltiples versiones de una función
#define DEFINE_CONVERTERS(type) ;
   FUNCTION To##type##String(value) ; ;
      RETURN type##ToString(value) ; ;
   ; ;
   FUNCTION From##type##String(str) ; ;
      RETURN StringTo##type(str)

DEFINE_CONVERTERS(Date)
DEFINE_CONVERTERS(Number)
DEFINE_CONVERTERS(Logical)
```

#### Macros recursivos

```harbour
// Macro para generar estructuras anidadas
#define REPEAT(code, times) ;
   RepeatCode(code, times)

// Función auxiliar para repetir código
STATIC FUNCTION RepeatCode(cCode, nTimes)
   LOCAL i, cResult := ""
   FOR i := 1 TO nTimes
      cResult += StrTran(cCode, "##i##", LTrim(Str(i))) + Chr(13) + Chr(10)
   NEXT
   RETURN cResult
```

### Depuración de macros

```harbour
// Macro para mostrar el código generado
#define DEBUG_MACRO(code) ;
   QOut("Generated code: " + #code) ; ;
   code

// Macro para tracing de ejecución
#define TRACE_ENTRY(funcname) ;
   IF s_lTrace ; ;
      QOut("ENTER: " + funcname + "(" + ProcFile() + ":" + LTrim(Str(ProcLine())) + ")") ; ;
   ENDIF

#define TRACE_EXIT(funcname, result) ;
   IF s_lTrace ; ;
      QOut("EXIT: " + funcname + " -> " + ValToChar(result)) ; ;
   ENDIF
```

El preprocesador de Harbour es una herramienta increíblemente versátil que permite crear abstracciones poderosas, simplificar código repetitivo y crear DSLs especializados para dominios específicos. Su uso maestro puede transformar dramáticamente la productividad y legibilidad del código.