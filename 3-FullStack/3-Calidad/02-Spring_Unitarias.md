# Unidad 3 - Clase 2: Pruebas Unitarias en Spring Boot con JUnit 5 y Mockito

- **Duración**: 2 horas
- **Objetivo**: El objetivo fundamental de esta sesión es dominar el **aislamiento de lógica de negocio** mediante una mentalidad de Desarrollador de Software. En el desarrollo de sistemas empresariales, la deuda técnica suele acumularse por la falta de una red de seguridad confiable.

Aprenderemos a construir esta red validando la integridad de nuestros servicios mediante pruebas que sean:

- **Atómicas**: Que prueben una única unidad de lógica.
- **Deterministas**: Que siempre produzcan el mismo resultado ante la misma entrada, eliminando efectos secundarios de infraestructura.
- **Eficientes**: Optimizadas para ejecutarse en milisegundos, permitiendo que el desarrollador valide su código tras cada pequeño cambio (TDD _friendly_).

Utilizaremos el estándar de oro de la industria: **JUnit 5** como motor de ejecución y **Mockito** como herramienta de simulación de dependencias.

## Parte 1: Teoría (45 min)

### A. Anatomía y Ciclo de Vida de JUnit 5 (The Jupiter Engine)

JUnit 5 no es solo una actualización; es un rediseño modular que separa la API del motor de ejecución. Esta modularidad permite que Jupiter sea el entorno nativo para escribir pruebas modernas en Java 25.

- `@Test`: Define el método como una unidad de prueba. Internamente, JUnit instancia la clase de prueba para cada método `@Test`, garantizando independencia total entre casos.
- `@BeforeEach` es nuestra herramienta principal para el aislamiento, reseteando los mocks y estados antes de cada prueba.

  ```java
  @BeforeEach
  void init() {
      // Se ejecuta antes de cada @Test
      this.service = new AppointmentService(repository);
  }
  ```

- `@BeforeAll` (estático) se usa para configuraciones pesadas una sola vez (raro en pruebas unitarias puras).
- `@DisplayName`: Un desarrollador escribe código para humanos. Esta anotación permite que el reporte de Jenkins o GitHub Actions muestre "Debería rechazar citas si el médico está de vacaciones" en lugar de `test_err_01()`.

  ```java
  @Test
  @DisplayName("Debería fallar si el paciente es menor de edad")
  void testMinorPatient() { ... }
  ```

### B. El Arsenal de Aserciones de JUnit 5

Las aserciones son los predicados que determinan si un test pasa o falla. JUnit 5 ofrece un catálogo extenso para cubrir diversas necesidades semánticas:

- `assertEquals(esperado, actual)` / `assertNotEquals(...)`: La base de cualquier test. Compara valores primitivos u objetos usando el método `.equals()`.
- `assertTrue(condición)` / `assertFalse(condición)`: Valida estados booleanos o predicados lógicos.
- `assertNull(objeto)` / `assertNotNull(objeto)`: Crítico para validar que servicios o repositorios no retornen nulidad inesperada, o para asegurar que un campo se limpió correctamente.
- `assertSame(obj1, obj2)`: A diferencia de `assertEquals`, esta valida la identidad referencial (que ambos apunten a la misma posición de memoria).
- `assertIterableEquals(lista1, lista2)`: Compara el contenido y el orden de dos colecciones, asegurando que la secuencia de datos sea idéntica.
- `assertAll("Etiqueta", ...lambdas)`: Conocida como _Grouped Assertions_. Permite ejecutar múltiples aserciones incluso si las primeras fallan. Es ideal para validar todos los campos de un objeto de una sola vez sin que el test se detenga en el primer error.
- `assertThrows(Exception.class, () -> código)`: Captura la excepción lanzada. Esto nos permite realizar aserciones adicionales sobre el contenido del error, como códigos de error internos o mensajes personalizados de validación.
- `assertTimeout(Duration, () -> código)`: Valida que un algoritmo cumpla con un SLA (Service Level Agreement) de tiempo máximo de ejecución.

