# Chapter 9: User Interface Development

## User Interfaces in Harbour: From Console to Modern GUIs

Harbour, as a descendant of Clipper, was born in the world of text-mode applications, but has evolved to support complete and modern graphical user interfaces (GUI). Below are the different layers and libraries available for interface development in Harbour.

### Graphic Terminal (GT): The foundation of portability

Harbour's **GT** family of functions provides a fundamental abstraction layer for text-mode screen output. Its purpose is to make application code independent from the specific operating system or terminal on which it runs. Instead of writing directly to the Windows console or a Linux terminal, GT functions are used, and Harbour takes care of "translating" those operations to the underlying system.

**Main features:**

*   **Platform independence**: The same code that manages a text-mode screen works on Windows, Linux and macOS without changes.
*   **Basic functionality**: Includes functions to position the cursor (`SetPos()`), write text (`DispOut()`), manage colors (`SetColor()`) and draw boxes (`DispBox()`).
*   **Support for multiple "drivers"**: Harbour can use different backends for GT output, such as `GTWIN` on Windows, `GTCGI` for web applications, or `GTCurses` on Unix-like systems.

### Command Line Interfaces (CLI)

Command line interface (CLI) application development is the most direct form of interaction in Harbour. It is based on standard input and output functions that read from the keyboard and write to the screen. It is ideal for utilities, batch processes and applications where simplicity is key.

**Key functions**:

*   `QOut()` and `QQOut()`: To display text at the current cursor position.
*   `Inkey()`: Captures a single key press, allowing real-time interaction.
*   `Accept`, `Input`: For the user to enter text strings or numbers.
*   `Wait`: Pauses execution until the user presses a key.

These functions, combined with the GT layer, allow creating interactive and robust text interfaces.

### Contributed Libraries for GUI: GTWVG, HwGUI and hbqt

For graphical user interface (GUI) development, Harbour relies on libraries contributed by the community that extend its native capabilities. Three of the most important are:

*   **GTWVG (GT for Windows Virtual Graphics)**: Is an ingenious solution that builds a graphical-looking interface over the text-mode GT layer. It allows modernizing console applications by adding elements like windows, buttons, drop-down menus and mouse support, all without leaving the text environment. It is an excellent option for migrating old Clipper applications giving them a more modern touch with minimal effort.

*   **HwGUI (Harbour for Windows GUI)**: Is a powerful and mature library for creating **native Windows GUI** applications. HwGUI offers a complete set of controls (widgets) such as forms, buttons, text boxes, grids (tables), and more. Its programming model is purely event-driven, which is standard in GUI development. It is one of the most popular options for developing professional desktop applications in Harbour for the Windows ecosystem.

*   **hbqt (Harbour Binding for Qt)**: Is a *binding* for the prestigious **Qt** library, one of the most powerful and complete frameworks for cross-platform application development. By using hbqt, Harbour developers can create applications with modern and powerful graphical interfaces that compile and run natively on **Windows, Linux and macOS**. hbqt gives access to Qt's immense functionality, including advanced graphics, multimedia, database access and network communications, all from Harbour code.

### Event Handling and Form Design

The programming paradigm in GUI applications is **event handling**. Unlike console programs that follow a sequential flow, GUI applications wait for the user to perform an action (a mouse click, key press, etc.).

*   **Event Loop**: The heart of a GUI application is a loop that constantly captures events from the operating system and directs them to the corresponding part of the code.
*   **Callbacks and Code Blocks**: In libraries like HwGUI or hbqt, a function or code block is associated with a specific event of a control. For example, a code block is defined that will execute only when a user clicks on a specific button.
*   **Visual Designers**: For libraries like HwGUI and hbqt, there are third-party tools that allow designing forms visually (drag and drop controls), which greatly speeds up development and facilitates the creation of complex interfaces. The designer generates Harbour code that is then integrated into the application.