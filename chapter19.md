# Chapter 19: The RDDs (Replaceable Database Drivers)

RDDs (Replaceable Database Drivers) form the core of Harbour's data management system. This modular system allows working with different database formats transparently, from traditional DBF files to modern SQL database systems, maintaining a consistent and familiar API.

---

## Relational Database Drivers (The RDDs): Harbour's data architecture

### General RDD architecture

RDDs provide an abstraction layer between application code and physical data storage:

```
Harbour Application
       ↓
   Standard RDD API
       ↓
┌─────────────────────────────┐
│  RDD abstraction layer      │
├─────────────────────────────┤
│ DBFNTX │ DBFCDX │ DBFNSX   │ ← RDDs for local files
│ SQLRDD │ ADORDS │ ODBCRDD  │ ← RDDs for SQL databases
│ Custom │ Memory │ NetRDD   │ ← Specialized RDDs
└─────────────────────────────┘
       ↓
Physical storage
```

### RDD components

Each RDD implements a standard set of functions:

```c
// Basic RDD structure
typedef struct _RDDFUNCS
{
   // Work area operations
   ERRCODE (*Open)(AREAP pArea, LPDBOPENINFO pOpenInfo);
   ERRCODE (*Close)(AREAP pArea);
   ERRCODE (*Create)(AREAP pArea, LPDBOPENINFO pCreateInfo);
   
   // Record navigation
   ERRCODE (*GoTo)(AREAP pArea, HB_ULONG ulRecNo);
   ERRCODE (*GoTop)(AREAP pArea);
   ERRCODE (*GoBottom)(AREAP pArea);
   ERRCODE (*Skip)(AREAP pArea, HB_LONG lToSkip);
   
   // Data manipulation
   ERRCODE (*GetValue)(AREAP pArea, HB_USHORT uiIndex, PHB_ITEM pItem);
   ERRCODE (*PutValue)(AREAP pArea, HB_USHORT uiIndex, PHB_ITEM pItem);
   ERRCODE (*Append)(AREAP pArea, HB_BOOL fUnLockAll);
   ERRCODE (*Delete)(AREAP pArea);
   
   // Filters and indexes
   ERRCODE (*SetFilter)(AREAP pArea, PHB_ITEM pFilter);
   ERRCODE (*OrderCreate)(AREAP pArea, LPDBORDERCREATEINFO pOrderInfo);
   ERRCODE (*OrderListAdd)(AREAP pArea, LPDBORDERINFO pOrderInfo);
   
   // ... and many more functions
} RDDFUNCS, *PRDDFUNCS;
```

---

## Native Harbour RDDs

### DBFNTX - DBF files with NTX indexes

The most traditional RDD, compatible with Clipper:

```harbour
FUNCTION ExampleDBFNTX()
   LOCAL aStructure := {;
      {"ID",       "N", 8, 0},;
      {"NAME",     "C", 30, 0},;
      {"LASTNAME", "C", 30, 0},;
      {"AGE",      "N", 3, 0},;
      {"ACTIVE",   "L", 1, 0},;
      {"DATE",     "D", 8, 0};
   }
   
   // Use DBFNTX RDD explicitly
   RddSetDefault("DBFNTX")
   
   // Create file
   DbCreate("employees.dbf", aStructure)
   
   // Open and use
   USE employees
   
   // Create NTX index
   INDEX ON UPPER(NAME + LASTNAME) TO employees TAG "FULL_NAME"
   INDEX ON STR(ID, 8) TO employees_id TAG "ID"
   
   // Add data
   APPEND BLANK
   REPLACE ID WITH 1, NAME WITH "John", LASTNAME WITH "Doe"
   REPLACE AGE WITH 30, ACTIVE WITH .T., DATE WITH Date()
   
   CLOSE ALL
   
   RETURN NIL
```

### DBFCDX - DBF files with CDX indexes

More efficient for applications with multiple indexes:

