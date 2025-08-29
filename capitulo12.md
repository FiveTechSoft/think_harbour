# Capítulo 12: Temas Avanzados

En este capítulo, exploraremos algunas de las capacidades más potentes y avanzadas de Harbour. Estos temas te permitirán crear aplicaciones más complejas, robustas y eficientes, desde servicios multihilo hasta aplicaciones web dinámicas y la integración con código nativo.

## 1. Programación Multihilo (Multithreading)

La programación multihilo permite que una aplicación ejecute varias tareas de forma concurrente, mejorando el rendimiento y la capacidad de respuesta, especialmente en sistemas con múltiples núcleos de procesador. Harbour ofrece un conjunto completo de funciones para la gestión de hilos y la sincronización.

### Creación de un Hilo

El corazón del multihilo en Harbour es la función `hb_threadStart()`. Esta función inicia un nuevo hilo de ejecución que correrá en paralelo al hilo principal.

**Sintaxis:**
`hb_threadStart( <@Function()|cFunctionName>, [ <Parameter1> ], [ ... ] ) --> hThread`

- `<@Function()|cFunctionName>`: Una referencia a la función de Harbour o el nombre de una función en C que se ejecutará en el nuevo hilo.
- `<ParameterN>`: Parámetros opcionales que se pasarán a la función del hilo.
- `hThread`: La función devuelve un manejador (handle) para el nuevo hilo, que puede ser usado para gestionarlo.

**Ejemplo Básico:**

```harbour
PROCEDURE Main()
   LOCAL hThread1, hThread2

   ? "Iniciando hilos..."

   // Iniciar un hilo que ejecuta la función Worker()
   hThread1 := hb_threadStart( @Worker(), "Hilo A", 5 )

   // Iniciar otro hilo
   hThread2 := hb_threadStart( @Worker(), "Hilo B", 3 )

   ? "Hilos iniciados. Esperando a que terminen."

   // Esperar a que ambos hilos finalicen su ejecución
   hb_threadJoin( hThread1 )
   hb_threadJoin( hThread2 )

   ? "Todos los hilos han terminado."
   RETURN

STATIC PROCEDURE Worker( cNombre, nRepeticiones )
   LOCAL i

   FOR i := 1 TO nRepeticiones
      // Imprimir el nombre del hilo y el número de iteración
      // hb_outStd() es seguro para usar en hilos (thread-safe)
      hb_outStd( cNombre + ": trabajando... " + LTrim( Str( i ) ) + hb_eol() )
      // Simular trabajo
      hb_idleSleep( 0.5 )
   NEXT

   hb_outStd( cNombre + ": finalizado." + hb_eol() )
   RETURN
```

### Sincronización de Hilos: Mutex

Cuando varios hilos acceden a un recurso compartido (como una variable global o un archivo), es crucial evitar condiciones de carrera. Los "mutex" (exclusión mutua) son el mecanismo más común para garantizar que solo un hilo a la vez pueda acceder a una sección crítica del código.

- `hb_mutexCreate()`: Crea un nuevo mutex.
- `hb_mutexLock( hMutex )`: Bloquea el mutex. Si ya está bloqueado por otro hilo, este hilo esperará.
- `hb_mutexUnlock( hMutex )`: Desbloquea el mutex, permitiendo que otros hilos lo adquieran.
- `hb_mutexDestroy( hMutex )`: Libera los recursos del mutex.

**Ejemplo con Mutex:**

```harbour
// Variable global compartida
STATIC nContador := 0
// Mutex para proteger el contador
STATIC hMutex

PROCEDURE Main()
   LOCAL aThreads := {}
   LOCAL i

   // Crear el mutex antes de iniciar los hilos
   hMutex := hb_mutexCreate()

   // Iniciar 10 hilos que incrementarán el contador
   FOR i := 1 TO 10
      AAdd( aThreads, hb_threadStart( @Incrementador() ) )
   NEXT

   // Esperar a que todos terminen
   FOR EACH hThread IN aThreads
      hb_threadJoin( hThread )
   NEXT

   // Destruir el mutex
   hb_mutexDestroy( hMutex )

   ? "Valor final del contador:", nContador // Debería ser 10000
   RETURN

STATIC PROCEDURE Incrementador()
   LOCAL i

   FOR i := 1 TO 1000
      // Sección crítica: solo un hilo puede entrar a la vez
      hb_mutexLock( hMutex )
      nContador++
      hb_mutexUnlock( hMutex )
   NEXT
   RETURN
```

