# Chapter 6: Files and databases

This chapter explores Harbour's capabilities for file handling, from low-level operations to database management, a fundamental pillar of the language.

## Low-level file handling

Harbour, like C, offers a set of functions for low-level file manipulation. These functions operate directly on files at the operating system level, providing total control over reading and writing data. They are ideal for handling text files, configuration files, binaries or any format other than `.dbf`.

The main functions are:

-   `FCREATE( <cFile>, [<nAttribute>] ) -> hFile`: Creates a new file on disk. Returns a "handle" (an identifier number) to be used by other functions. If there's an error, returns -1.
    -   `<cFile>`: The name of the file to create.
    -   `<nAttribute>`: An optional attribute for the file (0: normal, 1: read-only, 2: hidden, 4: system).

-   `FOPEN( <cFile>, [<nMode>] ) -> hFile`: Opens an existing file.
    -   `<cFile>`: The name of the file to open.
    -   `<nMode>`: The access mode (0: read-only, 1: write-only, 2: read and write).

-   `FCLOSE( <hFile> )`: Closes a file that was previously opened, freeing the "handle".

Once a file is open, you can read and write to it:

-   `FREAD( <hFile>, @cBuffer, <nBytes> ) -> nBytesRead`: Reads a number of bytes from the file and stores them in a variable (buffer).
-   `FWRITE( <hFile>, <cBuffer>, [<nBytes>] ) -> nBytesWritten`: Writes the contents of a variable (buffer) to the file.
-   `FSEEK( <hFile>, <nOffset>, [<nOrigin>] )`: Moves the file pointer to a new position to read or write from there.

```harbour
// Example of low-level writing and reading
#include "fileio.ch"

PROCEDURE Main()
    LOCAL hFile, cText

    // Create and write to a file
    hFile := FCREATE( "test.txt" )
    IF hFile != -1
        FWRITE( hFile, "Hello, Harbour!" )
        FCLOSE( hFile )
        ? "File 'test.txt' created."
    ELSE
        ? "Error creating file."
    ENDIF

    // Open and read the same file
    hFile := FOPEN( "test.txt", FO_READ ) // FO_READ = 0
    IF hFile != -1
        // Reserve space to read up to 1024 bytes
        cText := Space( 1024 )
        FREAD( hFile, @cText, 1024 )
        FCLOSE( hFile )
        ? "Content:", Trim(cText)
    ELSE
        ? "Error opening file."
    ENDIF
RETURN
```

## The .dbf file format

The heart of data management in Harbour is the `.dbf` format. Inherited from dBase, it is a standard file format for storing structured data in tables.

A `.dbf` file consists of two main parts:
1.  **Header**: Stores metadata about the table, such as the number of records, the date of last update and, most importantly, the definition of each column (field): name, data type (character, numeric, date, logical, memo), length and number of decimals.
2.  **Records**: A sequence of fixed-length records that contain the actual data.

Thanks to this structure, Harbour can access data very efficiently.

## Database functions

Harbour provides a high-level language for manipulating `.dbf` files in a simple and intuitive way.

-   `USE <cFile>`: Opens a `.dbf` table in the current work area. Once opened, you can start working with it.
    ```harbour
    USE "customers.dbf" // Opens the customers table
    ```

-   `APPEND BLANK`: Adds a new empty record to the end of the table. The record pointer moves to this new position, ready for data to be added.

-   `REPLACE <field> WITH <value> [,...]`: Assigns a value to one or more fields of the current record. This is the main function for saving data in a record.

```harbour
// Example of how to add a new customer
PROCEDURE Main()
    // Open the table. If it doesn't exist, create it with this structure.
    IF !File("customers.dbf")
        dbCreate( "customers.dbf", { ;
            { "NAME", "C", 25, 0 }, ;
            { "CITY", "C", 20, 0 }, ;
            { "AGE",  "N",  3, 0 }  ;
        } )
    ENDIF

    USE "customers.dbf"

    APPEND BLANK
    REPLACE NAME WITH "John Doe"
    REPLACE CITY WITH "Madrid"
    REPLACE AGE  WITH 30

    // To save changes to disk
    COMMIT

    // Close the table
    CLOSE DATABASES

    ? "Record added to customers.dbf"
RETURN
```

## Indexes: how to speed up searches

Working with large tables can be slow if searching sequentially record by record (`LOCATE`). To solve this, **indexes** are used.

An index file (`.ntx` by default in Harbour) is a complementary file to the `.dbf` that contains a sorted version of one or more keys from the table. Instead of searching in the main table, Harbour searches in the index (which is much faster) and gets the exact position of the record in the `.dbf` file.

-   `INDEX ON <expression> TO <cIndexFile>`: Creates an index file. The expression is usually a field or a combination of fields.
-   `SET INDEX TO <cIndexFile>`: Opens an existing index and associates it with the current table.
-   `SEEK <value>`: Performs a very fast search using the active index. If it finds the value, the record pointer moves to the corresponding record.

```harbour
// Using an index to search for a customer
USE "customers.dbf"
// Create an index by the NAME field if it doesn't exist
IF !File("customers.ntx")
    INDEX ON NAME TO "customers"
ENDIF

SET INDEX TO "customers"

// Search for "John Doe"
SEEK "John Doe"

IF Found()
    ? "Found:", NAME, CITY
ELSE
    ? "Customer not found."
ENDIF

CLOSE DATABASES
```

## Relational Database Management Systems (RDDs)

One of Harbour's most powerful features is its **Replaceable Database Drivers** (RDD) architecture.

The RDD system allows Harbour code for data manipulation (`USE`, `SEEK`, `REPLACE`, etc.) to work with different file formats and databases, not just with `.dbf`. Harbour abstracts the database engine, so you can switch from one format to another with minimal modifications to your code.

To use a different RDD, simply request it before opening tables.

```harbour
// Example of how to use the RDD for DBF with FoxPro format
REQUEST DBFCDX
RDDSETDEFAULT("DBFCDX")

// From here on, USE will use the DBFCDX driver
// which supports FoxPro .cdx indexes
USE "suppliers.dbf" // Opens a DBF compatible with FoxPro
...
```

Harbour includes native support for several formats (`DBFNTX`, `DBFCDX`, `DBFNSX`) and there are third-party RDDs to connect to modern database systems like **PostgreSQL, MySQL, SQLite and Oracle**. This allows Harbour applications to interact with powerful client/server database engines, combining the simplicity of the xBase language with the robustness of relational DBMS.