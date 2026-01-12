# Preguntas de entrevista del Módulo 1: Java Junior Developer

Este documento recopila preguntas técnicas detalladas, diseñadas no solo para validar la memorización de conceptos, sino para demostrar una comprensión profunda de la ingeniería de software en Java moderno. Cubre desde la configuración del entorno hasta el procesamiento avanzado de datos.

## Unidad 1: Fundamentos Modernos de Programación

### Q1: ¿Por qué ignoramos la carpeta `.vscode`, `target` o `.idea` en el archivo `.gitignore`?

- **Archivos Generados**: Carpetas como `target` (Maven) o `build` (Gradle) contienen el resultado de la compilación (`.class`, `.jar`). Estos archivos son redundantes porque se pueden regenerar a partir del código fuente. Incluirlos infla innecesariamente el tamaño del repositorio.

- **Conflictos de Entorno**: Carpetas como `.vscode` o `.idea` contienen configuraciones específicas del IDE del usuario (rutas locales, preferencias de color, configuraciones de ejecución). Si se comparten, obligan a todos los miembros del equipo a usar la misma configuración, lo que genera conflictos constantes al hacer git merge cuando dos desarrolladores tienen preferencias distintas.

- **Seguridad**: A menudo, archivos de entorno local (`.env`) o logs (`*.log`) pueden contener claves API o información sensible que nunca debe subir al repositorio remoto.

### Q2: ¿Cuál es la diferencia técnica entre el JDK y el JRE?

- **JRE (Java Runtime Environment)**: Es el entorno mínimo necesario para ejecutar una aplicación Java. Contiene la JVM (Java Virtual Machine), las librerías del núcleo (como `java.lang`, `java.util`) y otros componentes necesarios para correr bytecode.

- `JDK (Java Development Kit)`: Es un superconjunto del JRE diseñado para desarrolladores. Además de todo lo que trae el JRE, incluye herramientas vitales:
  - `javac`: El compilador que transforma código fuente `.java` en bytecode `.class`.
  - `javadoc`: Generador de documentación HTML.
  - `jdb`: El debugger de línea de comandos.
  - `jar`: Herramienta para empaquetar librerías.

- _Nota moderna_: Desde Java 11, la distribución de JRE por separado es menos común; los desarrolladores suelen instalar el JDK y usan herramientas como `jlink` para crear runtimes personalizados y livianos para producción.

### Q3: ¿Qué es la inferencia de tipos con `var` y cuáles son sus limitaciones estrictas?

- **Concepto**: Introducido en Java 10, `var` permite al compilador deducir el tipo de la variable basándose en el valor asignado a la derecha (`var lista = new ArrayList<String>();` infiere `ArrayList<String>`). Reduce el "ruido" visual sin sacrificar el tipado estático fuerte; la variable sigue teniendo un tipo fijo en tiempo de compilación.

- **Limitaciones**:
    1. **Solo variables locales**: No se puede usar para atributos de clase (campos) ni parámetros de métodos, ya que el tipo debe ser explícito para la API pública de la clase.
    2. **Inicialización obligatoria**: `var x;` causa error de compilación porque el compilador no tiene información para deducir el tipo.
    3. **No nulos puros**: `var x = null;` es ilegal porque `null` no tiene tipo por sí mismo.

### Q4: ¿Cuál es la ventaja arquitectónica de usar un `switch` como expresión (Java 14+) frente al clásico?

- **Inmutabilidad y Retorno**: El `switch` clásico era una estructura de control que ejecutaba acciones (side-effects). El `switch` moderno es una _expresión_ que evalúa y _retorna_ un valor, lo que permite asignarlo directamente a una variable `final` o retornarlo, fomentando la inmutabilidad.
- **Seguridad (Exhaustiveness)**: El compilador obliga a cubrir todos los casos posibles (o poner un `default`), eliminando bugs silenciosos donde un caso no se maneja.
- **Sintaxis Limpia**: El uso de la flecha `->` elimina la necesidad de escribir `break;` en cada caso, previniendo el error común de "fall-through" accidental (donde la ejecución pasa al siguiente caso por error).
- Ejemplo: `int dias = switch(mes) { case ABRIL, JUNIO -> 30; default -> 31; };`

### Q5: ¿Para qué sirven los **Text Blocks** (""") y qué problema de mantenimiento resuelven?

