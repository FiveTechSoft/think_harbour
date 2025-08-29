# Capítulo 19: Los RDDs (`The RDDs`)

Los RDDs (Replaceable Database Drivers) constituyen el núcleo del sistema de gestión de datos de Harbour. Este sistema modular permite trabajar con diferentes formatos de base de datos de manera transparente, desde los tradicionales archivos DBF hasta sistemas de bases de datos SQL modernos, manteniendo una API consistente y familiar.

---

## Controladores de bases de datos relacionales (`The RDDs`): la arquitectura de datos de Harbour

### Arquitectura general de RDDs

Los RDDs proporcionan una capa de abstracción entre el código de aplicación y el almacenamiento físico de datos:

```
Aplicación Harbour
       ↓
   API RDD estándar
       ↓
┌─────────────────────────────┐
│  Capa de abstracción RDD    │
├─────────────────────────────┤
│ DBFNTX │ DBFCDX │ DBFNSX   │ ← RDDs para archivos locales
│ SQLRDD │ ADORDS │ ODBCRDD  │ ← RDDs para bases de datos SQL
│ Custom │ Memory │ NetRDD   │ ← RDDs especializados
└─────────────────────────────┘
       ↓
Almacenamiento físico
```

### Componentes de un RDD

Cada RDD implementa un conjunto estándar de funciones:

```c
// Estructura básica de un RDD
typedef struct _RDDFUNCS
{
   // Operaciones de área de trabajo
   ERRCODE (*Open)(AREAP pArea, LPDBOPENINFO pOpenInfo);
   ERRCODE (*Close)(AREAP pArea);
   ERRCODE (*Create)(AREAP pArea, LPDBOPENINFO pCreateInfo);
   
   // Navegación de registros
   ERRCODE (*GoTo)(AREAP pArea, HB_ULONG ulRecNo);
   ERRCODE (*GoTop)(AREAP pArea);
   ERRCODE (*GoBottom)(AREAP pArea);
   ERRCODE (*Skip)(AREAP pArea, HB_LONG lToSkip);
   
   // Manipulación de datos
   ERRCODE (*GetValue)(AREAP pArea, HB_USHORT uiIndex, PHB_ITEM pItem);
   ERRCODE (*PutValue)(AREAP pArea, HB_USHORT uiIndex, PHB_ITEM pItem);
   ERRCODE (*Append)(AREAP pArea, HB_BOOL fUnLockAll);
   ERRCODE (*Delete)(AREAP pArea);
   
   // Filtros y índices
   ERRCODE (*SetFilter)(AREAP pArea, PHB_ITEM pFilter);
   ERRCODE (*OrderCreate)(AREAP pArea, LPDBORDERCREATEINFO pOrderInfo);
   ERRCODE (*OrderListAdd)(AREAP pArea, LPDBORDERINFO pOrderInfo);
   
   // ... y muchas más funciones
} RDDFUNCS, *PRDDFUNCS;
```

---

## RDDs nativos de Harbour

### DBFNTX - Archivos DBF con índices NTX

El RDD más tradicional, compatible con Clipper:

```harbour
FUNCTION EjemploDBFNTX()
   LOCAL aEstructura := {;
      {"ID",       "N", 8, 0},;
      {"NOMBRE",   "C", 30, 0},;
      {"APELLIDO", "C", 30, 0},;
      {"EDAD",     "N", 3, 0},;
      {"ACTIVO",   "L", 1, 0},;
      {"FECHA",    "D", 8, 0};
   }
   
   // Usar RDD DBFNTX explícitamente
   RddSetDefault("DBFNTX")
   
   // Crear archivo
   DbCreate("empleados.dbf", aEstructura)
   
   // Abrir y usar
   USE empleados
   
   // Crear índice NTX
   INDEX ON UPPER(NOMBRE + APELLIDO) TO empleados TAG "NOMBRE_COMPLETO"
   INDEX ON STR(ID, 8) TO empleados_id TAG "ID"
   
   // Agregar datos
   APPEND BLANK
   REPLACE ID WITH 1, NOMBRE WITH "Juan", APELLIDO WITH "Pérez"
   REPLACE EDAD WITH 30, ACTIVO WITH .T., FECHA WITH Date()
   
   CLOSE ALL
   
   RETURN NIL
```

### DBFCDX - Archivos DBF con índices CDX

