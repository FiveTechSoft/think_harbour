# Capítulo 10: Conectividad y Bases de Datos Avanzadas

En el desarrollo de aplicaciones modernas, la capacidad de interactuar con sistemas de gestión de bases de datos (SGBD) es fundamental. Harbour, gracias a su flexibilidad y a su ecosistema de librerías contribuidas (`contrib`), ofrece potentes herramientas para conectar y manipular bases de datos SQL, superando las limitaciones de los formatos de tabla nativos como DBF.

Este capítulo explora cómo conectar aplicaciones Harbour con los SGBD más populares y cómo utilizar el lenguaje SQL para gestionar los datos.

## 1. Conexión a Bases de Datos SQL

Harbour puede conectarse a bases de datos SQL de dos maneras principales:
1.  **A través de ODBC (Open Database Connectivity):** Una capa de abstracción que permite a una aplicación acceder a diferentes bases de datos usando un driver común.
2.  **Usando librerías nativas:** Librerías optimizadas para un SGBD específico (MySQL, PostgreSQL, SQLite, etc.), que ofrecen mejor rendimiento y acceso a funcionalidades particulares de cada motor.

Aunque ODBC es una solución genérica, el uso de librerías nativas es la práctica recomendada por su eficiencia y control. Las más importantes se encuentran en el directorio `contrib` del código fuente de Harbour.

## 2. Uso de las Librerías `contrib`

Para utilizar estas librerías, necesitas compilarlas junto con tu proyecto. Por ejemplo, si usas `hbmk2`, puedes incluir el fichero `.hbp` correspondiente o simplemente invocar el nombre de la librería.

**Ejemplo con hbmk2:**
```bash
hbmk2 mi_programa.prg -lhbmysql -L/ruta/a/mysql/lib
```

### 2.1. Conexión con MySQL (`hbmysql`)

La librería `hbmysql` proporciona una interfaz directa a la API de cliente de MySQL.

**Pasos para la conexión:**
1.  **Incluir el header:** `REQUEST HB_EXTENSION_HBMYSQL` o `#include "hbmysql.ch"`
2.  **Conectar al servidor:** `hb_mysql_connect( cHost, cUsuario, cPassword, cBaseDeDatos )`
3.  **Ejecutar consultas:** `hb_mysql_query( nConexion, cSQL )`
4.  **Procesar resultados:** Usar `hb_mysql_fetch_row()` en un bucle para leer las filas.
5.  **Cerrar la conexión:** `hb_mysql_close( nConexion )`

**Ejemplo práctico:**
```prg
#include "hbmysql.ch"

PROCEDURE Main()
    LOCAL nConnection, aRow
    LOCAL cServer := "localhost"
    LOCAL cUser   := "root"
    LOCAL cPass   := "tu_contraseña"
    LOCAL cDb     := "test"

    // Conectar a la base de datos
    nConnection := hb_mysql_connect( cServer, cUser, cPass, cDb )

    IF nConnection == 0
        ? "Error al conectar:", hb_mysql_error()
        RETURN
    ENDIF

    ? "Conexión exitosa!"

    // Ejecutar una consulta
    IF hb_mysql_query( nConnection, "SELECT id, nombre FROM usuarios" ) == 0
        // Recorrer los resultados
        WHILE ( aRow := hb_mysql_fetch_row( nConnection ) ) != NIL
            ? "ID:", aRow[ 1 ], "Nombre:", aRow[ 2 ]
        END
    ELSE
        ? "Error en la consulta:", hb_mysql_error( nConnection )
    ENDIF

    // Cerrar la conexión
    hb_mysql_close( nConnection )
RETURN
```

### 2.2. Conexión con PostgreSQL (`hbpgsql`)

La librería `hbpgsql` funciona de manera muy similar, proveyendo acceso a la API `libpq` de PostgreSQL.

**Pasos para la conexión:**
1.  **Incluir el header:** `REQUEST HB_EXTENSION_HBPGSQL` o `#include "hbpgsql.ch"`
2.  **Conectar al servidor:** `hb_pg_connect( "host=... dbname=... user=... password=..." )`
3.  **Ejecutar consultas:** `nResult := hb_pg_exec( nConexion, cSQL )`
4.  **Procesar resultados:** Usar `hb_pg_fetch_row( nResult, nFila, nColumna )` o iterar sobre el resultado.
5.  **Cerrar la conexión:** `hb_pg_close( nConexion )`

**Ejemplo práctico:**
```prg
#include "hbpgsql.ch"

PROCEDURE Main()
    LOCAL nConnection, nResult, nRows, i
    LOCAL cConnStr := "host=localhost dbname=test user=postgres password=tu_contraseña"

    // Conectar a la base de datos
    nConnection := hb_pg_connect( cConnStr )

    IF nConnection == 0
        ? "Error de conexión."
        RETURN
    ENDIF

    ? "Conexión exitosa!"

    // Ejecutar una consulta
    nResult := hb_pg_exec( nConnection, "SELECT id, producto FROM ventas" )

    IF hb_pg_resultStatus( nResult ) == PGRES_TUPLES_OK
        nRows := hb_pg_ntuples( nResult )
        ? "Registros encontrados:", nRows
        FOR i := 1 TO nRows
            ? "ID:", hb_pg_getvalue( nResult, i, 0 ), "Producto:", hb_pg_getvalue( nResult, i, 1 )
        NEXT
    ELSE
        ? "Error en la consulta:", hb_pg_resultErrorMessage( nResult )
    ENDIF

    // Liberar memoria del resultado
    hb_pg_clear( nResult )
    // Cerrar la conexión
    hb_pg_close( nConnection )
RETURN
```

