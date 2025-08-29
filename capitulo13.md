# Capítulo 13: El núcleo de la máquina virtual de Harbour

Este capítulo profundiza en el funcionamiento interno de Harbour, explorando cómo el compilador convierte código fuente en código ejecutable y cómo la máquina virtual gestiona la ejecución.

---

## El compilador: el proceso de conversión de código fuente a P-Code

El compilador de Harbour realiza una transformación compleja del código fuente legible por humanos en P-Code optimizado que puede ser ejecutado eficientemente por la máquina virtual.

### Fases de compilación

#### 1. Análisis léxico (Lexical Analysis)

El compilador descompone el código fuente en tokens o elementos básicos:

- **Palabras clave**: `FUNCTION`, `IF`, `LOCAL`, etc.
- **Identificadores**: nombres de variables y funciones
- **Literales**: números, cadenas, constantes
- **Operadores**: `+`, `-`, `==`, `:=`, etc.
- **Símbolos**: paréntesis, comas, puntos y comas

#### 2. Análisis sintáctico (Parsing)

Construye un árbol de sintaxis abstracta (AST) que representa la estructura del programa:

- Verifica que la sintaxis sea correcta
- Identifica las relaciones entre elementos
- Detecta errores de sintaxis

#### 3. Análisis semántico

Realiza verificaciones de significado y coherencia:

- **Verificación de tipos**: Aunque Harbour es dinámico, realiza algunas verificaciones
- **Resolución de nombres**: Vincula identificadores con sus declaraciones
- **Verificación de alcance**: Valida el acceso a variables según su scope

#### 4. Generación de P-Code

Convierte el AST en P-Code optimizado:

```harbour
// Código fuente original
LOCAL nNumero := 10
nNumero := nNumero + 5
? nNumero

// Se convierte en P-Code (representación simplificada)
// LOCVAR 1     // Declarar variable local 1
// PUSHNI 10    // Apilar número 10
// POPVL 1      // Guardar en variable local 1
// PUSHVL 1     // Apilar variable local 1
// PUSHNI 5     // Apilar número 5
// PLUS         // Sumar los dos valores en la pila
// POPVL 1      // Guardar resultado en variable local 1
// PUSHVL 1     // Apilar variable local 1
// DISPLAY      // Mostrar valor
```

---

## La máquina virtual (VM): el motor de ejecución de Harbour

La máquina virtual de Harbour es el componente que ejecuta el P-Code generado por el compilador.

### Arquitectura de la VM

#### Componentes principales

- **Intérprete de P-Code**: Ejecuta las instrucciones P-Code una por una
- **Pila de ejecución**: Almacena datos temporales y maneja llamadas a funciones
- **Gestión de memoria**: Maneja la asignación y liberación de memoria
- **Sistema de items**: Maneja los diferentes tipos de datos de Harbour

#### Motor de ejecución

La VM opera en un ciclo de ejecución continuo:

1. **Fetch**: Obtiene la siguiente instrucción P-Code
2. **Decode**: Decodifica la instrucción para determinar qué hacer
3. **Execute**: Ejecuta la operación correspondiente
4. **Repeat**: Repite el ciclo hasta finalizar el programa

### Optimizaciones de la VM

- **Instrucciones especializadas**: P-Codes optimizados para operaciones comunes
- **Cache de símbolos**: Acelera la resolución de nombres de funciones y variables
- **Optimización de llamadas**: Manejo eficiente de llamadas a funciones
- **Gestión de memoria optimizada**: Minimiza la fragmentación y las asignaciones

---

## P-Code: el lenguaje intermedio optimizado

El P-Code (Pseudo-Code) es una representación intermedia entre el código fuente y el código máquina nativo.

### Características del P-Code

#### Independencia de plataforma

- El mismo P-Code se ejecuta en diferentes sistemas operativos
- La VM se adapta a las características específicas de cada plataforma
- Facilita la portabilidad de aplicaciones

#### Optimización

```harbour
// Código original con redundancias
LOCAL a := 1
LOCAL b := 2
LOCAL c := a + b + a + b

// P-Code optimizado elimina cálculos redundantes
// y reutiliza valores cuando es posible
```

#### Tipos de instrucciones P-Code

**Manipulación de pila:**
- `PUSH*`: Apilar valores (números, cadenas, variables)
- `POP*`: Desapilar y almacenar valores
- `DUP`: Duplicar el valor en la cima de la pila

**Operaciones aritméticas:**
- `PLUS`, `MINUS`, `MULT`, `DIVIDE`
- `MOD`: Módulo
- `POWER`: Potenciación

**Operaciones lógicas:**
- `AND`, `OR`, `NOT`
- Comparaciones: `EQUAL`, `LESS`, `GREATER`

**Control de flujo:**
- `JUMP`: Salto incondicional
- `JUMPF`: Salto si falso
- `JUMPT`: Salto si verdadero

**Llamadas a funciones:**
- `CALL`: Llamar función
- `RETURN`: Retornar de función
- `PARAM`: Pasar parámetros

---

## La pila (`The stack`): gestión de llamadas a funciones y variables locales

La pila es fundamental para el funcionamiento de la VM, especialmente para el manejo de llamadas a funciones y variables locales.

### Estructura de la pila

#### Stack frames (marcos de pila)

Cada llamada a función crea un nuevo marco en la pila que contiene:

```
+------------------+  <- Tope de la pila
| Variables locales |
| Parámetros       |
| Dirección retorno|
| Marco anterior   |
+------------------+
| Marco función    |
| anterior         |
+------------------+
```

#### Gestión de variables locales

```harbour
FUNCTION Ejemplo( nParam1, nParam2 )
   LOCAL nLocal1 := 10     // Slot 1 en el marco actual
   LOCAL cLocal2 := "test" // Slot 2 en el marco actual
   LOCAL nResult
   
   nResult := nParam1 + nParam2 + nLocal1
   
   RETURN nResult
```

### Operaciones de pila

#### Push/Pop operations

- **Push**: Añade elementos al tope de la pila
- **Pop**: Remueve y retorna el elemento del tope
- **Peek**: Examina el elemento del tope sin removerlo

#### Gestión de memoria de pila

- **Asignación automática**: Los marcos se crean automáticamente
- **Liberación automática**: Se liberan al retornar de funciones
- **Detección de overflow**: Protección contra desbordamiento de pila
- **Optimización de espacio**: Reutilización eficiente del espacio de pila

### Recursión y pila

La pila permite el manejo natural de funciones recursivas:

```harbour
FUNCTION Factorial( n )
   IF n <= 1
      RETURN 1
   ENDIF
   RETURN n * Factorial( n - 1 )  // Cada llamada crea un nuevo marco
```

Cada llamada recursiva crea un nuevo marco de pila, manteniendo el estado independiente de cada nivel de recursión.