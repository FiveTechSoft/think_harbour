# Capítulo 4: Estructuras de control

En Harbour, como en la mayoría de los lenguajes de programación, las estructuras de control son fundamentales para dirigir el flujo de ejecución de un programa. Permiten tomar decisiones, repetir acciones y manejar secuencias de código de manera ordenada.

## Sentencias condicionales

Las sentencias condicionales ejecutan bloques de código solo si se cumplen ciertas condiciones.

### IF, ELSEIF, ELSE, ENDIF

Esta es la estructura condicional más común. Permite ejecutar diferentes bloques de código basados en una o más condiciones lógicas.

**Sintaxis:**
```harbour
IF <condicionLogica1>
   // Bloque de código si la condicionLogica1 es verdadera (.T.)
[ELSEIF <condicionLogica2>]
   // Bloque de código si la condicionLogica2 es verdadera (.T.)
[ELSE]
   // Bloque de código si ninguna de las condiciones anteriores es verdadera
ENDIF
```

**Ejemplo:**
```harbour
PROCEDURE VerificarEdad( nEdad )
   IF nEdad >= 18
      ? "Es mayor de edad."
   ELSE
      ? "Es menor de edad."
   ENDIF
RETURN
```

### SWITCH

La estructura `SWITCH` es útil cuando se necesita comparar una única variable o expresión con una serie de valores posibles. Es una alternativa más legible a una serie de `IF...ELSEIF`.

**Sintaxis:**
```harbour
SWITCH <expresion>
   CASE <valor1>
      // Bloque de código si la expresion es igual a valor1
      [EXIT] // Opcional, sale del SWITCH
   CASE <valor2>
      // Bloque de código si la expresion es igual a valor2
      [EXIT]
   [OTHERWISE]
      // Bloque de código si la expresion no coincide con ningún CASE
ENDSWITCH
```
*Nota: A diferencia de otros lenguajes, si no se usa `EXIT`, la ejecución continúa en el siguiente `CASE`.*

**Ejemplo:**
```harbour
FUNCTION DiaDeLaSemana( nDia )
   LOCAL cDia := ""
   SWITCH nDia
      CASE 1
         cDia := "Lunes"
      CASE 2
         cDia := "Martes"
      CASE 3
         cDia := "Miércoles"
      // ... otros días
      OTHERWISE
         cDia := "Día inválido"
   ENDSWITCH
RETURN cDia
```

## Bucles

Los bucles permiten repetir la ejecución de un bloque de código múltiples veces.

### DO WHILE

Este bucle repite un bloque de código mientras una condición sea verdadera. La condición se evalúa al principio del bucle.

**Sintaxis:**
```harbour
DO WHILE <condicionLogica>
   // Bloque de código a repetir
   [LOOP]   // Vuelve al inicio del DO WHILE
   [EXIT]   // Sale del bucle
ENDDO
```

**Ejemplo:**
```harbour
PROCEDURE ContarHastaCinco()
   LOCAL nContador := 1
   DO WHILE nContador <= 5
      ? nContador
      nContador++
   ENDDO
RETURN
```

### FOR...NEXT

Este bucle se utiliza para repetir un bloque de código un número específico de veces. Es ideal para iterar con un contador.

**Sintaxis:**
```harbour
FOR <variable> := <inicio> TO <fin> [STEP <incremento>]
   // Bloque de código a repetir
   [LOOP]   // Salta a la siguiente iteración
   [EXIT]   // Sale del bucle
NEXT
```

**Ejemplo:**
```harbour
PROCEDURE TablaDelDos()
   LOCAL nNumero
   FOR nNumero := 1 TO 10
      ? "2 x", nNumero, "=", 2 * nNumero
   NEXT
RETURN
```

### FOR EACH

Este bucle está diseñado para iterar sobre los elementos de una colección, como un array o un hash.

**Sintaxis:**
```harbour
FOR EACH <variable> IN <coleccion>
   // Bloque de código a ejecutar para cada elemento
   // La <variable> toma el valor de cada elemento en cada iteración
   [LOOP]   // Salta a la siguiente iteración
   [EXIT]   // Sale del bucle
NEXT
```

**Ejemplo:**
```harbour
PROCEDURE MostrarColores()
   LOCAL aColores := { "Rojo", "Verde", "Azul" }
   LOCAL cColor

   FOR EACH cColor IN aColores
      ? cColor
   NEXT
RETURN
```

## Manejador de secuencias

### BEGIN SEQUENCE...END

Esta estructura proporciona un mecanismo para el manejo de errores o para salir de una secuencia de código anidada de forma controlada, similar a los bloques `try...catch` en otros lenguajes.

- `BREAK`: Interrumpe la secuencia y transfiere el control a la cláusula `RECOVER`.
- `RECOVER`: Se ejecuta si ocurre un `BREAK` en la secuencia.

**Sintaxis:**
```harbour
BEGIN SEQUENCE
   // Código principal de la secuencia
   [BREAK] // Interrumpe la secuencia
[RECOVER [USING <variableError>]]
   // Código de recuperación que se ejecuta después de un BREAK
END SEQUENCE
```

**Ejemplo de manejo de error:**
```harbour
FUNCTION Dividir( nNum, nDen )
   LOCAL nResultado

   BEGIN SEQUENCE
      IF nDen == 0
         // No se puede dividir por cero, rompemos la secuencia
         BREAK
      ENDIF
      nResultado := nNum / nDen
   RECOVER
      // Si hubo un BREAK, asignamos un valor por defecto
      nResultado := 0
      ? "Error: División por cero."
   END SEQUENCE

RETURN nResultado
``` 
