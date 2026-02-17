# Proyecto de Grado: Senior Full-Stack Capstone & Cloud Defense

## Escenario del Proyecto

Tras 11 unidades de aprendizaje intenso, has llegado a la etapa final de tu formación. El **Proyecto de Grado** no es una tarea guiada; es la demostración definitiva de tu capacidad para concebir, diseñar, desarrollar, asegurar y desplegar una solución de software de grado empresarial de forma autónoma.

Ahora presentarás y defenderás tu proyecto ante un comité técnico. El sistema debe ser una aplicación **Full-Stack original** (basada en MediConnect o una idea propia aprobada) que resuelva un problema real, esté blindada con pruebas automatizadas y viva en una infraestructura de nube pública mediante un pipeline de CI/CD profesional. Se evaluará tu capacidad para justificar decisiones arquitectónicas, gestionar la deuda técnica y utilizar la **IA (Cursor)** como un multiplicador de productividad y calidad.

## Requerimientos Maestros

### A. Arquitectura y Stack Tecnológico

- **Backend**: Microservicio robusto en **Spring Boot 4 / Java 25** con persistencia relacional (PostgreSQL) y seguridad JWT.
- **Frontend**: SPA reactiva en **Angular 21+ (Zoneless)** con gestión de estado mediante Signals y diseño responsivo con Tailwind CSS.
- **IA Governance**: Uso estratégico de `.cursorrules` para mantener la coherencia del estilo de código y patrones de diseño en ambos repositorios.

### B. Funcionalidades de Alto Valor

- **Integración Real-Time**: Implementación de WebSockets o Server-Sent Events para interactividad inmediata.
- **Gestión de Medios**: Procesamiento de imágenes/archivos en el cliente (Canvas/WebP) y almacenamiento seguro.
- **Resiliencia**: Manejo de estados offline y sincronización de datos diferida.

### C. Ecosistema de Calidad (

- **Cobertura Mínima**: 80% de cobertura en lógica de negocio demostrable mediante reportes de JaCoCo (Backend) y Vitest (Frontend).
- **Pirámide de Pruebas**: Inclusión de pruebas unitarias, de integración (con Testcontainers) y al menos 3 flujos críticos automatizados con Cypress (E2E).

### D. Infraestructura y DevOps

- **Contenerización**: Dockerfiles optimizados (Multi-stage) y orquestación con Docker Compose.
- **Automatización**: Pipelines de GitHub Actions que ejecuten el ciclo CI (Build/Test/Lint) y CD (Push a Registro y Despliegue Cloud).
- **Disponibilidad**: La aplicación debe estar desplegada en una URL pública funcional (Render, Railway, AWS, etc.).

## Estructura de la Defensa (Presentación de 15 min)

La defensa se dividirá en cuatro bloques fundamentales:

1. **Arquitectura y Diseño (3 min)**: Justificación del modelo de datos, diagramas de flujo y elección de patrones de diseño.
2. **Demo en Vivo (5 min)**: Recorrido por las funcionalidades clave desde la URL de producción, demostrando la fluidez de la UI y la respuesta del backend.
3. **Deep Dive Técnico (4 min)**: Explicación de un desafío complejo resuelto (ej. configuración de WebSockets, un test de integración difícil o la lógica de un interceptor).
4. **Q&A y Code Review (3 min)**: El comité seleccionará una parte aleatoria del código para que expliques su funcionamiento y lógica de testing.

## Rúbrica de Evaluación Final

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio de Evaluación | Puntos |
| Dominio Full-Stack | Integración impecable entre Angular 21 y Spring Boot 4 con manejo de seguridad JWT. | 25 |
| Calidad y Testing | Evidencia de TDD, cobertura >80% y estabilidad de los tests E2E. | 25 |
| DevOps y Cloud | Automatización total del despliegue y correcta configuración de contenedores. | 20 |
| Uso de IA y Limpieza | Calidad del código, uso de .cursorrules y reporte de "Prompt Engineering". | 15 |
| Defensa y Oratoria | Capacidad de comunicación técnica y resolución de preguntas complejas.
| 15 |

## Entregables para la Certificación

1. **Repositorios Finales**: Documentados con READMEs profesionales (Setup, Arquitectura, API Docs).
2. **Reportes de Calidad**: Dashboards de SonarQube/SonarCloud y Cobertura.
3. **Bitácora de IA**: Documento que resuma cómo Cursor AI ayudó a acelerar el desarrollo y qué reglas se definieron.
4. **Enlace de Producción**: URL activa para pruebas por parte del jurado.

> **Mensaje de Cierre**: Este no es solo el fin de un curso, es el inicio de tu carrera como Senior Full-Stack Engineer. Tu proyecto es tu carta de presentación ante el mundo. ¡Haz que sea extraordinario!
