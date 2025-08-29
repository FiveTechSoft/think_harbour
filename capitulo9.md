## Interfaces de Usuario en Harbour: Desde la Consola hasta GUI Modernas

Harbour, como descendiente de Clipper, nació en el mundo de las aplicaciones de modo texto, pero ha evolucionado para soportar interfaces gráficas de usuario (GUI) completas y modernas. A continuación, se detallan las distintas capas y librerías disponibles para el desarrollo de interfaces en Harbour.

### Graphic Terminal (GT): La base de la portabilidad

La familia de funciones **GT** de Harbour proporciona una capa de abstracción fundamental para la salida a pantalla en modo texto. Su propósito es independizar el código de la aplicación del sistema operativo o terminal específico en el que se ejecuta. En lugar de escribir directamente en la consola de Windows o en un terminal de Linux, se utilizan las funciones GT, y Harbour se encarga de "traducir" esas operaciones al sistema subyacente.

**Características principales:**

*   **Independencia de la plataforma**: El mismo código que gestiona una pantalla en modo texto funciona en Windows, Linux y macOS sin cambios.
*   **Funcionalidad básica**: Incluye funciones para posicionar el cursor (`SetPos()`), escribir texto (`DispOut()`), gestionar colores (`SetColor()`) y dibujar cajas (`DispBox()`).
*   **Soporte para múltiples "drivers"**: Harbour puede usar diferentes backends para la salida GT, como `GTWIN` en Windows, `GTCGI` para aplicaciones web, o `GTCurses` en sistemas tipo Unix.

### Interfaces de Línea de Comandos (CLI)

El desarrollo de aplicaciones de línea de comandos (CLI) es la forma más directa de interacción en Harbour. Se basa en funciones de entrada y salida estándar que leen del teclado y escriben en la pantalla. Es ideal para utilidades, procesos por lotes (batch) y aplicaciones donde la simplicidad es clave.

**Funciones clave**:

*   `QOut()` y `QQOut()`: Para mostrar texto en la posición actual del cursor.
*   `Inkey()`: Captura una única pulsación de tecla, permitiendo una interacción en tiempo real.
*   `Accept`, `Input`: Para que el usuario introduzca cadenas de texto o números.
*   `Wait`: Pausa la ejecución hasta que el usuario presiona una tecla.

Estas funciones, combinadas con la capa GT, permiten crear interfaces de texto interactivas y robustas.

### Librerías Contribuidas para GUI: GTWVG, HwGUI y hbqt

Para el desarrollo de interfaces gráficas de usuario (GUI), Harbour depende de librerías contribuidas por la comunidad que extienden sus capacidades nativas. Tres de las más importantes son:

*   **GTWVG (GT for Windows Virtual Graphics)**: Es una solución ingeniosa que construye una interfaz con aspecto gráfico sobre la capa GT de modo texto. Permite modernizar aplicaciones de consola añadiendo elementos como ventanas, botones, menús desplegables y soporte para ratón, todo ello sin abandonar el entorno de texto. Es una excelente opción para migrar aplicaciones Clipper antiguas dándoles un toque más moderno con un esfuerzo mínimo.

*   **HwGUI (Harbour for Windows GUI)**: Es una librería potente y madura para crear aplicaciones GUI **nativas para Windows**. HwGUI ofrece un conjunto completo de controles (widgets) como formularios, botones, cajas de texto, grids (tablas), y más. Su modelo de programación es puramente event-driven (dirigido por eventos), lo que es estándar en el desarrollo GUI. Es una de las opciones más populares para desarrollar aplicaciones de escritorio profesionales en Harbour para el ecosistema Windows.

*   **hbqt (Harbour Binding for Qt)**: Es un *binding* de la prestigiosa librería **Qt**, uno de los frameworks más potentes y completos para el desarrollo de aplicaciones multiplataforma. Al usar hbqt, los desarrolladores de Harbour pueden crear aplicaciones con interfaces gráficas modernas y potentes que compilan y se ejecutan de forma nativa en **Windows, Linux y macOS**. hbqt da acceso a la inmensa funcionalidad de Qt, incluyendo gráficos avanzados, multimedia, acceso a bases de datos y comunicaciones en red, todo desde código Harbour.

### Manejo de Eventos y Diseño de Formularios

El paradigma de programación en las aplicaciones GUI es el **manejo de eventos**. A diferencia de los programas de consola que siguen un flujo secuencial, las aplicaciones GUI esperan a que el usuario realice una acción (un clic de ratón, pulsar una tecla, etc.).

*   **Bucle de eventos (Event Loop)**: El corazón de una aplicación GUI es un bucle que captura constantemente los eventos del sistema operativo y los dirige a la parte correspondiente del código.
*   **Callbacks y Bloques de Código**: En librerías como HwGUI o hbqt, se asocia una función o un bloque de código a un evento específico de un control. Por ejemplo, se define un bloque de código que se ejecutará únicamente cuando un usuario haga clic en un botón determinado.
*   **Diseñadores Visuales**: Para librerías como HwGUI y hbqt, existen herramientas de terceros que permiten diseñar formularios de manera visual (arrastrando y soltando controles), lo que acelera enormemente el desarrollo y facilita la creación de interfaces complejas. El diseñador genera código Harbour que luego se integra en la aplicación.
