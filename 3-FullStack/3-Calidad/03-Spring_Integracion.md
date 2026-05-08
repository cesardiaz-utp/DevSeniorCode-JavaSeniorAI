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

#### Matchers útiles de `MockMvc`

Los matchers de `MockMvc` permiten verificar el código de respuesta, el contenido, los headers y el cuerpo JSON con precisión.

- `status().isOk()`, `status().isCreated()`, `status().isBadRequest()`, `status().isConflict()`:
  - Validan el código HTTP devuelto por el endpoint.
  - Siempre deben acompañar otras verificaciones para asegurar que la respuesta es la esperada.

- `content().contentType(MediaType.APPLICATION_JSON)`:
  - Verifica que el `Content-Type` sea JSON.
  - Es útil cuando la API debe devolver datos estructurados y no texto plano.

- `content().json("{...}")`:
  - Compara el JSON completo de la respuesta.
  - Ideal cuando se conoce exactamente el payload esperado.
  - Ejemplo: `content().json("{\"patientName\":\"Ana Smith\"}")`.

- `content().string("...")`:
  - Verifica respuestas de texto plano.
  - Útil para endpoints que devuelven mensajes simples o HTML.

- `header().string("Location", "/api/appointments/1")`:
  - Verifica valores de encabezados HTTP.
  - Muy útil para respuestas `201 Created` con `Location` o para verificar `Cache-Control`, `Authorization`, etc.

- `jsonPath("$.patientName").value("Ana Smith")`:
  - Extrae y valida una propiedad específica del JSON.
  - Es la forma más común de verificar respuestas parciales.

- `jsonPath("$.id").exists()`:
  - Verifica que el campo existe en la respuesta.

- `jsonPath("$.items", hasSize(3))`:
  - Valida el tamaño de un array dentro del JSON.
  - Requiere importar `org.hamcrest.Matchers.hasSize`.

- `jsonPath("$.doctor.name").value("Dr. García")`:
  - Sirve para validar objetos anidados.

#### Sintaxis de `jsonPath()`

La expresión de `jsonPath()` sigue un formato similar a JSONPath estándar:

- `$` representa la raíz del JSON.
- `$.campo` selecciona un campo directo en el objeto raíz.
- `$.objeto.campo` selecciona un campo dentro de un objeto anidado.
- `$.items[0]` selecciona el primer elemento de un array.
- `$.items[1].name` selecciona el campo `name` del segundo elemento del array.
- `$.items[*].id` selecciona todos los `id` dentro del array `items`.
- `$.items[?(@.completed == true)]` busca elementos con condición (requiere soporte de JSONPath del parser integrado).

Ejemplos prácticos:

- `jsonPath("$.patientName")` → selecciona `patientName` en raíz.
- `jsonPath("$.doctor.name")` → selecciona `name` dentro de `doctor`.
- `jsonPath("$.appointments[0].dateTime")` → selecciona la fecha del primer appointment.
- `jsonPath("$.items", hasSize(3))` → valida que el array tenga exactamente 3 elementos.

```java
mockMvc.perform(post("/api/appointments")
        .contentType(MediaType.APPLICATION_JSON)
        .content(json))
        .andExpect(status().isCreated())
        .andExpect(content().contentType(MediaType.APPLICATION_JSON))
        .andExpect(header().string("Location", containsString("/api/appointments/")))
        .andExpect(jsonPath("$.patientName").value("Ana Smith"))
        .andExpect(jsonPath("$.dateTime").exists());
```

#### Buenas prácticas con matchers

- Usa `status()` siempre, incluso si también verificas el body.
- Usa `content().json(...)` para comparar payloads completos y `jsonPath(...)` para validar campos concretos.
- Usa `header()` para validar metadatos HTTP importantes.
- No confíes solo en `status()`; valida también el contenido cuando la lógica depende de él.
- Prefiere `jsonPath` cuando la respuesta JSON es grande o anidada.

#### Capturar la respuesta con `.andReturn()` para hacer assertions avanzados

En ocasiones necesitas acceder a la respuesta completa para hacer assertions más complejas. Puedes usar `.andReturn()` para obtener el objeto `MvcResult`:

```java
// Capturar la respuesta completa
MvcResult result = mockMvc.perform(post("/api/appointments")
        .contentType(MediaType.APPLICATION_JSON)
        .content(payload))
    .andExpect(status().isCreated())
    .andReturn();

// Obtener el contenido como String
String responseBody = result.getResponse().getContentAsString();

// Obtener headers
String location = result.getResponse().getHeader("Location");

// Parsear a objeto con ObjectMapper (necesita Jackson)
ObjectMapper mapper = new ObjectMapper();
Appointment appointment = mapper.readValue(responseBody, Appointment.class);

// Hacer assertions con AssertJ
assertThat(appointment.getPatientName()).isEqualTo("Ana Smith");
assertThat(appointment.getId()).isNotNull();
assertThat(location).contains("/api/appointments/");
```

**Casos de uso para `.andReturn()`:**

1. **Extraer ID de recurso creado** para usarlo en tests posteriores:

   ```java
   @Test
   void shouldCreateAndThenUpdateAppointment() throws Exception {
       // Crear appointment
       String createPayload = "{\"dateTime\": \"2026-05-10T09:00:00\", \"patientName\": \"Ana Smith\"}";
       MvcResult createResult = mockMvc.perform(post("/api/appointments")
               .contentType(MediaType.APPLICATION_JSON)
               .content(createPayload))
           .andExpect(status().isCreated())
           .andReturn();

       // Parsear respuesta para obtener ID
       ObjectMapper mapper = new ObjectMapper();
       Appointment created = mapper.readValue(createResult.getResponse().getContentAsString(), Appointment.class);
       Long appointmentId = created.getId();

       // Usar el ID para actualizar
       String updatePayload = "{\"dateTime\": \"2026-05-10T11:00:00\", \"patientName\": \"Ana García\"}";
       mockMvc.perform(put("/api/appointments/{id}", appointmentId)
               .contentType(MediaType.APPLICATION_JSON)
               .content(updatePayload))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.patientName").value("Ana García"));
   }
   ```

2. **Validar arrays complejos en respuesta**:

   ```java
   @Test
   void shouldValidateAppointmentListStructure() throws Exception {
       // Preparar datos
       appointmentRepository.save(new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith"));
       appointmentRepository.save(new Appointment(null, LocalDateTime.parse("2026-05-10T10:00:00"), "Juan Pérez"));

       // Obtener respuesta
       MvcResult result = mockMvc.perform(get("/api/appointments"))
           .andExpect(status().isOk())
           .andReturn();

       // Parsear como lista
       ObjectMapper mapper = new ObjectMapper();
       List<Appointment> appointments = mapper.readValue(
           result.getResponse().getContentAsString(),
           new TypeReference<List<Appointment>>() {}
       );

       // Assertions complejas
       assertThat(appointments).hasSize(2);
       assertThat(appointments).extracting("patientName")
           .containsExactly("Ana Smith", "Juan Pérez");
       assertThat(appointments).allMatch(app -> app.getId() != null);
   }
   ```

3. **Verificar headers de respuesta**:

   ```java
   @Test
   void shouldIncludeLocationHeader() throws Exception {
       String payload = "{\"dateTime\": \"2026-05-10T09:00:00\", \"patientName\": \"Ana Smith\"}";

       MvcResult result = mockMvc.perform(post("/api/appointments")
               .contentType(MediaType.APPLICATION_JSON)
               .content(payload))
           .andExpect(status().isCreated())
           .andReturn();

       String location = result.getResponse().getHeader("Location");
       assertThat(location).isNotNull();
       assertThat(location).startsWith("/api/appointments/");

       // Extraer ID del header
       Long appointmentId = Long.parseLong(location.substring(location.lastIndexOf("/") + 1));
       assertThat(appointmentId).isGreaterThan(0);
   }
   ```

**Configuración necesaria:**

Asegúrate de tener Jackson en tus dependencias (`pom.xml` o `build.gradle`):

```xml
<!-- Maven -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

```gradle
// Gradle
implementation 'com.fasterxml.jackson.core:jackson-databind'
```

**Combinación de `andExpect()` y `andReturn()`:**

Ambos se pueden usar juntos en la misma cadena para validar al mismo tiempo que capturas la respuesta:

```java
MvcResult result = mockMvc.perform(post("/api/appointments")
        .contentType(MediaType.APPLICATION_JSON)
        .content(payload))
    .andExpect(status().isCreated())           // Validar estado
    .andExpect(jsonPath("$.patientName").value("Ana Smith"))  // Validar campo
    .andExpect(jsonPath("$.id").exists())      // Validar existencia
    .andReturn();                              // Capturar resultado

