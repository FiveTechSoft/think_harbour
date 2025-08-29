# Capítulo 20: La biblioteca de tiempo de ejecución (`The runtime library`)

La biblioteca de tiempo de ejecución de Harbour es una colección integral de funciones, procedimientos y clases que proporcionan la funcionalidad fundamental para el desarrollo de aplicaciones. Esta biblioteca incluye desde operaciones básicas de manipulación de datos hasta funciones avanzadas de conectividad y procesamiento.

---

## Funciones estándar integradas

### Funciones de manipulación de cadenas

Harbour incluye un conjunto robusto de funciones para el manejo de texto:

```harbour
FUNCTION EjemplosCadenas()
   LOCAL cTexto := "  Harbour Programming Language  "
   LOCAL cResultado
   
   // Limpieza y formato
   ? "Original:", "[" + cTexto + "]"
   ? "Trim:", "[" + Trim(cTexto) + "]"
   ? "LTrim:", "[" + LTrim(cTexto) + "]"
   ? "RTrim:", "[" + RTrim(cTexto) + "]"
   ? "AllTrim:", "[" + AllTrim(cTexto) + "]"
   
   // Transformaciones
   ? "Upper:", Upper(cTexto)
   ? "Lower:", Lower(cTexto)
   ? "Proper:", Proper(cTexto)
   
   // Búsqueda y reemplazo
   cTexto := "Harbour Programming Language"
   ? "At('Program'):", At("Program", cTexto)
   ? "RAt('a'):", RAt("a", cTexto)
   ? "StrTran:", StrTran(cTexto, "Programming", "Development")
   
   // Subcadenas
   ? "Left(10):", Left(cTexto, 10)
   ? "Right(8):", Right(cTexto, 8)
   ? "SubStr(9,11):", SubStr(cTexto, 9, 11)
   
   // Comparaciones
   ? "Like pattern:", "Harbour" $ cTexto
   ? "Starts with H:", Left(cTexto, 1) == "H"
   
   RETURN NIL

// Funciones avanzadas de cadenas
FUNCTION CadenasAvanzadas()
   LOCAL aTokens, cCadena, i
   
   // Tokenización
   cCadena := "uno,dos,tres,cuatro,cinco"
   aTokens := StrTokenize(cCadena, ",")
   
   ? "Tokens encontrados:", Len(aTokens)
   FOR i := 1 TO Len(aTokens)
      ? i, ":", aTokens[i]
   NEXT
   
   // Formateo numérico en cadenas
   LOCAL nNumero := 1234567.89
   ? "StrZero:", StrZero(nNumero, 12, 2)
   ? "Transform:", Transform(nNumero, "999,999.99")
   ? "PadL:", PadL(LTrim(Str(nNumero)), 15, "*")
   ? "PadR:", PadR("Texto", 20, ".")
   ? "PadC:", PadC("Centro", 20, "-")
   
   // Verificaciones de contenido
   ? "IsAlpha:", IsAlpha("ABC")
   ? "IsDigit:", IsDigit("123")
   ? "IsAlNum:", IsAlNum("ABC123")
   ? "IsSpace:", IsSpace("   ")
   
   RETURN NIL
```

### Funciones numéricas y matemáticas

```harbour
FUNCTION EjemplosNumericos()
   LOCAL nNum := -123.456
   
   // Funciones básicas
   ? "Abs:", Abs(nNum)
   ? "Int:", Int(nNum)
   ? "Round:", Round(nNum, 2)
   ? "Ceiling:", Ceiling(nNum)
   ? "Floor:", Floor(nNum)
   
   // Funciones trigonométricas
   LOCAL nAngulo := 45
   LOCAL nRadianes := nAngulo * (Pi() / 180)
   
   ? "Sin(45°):", Sin(nRadianes)
   ? "Cos(45°):", Cos(nRadianes)
   ? "Tan(45°):", Tan(nRadianes)
   ? "ATan(1):", ATan(1) * (180 / Pi())  // Debería ser 45°
   
   // Funciones exponenciales y logarítmicas
   ? "Sqrt(16):", Sqrt(16)
   ? "Power(2,8):", Power(2, 8)
   ? "Exp(1):", Exp(1)  // e^1
   ? "Log(Exp(1)):", Log(Exp(1))  // Logaritmo natural
   
   // Funciones estadísticas básicas
   LOCAL aNumeros := {10, 20, 30, 40, 50}
   ? "Min:", Min(aNumeros)
   ? "Max:", Max(aNumeros)
   ? "Sum:", Sum(aNumeros)
   ? "Average:", Average(aNumeros)
   
   RETURN NIL

// Funciones avanzadas para cálculos financieros
FUNCTION CalculosFinancieros()
   LOCAL nCapital := 100000    // Capital inicial
   LOCAL nTasa := 0.05         // 5% anual
   LOCAL nPeriodos := 12       // 12 meses
   
   // Interés compuesto
   LOCAL nMontoFinal := nCapital * Power(1 + nTasa/12, nPeriodos)
   ? "Monto final (interés compuesto):", Transform(nMontoFinal, "999,999.99")
   
   // Valor presente
   LOCAL nValorFuturo := 110000
   LOCAL nValorPresente := nValorFuturo / Power(1 + nTasa, 1)
   ? "Valor presente:", Transform(nValorPresente, "999,999.99")
   
   // Cálculo de cuotas (aproximado)
   LOCAL nCuota := (nCapital * nTasa/12) / (1 - Power(1 + nTasa/12, -nPeriodos))
   ? "Cuota mensual:", Transform(nCuota, "999,999.99")
   
   RETURN NIL
```