```harbour
FUNCTION ExampleDBFCDX()
   RddSetDefault("DBFCDX")
   
   // More complex structure
   LOCAL aStructure := {;
      {"CODE",        "C", 10, 0},;
      {"DESCRIPTION", "C", 50, 0},;
      {"PRICE",       "N", 12, 2},;
      {"STOCK",       "N", 8, 0},;
      {"CATEGORY",    "C", 20, 0},;
      {"SUPPLIER",    "N", 6, 0},;
      {"ACTIVE",      "L", 1, 0};
   }
   
   DbCreate("products.dbf", aStructure)
   USE products
   
   // Create compound CDX index
   INDEX ON CODE TAG CODE
   INDEX ON UPPER(DESCRIPTION) TAG DESCRIPTION
   INDEX ON CATEGORY + CODE TAG CAT_CODE
   INDEX ON PRICE TAG PRICE DESCENDING
   INDEX ON STOCK TAG STOCK FOR ACTIVE  // Conditional index
   
   // Indexes are stored in products.cdx
   
   // Add sample data
   LOCAL i
   FOR i := 1 TO 100
      APPEND BLANK
      REPLACE CODE WITH "PROD" + StrZero(i, 4)
      REPLACE DESCRIPTION WITH "Product " + LTrim(Str(i))
      REPLACE PRICE WITH Random(1000) + Random(99)/100
      REPLACE STOCK WITH Random(500)
      REPLACE CATEGORY WITH Choose(Random(5), "A", "B", "C", "D", "E")
      REPLACE SUPPLIER WITH Random(50)
      REPLACE ACTIVE WITH (Random(10) > 2)
   NEXT
   
   // Using indexes
   SET INDEX TO products  // Use all indexes from CDX
   SET ORDER TO DESCRIPTION
   
   SEEK "PRODUCT 50"
   IF Found()
      ? "Found:", CODE, DESCRIPTION, PRICE
   ENDIF
   
   CLOSE ALL
   RETURN NIL
```

### DBFNSX - FoxPro compatible

```harbour
FUNCTION ExampleDBFNSX()
   RddSetDefault("DBFNSX")
   
   // Use NSX-specific features
   LOCAL aStructure := {;
      {"ID",         "I", 4, 0},;   // Integer (4 bytes)
      {"TIMESTAMP",  "T", 8, 0},;   // DateTime
      {"MEMO",       "M", 10, 0},;  // Memo field
      {"CURRENCY",   "Y", 8, 4},;   // Currency
      {"DOUBLE",     "B", 8, 0};    // Double
   }
   
   DbCreate("nsx_data.dbf", aStructure)
   USE nsx_data
   
   // Create NSX indexes with complex expressions
   INDEX ON ID TAG ID
   INDEX ON TIMESTAMP TAG DATE
   INDEX ON YEAR(TIMESTAMP) * 100 + MONTH(TIMESTAMP) TAG YEAR_MONTH
   
   CLOSE ALL
   RETURN NIL
```

---

## RDDs for SQL databases

### SQLRDD - Generic SQL access

```harbour
FUNCTION ExampleSQLRDD()
   LOCAL cConnectionString
   
   // Configure connection
   cConnectionString := "Driver={MySQL ODBC 8.0 Driver};" + ;
                       "Server=localhost;" + ;
                       "Database=my_company;" + ;
                       "User=user;" + ;
                       "Password=password;"
   
   // Use SQLRDD
   RddSetDefault("SQLRDD")
   
   // Configure SQL connection
   rddInfo(RDDI_CONNECT, cConnectionString)
   
   // Open SQL table as if it were DBF
   USE employees_sql VIA "SQLRDD"
   
   // Standard operations
   GO TOP
   WHILE !EOF()
      ? NAME, LASTNAME, SALARY
      SKIP
   ENDDO
   
   // Optimized search (translates to WHERE in SQL)
   SEEK "John"
   IF Found()
      ? "Employee found:", NAME, LASTNAME
   ENDIF
   
   // Add record (translates to INSERT)
   APPEND BLANK
   REPLACE NAME WITH "Carlos", LASTNAME WITH "González"
   REPLACE SALARY WITH 45000
   
   CLOSE ALL
   RETURN NIL
```

