# Capítulo 11: Librerías y Paquetes Externos

Harbour posee un núcleo de lenguaje potente, pero su verdadera fuerza reside en su ecosistema de librerías y herramientas que extienden sus capacidades a casi cualquier área imaginable de la programación. Este capítulo explora cómo gestionar estas librerías, cómo construir proyectos complejos y cómo utilizar algunas de las librerías más populares para tareas modernas como la manipulación de XML y JSON.

## 1. El Sistema de Construcción `hbmk2`

`hbmk2` es la herramienta de construcción oficial de Harbour. Es un potente script que automatiza el proceso de compilación y enlazado de programas, manejando dependencias, librerías externas, diferentes compiladores de C y construcción multiplataforma.

### ¿Por qué usar `hbmk2`?

- **Simplicidad:** Automatiza la detección del compilador de C, las rutas y las librerías necesarias.
- **Portabilidad:** Un mismo script de construcción puede funcionar en Windows, Linux y macOS.
- **Potencia:** Soporta proyectos multi-módulo, diferentes dialectos (xBase++), y es altamente configurable.
- **Gestión de dependencias:** Puede compilar y enlazar automáticamente las librerías requeridas por tu proyecto.

### Uso Básico

El uso más simple es pasarle el archivo `.prg` principal:

```bash
# Compila y enlaza myapp.prg para crear myapp.exe (o myapp en Linux)
hbmk2 myapp.prg
```

Para incluir una librería externa (por ejemplo, `hbcurl`), se utiliza el flag `-l`:

```bash
# Enlaza la librería hbcurl.
# hbmk2 busca automáticamente hbcurl.hbc en las rutas estándar.
hbmk2 myapp.prg -lhbcurl
```

### Archivos de Proyecto (`.hbp`)

Para proyectos más complejos, es recomendable usar un archivo `.hbp` (Harbour Project) que agrupe todas las opciones de compilación.

**Ejemplo de `myapp.hbp`:**

```
# Archivos de código fuente
myapp.prg
utils.prg
reports.prg

# Librerías a enlazar
-lhbcurl
-lhbzip

# Opciones del compilador
-w3 -es2
```

Para construir el proyecto, simplemente ejecuta:

```bash
hbmk2 myapp.hbp
```

## 2. Librerías `contrib` Populares

La distribución de Harbour incluye una carpeta `contrib` que contiene una vasta colección de librerías aportadas por la comunidad. Estas librerías son una parte fundamental del ecosistema. Para usarlas, normalmente solo necesitas añadir el flag `-l<libname>` a `hbmk2`.

Algunas de las más utilizadas son:

- **`hbcurl`**: Un wrapper completo sobre la popular librería `libcurl`. Permite realizar peticiones HTTP/S (GET, POST), FTP, etc. Esencial para interactuar con APIs web.
- **`hbtip`**: Proporciona funcionalidades de red a bajo nivel basadas en TCP/IP, permitiendo crear clientes y servidores para protocolos personalizados.
- **`hbzip`**: Ofrece funciones para crear y extraer archivos en formato `.zip`. Incluye `hbziparc` para compresión y `hbunzip` para descompresión.
- **`hbsqlite`**: Permite trabajar con bases de datos SQLite3, una base de datos ligera y embebida, ideal para aplicaciones de escritorio o móviles.
- **`hbpgsql`**: Cliente para la base de datos PostgreSQL.
- **`hbexpat`**: Un parser de XML basado en la librería `expat`, útil para leer y procesar documentos XML de forma eficiente.
- **`hbhpdf`**: Permite la creación de documentos PDF de forma nativa desde Harbour, utilizando la librería `libHaru`.
- **`hbide`**: Un entorno de desarrollo integrado (IDE) simple, escrito en Harbour.

## 3. Manipulación de Archivos XML y JSON

El intercambio de datos con sistemas modernos se realiza mayoritariamente a través de formatos como JSON y XML. Harbour ofrece soporte nativo o a través de librerías `contrib` para ambos.

### Manipulación de JSON

Harbour incluye soporte nativo para JSON en su núcleo, por lo que no se necesita ninguna librería externa. Las funciones principales son `hb_jsonEncode()` y `hb_jsonDecode()`.

