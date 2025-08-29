# Capítulo 19: Los RDDs (`The RDDs`)

Este capítulo explora los Controladores de Bases de Datos Relacionales (RDDs), que constituyen la arquitectura fundamental de manejo de datos en Harbour.

---

## Controladores de bases de datos relacionales (`The RDDs`): la arquitectura de datos de Harbour

Los RDDs (Replaceable Database Drivers) son una característica distintiva de Harbour que proporciona una arquitectura flexible y extensible para el acceso a diferentes tipos de bases de datos y formatos de archivo.

### ¿Qué son los RDDs?

Los RDDs son controladores intercambiables que definen cómo Harbour accede y manipula los datos. Esta arquitectura permite:

- **Abstracción**: El mismo código puede trabajar con diferentes formatos de datos
- **Extensibilidad**: Se pueden crear nuevos RDDs para formatos específicos
- **Compatibilidad**: Soporte para múltiples formatos de bases de datos
- **Optimización**: Cada RDD puede optimizarse para su formato específico

### Arquitectura de los RDDs

```
┌─────────────────────────────┐
│     Aplicación Harbour      │
├─────────────────────────────┤
│      API de Base de Datos   │ ← USE, APPEND, REPLACE, etc.
├─────────────────────────────┤
│      Capa de RDD            │ ← Interfaz unificada
├─────────────────────────────┤
│  RDD Específico (DBF/SQL)   │ ← Implementación específica
├─────────────────────────────┤
│      Sistema de Archivos    │ ← Acceso físico a datos
└─────────────────────────────┘
```

### RDDs incluidos en Harbour

#### DBFNTX - Formato DBF con índices NTX

```harbour
// Usar RDD DBFNTX (por defecto)
REQUEST DBFNTX

FUNCTION EjemploDBFNTX()
   // Especificar RDD explícitamente
   USE "clientes.dbf" VIA "DBFNTX"
   
   // Crear índice NTX
   INDEX ON Upper(nombre) TO "clientes.ntx"
   
   // Buscar registro
   SEEK "JUAN"
   
   CLOSE ALL
   RETURN NIL
```

#### DBFCDX - Formato DBF con índices CDX

```harbour
REQUEST DBFCDX

FUNCTION EjemploDBFCDX()
   // Cambiar RDD por defecto
   RddSetDefault("DBFCDX")
   
   USE "productos.dbf"
   
   // Los índices CDX pueden ser compuestos
   INDEX ON codigo + descripcion TO "productos" TAG "codigo_desc"
   INDEX ON precio TO "productos" TAG "precio"
   
   // Cambiar de orden activo
   SET ORDER TO "precio"
   
   CLOSE ALL
   RETURN NIL
```

#### DBFFPT - DBF con campos Memo

```harbour
REQUEST DBFFPT

FUNCTION EjemploDBFFPT()
   // Crear estructura con campo memo
   LOCAL aEstructura := { ;
      {"CODIGO",      "C", 10, 0}, ;
      {"NOMBRE",      "C", 30, 0}, ;
      {"DESCRIPCION", "M", 10, 0}, ;  // Campo Memo
      {"PRECIO",      "N", 10, 2} ;
   }
   
   dbCreate("articulos.dbf", aEstructura, "DBFFPT")
   
   USE "articulos.dbf" VIA "DBFFPT"
   
   APPEND BLANK
   REPLACE codigo WITH "ART001"
   REPLACE nombre WITH "Producto 1"
   REPLACE descripcion WITH "Esta es una descripción muy larga que requiere un campo memo..."
   REPLACE precio WITH 99.99
   
   CLOSE ALL
   RETURN NIL
```

### RDDs para bases de datos SQL

#### SQLRDD - Acceso a bases de datos SQL

