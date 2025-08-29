# Capítulo 20: La biblioteca de tiempo de ejecución (`The runtime library`)

Este capítulo explora la biblioteca de tiempo de ejecución de Harbour, que proporciona las funciones estándar integradas que forman la base de todas las aplicaciones Harbour.

---

## Funciones estándar integradas

La biblioteca de tiempo de ejecución (runtime library) de Harbour contiene cientos de funciones predefinidas que cubren todas las áreas de funcionalidad básica, desde manipulación de cadenas hasta acceso a archivos y gestión de memoria.

### Categorías de funciones de la biblioteca

#### Funciones de cadenas (String Functions)

Las funciones de cadenas son fundamentales para el procesamiento de texto:

```harbour
FUNCTION EjemploFuncionesCadenas()
   LOCAL cTexto := "  Harbour Programming Language  "
   
   // Funciones básicas
   ? "Longitud:", Len(cTexto)                    // 33
   ? "Sin espacios:", Trim(cTexto)               // "Harbour Programming Language"
   ? "Mayúsculas:", Upper(cTexto)                // "  HARBOUR PROGRAMMING LANGUAGE  "
   ? "Minúsculas:", Lower(cTexto)                // "  harbour programming language  "
   
   // Funciones de búsqueda
   ? "Posición 'Program':", At("Program", cTexto)  // 9
   ? "Última 'a':", RAt("a", cTexto)             // 25
   
   // Extracción de subcadenas
   ? "Primeros 7:", Left(cTexto, 7)              // "  Harbo"
   ? "Últimos 8:", Right(cTexto, 8)              // "anguage  "
   ? "Desde pos 3, 7 chars:", SubStr(cTexto, 3, 7)  // "Harbour"
   
   // Transformaciones
   ? "Reemplazar:", StrTran(cTexto, "Harbour", "xBase")  // "  xBase Programming Language  "
   ? "Rellenar:", PadL("123", 6, "0")            // "000123"
   ? "Rellenar der:", PadR("ABC", 6, "*")        // "ABC***"
   
   RETURN NIL
```

#### Funciones numéricas (Numeric Functions)

```harbour
FUNCTION EjemploFuncionesNumericas()
   LOCAL nNumero := 123.456
   LOCAL nNegativo := -45.789
   
   // Funciones de redondeo
   ? "Round:", Round(nNumero, 2)                 // 123.46
   ? "Int:", Int(nNumero)                        // 123
   ? "Valor absoluto:", Abs(nNegativo)           // 45.789
   
   // Funciones matemáticas
   ? "Raíz cuadrada:", Sqrt(16)                  // 4
   ? "Logaritmo:", Log(100)                      // 4.605 (log natural)
   ? "Exponencial:", Exp(1)                      // 2.718
   
   // Funciones trigonométricas
   ? "Seno de π/2:", Sin(HB_PI/2)               // 1
   ? "Coseno de π:", Cos(HB_PI)                 // -1
   ? "Tangente de π/4:", Tan(HB_PI/4)           // 1
   
   // Funciones de conversión
   ? "String a número:", Val("123.45")           // 123.45
   ? "Número a string:", Str(nNumero, 10, 3)     // "   123.456"
   ? "Transform:", Transform(nNumero, "999.99")   // "123.46"
   
   RETURN NIL
```

#### Funciones de fecha y hora (Date/Time Functions)

```harbour
FUNCTION EjemploFechasHoras()
   LOCAL dHoy := Date()
   LOCAL dFecha := CToD("25/12/2023")
   LOCAL cHora := Time()
   
   // Funciones básicas de fecha
   ? "Fecha actual:", dHoy                       // 2023-12-15 (formato depende de SET DATE)
   ? "Año:", Year(dHoy)                         // 2023
   ? "Mes:", Month(dHoy)                        // 12
   ? "Día:", Day(dHoy)                          // 15
   ? "Día de semana:", DoW(dHoy)                // 6 (viernes)
   
   // Operaciones con fechas
   ? "Días entre fechas:", dHoy - dFecha        // -10 (diferencia en días)
   ? "Suma 30 días:", dHoy + 30                 // 2024-01-14
   
   // Funciones de hora
   ? "Hora actual:", cHora                      // "14:30:25"
   ? "Segundos desde medianoche:", Seconds()     // 52225.5
   
   // Formateo de fechas
   ? "Fecha formato US:", DToC(dHoy)            // "12/15/23"
   ? "Fecha ISO:", DToS(dHoy)                   // "20231215"
   
   // Fecha y hora combinadas
   LOCAL tDateTime := hb_DateTime()
   ? "DateTime:", hb_TToC(tDateTime)            // "2023-12-15 14:30:25.123"
   
   RETURN NIL
```

