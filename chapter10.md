# Chapter 10: Connectivity and Advanced Databases

In modern application development, the ability to interact with database management systems (DBMS) is fundamental. Harbour, thanks to its flexibility and its ecosystem of contributed libraries (`contrib`), offers powerful tools to connect and manipulate SQL databases, overcoming the limitations of native table formats like DBF.

This chapter explores how to connect Harbour applications with the most popular DBMS and how to use the SQL language to manage data.

## 1. Connection to SQL Databases

Harbour can connect to SQL databases in two main ways:
1.  **Through ODBC (Open Database Connectivity):** An abstraction layer that allows an application to access different databases using a common driver.
2.  **Using native libraries:** Libraries optimized for a specific DBMS (MySQL, PostgreSQL, SQLite, etc.), which offer better performance and access to particular functionalities of each engine.

Although ODBC is a generic solution, the use of native libraries is the recommended practice for its efficiency and control. The most important ones are found in the `contrib` directory of the Harbour source code.

## 2. Using `contrib` Libraries

To use these libraries, you need to compile them along with your project. For example, if you use `hbmk2`, you can include the corresponding `.hbp` file or simply invoke the library name.

**Example with hbmk2:**
```bash
hbmk2 my_program.prg -lhbmysql -L/path/to/mysql/lib
```

### 2.1. Connection with MySQL (`hbmysql`)

The `hbmysql` library provides a direct interface to the MySQL client API.

**Steps for connection:**
1.  **Include the header:** `REQUEST HB_EXTENSION_HBMYSQL` or `#include "hbmysql.ch"`
2.  **Connect to server:** `hb_mysql_connect( cHost, cUser, cPassword, cDatabase )`
3.  **Execute queries:** `hb_mysql_query( nConnection, cSQL )`
4.  **Process results:** Use `hb_mysql_fetch_row()` in a loop to read rows.
5.  **Close connection:** `hb_mysql_close( nConnection )`

**Practical example:**
```prg
#include "hbmysql.ch"

PROCEDURE Main()
    LOCAL nConnection, aRow
    LOCAL cServer := "localhost"
    LOCAL cUser   := "root"
    LOCAL cPass   := "your_password"
    LOCAL cDb     := "test"

    // Connect to database
    nConnection := hb_mysql_connect( cServer, cUser, cPass, cDb )

    IF nConnection == 0
        ? "Connection error:", hb_mysql_error()
        RETURN
    ENDIF

    ? "Successful connection!"

    // Execute a query
    IF hb_mysql_query( nConnection, "SELECT id, name FROM users" ) == 0
        // Iterate through results
        WHILE ( aRow := hb_mysql_fetch_row( nConnection ) ) != NIL
            ? "ID:", aRow[ 1 ], "Name:", aRow[ 2 ]
        END
    ELSE
        ? "Query error:", hb_mysql_error( nConnection )
    ENDIF

    // Close connection
    hb_mysql_close( nConnection )
RETURN
```

### 2.2. Connection with PostgreSQL (`hbpgsql`)

The `hbpgsql` library works very similarly, providing access to PostgreSQL's `libpq` API.

**Steps for connection:**
1.  **Include the header:** `REQUEST HB_EXTENSION_HBPGSQL` or `#include "hbpgsql.ch"`
2.  **Connect to server:** `hb_pg_connect( "host=... dbname=... user=... password=..." )`
3.  **Execute queries:** `nResult := hb_pg_exec( nConnection, cSQL )`
4.  **Process results:** Use `hb_pg_fetch_row( nResult, nRow, nColumn )` or iterate over the result.
5.  **Close connection:** `hb_pg_close( nConnection )`

**Practical example:**
```prg
#include "hbpgsql.ch"

PROCEDURE Main()
    LOCAL nConnection, nResult, nRows, i
    LOCAL cConnStr := "host=localhost dbname=test user=postgres password=your_password"

    // Connect to database
    nConnection := hb_pg_connect( cConnStr )

    IF nConnection == 0
        ? "Connection error."
        RETURN
    ENDIF

    ? "Successful connection!"

    // Execute a query
    nResult := hb_pg_exec( nConnection, "SELECT id, product FROM sales" )

    IF hb_pg_resultStatus( nResult ) == PGRES_TUPLES_OK
        nRows := hb_pg_ntuples( nResult )
        ? "Records found:", nRows
        FOR i := 1 TO nRows
            ? "ID:", hb_pg_getvalue( nResult, i, 0 ), "Product:", hb_pg_getvalue( nResult, i, 1 )
        NEXT
    ELSE
        ? "Query error:", hb_pg_resultErrorMessage( nResult )
    ENDIF

    // Free result memory
    hb_pg_clear( nResult )
    // Close connection
    hb_pg_close( nConnection )
RETURN
```

