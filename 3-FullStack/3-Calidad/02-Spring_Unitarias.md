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

#### Anotaciones Esenciales

- `@Test`: Define el método como una unidad de prueba. Internamente, JUnit instancia la clase de prueba para cada método `@Test`, garantizando independencia total entre casos.
- `ParameterizedTest`: Permite ejecutar el mismo test con múltiples conjuntos de datos, ideal para validar reglas de negocio con diferentes escenarios sin duplicar código.
  - `@ValueSource`: Para tipos primitivos o Strings.  
    _Ejemplo_: Validar que un paciente menor de edad no pueda agendar una cita, probando con edades 0, 5, 17.

    ```java
    @ParameterizedTest
    @ValueSource(ints = {0, 5, 17})
    void testMinorPatient(int age) {
        Patient patient = new Patient(age);
        assertFalse(service.canSchedule(patient));
    }
    ```

  - `@CsvSource`: Para múltiples parámetros.  
    _Ejemplo_: Validar combinaciones de roles y permisos para acceder a un recurso.

    ```java
    @ParameterizedTest
    @CsvSource({
        "ADMIN, true",
        "USER, false",
        "GUEST, false"
    })
    void testAccessControl(String role, boolean expected) {
        User user = new User(role);
        assertEquals(expected, service.hasAccess(user));
    }
    ```

  - `@MethodSource`: Para casos complejos donde los datos de prueba requieren lógica de construcción o son objetos personalizados.  
    _Ejemplo_: Validar la capacidad de agendar citas para pacientes con diferentes edades y condiciones médicas.

    ```java
    @ParameterizedTest
    @MethodSource("providePatients")
    void testPatientScheduling(Patient patient, boolean expected) {
        assertEquals(expected, service.canSchedule(patient));
    }

    static Stream<Arguments> providePatients() {
        return Stream.of(
            Arguments.of(new Patient(0), false),
            Arguments.of(new Patient(5), false),
            Arguments.of(new Patient(17), false),
            Arguments.of(new Patient(18), true)
        );
    }
    ```

- `@BeforeEach` es nuestra herramienta principal para el aislamiento, reseteando los mocks y estados antes de cada prueba.

  ```java
  @BeforeEach
  void init() {
      // Se ejecuta antes de cada @Test
      this.service = new AppointmentService(repository);
  }
  ```

- `@AfterEach` se usa para limpiar recursos o reiniciar los estados que podrían afectar a pruebas posteriores, aunque en pruebas unitarias puras es menos común.
- `@BeforeAll` (estático) se usa para configuraciones pesadas una sola vez (raro en pruebas unitarias puras).
- `@AfterAll` (estático) para liberar recursos globales, como conexiones a bases de datos en pruebas de integración.
- `@DisplayName`: Un desarrollador escribe código para humanos. Esta anotación permite que el reporte de Jenkins o GitHub Actions muestre "Debería rechazar citas si el médico está de vacaciones" en lugar de `test_err_01()`.

  ```java
  @Test
  @DisplayName("Debería fallar si el paciente es menor de edad")
  void testMinorPatient() { ... }
  ```

#### `@ExtendWith` en JUnit 5

`@ExtendWith` es la anotación que permite registrar una o más extensiones de JUnit 5 para una clase de prueba.

##### Qué hace

- Le dice a JUnit que use una extensión específica durante el ciclo de vida de la prueba.
- La extensión puede:
  - inicializar mocks
  - manejar inyección de dependencias
  - controlar el ciclo de vida de los tests
  - ejecutar lógica antes/después de cada prueba o antes/después de toda la clase

##### Ejemplo típico

```java
@ExtendWith(MockitoExtension.class)
class AppointmentServiceTest {
    @Mock
    private AppointmentRepository repository;

    @InjectMocks
    private AppointmentService service;

    // tests...
}
```

Aquí `MockitoExtension`:

- crea los mocks marcados con `@Mock`
- inyecta dependencias en el `@InjectMocks`
- permite usar Mockito sin inicializar manualmente `MockitoAnnotations.openMocks(this)`

##### Por qué es útil

- Reemplaza al antiguo `@RunWith` de JUnit 4.
- Hace más limpio el setup de pruebas.
- Permite usar extensiones estándar o personalizadas.
- Es compatible con múltiples extensiones:
  - `@ExtendWith(MockitoExtension.class)`
  - `@ExtendWith(SpringExtension.class)`
  - `@ExtendWith(MyCustomExtension.class)`

##### Cuando usarlo