### C. Aislamiento Estratégico con Mockito

Mockito nos permite aplicar el principio de Inversión de Control en nuestras pruebas. Si el `AppointmentService` depende de un `Repository`, el "Contrato" es lo que importa, no la implementación.

| Concepto                    | Profundización Técnica                                        | Rol en el Ecosistema                                                    |
| --------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `@Mock`                     | Crea un objeto Proxy que intercepta todas las llamadas.       | Sustituye componentes pesados o externos (DB, Colas, APIs).             |
| `@InjectMocks`              | Realiza una inyección de dependencias basada en tipos.        | Automatiza la creación del Sujeto Bajo Prueba (SUT).                    |
| **Stubbing (`when`)**       | Programación del comportamiento esperado (Entrada -> Salida). | Define los escenarios (Éxito, Error, Datos vacíos).                     |
| **Verification (`verify`)** | Auditoría de comportamiento posterior a la ejecución.         | Asegura que las reglas de negocio se ejecuten (ej: ¿Se envió el mail?). |

**Ejemplo Detallado de Stubbing**: El stubbing es el arte de predecir el futuro de una dependencia para probar cómo reacciona nuestro código. En escenarios profesionales, no solo devolvemos valores simples; manejamos flujos complejos:

- **Lanzamiento de Excepciones**: Fundamental para probar la resiliencia y el manejo de errores (`try-catch` o `ControllerAdvice`).

  ```java
  // Programamos el mock para que lance una excepción al recibir cualquier ID
  // Esto simula un error de base de datos o una entidad inexistente.
  when(repository.findById(anyLong())).thenThrow(new EntityNotFoundException("No encontrado"));
  ```

- **Respuestas Consecutivas (Iterativas)**: Útil para probar bucles o reintentos donde la primera llamada falla y la segunda tiene éxito.

  ```java
  // La primera llamada devuelve un error, la segunda éxito
  when(externalApi.checkStatus())
      .thenReturn(Status.PENDING)
      .thenReturn(Status.COMPLETED);
  ```

- **Stubbing basado en lógica (ThenAnswer)**: Cuando la respuesta depende de los argumentos de entrada de forma dinámica.

  ```java
  when(repository.save(any(Appointment.class)))
      .thenAnswer(invocation -> invocation.getArgument(0)); // Devuelve el mismo objeto que recibe
  ```

**Verificación por Número de Invocaciones**: A menudo, la lógica de negocio exige que un método se ejecute una cantidad específica de veces. Mockito permite auditar esto con precisión:

- **Exactitud (`times(n)`)**: Este verificador asegura que un método se haya ejecutado exactamente un número determinado de veces. Su uso es crítico para prevenir errores de duplicidad que podrían causar estados inconsistentes en la base de datos o disparar eventos repetidos innecesarios.

  ```java
  // Útil para asegurar que no se guardó la cita dos veces accidentalmente
  verify(repository, times(1)).save(any());
  ```

- **Ausencia de Llamada (`never()`)**: Es la herramienta por excelencia para las pruebas de flujos negativos. En escenarios donde una validación falla, debemos certificar que no se produjeron efectos secundarios, como escrituras en disco o llamadas a APIs externas.

  ```java
  // Si la validación falló, el repositorio NUNCA debe ser llamado
  verify(repository, never()).save(any());
  ```

- **Rangos y Límites (`atLeast`, `atMost`)**: Existen casos donde el número exacto de llamadas no es predecible pero sí debe estar dentro de un rango lógico. Esto es común en procesos de reintento, logging o notificaciones opcionales.

  ```java
  verify(notificationService, atLeastOnce()).send(any());
  verify(logger, atMost(3)).info(anyString());
  ```

- **Control de Flujos Únicos (`only()`)**: Verifica que un método específico fue el único que se llamó en todo el Mock, garantizando que no hubo interacciones colaterales imprevistas que pudieran indicar una fuga de lógica o una mala implementación del servicio.