```harbour
REQUEST SQLRDD

FUNCTION EjemploSQLRDD()
   LOCAL cConnection := "Driver={MySQL ODBC 8.0};Server=localhost;Database=ventas;User=root;Password=123456;"
   
   // Conectar a base de datos SQL
   CONNECT TO (cConnection) AS "ventas"
   
   // Usar tabla SQL como si fuera DBF
   USE "SELECT * FROM clientes" VIA "SQLRDD" CONNECTION "ventas" ALIAS "clientes"
   
   // Navegar como cualquier tabla Harbour
   GO TOP
   WHILE !EOF()
      ? clientes->nombre, clientes->ciudad
      SKIP
   ENDDO
   
   CLOSE ALL
   DISCONNECT ALL
   RETURN NIL
```

### Funciones de gestión de RDDs

#### Consultar RDDs disponibles

```harbour
FUNCTION MostrarRDDsDisponibles()
   LOCAL aRDDs := RddList()
   LOCAL i
   
   ? "RDDs disponibles:"
   FOR i := 1 TO Len(aRDDs)
      ? i, aRDDs[i]
   NEXT
   
   ? "RDD por defecto:", RddSetDefault()
   
   RETURN NIL
```

#### Cambiar RDD por defecto

```harbour
FUNCTION ConfigurarRDD()
   // Guardar RDD actual
   LOCAL cRDDAnterior := RddSetDefault()
   
   // Cambiar a DBFCDX
   RddSetDefault("DBFCDX")
   
   // Trabajar con el nuevo RDD
   USE "datos.dbf"
   // ... operaciones ...
   CLOSE ALL
   
   // Restaurar RDD anterior si es necesario
   RddSetDefault(cRDDAnterior)
   
   RETURN NIL
```

### Crear un RDD personalizado

#### Estructura básica de un RDD

```c
// Ejemplo simplificado de estructura RDD en C
typedef struct _RDDFUNCS
{
   // Funciones básicas
   DBERROR   ( * Init )       ( LPRDDNODE );
   DBERROR   ( * Exit )       ( LPRDDNODE );
   DBERROR   ( * Open )       ( AREAP, LPDBOPENINFO );
   DBERROR   ( * Close )      ( AREAP );
   
   // Navegación
   DBERROR   ( * GoTo )       ( AREAP, ULONG );
   DBERROR   ( * GoTop )      ( AREAP );
   DBERROR   ( * GoBottom )   ( AREAP );
   DBERROR   ( * Skip )       ( AREAP, LONG );
   
   // Lectura/Escritura
   DBERROR   ( * GetValue )   ( AREAP, USHORT, PHB_ITEM );
   DBERROR   ( * PutValue )   ( AREAP, USHORT, PHB_ITEM );
   
   // ... más funciones
} RDDFUNCS;
```

#### Registrar RDD personalizado

```c
// Función para registrar el RDD
static BOOL hb_rddRegisterMiRDD( void )
{
   RDDFUNCS * pTable;
   USHORT uiCount = 0;
   
   // Crear tabla de funciones
   pTable = ( RDDFUNCS * ) hb_xgrab( sizeof( RDDFUNCS ) );
   memset( pTable, 0, sizeof( RDDFUNCS ) );
   
   // Asignar funciones
   pTable->Init = hb_miRDDInit;
   pTable->Exit = hb_miRDDExit;
   pTable->Open = hb_miRDDOpen;
   // ... más asignaciones
   
   // Registrar RDD
   return hb_rddRegister( "MIRDD", RDT_TRANSFER, pTable, uiCount );
}
```

### Optimización y configuración de RDDs

#### Configuración de buffers

```harbour
FUNCTION ConfigurarBuffers()
   // Configurar tamaño de buffer para lecturas
   SET DBFLOCKSCHEME TO DB_DBFLOCK_HB64
   
   // Configurar modo de bloqueo
   SET LOCK MODE TO EXTENDED
   
   // Configurar timeout para bloqueos
   SET LOCKTIMEOUT TO 5
   
   RETURN NIL
```

#### Optimización de índices