- cuando necesitas inicializar mocks en JUnit 5
- cuando quieres aplicar configuración global de tests
- cuando quieres agregar comportamiento transversal a tus pruebas

**Nota**: En JUnit 5, `@ExtendWith` no es exclusivo de Mockito: es el mecanismo genérico de extensión de todo el framework.

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

Mockito es la librería estándar para crear dobles de prueba en Java. En pruebas unitarias, nos permite aislar la lógica de negocio del `AppointmentService` sin levantar Spring ni conectar a la base de datos.

- `@ExtendWith(MockitoExtension.class)`: Inicializa Mockito en JUnit 5 y activa las anotaciones de creación de mocks.
- `@Mock`: Crea una simulación controlada de una dependencia.
- `@InjectMocks`: Instancia la clase bajo prueba (SUT) e inyecta los mocks en sus dependencias.
- `@Spy`: Crea un objeto real que puede ser parcial o completamente verificado; se usa cuando queremos conservar comportamiento real salvo alguna llamada.
- `@MockBean`: se usa en pruebas de integración con Spring Boot, no en pruebas unitarias puras.

#### Mock vs Stub

- Un _mock_ es el objeto simulado completo que registra interacciones.
- Un _stub_ es el comportamiento que programamos en ese mock con `when(...).thenReturn(...)` o `thenThrow(...)`.

| Concepto                        | Qué hace                                                  | Cuándo usarlo                                                                      |
| ------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `@Mock`                         | Crea un objeto Mockito simulado                           | Cuando la clase bajo prueba depende de un repositorio, servicio externo, API o DAO |
| `@InjectMocks`                  | Inyecta mocks en la clase bajo prueba                     | Para crear automáticamente el SUT con sus dependencias simuladas                   |
| `@Spy`                          | Crea un objeto real con capacidad de verificación parcial | Cuando necesitamos usar comportamiento real y al mismo tiempo verificar llamadas   |
| `when(...)` / `thenReturn(...)` | Define la respuesta esperada de un mock                   | Para simular datos de repositorio, respuestas de API o condiciones de error        |
| `verify(...)`                   | Comprueba que un método fue invocado                      | Para asegurar el comportamiento y evitar efectos secundarios inesperados           |

#### Configuración típica

```java
@ExtendWith(MockitoExtension.class)
class AppointmentServiceTest {

    @Mock
    private AppointmentRepository repository;

    @InjectMocks
    private AppointmentService service;

    private Appointment appointmentData;

    @BeforeEach
    void init() {
        appointmentData = new Appointment(10L, LocalDateTime.now().plusDays(2), "Ana Smith");
    }

    @Test
    @DisplayName("Reservar cita cuando el slot está vacío")
    void shouldCompleteBookingProcess() {
        when(repository.findByDateTime(any())).thenReturn(Optional.empty());
        when(repository.save(any())).thenReturn(appointmentData);

        Appointment saved = service.bookAppointment(appointmentData);

        assertAll(
            () -> assertNotNull(saved),
            () -> assertEquals("Ana Smith", saved.patientName()),
            () -> verify(repository, times(1)).save(any())
        );
    }
}
```

#### Inicialización manual en `@BeforeEach`

Si prefieres no usar `@InjectMocks`, puedes inicializar los mocks manualmente. Esto hace explícita la creación del SUT y ayuda a comprender mejor la inyección de dependencias en pruebas unitarias.

```java
class AppointmentServiceTest {

    private AppointmentRepository repository;
    private AppointmentService service;
    private AutoCloseable closeable;
    private Appointment appointmentData;

    @BeforeEach
    void init() {
        repository = mock(AppointmentRepository.class);
        service = new AppointmentService(repository);
        appointmentData = new Appointment(10L, LocalDateTime.now().plusDays(2), "Ana Smith");
    }

    @AfterEach
    void cleanup() throws Exception {
        if (closeable != null) {
            closeable.close();
        }
    }

    @Test
    void shouldCompleteBookingProcess() {
        when(repository.findByDateTime(any())).thenReturn(Optional.empty());
        when(repository.save(any())).thenReturn(appointmentData);

        Appointment saved = service.bookAppointment(appointmentData);

        assertAll(
            () -> assertNotNull(saved),
            () -> assertEquals("Ana Smith", saved.patientName()),
            () -> verify(repository, times(1)).save(any())
        );
    }
}
```

Otra forma de inicializar los mocks es con `MockitoAnnotations.openMocks(this)` en `@BeforeEach`, especialmente si deseas mantener las anotaciones `@Mock` sin usar `MockitoExtension`.