### Funciones de fecha y hora

```harbour
FUNCTION EjemplosFechas()
   LOCAL dHoy := Date()
   LOCAL tAhora := hb_DateTime()
   
   // Información básica
   ? "Fecha actual:", DToC(dHoy)
   ? "Día de la semana:", DOW(dHoy), CDoW(dHoy)
   ? "Día del mes:", Day(dHoy)
   ? "Mes:", Month(dHoy), CMonth(dHoy)
   ? "Año:", Year(dHoy)
   
   // Manipulación de fechas
   ? "Primer día del mes:", dHoy - Day(dHoy) + 1
   ? "Último día del mes:", dHoy + (45 - Day(dHoy + 45))
   ? "Hace una semana:", dHoy - 7
   ? "En 30 días:", dHoy + 30
   
   // Cálculos de edad
   LOCAL dNacimiento := CToD("15/06/1990")
   LOCAL nEdad := Int((dHoy - dNacimiento) / 365.25)
   ? "Edad calculada:", nEdad, "años"
   
   // Fechas en diferentes formatos
   ? "ISO format:", Transform(dHoy, "@D")
   ? "US format:", Transform(dHoy, "99/99/9999")
   ? "European:", Transform(dHoy, "99.99.9999")
   
   RETURN NIL

FUNCTION EjemplosDateTime()
   LOCAL tTimestamp := hb_DateTime()
   LOCAL cTimeStamp
   
   // Componentes de fecha/hora
   ? "Fecha completa:", hb_TToD(tTimestamp)
   ? "Solo hora:", hb_TToT(tTimestamp)
   ? "Año:", hb_Year(tTimestamp)
   ? "Mes:", hb_Month(tTimestamp)
   ? "Día:", hb_Day(tTimestamp)
   ? "Hora:", hb_Hour(tTimestamp)
   ? "Minuto:", hb_Minute(tTimestamp)
   ? "Segundo:", hb_Sec(tTimestamp)
   ? "Milisegundo:", hb_MilliSec(tTimestamp)
   
   // Formateo de timestamp
   cTimeStamp := hb_TToC(tTimestamp, "YYYY-MM-DD", "HH:MM:SS.FFF")
   ? "Formato ISO:", cTimeStamp
   
   // Operaciones con timestamps
   LOCAL tFuturo := hb_DateTime() + 1.5  // 1.5 días en el futuro
   ? "Diferencia en días:", tFuturo - tTimestamp
   ? "Diferencia en horas:", (tFuturo - tTimestamp) * 24
   
   RETURN NIL
```

### Funciones de conversión de tipos

```harbour
FUNCTION EjemplosConversion()
   LOCAL xValor
   
   // Conversiones numéricas
   ? "Val('123.45'):", Val("123.45")
   ? "Val('  99  '):", Val("  99  ")
   ? "Val('ABC123'):", Val("ABC123")  // Retorna 0
   
   // Conversiones a cadena
   ? "Str(123):", "[" + Str(123) + "]"
   ? "Str(123, 8):", "[" + Str(123, 8) + "]"
   ? "Str(123.45, 8, 2):", "[" + Str(123.45, 8, 2) + "]"
   ? "LTrim(Str(123)):", "[" + LTrim(Str(123)) + "]"
   
   // Conversiones de fecha
   ? "DToS(Date()):", DToS(Date())
   ? "SToD('20240315'):", SToD("20240315")
   ? "CToD('15/03/2024'):", CToD("15/03/2024")
   ? "DToC(Date()):", DToC(Date())
   
   // Conversiones lógicas
   ? "ValType(.T.):", ValType(.T.)
   ? "ValType(.F.):", ValType(.F.)
   
   // Conversión universal ValToChar
   xValor := {1, "dos", .T., Date(), NIL}
   LOCAL i
   FOR i := 1 TO Len(xValor)
      ? i, ":", ValToChar(xValor[i])
   NEXT
   
   RETURN NIL
```

