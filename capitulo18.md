# Capítulo 18: Depuración y estados inactivos

La depuración eficaz y el manejo de estados inactivos son habilidades esenciales para el desarrollo profesional en Harbour. Este capítulo explora las herramientas y técnicas avanzadas para identificar, diagnosticar y resolver problemas en aplicaciones Harbour, así como la gestión de tareas en segundo plano.

---

## El depurador: herramientas para rastrear y resolver errores

Harbour incluye un depurador integrado sofisticado que permite la inspección en tiempo real del estado de la aplicación.

### Compilación con información de depuración

Para utilizar el depurador, el código debe compilarse con información de depuración:

```bash
# Compilar con información de debug
hbmk2 -b mi_programa.prg

# Con información extendida de debug
hbmk2 -b -debug mi_programa.prg

# Debug completo con símbolos
hbmk2 -b -debug -debugger mi_programa.prg
```

### Activación del depurador

```harbour
FUNCTION Main()
   // Activar depurador programáticamente
   AltD()  // Activa el depurador en el siguiente paso
   
   LOCAL nResult := CalcularResultado(10, 20)
   ? nResult
   
   RETURN NIL

FUNCTION CalcularResultado(nA, nB)
   LOCAL nTemp
   
   // Punto de ruptura condicional
   IF nA > nB
      AltD()  // Solo se activa si nA > nB
   ENDIF
   
   nTemp := nA * nB
   RETURN nTemp + (nA + nB)
```

### Comandos del depurador

Cuando el depurador está activo, los comandos disponibles incluyen:

```
F5 o Go      - Continuar ejecución
F8 o Step    - Ejecutar siguiente línea (step over)
F7 o Into    - Entrar en función (step into)
F6 o Out     - Salir de función actual (step out)
F9 o Toggle  - Alternar punto de ruptura
F10 o Quit   - Terminar depuración

// Comandos de inspección
? <variable>     - Mostrar valor de variable
?? <expression>  - Evaluar expresión
List             - Mostrar código fuente actual
Stack            - Mostrar pila de llamadas
Locals           - Mostrar variables locales
Statics          - Mostrar variables estáticas
```

### Depurador programático avanzado

```harbour
// Función para depuración condicional avanzada
FUNCTION DebugIf(lCondition, cMessage)
   IF lCondition
      ? "DEBUG:", cMessage
      ? "Pila de llamadas:"
      ShowCallStack()
      AltD()
   ENDIF
   RETURN NIL

STATIC FUNCTION ShowCallStack()
   LOCAL i := 1
   LOCAL cProc
   
   WHILE !Empty(cProc := ProcName(i))
      ? Space(i * 2) + Str(i, 2) + ": " + cProc + "(" + ProcFile(i) + ":" + ;
        LTrim(Str(ProcLine(i))) + ")"
      i++
   ENDDO
   RETURN NIL

// Uso del depurador avanzado
FUNCTION ProcesarDatos(aDatos)
   LOCAL i, resultado
   
   DebugIf(Empty(aDatos), "Array de datos está vacío")
   DebugIf(Len(aDatos) > 1000, "Array muy grande: " + LTrim(Str(Len(aDatos))))
   
   FOR i := 1 TO Len(aDatos)
      resultado := ProcesarItem(aDatos[i])
      DebugIf(resultado == NIL, "Error procesando item " + LTrim(Str(i)))
   NEXT
   
   RETURN resultado
```

### Depurador remoto

```harbour
// Configurar depuración remota
FUNCTION ConfigurarDebugRemoto()
   // Activar servidor de debug
   hb_DebugServerStart(8080)
   
   // Conectar cliente remoto
   hb_DebugClientConnect("192.168.1.100", 8080)
   
   RETURN NIL

// Puntos de ruptura remotos
FUNCTION SetRemoteBreakpoint(cFile, nLine)
   hb_DebugSetBreakpoint(cFile, nLine, .T.)
   RETURN NIL
```

---

## Sentencias de depuración: `PROCNAME()`, `VALTYPE()`, etc.

Harbour proporciona funciones intrínsecas especializadas para la depuración y diagnóstico.

### Funciones de introspección del programa

```harbour
FUNCTION InformacionEjecucion()
   ? "Función actual:", ProcName()
   ? "Archivo actual:", ProcFile()
   ? "Línea actual:", ProcLine()
   ? "Nivel de anidamiento:", ProcLevel()
   
   // Mostrar toda la pila de llamadas
   LOCAL i := 0
   ? "=== PILA DE LLAMADAS ==="
   WHILE i <= ProcLevel()
      ? Str(i, 3) + ":", ProcName(i), "en", ProcFile(i), "línea", ProcLine(i)
      i++
   ENDDO
   
   RETURN NIL

FUNCTION FuncionAnidada()
   InformacionEjecucion()
   RETURN NIL

FUNCTION Main()
   FuncionAnidada()
   RETURN NIL
```

