# Capítulo 18: Depuración y estados inactivos

Este capítulo cubre las herramientas y técnicas avanzadas de depuración en Harbour, así como el manejo de tareas en segundo plano mediante estados inactivos.

---

## El depurador: herramientas para rastrear y resolver errores

Harbour incluye un depurador interactivo completo que permite inspeccionar el estado del programa durante la ejecución.

### Activación del depurador

#### Compilación con soporte de depuración

```bash
# Compilar con información de depuración
hbmk2 -b programa.prg

# O usando la opción debug completa
hbmk2 -debug programa.prg
```

#### Iniciar depuración

```harbour
// Método 1: Llamar explícitamente al depurador
__dbgEntry()

// Método 2: Configurar breakpoint automático
AltD()

// Método 3: Depuración condicional
IF lModoDebug
   AltD()
ENDIF
```

### Interface del depurador

#### Comandos básicos del depurador

```
F5  - Continuar ejecución
F8  - Step Over (ejecutar línea, no entrar en funciones)
F7  - Step Into (entrar en funciones)
F6  - Step Out (salir de función actual)
F9  - Toggle breakpoint
F10 - Salir del depurador

Ctrl+G - Ir a línea específica
Ctrl+F - Buscar texto
Alt+V  - Ver variables
Alt+W  - Ventana de watch
Alt+S  - Ver stack de llamadas
```

#### Ventanas del depurador

**Ventana de código:**
- Muestra el código fuente actual
- Indica la línea de ejecución actual
- Permite establecer/quitar breakpoints

**Ventana de variables locales:**
```
Variables Locales:
  nContador    = 10 (N)
  cNombre      = "Juan Pérez" (C)
  lActivo      = .T. (L)
  aLista       = Array(5) (A)
  hDatos       = Hash{3} (H)
```

**Ventana de watch:**
```
Watch Expressions:
  nContador * 2     = 20
  Len(aLista)       = 5
  hDatos["clave"]   = "valor"
  Upper(cNombre)    = "JUAN PÉREZ"
```

**Stack de llamadas:**
```
Call Stack:
  0: PROCESARDATOS(10) line 25 in proceso.prg
  1: MAIN() line 15 in programa.prg
```

### Breakpoints avanzados

#### Breakpoints condicionales

```harbour
// Breakpoint que solo se activa cuando se cumple una condición
FUNCTION ProcesarDatos( nValor )
   FOR i := 1 TO 1000
      IF i == 500  // Condición específica
         AltD()    // Solo parar cuando i = 500
      ENDIF
      
      // Procesar datos...
   NEXT
   RETURN NIL
```

#### Breakpoints programáticos

```harbour
#ifdef DEBUG
   #define DEBUG_BREAK() AltD()
#else
   #define DEBUG_BREAK()
#endif

FUNCTION FuncionCompleja()
   LOCAL nResultado
   
   // Lógica compleja...
   
   DEBUG_BREAK()  // Solo se activa en modo debug
   
   // Más lógica...
   
   RETURN nResultado
```

### Inspección de datos complejos

#### Arrays y hashes

```harbour
// En el depurador, expandir arrays y hashes
LOCAL aPersonas := { ;
   {"Juan", 30, .T.}, ;
   {"María", 25, .F.}, ;
   {"Pedro", 35, .T.} ;
}

LOCAL hConfiguracion := { ;
   "servidor" => "localhost", ;
   "puerto" => 8080, ;
   "debug" => .T., ;
   "usuarios" => {"admin", "guest"} ;
}

// El depurador mostrará:
// aPersonas[1] = {"Juan", 30, .T.}
// aPersonas[2] = {"María", 25, .F.}
// hConfiguracion["servidor"] = "localhost"
```

#### Objetos

```harbour
LOCAL oPersona := Persona():New("Juan", 30)

// En el depurador se puede inspeccionar:
// oPersona:cNombre = "Juan"
// oPersona:nEdad = 30
// oPersona:lActivo = .T.
```

---

## Sentencias de depuración: `PROCNAME()`, `VALTYPE()`, etc.

Harbour proporciona funciones específicas para ayudar en la depuración y análisis del programa.

### Funciones de información del programa

#### `PROCNAME()` - Nombre del procedimiento actual

