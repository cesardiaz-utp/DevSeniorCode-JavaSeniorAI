# Unidad 2: MediConnect Data Pro

## Escenario de Negocio: La Evolución de MediConnect

**MediConnect** ha dejado de ser un prototipo conceptual para convertirse en una plataforma crítica que procesa miles de registros diariamente. En este punto de crecimiento, la gestión de datos en memoria es inviable debido a la volatilidad de la información, la falta de integridad referencial y la necesidad de cumplir con normativas legales de auditoría regulatoria en salud.

En esta unidad, transformaremos la aplicación en un sistema de grado empresarial capaz de manejar datos heterogéneos. Utilizaremos **PostgreSQL** como el núcleo transaccional de alta consistencia para entidades estructuradas (Doctores, Pacientes, Citas). Simultáneamente, integraremos **MongoDB** para gestionar el módulo de **Historia Clínica No Estructurada (Evolution Notes)**. Este enfoque híbrido permite que los doctores guarden notas de texto libre, diagnósticos con esquemas variables y observaciones clínicas detalladas que no encajan en la rigidez de una tabla SQL. Finalmente, implementaremos una estrategia avanzada de **Perfiles de Spring** para garantizar que la transición entre entornos de desarrollo y producción sea segura, automatizada y libre de errores de configuración manual.

## Requerimientos Funcionales y de Datos

### A. Capa Relacional

El sistema debe migrar la lógica de negocio a un modelo relacional normalizado, implementando las siguientes entidades con sus respectivos constraints de integridad:

1. **`Doctors` (Especialistas)**: Registro de información profesional, especialidad médica y número de colegiado. Se debe asegurar que el número de colegiado sea único a nivel de base de datos.
2. **`Patients` (Pacientes)**: Datos personales, contacto y número de identificación. Es vital garantizar la integridad de los datos de contacto para las notificaciones.
3. **`Appointments` (Citas Médicas)**: Relación **Many-to-One** entre Pacientes y Doctores. Debe incluir campos de fecha, hora, estado de la cita (PROGRAMADA, COMPLETADA, CANCELADA) y un enlace lógico hacia la nota clínica en MongoDB.
4. **`MedicalOffices` (Consultorios)**: Implementar una relación **One-to-One** con la entidad `Doctor`. Cada especialista tiene asignado un espacio físico único para sus consultas presenciales.

### B. Capa Documental

Cada vez que una cita se marca como "COMPLETADA", el sistema debe disparar la creación de un documento en MongoDB que represente la evolución del paciente:

1. **Clinical Notes (Notas de Evolución)**:
    - **Referencia**: Debe contener el ID de la cita generado en PostgreSQL para mantener el vínculo entre ambas bases de datos.
    - **Observaciones**: Un campo de texto enriquecido para las notas subjetivas del doctor.
    - **Sintomatología**: Un arreglo (Array) de strings para facilitar búsquedas posteriores por síntomas comunes.
    - **Diagnóstico**: Un objeto anidado que permita flexibilidad (ej. código CIE-10, descripción, nivel de gravedad).
    - **Metadatos BSON**: Campos dinámicos que permitan al doctor añadir información extra (ej. signos vitales capturados por dispositivos externos).

## Especificaciones Técnicas

### 1. Configuración por Perfiles

Para desacoplar la lógica de la infraestructura, se utilizará el formato tradicional de Java Properties. El estudiante debe configurar:

- `application-dev.properties` (Entorno de Desarrollo):
  - `spring.datasource.url=jdbc:h2:mem:mediconnect_db` (Uso de base de datos en memoria para rapidez).
  - `spring.datasource.driverClassName=org.h2.Driver`
  - `spring.h2.console.enabled=true` (Habilitar consola web para inspección).
  - `spring.jpa.database-platform=org.hibernate.dialect.H2Dialect`
  - `spring.data.mongodb.uri=mongodb://localhost:27017/mediconnect_dev`
  - `spring.jpa.show-sql=true`
  - `logging.level.com.mediconnect.api=DEBUG`
- `application-prod.properties` (Simulación de Producción Robusta):
  - `spring.datasource.url=jdbc:postgresql://localhost:5432/mediconnect_prod`
  - `spring.datasource.username=${DB_USER}` (Uso de variables de entorno).
  - `spring.datasource.password=${DB_PASSWORD}`
  - `spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect`
  - `spring.data.mongodb.uri=${MONGO_URI}`
  - `spring.jpa.show-sql=false`
||- `logging.level.root=WARN`
- `application.properties`:
  - `spring.application.name=MediConnect-API`
  - `spring.profiles.active=dev` (Perfil por defecto).

### 2. Persistencia, Mapeo y Rendimiento (ORM & ODM)

