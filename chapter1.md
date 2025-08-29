# Chapter 1: The Program's Path

This chapter introduces the fundamental concepts of programming in Harbour, from the definition of a program to creating your first executable application.

---

### What is a program?

A **program** is a sequence of instructions written in a language that the computer can understand and execute. In the context of Harbour, a program is a plain text file, usually with the `.prg` extension, that contains source code.

The source code is human-readable and follows the syntactic rules of the Harbour language. However, a computer cannot directly execute this code. First, it must be translated into a format that the processor can understand. This process is called **compilation**.

The Harbour compiler reads your `.prg` file and converts it into an **executable** file (for example, a `.exe` on Windows). When you run this file, the computer performs the tasks you have programmed.

---

### The Harbour language: a compiler and xBase environment

**Harbour** is a high-level, compiled, and cross-platform programming language. It is known for being a modern and open-source successor to the popular **Clipper** language, which places it within the **xBase** family of languages (derived from dBase).

*   **Compiler**: Harbour is not an interpreter. It takes your source code and translates it to C code, which is then compiled into a native binary. This process ensures very high performance and the ability to create standalone applications.
*   **xBase Environment**: Harbour inherits the syntax and philosophy of xBase languages, which are specially designed for data management and business applications. It includes a robust database system (RDDs) and functions for handling tables (`.dbf`), indexes, and record manipulation natively.
*   **Modern Features**: Although its root is xBase, Harbour has evolved to include modern features such as:
    *   Object-Oriented Programming (OOP).
    *   Cross-platform support (Windows, Linux, macOS, etc.).
    *   Multithreading for concurrent tasks.
    *   Extensibility through C/C++ libraries.
    *   A large collection of `contrib` libraries for GUI, web connectivity, etc.

---

### The Harbour preprocessor: macros (`#define`), file inclusion (`#include`), and user-defined commands (`#command`, `#xcommand`)

The **preprocessor** is a tool that runs before compilation. Its function is to modify the source code based on specific directives.

*   **`#include`**: Inserts the contents of another file at the current location. It is commonly used to include header files (`.h` or `.ch`) that contain constants, function declarations, or commands.
    ```harbour
    // Include Harbour's standard constants and commands
    #include "hbclass.ch"
    ```

*   **`#define`**: Creates substitution macros. The preprocessor will replace all instances of the macro name with the defined value.
    ```harbour
    #define MAX_ITEMS 100
    #define GREETING "Hello!"

    // In the code, the preprocessor will replace MAX_ITEMS with 100
    aDatos := ARRAY(MAX_ITEMS)
    ```

*   **`#command` and `#xcommand`**: Allow creating new commands in the language, simplifying syntax and creating high-level abstractions. `#command` uses a simple pattern matching system, while `#xcommand` offers a more powerful regular expression engine.
    ```harbour
    // Define a new command to display a message at a position
    #command MSG <row>, <col> PROMPT <message> => QOut( <row>, <col>, <message> )

    // Using the new command
    MSG 10, 5 PROMPT "This is a test message"
    ```

---

### Your first program: "Hello, world!"

The "Hello, world!" is the simplest program you can write. It demonstrates that your compilation environment is properly configured.

1.  Create a file called `hello.prg`.
2.  Write the following code in it:

    ```harbour
    /*
     * My first program in Harbour
     * Displays "Hello, world!" on the console.
     */
    FUNCTION Main()

       // Clear the screen
       CLS

       // Display the text at row 10, column 5
       @ 10, 5 SAY "Hello, world!"

       // Wait for the user to press a key
       WAIT

    RETURN NIL
    ```

3.  **Compilation**: To compile this program, you'll need the Harbour compiler and `hbmk2`, its build tool. Open a terminal or command line and execute:
    ```bash
    hbmk2 hello.prg
    ```
    `hbmk2` will handle compiling and linking everything necessary, generating a `hello.exe` file (on Windows) or `hello` (on Linux/macOS).

4.  **Execution**: Run the program from the terminal:
    ```bash
    ./hello
    ```
    You'll see the message "Hello, world!" on the screen.

---

### Problem solving and debugging

When a program doesn't work as expected, it's time to debug.

*   **Compilation errors**: Occur if the code violates the syntactic rules of the language. The Harbour compiler will inform you of the error and the line number so you can correct it.

*   **Runtime errors**: Happen while the program is executing. For example, trying to divide by zero or accessing an array outside its bounds. Harbour has an error handling system (`BEGIN SEQUENCE...END`) that allows you to catch these failures and handle them in a controlled manner.

*   **Logic errors**: These are the most difficult to find. The program runs without failing, but the result is not as expected. Tools for solving these errors include:
    *   **Variable display**: Use `QOut()` or `Alert()` to show the contents of a variable at a specific point in the program.
    *   **The Harbour debugger**: Allows running the program step by step, inspecting variables in real time, and analyzing the execution flow. To compile with debugging support, use the `-b` option with `hbmk2`:
      ```bash
      hbmk2 -b my_program.prg
      ```
    *   **Logging messages**: Writing relevant information to a text file (`.log`) can help you trace what the program did, especially in long or unattended processes.