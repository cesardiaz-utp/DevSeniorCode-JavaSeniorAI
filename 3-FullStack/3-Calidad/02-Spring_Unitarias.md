# Unidad 3 - Clase 2: Pruebas Unitarias en Spring Boot con JUnit 5 y Mockito

- **Duración**: 2 horas
- **Objetivo**: El objetivo fundamental de esta sesión es dominar el **aislamiento de lógica de negocio** mediante una mentalidad de Arquitecto de Software. En el desarrollo de sistemas empresariales, la deuda técnica suele acumularse por la falta de una red de seguridad confiable.

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
- `@DisplayName`: Un arquitecto escribe código para humanos. Esta anotación permite que el reporte de Jenkins o GitHub Actions muestre "Debería rechazar citas si el médico está de vacaciones" en lugar de `test_err_01()`.

  ```java
  @Test
  @DisplayName("Debería fallar si el paciente es menor de edad")
  void testMinorPatient() { ... }
  ```

### B. El Arsenal de Aserciones de JUnit 5

Las aserciones son los predicados que determinan si un test pasa o falla. JUnit 5 ofrece un catálogo extenso para cubrir diversas necesidades semánticas:

- `assertEquals(esperado, actual)` / `assertNotEquals(...)`: La base de cualquier test. Compara valores primitivos u objetos usando el método `.equals()`.
- `assertTrue(condicion)` / `assertFalse(condicion)`: Valida estados booleanos o predicados lógicos.
- `assertNull(objeto)` / `assertNotNull(objeto)`: Crítico para validar que servicios o repositorios no retornen nulidad inesperada, o para asegurar que un campo se limpió correctamente.
- `assertSame(obj1, obj2)`: A diferencia de `assertEquals`, esta valida la identidad referencial (que ambos apunten a la misma posición de memoria).
- `assertIterableEquals(lista1, lista2)`: Compara el contenido y el orden de dos colecciones, asegurando que la secuencia de datos sea idéntica.
- `assertAll("Etiqueta", ...lambdas)`: Conocida como _Grouped Assertions_. Permite ejecutar múltiples aserciones incluso si las primeras fallan. Es ideal para validar todos los campos de un objeto de una sola vez sin que el test se detenga en el primer error.
- `assertThrows(Excepcion.class, () -> codigo)`: Captura la excepción lanzada. Esto nos permite realizar aserciones adicionales sobre el contenido del error, como códigos de error internos o mensajes personalizados de validación.
- `assertTimeout(Duration, () -> codigo)`: Valida que un algoritmo cumpla con un SLA (Service Level Agreement) de tiempo máximo de ejecución.

### C. Aislamiento Estratégico con Mockito

Mockito nos permite aplicar el principio de Inversión de Control en nuestras pruebas. Si el `AppointmentService` depende de un `Repository`, el "Contrato" es lo que importa, no la implementación.

| Concepto | Profundización Técnica | Rol en el Ecosistema |
| --- | --- | --- |
| `@Mock` | Crea un objeto Proxy que intercepta todas las llamadas. | Sustituye componentes pesados o externos (DB, Colas, APIs). |
| `@InjectMocks` | Realiza una inyección de dependencias basada en tipos. | Automatiza la creación del Sujeto Bajo Prueba (SUT). |
| **Stubbing (`when`)** | Programación del comportamiento esperado (Entrada -> Salida). | Define los escenarios (Éxito, Error, Datos vacíos). |
| **Verification (`verify`)** | Auditoría de comportamiento posterior a la ejecución. | Asegura que las reglas de negocio se ejecuten (ej: ¿Se envió el mail?). |

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

### E. Justificación Arquitectónica del Aislamiento

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

    // Inyección por constructor: La mejor práctica para facilitar el testeo
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
