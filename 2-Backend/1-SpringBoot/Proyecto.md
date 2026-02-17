# Unidad 1: MediConnect API

## Escenario de Negocio

El sector salud a nivel global está experimentando una transformación digital acelerada, impulsada por la necesidad de accesibilidad y eficiencia. La startup **MediConnect** nace con el propósito de centralizar la gestión de la salud digital, permitiendo que la interacción entre pacientes y especialistas sea fluida, segura y basada en datos. Actualmente, los centros médicos sufren de fragmentación en sus agendas y dificultades para validar la disponibilidad real de sus especialistas en tiempo real.

El objetivo primordial de esta unidad es construir la **Core API de MediConnect**. Esta infraestructura servirá como el corazón de un ecosistema donde convergen aplicaciones móviles de pacientes y portales web de doctores. El desarrollo se centrará en la **excelencia arquitectónica**, la implementación de una **validación de datos estricta** (basada en protocolos médicos y de seguridad) y la adopción de los estándares **RESTful** de nivel empresarial.

En esta etapa inicial, aunque la persistencia se manejará **en memoria** para enfocarnos en la lógica de Spring, el diseño debe ser lo suficientemente desacoplado (usando interfaces y DTOs) para que la migración a una base de datos SQL sea una tarea de configuración mínima en el futuro.

## Requerimientos Funcionales

El sistema debe exponer y gestionar los siguientes recursos mediante una interfaz profesional:

### A. Gestión de Especialistas (Doctor Resource)

- **Listar Profesionales (`GET /api/v1/doctors`)**: Recuperar una lista de todos los especialistas. Debe soportar la inclusión de metadatos básicos como nombre y especialidad.
- **Consulta Detallada (`GET /api/v1/doctors/{id}`)**: Obtener el perfil clínico completo del doctor, incluyendo años de experiencia y horarios generales.
- **Registro de Facultativos (`POST /api/v1/doctors`)**: Crear un nuevo perfil de especialista. El sistema debe validar que la **Tarjeta Profesional** cumpla con el formato legal y que la especialidad pertenezca a un catálogo predefinido.
- **Búsqueda por Especialidad (`GET /api/v1/doctors/specialty/{name}`)**: Filtrar de forma dinámica para que los pacientes puedan encontrar rápidamente cardiólogos, pediatras, etc.

### B. Gestión Inteligente de Citas (Appointment Resource)

- **Agendamiento (`POST /api/v1/appointments/schedule`)**: Solicitar una cita médica vinculando un paciente con un doctor en una fecha y hora específicas.
- **Monitor de Citas (`GET /api/v1/appointments`)**: Una vista administrativa para listar todas las citas, permitiendo auditoría del sistema.
- **Historial de Paciente (`GET /api/v1/appointments/patient/{id}`)**: Consultar todas las intervenciones y citas pasadas/futuras de un paciente para mantener la continuidad del cuidado.
- **Cancelación y Gestión (`DELETE /api/v1/appointments/{id}`)**: Permitir la cancelación de citas con un sistema de notificación lógica (por ahora, vía logs).

## Especificaciones Técnicas y Arquitectura

### 1. Estructura de Capas de Alto Nivel (Package Structure)

Se requiere una separación de responsabilidades estricta siguiendo el principio de **Separación de responsabilidades (SoC - Separation of Concerns)**:

- `controller`: Actúa como la fachada del sistema. Gestiona las peticiones HTTP, define los contratos de la API y delega la ejecución a los servicios.
- `service`: El cerebro de la aplicación. Aquí reside la lógica de negocio, las validaciones cruzadas (ej: disponibilidad horaria) y la orquestación de datos.
- `repository`: Capa de persistencia. Implementada inicialmente con `ConcurrentHashMap` o `ArrayList` para garantizar que la API funcione de forma volátil pero estructurada.
- `dto`: Implementación mediante **Java Records**. Proporcionan objetos de transporte inmutables, protegiendo el modelo interno y optimizando la serialización JSON.
- `exception`: Implementación del patrón **Global Exception Handler** para evitar fugas de información técnica hacia el cliente.

### 2. Modelado de Datos y Serialización

