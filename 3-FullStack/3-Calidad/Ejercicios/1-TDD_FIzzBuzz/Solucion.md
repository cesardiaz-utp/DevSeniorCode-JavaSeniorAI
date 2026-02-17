# Solución Ejercicio TDD: El Problema FizzBuzz

## Implementación

```java
/**
 * Implementación del problema FizzBuzz siguiendo los principios de TDD.
 */
public class FizzBuzz {

    /**
     * Ejecuta la lógica de FizzBuzz para un número dado.
     *
     * @param number El número a evaluar.
     * @return "Fizz" si es divisible por 3, "Buzz" si es divisible por 5,
     *   "FizzBuzz" si es divisible por ambos, o el número en String si no cumple ninguna.
     * @throws IllegalArgumentException si el número es menor o igual a 0.
     */
    public String execute(int number) {
        if (number <= 0) {
            throw new IllegalArgumentException("El número debe ser un entero positivo.");
        }

        if (isDivisibleBy(number, 3) && isDivisibleBy(number, 5)) {
            return "FizzBuzz";
        }

        if (isDivisibleBy(number, 3)) {
            return "Fizz";
        }

        if (isDivisibleBy(number, 5)) {
            return "Buzz";
        }

        return String.valueOf(number);
    }

    private boolean isDivisibleBy(int number, int divisor) {
        return number % divisor == 0;
    }
}
```

## Pruebas unitarias

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class FizzBuzzTest {

    private FizzBuzz fizzBuzz;

    @BeforeEach
    void setUp() {
        fizzBuzz = new FizzBuzz();
    }

    @Test
    @DisplayName("Debe retornar '1' cuando la entrada es 1")
    void shouldReturn1WhenInputIs1() {
        assertEquals("1", fizzBuzz.execute(1));
    }

    @Test
    @DisplayName("Debe retornar 'Fizz' cuando el número es divisible por 3")
    void shouldReturnFizzWhenDivisibleBy3() {
        assertEquals("Fizz", fizzBuzz.execute(3));
        assertEquals("Fizz", fizzBuzz.execute(6));
    }

    @Test
    @DisplayName("Debe retornar 'Buzz' cuando el número es divisible por 5")
    void shouldReturnBuzzWhenDivisibleBy5() {
        assertEquals("Buzz", fizzBuzz.execute(5));
        assertEquals("Buzz", fizzBuzz.execute(10));
    }

    @Test
    @DisplayName("Debe retornar 'FizzBuzz' cuando es divisible por 3 y 5")
    void shouldReturnFizzBuzzWhenDivisibleBy15() {
        assertEquals("FizzBuzz", fizzBuzz.execute(15));
        assertEquals("FizzBuzz", fizzBuzz.execute(30));
    }

    @ParameterizedTest(name = "Entrada {0} debe resultar en {1}")
    @DisplayName("Pruebas parametrizadas para diversos casos")
    @CsvSource({
        "1, 1",
        "2, 2",
        "3, Fizz",
        "4, 4",
        "5, Buzz",
        "15, FizzBuzz",
        "98, 98",
        "99, Fizz",
        "100, Buzz"
    })
    void testVariousCases(int input, String expected) {
        assertEquals(expected, fizzBuzz.execute(input));
    }

    @Test
    @DisplayName("Debe lanzar excepción para números no positivos")
    void shouldThrowExceptionForInvalidInput() {
        assertThrows(IllegalArgumentException.class, () -> fizzBuzz.execute(0));
        assertThrows(IllegalArgumentException.class, () -> fizzBuzz.execute(-5));
    }
}
```