### Advanced SQLRDD configuration

```harbour
FUNCTION ConfigureSQLRDD()
   // Configure RDD-specific options
   rddInfo(RDDI_AUTOOPEN, .T.)        // Open connections automatically
   rddInfo(RDDI_AUTOCOMMIT, .F.)      // Manual transaction control
   rddInfo(RDDI_QUERYTIMEOUT, 30)     // Query timeout in seconds
   
   // Configure data type mapping
   rddInfo(RDDI_FIELDASTYPE, {"VARCHAR" => "C", "INTEGER" => "N"})
   
   // Configure custom SQL
   rddInfo(RDDI_CUSTOMSQL, {;
      "SELECT" => "SELECT {FIELDS} FROM {TABLE} {WHERE} {ORDER}",;
      "INSERT" => "INSERT INTO {TABLE} ({FIELDS}) VALUES ({VALUES})",;
      "UPDATE" => "UPDATE {TABLE} SET {ASSIGNMENTS} WHERE {CONDITION}",;
      "DELETE" => "DELETE FROM {TABLE} WHERE {CONDITION}";
   })
   
   RETURN NIL
```

---

## Specialized RDDs

### MEMORDD - In-memory tables

```harbour
FUNCTION ExampleMemoryRDD()
   LOCAL aStructure := {;
      {"ID", "N", 8, 0},;
      {"NAME", "C", 30, 0},;
      {"TEMP", "N", 10, 2};
   }
   
   // Create temporary table in memory
   RddSetDefault("MEMORDD")
   DbCreate("TEMP_DATA", aStructure, "MEMORDD")
   
   USE TEMP_DATA
   
   // Fill with temporary data
   LOCAL i
   FOR i := 1 TO 1000
      APPEND BLANK
      REPLACE ID WITH i
      REPLACE NAME WITH "Temp" + LTrim(Str(i))
      REPLACE TEMP WITH Random(1000) / 10
   NEXT
   
   // Create index in memory
   INDEX ON NAME TAG NAME
   INDEX ON TEMP TAG TEMP_ORDER
   
   // Process data quickly
   SET ORDER TO TEMP_ORDER
   GO TOP
   ? "Highest value:", TEMP
   
   GO BOTTOM
   ? "Lowest value:", TEMP
   
   CLOSE ALL
   RETURN NIL
```

### Custom RDD

```harbour
// Create custom RDD for CSV files
FUNCTION RegisterCSVRDD()
   // Register new RDD
   rddRegister("CSVRDD", 1)
   
   // Configure specific functions
   rddInfo(RDDI_TABLEEXT, "csv")
   rddInfo(RDDI_DELIMITER, ",")
   rddInfo(RDDI_QUALIFIER, '"')
   
   RETURN NIL

FUNCTION ExampleCSVRDD()
   RegisterCSVRDD()
   
   // Use custom RDD
   RddSetDefault("CSVRDD")
   
   // Open CSV file as table
   USE "data.csv"
   
   // Process as normal table
   GO TOP
   WHILE !EOF()
      ? FieldGet(1), FieldGet(2), FieldGet(3)
      SKIP
   ENDDO
   
   CLOSE ALL
   RETURN NIL
```

---

## Advanced RDD management

### Multiple RDDs simultaneously

