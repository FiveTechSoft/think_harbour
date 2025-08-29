## Graphic Terminal (GT)

La familia de funciones GT de Harbour proporciona una capa de abstracción para la salida a pantalla en modo texto. Permite que las aplicaciones escriban en la consola de una manera independiente del sistema operativo, soportando diferentes terminales y codificaciones de caracteres.

## Interfaces de línea de comandos (CLI)

El desarrollo de aplicaciones de línea de comandos (CLI) en Harbour es su forma más básica de interacción. Se basa en funciones de entrada y salida estándar como `QOut()` y `Inkey()` para crear interfaces de usuario sencillas y eficientes en entornos de texto.

## Librerías contrib para GUI: GTWVG, HwGUI, y hbqt

Para el desarrollo de interfaces gráficas de usuario (GUI), Harbour cuenta con varias librerías contribuidas por la comunidad que extienden sus capacidades:
- **GTWVG**: Proporciona widgets y elementos gráficos que se ejecutan sobre la capa GT, permitiendo crear interfaces con aspecto gráfico en un entorno de modo texto.
- **HwGUI**: Es una librería para crear aplicaciones GUI nativas para Windows, ofreciendo un conjunto completo de controles y funcionalidades.
- **hbqt**: Es un binding de la librería Qt, que permite desarrollar aplicaciones multiplataforma (Windows, Linux, macOS) con una interfaz gráfica moderna y potente.

## Manejo de eventos y diseño de formularios

El manejo de eventos es fundamental en las aplicaciones GUI. En librerías como HwGUI o hbqt, se utiliza un bucle de eventos para capturar y responder a las interacciones del usuario (clics de ratón, pulsaciones de teclas, etc.). El diseño de formularios se realiza definiendo ventanas y colocando controles (botones, cajas de texto, etc.) en ellas, y asociando código a los eventos de dichos controles para implementar la lógica de la aplicación.
