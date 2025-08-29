# Capítulo 10: Conectividad y bases de datos avanzadas

Este capítulo explora las capacidades avanzadas de Harbour para conectarse a bases de datos SQL modernas y trabajar con sistemas de gestión de bases de datos relacionales más complejos que van más allá del formato DBF tradicional.

---

## Conexión a bases de datos SQL (ODBC, MySQL, PostgreSQL, SQLite)

Harbour proporciona soporte robusto para conectarse a una amplia variedad de sistemas de gestión de bases de datos SQL a través de diferentes mecanismos:

### ODBC (Open Database Connectivity)

ODBC es un estándar que permite a las aplicaciones acceder a diferentes sistemas de gestión de bases de datos de manera uniforme. Harbour incluye soporte para ODBC, lo que permite conectarse a prácticamente cualquier base de datos que proporcione un driver ODBC.

### Bases de datos específicas

Harbour ofrece conectores nativos para las bases de datos SQL más populares:

- **MySQL**: Una de las bases de datos de código abierto más populares del mundo
- **PostgreSQL**: Un sistema de base de datos relacional y orientado a objetos avanzado
- **SQLite**: Una biblioteca que implementa un motor de base de datos SQL autónomo

---

## Uso de las librerías `contrib` `hbmysql`, `hbpgsql`, `hbsqlit3`

Las librerías contrib de Harbour proporcionan acceso directo y optimizado a bases de datos específicas:

### hbmysql

La librería `hbmysql` proporciona una interfaz nativa para conectarse a servidores MySQL. Ofrece funciones específicas para:

- Establecer conexiones con parámetros de autenticación
- Ejecutar consultas SQL de manera eficiente
- Manejar transacciones
- Procesar resultados de consultas

### hbpgsql

La librería `hbpgsql` permite la conectividad nativa con PostgreSQL, aprovechando las características avanzadas de este sistema de base de datos:

- Soporte para tipos de datos avanzados
- Funciones y procedimientos almacenados
- Transacciones complejas
- Características específicas de PostgreSQL

### hbsqlit3

`hbsqlit3` proporciona acceso a SQLite, ideal para aplicaciones que necesitan una base de datos embebida:

- Base de datos sin servidor
- Almacenamiento en un solo archivo
- Ideal para aplicaciones distribuidas
- Sin necesidad de configuración

---

## Conceptos de SQL: sentencias `SELECT`, `INSERT`, `UPDATE`, `DELETE`

### SELECT: Consulta de datos

La sentencia `SELECT` es utilizada para consultar y recuperar datos de una o más tablas:

```sql
SELECT columna1, columna2 FROM tabla WHERE condicion;
```

### INSERT: Inserción de datos

La sentencia `INSERT` permite agregar nuevos registros a una tabla:

```sql
INSERT INTO tabla (columna1, columna2) VALUES (valor1, valor2);
```

### UPDATE: Actualización de datos

La sentencia `UPDATE` modifica registros existentes en una tabla:

```sql
UPDATE tabla SET columna1 = valor1 WHERE condicion;
```

### DELETE: Eliminación de datos

La sentencia `DELETE` elimina registros de una tabla:

```sql
DELETE FROM tabla WHERE condicion;
```

### Integración con Harbour

Estas sentencias SQL se integran con Harbour a través de las librerías contrib, permitiendo ejecutar comandos SQL dinámicos desde código Harbour y procesar los resultados de manera eficiente.