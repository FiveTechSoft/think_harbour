# Capítulo 3: Funciones

Las funciones y procedimientos son los bloques de construcción fundamentales de cualquier programa en Harbour. Permiten organizar el código en unidades lógicas, reutilizables y fáciles de mantener. En este capítulo, exploraremos cómo definir y utilizar funciones, la diferencia entre ellas y los procedimientos, cómo manejar parámetros y algunas de las funciones de utilidad más importantes.

---

### Procedimientos (`PROCEDURE`) vs. Funciones (`FUNCTION`)

En Harbour, tanto `PROCEDURE` como `FUNCTION` se utilizan para definir subrutinas. Aunque son muy similares, la principal diferencia conceptual radica en si devuelven un valor o no.

*   **`FUNCTION`**: Una función es una subrutina diseñada para realizar una tarea específica y **devolver un valor** al código que la llamó. El valor se devuelve usando la sentencia `RETURN`. Si no se especifica un valor de retorno, una función devuelve `NIL` por defecto.

    ```harbour
    // Define una función que suma dos números y devuelve el resultado
    FUNCTION Sumar( n1, n2 )
       LOCAL nResultado
       nResultado := n1 + n2
    RETURN nResultado

    // Llama a la función y almacena el valor devuelto
    LOCAL nMiSuma
    nMiSuma := Sumar( 5, 10 ) // nMiSuma contendrá 15
    ? nMiSuma
    ```

*   **`PROCEDURE`**: Un procedimiento es una subrutina que realiza una serie de acciones pero **no devuelve un valor explícito**. Su propósito es ejecutar un proceso, como imprimir en pantalla o modificar una base de datos. En la práctica, un procedimiento en Harbour es simplemente una función que no tiene una sentencia `RETURN` con un valor. Internamente, también devuelve `NIL`.

    ```harbour
    // Define un procedimiento para mostrar un saludo
    PROCEDURE Saludar( cNombre )
       ? "¡Hola, " + cNombre + "!"
    RETURN

    // Llama al procedimiento
    Saludar( "Mundo" ) // Imprimirá "¡Hola, Mundo!"
    ```

**Conclusión clave:** En Harbour moderno, la distinción es principalmente semántica. Puedes usar `FUNCTION` para todo y simplemente omitir el valor de `RETURN` si no es necesario. Sin embargo, usar `PROCEDURE` puede clarificar la intención de que una subrutina no está destinada a devolver un resultado.

---

### Parámetros y Argumentos

Los parámetros son variables locales que reciben los valores (argumentos) pasados a una función o procedimiento.

*   **Declaración**: Los parámetros se declaran en una lista separada por comas en la definición de la función.
*   **Paso por valor (predeterminado)**: Por defecto, Harbour pasa los argumentos por valor. Esto significa que la función recibe una copia del dato original. Si la función modifica el parámetro, la variable original no se ve afectada.
*   **Paso por referencia**: Para modificar la variable original, puedes pasarla por referencia usando el operador `@` antes del nombre de la variable al llamar a la función. Dentro de la función, el parámetro contendrá una referencia (puntero) a la variable original.

```harbour
FUNCTION ModificarValor( nValorPorReferencia, nValorPorCopia )
   nValorPorReferencia += 100 // Modifica el original
   nValorPorCopia      += 100 // Modifica solo la copia local
RETURN NIL

LOCAL nA := 10, nB := 10

? "Valores originales:", nA, nB // Imprime: 10, 10

ModificarValor( @nA, nB ) // nA se pasa por referencia, nB por valor

? "Valores modificados:", nA, nB // Imprime: 110, 10
```

*   **Parámetros opcionales**: Harbour permite llamar a una función con menos argumentos de los declarados. Los parámetros no recibidos contendrán `NIL`. La función `PCOUNT()` se puede usar para saber cuántos argumentos se pasaron realmente.

```harbour
FUNCTION Mensaje( cTexto, cTitulo )
   IF PCOUNT() < 2
      cTitulo := "Aviso" // Asigna un valor por defecto si no se pasó el título
   ENDIF
   ? cTitulo + ": " + cTexto
RETURN NIL

Mensaje( "Operación completada." ) // Imprime: "Aviso: Operación completada."
Mensaje( "Error fatal.", "Error" )  // Imprime: "Error: Error fatal."
```

---

### Funciones de Entrada/Salida (E/S)

Las funciones de E/S son esenciales для la interacción con el usuario. Harbour proporciona un conjunto de funciones sencillas para manejar la consola.

*   `? <expresion>`: Imprime una o más expresiones en la consola, seguidas de un salto de línea.
*   `QOUT(<expresion>)` o `?? <expresion>`: Imprime una expresión sin el salto de línea final.
*   `ACCEPT "<prompt>" TO <variable>`: Muestra un mensaje y espera que el usuario introduzca una cadena de caracteres, que se guardará en la variable.
*   `INPUT "<prompt>" TO <variable>`: Similar a `ACCEPT`, pero evalúa la entrada como una expresión de Harbour. Es menos segura y su uso no se recomienda.
*   `INKEY(0)`: Espera a que el usuario presione una tecla y devuelve su código numérico ASCII. Es útil para menús o pausas.
*   `WAIT "<prompt>"`: Muestra un mensaje y pausa la ejecución hasta que se presiona una tecla.

```harbour
LOCAL cNombre

// Pide el nombre al usuario
ACCEPT "Por favor, introduce tu nombre: " TO cNombre

// Saluda al usuario
? "Bienvenido,", cNombre

// Espera a que el usuario presione una tecla para salir
WAIT "Presiona cualquier tecla para continuar..."
```

---

### Funciones de Utilidad (`VALTYPE()`, `PROCNAME()`)

Harbour incluye un rico conjunto de funciones integradas que proporcionan información sobre el estado del programa y los datos que se están manejando.

*   **`VALTYPE( <expresion> )`**: Devuelve una cadena de un solo carácter que indica el tipo de dato de la expresión. Es fundamental para la validación de parámetros o variables.
    *   `"C"`: String (Character)
    *   `"N"`: Número
    *   `"L"`: Lógico
    *   `"D"`: Fecha
    *   `"A"`: Array
    *   `"H"`: Hash
    *   `"B"`: Codeblock
    *   `"O"`: Objeto
    *   `"U"`: Indefinido (`NIL`)

*   **`PROCNAME( [nNivel] )`**: Devuelve el nombre de la función o procedimiento actual. Si se proporciona un nivel, puede inspeccionar la pila de llamadas para saber qué función llamó a la actual. Es una herramienta invaluable para la depuración y el registro de errores.

```harbour
FUNCTION MiRutina( xParametro )
   LOCAL cTipoDato

   // Obtener el nombre de la función actual
   ? "Ejecutando la función:", PROCNAME()

   // Validar el tipo del parámetro
   cTipoDato := VALTYPE( xParametro )

   IF cTipoDato == "C"
      ? "El parámetro es una cadena:", xParametro
   ELSEIF cTipoDato == "N"
      ? "El parámetro es un número:", xParametro
   ELSE
      ? "El parámetro es de un tipo no esperado:", cTipoDato
   ENDIF
RETURN NIL

// Llamadas de ejemplo
MiRutina( "Hola" )
MiRutina( 123.45 )
MiRutina( .T. )
```
