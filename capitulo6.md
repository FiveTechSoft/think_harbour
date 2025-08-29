# Capítulo 6: Archivos y bases de datos

Este capítulo explora las capacidades de Harbour para el manejo de archivos, desde operaciones de bajo nivel hasta la gestión de bases de datos, un pilar fundamental del lenguaje.

## Manejo de archivos de bajo nivel

Harbour, al igual que C, ofrece un conjunto de funciones para la manipulación de archivos a bajo nivel. Estas funciones operan directamente sobre los archivos a nivel del sistema operativo, proporcionando un control total sobre la lectura y escritura de datos. Son ideales para manejar archivos de texto, archivos de configuración, binarios o cualquier formato que no sea `.dbf`.

Las funciones principales son:

-   `FCREATE( <cFile>, [<nAttribute>] ) -> hFile`: Crea un nuevo archivo en disco. Devuelve un "handle" (un número identificador) para ser usado por otras funciones. Si hay un error, devuelve -1.
    -   `<cFile>`: El nombre del archivo a crear.
    -   `<nAttribute>`: Un atributo opcional para el archivo (0: normal, 1: solo lectura, 2: oculto, 4: sistema).

-   `FOPEN( <cFile>, [<nMode>] ) -> hFile`: Abre un archivo existente.
    -   `<cFile>`: El nombre del archivo a abrir.
    -   `<nMode>`: El modo de acceso (0: solo lectura, 1: solo escritura, 2: lectura y escritura).

-   `FCLOSE( <hFile> )`: Cierra un archivo que fue abierto previamente, liberando el "handle".

Una vez que un archivo está abierto, puedes leer y escribir en él:

-   `FREAD( <hFile>, @cBuffer, <nBytes> ) -> nBytesRead`: Lee una cantidad de bytes del archivo y los almacena en una variable (buffer).
-   `FWRITE( <hFile>, <cBuffer>, [<nBytes>] ) -> nBytesWritten`: Escribe el contenido de una variable (buffer) en el archivo.
-   `FSEEK( <hFile>, <nOffset>, [<nOrigin>] )`: Mueve el puntero del archivo a una nueva posición para leer o escribir desde allí.

```harbour
// Ejemplo de escritura y lectura de bajo nivel
#include "fileio.ch"

PROCEDURE Main()
    LOCAL hFile, cTexto

    // Crear y escribir en un archivo
    hFile := FCREATE( "test.txt" )
    IF hFile != -1
        FWRITE( hFile, "Hola, Harbour!" )
        FCLOSE( hFile )
        ? "Archivo 'test.txt' creado."
    ELSE
        ? "Error creando el archivo."
    ENDIF

    // Abrir y leer el mismo archivo
    hFile := FOPEN( "test.txt", FO_READ ) // FO_READ = 0
    IF hFile != -1
        // Reservamos espacio para leer hasta 1024 bytes
        cTexto := Space( 1024 )
        FREAD( hFile, @cTexto, 1024 )
        FCLOSE( hFile )
        ? "Contenido:", Trim(cTexto)
    ELSE
        ? "Error abriendo el archivo."
    ENDIF
RETURN
```

## El formato de archivo .dbf

El corazón de la gestión de datos en Harbour es el formato `.dbf`. Heredado de dBase, es un formato de archivo estándar para almacenar datos estructurados en tablas.

Un archivo `.dbf` se compone de dos partes principales:
1.  **Cabecera (Header)**: Almacena metadatos sobre la tabla, como el número de registros, la fecha de la última actualización y, lo más importante, la definición de cada columna (campo): nombre, tipo de dato (carácter, numérico, fecha, lógico, memo), longitud y número de decimales.
2.  **Registros (Records)**: Una secuencia de registros de longitud fija que contienen los datos reales.

Gracias a esta estructura, Harbour puede acceder a los datos de forma muy eficiente.

## Funciones de base de datos

Harbour proporciona un lenguaje de alto nivel para manipular archivos `.dbf` de forma sencilla e intuitiva.

-   `USE <cFile>`: Abre una tabla `.dbf` en el área de trabajo actual. Una vez abierta, se puede empezar a trabajar con ella.
    ```harbour
    USE "clientes.dbf" // Abre la tabla clientes
    ```