**Ejemplo Detallado de Verification**: La verificación no solo confirma que el código "pasó por ahí", sino que lo hizo bajo las condiciones exactas de seguridad y negocio:

- **Verificación de Inactividad (Negative Testing)**: Es vital asegurar que si una validación falla, los datos no se persistan.

  ```java
  // Si la lógica de negocio detecta un error previo, garantizamos integridad
  service.process(data); // El proceso debería fallar antes de llegar a la API
  verify(externalApi, never()).sendRequest(any());
  ```

- **Verificación de Orden (InOrder)**: Crucial en sistemas financieros o de reservas donde el paso A debe ocurrir antes que el B.

  ```java
  InOrder inOrder = inOrder(repository, notificationService);
  inOrder.verify(repository).save(any());
  inOrder.verify(notificationService).sendEmail(any());
  ```

- **Verificación Exacta de Llamadas**: Controlar que no existan efectos secundarios inesperados.

  ```java
  // Asegura que no se llamaron a otros métodos de este mock aparte de los verificados
  verifyNoMoreInteractions(repository);
  ```

### D. El Patrón AAA (Arrange, Act, Assert)

Para que una prueba sea legible y profesional, debe seguir una estructura clara que incluso un no-programador pueda seguir:

1. **Arrange (Preparar)**: Configuramos los Mocks y los datos de entrada.
2. **Act (Actuar)**: Invocamos el método específico que estamos probando.
3. **Assert (Verificar)**: Validamos que el resultado y el comportamiento de los mocks coincidan con lo esperado.

### E. Cobertura de Código con JaCoCo (Java Code Coverage)

**JaCoCo** no es solo un generador de reportes; es una herramienta de análisis dinámico que instrumenta el _bytecode_ de Java para detectar qué partes de tu aplicación se ejecutaron realmente durante las pruebas.

#### 1. Desglose de Métricas Críticas

- **Branch Coverage (C1)**: Para un Desarrollador, esta es la métrica reina. Verifica si se han evaluado todos los caminos posibles en estructuras de control (`if`, `switch`, bucles).
  - _Ejemplo_: En un `if (x > 0)`, tener 50% de Branch Coverage significa que solo probaste el caso `true` o el `false`, dejando una vulnerabilidad lógica sin descubrir.
- **Cyclomatic Complexity**: Mide la complejidad de tus métodos basándose en el número de caminos lineales independientes. Un método con complejidad alta y baja cobertura es un candidato número uno para Refactorización o bugs.
- **Line Coverage**: Métrica básica que indica si la línea fue "tocada" por el test. Es útil, pero engañosa (puedes tocar una línea sin validar su lógica).

#### 2. Configuración de Quality Gates (Umbrales de Calidad)

En un entorno profesional, no confiamos en la buena voluntad. Configuramos el sistema de construcción para que falle automáticamente si no se cumplen los estándares.

##### Opción A: Configuración en Maven (`pom.xml`)

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.14</version>
    <executions>
        <!-- 1. Prepara el agente de runtime para instrumentar el código -->
        <execution>
            <goals>
              <goal>prepare-agent</goal>
            </goals>
        </execution>
        <!-- 2. Genera el reporte HTML tras los tests -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
        <!-- 3. REGLA DE ORO: Falla el build si no se cumple el umbral -->
        <execution>
            <id>jacoco-check</id>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum> <!-- Mínimo 80% de líneas -->
                            </limit>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.70</minimum> <!-- Mínimo 70% de ramas lógicas -->
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Para ejecutar las pruebas y ver la cobertura, los alumnos deben ejecutar:

```bash
mvn clean verify
```

##### Opción B: Configuración en Gradle (`build.gradle`)

Para proyectos que usan Gradle, la configuración equivalente se ve así:

```groovy
plugins {
    id 'jacoco'
}

test {
    finalizedBy jacocoTestReport // El reporte se genera después de los tests
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            element = 'PACKAGE'
            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.80 // Mínimo 80% líneas
            }
            limit {
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 0.70 // Mínimo 70% ramas
            }
        }
    }
}

// Obliga a verificar cobertura al ejecutar 'check'
check.dependsOn jacocoTestCoverageVerification
```

Para ejecutar las pruebas y ver la cobertura, los alumnos deben ejecutar:

```bash
./gradlew check
```

#### 3. Interpretación Estratégica

Un 100% de cobertura no garantiza ausencia de bugs (puedes cubrir código con aserciones malas), pero una cobertura baja garantiza deuda técnica. Usamos JaCoCo para encontrar **código muerto** o **casos de borde olvidados**.

### F. Manifiesto de "Clean Testing" (Mejores Prácticas)

Para que una suite de pruebas sea mantenible y escale junto con el proyecto, debemos adherirnos a principios estrictos de diseño. Un mal test es peor que no tener tests, ya que genera falsa confianza y altos costos de mantenimiento.

1. **Naming Convention (Semántica)**: El Código es la Documentación Viva

   El nombre del test es el primer punto de contacto para cualquier desarrollador que necesite entender qué valida ese test, en qué condiciones y qué espera. Un nombre ambiguo (`testSave()`) genera fricción: los desarrolladores deben leer el cuerpo del test para entender su propósito, multiplicando el tiempo de mantenimiento y aumentando la probabilidad de cambios inadecuados durante refactorizaciones.

   En sistemas empresariales, una suite de 500 tests sin nombrado coherente se convierte en un **pasivo de deuda técnica**, donde los propios tests son un riesgo de regresión.
   1. **BDD-Style (Behavior-Driven Development)**

      **Sintaxis**: `should<Resultado>_When<Condición>()`

      **Ejemplos del Caso AppointmentService**:

      ```java
      @Test
      void shouldThrowConflictException_WhenDateTimeAlreadyTaken() { ... }

      @Test
      void shouldPersistAppointment_WhenSlotIsAvailable() { ... }

      @Test
      void shouldRejectMinorPatient_WhenAgeIsUnder18() { ... }
      ```

      **Ventajas**:
      - Comienza con el resultado esperado (`should...`), enfatizando la intención
      - Extremadamente conciso y legible
      - Ideal para TDD (Red-Green-Refactor): escribes primero qué **debe** pasar
      - Reportes de test legibles incluso sin `@DisplayName`

      **Desventajas**:
      - Puede ser ambiguo si hay múltiples condiciones complejas
      - Requiere buen juicio para mantener nombres cortos pero informativos
      - No explícito sobre los datos de entrada (Arrange)

   2. **Given-When-Then (GWT Explícito)**

      **Sintaxis**: `test<Acción>_Given<Precondición>_When<Estímulo>_Then<Resultado>()`

      **Ejemplos del Caso AppointmentService**:

      ```java
      @Test
      void testBookAppointment_GivenExistingConflict_WhenBookingSameTime_ThenThrowException() { ... }

      @Test
      void testBookAppointment_GivenAvailableSlot_WhenProvidingValidData_ThenPersistSuccessfully() { ... }

      @Test
      void testPatientValidation_GivenMinorAge_WhenAttemptingBooking_ThenRejectAndNotify() { ... }
      ```

      **Ventajas**:
      - **Explícitamente estructurado**: cada fase del patrón AAA está en el nombre
      - Excelente para documentación de requisitos (cercano a lenguaje natural)
      - Ideal para equipos distribuidos: no hay ambigüedad
      - Facilita generación automática de reportes de cobertura

      **Desventajas**:
      - Nombres muy largos (> 80 caracteres en muchos IDEs)
      - Verbosidad puede abrumar si hay muchas combinaciones de casos
      - Difícil de leer en algunos contextos (p.ej., grep o búsquedas rápidas)

      **Nota**: Este patrón es el estándar en metodologías Agile/BDD formales (Cucumber, SpecFlow).

   3. **AAA-Pattern (Enfocado en Flujo)**

      **Sintaxis**: `test<Método><Resultado>_With<Condición>()`

      **Ejemplos del Caso AppointmentService**:

      ```java
      @Test
      void testBookingConflictDetection_WithOccupiedSlot() { ... }

      @Test
      void testAppointmentPersistence_WithValidData() { ... }

      @Test
      void testPatientAgeValidation_WithMinorAge() { ... }
      ```

      **Ventajas**:
      - Equilibrio entre concisión y claridad
      - Enfatiza qué se está probando (testX) y bajo qué condición (WithY)
      - Fácil de escanear en listas de tests
      - Nombres moderados en longitud

      **Desventajas**:
      - Menos descriptivo que GWT para requisitos complejos
      - Requiere que el lector abra el test para entender qué es "conflict detection"
      - No es tan "legible en lenguaje natural"

   4. **Técnico/Legacy JUnit Style**

      **Sintaxis**: `test<Método>` o `test<Método><Resultado>`

      **Ejemplos del Caso AppointmentService**:

      ```java
      @Test
      void testBookAppointment() { ... }           // ❌ Anti-patrón: demasiado vago

      @Test
      void testBookAppointmentConflict() { ... }   // ⚠️ Ambiguo: ¿qué tipo de conflicto?

      @Test
      void testBookAppointmentSuccess() { ... }    // ⚠️ Mejor, pero aún genérico
      ```

      **Ventajas**:
      - Conciso al extremo
      - Nombres cortos, fáciles de escribir
      - Herencia de código legacy de hace 10+ años

      **Desventajas**:
      - **No recomendado en código nuevo**: fuerza a leer el cuerpo del test
      - Sin contexto de negocio: `testBookAppointment()` no dice si es éxito o fallo
      - Genera reportes de tests inútiles ("Test: testBookAppointment - FAILED")
      - Aumenta significativamente el tiempo de debugging

   5. **Descriptivo Simple (Narrativo)**

      **Sintaxis**: `should<Acción><Resultado>_When<Condición>()` (narrativa fluida)

      **Ejemplos del Caso AppointmentService**:

      ```java
      @Test
      void shouldRejectDuplicateAppointmentTime() { ... }

      @Test
      void shouldAllowBookingWhenSlotIsFree() { ... }

      @Test
      void shouldValidatePatientAgeBeforePersistence() { ... }
      ```

      **Ventajas**:
      - Lee como una oración natural: "Should reject duplicate appointment time"
      - Menos "técnico", más "requisito"
      - Versátil para tutoriales y documentación educativa
      - Fácil de traducir a otros idiomas

      **Desventajas**:
      - Menos estructurado que GWT
      - Requiere disciplina para evitar ambigüedades
      - Pueden omitirse detalles sobre condiciones complejas

   **Tabla Comparativa de Patrones**

   | Patrón                 | Sintaxis                             | Longitud  | Claridad | Estructurado     | Escalable | Recomendado para                     |
   | ---------------------- | ------------------------------------ | --------- | -------- | ---------------- | --------- | ------------------------------------ |
   | **BDD-Style**          | `should<R>_When<C>()`                | Corta     | Alta     | Moderada         | Buena     | TDD, Equipos ágiles                  |
   | **Given-When-Then**    | `test<A>_Given<P>_When<S>_Then<R>()` | Muy Larga | Muy Alta | Muy Estructurada | Excelente | Specs formales, Equipos distribuidos |
   | **AAA-Pattern**        | `test<M><R>_With<C>()`               | Moderada  | Buena    | Moderada         | Buena     | Proyectos balance                    |
   | **Legacy/Técnico**     | `test<M>()`                          | Muy Corta | Baja     | Nula             | Pobre     | ❌ Evitar en código nuevo            |
   | **Descriptivo Simple** | `should<A><R>_When<C>()`             | Moderada  | Alta     | Moderada         | Buena     | Documentación, Educación             |

   **Anti-patrones a Evitar**

   ```java
   // ❌ ANTI-PATRÓN 1: Nombres genéricos
   void testSave() { ... }           // ¿Save qué? ¿Éxito o error?
   void testError() { ... }          // ¿Qué tipo de error?
   void test1() { ... }              // Totalmente inútil

   // ❌ ANTI-PATRÓN 2: Nombres que documentan lógica de test, no requisito
   void testSaveAndThenUpdate() { ... }  // Multiple responsabilidades
   void testLoopThroughArray() { ... }   // Detalles de implementación

   // ❌ ANTI-PATRÓN 3: Nombres con jerga de infraestructura
   void testJdbcConnection() { ... }  // En un test unitario, no debe depender de JDBC
   void testMockRepository() { ... }  // El nombre es sobre el test, no sobre el requisito

   // ❌ ANTI-PATRÓN 4: Nombres que dicen lo que hace, no por qué falla
   void testNullPointerException() { ... }  // Síntoma, no causa
   void testAssertionFailed() { ... }       // Circulación: el test SIEMPRE afirma algo
   ```

   **Matriz de Decisión: Qué Patrón Usar**

   ```plain
   ¿Es tu proyecto...?

   ├─ NUEVO desde cero
   │  └─ ¿Tienes metodología BDD/TDD formalmente definida?
   │     ├─ SÍ (Scrum/Kanban con historias de usuario)
   │     │  └─ Usa: GIVEN-WHEN-THEN o BDD-STYLE
   │     └─ NO
   │        └─ Usa: BDD-STYLE (más pragmático)
   │
   ├─ LEGACY / Heredado
   │  ├─ ¿Está adoptando mejoras de testing?
   │  │  ├─ Incrementalmente
   │  │  │  └─ Usa: BDD-STYLE (introduce sin disrupción)
   │  │  └─ Completa renovación
   │  │     └─ Usa: GIVEN-WHEN-THEN (reinicio limpio)
   │  └─ Mantenimiento solamente
   │     └─ Usa: AAA-PATTERN (balance pragmático)
   │
   ├─ EQUIPO DISTRIBUIDO / REMOTO
   │  └─ Usa: GIVEN-WHEN-THEN (máxima claridad sin ambigüedad)
   │
   ├─ EDUCATIVO / Bootcamp
   │  └─ Usa: DESCRIPTIVO-SIMPLE (enfatiza intención, no jerga)
   │
   └─ STARTUP / MVP rápido
       └─ Usa: BDD-STYLE (velocidad + claridad balance)
   ```

   **Integración con `@DisplayName` (El Complemento Perfecto)**

   El nombre del método es para el **código** (buscabilidad, refactorización). `@DisplayName` es para **reportes** (Jenkins, GitHub Actions, SonarQube):

   ```java
   @Test
   @DisplayName("❌ Rechazar reserva de cita si el horario ya está ocupado")
   void shouldThrowConflictException_WhenDateTimeAlreadyTaken() {
       // Arrange
       Appointment existing = new Appointment(1L, LocalDateTime.now().plusDays(1), "Dr. García");
       when(repository.findByDateTime(any())).thenReturn(Optional.of(existing));

       // Act & Assert
       AppointmentConflictException ex = assertThrows(
           AppointmentConflictException.class,
           () -> service.bookAppointment(new Appointment(null, LocalDateTime.now().plusDays(1), "Paciente"))
       );
       assertTrue(ex.getMessage().contains("reservado"));
   }
   ```

   En el reporte de Jenkins, verás:

   ```plain
   ❌ Rechazar reserva de cita si el horario ya está ocupado
   ```

   En búsquedas de código, seguirás encontrando por:

   ```plain
   shouldThrowConflictException_WhenDateTimeAlreadyTaken
   ```

   **Recomendación Final: Adopta UNA Convención y Sé Consistente**

   _La mejor convención es la que elige tu EQUIPO y respeta en TODOS los tests._
   - **Startups / Pequeños Equipos**: BDD-STYLE
   - **Equipos Grandes / Distribuidos**: GIVEN-WHEN-THEN
   - **Migrando código legacy**: AAA-PATTERN
   - **Proyectos educativos**: DESCRIPTIVO-SIMPLE

   _Implementa una rule en SonarQube o Checkstyle_ para validar automáticamente que los tests sigan el patrón elegido. Esto evita debates sin fin en PRs.

