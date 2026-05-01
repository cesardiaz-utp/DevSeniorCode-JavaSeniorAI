# Unidad 3 - Clase 3: Pruebas de Integración en Spring Boot con @SpringBootTest

- **Duración**: 2 horas
- **Objetivo**: Dominar la ejecución de pruebas de integración en Spring Boot, validando el comportamiento del sistema completo con contexto de Spring, HTTP layer y persistencia real.

En esta clase aprenderemos a verificar que la aplicación funciona como un todo, no solo como piezas aisladas. Usaremos `@SpringBootTest`, `MockMvc` y patrones de integración para garantizar que los controladores, servicios y la capa de datos colaboran correctamente.

## Parte 1: Teoría (45 min)

### A. ¿Qué es una prueba de integración en Spring Boot?

Una prueba de integración valida la interacción entre varios componentes del sistema. En Spring Boot esto significa cargar el contexto de la aplicación, inyectar beans reales y, en muchos casos, ejecutar solicitudes HTTP a los controladores.

- En una prueba unitaria comprobamos una sola clase aislada.
- En una prueba de integración comprobamos la integración entre varias capas: Web, servicio, persistencia y configuración.
- El objetivo no es probar el framework, sino usarlo como soporte para probar el flujo real de la aplicación.

### B. `@SpringBootTest` y su comportamiento

`@SpringBootTest` carga el contexto completo de Spring Boot. Esto incluye:

- Beans de la aplicación.
- Configuraciones de seguridad, serialización y validaciones.
- Conexiones a bases de datos, dependencias de infraestructura y profile-specific properties.

Es la anotación más cercana a ejecutar la aplicación en un entorno de prueba.

#### Ejemplo básico

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class AppointmentIntegrationTest {
    @Autowired
    private MockMvc mockMvc;

    // tests...
}
```

### C. Comparación con otras anotaciones de Spring Boot Test

| Anotación         | Propósito                                 | Cuándo usarla                                                                      |
| ----------------- | ----------------------------------------- | ---------------------------------------------------------------------------------- |
| `@SpringBootTest` | Carga el contexto completo de Spring Boot | Pruebas de integración que necesitan varias capas y configuración real             |
| `@WebMvcTest`     | Carga solo la capa web                    | Validar controladores, serialización y filtros HTTP sin servicio/persistencia real |
| `@DataJpaTest`    | Carga solo JPA y repositorios             | Validar mapeo de entidades y consultas con base de datos en memoria/real           |
| `@MockBean`       | Reemplaza un bean de Spring con un mock   | Cuando necesita aislar una dependencia externa dentro de una prueba de integración |

### D. La pirámide de pruebas: dónde encaja `@SpringBootTest`

- Unitarias: rápidas, aisladas, `MockitoExtension`, `@Mock`.
- Integración: más lentas, contexto real, `@SpringBootTest`, `Testcontainers`, `MockMvc`.
- E2E: más lentas, interactúan con toda la aplicación y servicios externos.

> `@SpringBootTest` es ideal para cuando quieres confianza en que los componentes están correctamente conectados.

### E. Anotaciones clave de las pruebas de integración

- `@AutoConfigureMockMvc`: Permite usar `MockMvc` sin arrancar un servidor real.

  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  class AppointmentIntegrationTest {
      @Autowired
      private MockMvc mockMvc;

      // tests...
  }
  ```

- `@ActiveProfiles("test")`: Carga propiedades específicas para pruebas.
- `@MockBean`: Sustituye un bean del contexto por un mock controlado.

  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  @ActiveProfiles("test")
  class AppointmentIntegrationTest {
      @Autowired
      private MockMvc mockMvc;

      @MockBean
      private ExternalApiService externalApiService;

      // tests...
  }
  ```

- `@Transactional`: Hace rollback tras cada prueba para mantener datos limpios.

  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  @ActiveProfiles("test")
  @Transactional
  class AppointmentIntegrationTest {
      @Autowired
      private MockMvc mockMvc;

      @Autowired
      private AppointmentRepository appointmentRepository;

      @Test
      void shouldRollbackDatabaseChangesAfterTest() throws Exception {
          // acciones que escriben en la BD
      }
  }
  ```

