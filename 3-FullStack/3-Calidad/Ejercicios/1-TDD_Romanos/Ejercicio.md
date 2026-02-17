# Ejercicio TDD: Conversor de Números Romanos

El objetivo es desarrollar un conversor que transforme números arábigos en su representación romana clásica (sistema aditivo-sustractivo).

La funcionalidad se basa en mapear valores decimales específicos a símbolos (I, V, X, L, C, D, M) y aplicar reglas de repetición y sustracción para valores como el 4 (IV) o el 9 (IX), limitando el rango operativo de 1 a 3999 para cumplir con las convenciones estándar.

## 1. Casos de Símbolos Simples (Unidades)

Estos casos establecen la estructura inicial y la lógica de repetición básica.

- **Caso 1**: Entrada `1` → Esperado `"I"` (Prueba de existencia).
- **Caso 2**: Entrada `2` → Esperado `"II"` (Prueba de acumulación/repetición).
- **Caso 3**: Entrada `3` → Esperado `"III"`.

## 2. Casos de Cambio de Símbolo (Puntos de Quiebre)

Aquí es donde el código debe empezar a diferenciar valores.

- **Caso 4**: Entrada `5` → Esperado `"V"` (Introducción de nuevo símbolo).
- **Caso 5**: Entrada `10` → Esperado `"X"`.
- **Caso 6**: Entrada `50` → Esperado `"L"`.
- **Caso 7**: Entrada `100` → Esperado `"C"`.
- **Caso 8**: Entrada `500` → Esperado `"D"`.
- **Caso 9**: Entrada `1000` → Esperado `"M"`.

## 3. Casos de Adición (Combinaciones Directas)

Prueban que el algoritmo puede sumar símbolos de mayor a menor.

- **Caso 10**: Entrada `6` → Esperado `"VI"` (5 + 1).
- **Caso 11**: Entrada `15` → Esperado `"XV"` (10 + 5).
- **Caso 12**: Entrada `1550` → Esperado `"MDL"` (1000 + 500 + 50).

## 4. Casos de Sustracción (La Regla del "4" y el "9")

Estos casos suelen romper la lógica simple y obligan a refactorizar hacia un mapa de constantes o reglas especiales.

- **Caso 13**: Entrada `4` → Esperado `"IV"`.
- **Caso 14**: Entrada `9` → Esperado `"IX"`.
- **Caso 15**: Entrada `40` → Esperado `"XL"`.
- **Caso 16**: Entrada `90` → Esperado `"XC"`.
- **Caso 17**: Entrada `400` → Esperado `"CD"`.
- **Caso 18**: Entrada `900` → Esperado `"CM"`.

## 5. Casos Compuestos Complejos

Pruebas finales para asegurar que todas las reglas funcionan juntas.

- **Caso 19**: Entrada `14` → Esperado `"XIV"`.
- **Caso 20**: Entrada `44` → Esperado `"XLIV"`.
- **Caso 21**: Entrada `99` → Esperado `"XCIX"`.
- **Caso 22**: Entrada `1994` → Esperado `"MCMXCIV"`.
- **Caso 23**: Entrada `3999` → Esperado `"MMMCMXCIX"` (Límite estándar del sistema romano).

## 6. Casos de Validación y Excepciones

Para asegurar la robustez del programa ante entradas inválidas.

- **Caso 24**: Entrada `0` → Esperado `IllegalArgumentException`.
- **Caso 25**: Entrada `-1` → Esperado `IllegalArgumentException`.
- **Caso 26**: Entrada `4000` → Esperado `IllegalArgumentException` (Opcional, según el alcance).