-   `APPEND BLANK`: Añade un nuevo registro vacío al final de la tabla. El puntero de registro se mueve a esta nueva posición, listo para que se le añadan datos.

-   `REPLACE <campo> WITH <valor> [,...]`: Asigna un valor a uno o más campos del registro actual. Es la función principal para guardar datos en un registro.

```harbour
// Ejemplo de cómo añadir un nuevo cliente
PROCEDURE Main()
    // Abrir la tabla. Si no existe, se crea con esta estructura.
    IF !File("clientes.dbf")
        dbCreate( "clientes.dbf", { ;
            { "NOMBRE", "C", 25, 0 }, ;
            { "CIUDAD", "C", 20, 0 }, ;
            { "EDAD",   "N",  3, 0 }  ;
        } )
    ENDIF

    USE "clientes.dbf"

    APPEND BLANK
    REPLACE NOMBRE WITH "Juan Perez"
    REPLACE CIUDAD WITH "Madrid"
    REPLACE EDAD   WITH 30

    // Para guardar los cambios en disco
    COMMIT

    // Cerramos la tabla
    CLOSE DATABASES

    ? "Registro añadido a clientes.dbf"
RETURN
```

## Índices: cómo acelerar búsquedas

Trabajar con tablas grandes puede ser lento si se busca secuencialmente registro por registro (`LOCATE`). Para solucionar esto, se utilizan los **índices**.

Un archivo de índice (`.ntx` por defecto en Harbour) es un archivo complementario al `.dbf` que contiene una versión ordenada de una o más claves de la tabla. En lugar de buscar en la tabla principal, Harbour busca en el índice (que es mucho más rápido) y obtiene la posición exacta del registro en el archivo `.dbf`.

-   `INDEX ON <expresion> TO <cArchivoIndice>`: Crea un archivo de índice. La expresión suele ser un campo o una combinación de campos.
-   `SET INDEX TO <cArchivoIndice>`: Abre un índice existente y lo asocia con la tabla actual.
-   `SEEK <valor>`: Realiza una búsqueda rapidísima utilizando el índice activo. Si encuentra el valor, el puntero de registro se mueve al registro correspondiente.

```harbour
// Usando un índice para buscar un cliente
USE "clientes.dbf"
// Creamos un índice por el campo NOMBRE si no existe
IF !File("clientes.ntx")
    INDEX ON NOMBRE TO "clientes"
ENDIF

SET INDEX TO "clientes"

// Buscamos a "Juan Perez"
SEEK "Juan Perez"

IF Found()
    ? "Encontrado:", NOMBRE, CIUDAD
ELSE
    ? "Cliente no encontrado."
ENDIF

CLOSE DATABASES
```

## Sistemas de gestión de bases de datos relacionales (RDDs)

Una de las características más potentes de Harbour es su arquitectura de **Controladores de Datos Reemplazables** (RDD, por sus siglas en inglés: _Replaceable Database Drivers_).

El sistema RDD permite que el código Harbour para la manipulación de datos (`USE`, `SEEK`, `REPLACE`, etc.) funcione con diferentes formatos de archivo y bases de datos, no solo con `.dbf`. Harbour abstrae el motor de base de datos, por lo que puedes cambiar de un formato a otro con mínimas modificaciones en tu código.

Para usar un RDD diferente, simplemente se solicita antes de abrir las tablas.

```harbour
// Ejemplo de cómo usar el RDD para DBF con formato de FoxPro
REQUEST DBFCDX
RDDSETDEFAULT("DBFCDX")

// A partir de aquí, USE usará el driver DBFCDX
// que soporta índices .cdx de FoxPro
USE "proveedores.dbf" // Abre un DBF compatible con FoxPro
...
```

Harbour incluye soporte para varios formatos de forma nativa (`DBFNTX`, `DBFCDX`, `DBFNSX`) y existen RDDs de terceros para conectarse a sistemas de bases de datos modernos como **PostgreSQL, MySQL, SQLite y Oracle**. Esto permite a las aplicaciones Harbour interactuar con potentes motores de bases de datos cliente/servidor, combinando la simplicidad del lenguaje xBase con la robustez de los SGBD relacionales.