### Funciones de análisis de tipos

```harbour
FUNCTION AnalizarVariable(xVariable, cNombre)
   LOCAL cTipo := ValType(xVariable)
   LOCAL cInfo := "Variable: " + cNombre + " "
   
   DO CASE
   CASE cTipo == "C"
      cInfo += "String(" + LTrim(Str(Len(xVariable))) + "): '" + xVariable + "'"
      
   CASE cTipo == "N"
      cInfo += "Numeric: " + LTrim(Str(xVariable))
      IF xVariable == Int(xVariable)
         cInfo += " (Entero)"
      ELSE
         cInfo += " (Decimal)"
      ENDIF
      
   CASE cTipo == "L"
      cInfo += "Logical: " + iif(xVariable, ".T.", ".F.")
      
   CASE cTipo == "D"
      cInfo += "Date: " + DToC(xVariable)
      
   CASE cTipo == "A"
      cInfo += "Array[" + LTrim(Str(Len(xVariable))) + "]"
      AnalizarArray(xVariable, cNombre)
      
   CASE cTipo == "H"
      cInfo += "Hash{" + LTrim(Str(Len(xVariable))) + "}"
      AnalizarHash(xVariable, cNombre)
      
   CASE cTipo == "B"
      cInfo += "CodeBlock"
      
   CASE cTipo == "O"
      cInfo += "Object: " + xVariable:ClassName()
      AnalizarObjeto(xVariable, cNombre)
      
   OTHERWISE
      cInfo += "Tipo desconocido: " + cTipo
   ENDCASE
   
   ? cInfo
   RETURN NIL

STATIC FUNCTION AnalizarArray(aArray, cNombre)
   LOCAL i
   ? "  Contenido del array " + cNombre + ":"
   FOR i := 1 TO Min(Len(aArray), 10)  // Mostrar solo primeros 10 elementos
      ?? "  [" + LTrim(Str(i)) + "] "
      AnalizarVariable(aArray[i], cNombre + "[" + LTrim(Str(i)) + "]")
   NEXT
   IF Len(aArray) > 10
      ? "  ... (" + LTrim(Str(Len(aArray) - 10)) + " elementos más)"
   ENDIF
   RETURN NIL

STATIC FUNCTION AnalizarHash(hHash, cNombre)
   LOCAL aKeys := hb_HKeys(hHash)
   LOCAL i
   ? "  Contenido del hash " + cNombre + ":"
   FOR i := 1 TO Min(Len(aKeys), 10)
      ? "  [" + ValToChar(aKeys[i]) + "] =>",
      AnalizarVariable(hHash[aKeys[i]], cNombre + "[" + ValToChar(aKeys[i]) + "]")
   NEXT
   RETURN NIL
```

### Sistema de logging avanzado