2. **Determinismo Absoluto (No Flaky Tests)**: Un test debe pasar hoy, mañana, en el año 3000 y en cualquier zona horaria.
   - **Prohibido**: Usar `LocalDateTime.now()` o `new Date()` directamente dentro de la lógica de negocio sin inyectar un `Clock`.
   - **Solución**: Simular (Mock) el proveedor de tiempo para controlar el "ahora" durante el test.
3. **Independencia y Aislamiento**: Nunca encadenes tests. El resultado del _Test A_ no debe ser pre-condición para el _Test B_. JUnit no garantiza el orden de ejecución. Cada test debe preparar su propio entorno (`@BeforeEach`) y limpiarlo si es necesario.
4. **No Logic inside Tests**: Si tu test contiene estructuras de control como `if`, `while`, `for` o `switch`, es una señal de alta complejidad ciclomática en el test.
   - _Regla_: El flujo del test debe ser lineal: _Arrange -> Act -> Assert_. Si necesitas lógica condicional, probablemente necesitas dos tests separados.
5. **Una Razón para Fallar (Single Responsibility Principle)**: Evita probar múltiples comportamientos en un solo método (`testSaveAndThenDelete`). Si falla, el diagnóstico es confuso. Divide y vencerás: un test para el guardado, otro para el borrado.