### 2.3. Connection with SQLite (`hbsqlit3`)

SQLite is an embedded database, based on a single file, which makes it ideal for desktop applications or as a portable file format.

**Steps for connection:**
1.  **Include the header:** `REQUEST HB_EXTENSION_HBSQLIT3` or `#include "hbsqlit3.ch"`
2.  **Open the database file:** `nDb := sqlite3_open( "my_database.db" )`
3.  **Execute commands:** `sqlite3_exec( nDb, cSQL )`
4.  **For SELECT queries, use a prepared approach:**
    *   `sqlite3_prepare_v2()`
    *   `sqlite3_step()` to advance row by row.
    *   `sqlite3_column_*()` to get data from each column.
    *   `sqlite3_finalize()` to close the query.
5.  **Close the database:** `sqlite3_close( nDb )`

**Practical example:**
```prg
#include "hbsqlit3.ch"

PROCEDURE Main()
    LOCAL nDb, nStmt, nResult
    LOCAL cDbFile := "inventory.db"
    LOCAL cSQL

    // Open (or create) the database
    nDb := sqlite3_open( cDbFile )
    IF nDb == 0
        ? "Could not open database"
        RETURN
    ENDIF

    // Create a table if it doesn't exist
    cSQL := "CREATE TABLE IF NOT EXISTS products (id INTEGER PRIMARY KEY, name TEXT, price REAL)"
    sqlite3_exec( nDb, cSQL )

    // Prepare a SELECT query
    cSQL := "SELECT name, price FROM products WHERE price > ?"
    nStmt := sqlite3_prepare_v2( nDb, cSQL )

    // Bind a parameter (in this case, the price)
    sqlite3_bind_double( nStmt, 1, 50.0 )

    // Execute the query step by step
    WHILE ( nResult := sqlite3_step( nStmt ) ) == SQLITE_ROW
        ? "Product:", sqlite3_column_text( nStmt, 0 ), "Price:", sqlite3_column_double( nStmt, 1 )
    END

    // Finalize the query and free resources
    sqlite3_finalize( nStmt )

    // Close the database
    sqlite3_close( nDb )
RETURN
```

## 3. Fundamental SQL Concepts

Regardless of the DBMS you choose, interaction is performed through Structured Query Language (SQL). The four basic operations are known as **CRUD** (Create, Read, Update, Delete).

### `SELECT`: Read data

Used to retrieve data from one or more tables.

**Basic syntax:**
`SELECT column1, column2 FROM table WHERE condition ORDER BY column1;`

*   **`SELECT name, email FROM customers`**: Gets only the name and email of all customers.
*   **`SELECT * FROM products WHERE category = 'Dairy'`**: Gets all columns of products that belong to the 'Dairy' category.
*   **`SELECT * FROM sales ORDER BY date DESC`**: Gets all sales ordered by date, from most recent to oldest.

### `INSERT`: Create data

Adds a new record (row) to a table.

**Basic syntax:**
`INSERT INTO table (column1, column2) VALUES (value1, value2);`

*   **`INSERT INTO customers (name, city) VALUES ('John Doe', 'Madrid')`**: Adds a new customer.

**Important:** In Harbour, it is crucial to escape text values to prevent SQL injection. Connection libraries usually offer functions for this (e.g. `hb_mysql_real_escape_string()`).

### `UPDATE`: Update data

Modifies existing records in a table.

**Basic syntax:**
`UPDATE table SET column1 = value1 WHERE condition;`

*   **`UPDATE products SET price = 25.50 WHERE id = 101`**: Changes the price of the product with ID 101.
*   **Careful!** If you omit the `WHERE` clause, you'll update **all** records in the table!

### `DELETE`: Delete data

Removes records from a table.

**Basic syntax:**
`DELETE FROM table WHERE condition;`

*   **`DELETE FROM orders WHERE status = 'Cancelled'`**: Removes all cancelled orders.
*   **Careful!** Like with `UPDATE`, if you omit the `WHERE` clause, you'll delete **all** data from the table!