// Luego hacer más validaciones complejas
String responseBody = result.getResponse().getContentAsString();
ObjectMapper mapper = new ObjectMapper();
Appointment appointment = mapper.readValue(responseBody, Appointment.class);

// Assertions adicionales con AssertJ
assertThat(appointment.getDateTime())
    .isEqualTo(LocalDateTime.parse("2026-05-10T09:00:00"));
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

### J. Testing con Spring Security

Cuando la aplicación usa Spring Security, los tests de integración deben validar que los endpoints respeten las restricciones de autenticación y autorización. Spring Test proporciona anotaciones y helpers para simular usuarios autenticados sin necesidad de un servidor real.

#### Anotaciones principales para mockear usuarios autenticados

**`@WithMockUser`** - La forma más simple de simular un usuario autenticado:

```java
@Test
@WithMockUser(username = "patient@example.com", roles = {"PATIENT"})
void shouldAllowPatientToViewAppointments() throws Exception {
    mockMvc.perform(get("/api/appointments"))
        .andExpect(status().isOk());
}

@Test
@WithMockUser(username = "admin@hospital.com", roles = {"ADMIN"})
void shouldAllowAdminToViewAllAppointments() throws Exception {
    mockMvc.perform(get("/api/appointments/all"))
        .andExpect(status().isOk());
}
```

**`@WithUserDetails`** - Carga un usuario completo desde `UserDetailsService`:

```java
@Test
@WithUserDetails("patient@example.com")
void shouldLoadFullUserDetailsFromDatabase() throws Exception {
    mockMvc.perform(get("/api/appointments"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.patientEmail").value("patient@example.com"));
}
```

**`@WithSecurityContext`** - Para escenarios complejos con contexto personalizado:

```java
// Crear una anotación personalizada
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = AdminSecurityContextFactory.class)
public @interface WithAppointmentAdmin {}

// Factory que crea el contexto
public class AdminSecurityContextFactory implements WithSecurityContextFactory<WithAppointmentAdmin> {
    @Override
    public SecurityContext createSecurityContext(WithAppointmentAdmin annotation) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        UserDetails admin = User.builder()
            .username("admin@hospital.com")
            .password("admin123")
            .roles("ADMIN")
            .build();

        Authentication auth = new UsernamePasswordAuthenticationToken(
            admin, null, admin.getAuthorities());

        context.setAuthentication(auth);
        return context;
    }
}

// Uso
@Test
@WithAppointmentAdmin
void adminShouldManageAllAppointments() throws Exception {
    mockMvc.perform(delete("/api/appointments/1"))
        .andExpect(status().isNoContent());
}
```

#### Testing de acceso basado en roles