### G. Justificación Arquitectónica del Aislamiento

¿Por qué evitar `@SpringBootTest` en esta fase?

- **Costo de Arranque**: Levantar el _ApplicationContext_ de Spring implica escanear componentes, cargar Beans y configurar el entorno. Multiplicado por 500 pruebas, esto destruye la agilidad del equipo.
- **Falsos Positivos/Negativos**: En un test de integración, el test puede fallar por una mala configuración del `application-test.yml` y no por un error en el código Java. El aislamiento con `MockitoExtension` garantiza que el fallo es lógico, no de infraestructura.

## Parte 2: Laboratorio (1h 15m)

### A. Escenario del Laboratorio: Sistema de Citas Médicas

Implementaremos un motor de reglas para la reserva de citas en una clínica. Este escenario es ideal para pruebas unitarias porque contiene lógica condicional crítica que no debe depender de si la base de datos está encendida o apagada.

**Reglas de Negocio a Implementar**:

1. **Validación de Disponibilidad**: Antes de guardar, el sistema debe consultar el repositorio. Si existe una cita en el mismo `LocalDateTime`, la operación debe abortar.
2. **Persistencia Atómica**: Solo si la validación es exitosa, se debe proceder al método `save()`.
3. **Gestión de Errores**: En caso de conflicto, se debe propagar una excepción personalizada que el controlador pueda mapear a un código HTTP 409 (Conflict).