```harbour
// Sistema de logging con niveles
#define LOG_ERROR   1
#define LOG_WARNING 2
#define LOG_INFO    3
#define LOG_DEBUG   4

STATIC s_nLogLevel := LOG_INFO
STATIC s_cLogFile := "app.log"

FUNCTION SetLogLevel(nLevel)
   s_nLogLevel := nLevel
   RETURN NIL

FUNCTION WriteLog(nLevel, cMessage, xData)
   LOCAL cLevel, cLogEntry, hFile
   LOCAL cTimeStamp := DToC(Date()) + " " + Time()
   
   IF nLevel > s_nLogLevel
      RETURN NIL  // No registrar si está por debajo del nivel
   ENDIF
   
   DO CASE
   CASE nLevel == LOG_ERROR
      cLevel := "ERROR"
   CASE nLevel == LOG_WARNING
      cLevel := "WARN "
   CASE nLevel == LOG_INFO
      cLevel := "INFO "
   CASE nLevel == LOG_DEBUG
      cLevel := "DEBUG"
   OTHERWISE
      cLevel := "UNKN "
   ENDCASE
   
   cLogEntry := cTimeStamp + " [" + cLevel + "] " + ;
                ProcName(1) + "(" + LTrim(Str(ProcLine(1))) + "): " + cMessage
   
   IF xData != NIL
      cLogEntry += " | Datos: " + ValToChar(xData)
   ENDIF
   
   // Escribir al archivo
   hFile := FOpen(s_cLogFile, FO_CREAT + FO_WRITE)
   IF hFile >= 0
      FSeek(hFile, 0, FS_END)
      FWrite(hFile, cLogEntry + hb_eol())
      FClose(hFile)
   ENDIF
   
   // Mostrar en consola si es error crítico
   IF nLevel == LOG_ERROR
      ? cLogEntry
   ENDIF
   
   RETURN NIL

// Macros para simplificar el uso
#command LOG_ERR <msg> [DATA <data>] => WriteLog(LOG_ERROR, <msg> [, <data>])
#command LOG_WARN <msg> [DATA <data>] => WriteLog(LOG_WARNING, <msg> [, <data>])
#command LOG_INFO <msg> [DATA <data>] => WriteLog(LOG_INFO, <msg> [, <data>])
#command LOG_DEBUG <msg> [DATA <data>] => WriteLog(LOG_DEBUG, <msg> [, <data>])

// Uso
FUNCTION ProcesarPedido(nId)
   LOCAL oPedido
   
   LOG_INFO "Iniciando procesamiento de pedido" DATA nId
   
   oPedido := CargarPedido(nId)
   IF oPedido == NIL
      LOG_ERR "No se pudo cargar el pedido" DATA nId
      RETURN .F.
   ENDIF
   
   LOG_DEBUG "Pedido cargado correctamente" DATA oPedido
   
   IF !ValidarPedido(oPedido)
      LOG_WARN "Pedido no válido" DATA oPedido:ToHash()
      RETURN .F.
   ENDIF
   
   LOG_INFO "Pedido procesado exitosamente" DATA nId
   RETURN .T.
```

---

## `Idle states`: manejo de tareas en segundo plano

Los estados inactivos permiten ejecutar tareas cuando la aplicación no está procesando eventos principales.

### Configuración básica de idle states

```harbour
FUNCTION ConfigurarIdleStates()
   // Configurar función que se ejecuta en idle
   hb_IdleAdd({|| ProcesarTareasSegundoPlano()})
   
   // Configurar múltiples funciones idle
   hb_IdleAdd({|| LimpiarCacheTemporal()})
   hb_IdleAdd({|| ActualizarEstadisticas()})
   hb_IdleAdd({|| VerificarConexiones()})
   
   RETURN NIL

STATIC FUNCTION ProcesarTareasSegundoPlano()
   STATIC s_nContador := 0
   
   s_nContador++
   
   // Ejecutar cada 100 ciclos idle (aprox.)
   IF s_nContador % 100 == 0
      ? "Procesando tareas en segundo plano:", Time()
      GarbageCollect()
   ENDIF
   
   RETURN NIL
```

### Sistema de tareas programadas

```harbour
// Estructura para tareas programadas
STATIC s_aTareas := {}

FUNCTION AgendarTarea(cNombre, bTarea, nIntervalo, tUltimaEjecucion)
   LOCAL hTarea := {=>}
   
   hTarea["nombre"] := cNombre
   hTarea["tarea"] := bTarea
   hTarea["intervalo"] := nIntervalo  // En segundos
   hTarea["ultima"] := iif(tUltimaEjecucion == NIL, hb_DateTime(), tUltimaEjecucion)
   hTarea["activa"] := .T.
   
   AAdd(s_aTareas, hTarea)
   
   RETURN Len(s_aTareas)

FUNCTION EjecutarTareasProgramadas()
   LOCAL i, hTarea, tAhora := hb_DateTime()
   
   FOR i := 1 TO Len(s_aTareas)
      hTarea := s_aTareas[i]
      
      IF hTarea["activa"] .AND. ;
         (tAhora - hTarea["ultima"]) >= hTarea["intervalo"]
         
         LOG_DEBUG "Ejecutando tarea programada: " + hTarea["nombre"]
         
         TRY
            Eval(hTarea["tarea"])
            hTarea["ultima"] := tAhora
         CATCH
            LOG_ERR "Error en tarea programada: " + hTarea["nombre"]
         END
      ENDIF
   NEXT
   
   RETURN NIL

// Configurar el sistema de tareas
FUNCTION ConfigurarSistemaTareas()
   // Limpiar archivos temporales cada 5 minutos
   AgendarTarea("LimpiarTemp", {|| LimpiarArchivosTemporales()}, 300)
   
   // Respaldar datos cada 30 minutos
   AgendarTarea("Respaldo", {|| CrearRespaldoAutomatico()}, 1800)
   
   // Verificar conexión a BD cada minuto
   AgendarTarea("VerificarBD", {|| VerificarConexionBD()}, 60)
   
   // Configurar idle para ejecutar tareas
   hb_IdleAdd({|| EjecutarTareasProgramadas()})
   
   RETURN NIL
```

