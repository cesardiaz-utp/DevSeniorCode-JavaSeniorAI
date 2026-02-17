# Solución Ejercicio TDD: Conversor de Números Romanos

## Implementación

```java
import java.util.Collections;
import java.util.Map;
import java.util.TreeMap;

/**
 * Implementación de un conversor de números arábigos a romanos.
 * Se utiliza el constructor para inicializar las reglas de conversión.
 */
public class RomanConverter {

    private final TreeMap<Integer, String> mapping;

    /**
     * Constructor que inicializa las reglas estándar de la numeración romana.
     */
    public RomanConverter() {
        this.mapping = new TreeMap<>(Collections.reverseOrder());
        initializeStandardRules();
    }

    private void initializeStandardRules() {
        mapping.put(1000, "M");
        mapping.put(900, "CM");
        mapping.put(500, "D");
        mapping.put(400, "CD");
        mapping.put(100, "C");
        mapping.put(90, "XC");
        mapping.put(50, "L");
        mapping.put(40, "XL");
        mapping.put(10, "X");
        mapping.put(9, "IX");
        mapping.put(5, "V");
        mapping.put(4, "IV");
        mapping.put(1, "I");
    }

    /**
     * Convierte un número entero a su representación en numeración romana.
     * * @param number Número entre 1 y 3999
     * @return String con el equivalente romano
     * @throws IllegalArgumentException si el número está fuera del rango permitido
     */
    public String convert(int number) {
        if (number <= 0 || number > 3999) {
            throw new IllegalArgumentException("El número debe estar entre 1 y 3999");
        }

        StringBuilder result = new StringBuilder();
        int remaining = number;

        for (Map.Entry<Integer, String> entry : mapping.entrySet()) {
            int value = entry.getKey();
            String symbol = entry.getValue();

            while (remaining >= value) {
                result.append(symbol);
                remaining -= value;
            }
        }

        return result.toString();
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

class RomanConverterTest {

    private RomanConverter converter;

    @BeforeEach
    void setUp() {
        converter = new RomanConverter();
    }

    @ParameterizedTest(name = "Entrada {0} debe ser {1}")
    @DisplayName("Validación de casos del Plan de Pruebas")
    @CsvSource({
        "1, I",       // Caso 1
        "2, II",      // Caso 2
        "3, III",     // Caso 3
        "5, V",       // Caso 4
        "10, X",      // Caso 5
        "6, VI",      // Caso 10
        "4, IV",      // Caso 13
        "9, IX",      // Caso 14
        "14, XIV",    // Caso 19
        "99, XCIX",   // Caso 21
        "1994, MCMXCIV", // Caso 22
        "3999, MMMCMXCIX" // Caso 23
    })
    void testConversion(int input, String expected) {
        assertEquals(expected, converter.convert(input));
    }

    @Test
    @DisplayName("Debe lanzar excepción para números menores o iguales a 0")
    void testInvalidNumbers() {
        assertThrows(IllegalArgumentException.class, () -> converter.convert(0)); // Caso 24
        assertThrows(IllegalArgumentException.class, () -> converter.convert(-1)); // Caso 25
    }

    @Test
    @DisplayName("Debe lanzar excepción para números mayores a 3999")
    void testUpperLimit() {
        assertThrows(IllegalArgumentException.class, () -> converter.convert(4000)); // Caso 26
    }
}
```