Más eficiente para aplicaciones con múltiples índices:

```harbour
FUNCTION EjemploDBFCDX()
   RddSetDefault("DBFCDX")
   
   // Estructura más compleja
   LOCAL aEstructura := {;
      {"CODIGO",     "C", 10, 0},;
      {"DESCRIPCION","C", 50, 0},;
      {"PRECIO",     "N", 12, 2},;
      {"STOCK",      "N", 8, 0},;
      {"CATEGORIA",  "C", 20, 0},;
      {"PROVEEDOR",  "N", 6, 0},;
      {"ACTIVO",     "L", 1, 0};
   }
   
   DbCreate("productos.dbf", aEstructura)
   USE productos
   
   // Crear índice compuesto CDX
   INDEX ON CODIGO TAG CODIGO
   INDEX ON UPPER(DESCRIPCION) TAG DESCRIPCION
   INDEX ON CATEGORIA + CODIGO TAG CAT_CODIGO
   INDEX ON PRECIO TAG PRECIO DESCENDING
   INDEX ON STOCK TAG STOCK FOR ACTIVO  // Índice condicional
   
   // Los índices se almacenan en productos.cdx
   
   // Agregar datos de ejemplo
   LOCAL i
   FOR i := 1 TO 100
      APPEND BLANK
      REPLACE CODIGO WITH "PROD" + StrZero(i, 4)
      REPLACE DESCRIPCION WITH "Producto " + LTrim(Str(i))
      REPLACE PRECIO WITH Random(1000) + Random(99)/100
      REPLACE STOCK WITH Random(500)
      REPLACE CATEGORIA WITH Choose(Random(5), "A", "B", "C", "D", "E")
      REPLACE PROVEEDOR WITH Random(50)
      REPLACE ACTIVO WITH (Random(10) > 2)
   NEXT
   
   // Uso de índices
   SET INDEX TO productos  // Usar todos los índices del CDX
   SET ORDER TO DESCRIPCION
   
   SEEK "PRODUCTO 50"
   IF Found()
      ? "Encontrado:", CODIGO, DESCRIPCION, PRECIO
   ENDIF
   
   CLOSE ALL
   RETURN NIL
```

### DBFNSX - Compatible con FoxPro

```harbour
FUNCTION EjemploDBFNSX()
   RddSetDefault("DBFNSX")
   
   // Usar características específicas de NSX
   LOCAL aEstructura := {;
      {"ID",         "I", 4, 0},;   // Integer (4 bytes)
      {"TIMESTAMP",  "T", 8, 0},;   // DateTime
      {"MEMO",       "M", 10, 0},;  // Memo field
      {"CURRENCY",   "Y", 8, 4},;   // Currency
      {"DOUBLE",     "B", 8, 0};    // Double
   }
   
   DbCreate("datos_nsx.dbf", aEstructura)
   USE datos_nsx
   
   // Crear índices NSX con expresiones complejas
   INDEX ON ID TAG ID
   INDEX ON TIMESTAMP TAG FECHA
   INDEX ON YEAR(TIMESTAMP) * 100 + MONTH(TIMESTAMP) TAG ANO_MES
   
   CLOSE ALL
   RETURN NIL
```

---

## RDDs para bases de datos SQL

### SQLRDD - Acceso SQL genérico

```harbour
FUNCTION EjemploSQLRDD()
   LOCAL cConnectionString
   
   // Configurar conexión
   cConnectionString := "Driver={MySQL ODBC 8.0 Driver};" + ;
                       "Server=localhost;" + ;
                       "Database=mi_empresa;" + ;
                       "User=usuario;" + ;
                       "Password=clave;"
   
   // Usar SQLRDD
   RddSetDefault("SQLRDD")
   
   // Configurar conexión SQL
   rddInfo(RDDI_CONNECT, cConnectionString)
   
   // Abrir tabla SQL como si fuera DBF
   USE empleados_sql VIA "SQLRDD"
   
   // Operaciones estándar
   GO TOP
   WHILE !EOF()
      ? NOMBRE, APELLIDO, SALARIO
      SKIP
   ENDDO
   
   // Búsqueda optimizada (se traduce a WHERE en SQL)
   SEEK "Juan"
   IF Found()
      ? "Empleado encontrado:", NOMBRE, APELLIDO
   ENDIF
   
   // Agregar registro (se traduce a INSERT)
   APPEND BLANK
   REPLACE NOMBRE WITH "Carlos", APELLIDO WITH "González"
   REPLACE SALARIO WITH 45000
   
   CLOSE ALL
   RETURN NIL
```