- **El Problema**: Antes de Java 15, escribir strings multilínea (como JSON, SQL o HTML) requieria concatenación excesiva (`+)` y caracteres de escape (`\"`, `\n`), lo que hacía el código ilegible y difícil de copiar/pegar desde otras herramientas.
- **La Solución**: Los Text Blocks permiten pegar el texto tal cual es.
- **Incidental Whitespace**: Una característica clave es que el compilador es inteligente eliminando la indentación "accidental". Si indentas el bloque de texto para que coincida con tu código Java, Java eliminará esos espacios iniciales comunes para no alterar el contenido real del string.

### Q6: En el contexto de Debugging, ¿qué es un "Breakpoint Condicional" y cuándo es indispensable?

- **Definición**: Es un punto de interrupción que no detiene el programa cada vez que la línea se ejecuta, sino solo cuando una expresión booleana definida por el programador se evalúa como `true`.
- **Caso de Uso**: Imagina un bucle `for` que itera 10,000 veces procesando transacciones, y sabes que el error ocurre solo en la transacción con ID "TX-99". Poner un breakpoint normal te obligaría a presionar "Continuar" miles de veces. Un breakpoint condicional con `id.equals("TX-99")` detiene la ejecución exactamente en el momento del error, ahorrando horas de depuración.

## Unidad 2: Programación Orientada a Objetos (Modern OOP)

### Q7: ¿Por qué el uso de `double` es inaceptable para sistemas financieros y por qué `BigDecimal` es la solución?

- **Punto Flotante IEEE 754**: Los tipos primitivos `float` y `double` están diseñados para cálculos científicos rápidos, no para precisión exacta. Representan números en binario, y valores simples como `0.1` tienen representaciones binarias periódicas infinitas.
- **El Error**: `0.1 + 0.2` en `double` resulta en `0.30000000000000004`. En contabilidad, estos micro-errores se acumulan y causan descuadres financieros legales.
- **BigDecimal**: Almacena el número como un entero arbitrariamente grande no escalado y una escala decimal (básicamente, cuenta los dígitos exactos). Permite control total sobre las reglas de redondeo (ej. `RoundingMode.HALF_EVEN` para redondeo bancario), garantizando precisión absoluta.

### Q8: ¿Qué diferencia conceptual y práctica existe entre un `record` y una clase tradicional (JavaBean/POJO)?

- **Propósito**: Un JavaBean/POJO es mutable y orientado a encapsular estado que puede cambiar. Un `record` es un portador de datos transparente e inmutable.
- **Inmutabilidad**: Los campos de un `record` son `private final` por defecto. No hay setters. Esto los hace ideales para **DTOs** (Data Transfer Objects), claves de mapas y programación concurrente (thread-safe por defecto).
- **Semántica**: Java asume que el estado del `record` es su identidad. Genera automáticamente `equals()` y `hashCode()` basándose en todos los campos, asegurando que dos records con los mismos datos sean intercambiables, algo que en clases normales debe implementarse manualmente y es propenso a errores.

### Q9: Explica en profundidad la diferencia entre Composición y Agregación con un ejemplo de ciclo de vida

- **Agregación (Relación débil)**: El objeto hijo puede existir independientemente del padre.
  - **_Ejemplo_**: Un `Equipo` de fútbol y un `Jugador`. Si el equipo se disuelve (se elimina el objeto Equipo), los jugadores siguen existiendo como agentes libres. En código, el `Jugador` se pasa al Equipo (usualmente por constructor o setter) pero no se crea dentro de él.
- **Composición (Relación fuerte)**: El objeto hijo no tiene sentido o vida fuera del padre. El padre gestiona el ciclo de vida del hijo.
  - **_Ejemplo_**: Una `Casa` y una `Habitación`. Si demueles la casa (`objeto Casa = null`), las habitaciones dejan de existir físicamente. En código, la `Habitación` se instancia dentro del constructor de la `Casa`, asegurando que mueran juntas.

### Q10: ¿Cómo cambia el diseño de software el uso de `sealed classes` (clases selladas)?

- Antes de Java 17, la herencia era "abierta para todos" (public/protected) o "cerrada para todos" (final). No había un punto medio.
- `sealed classes` permite definir una jerarquía de dominio cerrada y finita. Tú dices: "Un `ResultadoPago` solo puede ser `Exito` o `Fallo`, nada más".
- Esto permite al compilador razonar sobre tu dominio. Si alguien intenta crear `class Hack extends ResultadoPago` fuera de tu control, el código no compilará. Es fundamental para modelar dominios seguros y predecibles.

### Q11: ¿Qué implica la "verificación de exhaustividad" (exhaustiveness checking) en el Pattern Matching?

- Es una característica de seguridad del compilador. Cuando usas un `switch` sobre una clase sellada o un Enum, el compilador sabe exactamente cuántas subclases posibles existen.
- Si escribes un switch que cubre `Exito` pero olvidas `Fallo`, el compilador lanza un error: _"the switch expression does not cover all possible input values"_.
- Esto elimina la necesidad de escribir bloques `default` defensivos ("por si acaso") y convierte lo que antes eran errores en tiempo de ejecución (bugs) en errores de compilación (fáciles de arreglar).

## Unidad 3: Calidad de Software, Testing y Manejo de Errores

### Q12: ¿Cómo funciona internamente `try-with-resources` y qué problema de "excepción suprimida" resuelve?

- **Mecánica**: Cualquier objeto que implemente `AutoCloseable` puede declararse dentro del paréntesis del `try`. Al finalizar el bloque (ya sea exitosamente o por error), Java llama automáticamente al método `close()` del recurso.
- **El Problema Antiguo**: En el bloque `finally` clásico, si cerrabas un recurso y eso lanzaba una excepción mientras ya había ocurrido otra excepción en el `try`, la excepción original se perdía (era enmascarada).
- **La Solución**: `try-with-resources` maneja esto elegantemente. Si tanto el bloque `try` como el `close()` fallan, la excepción del `close()` se agrega como una "excepción suprimida" (suppressed) a la excepción principal, permitiéndote ver ambos errores en el stack trace.

### Q13: ¿Cuándo es arquitectónicamente correcto crear una Excepción Personalizada (Custom Exception)?

- No debes crear excepciones para todo. Usa las estándar (`IllegalArgumentException`, `IllegalStateException`) cuando apliquen.
- Crea una propia cuando el error represente una regla de negocio específica que el código llamador podría querer manejar de forma diferenciada.
  - _**Ejemplo**_: `SaldoInsuficienteException`. Esto le dice claramente al controlador "El problema no es técnico (null pointer), es que el usuario no tiene dinero". Permite capturar (`catch`) esa regla específica y mostrar un mensaje amigable al usuario, mientras dejas pasar los errores técnicos al log de errores.

### Q14: ¿Por qué se considera un anti-patrón usar Log4j 2 directamente sin pasar por SLF4J?

- **Acoplamiento**: Si usas las clases de Log4j directamente (`org.apache.logging.log4j.Logger`), tu código queda atado a esa librería específica.
- **Patrón Facade**: SLF4J (`org.slf4j.Logger`) es una interfaz genérica. Al programar contra la interfaz, puedes cambiar la implementación subyacente (por ejemplo, pasar de **Log4j** a **Logback** o a **JUL (`java.util.logging`)**) simplemente cambiando una dependencia en el `pom.xml`, sin tocar una sola línea de código Java. Esto es vital en el mantenimiento de software a largo plazo y en la resolución de conflictos de dependencias en proyectos grandes.

### Q15: En JUnit 5, ¿cuál es la diferencia de ciclo de vida entre `@BeforeEach` y `@BeforeAll` y cuándo usar cada uno?

- `@BeforeEach`: Se ejecuta antes de cada test individual. Es ideal para preparar un estado limpio, como reiniciar variables o vaciar una lista, asegurando que los tests sean independientes entre sí (un test no afecta al siguiente).
- `@BeforeAll`: Se ejecuta una sola vez antes de que se instancie la clase de test. Debe ser estático. Se usa para operaciones costosas que se pueden compartir, como levantar una conexión a base de datos en memoria o cargar un archivo de configuración pesado. Usarlo mal puede hacer que los tests compartan estado sucio.

### Q16: ¿Qué es el concepto de "Mocking" y cómo se diferencia de un "Stub"?

- Ambos son "dobles de prueba" (Test Doubles) usados para aislar la unidad bajo prueba.
- **Stub**: Es un objeto bobo que devuelve datos predefinidos ("Si te llaman, devuelve 5"). Se usa para simular el estado.
- **Mock (Mockito)**: Es más inteligente. No solo devuelve datos, sino que verifica comportamiento. Con un Mock puedes preguntar: "¿Se llamó al método `guardar()` exactamente una vez con el argumento 'X'?".
- **Uso**: Sirven para testear tu lógica sin depender de sistemas externos lentos o impredecibles (API REST de terceros, Base de Datos real).

## Unidad 4: Colecciones y Streams en Java

### Q17: ¿Qué problema de inconsistencia resuelven las _Sequenced Collections_ (Java 21)?

- Antes de Java 21, obtener el primer elemento de una colección variaba absurdamente según el tipo: `list.get(0)`, `deque.getFirst()`, `sortedSet.first()`. Peor aún, obtener el último en un `List` requería `list.get(list.size() - 1)`.
- Java 21 introduce interfaces como `SequencedCollection` que todas estas clases implementan. Ahora, sin importar si es una lista o un set ordenado, siempre tienes `addFirst()`, `getLast()`, y el poderoso `reversed()` que devuelve una vista inversa de la colección sin copiar datos.

### Q18: ¿Por qué se rompe un `HashMap` si el objeto clave implementa `equals` pero no `hashCode`?

- **Contrato**: Si dos objetos son iguales según equals, deben tener el mismo `hashCode`.
- **Funcionamiento**: El `HashMap` usa el hash para saber en qué "cubeta" (bucket) de memoria buscar. Si no sobrescribes `hashCode`, se usa la dirección de memoria por defecto.
- **Consecuencia**: Puedes insertar un objeto y luego buscarlo con otro objeto idéntico (mismos datos), pero como tienen hash codes diferentes, el mapa mirará en la cubeta equivocada y devolverá `null`. Es un error lógico crítico y silencioso.

### Q19: ¿Qué significa que los Streams sean "Lazy" (perezosos) y cómo afecta al rendimiento?

- **Definición**: Invocar operaciones intermedias (`filter`, `map`) no procesa datos inmediatamente; solo construye una "receta" o pipeline de ejecución.
- **Ejecución**: Nada sucede hasta que se llama a una operación terminal (`collect`, `findFirst`).
- **Beneficio (Short-circuiting)**: Si tienes un Stream de 1 millón de elementos y haces `.filter(...).findFirst()`, el sistema no filtra el millón. Procesa uno por uno y se detiene tan pronto encuentra el primero que cumple, ignorando el resto. Esto hace que los Streams sean eficientes incluso con datos infinitos o masivos.

### Q20: ¿Por qué `Files.lines()` es crítico para el procesamiento de Big Data en archivos locales?

- **El enfoque ingenuo**: `Files.readAllLines()` carga todo el contenido del archivo en una `List<String>` en la memoria RAM (Heap). Si el archivo pesa 2GB y tienes 1GB de RAM, la aplicación crashea con `OutOfMemoryError`.
- **El enfoque Stream**: `Files.lines()` crea un Stream conectado al archivo en disco. Lee una línea, la procesa, y la descarta de memoria antes de leer la siguiente. El consumo de memoria es constante y mínimo, sin importar si el archivo pesa 5KB o 5TB. Nota: Al ser un recurso I/O, el Stream debe cerrarse, por lo que debe usarse dentro de un `try-with-resources`.

## Q21: ¿Cuándo es obligatorio usar `flatMap` en lugar de `map`?

- Usa `map` cuando la transformación es 1 a 1 (ej. Persona -> String nombre).
- Usa `flatMap` cuando la transformación es 1 a N (uno a muchos) o cuando la transformación devuelve un Stream por sí misma.
- _**Ejemplo**_: Tienes una lista de `Pedidos`, y cada Pedido tiene una lista de `Productos`. Si quieres obtener un listado único de todos los productos vendidos:
- `map` devolvería `Stream<List<Productos>>` (Streams anidados, difícil de usar).
- `flatMap` "aplana" esas listas internas, devolviendo un único `Stream<Producto>` continuo listo para ser procesado.

## Coding Challenge (Live Coding)

**Problema**: "Tienes un archivo CSV `ventas.csv` con tres columnas (id, producto, monto). Escribe un método que lea el archivo, filtre las ventas mayores a 100, y devuelva la suma total agrupada por producto. Debes manejar correctamente los recursos."

```java
public Map<String, Double> sumarVentasPorProducto(Path ruta) throws IOException {
    // try-with-resources es OBLIGATORIO aquí porque Files.lines abre un archivo en disco.
    // Si no se cierra, el archivo queda bloqueado por el sistema operativo.
    try (Stream<String> lineas = Files.lines(ruta)) {
        return lineas
            .skip(1) // 1. Saltar cabecera (header del CSV)
            .map(linea -> linea.split(",")) // 2. Transformar String -> Array de Strings
            // Validación defensiva básica (opcional pero recomendada en entrevistas)
            .filter(parts -> parts.length >= 3) 
            // 3. Convertir el monto a double y filtrar
            .filter(parts -> {
                double monto = Double.parseDouble(parts[2]);
                return monto > 100.0;
            })
            // 4. Operación Terminal: Agrupar y reducir
            .collect(Collectors.groupingBy(
                parts -> parts[1], // La Clave del mapa es el nombre del producto (columna 1)
                // Downstream collector: Qué hacer con los valores agrupados
                Collectors.summingDouble(parts -> Double.parseDouble(parts[2])) 
            ));
    }
}
```