### B. Implementación de la Infraestructura de Código (Java 25)

#### Definición del Modelo y Excepciones

```java
// Los Records de Java 25 eliminan el ruido visual de Getters/Setters
public record Appointment(Long id, LocalDateTime dateTime, String patientName) {}
```

```java
// Excepción de dominio para desacoplar el negocio del framework
public class AppointmentConflictException extends RuntimeException {
    public AppointmentConflictException(String message) {
        super(message);
    }
}
```

#### Lógica del Servicio (El SUT - System Under Test)

```java
@Service
public class AppointmentService {
    private final AppointmentRepository repository;

    // Inyección por constructor: La mejor práctica para facilitar el testing
    public AppointmentService(AppointmentRepository repository) {
        this.repository = repository;
    }

    public Appointment bookAppointment(Appointment appointment) {
        // 1. Consultar disponibilidad (Llamada al Mock)
        repository.findByDateTime(appointment.dateTime())
            .ifPresent(a -> {
                // 2. Si hay conflicto, lanzamos excepción y detenemos el flujo
                throw new AppointmentConflictException("El horario ya está reservado: " + appointment.dateTime());
            });

        // 3. Persistir si todo es correcto
        return repository.save(appointment);
    }
}
```

#### C. Suite de Pruebas Unitarias de Alto Nivel

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;
import org.mockito.InjectMocks;
import org.mockito.Mock;

