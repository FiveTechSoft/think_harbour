# Capítulo 8: Manejo de Errores y Depuración

Una parte fundamental del desarrollo de software robusto es la capacidad de anticipar, manejar y corregir errores. Harbour, como descendiente de Clipper, hereda un sistema de manejo de errores potente que ha sido modernizado con construcciones de lenguaje más estructuradas.

## Tipos de Errores

En el desarrollo con Harbour, podemos clasificar los errores en tres categorías principales:

### 1. Errores de Sintaxis

Estos errores ocurren en la fase de compilación. Son violaciones de las reglas gramaticales del lenguaje Harbour. El compilador de Harbour detectará estos errores y detendrá el proceso de compilación, informando del error y el número de línea aproximado donde se encuentra.

**Causas comunes:**
- Palabras clave mal escritas (`FUNCTON` en lugar de `FUNCTION`).
- Falta de paréntesis, comillas o bloques de cierre (`ENDIF`, `NEXT`, `END`).
- Uso incorrecto de operadores.

**Ejemplo:**
```harbour
// Error de sintaxis: falta el paréntesis de cierre en la llamada a la función
? SUBSTR("Hola Mundo"
```
El compilador emitirá un error y no generará un archivo de objeto (.obj) para este código.

### 2. Errores Lógicos

Estos son los errores más difíciles de detectar, ya que el programa compila y se ejecuta sin fallar, pero no produce el resultado esperado. El error reside en la lógica o el algoritmo implementado por el programador.

**Causas comunes:**
- Condiciones incorrectas en sentencias `IF` o `DO WHILE`.
- Cálculos matemáticos incorrectos.
- Flujo de programa mal diseñado que omite casos importantes.

**Ejemplo:**
```harbour
// Error lógico: el bucle se ejecutará 10 veces (de 1 a 10),
// pero podría esperarse que se ejecute 9 veces.
FOR i := 1 TO 10
   IF i > 9  // Esta condición solo es verdadera en la última iteración
      ? "Nueve"
   ENDIF
NEXT
```
El programa no falla, pero su comportamiento no es el correcto si la intención era otra. La depuración es la herramienta principal para encontrar estos errores.

### 3. Errores de Tiempo de Ejecución (Runtime Errors)

Estos errores ocurren mientras el programa está en ejecución. Son situaciones excepcionales que impiden que el programa continúe su flujo normal. Harbour tiene un sistema de errores para interceptar y manejar estos fallos.

**Causas comunes:**
- Intentar abrir un archivo que no existe.
- División por cero.
- Tipo de datos incorrecto en una operación (p.ej., sumar un número a una cadena).
- Índice fuera de los límites de un array.
- Llamar a una función o método que no existe.

Si no se manejan, estos errores suelen terminar la aplicación mostrando un mensaje de error en la pantalla.

## Manejo Estructurado de Errores: `BEGIN SEQUENCE`

La forma moderna y recomendada para manejar errores de tiempo de ejecución en Harbour es la construcción `BEGIN SEQUENCE ... RECOVER ... END SEQUENCE`. Permite escribir código "protegido" y definir un bloque de código específico para manejar cualquier error que ocurra dentro de él, similar a `try-catch` en otros lenguajes.

**Sintaxis:**
```harbour
BEGIN SEQUENCE
   // ... Código propenso a errores ...
   // Si ocurre un error aquí, el control salta al bloque RECOVER
[BREAK] // Sale de la secuencia sin error
RECOVER [USING oError]
   // ... Código para manejar el error ...
   // El objeto 'oError' contiene toda la información del error
END SEQUENCE
```

- **`RECOVER [USING oError]`**: Este bloque solo se ejecuta si ocurre un error en el bloque superior. La cláusula opcional `USING oError` captura los detalles del error en un objeto (el "objeto de error").
- **`BREAK`**: Se puede usar para salir de la secuencia prematuramente, evitando el bloque `RECOVER`.

**Ejemplo Práctico:**
```harbour
FUNCTION Dividir( nNum, nDen )
   LOCAL nResultado

   BEGIN SEQUENCE
      IF nDen == 0
         // Podemos generar un error personalizado
         HB_DefaultError():New( "Error de División", ;
            "El denominador no puede ser cero", ;
            NIL, NIL, NIL, NIL, NIL, NIL, NIL, 20 ):Throw()
      ENDIF
      nResultado := nNum / nDen
   RECOVER USING oError
      ? "Ocurrió un error al dividir:"
      ? "Descripción:", oError:Description
      ? "Operación:",   oError:Operation
      nResultado := NIL // Asignar un valor seguro
   END SEQUENCE

   RETURN nResultado
```

### El Objeto de Error