```harbour
FUNCTION EjemploProcName()
   ? "Función actual:", ProcName()          // "EJEMPLOPROCNAME"
   ? "Función anterior:", ProcName(1)       // Función que llamó a esta
   ? "Dos niveles atrás:", ProcName(2)      // Dos niveles en el stack
   
   SubFuncion()
   RETURN NIL

FUNCTION SubFuncion()
   ? "En SubFuncion:", ProcName()           // "SUBFUNCION"
   ? "Llamada desde:", ProcName(1)          // "EJEMPLOPROCNAME"
   RETURN NIL
```

#### `PROCLINE()` - Línea actual en el procedimiento

```harbour
FUNCTION EjemploLine()
   ? "Línea actual:", ProcLine()            // Número de línea actual
   
   // Algunas líneas más...
   
   ? "Ahora en línea:", ProcLine()          // Número de línea diferente
   RETURN NIL
```

#### `PROCFILE()` - Archivo fuente actual

```harbour
FUNCTION EjemploFile()
   ? "Archivo actual:", ProcFile()          // "ejemplo.prg"
   ? "Línea:", ProcLine(), "en", ProcFile()
   RETURN NIL
```

### Funciones de tipo de datos

#### `VALTYPE()` - Tipo de una variable

```harbour
FUNCTION AnalizarTipos()
   LOCAL xVar
   
   xVar := 42
   ? "Tipo:", ValType(xVar)                 // "N" (Numérico)
   
   xVar := "Hola"
   ? "Tipo:", ValType(xVar)                 // "C" (Caracter/String)
   
   xVar := .T.
   ? "Tipo:", ValType(xVar)                 // "L" (Lógico)
   
   xVar := Date()
   ? "Tipo:", ValType(xVar)                 // "D" (Fecha)
   
   xVar := {}
   ? "Tipo:", ValType(xVar)                 // "A" (Array)
   
   xVar := {=>}
   ? "Tipo:", ValType(xVar)                 // "H" (Hash)
   
   xVar := {|| .T. }
   ? "Tipo:", ValType(xVar)                 // "B" (Block/Codeblock)
   
   RETURN NIL
```

#### `TYPE()` - Tipo de una expresión como string

```harbour
FUNCTION EjemploType()
   LOCAL nNumero := 42
   
   ? "Tipo de nNumero:", Type("nNumero")    // "N"
   ? "Tipo de variable inexistente:", Type("xNoExiste")  // "U" (Undefined)
   
   // Útil para verificar si una variable existe
   IF Type("nNumero") != "U"
      ? "La variable nNumero existe y vale:", nNumero
   ENDIF
   
   RETURN NIL
```

### Funciones de debugging avanzadas

#### Tracing personalizado

```harbour
FUNCTION TraceFunction()
   LOCAL cFuncName := ProcName()
   LOCAL nLine := ProcLine()
   
   ? "ENTRADA:", cFuncName, "línea", nLine
   
   // Tu código aquí...
   
   ? "SALIDA:", cFuncName
   RETURN NIL
```

#### Logging con contexto

```harbour
FUNCTION LogWithContext( cMessage )
   LOCAL cTimeStamp := DToC(Date()) + " " + Time()
   LOCAL cContext := ProcFile() + ":" + LTrim(Str(ProcLine())) + " " + ProcName(1)
   LOCAL cLogLine := cTimeStamp + " [" + cContext + "] " + cMessage
   
   // Escribir a archivo de log
   LogToFile( cLogLine )
   
   // También mostrar en pantalla si estamos en modo debug
   IF lDebugMode
      ? cLogLine
   ENDIF
   
   RETURN NIL

// Uso
LogWithContext( "Iniciando proceso de datos" )
```

---

## `Idle states`: manejo de tareas en segundo plano

Los estados inactivos permiten ejecutar tareas en segundo plano cuando la aplicación no está procesando entrada del usuario.

### Concepto de idle states

Los idle states se ejecutan cuando:
- La aplicación está esperando entrada del usuario
- No hay tareas prioritarias ejecutándose
- El sistema tiene ciclos de CPU disponibles

### Configuración de idle states

#### Registrar función idle

