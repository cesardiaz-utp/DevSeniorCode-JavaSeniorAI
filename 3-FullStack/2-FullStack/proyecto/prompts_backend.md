# Secuencia de Prompts para el Backend de BiblioKeep (Metodología CIFR)

> **Nota**: Se asume que el proyecto ya fue generado vía Spring Initializr con las dependencias base.

## Prompt 1: Configuración de Entorno y Plugins

El proyecto Spring Boot 3.5+ ya está creado. Necesitamos dejarlo listo para el desarrollo basado en `@especificaciones_detalladas.md`. Configura el archivo `build.gradle` para asegurar la compatibilidad entre `Lombok` y `MapStruct` usando `annotationProcessor`. Además, configura el archivo `application.yml` creando un perfil de dev que defina:

1. Conexión a PostgreSQL (puerto 5432).
2. Estrategia de nombrado de Hibernate (`snake_case`).

No crees clases de Java aún. Solo asegura que el entorno de compilación y configuración sea correcto.

## Prompt 2: Entidades de Datos, DTOs y Mappers (MapStruct)

Con el entorno configurado, definiremos el modelo de datos según la Sección 3 de `@especificaciones_detalladas.md`. 

1. Implementa las entidades JPA (`User`, `Book`, `Loan`).
2. Crea registros (Records) de Java para los DTOs de entrada y salida (ej: `BookRequestDTO`, `BookResponseDTO`).
3. Crea interfaces de MapStruct para el mapeo entre Entidades y DTOs (ej: `BookMapper`) usando `componentModel = "spring"`.

Restricciones:

- Usa `UUID` para `User`.
- Implementa los Enums para `status`.
- Asegura unicidad en `isbn` y `email`.
- Aplica las validaciones de `jakarta.validations` donde corresponda.

## Prompt 3: Funcionalidad de Préstamos y Dashboard (Lógica Local)

Implementaremos la gestión de préstamos y estadísticas locales antes de servicios externos. Implementa el `LoanService`, `LoanRepository` y el `BookController` para el CRUD básico. Además, crea el endpoint `/api/stats/dashboard` y la tarea programada (`@Scheduled`) de la Sección 5 para notificaciones de mora.

Restricciones:

- Inyecta y usa los **Mappers** para las respuestas de la API.
- El Dashboard debe calcular el progreso hacia la meta anual (`annualGoal`) del usuario.
- Simula el usuario autenticado extrayendo un `user-id` desde un Header manual por ahora.

## Prompt 4: Gestión de Préstamos y Lógica de Negocio

Implementaremos la lógica de préstamos descrita en la Sección 3 y 4. Implementa el `LoanRepository`, `LoanService` y `LoanController`.

1. Crea la lógica para registrar un préstamo.
2. Al prestar un libro, actualiza automáticamente el campo `isLent=true` en la entidad `Book`.
3. Crea la lógica para marcar un préstamo como devuelto (`returned=true`).

Restricciones:

- Valida que un libro no pueda ser prestado si ya está marcado como `isLent=true`.
- Sigue el contrato de endpoints de la Sección 4.

## Prompt 5: Integración con Redis y API Externa (Caché Híbrida)

Ahora que la lógica local es sólida, integraremos Redis y Google Books según la Sección 5 de `@especificaciones_detalladas.md`.

1. Configura la conexión a **Redis** (puerto 6379) en el `application.yml`.
2. Implementa el `SearchService` con la siguiente lógica:
    - Buscar por ISBN en `BookRepository`.
    - Si falla, buscar en **Redis** (`isbn:{number}`).
    - Si falla, consultar **Google Books API**.
    - Si se halla externamente, guardar en Redis con un TTL de 24 horas.

Restricciones:

- Maneja excepciones para API no disponible.
- Asegúrate de que los datos externos se mapeen correctamente al DTO de nuestra aplicación usando el Mapper.

## Prompt 6: Configuración de MailSender y Notificaciones Programadas

Implementaremos el sistema de alertas asíncronas definido en la Sección 5.

1. Configura el **JavaMailSender** en el `application.yml` para conectar con **MailHog** (puerto 1025).
2. Implementa la tarea programada (`@Scheduled`) que se ejecute diariamente a las 8 AM.
3. La tarea debe buscar préstamos vencidos y enviar un correo electrónico al dueño del libro con los detalles.
Formato: Actualización de `application.yml`, servicio de correo y componente `@Scheduled`.

Restricciones:

- El contenido del correo debe ser profesional y parametrizado.
- Asegúrate de habilitar el soporte para tareas programadas con `@EnableScheduling`.

## Prompt 7: Infraestructura con Docker y Testing

El código es funcional y requiere servicios de soporte para pruebas finales.

1. Genera el archivo docker-compose.yml basado en la Sección 7 de `@especificaciones_detalladas.md` (PostgreSQL, Redis, MailHog).
2. Genera tests unitarios para SearchService (usando Mockito) y un test de integración para el flujo de préstamos.

Restricciones:

- El test de búsqueda debe verificar la interacción con la caché de Redis.
- No realices peticiones reales a Google; usa mocks.

## Prompt 8: Seguridad y JWT (Cierre del Proyecto)

Implementación de la capa de seguridad final (Sección 4). Configura Spring Security con JWT. Implementa el `AuthService` para registro/login y el filtro de validación de tokens. Refactoriza los controladores para obtener el usuario real desde el `SecurityContext` en el servicio.

- Rutas de `/api/auth/**` deben ser públicas.
- Usa BCrypt para passwords.
- Implementa el manejo de errores 401/403 con respuestas JSON estructuradas.