Verifica que cada rol tiene acceso correcto a sus endpoints y se le deniega acceso a otros:

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
class AppointmentRoleBasedSecurityTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppointmentRepository appointmentRepository;

    // PATIENT ROLE TESTS
    @Test
    @WithMockUser(roles = {"PATIENT"})
    @DisplayName("Paciente puede crear una cita")
    void patientCanCreateAppointment() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isCreated());
    }

    @Test
    @WithMockUser(roles = {"PATIENT"})
    @DisplayName("Paciente no puede acceder a panel administrativo")
    void patientCannotAccessAdminPanel() throws Exception {
        mockMvc.perform(get("/api/appointments/admin/report"))
            .andExpect(status().isForbidden()); // 403 Forbidden
    }

    // DOCTOR ROLE TESTS
    @Test
    @WithMockUser(roles = {"DOCTOR"})
    @DisplayName("Doctor puede ver su agenda")
    void doctorCanViewOwnSchedule() throws Exception {
        mockMvc.perform(get("/api/appointments/my-schedule"))
            .andExpect(status().isOk());
    }

    @Test
    @WithMockUser(roles = {"DOCTOR"})
    @DisplayName("Doctor no puede crear citas en el sistema")
    void doctorCannotCreateAppointments() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isForbidden());
    }

    // ADMIN ROLE TESTS
    @Test
    @WithMockUser(roles = {"ADMIN"})
    @DisplayName("Admin puede ver todas las citas")
    void adminCanViewAllAppointments() throws Exception {
        mockMvc.perform(get("/api/appointments/all"))
            .andExpect(status().isOk());
    }

    @Test
    @WithMockUser(roles = {"ADMIN"})
    @DisplayName("Admin puede eliminar cualquier cita")
    void adminCanDeleteAnyAppointment() throws Exception {
        Appointment appointment = appointmentRepository.save(
            new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
        );

        mockMvc.perform(delete("/api/appointments/{id}", appointment.getId()))
            .andExpect(status().isNoContent());
    }
}
```

#### Comparación de anotaciones de autenticación

| Anotación              | Uso                               | Ventajas                     | Desventajas                 |
| ---------------------- | --------------------------------- | ---------------------------- | --------------------------- |
| `@WithMockUser`        | Usuario simple con roles          | Sintaxis simple, rápido      | No valida usuario en DB     |
| `@WithUserDetails`     | Cargar usuario real               | Valida datos completos de BD | Requiere UserDetailsService |
| `@WithSecurityContext` | Contexto personalizado            | Máximo control, flexible     | Más código de setup         |
| Programático           | Directo con SecurityContextHolder | Para lógica compleja         | Requiere limpieza manual    |

#### Testing de endpoints sin autenticación vs con autenticación

Distingue entre endpoints públicos y protegidos:

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class AppointmentAuthenticationTest {

    @Autowired
    private MockMvc mockMvc;

    // ENDPOINTS PÚBLICOS (sin autenticación)
    @Test
    @DisplayName("Endpoint público es accesible sin autenticación")
    void publicHealthCheckDoesNotRequireAuth() throws Exception {
        mockMvc.perform(get("/api/health"))
            .andExpect(status().isOk());
    }

    @Test
    @DisplayName("Registro es público sin requieren autenticación")
    void publicRegistrationDoesNotRequireAuth() throws Exception {
        String payload = "{\"email\": \"newuser@example.com\", \"password\": \"SecurePass123\"}";

        mockMvc.perform(post("/api/auth/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isCreated());
    }

    // ENDPOINTS PROTEGIDOS (requieren autenticación)
    @Test
    @DisplayName("Endpoint protegido retorna 401 sin autenticación")
    void protectedAppointmentListRequiresAuth() throws Exception {
        mockMvc.perform(get("/api/appointments"))
            .andExpect(status().isUnauthorized()); // 401
    }

    @Test
    @DisplayName("POST protegido retorna 401 sin autenticación")
    void protectedCreateAppointmentRequiresAuth() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isUnauthorized());
    }

    // MISMO ENDPOINT: con y sin autenticación
    @Test
    @DisplayName("Mismo endpoint rechaza sin auth, acepta con auth")
    void appointmentEndpointBehavesCorrectly() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        // Sin autenticación: 401
        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isUnauthorized());

        // Con autenticación: 201 Created
        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload)
                .with(SecurityMockMvcRequestPostProcessors.httpBasic("patient@example.com", "password")))
            .andExpect(status().isCreated());
    }
}
```

#### Patrones de autenticación en MockMvc

```java
// 1. HTTP Basic
mockMvc.perform(get("/api/appointments")
    .with(httpBasic("user@example.com", "password")))
    .andExpect(status().isOk());

// 2. Bearer Token (JWT)
String token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
mockMvc.perform(get("/api/appointments")
    .header("Authorization", "Bearer " + token))
    .andExpect(status().isOk());

// 3. API Key en header personalizado
mockMvc.perform(get("/api/appointments")
    .header("X-API-Key", "secret-key-123"))
    .andExpect(status().isOk());

// 4. Cookie de sesión
mockMvc.perform(get("/api/appointments")
    .cookie(new Cookie("JSESSIONID", "session-id-123")))
    .andExpect(status().isOk());
```

#### Testing con JWT (JSON Web Tokens)

Cuando la aplicación usa JWT para autenticación (por ejemplo, un `Authorization: Bearer <token>`), hay varias estrategias para probar endpoints protegidos:

- Obtener un token real mediante el endpoint de login y reutilizarlo en requests.
- Generar un JWT en el propio test usando una librería (ej. JJWT) con la clave de prueba.
- Mockear el `JwtDecoder`/verificador cuando se usa `oauth2ResourceServer` para aceptar tokens de test.

Ejemplos prácticos:

1. Obtener token vía login y reutilizarlo

   ```java
   // 1. Login y extraer token
   MvcResult loginResult = mockMvc.perform(post("/api/auth/login")
           .contentType(MediaType.APPLICATION_JSON)
           .content("{\"email\": \"patient@example.com\", \"password\": \"password123\"}"))
       .andExpect(status().isOk())
       .andReturn();

   String loginBody = loginResult.getResponse().getContentAsString();
   ObjectMapper mapper = new ObjectMapper();
   JsonNode loginJson = mapper.readTree(loginBody);
   String token = loginJson.get("token").asText();

   // 2. Usar token en requests subsecuentes
   mockMvc.perform(get("/api/appointments")
           .header("Authorization", "Bearer " + token))
       .andExpect(status().isOk());
   ```

2. Generar un JWT programáticamente dentro del test (JJWT)

   ```java
   // Dependencia de ejemplo: io.jsonwebtoken:jjwt-api / jjwt-impl / jjwt-jackson
   Key key = Keys.hmacShaKeyFor("test-key-which-is-long-enough-for-hs256".getBytes(StandardCharsets.UTF_8));
   String jwt = Jwts.builder()
       .setSubject("patient@example.com")
       .claim("roles", List.of("ROLE_PATIENT"))
       .setIssuedAt(new Date())
       .setExpiration(Date.from(Instant.now().plus(1, ChronoUnit.HOURS)))
       .signWith(key, SignatureAlgorithm.HS256)
       .compact();

   mockMvc.perform(get("/api/appointments")
           .header("Authorization", "Bearer " + jwt))
       .andExpect(status().isOk());
   ```