### Configuración avanzada de SQLRDD

```harbour
FUNCTION ConfigurarSQLRDD()
   // Configurar opciones específicas del RDD
   rddInfo(RDDI_AUTOOPEN, .T.)        // Abrir conexiones automáticamente
   rddInfo(RDDI_AUTOCOMMIT, .F.)      // Control manual de transacciones
   rddInfo(RDDI_QUERYTIMEOUT, 30)     // Timeout de consultas en segundos
   
   // Configurar mapeo de tipos de datos
   rddInfo(RDDI_FIELDASTYPE, {"VARCHAR" => "C", "INTEGER" => "N"})
   
   // Configurar SQL personalizado
   rddInfo(RDDI_CUSTOMSQL, {;
      "SELECT" => "SELECT {FIELDS} FROM {TABLE} {WHERE} {ORDER}",;
      "INSERT" => "INSERT INTO {TABLE} ({FIELDS}) VALUES ({VALUES})",;
      "UPDATE" => "UPDATE {TABLE} SET {ASSIGNMENTS} WHERE {CONDITION}",;
      "DELETE" => "DELETE FROM {TABLE} WHERE {CONDITION}";
   })
   
   RETURN NIL
```

---

## RDDs especializados

### MEMORDD - Tablas en memoria

```harbour
FUNCTION EjemploMemoryRDD()
   LOCAL aEstructura := {;
      {"ID", "N", 8, 0},;
      {"NOMBRE", "C", 30, 0},;
      {"TEMP", "N", 10, 2};
   }
   
   // Crear tabla temporal en memoria
   RddSetDefault("MEMORDD")
   DbCreate("TEMP_DATA", aEstructura, "MEMORDD")
   
   USE TEMP_DATA
   
   // Llenar con datos temporales
   LOCAL i
   FOR i := 1 TO 1000
      APPEND BLANK
      REPLACE ID WITH i
      REPLACE NOMBRE WITH "Temp" + LTrim(Str(i))
      REPLACE TEMP WITH Random(1000) / 10
   NEXT
   
   // Crear índice en memoria
   INDEX ON NOMBRE TAG NOMBRE
   INDEX ON TEMP TAG TEMP_ORDER
   
   // Procesar datos rápidamente
   SET ORDER TO TEMP_ORDER
   GO TOP
   ? "Mayor valor:", TEMP
   
   GO BOTTOM
   ? "Menor valor:", TEMP
   
   CLOSE ALL
   RETURN NIL
```

### RDD personalizado

```harbour
// Crear un RDD personalizado para archivos CSV
FUNCTION RegistrarCSVRDD()
   // Registrar nuevo RDD
   rddRegister("CSVRDD", 1)
   
   // Configurar funciones específicas
   rddInfo(RDDI_TABLEEXT, "csv")
   rddInfo(RDDI_DELIMITER, ",")
   rddInfo(RDDI_QUALIFIER, '"')
   
   RETURN NIL

FUNCTION EjemploCSVRDD()
   RegistrarCSVRDD()
   
   // Usar el RDD personalizado
   RddSetDefault("CSVRDD")
   
   // Abrir archivo CSV como tabla
   USE "datos.csv"
   
   // Procesar como tabla normal
   GO TOP
   WHILE !EOF()
      ? FieldGet(1), FieldGet(2), FieldGet(3)
      SKIP
   ENDDO
   
   CLOSE ALL
   RETURN NIL
```

---

## Gestión avanzada de RDDs

### Múltiples RDDs simultáneamente