```java
class AppointmentServiceTest {

    @Mock
    private AppointmentRepository repository;
    private AppointmentService service;

    @BeforeEach
    void init() {
        MockitoAnnotations.openMocks(this);
        service = new AppointmentService(repository);
    }
}
```

#### Uso de matchers

Mockito ofrece una variedad de matchers para definir condiciones flexibles en los stubs y verificaciones:

- `any()`: acepta cualquier valor compatible.
- `anyLong()`, `anyString()`, `any(Appointment.class)`: versiones específicas por tipo.
- `eq(value)`: compara con un valor exacto.
- `argThat(predicate)`: para validaciones más complejas sobre el argumento.

> Importante: no mezcles valores literales y matchers en una misma invocación; si usas matchers en un parámetro, usa matchers en todos los parámetros de ese método.

#### Stubbing práctico

- `when(repository.findById(anyLong())).thenThrow(new EntityNotFoundException("No encontrado"));`
  - simula que la dependencia lanza una excepción.
- `when(externalApi.checkStatus()).thenReturn(Status.PENDING).thenReturn(Status.COMPLETED);`
  - simula respuestas consecutivas, útil para reintentos.
- `when(repository.save(any(Appointment.class))).thenAnswer(invocation -> invocation.getArgument(0));`
  - devuelve dinámicamente el objeto recibido.

#### Verificación de comportamiento

- `verify(repository, times(1)).save(any());`
  - asegura que se guardó exactamente una vez.
- `verify(repository, never()).save(any());`
  - garantiza que no se realizó ninguna persistencia en flujos fallidos.
- `verify(notificationService, atLeastOnce()).send(any());`
  - valida que la notificación se envió al menos una vez.
- `verify(logger, atMost(3)).info(anyString());`
  - evita que un código loguee más de lo esperado.
- `verify(repository, only()).save(any());`
  - comprueba que el único método llamado en el mock fue `save()`.
- `verifyNoMoreInteractions(repository);`
  - asegura que no hubo otras llamadas no previstas.

#### Orden de llamadas

```java
InOrder inOrder = inOrder(repository, notificationService);
inOrder.verify(repository).save(any());
inOrder.verify(notificationService).sendEmail(any());
```

#### Mejores prácticas con Mockito

- No mockees la clase que estás probando; mockea solo sus dependencias.
- Usa Mockito para aislar dependencias externas o costosas, no para probar lógica interna.
- Evita lógica compleja en los tests: el test debe ser Arrange, Act, Assert.
- Prefiere nombres descriptivos para los tests y usa `@DisplayName` para mejorar los reportes.
- Cuando tu prueba necesita el contexto de Spring, usa pruebas de integración específicas; en unitarias, mantén `MockitoExtension` y POJOs.

#### Ejemplo de flujo negativo

```java
when(repository.findByDateTime(appointmentData.dateTime()))
    .thenReturn(Optional.of(new Appointment(1L, appointmentData.dateTime(), "Paciente Existente")));

AppointmentConflictException ex = assertThrows(AppointmentConflictException.class,
    () -> service.bookAppointment(appointmentData));

assertTrue(ex.getMessage().contains("horario ya está reservado"));
verify(repository, never()).save(any());
```

### D. Los patrones AAA (Arrange, Act, Assert) y GWT (Given-When-Then)

#### Patrón AAA (Arrange-Act-Assert)

Para que una prueba sea legible y profesional, debe seguir una estructura clara que incluso un no-programador pueda seguir:

1. **Arrange (Preparar)**: Configuramos los Mocks y los datos de entrada.
2. **Act (Actuar)**: Invocamos el método específico que estamos probando.
3. **Assert (Verificar)**: Validamos que el resultado y el comportamiento de los mocks coincidan con lo esperado.

**Ejemplo de AAA en una prueba unitaria:**

```java
@Test
void shouldBookAppointmentWhenSlotIsAvailable() {
    // Arrange: Preparar datos y mocks
    when(repository.findByDateTime(any())).thenReturn(Optional.empty());
    when(repository.save(any())).thenReturn(appointmentData);

    // Act: Ejecutar la acción bajo prueba
    Appointment saved = service.bookAppointment(appointmentData);

    // Assert: Verificar resultados y comportamiento
    assertAll(
        () -> assertNotNull(saved),
        () -> assertEquals("Ana Smith", saved.patientName()),
        () -> verify(repository, times(1)).save(any())
    );
}
```

