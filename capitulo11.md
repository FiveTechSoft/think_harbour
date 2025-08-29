# Capítulo 11: Librerías y paquetes externos

Este capítulo cubre el ecosistema de librerías y herramientas externas disponibles para Harbour, incluyendo el sistema de construcción, las librerías contrib populares y el manejo de formatos de datos modernos.

---

## El sistema de construcción `hbmk2`

`hbmk2` es el sistema de construcción oficial de Harbour, diseñado para simplificar el proceso de compilación y enlazado de aplicaciones Harbour complejas.

### Características principales

- **Detección automática**: Detecta automáticamente las dependencias y librerías necesarias
- **Multiplataforma**: Funciona de manera consistente en Windows, Linux y macOS
- **Configuración flexible**: Permite configuraciones complejas a través de archivos `.hbp`
- **Soporte para librerías**: Maneja automáticamente el enlazado de librerías contrib y externas

### Uso básico

```bash
# Compilar un programa simple
hbmk2 mi_programa.prg

# Usar un archivo de proyecto
hbmk2 mi_proyecto.hbp

# Compilar con librerías específicas
hbmk2 mi_programa.prg -lhbmysql -lhbpgsql
```

### Archivos de proyecto (.hbp)

Los archivos `.hbp` permiten definir configuraciones de proyecto complejas:

```
# Mi proyecto
-o=mi_aplicacion
-gui
-inc=include/
-lib=lib/
programa_principal.prg
modulo1.prg
modulo2.prg
```

---

## Librerías `contrib` populares

Las librerías contrib extienden las capacidades base de Harbour con funcionalidades especializadas:

### Librerías de base de datos

- **hbmysql**: Conectividad MySQL
- **hbpgsql**: Conectividad PostgreSQL
- **hbsqlit3**: Conectividad SQLite
- **hbodbc**: Conectividad ODBC genérica

### Librerías de red y web

- **hbcurl**: Cliente HTTP/HTTPS basado en libcurl
- **hbhttpd**: Servidor HTTP embebido
- **hbssl**: Soporte SSL/TLS
- **hbsocket**: Funciones de socket de bajo nivel

### Librerías de interfaz de usuario

- **hbqt**: Binding para Qt framework
- **hbfimage**: Manipulación de imágenes
- **hbgd**: Interfaz para la librería GD de gráficos

### Librerías de utilidad

- **hbzlib**: Compresión/descompresión
- **hbcrypto**: Funciones criptográficas
- **hbregex**: Expresiones regulares
- **hbtip**: Protocolo de Internet (TIP)

---

## Manipulación de archivos XML y JSON

Harbour proporciona soporte para trabajar con los formatos de datos más comunes en aplicaciones modernas.

### Trabajo con XML

Harbour incluye funciones para parsear y generar documentos XML:

```harbour
// Ejemplo de procesamiento XML
LOCAL cXmlString := '<root><item>valor</item></root>'
LOCAL oXmlDoc
LOCAL oNode

// Parsear XML
oXmlDoc := TXmlDocument():New()
oXmlDoc:LoadFromString( cXmlString )

// Acceder a nodos
oNode := oXmlDoc:FindFirst( "item" )
? "Valor:", oNode:GetText()
```

### Trabajo con JSON

El soporte para JSON es esencial para aplicaciones web y APIs modernas:

```harbour
// Ejemplo de procesamiento JSON
LOCAL cJsonString := '{"nombre": "Juan", "edad": 30}'
LOCAL hData

// Parsear JSON a hash
hData := hb_jsonDecode( cJsonString )
? "Nombre:", hData["nombre"]
? "Edad:", hData["edad"]

// Generar JSON desde hash
LOCAL hNuevoDato := {"producto" => "Harbour", "version" => "3.2"}
LOCAL cJsonResult := hb_jsonEncode( hNuevoDato )
? "JSON generado:", cJsonResult
```

### Validación y transformación

- **Validación de esquemas**: Verificar que los datos cumplan con estructuras esperadas
- **Transformaciones**: Convertir entre diferentes formatos de datos
- **Manejo de errores**: Gestión robusta de datos malformados o incompletos

Las funciones de manipulación de XML y JSON facilitan la integración de aplicaciones Harbour con servicios web modernos y el intercambio de datos estructurados.