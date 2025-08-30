# Capítulo 2: Variables, expresiones y sentencias

En este capítulo, exploraremos los componentes fundamentales de cualquier programa en Harbour: cómo se almacenan los datos, cómo se manipulan y cómo se combinan para formar instrucciones que el compilador puede entender.

---

### Valores y tipos de datos en Harbour

Harbour es un lenguaje de tipado dinámico, lo que significa que una variable puede cambiar de tipo durante la ejecución del programa. Sin embargo, cada valor tiene un tipo de dato específico.

*   **`String` (Cadena de caracteres)**: Representa texto. Se puede delimitar con comillas simples (`'`), dobles (`"`) o corchetes (`[]`).
    ```harbour
    LOCAL cNombre := "Juan Pérez"
    LOCAL cDireccion := 'Calle Mayor, 123'
    LOCAL cTextoLargo := [Este es un texto que puede contener ' y " sin problemas.]
    ```

*   **`Number` (Número)**: Puede ser un número entero o decimal. Harbour maneja la precisión automáticamente.
    ```harbour
    LOCAL nEdad := 30
    LOCAL nPrecio := 19.95
    LOCAL nNegativo := -100
    ```

*   **`Logical` (Lógico)**: Representa un valor booleano, que puede ser verdadero (`.T.`) o falso (`.F.`).
    ```harbour
    LOCAL lEsCliente := .T.
    LOCAL lActivo := .F.
    ```

*   **`Date` (Fecha)**: Almacena una fecha (día, mes y año). El formato por defecto es `mm/dd/yyyy`, pero puede cambiarse con `SET DATE`.
    ```harbour
    LOCAL dNacimiento := CTOD("12/25/1990") // Convertir cadena a fecha
    LOCAL dHoy := DATE() // Obtiene la fecha actual
    ```

*   **`CodeBlock` (Bloque de código)**: Es un fragmento de código compilado que se puede almacenar en una variable y ejecutar más tarde. Es una de las características más potentes de Harbour. Se define con `{|parametros| expresión}`.
    ```harbour
    LOCAL bMiBloque := { |x, y| x + y } // Un bloque que suma dos números
    AEVAL( aMiArray, { |elemento| QOUT(elemento) } ) // Evaluar un bloque para cada elemento de un array
    ```

*   **`Symbol` (Símbolo)**: Es una referencia interna a una función o variable. Se utiliza para llamadas indirectas.
    ```harbour
    LOCAL symFunc := @MiFuncion()
    EVAL( symFunc ) // Llama a MiFuncion()
    ```

*   **`Pointer` (Puntero)**: Es una referencia a la dirección de memoria de otra variable. Se utiliza principalmente para la integración con código C.

*   **`NIL`**: Representa la ausencia de valor. Una variable no inicializada tiene el valor `NIL`.
    ```harbour
    LOCAL x // x es NIL por defecto
    IF x == NIL
       QOUT("La variable no tiene valor")
    ENDIF
    ```
---

### Variables y su alcance (`scope`)

El alcance determina dónde es visible y accesible una variable.

*   **`LOCAL`**: La variable solo existe dentro de la función o procedimiento donde se declara. Es la forma más recomendada y segura de trabajar.
    ```harbour
    FUNCTION MiFuncion()
       LOCAL nContador := 0 // Solo visible aquí
       nContador++
       RETURN nContador
    ```

*   **`STATIC`**: Similar a `LOCAL`, pero la variable retiene su valor entre llamadas sucesivas a la misma función. Se inicializa solo la primera vez que se ejecuta la función.
    ```harbour
    FUNCTION ContadorDeLlamadas()
       STATIC nVeces := 0 // Se inicializa a 0 solo la primera vez
       nVeces++
       QOUT("Esta función ha sido llamada", nVeces, "veces.")
       RETURN NIL
    ```

*   **`PRIVATE`**: La variable es visible en la función donde se declara y en todas las funciones llamadas desde ella. Es un mecanismo heredado de Clipper y su uso puede generar código difícil de mantener.
    ```harbour
    PROCEDURE Principal()
       PRIVATE cMensaje := "Hola desde Principal"
       Subrutina()
       RETURN

    PROCEDURE Subrutina()
       QOUT(cMensaje) // Muestra "Hola desde Principal"
       RETURN
    ```

*   **`PUBLIC`**: La variable es global y accesible desde cualquier parte del programa. Su uso está desaconsejado, ya que puede provocar efectos secundarios inesperados. Es preferible usar funciones para obtener o establecer valores globales de forma controlada.
    ```harbour
    PUBLIC g_cUsuarioActivo
    // ... en alguna parte del código
    g_cUsuarioActivo := "admin"
    ```

---

### Sentencias de asignación y operadores

*   **Asignación**: Se utiliza para dar un valor a una variable. El operador de asignación en línea `:=` declara e inicializa la variable como `LOCAL`. El operador `=` asigna un valor a una variable ya existente.
    ```harbour
    LOCAL nNumero
    nNumero := 10 // Declaración y asignación en línea (implica LOCAL)
    nNumero = 20  // Asignación a una variable existente
    ```

*   **Operadores de Asignación Compuesta**: Estos operadores combinan una operación aritmética con asignación, proporcionando una forma más concisa de modificar el valor de una variable.
    ```harbour
    LOCAL nValor := 10
    nValor += 5   // Equivalente a: nValor = nValor + 5 (resultado: 15)
    nValor -= 3   // Equivalente a: nValor = nValor - 3 (resultado: 12)
    nValor *= 2   // Equivalente a: nValor = nValor * 2 (resultado: 24)
    nValor /= 4   // Equivalente a: nValor = nValor / 4 (resultado: 6)
    ```

*   **Operadores Aritméticos**: `+` (suma), `-` (resta), `*` (multiplicación), `/` (división), `%` (módulo), `^` o `**` (potencia).
*   **Operadores de Cadenas**: `+` (concatena), `-` (concatena eliminando espacios finales).
*   **Operadores de Fechas**: `+` (suma días), `-` (resta días o calcula diferencia entre fechas).
*   **Operadores Relacionales**: `==` (exactamente igual), `=` (igual), `!=` o `<>` (distinto), `<` (menor que), `>` (mayor que), `<=` (menor o igual), `>=` (mayor o igual).
*   **Operadores Lógicos**: `.AND.` (y), `.OR.` (o), `.NOT.` (no). Se escriben con puntos alrededor.
*   **Operadores de Incremento/Decremento**: `++` (incrementa en 1), `--` (decrementa en 1). Pueden ser prefijos (`++n`) o sufijos (`n++`).

---

### Expresiones

Una expresión es una combinación de valores, variables, operadores y llamadas a funciones que el lenguaje puede evaluar para producir un valor.

*   **Expresión simple**:
    ```harbour
    nEdad + 1
    ```

*   **Expresión compleja**:
    ```harbour
    ( ( nSubTotal + nImpuestos ) * ( 1 - nDescuento / 100 ) ) > 0 .AND. !EMPTY(cNombre)
    ```

*   **Expresión en una sentencia**:
    ```harbour
    // La expresión es nPrecio * 1.21
    REPLACE precio WITH nPrecio * 1.21

    // La expresión es lCondicion .AND. nValor > 10
    IF lCondicion .AND. nValor > 10
       // ...
    ENDIF
    ```