---

## Funciones de arrays y hashes

### Manipulación de arrays

```harbour
FUNCTION EjemplosArrays()
   LOCAL aOriginal := {10, 20, 30, 40, 50}
   LOCAL aNuevo, i
   
   // Información básica
   ? "Longitud:", Len(aOriginal)
   ? "Primer elemento:", aOriginal[1]
   ? "Último elemento:", ATail(aOriginal)
   
   // Modificación de tamaño
   aNuevo := Array(10)
   AFill(aNuevo, 0)
   ? "Array lleno de ceros:", ValToChar(aNuevo)
   
   // Copiar arrays
   aNuevo := ACopy(aOriginal, aNuevo, 1, Len(aOriginal), 1)
   ? "Array copiado:", ValToChar(aNuevo)
   
   // Inserción y eliminación
   AIns(aNuevo, 3)          // Insertar en posición 3
   aNuevo[3] := 25          // Asignar valor
   ? "Después de insertar:", ValToChar(aNuevo)
   
   ADel(aNuevo, 7)          // Eliminar posición 7
   ASize(aNuevo, Len(aNuevo) - 1)  // Reducir tamaño
   ? "Después de eliminar:", ValToChar(aNuevo)
   
   // Búsqueda
   ? "AScan(30):", AScan(aNuevo, 30)
   ? "AScan(99):", AScan(aNuevo, 99)  // No encontrado
   
   // Ordenamiento
   LOCAL aTextos := {"Zebra", "Alpha", "Beta", "Gamma"}
   ASort(aTextos)
   ? "Ordenado:", ValToChar(aTextos)
   
   RETURN NIL

FUNCTION ArraysAvanzados()
   LOCAL aMatrix := Array(3, 4)  // Matriz 3x4
   LOCAL i, j
   
   // Llenar matriz
   FOR i := 1 TO 3
      FOR j := 1 TO 4
         aMatrix[i][j] := i * 10 + j
      NEXT
   NEXT
   
   // Mostrar matriz
   ? "Matriz 3x4:"
   FOR i := 1 TO 3
      FOR j := 1 TO 4
         ?? Str(aMatrix[i][j], 4)
      NEXT
      ?
   NEXT
   
   // Evaluación en arrays
   LOCAL aNumeros := {1, 2, 3, 4, 5}
   LOCAL aCuadrados := Array(Len(aNumeros))
   
   AEval(aNumeros, {|n, i| aCuadrados[i] := n * n})
   ? "Números:", ValToChar(aNumeros)
   ? "Cuadrados:", ValToChar(aCuadrados)
   
   // Filtrado con AEval
   LOCAL aPares := {}
   AEval(aNumeros, {|n| iif(n % 2 == 0, AAdd(aPares, n), NIL)})
   ? "Números pares:", ValToChar(aPares)
   
   RETURN NIL
```

### Funciones de hash