```harbour
FUNCTION MultiplesRDDs()
   // Abrir diferentes tipos de datos simultáneamente
   
   // Tabla local DBF
   SELECT 1
   RddSetDefault("DBFCDX")
   USE clientes
   
   // Tabla SQL remota
   SELECT 2
   RddSetDefault("SQLRDD")
   rddInfo(RDDI_CONNECT, cConnectionString)
   USE pedidos
   
   // Tabla temporal en memoria
   SELECT 3
   RddSetDefault("MEMORDD")
   DbCreate("TEMP_CALC", aEstructuraCalculo, "MEMORDD")
   USE TEMP_CALC
   
   // Operaciones entre diferentes fuentes
   SELECT clientes
   GO TOP
   WHILE !EOF()
      SELECT pedidos
      SEEK clientes->ID
      
      LOCAL nTotalPedidos := 0
      WHILE Found() .AND. pedidos->CLIENTE_ID == clientes->ID
         nTotalPedidos += pedidos->MONTO
         SKIP
      ENDDO
      
      SELECT TEMP_CALC
      APPEND BLANK
      REPLACE CLIENTE_ID WITH clientes->ID
      REPLACE TOTAL_PEDIDOS WITH nTotalPedidos
      
      SELECT clientes
      SKIP
   ENDDO
   
   CLOSE ALL
   RETURN NIL
```

### Optimización de rendimiento

```harbour
FUNCTION OptimizarRDD()
   // Configurar buffer de red para RDDs remotos
   rddInfo(RDDI_BUFFERSIZE, 8192)
   
   // Configurar cache de registros
   rddInfo(RDDI_CACHESIZE, 100)
   
   // Deshabilitar autoflush para operaciones batch
   rddInfo(RDDI_AUTOFLUSH, .F.)
   
   // Usar transacciones para operaciones múltiples
   BEGIN TRANSACTION
   
   LOCAL i
   FOR i := 1 TO 10000
      APPEND BLANK
      REPLACE CAMPO1 WITH "Valor" + LTrim(Str(i))
      REPLACE CAMPO2 WITH i * 1.5
   NEXT
   
   COMMIT TRANSACTION
   
   // Reactivar autoflush
   rddInfo(RDDI_AUTOFLUSH, .T.)
   
   RETURN NIL
```

### Manejo de errores en RDDs

```harbour
FUNCTION ManejoErroresRDD()
   LOCAL oError
   
   BEGIN SEQUENCE
      
      USE tabla_inexistente
      
   RECOVER USING oError
      
      DO CASE
      CASE oError:genCode == EG_OPEN
         ? "Error abriendo tabla:", oError:description
         ? "Archivo:", oError:filename
         
      CASE oError:genCode == EG_CREATE
         ? "Error creando tabla:", oError:description
         
      CASE oError:genCode == EG_CORRUPTION
         ? "Tabla corrupta:", oError:filename
         ? "Intentando reparar..."
         RepararTabla(oError:filename)
         
      OTHERWISE
         ? "Error RDD desconocido:", oError:description
      ENDCASE
      
   END SEQUENCE
   
   RETURN NIL

STATIC FUNCTION RepararTabla(cTabla)
   LOCAL cComando
   
   // Ejemplo de reparación para DBF
   IF Right(Upper(cTabla), 4) == ".DBF"
      // Usar utilidad externa o RDD específico para reparar
      cComando := "REINDEX " + cTabla
      // Ejecutar comando de reparación
   ENDIF
   
   RETURN NIL
```

### Hooks y eventos en RDDs

```harbour
// Sistema de eventos para operaciones RDD
STATIC s_aRDDHooks := {}

FUNCTION AgregarHookRDD(cEvento, bAccion)
   AAdd(s_aRDDHooks, {cEvento, bAccion})
   RETURN Len(s_aRDDHooks)

FUNCTION EjecutarHookRDD(cEvento, aParametros)
   LOCAL i, aHook
   
   FOR i := 1 TO Len(s_aRDDHooks)
      aHook := s_aRDDHooks[i]
      IF aHook[1] == cEvento
         Eval(aHook[2], aParametros)
      ENDIF
   NEXT
   
   RETURN NIL

// Configurar hooks de auditoría
FUNCTION ConfigurarAuditoriaRDD()
   AgregarHookRDD("BEFORE_APPEND", {|a| LogOperacion("APPEND", RecNo())})
   AgregarHookRDD("AFTER_REPLACE", {|a| LogOperacion("REPLACE", RecNo(), a)})
   AgregarHookRDD("BEFORE_DELETE", {|a| LogOperacion("DELETE", RecNo())})
   
   RETURN NIL
```

Los RDDs proporcionan la flexibilidad necesaria para trabajar con cualquier tipo de almacenamiento de datos manteniendo una interfaz consistente, lo que permite migrar entre diferentes tecnologías de base de datos con cambios mínimos en el código de aplicación.