- **Lombok**: Integración mandatoria de `@Data`, `@Builder` y `@NoArgsConstructor` para garantizar que las entidades sean legibles y libres de código repetitivo.
- **JPA & Hibernate**: Gestión experta de relaciones. Se evaluará rigurosamente el uso de `FetchType.LAZY`. Cargar datos de forma "Eager" en un sistema de salud puede degradar el rendimiento al traer historiales médicos completos innecesariamente.
- **MapStruct**: Creación de interfaces de mapeo para asegurar que las Entities nunca salgan de la capa de servicio. Los DTOs deben ser los únicos objetos que viajen hacia los controladores y el cliente final.

### 3. Ciclo de Vida y Transaccionalidad

- **Flyway**: Implementación de migraciones SQL versionadas. Cada cambio estructural en PostgreSQL debe estar documentado en archivos como `V1__create_initial_schema.sql`. No se permite el uso de `ddl-auto: update`.
- **Atomicidad Transaccional**: El uso de `@Transactional` es crítico. Si la creación de la nota en MongoDB falla por un error de red, la cita en PostgreSQL debe hacer un **rollback** y no marcarse como completada, garantizando que no existan citas "huérfanas" de historia clínica.

## Rúbrica de Evaluación

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Persistencia Híbrida** | Integración funcional de SQL (H2/Postgres) y Mongo (NoSQL). Sincronización exitosa entre ambos motores. | 20 |
| **Configuración de Perfiles** | Implementación correcta de perfiles diferenciando la tecnología de BD (H2 en dev vs Postgres en prod). | 20 |
| **Modelado y Normalización** | Diseño del esquema SQL en 3ra Forma Normal. Uso de tipos de datos adecuados y constraints de integridad. | 15 |
| **JPA & Performance** | Uso de relaciones `@OneToMany` / `@ManyToOne` con estrategia `LAZY` para evitar el problema de N+1 consultas. | 15 |
| **Migraciones con Flyway** | Historial de scripts SQL coherente y numerado que permita la reconstrucción total de la base de datos. | 10 |
| **Mapeo Automático** | Uso de MapStruct para desacoplar Entidades de DTOs, manteniendo la arquitectura limpia. | 10 |
| **Transaccionalidad** | Aplicación de `@Transactional` para asegurar la consistencia eventual entre motores de base de datos. | 10 |

## Niveles de Desempeño

- **Senior (90-100)**:
  - Logra una integración impecable de los dos motores de base de datos.
  - Implementa el rollback transaccional de forma efectiva (si falla Mongo, Postgres no guarda cambios).
  - Maneja perfiles de configuración de forma experta, utilizando variables de entorno para datos sensibles.
  - El código es modular, utiliza `MapStruct` correctamente y no presenta consultas N+1 gracias al uso de `LAZY loading` y `JOIN FETCH` donde es necesario.
  - Documentación de Flyway completa y funcional para ambos entornos.
- **Semi-Senior (75-89)**:
  - La aplicación funciona y persiste datos en ambos motores.
  - Implementa perfiles pero quizás con redundancia en las propiedades o falta de uso de variables de entorno en producción.
  - Las relaciones JPA están bien mapeadas, aunque podría faltar optimización en la carga de datos (`algunos EAGER`).
  - El manejo de excepciones es centralizado pero no cubre todos los escenarios de fallo de red de las bases de datos.
- **Junior (60-74)**:
  - Consigue conectar las bases de datos, pero la lógica de persistencia está mezclada en los servicios sin una clara distinción de responsabilidades.
  - No utiliza MapStruct (mapeo manual) y presenta algunas debilidades en el modelo relacional (falta de constraints UNIQUE o índices).
  - El sistema de perfiles es básico y no diferencia claramente las tecnologías entre entornos.
- **Insuficiente (<60)**:
  - La aplicación no logra conectarse a PostgreSQL o MongoDB.
  - No hay uso de JPA o se ignoran los principios de relaciones entre tablas.
  - El código no sigue la arquitectura de capas propuesta.

## Escenarios de Error y Casos de Prueba (Nivel Senior)

Para alcanzar la nota máxima, el sistema debe ser capaz de manejar los siguientes escenarios:

- **Aislamiento de Entorno**: Demostrar que los datos de la base de datos H2 (dev) no existen cuando se cambia al perfil de Postgres (prod).
- **Rollback Transaccional**: Demostrar que un fallo en la conexión con MongoDB impide que la cita se guarde erróneamente en la base de datos relacional.
- **Consola H2**: Mostrar el acceso a la consola de H2 únicamente cuando el perfil de desarrollo está activo.

## Entregables

1. **Repositorio de GitHub**: Código fuente completo con los archivos `.properties` y la carpeta `db/migration`.
2. **Scripts SQL**: Archivos Flyway debidamente validados.
3. **Captura de Pantalla / Evidencia**: Imagen de los logs de arranque indicando: `The following 1 profile is active: "dev"`.
4. **Video de Sustentación (Max 5 min)**:
    - Explicación de la diferencia entre `application-dev.properties` y `application-prod.properties`.
    - Demostración de cambio de perfil mediante argumentos de la JVM.
    - Muestra de cómo los datos se persisten en H2 (memoria) y cómo se pierden tras reiniciar en ese perfil, a diferencia de Postgres.
