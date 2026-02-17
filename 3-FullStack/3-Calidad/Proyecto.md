# Proyecto Módulo 2: MediConnect Quality & Testing Ecosystem

## Escenario de Negocio

Tras el despliegue exitoso de la infraestructura, **MediConnect** ha experimentado un crecimiento exponencial en su base de usuarios y volumen de datos. Sin embargo, con esta rápida expansión surge un riesgo sistémico crítico: la **regresión operativa**.

En un entorno clínico, un cambio menor en el motor de búsqueda no puede comprometer la integridad del módulo de citas, y una actualización en los protocolos de seguridad no puede bloquear el acceso de los médicos en medio de una consulta de telemedicina. La confianza de los pacientes y la viabilidad legal de la plataforma ahora dependen exclusivamente de una **calidad demostrable y automatizada**.

En este proyecto de cierre, asumirás el rol estratégico de **SDET (Software Development Engineer in Test) / Quality Lead**. Tu misión trasciende la simple escritura de código; debes blindar el ecosistema completo mediante una estrategia de _"Defensa en Profundidad"_. Esto implica no solo reaccionar ante errores, sino prevenirlos mediante la implementación de una cultura de **TDD (Test Driven Development)** para cada nueva funcionalidad. Configurarás entornos de prueba idénticos a producción utilizando **Testcontainers** y orquestarás flujos **E2E con Cypress** para simular el comportamiento humano real. El objetivo final es establecer un estándar de ingeniería donde alcancemos un 80% de cobertura de código, garantizando que MediConnect sea una plataforma resiliente, auditable e indestructible.

## Requerimientos Funcionales y de Calidad

### A. Backend: Blindaje de Lógica, Persistencia y Seguridad

- **TDD en Reglas de Negocio Complejas**: Deberás implementar un nuevo "Calculador de Prioridad de Citas Médicas" (que asigna urgencia según edad, síntomas y disponibilidad).
  - **Ciclo Red-Green-Refactor**: Es obligatorio que los commits demuestren que el test falló inicialmente antes de existir la lógica. Esto garantiza que el diseño del código esté orientado a la necesidad del usuario y no a la conveniencia del programador.
- **Aislamiento Total y Dobles de Prueba con Mockito**: Escribir pruebas unitarias exhaustivas para el `AppointmentService`.
  - **Profundidad**: No basta con probar el éxito. Debes simular comportamientos de red fallidos, latencias en repositorios y colisiones de horarios, validando que el sistema lance y maneje correctamente la `AppointmentConflictException` sin exponer trazas de error al usuario final.
- **Pruebas de Integración con Infraestructura Inmutable**: Configurar un entorno de pruebas que utilice **Testcontainers**.
  - **Implicación**: A diferencia de las bases de datos H2 (que suelen ocultar errores de sintaxis específicos de PostgreSQL), Testcontainers levantará una instancia real de la base de datos en un contenedor Docker efímero. Esto asegura que tus consultas JPA, triggers y restricciones de base de datos se comporten exactamente igual que en el servidor de producción.

### B. Frontend: Estabilidad Reactiva y UI Predictiva

- **Unit Testing de Lógica de Estado (Zoneless)**: Configurar **Vitest** para validar la integridad del `MedicalStateService`. Dado que operamos en una arquitectura sin Zone.js, es vital asegurar que las transformaciones de datos y los efectos secundarios de los Signals (`computed`, `effect`) se disparen en el orden correcto.
- **Component Testing & Comportamiento de Interfaz**: Testear los componentes `LoginComponent` y `DoctorSearchComponent` bajo condiciones extremas.
  - **Detalle**: Verificar que los estados de carga (spinners) aparezcan instantáneamente, que los botones de acción se deshabiliten durante las peticiones asíncronas y que los mensajes de error sean accesibles y claros. Debes simular entradas de usuario inválidas y verificar la sanitización de los inputs.
- **Testing de Interceptores y Flujo de Datos**: Utilizar `HttpTestingController` para auditar el `authInterceptor`. El test debe confirmar que cada petición saliente a la API incluye el header `Authorization: Bearer <token>` y que, ante un error 401, el sistema redirige automáticamente al usuario al login.

### C. Automatización E2E: El Usuario como Auditor

- **Flujos Críticos de Negocio (Happy & Sad Paths)**: Desarrollar una suite de Cypress que automatice el "viaje del paciente".
  1. **Autenticación**: Proceso de login con validación de almacenamiento de token.
  2. **Exploración**: Búsqueda filtrada de médicos y selección de perfil.
  3. **Transacción**: Agendamiento de cita con validación de feedback visual (Toasts).
  4. **Sincronización**: Verificar que tras la cita, el Dashboard se actualiza sin necesidad de recargar la página.