- **`hb_jsonEncode( <valor> )`**: Convierte un valor de Harbour (array, hash, string, número) a una cadena de texto en formato JSON.
- **`hb_jsonDecode( <jsonString> )`**: Convierte una cadena de texto JSON a un valor de Harbour (hashes para objetos, arrays para listas).

**Ejemplo de uso:**

```prg
PROCEDURE Main()
    LOCAL hData, cJson, hResult

    // 1. Crear un Hash en Harbour y convertirlo a JSON
    hData := { => }
    hData[ "nombre" ] := "Juan Pérez"
    hData[ "edad" ]   := 30
    hData[ "activo" ] := .T.
    hData[ "cursos" ] := { "Harbour", "SQL", "Git" }

    cJson := hb_jsonEncode( hData )

    ? "Objeto Harbour:"
    AEval( hData, { |v, k| QOut(hb_NtoS(k) + ": " + hb_ValToExp(v)) } )

    ?
    ? "Cadena JSON generada:"
    ? cJson
    // Salida: {"nombre":"Juan Pérez","edad":30,"activo":true,"cursos":["Harbour","SQL","Git"]}

    // 2. Convertir la cadena JSON de vuelta a un Hash de Harbour
    hResult := hb_jsonDecode( cJson )

    ?
    ? "Objeto Harbour decodificado:"
    ? hResult[ "nombre" ] // "Juan Pérez"
    ? hResult[ "cursos" ][ 2 ] // "SQL"

    RETURN
```

### Manipulación de XML

Para XML, la forma más común de trabajar es usando la librería `hbexpat`, que se encuentra en `contrib`. Esta librería proporciona un parser rápido y eficiente. No construye un árbol DOM en memoria, sino que funciona a través de eventos (SAX-like parser), lo que la hace muy eficiente para archivos grandes.

Para usarla, necesitas enlazar la librería:

```bash
hbmk2 myxmlapp.prg -lhbexpat
```

**Ejemplo de lectura de XML con `hbexpat`:**

Supongamos que tenemos un archivo `datos.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<personas>
    <persona id="1">
        <nombre>Ana</nombre>
        <ciudad>Madrid</ciudad>
    </persona>
    <persona id="2">
        <nombre>Luis</nombre>
        <ciudad>Barcelona</ciudad>
    </persona>
</personas>
```

Podemos procesarlo con el siguiente código:

```prg
#include "hbexpat.ch"

PROCEDURE Main()
    LOCAL oParser, cXml

    // Cargar el contenido del archivo XML
    cXml := MemoRead( "datos.xml" )

    // Crear el objeto parser
    oParser := hb_expat_Create()

    // Definir los "handlers" (funciones que se llamarán al encontrar elementos)
    oParser:startElementHandler := { |o, cTag, hAttr| StartElement(cTag, hAttr) }
    oParser:characterDataHandler := { |o, cText| CharData(cText) }
    oParser:endElementHandler := { |o, cTag| EndElement(cTag) }

    // Procesar el XML
    IF ! oParser:Parse( cXml )
        ? "Error de parsing:", oParser:ErrorCode(), oParser:ErrorString()
    ENDIF

    // Liberar recursos
    oParser:Destroy()

    RETURN

STATIC PROCEDURE StartElement( cTag, hAttributes )
    QOut( "Inicia etiqueta:", cTag )
    IF ! Empty( hAttributes )
        AEval( hAttributes, { |v, k| QOut( "  Atributo:", k, "=", v ) } )
    ENDIF
    RETURN

STATIC PROCEDURE CharData( cText )
    // El texto puede venir en fragmentos, por lo que se debe acumular
    // si es necesario. Aquí solo lo mostramos.
    IF ! Empty( AllTrim( cText ) )
        QOut( "  Texto:", AllTrim( cText ) )
    ENDIF
    RETURN

STATIC PROCEDURE EndElement( cTag )
    QOut( "Cierra etiqueta:", cTag )
    RETURN
```

Este código recorrerá el documento XML y disparará las funciones `StartElement`, `CharData` y `EndElement` a medida que encuentra etiquetas de apertura, texto y etiquetas de cierre, respectivamente, permitiéndote procesar el contenido de manera estructurada.
