# Proyecto Módulo 2: MediConnect Enterprise Pro

## Escenario de Negocio: La Evolución a MediConnect v2.0

En la unidad anterior, logramos que **MediConnect** persistiera datos de forma híbrida, integrando la rigidez transaccional de **PostgreSQL** con la flexibilidad documental de **MongoDB**. Sin embargo, en el mundo real, una aplicación de salud no solo debe "funcionar", sino que debe ser capaz de resistir ataques, escalar bajo demanda masiva y sobrevivir a fallos de servicios de terceros.

El sistema evoluciona hacia la categoría Enterprise. Ya no se trata de una API abierta de pruebas; ahora estamos construyendo una plataforma blindada y resiliente. El objetivo es convertir a **MediConnect** en un sistema de grado bancario aplicado a la salud, donde la información sensible de los pacientes esté custodiada por los estándares más estrictos de **Spring Security 7**, donde la infraestructura sea capaz de procesar miles de consultas de telemedicina simultáneas mediante **Virtual Threads**, y donde la experiencia del doctor sea ininterrumpida frente a la inestabilidad de la red gracias a patrones avanzados de **Resiliencia**.

## Requerimientos Funcionales

### A. Seguridad Avanzada y Jerarquía de Roles

El sistema debe abandonar la autenticación básica para implementar una arquitectura **Stateless** basada en Tokens. Es imperativo diferenciar los niveles de acceso para mitigar riesgos de filtración de datos sensibles:

- **Public Access (Endpoints Abiertos)**: Registro de nuevos pacientes y consulta de especialidades médicas (ej. "Cardiología", "Pediatría"). Esto permite la captación de nuevos usuarios sin barreras técnicas iniciales.
- **Patient Role (`ROLE_PATIENT`)**: Los pacientes son los dueños de sus datos. Tienen permisos exclusivos para agendar sus propias citas, consultar el historial de "Notas de Evolución" almacenado en MongoDB y actualizar su perfil demográfico. Bajo ninguna circunstancia un paciente podrá ver datos de otro paciente.
- **Doctor Role (`ROLE_DOCTOR`)**: El personal médico tiene una vista operativa. Pueden visualizar su agenda diaria cargada desde PostgreSQL, completar citas pendientes y escribir nuevas entradas en la historia clínica (NoSQL). La seguridad debe garantizar que un doctor solo pueda escribir notas para citas en las que él fue el profesional asignado.
- **Admin Role (`ROLE_ADMIN`)**: Gestión crítica de la infraestructura. Este rol tiene el poder de dar de alta a nuevos doctores, gestionar consultorios y acceder a los endpoints sensibles de **Actuator** para monitorear la salud del servidor.
- **Autenticación y JWT**: Implementar el endpoint `/api/v1/auth/login`. Tras una validación exitosa de credenciales, el servidor debe emitir un JWT firmado mediante un secreto HMAC-512 que incluya los permisos (Claims) necesarios para que el cliente no tenga que re-autenticarse constantemente.

### B. Lógica de Negocio Crítica y Transaccionalidad

La fiabilidad de los datos en salud es una cuestión de integridad física y logística:

- **Validación Atómica de Agendas**: Al intentar agendar una cita en PostgreSQL, el servicio debe garantizar que no existan traslapes. El sistema debe ejecutar una consulta de "overlap check" (ej. verificar si el `doctorId` ya tiene un registro entre `fechaInicio` y `fechaFin`). Este proceso debe ser transaccional; si dos pacientes intentan reservar el mismo minuto exacto, la base de datos debe rechazar uno de ellos mediante bloqueos optimistas o validaciones de servicio estrictas, arrojando una `AppointmentConflictException` (HTTP 409).
- **Políticas de Cancelación con Penalidad Lógica**: Para evitar huecos improductivos en la clínica, un paciente solo puede cancelar su cita de forma autónoma si faltan más de 24 horas para la misma. Si la solicitud llega dentro del marco de las 24 horas previas, el sistema debe denegar la operación y arrojar una `LateCancellationException` (HTTP 400).

### C. Estrategia de Resiliencia ante Fallos Externos

**MediConnect** no es una isla; necesita interactuar con el ecosistema de salud global:

- **Consumo de API de Farmacología**: Al finalizar una consulta, el sistema debe validar si los medicamentos recetados por el doctor están vigentes y son correctos consultando una API externa (ej. `OpenFDA`). Para esto, se debe utilizar el nuevo `RestClient` de Spring, aprovechando su sintaxis fluida y eficiente.
- **Implementación de Circuit Breaker (Resilience4j)**: Los servicios externos son inherentemente inestables. Debes configurar un "Interruptor" que, ante una tasa de fallo superior al 50% o tiempos de respuesta mayores a 1.5 segundos, "abra el circuito".
- **Fallback Method**: Cuando el circuito esté abierto, el sistema no debe fallar con un error 500. En su lugar, debe ejecutar una lógica de respaldo (fallback) que marque la receta con un estado de "Validación Manual Requerida por Farmacia". Esto permite que el flujo médico continúe sin depender de la conectividad de la API externa.

## Especificaciones Técnicas

### 1. Concurrencia Masiva y Escalabilidad

**Virtual Threads (Project Loom)**: En un escenario de alta demanda, los hilos tradicionales de la JVM consumen demasiada memoria RAM. Se debe habilitar el soporte de **Virtual Threads** en la configuración de Spring Boot. Esto permitirá que **MediConnect** maneje miles de conexiones concurrentes de telemedicina de forma ligera, escalando horizontalmente sin necesidad de servidores gigantescos.