```harbour
FUNCTION EjemplosHashes()
   LOCAL hPersona := {=>}  // Hash vacío
   LOCAL aClaves, aValores, i
   
   // Agregar elementos
   hPersona["nombre"] := "Juan Pérez"
   hPersona["edad"] := 30
   hPersona["ciudad"] := "Madrid"
   hPersona["activo"] := .T.
   hPersona["salario"] := 45000.50
   
   ? "Hash creado con", Len(hPersona), "elementos"
   
   // Acceso a elementos
   ? "Nombre:", hPersona["nombre"]
   ? "Edad:", hPersona["edad"]
   
   // Verificar existencia de claves
   ? "¿Existe 'telefono'?:", "telefono" $ hPersona
   ? "¿Existe 'edad'?:", "edad" $ hPersona
   
   // Obtener claves y valores
   aClaves := hb_HKeys(hPersona)
   aValores := hb_HValues(hPersona)
   
   ? "Claves:", ValToChar(aClaves)
   ? "Valores:", ValToChar(aValores)
   
   // Iterar sobre el hash
   ? "Contenido completo del hash:"
   FOR i := 1 TO Len(aClaves)
      ? "  ", aClaves[i], "=>", ValToChar(aValores[i])
   NEXT
   
   // Eliminar elemento
   hb_HDel(hPersona, "salario")
   ? "Después de eliminar 'salario':", Len(hPersona), "elementos"
   
   RETURN NIL

FUNCTION HashesAvanzados()
   // Hash con diferentes tipos de claves
   LOCAL hMixto := {=>}
   
   hMixto[1] := "Clave numérica"
   hMixto["texto"] := "Clave de texto"
   hMixto[Date()] := "Clave de fecha"
   hMixto[.T.] := "Clave lógica"
   
   // Hash anidado
   LOCAL hEmpresa := {=>}
   hEmpresa["nombre"] := "Tech Corp"
   hEmpresa["empleados"] := {=>}
   hEmpresa["empleados"]["juan"] := {"edad" => 30, "dept" => "IT"}
   hEmpresa["empleados"]["maria"] := {"edad" => 28, "dept" => "HR"}
   
   ? "Empleado Juan, edad:", hEmpresa["empleados"]["juan"]["edad"]
   
   // Combinar hashes
   LOCAL hConfigBase := {"debug" => .F., "timeout" => 30}
   LOCAL hConfigLocal := {"debug" => .T., "host" => "localhost"}
   LOCAL hConfigFinal := hb_HMerge(hConfigBase, hConfigLocal)
   
   ? "Configuración final:", ValToChar(hb_HKeys(hConfigFinal))
   
   RETURN NIL
```

---

## Funciones de E/S de archivos

### Manipulación de archivos de bajo nivel

```harbour
FUNCTION EjemplosArchivos()
   LOCAL hFile, cBuffer, nBytes
   LOCAL cNombreArchivo := "test_file.txt"
   LOCAL cContenido := "Línea 1" + hb_eol() + ;
                       "Línea 2" + hb_eol() + ;
                       "Línea 3" + hb_eol()
   
   // Crear y escribir archivo
   hFile := FCreate(cNombreArchivo)
   IF hFile >= 0
      nBytes := FWrite(hFile, cContenido)
      ? "Bytes escritos:", nBytes
      FClose(hFile)
   ELSE
      ? "Error creando archivo:", FError()
      RETURN NIL
   ENDIF
   
   // Leer archivo
   hFile := FOpen(cNombreArchivo, FO_READ)
   IF hFile >= 0
      cBuffer := Space(1024)
      nBytes := FRead(hFile, @cBuffer, Len(cBuffer))
      cBuffer := Left(cBuffer, nBytes)
      ? "Contenido leído:"
      ? cBuffer
      FClose(hFile)
   ELSE
      ? "Error abriendo archivo para lectura"
   ENDIF
   
   // Información del archivo
   ? "Tamaño del archivo:", FSeek(hFile, 0, FS_END)
   ? "¿Existe el archivo?:", File(cNombreArchivo)
   ? "Fecha del archivo:", FileDate(cNombreArchivo)
   ? "Hora del archivo:", FileTime(cNombreArchivo)
   
   // Limpiar
   FErase(cNombreArchivo)
   
   RETURN NIL

FUNCTION ArchivosAltoNivel()
   LOCAL cArchivo := "memo_test.txt"
   LOCAL cContenido := "Este es un archivo de prueba" + hb_eol() + ;
                       "con múltiples líneas" + hb_eol() + ;
                       "para demostrar las funciones de alto nivel."
   
   // Escribir archivo completo
   MemoWrit(cArchivo, cContenido)
   ? "Archivo escrito:", cArchivo
   
   // Leer archivo completo
   LOCAL cLeido := MemoRead(cArchivo)
   ? "Contenido leído:"
   ? cLeido
   
   // Contar líneas
   LOCAL aLineas := StrTokenize(cLeido, hb_eol())
   ? "Número de líneas:", Len(aLineas)
   
   // Procesar línea por línea
   LOCAL i
   FOR i := 1 TO Len(aLineas)
      ? "Línea", i, ":", aLineas[i]
   NEXT
   
   // Información adicional
   ? "Tamaño en bytes:", Len(cLeido)
   ? "Palabras aproximadas:", Len(StrTokenize(cLeido, " " + hb_eol()))
   
   // Limpiar
   FErase(cArchivo)
   
   RETURN NIL
```

### Manipulación de directorios

