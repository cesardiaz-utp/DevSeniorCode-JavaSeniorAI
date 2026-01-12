# Proyecto Integrador Módulo 2: "Sistema de Gestión Hospitalaria (MediCare API)"

## Descripción del Escenario

Una red de clínicas de rápido crecimiento necesita modernizar su infraestructura tecnológica crítica. El sistema actual de gestión de citas y pacientes es manual, propenso a errores de duplicación y carece de seguridad. Se requiere desarrollar el backend robusto ("MediCare API") que actúe como el cerebro digital de la clínica, permitiendo una gestión eficiente, segura y auditada de la atención médica.

## Objetivos de Aprendizaje Evaluados

Este proyecto valida la competencia del estudiante en:

1. **Ingeniería Backend (Spring Boot 4)**: Construcción de APIs RESTful siguiendo el modelo de madurez de Richardson y Arquitectura Limpia.
2. **Persistencia Híbrida**: Manejo avanzado de **JPA** (PostgreSQL) para datos relacionales críticos y **NoSQL** (MongoDB) para datos clínicos no estructurados.
3. **Seguridad Empresarial**: Implementación de **Spring Security 7** con JWT y control de acceso basado en roles (RBAC).
4. **Integración y Resiliencia**: Consumo de servicios externos con `RestClient` y patrones de tolerancia a fallos.

## Visión General del Sistema

El problema central que "MediCare" resuelve es la ineficiencia en la asignación de recursos médicos y la fragmentación de la información del paciente. Actualmente, los doctores pierden tiempo valioso buscando historiales en papel y los pacientes sufren esperas debido a agendas mal coordinadas.

El sistema debe soportar tres pilares operativos fundamentales:

1. **Gestión de Agendas Inteligente**: El sistema debe impedir estrictamente que un médico sea asignado a dos pacientes simultáneamente. La integridad de la agenda es la prioridad número uno. Además, debe permitir cancelaciones con reglas de negocio estrictas para minimizar los "huecos" improductivos.
2. **Historial Clínico Unificado**: Mientras que los datos administrativos (quién es el paciente, cuánto pagó) son rígidos y relacionales, la realidad médica (notas de evolución, síntomas, diagnósticos preliminares) es caótica y variable. El sistema debe ser capaz de almacenar estas notas evolutivas de forma flexible sin obligar a cambios constantes en el esquema de la base de datos.
3. **Seguridad y Auditoría**: Dado que se maneja información sensible de salud, el sistema no puede ser una "caja negra". Cada acción crítica (especialmente cambios en citas o diagnósticos) debe dejar un rastro digital inmutable de quién hizo qué y cuándo.

## Requerimientos Funcionales (Lo que hace el sistema)

### 1. Seguridad y Usuarios

- **Registro Público**: Los pacientes pueden registrarse libremente creando su cuenta (`Role.PATIENT`).
- **Gestión Administrativa**: Los roles `ADMIN` y `DOCTOR` solo pueden ser creados por un administrador existente (endpoint protegido).
- **Autenticación y Permisos**: Login seguro que retorna un **JWT**. Todas las peticiones subsiguientes deben validarse con este token.

### 2. Gestión de Citas (Core Relacional)

- **Agendamiento**: Un paciente puede reservar una cita (`Appointment`) con un médico específico en una fecha/hora disponible.
- **Validación de Disponibilidad**: El sistema debe rechazar (Error `400`/`409`) cualquier intento de cita si el doctor ya tiene otra asignada en ese horario (overlap check). Debe lanzar `AppointmentConflictException` (`409 Conflict`).
- **Cancelación**:
  - **Pacientes**: Pueden cancelar sus propias citas.
  - **Regla de Negocio**: No se permite cancelar con menos de 24 horas de antelación. Debe lanzar `LateCancellationException` (`400 Bad Request`).

### 3. Historial Clínico (Módulo NoSQL)

- **Notas de Consulta**: Al finalizar una cita, el doctor debe poder agregar una "Nota de Evolución" (`MedicalRecord` o `ClinicalNote`)..
- **Flexibilidad**: Estas notas deben guardarse en MongoDB. Deben incluir datos variables (ej. un día solo texto, otro día lista de síntomas y prescripción) sin alterar tablas SQL.

### 4. Integración Externa (Resiliencia)

- **Validación de Medicamentos**: Al crear una receta o nota, el sistema debe consultar una API pública externa (ej. _OpenFDA_ o una API Mock creada en Postman) para verificar si el medicamento existe.
- **Circuit Breaker**: Si la API externa tarda más de 2 segundos o falla, el sistema no debe caerse. Debe retornar una advertencia ("Verificación de medicamento pendiente") y permitir guardar la nota localmente.