```harbour
FUNCTION MultipleRDDs()
   // Open different types of data simultaneously
   
   // Local DBF table
   SELECT 1
   RddSetDefault("DBFCDX")
   USE clients
   
   // Remote SQL table
   SELECT 2
   RddSetDefault("SQLRDD")
   rddInfo(RDDI_CONNECT, cConnectionString)
   USE orders
   
   // Temporary table in memory
   SELECT 3
   RddSetDefault("MEMORDD")
   DbCreate("TEMP_CALC", aCalculationStructure, "MEMORDD")
   USE TEMP_CALC
   
   // Operations between different sources
   SELECT clients
   GO TOP
   WHILE !EOF()
      SELECT orders
      SEEK clients->ID
      
      LOCAL nTotalOrders := 0
      WHILE Found() .AND. orders->CLIENT_ID == clients->ID
         nTotalOrders += orders->AMOUNT
         SKIP
      ENDDO
      
      SELECT TEMP_CALC
      APPEND BLANK
      REPLACE CLIENT_ID WITH clients->ID
      REPLACE TOTAL_ORDERS WITH nTotalOrders
      
      SELECT clients
      SKIP
   ENDDO
   
   CLOSE ALL
   RETURN NIL
```

### Performance optimization

```harbour
FUNCTION OptimizeRDD()
   // Configure network buffer for remote RDDs
   rddInfo(RDDI_BUFFERSIZE, 8192)
   
   // Configure record cache
   rddInfo(RDDI_CACHESIZE, 100)
   
   // Disable autoflush for batch operations
   rddInfo(RDDI_AUTOFLUSH, .F.)
   
   // Use transactions for multiple operations
   BEGIN TRANSACTION
   
   LOCAL i
   FOR i := 1 TO 10000
      APPEND BLANK
      REPLACE FIELD1 WITH "Value" + LTrim(Str(i))
      REPLACE FIELD2 WITH i * 1.5
   NEXT
   
   COMMIT TRANSACTION
   
   // Reactivate autoflush
   rddInfo(RDDI_AUTOFLUSH, .T.)
   
   RETURN NIL
```

### Error handling in RDDs

```harbour
FUNCTION RDDErrorHandling()
   LOCAL oError
   
   BEGIN SEQUENCE
      
      USE nonexistent_table
      
   RECOVER USING oError
      
      DO CASE
      CASE oError:genCode == EG_OPEN
         ? "Error opening table:", oError:description
         ? "File:", oError:filename
         
      CASE oError:genCode == EG_CREATE
         ? "Error creating table:", oError:description
         
      CASE oError:genCode == EG_CORRUPTION
         ? "Corrupted table:", oError:filename
         ? "Attempting repair..."
         RepairTable(oError:filename)
         
      OTHERWISE
         ? "Unknown RDD error:", oError:description
      ENDCASE
      
   END SEQUENCE
   
   RETURN NIL

STATIC FUNCTION RepairTable(cTable)
   LOCAL cCommand
   
   // Example repair for DBF
   IF Right(Upper(cTable), 4) == ".DBF"
      // Use external utility or specific RDD to repair
      cCommand := "REINDEX " + cTable
      // Execute repair command
   ENDIF
   
   RETURN NIL
```

### Hooks and events in RDDs

```harbour
// Event system for RDD operations
STATIC s_aRDDHooks := {}

FUNCTION AddRDDHook(cEvent, bAction)
   AAdd(s_aRDDHooks, {cEvent, bAction})
   RETURN Len(s_aRDDHooks)

FUNCTION ExecuteRDDHook(cEvent, aParameters)
   LOCAL i, aHook
   
   FOR i := 1 TO Len(s_aRDDHooks)
      aHook := s_aRDDHooks[i]
      IF aHook[1] == cEvent
         Eval(aHook[2], aParameters)
      ENDIF
   NEXT
   
   RETURN NIL

// Configure audit hooks
FUNCTION ConfigureRDDAudit()
   AddRDDHook("BEFORE_APPEND", {|a| LogOperation("APPEND", RecNo())})
   AddRDDHook("AFTER_REPLACE", {|a| LogOperation("REPLACE", RecNo(), a)})
   AddRDDHook("BEFORE_DELETE", {|a| LogOperation("DELETE", RecNo())})
   
   RETURN NIL
```

RDDs provide the necessary flexibility to work with any type of data storage while maintaining a consistent interface, allowing migration between different database technologies with minimal changes to application code.