- `@Sql`: Permite preparar o limpiar la base de datos antes/después de cada prueba.

  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  @ActiveProfiles("test")
  @Sql(scripts = "/sql/clean-appointments.sql", executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
  @Sql(scripts = "/sql/cleanup-appointments.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
  class AppointmentIntegrationTest {
      // tests...
  }
  ```

- `@Testcontainers`: Se usa cuando se quiere ejecutar la DB dentro de un contenedor Docker real.

  ```java
  @Testcontainers
  @SpringBootTest
  @AutoConfigureMockMvc
  @ActiveProfiles("test")
  class AppointmentIntegrationTest {
      @Container
      static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
              .withDatabaseName("testdb")
              .withUsername("test")
              .withPassword("test");

      // tests...
  }
  ```

- `@DynamicPropertySource`: Ajusta propiedades dinámicamente para conectar Testcontainers con Spring.
- `@DataJpaTest`: Carga solo la capa de persistencia, ideal para validar consultas y mapeo de entidades sin cargar el contexto completo.

  ```java
  @DataJpaTest
  @ActiveProfiles("test")
  class AppointmentRepositoryTest {
      @Autowired
      private AppointmentRepository appointmentRepository;

      // tests...
  }
  ```

### F. ¿Por qué no usar `@SpringBootTest` en todas las pruebas?

Aunque es poderoso, tiene un costo:

- Arranque más lento.
- Más dependencia del entorno.
- Pruebas menos deterministas si no se controla la infraestructura.

Para lógica de negocio simple, las pruebas unitarias siguen siendo la mejor opción.

### G. El flujo básico de una prueba de integración con MockMvc

1. Preparar el entorno de datos (`@Sql`, `Testcontainers`, repositorios).
2. Ejecutar una petición HTTP simulada con `MockMvc`.
3. Verificar el estado de la respuesta y el cuerpo JSON.
4. Verificar el estado de la base de datos o el comportamiento del servicio.

#### Ejemplo de flujo

```java
mockMvc.perform(post("/api/appointments")
        .contentType(MediaType.APPLICATION_JSON)
        .content(json))
    .andExpect(status().isCreated())
    .andExpect(jsonPath("$.patientName").value("Ana Smith"));

assertThat(appointmentRepository.count()).isEqualTo(1);
```

### H. Uso de Testcontainers para persistencia real

En Spring Boot 4 es un buen estándar usar bases de datos reales en tests:

- Evita diferencias de comportamiento entre H2 y PostgreSQL/MySQL.
- Detecta problemas de SQL, constraints y dialectos reales.
- Genera confianza más cercana a producción.

#### Configuración típica

```java
@Testcontainers
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class AppointmentIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void datasourceProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;
    @Autowired
    private AppointmentRepository appointmentRepository;

    // tests...
}
```

### I. Buenas prácticas en pruebas de integración

- No mockear lo que se quiere integrar. Mockea solo dependencias externas que no formen parte de la prueba.
- Usa un perfil `test` con propiedades específicas para la base de datos, logging y beans de prueba.
- Mantén las pruebas limpias y deterministas. Evita datos dispersos en múltiples archivos.
- Usa `@Transactional` o `@Sql` para resetear el estado entre tests.
- Valida datos tanto en la respuesta HTTP como en la persistencia.
- Crea solo los tests necesarios para cubrir escenarios de negocio críticos.

## Parte 2: Laboratorio (1h 15m)

### A. Escenario: Sistema de Citas Médicas

Vamos a construir un conjunto de pruebas de integración para el flujo de reservas de cita médica. El sistema debe:

1. Exponer un endpoint HTTP para crear citas.
2. Validar que no existan citas en el mismo horario.
3. Persistir la cita en la base de datos solo si la validación es exitosa.
4. Devolver códigos HTTP adecuados según el resultado.

### B. Componentes del dominio

#### Modelo de datos

```java
@Entity
@Table(name = "appointments")
public class Appointment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime dateTime;
    private String patientName;

    // getters, setters, constructor
}
```

#### Repositorio

```java
public interface AppointmentRepository extends JpaRepository<Appointment, Long> {
    boolean existsByDateTime(LocalDateTime dateTime);
}
```

#### Servicio

```java
@Service
public class AppointmentService {
    private final AppointmentRepository repository;

    public AppointmentService(AppointmentRepository repository) {
        this.repository = repository;
    }

    public Appointment bookAppointment(Appointment appointment) {
        if (repository.existsByDateTime(appointment.getDateTime())) {
            throw new AppointmentConflictException("Horario ya reservado");
        }
        return repository.save(appointment);
    }
}
```

#### Controlador

```java
@RestController
@RequestMapping("/api/appointments")
public class AppointmentController {
    private final AppointmentService service;

