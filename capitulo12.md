# Capítulo 12: Temas avanzados

Este capítulo explora características avanzadas de Harbour que permiten desarrollar aplicaciones complejas y modernas, incluyendo programación multihilo, desarrollo web y integración con código nativo.

---

## Programación multihilo

Harbour proporciona soporte para programación multihilo, permitiendo que las aplicaciones ejecuten múltiples tareas simultáneamente.

### Conceptos básicos

La programación multihilo permite:

- **Paralelización**: Ejecutar tareas independientes simultáneamente
- **Mejora de rendimiento**: Aprovechar múltiples núcleos del procesador
- **Responsividad**: Mantener la interfaz de usuario responsive durante operaciones largas
- **Procesamiento asíncrono**: Manejar múltiples operaciones de E/S concurrentemente

### Creación y gestión de hilos

```harbour
// Ejemplo básico de creación de hilo
FUNCTION IniciarHilo()
   LOCAL pHilo
   
   // Crear un nuevo hilo
   pHilo := hb_threadStart( @TareaEnSegundoPlano() )
   
   // Esperar a que termine el hilo
   hb_threadJoin( pHilo )
   
   RETURN NIL

FUNCTION TareaEnSegundoPlano()
   LOCAL i
   
   FOR i := 1 TO 10
      ? "Procesando item:", i
      hb_idleSleep( 1 ) // Simular trabajo
   NEXT
   
   RETURN NIL
```

### Sincronización y comunicación

- **Mutexes**: Para proteger recursos compartidos
- **Variables de condición**: Para coordinar entre hilos
- **Canales de comunicación**: Para intercambio de datos seguro

---

## Desarrollo de aplicaciones web con `hbhttpd`

`hbhttpd` es el servidor HTTP embebido de Harbour que permite crear aplicaciones web completas.

### Características del servidor

- **Servidor HTTP completo**: Maneja requests HTTP/HTTPS
- **Embebido**: Se integra directamente en aplicaciones Harbour
- **Configurable**: Soporte para múltiples puertos y dominios virtuales
- **Extensible**: Permite handlers personalizados

### Ejemplo básico de servidor web

```harbour
#include "hbhttpd.ch"

FUNCTION Main()
   LOCAL pServer
   
   // Crear servidor HTTP
   pServer := uhttpd_New()
   
   // Configurar puerto
   uhttpd_SetPort( pServer, 8080 )
   
   // Definir handler para ruta
   uhttpd_AddHandler( pServer, "/", @HandleRoot() )
   uhttpd_AddHandler( pServer, "/api/*", @HandleAPI() )
   
   // Iniciar servidor
   uhttpd_Start( pServer )
   
   ? "Servidor iniciado en puerto 8080"
   ? "Presione cualquier tecla para detener..."
   Inkey(0)
   
   // Detener servidor
   uhttpd_Stop( pServer )
   
   RETURN NIL

FUNCTION HandleRoot( cRequest, hHeaders )
   LOCAL cResponse := "<h1>¡Hola desde Harbour!</h1>"
   RETURN cResponse

FUNCTION HandleAPI( cRequest, hHeaders )
   LOCAL cResponse := '{"mensaje": "API funcionando", "timestamp": "' + DToS(Date()) + '"}'
   RETURN cResponse
```

### Características avanzadas

- **Templates**: Sistema de plantillas para generar HTML dinámico
- **Sesiones**: Manejo de sesiones de usuario
- **Middleware**: Procesamiento de requests en capas
- **Autenticación**: Sistemas de login y autorización

---

## Uso del Garbage Collector de Harbour

El Garbage Collector (GC) de Harbour maneja automáticamente la liberación de memoria, pero se puede controlar para optimizar el rendimiento.

### Funcionamiento automático

- **Detección automática**: Identifica objetos que ya no están siendo referenciados
- **Liberación segura**: Libera memoria de manera segura sin afectar objetos activos
- **Ciclos de referencia**: Detecta y resuelve referencias circulares

### Control manual del GC

```harbour
// Forzar ejecución del garbage collector
hb_gcAll()

// Configurar frecuencia del GC
hb_gcSetAuto( .T. )

// Obtener estadísticas del GC
LOCAL nUsedMemory := hb_gcMemUsed()
LOCAL nTotalMemory := hb_gcMemTotal()

? "Memoria usada:", nUsedMemory
? "Memoria total:", nTotalMemory
```

### Optimización de memoria

- **Gestión de objetos grandes**: Estrategias para manejar objetos que consumen mucha memoria
- **Pooling de objetos**: Reutilización de objetos para reducir presión en el GC
- **Monitoreo**: Herramientas para detectar memory leaks

---

## Integración de código C y librerías externas

Harbour permite integrar código C nativo y utilizar librerías externas para extender funcionalidad.

### El sistema Extend

```c
// Ejemplo de función C que se puede llamar desde Harbour
#include "hbapi.h"

HB_FUNC( MI_FUNCION_C )
{
   int numero = hb_parni(1);  // Obtener parámetro entero
   char* texto = hb_parc(2);  // Obtener parámetro string
   
   // Procesar datos...
   int resultado = numero * 2;
   
   // Retornar resultado
   hb_retni( resultado );
}
```

### Llamada desde Harbour

```harbour
// Llamar función C desde código Harbour
LOCAL nResultado := MI_FUNCION_C( 42, "texto" )
? "Resultado:", nResultado
```

### Creación de librerías

- **Compilación**: Proceso para compilar código C en librerías
- **Enlazado**: Integración de librerías en proyectos Harbour
- **Distribución**: Empaquetado de librerías para reutilización

### Casos de uso comunes

- **Operaciones computacionalmente intensivas**: Algoritmos complejos en C
- **Integración con APIs del sistema**: Acceso a funciones específicas del OS
- **Librerías de terceros**: Uso de librerías existentes escritas en C/C++
- **Optimización de rendimiento**: Reescribir partes críticas en código nativo