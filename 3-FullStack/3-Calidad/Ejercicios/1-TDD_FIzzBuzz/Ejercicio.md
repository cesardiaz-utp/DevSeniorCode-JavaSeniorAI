# Ejercicio TDD: El Problema FizzBuzz

FizzBuzz es un ejercicio clásico de lógica de programación que se utiliza para enseñar el control de flujo y la importancia del orden de las condiciones. En TDD, es ideal para mostrar cómo las pruebas nos obligan a manejar casos de superposición (como cuando un número cumple dos reglas a la vez).

## Descripción de la Funcionalidad

El programa debe recibir un número entero positivo y devolver una cadena de texto (String) siguiendo estas reglas:

1. **Regla Fizz**: Si el número es divisible por **3**, el resultado es `"Fizz"`.
2. **Regla Buzz**: Si el número es divisible por **5**, el resultado es `"Buzz"`.
3. **Regla FizzBuzz**: Si el número es divisible por **3** y por **5** simultáneamente, el resultado es **"FizzBuzz"**.
4. **Regla por Defecto**: Si no se cumple ninguna de las anteriores, el resultado es el **propio número** convertido a texto (ej: `1` -> `"1"`).
5. **Regla Negativos**: si el numero es negativo, debe lanzar la excepción `IllegalArgumentException`.

## Plan de Pruebas Paso a Paso

Para aplicar TDD correctamente, debemos definir las pruebas en orden de especificidad, de lo más simple a lo más complejo:

### 1. Casos de Números Normales (Default)

Estos casos establecen la estructura básica del método.

- **Prueba 1**: Entrada `1` → Esperado `"1"`.
- **Prueba 2**: Entrada `2` → Esperado `"2"`.

### 2. La Regla de "Fizz" (Divisibilidad por 3)

- **Prueba 3**: Entrada `3` → Esperado `"Fizz"`.
- **Prueba 4**: Entrada `6` → Esperado `"Fizz"`.

### 3. La Regla de "Buzz" (Divisibilidad por 5)

- **Prueba 5**: Entrada `5` → Esperado `"Buzz"`.
- **Prueba 6**: Entrada `10` → Esperado `"Buzz"`.

### 4. La Regla Combinada "FizzBuzz"

Este es el punto crítico del ejercicio donde el orden de los if importa.

- **Prueba 7**: Entrada `15` → Esperado `"FizzBuzz"`.
- **Prueba 8**: Entrada `30` → Esperado `"FizzBuzz"`.

### 5. Casos de Borde

- **Prueba 9**: Entrada `0` → lanza `IllegalArgumentException`.
- **Prueba 10**: Entrada `-1` → lanza `IllegalArgumentException`.

## Ciclo de Desarrollo Sugerido

1. **Fase Red**: Ejecutar el test de `1` y `2`. Fallará porque el método no existe o devuelve `null`.
2. **Fase Green**: Implementar `return String.valueOf(number)`.
3. **Fase Red**: Ejecutar el test de `3`. Fallará (devolverá `"3"` en lugar de `"Fizz"`).
4. **Fase Green**: Añadir `if (number % 3 == 0) return "Fizz"`.
5. **Fase Refactor**: Una vez que todos los casos pasen (incluyendo el 15), revisar si se puede limpiar la lógica usando una estructura más escalable o eliminando duplicidad de operadores `%`.