#### Funciones de arrays (Array Functions)

```harbour
FUNCTION EjemploFuncionesArrays()
   LOCAL aNumeros := {5, 2, 8, 1, 9, 3}
   LOCAL aNombres := {"Juan", "María", "Pedro", "Ana"}
   
   // Funciones básicas
   ? "Tamaño array:", Len(aNumeros)             // 6
   ? "Agregar elemento:", AAdd(aNombres, "Luis") // Retorna 5 (nueva longitud)
   ? "Insertar en pos 2:", AIns(aNombres, 2)   // Inserta en posición 2
   aNombres[2] := "Carlos"
   
   // Búsqueda
   ? "Buscar 'Pedro':", AScan(aNombres, "Pedro")  // 4
   ? "Buscar con bloque:", AScan(aNumeros, {|x| x > 5})  // 3 (primer elemento > 5)
   
   // Ordenamiento
   ASort(aNumeros)                              // Ordena en orden ascendente
   ? "Array ordenado:", aNumeros                // {1, 2, 3, 5, 8, 9}
   
   // Evaluación con bloques
   AEval(aNombres, {|nombre, i| QOut(i, ":", nombre)})  // Imprime cada elemento
   
   // Clonado
   LOCAL aCopia := AClone(aNumeros)
   ? "Array clonado:", aCopia
   
   RETURN NIL
```

#### Funciones de archivos (File Functions)

```harbour
FUNCTION EjemploFuncionesArchivos()
   LOCAL cNombreArchivo := "test.txt"
   LOCAL nHandle
   LOCAL cContenido := "Este es un archivo de prueba" + Chr(13) + Chr(10)
   
   // Verificar existencia
   IF File(cNombreArchivo)
      ? "El archivo", cNombreArchivo, "existe"
   ELSE
      ? "El archivo", cNombreArchivo, "no existe"
   ENDIF
   
   // Crear archivo
   nHandle := FCreate(cNombreArchivo)
   IF nHandle != -1
      FWrite(nHandle, cContenido)
      FClose(nHandle)
      ? "Archivo creado exitosamente"
   ENDIF
   
   // Información del archivo
   ? "Tamaño:", FileSize(cNombreArchivo), "bytes"
   ? "Fecha modificación:", FileDate(cNombreArchivo)
   ? "Hora modificación:", FileTime(cNombreArchivo)
   
   // Leer archivo completo
   LOCAL cTextoLeido := MemoRead(cNombreArchivo)
   ? "Contenido leído:", cTextoLeido
   
   // Directorio
   LOCAL aArchivos := Directory("*.txt")
   ? "Archivos .txt encontrados:", Len(aArchivos)
   
   // Limpiar
   FErase(cNombreArchivo)
   
   RETURN NIL
```

### Funciones de conversión de tipos

```harbour
FUNCTION EjemploConversiones()
   // Números a cadenas
   ? "Str():", Str(123.45, 8, 2)               // "  123.45"
   ? "LTrim(Str()):", LTrim(Str(123))          // "123"
   ? "Transform():", Transform(1234.56, "@E 9,999.99")  // "1,234.56"
   
   // Cadenas a números
   ? "Val():", Val("123.45ABC")                 // 123.45
   ? "Val() con espacios:", Val("  456  ")      // 456
   
   // Lógicos a cadenas
   ? "Lógico .T.:", iif(.T., "Verdadero", "Falso")  // "Verdadero"
   
   // Fechas a cadenas
   ? "DToC():", DToC(Date())                    // "12/15/23"
   ? "DToS():", DToS(Date())                    // "20231215"
   
   // Cadenas a fechas
   ? "CToD():", CToD("12/25/2023")             // 2023-12-25
   ? "SToD():", SToD("20231225")               // 2023-12-25
   
   // Códigos ASCII
   ? "Chr(65):", Chr(65)                        // "A"
   ? "Asc('A'):", Asc("A")                     // 65
   
   RETURN NIL
```

### Funciones de sistema (System Functions)