## 2. Desarrollo de Aplicaciones Web con hbhttpd

Harbour incluye un servidor web integrado, `hbhttpd`, que permite desarrollar aplicaciones web y servicios RESTful directamente en Harbour. Se encuentra en el directorio `contrib/hbhttpd`.

### Estructura Básica de un Servidor Web

Para crear un servidor web, necesitas:
1.  Iniciar el servidor con `hb_httpServer()`.
2.  Proporcionar una función principal que procesará cada solicitud HTTP entrante.

**Ejemplo de "Hola Mundo" Web:**

```harbour
#include "hbhttpd.ch"

PROCEDURE Main()
   // Iniciar el servidor en el puerto 8080
   // La función HttpHandler() procesará cada solicitud
   hb_httpServer( 8080, @HttpHandler() )
   RETURN

// Función que maneja las solicitudes HTTP
FUNCTION HttpHandler( hConnection )
   LOCAL cMethod := hb_httpGetMethod( hConnection )
   LOCAL cPath   := hb_httpGetPath( hConnection )
   LOCAL cBody

   // Enviar la cabecera de respuesta (HTTP 200 OK, tipo de contenido HTML)
   hb_httpSendHeader( hConnection, 200, "Content-Type", "text/html; charset=utf-8" )

   // Construir el cuerpo de la respuesta HTML
   cBody := "<html><head><title>Servidor Harbour</title></head>"
   cBody += "<body>"
   cBody += "<h1>¡Hola desde hbhttpd!</h1>"
   cBody += "<p>Método de solicitud: " + cMethod + "</p>"
   cBody += "<p>Ruta solicitada: " + cPath + "</p>"
   cBody += "</body></html>"

   // Enviar el cuerpo de la respuesta al cliente
   hb_httpSendResponse( hConnection, cBody )

   // Devolver .T. para mantener la conexión si es keep-alive
   RETURN .T.
```

Para compilar, debes enlazar la librería `hbhttpd`. Por ejemplo, con `hbmk2`:
`hbmk2 mi_servidor.prg hbhttpd.hbc`

### Procesando Datos de Formularios y JSON

`hbhttpd` facilita el acceso a los datos enviados por el cliente, como parámetros GET o datos POST.

- `hb_httpGetParam( hConnection, cParamName, [ xDefault ] )`: Obtiene un parámetro de la URL (GET) o del cuerpo (POST `application/x-www-form-urlencoded`).
- `hb_httpGetBody( hConnection )`: Obtiene el cuerpo crudo de la solicitud, útil para procesar JSON o XML.

**Ejemplo de API REST (JSON):**

```harbour
#include "hbhttpd.ch"
#include "hbjson.ch"

PROCEDURE Main()
   hb_httpServer( 8080, @ApiHandler() )
   RETURN

FUNCTION ApiHandler( hConnection )
   LOCAL cPath := hb_httpGetPath( hConnection )
   LOCAL cJson

   hb_httpSendHeader( hConnection, 200, "Content-Type", "application/json" )

   IF cPath == "/api/usuario/1"
      // Crear un objeto Harbour y convertirlo a JSON
      cJson := hb_jsonEncode( { "id" => 1, "nombre" => "Juan Perez", "activo" => .T. } )
      hb_httpSendResponse( hConnection, cJson )
   ELSE
      // Devolver un error 404
      hb_httpSendHeader( hConnection, 404, "Content-Type", "application/json" )
      hb_httpSendResponse( hConnection, hb_jsonEncode( { "error" => "No encontrado" } ) )
   ENDIF

   RETURN .T.
```

## 3. Uso del Garbage Collector (GC) de Harbour

Harbour gestiona la memoria automáticamente a través de un Recolector de Basura (Garbage Collector o GC). El GC se encarga de liberar la memoria de los "items" (cadenas, arrays, objetos, etc.) que ya no están en uso.

### ¿Cómo funciona?

El GC de Harbour utiliza un algoritmo de "marcar y barrer" (mark-and-sweep). Periódicamente, o cuando se necesita más memoria, el GC realiza los siguientes pasos:
1.  **Fase de Marca:** Comienza desde las raíces (variables locales, estáticas, públicas) y recorre todos los objetos a los que se puede acceder. Cada objeto alcanzable se "marca" como en uso.
2.  **Fase de Barrido:** El GC barre toda la memoria, y cualquier objeto que no fue marcado en la fase anterior se considera "basura" y se libera.