@ExtendWith(MockitoExtension.class) // Inicializa los mocks de forma limpia
class AppointmentServiceTest {

    @Mock
    private AppointmentRepository repository;

    @InjectMocks
    private AppointmentService service;

    private Appointment appointmentData;

    @BeforeEach
    void init() {
        // Datos comunes para los escenarios
        appointmentData = new Appointment(10L, LocalDateTime.now().plusDays(2), "Ana Smith");
    }

    @Test
    @DisplayName("ESCENARIO ÉXITO: Reservar cita cuando el slot está vacío")
    void shouldCompleteBookingProcess() {
        // 1. ARRANGE
        when(repository.findByDateTime(any())).thenReturn(Optional.empty());
        when(repository.save(any())).thenReturn(appointmentData);

        // 2. ACT
        Appointment saved = service.bookAppointment(appointmentData);

        // 3. ASSERT
        assertAll("Verificaciones de Persistencia",
            () -> assertNotNull(saved, "El objeto guardado no debe ser nulo"),
            () -> assertEquals("Ana Smith", saved.patientName()),
            // Verificamos comportamiento: El repositorio DEBIÓ ser llamado
            () -> verify(repository, times(1)).save(any())
        );
    }

    @Test
    @DisplayName("ESCENARIO FALLO: Detectar conflicto de horario y abortar")
    void shouldFailWhenDateTimeIsAlreadyTaken() {
        // 1. ARRANGE
        Appointment existingApp = new Appointment(1L, appointmentData.dateTime(), "Paciente Existente");
        when(repository.findByDateTime(appointmentData.dateTime()))
            .thenReturn(Optional.of(existingApp));

        // 2. ACT & ASSERT
        // Validamos que se lance la excepción correcta
        AppointmentConflictException ex = assertThrows(AppointmentConflictException.class, () -> {
            service.bookAppointment(appointmentData);
        });

        // Validamos el mensaje de la excepción
        assertTrue(ex.getMessage().contains("horario ya está reservado"));

        // VERIFICACIÓN DE SEGURIDAD:
        // El repositorio NUNCA debe intentar guardar si hubo conflicto
        verify(repository, never()).save(any());
    }
}
```

#### D. Principios de "Clean Testing" para el Alumno

Para que estas pruebas sean mantenibles a largo plazo, se deben seguir estos consejos:

1. **Don't Logic in Tests**: No uses `if` ni `for` dentro de tus tests. Si necesitas lógica, tu test es demasiado complejo.
2. **Single Responsibility**: Un test debería fallar por una sola razón de negocio.
3. **Descriptive Naming**: El nombre del método debe explicar la regla de negocio que se está validando.

## Recursos y Referencias

- **Effective Unit Testing**: Libro recomendado sobre patrones de diseño de pruebas.
- [JUnit 6 User Guide](https://docs.junit.org/6.0.3/overview.html): Referencia técnica completa sobre el motor Jupiter.
- [Mockito Official Site](https://site.mockito.org/): Guía completa para aprender a realizar stubs, verifications y el uso de anotaciones.
- [Java 25 (JEP 459 - Records)](https://openjdk.org/jeps/459): Documentación sobre cómo los Records mejoran la inmutabilidad y reducen el código repetitivo en Java moderno.
- [Baeldung - Spring Boot Testing](https://www.baeldung.com/spring-boot-testing): Un recurso excelente con ejemplos prácticos sobre los diferentes niveles de prueba en Spring.
- [JaCoCo Official Site](https://www.jacoco.org/)