3. Mockear `JwtDecoder` en aplicaciones que usan `oauth2ResourceServer`

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class JwtDecoderMockTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private JwtDecoder jwtDecoder; // Bean usado por Spring Security

    @Test
    void protectedEndpointWithMockedJwtDecoder() throws Exception {
        Map<String, Object> headers = Map.of("alg", "none");
        Map<String, Object> claims = Map.of("sub", "patient@example.com", "scope", "openid", "roles", List.of("ROLE_PATIENT"));
        Jwt jwt = new Jwt("token-value", Instant.now(), Instant.now().plus(1, ChronoUnit.HOURS), headers, claims);

        when(jwtDecoder.decode(anyString())).thenReturn(jwt);

        mockMvc.perform(get("/api/appointments")
                .header("Authorization", "Bearer token-value"))
            .andExpect(status().isOk());
    }
}
```

Notas:

- Añade `spring-security-test` en el classpath de test para helpers y `SecurityMockMvcRequestPostProcessors`.
- Si tu app genera tokens firmados con una clave/secret, reutiliza una clave de prueba en `application-test.yml` o genera el token con la misma clave en el test.
- Mockear `JwtDecoder` es útil para evitar dependencia de infraestructura externa y para controlar claims en tests.

#### Buenas prácticas para testing de seguridad

1. **Limpiar contexto de seguridad entre tests**:

   ```java
   @BeforeEach
   void clearSecurityContext() {
       SecurityContextHolder.clearContext();
   }
   ```

2. **Testear casos positivos Y negativos**:
   - ✅ Usuario con rol correcto → 200 OK
   - ❌ Usuario sin rol → 403 Forbidden
   - ❌ Sin autenticación → 401 Unauthorized

3. **Validar que la autenticación se propaga al servicio**:

   ```java
   @Test
   @WithMockUser(username = "patient@example.com")
   void shouldPropagateAuthenticationThroughLayers() throws Exception {
       mockMvc.perform(get("/api/appointments/my-appointments"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$[0].patientEmail").value("patient@example.com"));
   }
   ```

4. **No mockear lo que quieres testear de seguridad**:
   - ✅ Carga el SecurityContext real
   - ❌ No hagas mock de UserDetailsService si quieres testear autenticación

5. **Usa `@Transactional` para aislamiento**:

   ```java
   @SpringBootTest
   @AutoConfigureMockMvc
   @Transactional  // Rollback tras cada test
   class SecurityTest {
   }
   ```

#### Verificación de dependencia

Asegúrate de tener en el `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

O en `build.gradle`:

```gradle
testImplementation 'org.springframework.security:spring-security-test'
```

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

### F. Ejemplos completos de operaciones CRUD

Para completar el flujo CRUD, aquí están los tests para las operaciones restantes:

#### Test de GET - Obtener todas las citas

```java
@Test
@DisplayName("Obtener todas las citas")
void shouldGetAllAppointments() throws Exception {
    // Preparar datos de prueba
    Appointment appointment1 = new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith");
    Appointment appointment2 = new Appointment(null, LocalDateTime.parse("2026-05-10T10:00:00"), "Juan Pérez");
    appointmentRepository.save(appointment1);
    appointmentRepository.save(appointment2);

    // Ejecutar GET request
    mockMvc.perform(get("/api/appointments"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$", hasSize(2))) // Array con 2 elementos
            .andExpect(jsonPath("$[0].patientName").value("Ana Smith"))
            .andExpect(jsonPath("$[1].patientName").value("Juan Pérez"))
            .andExpect(jsonPath("$[0].id").exists())
            .andExpect(jsonPath("$[1].id").exists());
}
```

#### Test de GET - Obtener cita específica

```java
@Test
@DisplayName("Obtener una cita específica por ID")
void shouldGetAppointmentById() throws Exception {
    // Preparar dato de prueba
    Appointment saved = appointmentRepository.save(
        new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
    );

    // Ejecutar GET request con ID
    mockMvc.perform(get("/api/appointments/{id}", saved.getId()))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(saved.getId()))
            .andExpect(jsonPath("$.patientName").value("Ana Smith"))
            .andExpect(jsonPath("$.dateTime").exists());
}

@Test
@DisplayName("Devolver 404 cuando la cita no existe")
void shouldReturnNotFoundWhenAppointmentDoesNotExist() throws Exception {
    mockMvc.perform(get("/api/appointments/{id}", 999L))
            .andExpect(status().isNotFound());
}
```

#### Test de PUT - Actualizar cita

```java
@Test
@DisplayName("Actualizar una cita existente")
void shouldUpdateAppointment() throws Exception {
    // Preparar dato de prueba
    Appointment saved = appointmentRepository.save(
        new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
    );

    // Payload de actualización
    String updatePayload = "{\"dateTime\": \"2026-05-10T11:00:00\", \"patientName\": \"Ana García\"}";

    // Ejecutar PUT request
    mockMvc.perform(put("/api/appointments/{id}", saved.getId())
            .contentType(MediaType.APPLICATION_JSON)
            .content(updatePayload))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(saved.getId()))
            .andExpect(jsonPath("$.patientName").value("Ana García"))
            .andExpect(jsonPath("$.dateTime").value("2026-05-10T11:00:00"));

    // Verificar que se actualizó en la base de datos
    Appointment updated = appointmentRepository.findById(saved.getId()).orElseThrow();
    assertThat(updated.getPatientName()).isEqualTo("Ana García");
    assertThat(updated.getDateTime()).isEqualTo(LocalDateTime.parse("2026-05-10T11:00:00"));
}

@Test
@DisplayName("Devolver 404 al actualizar cita inexistente")
void shouldReturnNotFoundWhenUpdatingNonExistentAppointment() throws Exception {
    String updatePayload = "{\"dateTime\": \"2026-05-10T11:00:00\", \"patientName\": \"Ana García\"}";

    mockMvc.perform(put("/api/appointments/{id}", 999L)
            .contentType(MediaType.APPLICATION_JSON)
            .content(updatePayload))
            .andExpect(status().isNotFound());
}
```

#### Test de DELETE - Eliminar cita

```java
@Test
@DisplayName("Eliminar una cita existente")
void shouldDeleteAppointment() throws Exception {
    // Preparar dato de prueba
    Appointment saved = appointmentRepository.save(
        new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
    );

    // Verificar que existe antes de eliminar
    assertThat(appointmentRepository.count()).isEqualTo(1);

    // Ejecutar DELETE request
    mockMvc.perform(delete("/api/appointments/{id}", saved.getId()))
            .andExpect(status().isNoContent()); // 204 No Content es común para DELETE

    // Verificar que se eliminó de la base de datos
    assertThat(appointmentRepository.count()).isEqualTo(0);
    assertThat(appointmentRepository.findById(saved.getId())).isEmpty();
}

@Test
@DisplayName("Devolver 404 al eliminar cita inexistente")
void shouldReturnNotFoundWhenDeletingNonExistentAppointment() throws Exception {
    mockMvc.perform(delete("/api/appointments/{id}", 999L))
            .andExpect(status().isNotFound());
}
```

### G. Testing con seguridad en el laboratorio

Ahora vamos a extender nuestros tests para incluir verificación de seguridad. Supongamos que el controlador de citas tiene restricciones de roles:

#### Actualizar el controlador con anotaciones de seguridad

```java
@RestController
@RequestMapping("/api/appointments")
public class AppointmentController {
    private final AppointmentService service;

    public AppointmentController(AppointmentService service) {
        this.service = service;
    }

    @PostMapping
    @PreAuthorize("hasRole('PATIENT')")
    public ResponseEntity<Appointment> book(@RequestBody @Valid AppointmentRequest request) {
        Appointment appointment = new Appointment(null, request.getDateTime(), request.getPatientName());
        Appointment saved = service.bookAppointment(appointment);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @GetMapping
    @PreAuthorize("hasAnyRole('PATIENT', 'DOCTOR', 'ADMIN')")
    public ResponseEntity<List<Appointment>> getAll() {
        return ResponseEntity.ok(service.getAll());
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

#### Tests de autenticación y autorización

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
class AppointmentSecurityLabTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppointmentRepository appointmentRepository;

    @BeforeEach
    void clearSecurityContext() {
        SecurityContextHolder.clearContext();
    }

    // TEST 1: Endpoint protegido sin autenticación retorna 401
    @Test
    @DisplayName("POST /api/appointments sin autenticación retorna 401")
    void createAppointmentWithoutAuthReturnsUnauthorized() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isUnauthorized());
    }

    // TEST 2: PATIENT puede crear cita
    @Test
    @WithMockUser(username = "patient@example.com", roles = {"PATIENT"})
    @DisplayName("PATIENT puede crear cita")
    void patientCanCreateAppointment() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.patientName").value("Ana Smith"));
    }

    // TEST 3: DOCTOR no puede crear cita (403 Forbidden)
    @Test
    @WithMockUser(username = "doctor@hospital.com", roles = {"DOCTOR"})
    @DisplayName("DOCTOR no puede crear cita (403 Forbidden)")
    void doctorCannotCreateAppointment() throws Exception {
        String payload = "{\"dateTime\": \"2026-05-15T10:00:00\", \"patientName\": \"Ana Smith\"}";

        mockMvc.perform(post("/api/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isForbidden()); // 403
    }

    // TEST 4: Solo ADMIN puede eliminar citas
    @Test
    @WithMockUser(username = "patient@example.com", roles = {"PATIENT"})
    @DisplayName("PATIENT no puede eliminar cita (403 Forbidden)")
    void patientCannotDeleteAppointment() throws Exception {
        Appointment saved = appointmentRepository.save(
            new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
        );

        mockMvc.perform(delete("/api/appointments/{id}", saved.getId()))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser(username = "admin@hospital.com", roles = {"ADMIN"})
    @DisplayName("ADMIN puede eliminar cita")
    void adminCanDeleteAppointment() throws Exception {
        Appointment saved = appointmentRepository.save(
            new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
        );

        mockMvc.perform(delete("/api/appointments/{id}", saved.getId()))
            .andExpect(status().isNoContent());

        assertThat(appointmentRepository.count()).isEqualTo(0);
    }

    // TEST 5: Múltiples roles
    @Test
    @WithMockUser(roles = {"DOCTOR", "ADMIN"})
    @DisplayName("Usuario con múltiples roles puede acceder a endpoints de ADMIN")
    void userWithMultipleRolesCanDelete() throws Exception {
        Appointment saved = appointmentRepository.save(
            new Appointment(null, LocalDateTime.parse("2026-05-10T09:00:00"), "Ana Smith")
        );

        mockMvc.perform(delete("/api/appointments/{id}", saved.getId()))
            .andExpect(status().isNoContent());
    }
}

// EJEMPLOS ADICIONALES CON JWT (Laboratorio)
// 1) Login -> extraer token -> reutilizar Authorization header
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
class AppointmentJwtFlowLabTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppointmentRepository appointmentRepository;

    @Test
    @DisplayName("Obtener JWT mediante login y usarlo para acceder a endpoint protegido")
    void loginAndUseJwtToken() throws Exception {
        // Crear usuario/credenciales de prueba en la BD si es necesario (depende de la app)

        String loginPayload = "{\"email\": \"patient@example.com\", \"password\": \"password123\"}";

        MvcResult loginResult = mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(loginPayload))
            .andExpect(status().isOk())
            .andReturn();

        String loginBody = loginResult.getResponse().getContentAsString();
        ObjectMapper mapper = new ObjectMapper();
        JsonNode loginJson = mapper.readTree(loginBody);
        String token = loginJson.get("token").asText();

        // Usar token en solicitud protegida
        mockMvc.perform(get("/api/appointments")
                .header("Authorization", "Bearer " + token))
            .andExpect(status().isOk());
    }
}

// 2) Generar JWT programáticamente dentro del test (ejemplo con JJWT)
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
class AppointmentJwtGenerateLabTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("Generar JWT de prueba y usarlo para autenticación")
    void generateJwtAndAccessProtectedEndpoint() throws Exception {
        // Key de ejemplo - en producción usa una clave segura y gestión adecuada
        Key key = Keys.hmacShaKeyFor("test-key-which-is-long-enough-for-hs256".getBytes(StandardCharsets.UTF_8));
        String jwt = Jwts.builder()
            .setSubject("patient@example.com")
            .claim("roles", List.of("ROLE_PATIENT"))
            .setIssuedAt(new Date())
            .setExpiration(Date.from(Instant.now().plus(1, ChronoUnit.HOURS)))
            .signWith(key, SignatureAlgorithm.HS256)
            .compact();

        mockMvc.perform(get("/api/appointments")
                .header("Authorization", "Bearer " + jwt))
            .andExpect(status().isOk());
    }
}

// 3) Mockear JwtDecoder cuando la app usa oauth2ResourceServer
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class AppointmentJwtDecoderMockLabTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private JwtDecoder jwtDecoder;

    @Test
    @DisplayName("Mockear JwtDecoder para aceptar token de prueba")
    void mockJwtDecoderAndAccessProtectedEndpoint() throws Exception {
        Map<String, Object> headers = Map.of("alg", "none");
        Map<String, Object> claims = Map.of("sub", "patient@example.com", "roles", List.of("ROLE_PATIENT"));
        Jwt jwt = new Jwt("token-value", Instant.now(), Instant.now().plus(1, ChronoUnit.HOURS), headers, claims);

        when(jwtDecoder.decode(anyString())).thenReturn(jwt);

        mockMvc.perform(get("/api/appointments")
                .header("Authorization", "Bearer token-value"))
            .andExpect(status().isOk());
    }
}
```

### I. Ejercicio práctico

Pide a los alumnos que implementen los siguientes tests adicionales:

- Validar un GET `/api/appointments` que devuelva todas las citas.
- Probar un `PUT` para actualizar el horario de una cita existente.
- Verificar un endpoint protegido por seguridad y la respuesta `401 Unauthorized`.

### J. Ejercicios adicionales

- Configurar `Testcontainers` con PostgreSQL y `Redis` si hay caché.
- Comparar el mismo test ejecutado con `H2` y con `PostgreSQL`.
- Añadir cobertura de pruebas para excepciones globales con `@ControllerAdvice`.
- Ajustar un Quality Gate para que falle si el build no ejecuta los tests de integración.

### K. Preguntas para reflexionar

**Sobre pruebas de integración en general:**

- ¿Por qué `@SpringBootTest` es más caro que una prueba unitaria?
- ¿Cuándo es correcto usar `@MockBean` en un test de integración?
- ¿Qué falla más probable detecta una prueba de integración que una unitaria no puede detectar?
- ¿Cómo se asegura la prueba de que la DB quedó en el estado esperado?

**Sobre testing de seguridad:**

- ¿Cuál es la diferencia entre `@WithMockUser` y `@WithUserDetails`? ¿Cuándo usar cada una?
- ¿Por qué es importante testear tanto casos exitosos (200 OK) como fallidos (401, 403)?
- ¿Qué diferencia hay entre un `401 Unauthorized` y un `403 Forbidden` en el contexto de tests?
- Si desactivas CSRF en pruebas, ¿qué no estás validando que probablemente deberías?
- ¿Cómo verificas que la autenticación se propaga correctamente desde el controller hasta la capa de servicio?
- ¿Cuándo es apropiado usar `@WithSecurityContext` vs `@WithMockUser`?

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