Esto significa que, en general, no necesitas preocuparte por liberar memoria manualmente.

### Funciones Relevantes del GC

Aunque el GC es automático, a veces es útil interactuar con él.

- `hb_gcCollect()`: Fuerza la ejecución de un ciclo completo de recolección de basura. Es raramente necesario, pero puede ser útil en situaciones de depuración o antes de operaciones que consumen mucha memoria.
- `hb_gcInfo( <nInfoType> )`: Proporciona información sobre el estado del GC y el uso de memoria. Por ejemplo, `hb_gcInfo( HB_GC_INFO_MEMORY_USED )` devuelve la cantidad de memoria actualmente en uso por los items de Harbour.

### Consideraciones sobre Referencias Circulares

Una de las ventajas del GC de Harbour es que puede manejar **referencias circulares**. Por ejemplo:

```harbour
oA := MyObject():New()
oB := MyObject():New()

oA:oRef := oB // oA hace referencia a oB
oB:oRef := oA // oB hace referencia a oA

// En este punto, oA y oB se referencian mutuamente.
// Si no hay otras referencias a ellos, se vuelven inalcanzables.

oA := NIL
oB := NIL

// El GC de Harbour detectará que este ciclo ya no es accesible
// desde ninguna raíz y liberará la memoria de ambos objetos.
```

En sistemas más antiguos basados en conteo de referencias, este escenario crearía una fuga de memoria. El GC de Harbour lo resuelve sin problemas.

## 4. Integración de Código C y Librerías Externas

Una de las características más potentes de Harbour es su capacidad para extenderse con código escrito en C/C++. Esto te permite:
-   Crear funciones de alto rendimiento.
-   Acceder a APIs del sistema operativo.
-   Integrar librerías de terceros.

### Escribiendo una Función en C para Harbour

Para que una función en C sea visible desde Harbour, debes usar la API de extensión de Harbour.

1.  **Incluir `hbapi.h`**: Este encabezado contiene todas las definiciones necesarias.
2.  **Usar la macro `HB_FUNC( nombre_funcion )`**: Así es como declaras una función que será llamada desde Harbour. El `nombre_funcion` será el que uses en tu código `.prg`.
3.  **Interactuar con los parámetros**: Usa `hb_pcount()` para obtener el número de parámetros y `hb_param(n, <type_check>)` para acceder a ellos.
4.  **Devolver un valor**: Usa las funciones `hb_ret*()` (ej. `hb_retc()` para un string, `hb_retni()` para un entero) para devolver un valor a Harbour.

**Ejemplo: Una función en C que suma dos números**

`mis_funciones.c`:
```c
#include "hbapi.h"
#include "hbapierr.h"
#include "hbvm.h"

// Definimos una función llamada SUMA_C
HB_FUNC( SUMA_C )
{
   // Comprobar que se pasaron dos parámetros numéricos
   if( hb_pcount() == 2 && HB_ISNUM( 1 ) && HB_ISNUM( 2 ) )
   {
      // Obtener los parámetros como dobles
      double dNum1 = hb_parnd( 1 );
      double dNum2 = hb_parnd( 2 );

      // Devolver la suma
      hb_retnd( dNum1 + dNum2 );
   }
   else
   {
      // Devolver NIL si los parámetros son incorrectos
      hb_ret();
   }
}
```

### Compilando y Usando la Función

1.  **Compila tu archivo C** como un objeto (`.o` o `.obj`).
2.  **Enlaza el objeto** con tu aplicación Harbour.

Con `hbmk2`, este proceso es muy sencillo. Simplemente añade el archivo `.c` a la línea de comandos:

`hbmk2 mi_app.prg mis_funciones.c`

`hbmk2` se encargará de compilar el C y enlazarlo correctamente.

**Usando la función en Harbour:**

`mi_app.prg`:
```harbour
PROCEDURE Main()
   LOCAL nResultado

   // Llamar a la función C como si fuera una función de Harbour
   nResultado := Suma_C( 10, 32 )

   ? "El resultado de la suma en C es:", nResultado // 42

   // Ejemplo de llamada incorrecta
   ? "Llamada incorrecta:", Suma_C( "hola" ) // Devolverá NIL
   RETURN
```

Esta capacidad de integración abre un mundo de posibilidades, permitiendo combinar la alta productividad de Harbour con el rendimiento y el acceso a bajo nivel del lenguaje C.