### 2. Optimización del Performance con Redis

**Capa de Caché Distribuido**: Implementar la abstracción de caché de Spring con **Redis**. El listado de doctores por especialidad y el catálogo de medicamentos validados deben ser cacheados. Dado que estos datos no cambian segundo a segundo, servirlos desde memoria (Redis) reducirá la latencia de 200ms a menos de 10ms, mejorando drásticamente la percepción de velocidad del usuario.

### 3. Observabilidad y Diagnóstico

- **Health & Metrics**: No puedes arreglar lo que no puedes ver. Actuator debe estar configurado para exponer la salud de las conexiones a Postgres, Mongo y el estado del cluster de Redis.
- **Métricas de Negocio**: Crear métricas personalizadas (Counter) para monitorear eventos críticos, como el número de cancelaciones tardías o el número de veces que el Circuit Breaker ha entrado en acción.

### 4. Excelencia Arquitectónica

- **Inmutabilidad con Records**: Todos los DTOs de entrada y salida deben definirse como **Java Records**. Esto garantiza que los datos no sufran mutaciones inesperadas durante el procesamiento.
- **Mapeo Desacoplado**: Utilizar **MapStruct** para transformar Entidades en Records y viceversa. Esto evita el código repetitivo de "getters y setters" y mantiene la lógica de negocio limpia de detalles de persistencia.
- **Estandarización de Errores**: Implementar un `GlobalExceptionHandler` que utilice el estándar **ProblemDetail** (RFC 7807), asegurando que el cliente siempre reciba una respuesta JSON coherente con el error ocurrido.

## Configuración de Entornos y Gestión de Propiedades

El uso correcto de archivos `.properties` es vital para el ciclo de vida de desarrollo de software (SDLC):

- `application-dev.properties` **(Agilidad)**:
  - **Relacional**: _H2 Database_ (In-memory). Ideal para pruebas rápidas y limpieza de datos en cada reinicio.
  - **Caché**: `SimpleCacheManager` (Map en memoria local).
  - **Logging**: Nivel `DEBUG` para rastrear la generación de SQL de Hibernate.
- `application-prod.properties` **(Robustez)**:
  - **Relacional**: _PostgreSQL_ con Pool de conexiones Hikari optimizado.
  - **Documental**: _MongoDB Atlas_ para persistencia de largo plazo de notas clínicas.
  - **Caché**: Servidor de _Redis_ dedicado.
  - **Secretos**: El `JWT_SECRET` y las credenciales de base de datos **nunca** deben estar hardcodeadas; deben inyectarse mediante variables de entorno del sistema.

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio de Excelencia (Senior) | Puntos |
| --- | --- | --- |
| **Seguridad JWT** | Filtro de seguridad que valida el token en cada request, manejo de excepciones de expiración de token y protección de métodos mediante `@PreAuthorize`. | 25 |
| **Lógica & Resiliencia** | Validación de solapamiento de citas que maneja condiciones de carrera. Circuit Breaker configurado con tiempos de espera (timeouts) y fallback funcional. | 25 |
| **Persistencia Híbrida** | Uso de transacciones para SQL y auditoría automática (`@CreatedBy`) para rastrear el origen de los datos en ambos motores. | 20 |
| **Rendimiento & Ops** | Demostración de hilos virtuales activos, uso efectivo de anotaciones `@Cacheable` y endpoints de Actuator personalizados. | 15 |
| **Arquitectura & Código** | Código 100% modular, uso de Records, MapStruct sin errores de mapeo y Swagger (OpenAPI) con descripciones de cada endpoint. | 15 |

## Niveles de Desempeño y Expectativas

- **Senior (90-100)**: El estudiante entrega una arquitectura desacoplada y profesional. El sistema es capaz de fallar con gracia (graceful degradation) gracias al Circuit Breaker. El código está autodocumentado y los tests unitarios cubren la lógica de validación de horarios y el cálculo de las 24 horas de cancelación.
- **Semi-Senior (70-89)**: La solución es funcional y segura. Implementa la mayoría de los requerimientos técnicos, pero carece de afinación en la configuración de la caché (ej. no define TTL para los datos en Redis) o el manejo de errores no sigue estrictamente el estándar RFC 7807.
- **Junior (60-69)**: El sistema cumple con los requisitos básicos de seguridad y persistencia, pero presenta lógica de negocio dentro de los controladores o el flujo de "fallback" del Circuit Breaker simplemente arroja un error genérico en lugar de una alternativa válida.

## Entregables y Metodología de Entrega

1. **Repositorio de GitHub**: Con un historial de commits que demuestre la evolución del proyecto. Debe incluir un archivo `.gitignore` adecuado para proyectos Spring Boot.
2. **Postman Collection**: Un archivo JSON con todas las rutas configuradas, incluyendo variables de entorno para el `Bearer Token`, facilitando la revisión de los flujos de seguridad.
3. **Archivo README.md**: Guía técnica que explique cómo levantar los servicios necesarios (Redis, Postgres, Mongo) preferiblemente mediante un archivo `docker-compose.yml` proporcionado.
4. **Video de Sustentación (Max 5 min)**: Demostración grabada donde se aprecie el bloqueo de un endpoint protegido (403 Forbidden) y cómo el sistema se recupera cuando simulas la caída de la API externa de medicamentos.