```harbour
FUNCTION OptimizarIndices()
   USE "ventas.dbf" VIA "DBFCDX"
   
   // Configurar optimización de índices
   SET OPTIMIZE ON
   
   // Reindexar para optimizar
   REINDEX
   
   // Verificar integridad
   IF !DbCheckIntegrity()
      ? "Advertencia: Problemas de integridad detectados"
   ENDIF
   
   CLOSE ALL
   RETURN NIL
```

### Trabajo con múltiples RDDs

#### Mezclar diferentes RDDs

```harbour
FUNCTION EjemploMultiplesRDDs()
   // Abrir tabla DBF tradicional
   SELECT 1
   USE "clientes.dbf" VIA "DBFNTX"
   
   // Abrir datos desde SQL
   SELECT 2
   CONNECT TO "DSN=MiBaseDatos" AS "sql_conn"
   USE "SELECT * FROM pedidos" VIA "SQLRDD" CONNECTION "sql_conn"
   
   // Relacionar datos
   SELECT clientes
   SET RELATION TO codigo INTO pedidos
   
   // Procesar datos relacionados
   GO TOP
   WHILE !EOF()
      ? clientes->nombre
      
      SELECT pedidos
      WHILE pedidos->cliente_codigo == clientes->codigo .AND. !EOF()
         ? "  Pedido:", pedidos->numero, pedidos->fecha
         SKIP
      ENDDO
      
      SELECT clientes
      SKIP
   ENDDO
   
   CLOSE ALL
   DISCONNECT ALL
   RETURN NIL
```

### Migración entre RDDs

#### Convertir de DBF a SQL

```harbour
FUNCTION MigrarDBFaSQL()
   // Abrir tabla DBF origen
   USE "clientes_old.dbf" VIA "DBFNTX" ALIAS "origen"
   
   // Conectar a base de datos SQL destino
   CONNECT TO "DSN=MiBaseDatos" AS "destino"
   
   // Crear tabla en SQL
   SQLExec("destino", "CREATE TABLE clientes_new (codigo VARCHAR(10), nombre VARCHAR(50), ciudad VARCHAR(30))")
   
   // Migrar datos
   GO TOP
   WHILE !EOF()
      LOCAL cSQL := "INSERT INTO clientes_new VALUES ('" + ;
                    origen->codigo + "', '" + ;
                    origen->nombre + "', '" + ;
                    origen->ciudad + "')"
      
      SQLExec("destino", cSQL)
      SKIP
   ENDDO
   
   CLOSE ALL
   DISCONNECT ALL
   
   ? "Migración completada"
   RETURN NIL
```

### Manejo de errores en RDDs

```harbour
FUNCTION ManejoErroresRDD()
   LOCAL bErrorHandler := ErrorBlock({|e| MiErrorHandler(e)})
   LOCAL lExito := .F.
   
   BEGIN SEQUENCE
      USE "archivo_problematico.dbf"
      
      // Operaciones con la base de datos
      GO TOP
      REPLACE campo WITH "nuevo valor"
      
      lExito := .T.
      
   RECOVER
      ? "Error al acceder a la base de datos"
      
   END SEQUENCE
   
   ErrorBlock(bErrorHandler)
   
   IF lExito
      CLOSE ALL
   ENDIF
   
   RETURN lExito

FUNCTION MiErrorHandler(oError)
   LOCAL cMensaje := "Error RDD: " + oError:Description
   
   DO CASE
   CASE oError:GenCode == EG_OPEN
      cMensaje += " - No se pudo abrir el archivo"
   CASE oError:GenCode == EG_CORRUPTION
      cMensaje += " - Archivo corrupto"
   CASE oError:GenCode == EG_LOCK
      cMensaje += " - Error de bloqueo"
   ENDCASE
   
   Alert(cMensaje)
   BREAK  // Continuar con RECOVER
   
   RETURN NIL
```

Los RDDs proporcionan la flexibilidad necesaria para trabajar con diversos formatos de datos mientras se mantiene una interfaz de programación consistente, siendo una de las características más poderosas de Harbour para el manejo de datos.