    public AppointmentController(AppointmentService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<Appointment> book(@RequestBody @Valid AppointmentRequest request) {
        Appointment appointment = new Appointment(null, request.getDateTime(), request.getPatientName());
        Appointment saved = service.bookAppointment(appointment);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}
```

#### DTO de entrada

```java
public record AppointmentRequest(
        @NotNull LocalDateTime dateTime,
        @NotBlank String patientName
) {}
```

### C. Configuración de pruebas

Usa un perfil de prueba separado para `application-test.yml`.

```yaml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/testdb
    username: test
    password: test
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
```

En la clase de prueba:

- `@SpringBootTest`
- `@AutoConfigureMockMvc`
- `@ActiveProfiles("test")`
- `@Testcontainers` o una configuración persistente de prueba

### D. Ejemplo de test de integración

```java
@Testcontainers
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class AppointmentIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void datasourceProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppointmentRepository appointmentRepository;

    @BeforeEach
    void setUp() {
        appointmentRepository.deleteAll();
    }

    @Test
    @DisplayName("Reservar cita cuando el slot está disponible")
    void shouldCreateAppointmentWhenSlotIsFree() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-10T09:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.patientName").value("Ana Smith"));

        assertThat(appointmentRepository.count()).isEqualTo(1);
    }
}
```

### E. Casos de prueba esenciales

1. **Reserva exitosa**
   - Input válido.
   - `201 Created`.
   - Cita persistida en BD.

2. **Conflicto de horario**
   - Existe una cita previa con el mismo horario.
   - `409 Conflict`.
   - No se guarda la nueva cita.

3. **Validación de payload**
   - Campo `patientName` vacío o `dateTime` nulo.
   - `400 Bad Request`.

4. **Prueba de rollback**
   - Usar `@Transactional` en la clase de prueba.
   - Asegurar que cada test empieza con datos limpios.

### F. Ejemplo de test de conflicto

```java
@Test
@DisplayName("Rechazar reserva si el horario ya está ocupado")
void shouldReturnConflictWhenSlotAlreadyBooked() throws Exception {
    Appointment existing = new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Paciente Existente");
    appointmentRepository.save(existing);

    String payload = "{\"dateTime\": \"2026-05-10T09:00:00\", \"patientName\": \"Ana Smith\"}";

    mockMvc.perform(post("/api/appointments")
            .contentType(MediaType.APPLICATION_JSON)
            .content(payload))
            .andExpect(status().isConflict())
            .andExpect(jsonPath("$.message").value("Horario ya reservado"));

    assertThat(appointmentRepository.count()).isEqualTo(1);
}
```

### G. Ejercicio práctico

Pide a los alumnos que implementen los siguientes tests adicionales:

- Validar un GET `/api/appointments` que devuelva todas las citas.
- Probar un `PUT` para actualizar el horario de una cita existente.
- Verificar un endpoint protegido por seguridad y la respuesta `401 Unauthorized`.

### H. Ejercicios adicionales

- Configurar `Testcontainers` con PostgreSQL y `Redis` si hay caché.
- Comparar el mismo test ejecutado con `H2` y con `PostgreSQL`.
- Añadir cobertura de pruebas para excepciones globales con `@ControllerAdvice`.
- Ajustar un Quality Gate para que falle si el build no ejecuta los tests de integración.

### I. Preguntas para reflexionar

- ¿Por qué `@SpringBootTest` es más caro que una prueba unitaria?
- ¿Cuándo es correcto usar `@MockBean` en un test de integración?
- ¿Qué falla más probable detecta una prueba de integración que una unitaria no puede detectar?
- ¿Cómo se asegura la prueba de que la DB quedó en el estado esperado?

## Cierre

Las pruebas de integración son la red de seguridad que confirma que la aplicación funciona como un bloque cohesivo. No reemplazan las pruebas unitarias; las complementan.

- Unitarias = lógica aislada.
- Integración = colaboración entre capas.
- E2E = experiencia completa del usuario.

Sigue la regla: usa `@SpringBootTest` cuando el comportamiento que debes validar dependa de Spring y de la integración entre beans.

Para ejecutar la suite de integración, usa:

```bash
mvn clean verify
```

o

```bash
./gradlew test
```