### Monitor de rendimiento en tiempo real

```harbour
STATIC s_hEstadisticas := {=>}

FUNCTION InicializarMonitor()
   s_hEstadisticas["inicio"] := hb_DateTime()
   s_hEstadisticas["ciclos_idle"] := 0
   s_hEstadisticas["memoria_max"] := 0
   s_hEstadisticas["consultas_bd"] := 0
   s_hEstadisticas["errores"] := 0
   
   // Monitor en idle
   hb_IdleAdd({|| ActualizarEstadisticasRendimiento()})
   
   RETURN NIL

STATIC FUNCTION ActualizarEstadisticasRendimiento()
   LOCAL nMemoriaActual
   
   s_hEstadisticas["ciclos_idle"]++
   
   // Monitorear uso de memoria
   nMemoriaActual := hb_MemoryUsed()
   IF nMemoriaActual > s_hEstadisticas["memoria_max"]
      s_hEstadisticas["memoria_max"] := nMemoriaActual
   ENDIF
   
   // Mostrar estadísticas cada 1000 ciclos
   IF s_hEstadisticas["ciclos_idle"] % 1000 == 0
      MostrarEstadisticas()
   ENDIF
   
   RETURN NIL

FUNCTION MostrarEstadisticas()
   LOCAL nTiempoTotal := hb_DateTime() - s_hEstadisticas["inicio"]
   
   ? "=== ESTADÍSTICAS DE RENDIMIENTO ==="
   ? "Tiempo total:", Int(nTiempoTotal), "segundos"
   ? "Ciclos idle:", s_hEstadisticas["ciclos_idle"]
   ? "Memoria máxima:", s_hEstadisticas["memoria_max"], "bytes"
   ? "Consultas BD:", s_hEstadisticas["consultas_bd"]
   ? "Errores:", s_hEstadisticas["errores"]
   ? "Promedio ciclos/seg:", Int(s_hEstadisticas["ciclos_idle"] / Max(nTiempoTotal, 1))
   
   RETURN NIL
```

### Gestión avanzada de idle states

```harbour
// Control granular de idle states
FUNCTION ControlIdleStates()
   LOCAL nIdleHandle1, nIdleHandle2
   
   // Agregar funciones idle con identificadores
   nIdleHandle1 := hb_IdleAdd({|| TareaCritica()})
   nIdleHandle2 := hb_IdleAdd({|| TareaOpcional()})
   
   // Pausar tarea específica temporalmente
   hb_IdleSleep(nIdleHandle2)
   
   // Reactivar tarea
   hb_IdleWake(nIdleHandle2)
   
   // Eliminar tarea específica
   hb_IdleDel(nIdleHandle1)
   
   RETURN NIL

// Idle state consciente de la carga del sistema
STATIC FUNCTION IdleAdaptativo()
   STATIC s_nCarga := 0
   LOCAL nMemoriaUsada := hb_MemoryUsed()
   LOCAL nCpuLoad := GetCpuLoad()  // Función hipotética
   
   // Ajustar frecuencia según la carga
   IF nCpuLoad > 80
      s_nCarga++
      IF s_nCarga > 10
         hb_IdleSleep()  // Pausar temporalmente
         hb_RelIdle(1000)  // Esperar 1 segundo
         s_nCarga := 0
      ENDIF
   ELSE
      s_nCarga := Max(0, s_nCarga - 1)
   ENDIF
   
   RETURN NIL
```

### Depuración de idle states

```harbour
FUNCTION DebugIdleStates()
   // Contador global de idle
   STATIC s_nIdleCount := 0
   
   s_nIdleCount++
   
   // Log periódico de actividad idle
   IF s_nIdleCount % 500 == 0
      LOG_DEBUG "Idle state ejecutado " + LTrim(Str(s_nIdleCount)) + " veces"
      
      // Verificar si hay deadlocks o loops infinitos
      IF s_nIdleCount > 100000
         LOG_ERR "Posible loop infinito en idle states"
         ? "ALERTA: Demasiados ciclos idle, presione una tecla para continuar"
         Inkey(0)
      ENDIF
   ENDIF
   
   RETURN NIL
```

Los estados inactivos son una herramienta poderosa para crear aplicaciones responsivas que pueden realizar tareas de mantenimiento, monitoreo y actualización en segundo plano sin bloquear la interacción del usuario.