```harbour
FUNCTION EjemplosDirectorios()
   LOCAL aArchivos, i
   LOCAL cDirTrabajo := CurDir()
   
   ? "Directorio actual:", cDirTrabajo
   
   // Cambiar directorio
   IF !DirChange("temp")
      ? "Creando directorio 'temp'..."
      MakeDir("temp")
      DirChange("temp")
   ENDIF
   
   ? "Nuevo directorio:", CurDir()
   
   // Crear algunos archivos de prueba
   MemoWrit("file1.txt", "Contenido 1")
   MemoWrit("file2.log", "Contenido 2")
   MemoWrit("data.csv", "col1,col2,col3")
   
   // Listar archivos
   aArchivos := Directory("*.*")
   ? "Archivos en el directorio:"
   FOR i := 1 TO Len(aArchivos)
      ? "  ", aArchivos[i][F_NAME], ;
            Str(aArchivos[i][F_SIZE], 8), ;
            DToC(aArchivos[i][F_DATE]), ;
            aArchivos[i][F_TIME]
   NEXT
   
   // Buscar archivos específicos
   aArchivos := Directory("*.txt")
   ? "Archivos .txt encontrados:", Len(aArchivos)
   
   // Volver al directorio original
   DirChange("..")
   
   // Limpiar archivos de prueba
   FErase("temp/file1.txt")
   FErase("temp/file2.log")
   FErase("temp/data.csv")
   RemoveDir("temp")
   
   RETURN NIL
```

---

## Funciones del sistema

### Información del entorno

```harbour
FUNCTION InformacionSistema()
   ? "=== INFORMACIÓN DEL SISTEMA ==="
   ? "Sistema operativo:", OS()
   ? "Versión de Harbour:", Version()
   ? "Directorio actual:", CurDir()
   ? "Fecha del sistema:", Date()
   ? "Hora del sistema:", Time()
   
   // Variables de entorno
   ? "PATH:", GetEnv("PATH")
   ? "USER:", GetEnv("USER")
   ? "HOME:", GetEnv("HOME")
   ? "TEMP:", GetEnv("TEMP")
   
   // Información de memoria
   ? "Memoria utilizada:", hb_MemoryUsed(), "bytes"
   ? "Memoria máxima:", hb_MemoryMax(), "bytes"
   
   // Información del procesador
   ? "Número de CPUs:", hb_NumCPU()
   
   RETURN NIL

FUNCTION EjecutarComandos()
   LOCAL nExitCode, cComando
   
   // Ejecutar comando del sistema
   #ifdef __PLATFORM__WINDOWS
      cComando := "dir"
   #else
      cComando := "ls -la"
   #endif
   
   ? "Ejecutando comando:", cComando
   nExitCode := hb_Run(cComando)
   ? "Código de salida:", nExitCode
   
   // Ejecutar comando y capturar salida
   LOCAL cSalida := ""
   nExitCode := hb_ProcessRun(cComando, NIL, @cSalida)
   ? "Salida capturada:"
   ? cSalida
   
   RETURN NIL
```

### Funciones de tiempo y temporización

```harbour
FUNCTION EjemplosTiempo()
   LOCAL nInicio, nFin, nTiempoTranscurrido
   
   ? "=== MEDICIÓN DE TIEMPO ==="
   
   // Medir tiempo de ejecución
   nInicio := Seconds()
   
   // Simular trabajo
   LOCAL i, nResultado := 0
   FOR i := 1 TO 1000000
      nResultado += i
   NEXT
   
   nFin := Seconds()
   nTiempoTranscurrido := nFin - nInicio
   
   ? "Resultado del cálculo:", nResultado
   ? "Tiempo transcurrido:", nTiempoTranscurrido, "segundos"
   
   // Tiempo de alta precisión
   LOCAL nInicioHP := hb_MilliSeconds()
   
   // Otra operación
   LOCAL aGrande := Array(100000)
   AFill(aGrande, Random(1000))
   ASort(aGrande)
   
   LOCAL nFinHP := hb_MilliSeconds()
   ? "Tiempo de ordenamiento:", nFinHP - nInicioHP, "milisegundos"
   
   // Pausas programadas
   ? "Esperando 2 segundos..."
   hb_IdleSleep(2000)  // 2000 milisegundos
   ? "Continuando..."
   
   RETURN NIL
```

La biblioteca de tiempo de ejecución de Harbour es extremadamente rica y potente, proporcionando todas las herramientas necesarias para el desarrollo de aplicaciones profesionales. Estas funciones están optimizadas para rendimiento y son compatibles entre diferentes plataformas, lo que hace que el código sea portable y confiable.