- **Inmutabilidad**: Uso mandatorio de Records para garantizar que los datos no sean alterados durante el tránsito entre capas.
- *Personalización JSON (Jackson)*:
  - `@JsonProperty`: Los contratos deben ser en `snake_case` (ej: `license_number`, `specialty_type`) para alinearse con estándares internacionales de APIs.
  - `@JsonFormat`: Las fechas deben seguir el estándar ISO-8601 (`yyyy-MM-dd'T'HH:mm:ss`) para evitar ambigüedades horarias.
  - `@JsonInclude`: Evitar el envío de campos nulos en las respuestas para ahorrar ancho de banda.

### 3. Reglas de Negocio y Validación Robusta

- **Integridad de Datos**: Uso de `jakarta.validation` para asegurar que ningún campo crítico sea nulo (`@NotNull`) o esté vacío (`@NotBlank`).
- **Seguridad Clínica**: Validar correos electrónicos con `@Email` y asegurar que las citas se programen únicamente en fechas futuras con `@Future`.
- **Excepciones de Negocio**: No usar excepciones genéricas. Se deben lanzar y capturar excepciones semánticas como `DoctorNotFoundException`, `PatientForbiddenException` o `ScheduleConflictException`.
- **RFC 7807**: Todas las respuestas de error deben incluir el "Problem Details", que contiene: tipo de error, título, código de estado y el mensaje de ayuda.

### 4. Configuración y Documentación

- **SpringDoc OpenAPI (Swagger)**: Configuración automática de la documentación. Cada endpoint debe estar mínimamente descrito para que un desarrollador de Frontend pueda consumirlo.
- **Configuración Externa**:
  - Uso de `application.properties` para centralizar propiedades.
  - Configuración de niveles de log (`logging.level.root: INFO` y logs específicos de la aplicación en `DEBUG`).
  - Definición de mensajes personalizados o valores constantes del sistema desde el archivo PROPERTIES.

## Rúbrica de Evaluación Detallada

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Diseño RESTful de Grado 2** | Uso correcto de verbos (GET, POST, DELETE), códigos de estado (201 para creación, 204 para borrado, 400 para error del cliente) y URIs pluralizadas. | 25 |
| **Separación de Responsabilidades** | El código debe estar desacoplado. Un controlador no debe conocer la lógica de persistencia. Inyección de dependencias obligatoria por constructor. | 20 |
| **Validación y Manejo de Errores** | El sistema no debe permitir datos inválidos. El `GlobalExceptionHandler` debe capturar errores de validación de Bean y excepciones de negocio. | 25 |
| **Principios SOLID** | Se evaluará la cohesión de los servicios y la inversión de dependencias. Uso de interfaces para definir contratos de servicio. | 20 |
| **Calidad de Configuración** | Uso de `application.properties` y una documentación Swagger funcional que permita probar el 100% de la API. | 10 |

## Niveles de Desempeño

- **Senior (90-100)**: La API es un producto terminado. Incluye validaciones complejas (ej: no permitir dos citas al mismo tiempo para el mismo doctor), manejo de errores impecable bajo estándar RFC 7807, y el código es altamente modular y testable.
- **Semi-Senior (75-89)**: La arquitectura en capas es sólida y los endpoints funcionan. El manejo de errores es centralizado pero quizás le faltan detalles de personalización o la documentación Swagger es muy básica.
- **Junior (60-74)**: El sistema cumple con la funcionalidad básica, pero existen mezclas de responsabilidades (ej: lógica de negocio en el controlador) o faltan validaciones críticas en los DTOs.
- **Insuficiente (<60)**: El código presenta errores de compilación, no utiliza el contenedor de Spring correctamente o los endpoints no siguen las convenciones mínimas de REST.

## Entregables y Criterios de Aceptación

1. **Repositorio de GitHub**: Debe contener una estructura de paquetes limpia y un historial de commits que refleje el progreso (mínimo 10 commits).
2. **Script de Pruebas Automáticas (`api-tests.http`)**: Un archivo ejecutable en VS Code que permita probar el flujo completo: Crear doctor -> Listar doctor -> Agendar cita -> Error por cita duplicada.
3. **Documentación Técnica (README.md)**: Guía clara sobre cómo ejecutar la aplicación, cómo cambiar de perfiles y ejemplos de los payloads JSON esperados.
4. **Video de Sustentación (Max 5 min)**:
    - Demostración de los controladores y servicios.
    - Prueba exitosa en Swagger UI.
    - Prueba de error de validación (ej: enviar un email inválido o una fecha pasada) y mostrar la respuesta JSON del error.
