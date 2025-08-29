# Chapter 11: Libraries and External Packages

Harbour has a powerful language core, but its true strength lies in its ecosystem of libraries and tools that extend its capabilities to almost any imaginable area of programming. This chapter explores how to manage these libraries, how to build complex projects, and how to use some of the most popular libraries for modern tasks like XML and JSON manipulation.

## 1. The `hbmk2` Build System

`hbmk2` is Harbour's official build tool. It is a powerful script that automates the process of compiling and linking programs, handling dependencies, external libraries, different C compilers, and cross-platform building.

### Why use `hbmk2`?

- **Simplicity:** Automates detection of the C compiler, paths, and necessary libraries.
- **Portability:** A single build script can work on Windows, Linux, and macOS.
- **Power:** Supports multi-module projects, different dialects (xBase++), and is highly configurable.
- **Dependency management:** Can automatically compile and link libraries required by your project.

### Basic Usage

The simplest use is to pass it the main `.prg` file:

```bash
# Compile and link myapp.prg to create myapp.exe (or myapp on Linux)
hbmk2 myapp.prg
```

To include an external library (for example, `hbcurl`), use the `-l` flag:

```bash
# Link the hbcurl library.
# hbmk2 automatically searches for hbcurl.hbc in standard paths.
hbmk2 myapp.prg -lhbcurl
```

### Project Files (`.hbp`)

For more complex projects, it's recommended to use an `.hbp` (Harbour Project) file that groups all compilation options.

**Example of `myapp.hbp`:**

```
# Source code files
myapp.prg
utils.prg
reports.prg

# Libraries to link
-lhbcurl
-lhbzip

# Compiler options
-w3 -es2
```

To build the project, simply execute:

```bash
hbmk2 myapp.hbp
```

## 2. Popular `contrib` Libraries

Harbour's distribution includes a `contrib` folder that contains a vast collection of libraries contributed by the community. These libraries are a fundamental part of the ecosystem. To use them, you usually only need to add the `-l<libname>` flag to `hbmk2`.

Some of the most used are:

- **`hbcurl`**: A complete wrapper over the popular `libcurl` library. Allows making HTTP/S requests (GET, POST), FTP, etc. Essential for interacting with web APIs.
- **`hbtip`**: Provides low-level networking functionality based on TCP/IP, allowing creation of clients and servers for custom protocols.
- **`hbzip`**: Offers functions to create and extract `.zip` format files. Includes `hbziparc` for compression and `hbunzip` for decompression.
- **`hbsqlite`**: Allows working with SQLite3 databases, a lightweight and embedded database, ideal for desktop or mobile applications.
- **`hbpgsql`**: Client for PostgreSQL database.
- **`hbexpat`**: An XML parser based on the `expat` library, useful for reading and processing XML documents efficiently.
- **`hbhpdf`**: Allows native creation of PDF documents from Harbour, using the `libHaru` library.
- **`hbide`**: A simple Integrated Development Environment (IDE), written in Harbour.

## 3. XML and JSON File Manipulation

Data exchange with modern systems is mainly done through formats like JSON and XML. Harbour offers native support or through `contrib` libraries for both.

### JSON Manipulation

Harbour includes native JSON support in its core, so no external library is needed. The main functions are `hb_jsonEncode()` and `hb_jsonDecode()`.

- **`hb_jsonEncode( <value> )`**: Converts a Harbour value (array, hash, string, number) to a JSON format text string.
- **`hb_jsonDecode( <jsonString> )`**: Converts a JSON text string to a Harbour value (hashes for objects, arrays for lists).

**Usage example:**

```prg
PROCEDURE Main()
    LOCAL hData, cJson, hResult

    // 1. Create a Hash in Harbour and convert it to JSON
    hData := { => }
    hData[ "name" ] := "John Doe"
    hData[ "age" ]  := 30
    hData[ "active" ] := .T.
    hData[ "courses" ] := { "Harbour", "SQL", "Git" }

    cJson := hb_jsonEncode( hData )

    ? "Harbour object:"
    AEval( hData, { |v, k| QOut(hb_NtoS(k) + ": " + hb_ValToExp(v)) } )

    ?
    ? "Generated JSON string:"
    ? cJson
    // Output: {"name":"John Doe","age":30,"active":true,"courses":["Harbour","SQL","Git"]}

    // 2. Convert the JSON string back to a Harbour Hash
    hResult := hb_jsonDecode( cJson )

    ?
    ? "Decoded Harbour object:"
    ? hResult[ "name" ] // "John Doe"
    ? hResult[ "courses" ][ 2 ] // "SQL"

    RETURN
```

### XML Manipulation

For XML, the most common way to work is using the `hbexpat` library, found in `contrib`. This library provides a fast and efficient parser. It doesn't build a DOM tree in memory, but works through events (SAX-like parser), making it very efficient for large files.

To use it, you need to link the library:

```bash
hbmk2 myxmlapp.prg -lhbexpat
```

**Example of reading XML with `hbexpat`:**

Suppose we have a `data.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<people>
    <person id="1">
        <name>Ana</name>
        <city>Madrid</city>
    </person>
    <person id="2">
        <name>Luis</name>
        <city>Barcelona</city>
    </person>
</people>
```

We can process it with the following code:

```prg
#include "hbexpat.ch"

PROCEDURE Main()
    LOCAL oParser, cXml

    // Load the XML file content
    cXml := MemoRead( "data.xml" )

    // Create the parser object
    oParser := hb_expat_Create()

    // Define the "handlers" (functions that will be called when finding elements)
    oParser:startElementHandler := { |o, cTag, hAttr| StartElement(cTag, hAttr) }
    oParser:characterDataHandler := { |o, cText| CharData(cText) }
    oParser:endElementHandler := { |o, cTag| EndElement(cTag) }

    // Process the XML
    IF ! oParser:Parse( cXml )
        ? "Parsing error:", oParser:ErrorCode(), oParser:ErrorString()
    ENDIF

    // Free resources
    oParser:Destroy()

    RETURN

STATIC PROCEDURE StartElement( cTag, hAttributes )
    QOut( "Start tag:", cTag )
    IF ! Empty( hAttributes )
        AEval( hAttributes, { |v, k| QOut( "  Attribute:", k, "=", v ) } )
    ENDIF
    RETURN

STATIC PROCEDURE CharData( cText )
    // Text can come in fragments, so it should be accumulated
    // if necessary. Here we just show it.
    IF ! Empty( AllTrim( cText ) )
        QOut( "  Text:", AllTrim( cText ) )
    ENDIF
    RETURN

STATIC PROCEDURE EndElement( cTag )
    QOut( "End tag:", cTag )
    RETURN
```

This code will traverse the XML document and trigger the `StartElement`, `CharData` and `EndElement` functions as it finds opening tags, text and closing tags, respectively, allowing you to process the content in a structured way.