# Proyecto Integrador Módulo 3: "MediCare Web Portal (Full Stack)"

## Descripción del Escenario

La "MediCare API" (construida en el Módulo 2) es funcional pero invisible para los usuarios finales. El objetivo de este proyecto es construir una **Single Page Application (SPA)** moderna, intuitiva y segura que permita a pacientes y médicos interactuar con el sistema. La aplicación debe ser responsive (móvil y escritorio), manejar la complejidad del estado asíncrono y ofrecer características de tiempo real.

## Objetivos de Aprendizaje Evaluados

1. **Dominio de Angular Moderno**: Uso exclusivo de **Standalone Components**, **Signals** para reactividad y el nuevo **Control Flow** (`@if`, `@for`).
2. **Integración Full Stack Avanzada**: Consumo de APIs REST, manejo de JWT, **Subida de Archivos** y comunicación en **Tiempo Real (WebSockets)**.
3. **Diseño Moderno**: Implementación de interfaces responsivas utilizando **Tailwind CSS**.
4. **Calidad Totalmente Automatizada**: Estrategia de testing híbrida con **Vitest** (Unitario) y **Cypress** (E2E).

## Visión General del Sistema

El ecosistema "MediCare" ha crecido. Ya contamos con un backend robusto capaz de gestionar datos complejos, pero carecemos de una "cara" visible para nuestros usuarios. Actualmente, un paciente no puede agendar una cita sin usar herramientas técnicas como Postman, lo cual es inviable en el mundo real.

Este proyecto se centra en cerrar esa brecha, construyendo el **Portal Web MediCare**. No es solo una capa visual; es una aplicación cliente inteligente que debe gestionar el estado de la sesión, la seguridad en el navegador y la sincronización de datos.

El sistema debe resolver dos flujos de experiencia de usuario críticos:

1. **Experiencia del Paciente (Autogestión)**: El objetivo es eliminar la fricción de los procesos manuales. El paciente debe poder buscar médicos por especialidad, ver su disponibilidad y reservar un espacio con confirmación inmediata, además de poder gestionar su propia identidad visual subiendo una foto de perfil.
2. **Experiencia del Médico (Gestión en Tiempo Real)**: Proveer al médico de un tablero de control vivo. La característica diferenciadora es la **inmediatez**: cuando un paciente agenda una cita, el médico debe recibir una alerta visual al instante (sin refrescar la página), permitiéndole reaccionar dinámicamente a su agenda del día.

## Requerimientos Funcionales (Frontend)

### 1. Portal Público y Autenticación

- **Landing Page**: Página de inicio atractiva con información de la clínica.
- **Auth**: Formularios de Login y Registro con validaciones reactivas.
  - _Feedback_: Si el login falla, mostrar alertas visuales usando componentes de Tailwind.
- **Sesión**: Persistencia del token y estado del usuario (usando `localStorage` abstraído en un servicio).

### 2. Portal del Paciente (Rol PATIENT)

- **Perfil** y **Avatar**: El paciente debe poder subir una foto de perfil. La imagen debe previsualizarse antes de enviarse al servidor.
- **Buscar Médicos**: Buscador en tiempo real (Type-ahead con RxJS `switchMap`) filtrando por nombre o especialidad.
- **Agendar Cita**:
  - Seleccionar Médico -> Seleccionar Fecha -> Confirmar.
  - **Notificación Real-Time**: Al confirmar la cita, debe aparecer un "Toast" de confirmación impulsado por un evento WebSocket recibido del servidor.

### 3. Portal del Médico (Rol DOCTOR)

- **Mi Agenda**: Vista de las citas asignadas.
- **Notificaciones en Vivo**: Si un paciente agenda una cita con el médico mientras este tiene la sesión abierta, debe aparecer una notificación emergente ("Nueva cita agendada por [Nombre Paciente]") sin necesidad de recargar la página (WebSockets).

## Requerimientos Técnicos

### 1. Arquitectura Angular

- **Standalone**: Prohibido usar `NgModules`. Configuración en `app.config.ts`.
- **Signals**: Uso de `signal()`, `computed()` y `effect()` para el manejo del estado local y global.
- **CSS Framework**: Uso estricto de **Tailwind CSS**. Diseño _Mobile First_.

### 2. Integración y Seguridad

- **Interceptors Funcionales**:
  - `authInterceptor`: Inyección del Token JWT.
  - `errorInterceptor`: Manejo global de errores HTTP.
- **WebSockets**: Implementar un servicio (`WebSocketService`) que se conecte al broker del backend (STOMP) y exponga los eventos como Signals o Observables para la UI.
- **Guards Funcionales**: Proteger las rutas `/portal` y `/admin` según el rol decodificado del token.

### 3. Calidad y Testing

- **Unit Testing (Vitest)**:
  - Crear tests unitarios para los **Servicios** (ej. `AuthService`, `MedicalService`) verificando la lógica de negocio y transformación de datos.
  - Verificar que los Signals actualizan su valor correctamente.
- **E2E Testing (Cypress)**:
  - Implementar un flujo crítico completo: _Login -> Buscar Médico -> Agendar Cita_.

## Rúbrica de Evaluación (100 Puntos)

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Angular Moderno** | Arquitectura 100% Standalone, uso de Signals para estado y Control Flow Syntax (`@if`, `@for`). | 20 |
| **Integración Avanzada** | Subida de imágenes (Multipart) y Notificaciones en Tiempo Real (WebSockets) funcionando correctamente. | 25 |
| **UX & Tailwind** | Diseño responsivo, limpio y uso correcto de clases de utilidad Tailwind. Feedback visual de carga y errores. | 15 |
| **Seguridad & API** | Interceptores funcionales, Guards de rutas y manejo correcto de JWT/Storage. | 20 |
| **Testing (Vitest/Cypress)** | Suite de tests unitarios con Vitest (lógica) y al menos un test E2E crítico con Cypress. | 20 |

## Entregables

1. Repositorio Git del Frontend.
2. Video demo mostrando:
    - Subida de imagen de perfil.
    - La notificación en tiempo real (abriendo dos navegadores, uno como paciente y otro como médico).
3. Reporte de cobertura de tests (Vitest) y captura de Cypress en verde.