- **Simulación de Condiciones de Red**: Implementar tests que intercepten llamadas (`cy.intercept`) para simular respuestas lentas o errores de servidor (500), verificando que la aplicación no se bloquee y ofrezca una vía de recuperación al usuario.

### D. Gobernanza de Métricas y Entrega Continua

- **Análisis de Cobertura de Código (Code Coverage)**: Configurar las herramientas de reporte para generar archivos LCOV/HTML. Se exige un mínimo del 80% de cobertura en las capas de lógica (Servicios en Backend y Logic Services en Frontend).
  - **Consecuencia**: El estudiante deberá justificar cualquier "zona muerta" del código que no esté bajo cobertura, fomentando la eliminación de código muerto.
- **Testing de Seguridad y Roles**: Implementar pruebas que utilicen `@WithMockUser` con diferentes roles (`USER`, `ADMIN`). Se debe demostrar que un usuario con rol de paciente recibe un error 403 al intentar acceder a endpoints de administración de facultativos.

## Especificaciones Técnicas

- **Ecosistema de Backend**:
  - **JUnit 5 & AssertJ**: Para aserciones descriptivas y legibles.
  - **Mockito**: Para la creación de Stubs y Mocks de alta fidelidad.
  - **Testcontainers**: Para orquestación de Docker containers dentro del ciclo de vida de los tests.
  - **Spring Security Test**: Para simulación de contextos de seguridad y tokens JWT.
- **Ecosistema de Frontend**:
  - **Vitest**: Elegido por su velocidad superior y compatibilidad nativa con Vite y Angular Moderno.
  - **Angular Testing Library**: Para realizar pruebas de componentes desde la perspectiva del DOM, evitando depender de la implementación interna.
  - **Cypress**: Para el testing de integración final y validación de flujos visuales.
- **Gobernanza de IA**: Las reglas en `.cursorrules` deben configurarse para que Cursor AI actúe como un revisor de calidad, sugiriendo tests unitarios automáticamente al detectar la creación de un nuevo método de servicio.

## Estructura de Repositorios

### Repositorio A: `mediconnect-backend`

```plain
/src/test/java/com/mediconnect/
├── unit/services/           # Mocks con Mockito y pruebas de lógica aislada
├── integration/persistence/ # Tests contra PostgreSQL real (Testcontainers)
├── integration/api/         # MockMvc validando contratos REST y seguridad
├── resources/               # Configuración específica para entornos de test
└── .cursorrules             # Prompt: "Required test file for every new service class"
```

### Repositorio B: `mediconnect-frontend`

```plain
/src/app/
├── **/*.spec.ts             # Tests de lógica de Signals y componentes (Vitest)
├── cypress/e2e/             # Scripts de automatización de flujos de usuario
├── coverage/                # Reportes de auditoría de cobertura generados
└── .cursorrules             # Prompt: "Ensure component tests cover accessibility and loading states"
```

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio de Excelencia (Nivel Senior) | Puntos |
| --- | --- | --- |
| **Backend Testing Mastery** | Implementación impecable de Mockito para aislamiento y Testcontainers para integración de base de datos. | 30 |
| **Frontend Reactive Quality** | Cobertura robusta de lógica de Signals con Vitest y validación de interceptores funcionales. | 25 |
| **E2E User Experience** | Suite de Cypress que cubre el 100% de los flujos críticos con aserciones de red y UI. | 20 |
| **Cultura de Ingeniería (TDD)** | Historial de Git que evidencia el flujo Red-Green-Refactor y diseño orientado a tests. | 15 |
| **Métricas y Documentación** | Reporte de Coverage >80% y README con estrategia técnica de testing detallada. | 10 |

## Entregables y Certificación de Calidad

1. **Repositorios en Verde**: Ambos proyectos deben tener sus pipelines de test pasando sin fallos (`all green`).
2. **Dashboard de Cobertura**: Captura de pantalla de los reportes de Vitest y JaCoCo demostrando el cumplimiento de la métrica del 80%.
3. **Engineering README**: Una sección técnica detallando cómo se resolvió el testing de asincronía en WebSockets y cómo se configuró el entorno efímero de Docker.
4. **Video de Auditoría (max 5 min)**: Grabación de pantalla ejecutando `mvn verify`, `npm test` y una ejecución completa de Cypress en modo "headed" para observar la interacción automatizada.

> **Nota de Ingeniería**: En esta unidad, tu éxito no se define por la cantidad de código nuevo que escribas, sino por la resiliencia del código que ya existe. Un buen ingeniero escribe funcionalidades; un ingeniero senior construye sistemas que pueden evolucionar sin romperse.
