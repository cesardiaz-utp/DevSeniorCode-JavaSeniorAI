# Proyecto Integrador Módulo 4: "MediCare Cloud-Native & DevOps"

## Descripción del Escenario

El sistema "MediCare" ha sido un éxito en las pruebas locales. Ahora, la junta directiva ha aprobado el presupuesto para llevarlo a producción. Sin embargo, el equipo de operaciones exige que el software cumpla con los estándares de la industria moderna: debe ser desplegable en cualquier nube (agnóstico), debe auto-testearse antes de cualquier cambio, y debe ser capaz de escalar si la carga de usuarios aumenta.

Como Arquitecto de Software y Líder Técnico, tu misión es tomar el código existente (Backend + Frontend) y construir la "Fábrica de Software" (Pipeline) y la "Infraestructura" (Docker) necesarias para soportar el negocio.

## Objetivos de Aprendizaje Evaluados

Este proyecto valida la competencia del estudiante en el perfil Senior:

1. **Containerización**: Capacidad de empaquetar aplicaciones Java y JS en imágenes Docker optimizadas y seguras.
2. **Automatización (CI/CD)**: Diseño de flujos de trabajo en GitHub Actions para garantizar la calidad continua.
3. **Arquitectura**: Refactorización hacia patrones desacoplados (Hexagonal/Eventos) y preparación para microservicios.
4. **Observabilidad**: Implementación de trazas y métricas para entornos productivos.

## Visión General del Sistema

El problema ya no es "¿Cómo guardo un paciente en la base de datos?", sino "¿Cómo garantizo que el sistema esté disponible 24/7 y que los nuevos cambios no rompan lo existente?".

Actualmente, el despliegue es manual (copiar archivos JAR), lo cual es lento y riesgoso. La base de datos corre en la máquina del desarrollador, lo cual es inaceptable.

La nueva arquitectura propone:

1. **Entornos Efímeros**: Cualquier desarrollador debe poder levantar todo el sistema (BD, Back, Front) con un solo comando (`docker-compose up`).
2. **Calidad Automatizada**: Nadie puede fusionar código a la rama principal (main) sin que pasen los tests y el análisis de calidad estática.
3. **Desacoplamiento**: El sistema empezará a emitir eventos de dominio (ej. `AppointmentBooked`) para permitir que futuros sistemas (como Facturación) se integren sin tocar el código núcleo.

## Requerimientos Funcionales (Operativos)

### 1. Infraestructura como Código (Docker)

- **Backend**: Crear un `Dockerfile` para la API. Debe usar una imagen base ligera (ej. `eclipse-temurin:25-jre-alpine`).
- **Frontend**: Crear un `Dockerfile` para Angular que compile el proyecto y lo sirva usando Nginx configurado como proxy inverso.
- **Orquestación**: Un archivo docker-compose.yml que levante:
  - PostgreSQL (con volumen persistente).
  - MongoDB.
  - MediCare API (esperando a que las BDs estén listas).
  - MediCare Web (expuesto en puerto 80).

### 2. Pipeline de CI/CD (GitHub Actions)

- Crear un workflow `.github/workflows/pipeline.yml` que se active en cada `push` a `main` o `pull_request`.
- **Jobs requeridos**:
  1. **Build & Test**: Compilar Backend y Frontend, ejecutar tests unitarios (JUnit/Vitest).
  2. **Docker Build**: Si los tests pasan, construir las imágenes Docker.
  3. _(Opcional/Simulado)_ **Deploy**: Simular un paso de despliegue (ej. imprimir "Deploying to Production...").

### 3. Refactorización Arquitectónica (Eventos)

- Introducir un pequeño bus de eventos (puede ser interno con `ApplicationEventPublisher` de Spring o externo con Kafka si el estudiante es avanzado).
- **Caso de Uso**: Cuando se agenda una cita, el servicio no debe enviar el email/notificación directamente. Debe publicar un evento `AppointmentCreatedEvent`. Un componente separado `NotificationListener` debe escuchar ese evento y loguear "Enviando email...". Esto demuestra desacoplamiento.

### 4. Observabilidad

- Configurar **OpenTelemetry** (o Sleuth/Micrometer) para que los logs incluyan un `traceId` y `spanId`.
- Demostrar que al hacer una petición desde el Frontend, se puede rastrear ese ID único en los logs del Backend.

## Requerimientos Técnicos

### 1. Estándares de Contenedores

- Las imágenes no deben correr como usuario `root` (seguridad).
- Los secretos (contraseñas de BD) no deben estar hardcodeados en el `Dockerfile`, deben inyectarse como variables de entorno.

### 2. Calidad de Código

- Integrar **SonarCloud** (versión gratuita) o un linter estricto en el Pipeline. El código no debe tener "Code Smells" críticos.

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de 100 puntos:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Dockerización** | Dockerfiles optimizados (multi-stage), uso correcto de capas, docker-compose funcional que levanta todo el stack con un comando. | 25 |
| **Pipeline CI/CD** | Workflow de GitHub Actions correcto. Ejecuta tests, reporta fallos y construye artefactos solo si hay éxito. | 25 |
| **Arquitectura (Eventos)** | Implementación correcta del patrón Observer/Pub-Sub para desacoplar la lógica de notificaciones. | 20 |
| **Observabilidad** | Logs estructurados con Trace IDs visibles. Configuración correcta de Actuator en entorno containerizado. | 15 |
| **Calidad & Docs** | README profesional (instrucciones de despliegue), código limpio verificado por herramientas estáticas. | 15 |

## Niveles de Desempeño

- **Senior (90-100)**: Pipeline completo con SonarQube, Dockerfiles seguros (non-root), arquitectura desacoplada y documentación excelente.
- **Semi-Senior (75-89)**: Docker funciona, el pipeline corre tests, pero quizás las imágenes son pesadas o falta trazabilidad.
- **Junior (60-74)**: Logra levantar los contenedores pero con configuración manual/frágil. El pipeline es básico.
- **Insuficiente (<60)**: No hay contenedorización funcional o el pipeline no ejecuta tests.

## Entregables

- Repositorio GitHub con código y carpeta `.github/workflows`.
- Captura de pantalla de una ejecución exitosa ("Verde") en la pestaña Actions de GitHub.
- Comando exacto para levantar el entorno (`docker-compose up`).
- Breve video mostrando los logs con Trace IDs mientras se usa la app.