#### Patrón GWT (Given-When-Then) - Alternativa BDD

El patrón **Given-When-Then** (GWT) es una variante del AAA enfocada en **Behavior-Driven Development (BDD)**. Se usa para escribir pruebas que describan el comportamiento esperado desde la perspectiva del usuario o negocio, facilitando la colaboración entre desarrolladores, testers y stakeholders.

- **Given (Dado)**: Establece el contexto inicial o precondiciones (equivalente a Arrange).
- **When (Cuando)**: Describe la acción o estímulo que ocurre (equivalente a Act).
- **Then (Entonces)**: Especifica el resultado esperado o post-condiciones (equivalente a Assert).

**Ejemplo de GWT en una prueba unitaria:**

```java
@Test
void givenAvailableSlot_whenBookingAppointment_thenShouldPersistSuccessfully() {
    // Given: Dado un slot disponible
    when(repository.findByDateTime(any())).thenReturn(Optional.empty());
    when(repository.save(any())).thenReturn(appointmentData);

    // When: Cuando se intenta reservar la cita
    Appointment saved = service.bookAppointment(appointmentData);

    // Then: Entonces la cita debe persistirse exitosamente
    assertAll(
        () -> assertNotNull(saved),
        () -> assertEquals("Ana Smith", saved.patientName()),
        () -> verify(repository, times(1)).save(any())
    );
}
```

**Cuándo usar AAA vs GWT:**

- **Usa AAA** cuando escribes pruebas técnicas enfocadas en la implementación (TDD puro).
- **Usa GWT** cuando las pruebas deben ser legibles para no-técnicos o cuando sigues metodologías BDD como Cucumber.
- Ambos patrones son compatibles; elige el que mejor se adapte al contexto del equipo y proyecto.

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

tasks.named('test') {
    finalizedBy jacocoTestReport // El reporte se genera después de los tests
}

def jacocoExcludes = [
    '**/config/**', // Excluye clases de configuración
    '**/dto/**',    // Excluye DTOs (si solo son contenedores de datos)
    '**/Application.class' // Excluye la clase principal de Spring Boot
]

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
    }

    afterEvaluate {
        classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it, exclude: jacocoExcludes)
        }))
    }
}

jacocoTestCoverageVerification {
    dependsOn test

    afterEvaluate {
        classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it, exclude: jacocoExcludes)
        }))
    }

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

bootJar dependsOn jacocoTestReport // Genera el reporte antes de empaquetar la aplicación
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

   ```mermaid
   graph TD
       A["¿Es tu proyecto...?"] --> B["NUEVO desde cero"]
       A --> C["LEGACY / Heredado"]
       A --> D["EQUIPO DISTRIBUIDO"]
       A --> E["EDUCATIVO / Bootcamp"]
       A --> F["STARTUP / MVP rápido"]

       B --> B1{"¿BDD/TDD<br/>formalmente<br/>definida?"}
       B1 -->|SÍ<br/>Scrum/Kanban| B2["✅ GIVEN-WHEN-THEN<br/>o BDD-STYLE"]
       B1 -->|NO| B3["✅ BDD-STYLE<br/>más pragmático"]

       C --> C1{"¿Mejorando<br/>testing?"}
       C1 -->|Incrementalmente| C2["✅ BDD-STYLE<br/>introduce sin disrupción"]
       C1 -->|Completa renovación| C3["✅ GIVEN-WHEN-THEN<br/>reinicio limpio"]
       C1 -->|Solo mantenimiento| C4["✅ AAA-PATTERN<br/>balance pragmático"]

       D --> D1["✅ GIVEN-WHEN-THEN<br/>máxima claridad"]

       E --> E1["✅ DESCRIPTIVO-SIMPLE<br/>enfatiza intención"]

       F --> F1["✅ BDD-STYLE<br/>velocidad + claridad"]

       style B2 fill:#c8e6c9,color:#000
       style B3 fill:#c8e6c9,color:#000
       style C2 fill:#c8e6c9,color:#000
       style C3 fill:#c8e6c9,color:#000
       style C4 fill:#c8e6c9,color:#000
       style D1 fill:#c8e6c9,color:#000
       style E1 fill:#c8e6c9,color:#000
       style F1 fill:#c8e6c9,color:#000
       style A fill:#fff9c4,color:#000
       style B fill:#ffe0b2,color:#000
       style C fill:#ffe0b2,color:#000
       style D fill:#ffe0b2,color:#000
       style E fill:#ffe0b2,color:#000
       style F fill:#ffe0b2,color:#000
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