Cuando un error es capturado, Harbour crea un "objeto de error" que contiene información valiosa para diagnosticar el problema. Sus propiedades principales son:
- `oError:args` (Array): Argumentos de la operación que falló.
- `oError:canDefault` (Lógico): ¿Se puede reintentar la operación?
- `oError:canRetry` (Lógico): ¿Se puede reintentar la operación?
- `oError:canSubstitute` (Lógico): ¿Se puede sustituir un resultado?
- `oError:description` (Carácter): Descripción del error.
- `oError:filename` (Carácter): Nombre del archivo donde ocurrió el error.
- `oError:operation` (Carácter): Nombre de la función o la operación que falló.
- `oError:osCode` (Numérico): Código de error del sistema operativo.
- `oError:subCode` (Numérico): Código de error específico de Harbour.
- `oError:tries` (Numérico): Número de intentos de la operación.

## Manejo de Errores Tradicional: `ON ERROR`

Este es el método heredado de Clipper. `ON ERROR` permite designar un procedimiento global que será llamado automáticamente cuando ocurra un error de tiempo de ejecución en cualquier parte de la aplicación que no esté cubierta por un `BEGIN SEQUENCE`.

**Sintaxis:**
`ON ERROR <procedure_name> | <code_block>`

**Ejemplo:**
```harbour
PROCEDURE Main()
   // Establece la función 'ManejadorDeErrores' como el manejador global
   ON ERROR ErrorHandler( PCount(), ProcName( 1 ), ProcLine( 1 ) )

   // ... código de la aplicación ...
   ? 1 / 0  // Esto provocará un error y llamará a ErrorHandler

   RETURN

// Función que maneja el error
PROCEDURE ErrorHandler( nParametros, cProcName, nLine )
   ALERT( "Error en: " + cProcName + " en la línea " + LTrim( Str( nLine ) ) )
   QUIT // Termina el programa
   RETURN
```
Aunque funcional, `ON ERROR` es menos flexible que `BEGIN SEQUENCE` porque es global y no está estructurado, lo que puede dificultar el manejo de errores específicos en contextos locales.

## Depuración y Herramientas de Rastreo

La depuración es el proceso de encontrar y corregir errores lógicos. Harbour incluye un depurador muy útil.

### Uso del Depurador (Debugger)

Para usar el depurador, necesitas compilar tu programa con la opción `-b`.

`hbmk2 mi_programa.prg -b`

Una vez compilado, puedes activar el depurador en cualquier momento de la ejecución presionando `Alt+D`. También puedes iniciar el programa directamente en modo de depuración.

**Características principales del depurador:**
- **Ventana de Código Fuente:** Muestra el código y resalta la línea actual.
- **Ventana de Inspección (Watch):** Permite monitorear el valor de variables o expresiones.
- **Ventana de Llamadas (Call Stack):** Muestra la secuencia de funciones que han sido llamadas para llegar al punto actual.

**Comandos comunes del depurador:**
- `F8` (Step Over): Ejecuta la línea actual y pasa a la siguiente. Si la línea es una llamada a función, la ejecuta completamente sin entrar en ella.
- `F7` (Step Into): Ejecuta la línea actual. Si es una llamada a función, entra en la función para depurarla línea por línea.
- `F9` (Run): Ejecuta el programa hasta el siguiente punto de interrupción (breakpoint) o hasta que termine.
- `Ctrl+F8` (Run to Cursor): Ejecuta el programa hasta la línea donde está el cursor.
- `F4` (Go to Cursor): Similar a `Ctrl+F8`.
- `F5` (Zoom): Maximiza la ventana activa del depurador.

### Herramientas de Rastreo

A veces, en lugar de una depuración interactiva, es útil registrar el flujo del programa o el valor de las variables en un archivo de texto (log).

**Funciones útiles:**
- `OutStd( ... )`, `OutErr( ... )`: Escriben en la salida estándar o en la salida de error, que puede ser redirigida a un archivo.
- `HB_TraceLog()`: Una función potente de la librería `hbdebug` que puede escribir mensajes en un archivo de log con formato.

**Ejemplo de rastreo manual:**
```harbour
#include "fileio.ch"

PROCEDURE ProcesoComplejo( cDato )
   LOCAL hFile

   // Abre un log para esta función
   hFile := FCreate( "proceso_log.txt" )
   FWrite( hFile, "Iniciando ProcesoComplejo a las " + Time() + HB_EOL )
   FWrite( hFile, "Dato recibido: " + cDato + HB_EOL )

   // ... Lógica compleja ...
   IF Empty( cDato )
      FWrite( hFile, "ERROR: El dato está vacío." + HB_EOL )
   ENDIF

   FWrite( hFile, "Finalizando ProcesoComplejo." + HB_EOL )
   FClose( hFile )

   RETURN
```
Esta técnica es invaluable para depurar aplicaciones en servidores o en entornos donde no es posible ejecutar el depurador interactivo.