## Requerimientos Técnicos (Cómo se construye)

### 1. Stack y Arquitectura

- **Framework**: Spring Boot 4 (Java 25).
- **Arquitectura**: Capas estrictas (`Controller` -> `Service` -> `Repository`). Uso de DTOs (Records) obligatorios para entrada y salida (ej. `AppointmentRequestDto`, `DoctorResponseDto`). No exponer Entidades JPA.
- **Bases de Datos (Instalación Local)**:
  - PostgreSQL para `User`, `Doctor`, `Patient`, `Appointment`.
  - MongoDB para `MedicalRecord`, `AuditLog`.

### 2. Modelo de Datos (JPA & Mongo)

- **Entidades (English Naming)**: `User`, `Role` (Enum: `ADMIN`, `DOCTOR`, `PATIENT`), `Doctor`, `Patient`, `Appointment`.
- **Relaciones**:
  - `Doctor` 1:N `Appointment`.
  - `Patient` 1:N `Appointment`.
- **Auditoría**: Usar JPA Auditing (`@CreatedDate`, `@LastModifiedBy`) en las entidades relacionales.
- **Migraciones (Flyway)**: La creación y evolución del esquema de PostgreSQL debe realizarse mediante scripts SQL versionados (`V1__init_schema.sql`, `V2__add_column.sql`). Está prohibido confiar en `ddl-auto` para la generación de tablas en producción.

### 3. Calidad y Testing

- **Logging (Log4j 2)**: Implementar logs estructurados para el seguimiento de errores y eventos de seguridad.
  - Ejemplo: `logger.info("Login successful for user: {}", username);`
  - Ejemplo: `logger.error("Failed to sync with external API", exception);`
- **Testing (JUnit 5 + Mockito)**:
  - Unit Tests para la lógica de validación de horarios (sin levantar contexto de Spring).
  - Integration Test para el flujo de Agendar Cita (usando H2 o Testcontainers si es posible, o mocks de repositorio).

### 4. Documentación

- **OpenAPI 3**: Configurar `springdoc-openapi` para generar la documentación interactiva en `/swagger-ui.html`. Los endpoints deben tener descripciones claras y ejemplos de respuestas (`200`, `400`, `404`, `500`).

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Arquitectura & Spring Boot** | Estructura de paquetes correcta, Inyección de Dependencias, uso de `application.yml` con perfiles, y separación limpia DTO/Entity (MapStruct o manual). | 20 |
| **Persistencia Híbrida** | Mapeo JPA correcto (Relaciones, Constraints). Uso adecuado de Repositorios, conexión exitosa a Postgres y Mongo locales, y gestión de esquema con Flyway. | 20 |
| **Seguridad (Security 7)** | Implementación correcta de `SecurityFilterChain`, Filtro JWT, y protección de endpoints por Rol (`@PreAuthorize`). | 20 |
| **Lógica & Resiliencia** | Algoritmo de validación de horarios correcto. Implementación de `RestClient` con `CircuitBreaker` para la API externa. | 20 |
| **Calidad (Tests & Docs)** | Swagger funcional y completo. Tests unitarios que cubren casos de éxito y fallo (excepciones como `AppointmentConflictException`) de la lógica de negocio. | 20 |

## Niveles de Desempeño

- **Senior (90-100)**: Arquitectura impecable, manejo de excepciones global (`GlobalExceptionHandler`) con formato estándar (RFC 7807), seguridad robusta y tests exhaustivos.
- **Semi-Senior (75-89)**: Funcionalidad completa, bases de datos integradas correctamente, pero faltan detalles en manejo de errores o la cobertura de tests es baja.
- **Junior (60-74)**: La aplicación funciona en el "happy path" pero falla con datos incorrectos, la seguridad es básica o mezcla lógica en el controlador.
- **Insuficiente (<60)**: No compila, no conecta a bases de datos o no implementa seguridad.

## Entregables

1. Repositorio de **GitHub** con el código fuente (ignorando archivos de configuración del IDE).
2. Archivo `README.md` explicando cómo levantar las bases de datos locales, configurar las variables de entorno y ejecutar el proyecto.
3. Colección de **Postman** o archivo JSON de Swagger exportado.
4. Video corto (3-5 min) mostrando: Login, Agendar Cita (éxito y fallo por horario ocupado) y Consulta de Historial.