```harbour
FUNCTION Main()
   // Registrar función que se ejecutará en idle
   SetIdleHandler( @MiIdleHandler() )
   
   // Bucle principal de la aplicación
   MenuPrincipal()
   
   RETURN NIL

FUNCTION MiIdleHandler()
   STATIC nContador := 0
   
   nContador++
   
   // Ejecutar cada 100 llamadas idle (para no saturar)
   IF nContador % 100 == 0
      ActualizarReloj()
      VerificarCorreo()
      LimpiarCaches()
   ENDIF
   
   RETURN NIL
```

#### Múltiples handlers idle

```harbour
FUNCTION ConfigurarIdleHandlers()
   // Handler para actualizar interfaz
   hb_idleAdd( @ActualizarInterfaz() )
   
   // Handler para tareas de mantenimiento
   hb_idleAdd( @TareasMantenimiento() )
   
   // Handler para comunicaciones de red
   hb_idleAdd( @ProcesarRed() )
   
   RETURN NIL

FUNCTION ActualizarInterfaz()
   STATIC nUltimaActualizacion := 0
   LOCAL nTiempoActual := Seconds()
   
   // Actualizar cada segundo
   IF nTiempoActual - nUltimaActualizacion > 1
      ActualizarReloj()
      ActualizarBarraEstado()
      nUltimaActualizacion := nTiempoActual
   ENDIF
   
   RETURN NIL
```

### Tareas comunes en idle states

#### Actualización de interfaz

```harbour
FUNCTION ActualizarInterfaz()
   // Actualizar reloj en pantalla
   @ 0, 70 SAY Time()
   
   // Actualizar indicador de estado
   @ 24, 60 SAY "Estado: " + EstadoAplicacion()
   
   // Actualizar barra de progreso si hay operación en curso
   IF lOperacionEnCurso
      ActualizarBarraProgreso()
   ENDIF
   
   RETURN NIL
```

#### Comunicaciones asíncronas

```harbour
FUNCTION ProcesarComunicaciones()
   // Verificar mensajes de red pendientes
   IF HayMensajesPendientes()
      ProcesarMensajesRed()
   ENDIF
   
   // Verificar correo electrónico
   IF lVerificarCorreo .AND. TiempoParaVerificarCorreo()
      VerificarNuevosCorreos()
   ENDIF
   
   // Sincronizar datos con servidor
   IF lSincronizacionAutomatica .AND. TiempoParaSincronizar()
      SincronizarDatos()
   ENDIF
   
   RETURN NIL
```

#### Mantenimiento del sistema

```harbour
FUNCTION TareasMantenimiento()
   STATIC nUltimaLimpieza := 0
   LOCAL nTiempoActual := Seconds()
   
   // Ejecutar mantenimiento cada 5 minutos
   IF nTiempoActual - nUltimaLimpieza > 300
      // Limpiar cachés
      LimpiarCachesTemporales()
      
      // Garbage collection manual si es necesario
      IF hb_gcMemUsed() > LIMITE_MEMORIA
         hb_gcAll()
      ENDIF
      
      // Comprimir logs si son muy grandes
      IF TamañoLog() > TAMAÑO_MAXIMO_LOG
         ComprimirArchivoLog()
      ENDIF
      
      nUltimaLimpieza := nTiempoActual
   ENDIF
   
   RETURN NIL
```

### Control de idle states

#### Deshabilitar temporalmente

```harbour
FUNCTION OperacionIntensiva()
   // Deshabilitar idle handlers durante operación intensiva
   LOCAL lIdleAnterior := hb_idleState(.F.)
   
   // Realizar operación que requiere todos los recursos
   ProcesarGranCantidadDatos()
   
   // Restaurar estado idle anterior
   hb_idleState(lIdleAnterior)
   
   RETURN NIL
```

#### Sleep controlado

```harbour
FUNCTION EsperaInteligente( nSegundos )
   LOCAL nInicio := Seconds()
   LOCAL nTiempoObjetivo := nInicio + nSegundos
   
   // Esperar pero permitir idle processing
   WHILE Seconds() < nTiempoObjetivo
      hb_idleSleep( 0.1 )  // Dormir 100ms pero procesar idle
   ENDDO
   
   RETURN NIL
```

Los idle states proporcionan una manera elegante de mantener la aplicación responsiva mientras se ejecutan tareas en segundo plano, mejorando significativamente la experiencia del usuario.