```harbour
FUNCTION EjemploFuncionesSistema()
   // Información del sistema
   ? "Sistema operativo:", OS()                 // "WINDOWS" o "LINUX"
   ? "Versión Harbour:", Version()              // "Harbour 3.2.0dev..."
   ? "Directorio actual:", CurDir()             // Directorio de trabajo actual
   
   // Variables de entorno
   ? "PATH:", GetEnv("PATH")                    // Variable PATH del sistema
   ? "HOME:", GetEnv("HOME")                    // Directorio home del usuario
   
   // Línea de comandos
   LOCAL aParametros := hb_AParams()
   LOCAL i
   ? "Parámetros de línea de comandos:"
   FOR i := 1 TO Len(aParametros)
      ? i, ":", aParametros[i]
   NEXT
   
   // Memoria
   ? "Memoria usada:", hb_gcMemUsed()           // Bytes de memoria usados
   ? "Memoria total:", hb_gcMemTotal()          // Total de memoria disponible
   
   RETURN NIL
```

### Funciones de entrada/salida (I/O Functions)

```harbour
FUNCTION EjemploEntradaSalida()
   // Salida básica
   QOut("Mensaje con QOut")                     // Imprime y nueva línea
   QQOut("Mensaje con QQOut")                   // Imprime sin nueva línea
   
   // Salida con posicionamiento
   @ 10, 20 SAY "Texto en fila 10, columna 20"
   @ Row() + 1, Col() SAY "En la siguiente línea"
   
   // Entrada de usuario
   LOCAL cNombre
   LOCAL nEdad
   
   @ 15, 10 SAY "Ingrese su nombre: " GET cNombre
   @ 16, 10 SAY "Ingrese su edad: " GET nEdad
   READ()  // Activar entrada de datos
   
   ? "Hola", cNombre, "tienes", nEdad, "años"
   
   // Entrada simple
   ACCEPT "Presione Enter para continuar" TO cNombre
   
   // Esperar tecla
   ? "Presione cualquier tecla..."
   Inkey(0)  // Esperar indefinidamente
   
   RETURN NIL
```

### Funciones de control de errores

```harbour
FUNCTION EjemploControlErrores()
   LOCAL bErrorAnterior := ErrorBlock({|e| MiManejadorError(e)})
   LOCAL xResultado
   
   BEGIN SEQUENCE
      // Operación que puede fallar
      xResultado := 10 / 0  // División por cero
      
   RECOVER
      ? "Se capturó un error"
      xResultado := 0
      
   END SEQUENCE
   
   ErrorBlock(bErrorAnterior)
   
   ? "Resultado:", xResultado
   
   RETURN NIL

FUNCTION MiManejadorError(oError)
   ? "Error:", oError:Description
   ? "Operación:", oError:Operation
   ? "Archivo:", oError:FileName
   ? "Línea:", oError:LineNumber
   
   // Decidir si continuar o abortar
   RETURN .F.  // Continuar con error default
```

### Funciones matemáticas avanzadas

```harbour
FUNCTION EjemploMatematicasAvanzadas()
   // Funciones trigonométricas inversas
   ? "ArcSin(1):", ASin(1) * 180 / HB_PI        // 90 grados
   ? "ArcCos(0):", ACos(0) * 180 / HB_PI        // 90 grados
   ? "ArcTan(1):", ATan(1) * 180 / HB_PI        // 45 grados
   
   // Funciones hiperbólicas
   ? "Seno hiperbólico:", Sinh(1)               // 1.175
   ? "Coseno hiperbólico:", Cosh(1)             // 1.543
   ? "Tangente hiperbólica:", Tanh(1)           // 0.762
   
   // Funciones de potencia y raíz
   ? "2 elevado a 8:", 2 ^ 8                    // 256
   ? "Raíz cúbica de 27:", 27 ^ (1/3)          // 3
   
   // Funciones de números aleatorios
   ? "Número aleatorio 0-1:", hb_Random()       // Entre 0 y 1
   ? "Entero aleatorio 1-100:", hb_RandomInt(100) + 1  // Entre 1 y 100
   
   RETURN NIL
```

### Organización de la biblioteca de tiempo de ejecución

La biblioteca está organizada en módulos funcionales:

- **RT_MAIN**: Funciones principales del sistema
- **RT_STRING**: Manipulación de cadenas
- **RT_NUMERIC**: Operaciones numéricas  
- **RT_DATE**: Manejo de fechas y horas
- **RT_ARRAY**: Operaciones con arrays
- **RT_FILE**: Acceso a archivos
- **RT_IO**: Entrada/salida
- **RT_ERROR**: Manejo de errores
- **RT_MATH**: Funciones matemáticas
- **RT_SYSTEM**: Funciones del sistema

Esta biblioteca proporciona todas las herramientas fundamentales necesarias para desarrollar aplicaciones completas en Harbour, manteniendo compatibilidad con estándares xBase mientras añade funcionalidades modernas y optimizaciones.