### 2.3. Conexión con SQLite (`hbsqlit3`)

SQLite es una base de datos embebida, basada en un único fichero, lo que la hace ideal para aplicaciones de escritorio o como formato de fichero portable.

**Pasos para la conexión:**
1.  **Incluir el header:** `REQUEST HB_EXTENSION_HBSQLIT3` o `#include "hbsqlit3.ch"`
2.  **Abrir el fichero de la base de datos:** `nDb := sqlite3_open( "mi_base.db" )`
3.  **Ejecutar comandos:** `sqlite3_exec( nDb, cSQL )`
4.  **Para consultas SELECT, usar un enfoque preparado:**
    *   `sqlite3_prepare_v2()`
    *   `sqlite3_step()` para avanzar fila a fila.
    *   `sqlite3_column_*()` para obtener los datos de cada columna.
    *   `sqlite3_finalize()` para cerrar la consulta.
5.  **Cerrar la base de datos:** `sqlite3_close( nDb )`

**Ejemplo práctico:**
```prg
#include "hbsqlit3.ch"

PROCEDURE Main()
    LOCAL nDb, nStmt, nResult
    LOCAL cDbFile := "inventario.db"
    LOCAL cSQL

    // Abrir (o crear) la base de datos
    nDb := sqlite3_open( cDbFile )
    IF nDb == 0
        ? "No se pudo abrir la base de datos"
        RETURN
    ENDIF

    // Crear una tabla si no existe
    cSQL := "CREATE TABLE IF NOT EXISTS productos (id INTEGER PRIMARY KEY, nombre TEXT, precio REAL)"
    sqlite3_exec( nDb, cSQL )

    // Preparar una consulta SELECT
    cSQL := "SELECT nombre, precio FROM productos WHERE precio > ?"
    nStmt := sqlite3_prepare_v2( nDb, cSQL )

    // Vincular un parámetro (en este caso, el precio)
    sqlite3_bind_double( nStmt, 1, 50.0 )

    // Ejecutar la consulta paso a paso
    WHILE ( nResult := sqlite3_step( nStmt ) ) == SQLITE_ROW
        ? "Producto:", sqlite3_column_text( nStmt, 0 ), "Precio:", sqlite3_column_double( nStmt, 1 )
    END

    // Finalizar la consulta y liberar recursos
    sqlite3_finalize( nStmt )

    // Cerrar la base de datos
    sqlite3_close( nDb )
RETURN
```

## 3. Conceptos Fundamentales de SQL

Independientemente del SGBD que elijas, la interacción se realiza mediante el Lenguaje de Consulta Estructurado (SQL). Las cuatro operaciones básicas se conocen como **CRUD** (Create, Read, Update, Delete).

### `SELECT`: Leer datos

Se utiliza para recuperar datos de una o más tablas.

**Sintaxis básica:**
`SELECT columna1, columna2 FROM tabla WHERE condicion ORDER BY columna1;`

*   **`SELECT nombre, email FROM clientes`**: Obtiene solo el nombre y el email de todos los clientes.
*   **`SELECT * FROM productos WHERE categoria = 'Lacteos'`**: Obtiene todas las columnas de los productos que pertenecen a la categoría 'Lacteos'.
*   **`SELECT * FROM ventas ORDER BY fecha DESC`**: Obtiene todas las ventas ordenadas por fecha, de la más reciente a la más antigua.

### `INSERT`: Crear datos

Añade un nuevo registro (fila) a una tabla.

**Sintaxis básica:**
`INSERT INTO tabla (columna1, columna2) VALUES (valor1, valor2);`

*   **`INSERT INTO clientes (nombre, ciudad) VALUES ('Juan Perez', 'Madrid')`**: Añade un nuevo cliente.

**Importante:** En Harbour, es crucial escapar los valores de texto para prevenir inyección SQL. Las librerías de conexión suelen ofrecer funciones para ello (ej. `hb_mysql_real_escape_string()`).

### `UPDATE`: Actualizar datos

Modifica registros existentes en una tabla.

**Sintaxis básica:**
`UPDATE tabla SET columna1 = valor1 WHERE condicion;`

*   **`UPDATE productos SET precio = 25.50 WHERE id = 101`**: Cambia el precio del producto con ID 101.
*   **¡Cuidado!** Si omites la cláusula `WHERE`, ¡actualizarás **todos** los registros de la tabla!

### `DELETE`: Borrar datos

Elimina registros de una tabla.

**Sintaxis básica:**
`DELETE FROM tabla WHERE condicion;`

*   **`DELETE FROM pedidos WHERE estado = 'Cancelado'`**: Elimina todos los pedidos cancelados.
*   **¡Cuidado!** Al igual que con `UPDATE`, si omites la cláusula `WHERE`, ¡borrarás **todos** los datos de la tabla!
