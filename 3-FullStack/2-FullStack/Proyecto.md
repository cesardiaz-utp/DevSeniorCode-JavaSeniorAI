# Unidad 2: MediConnect Full-Stack Ecosystem

## Escenario de Negocio

Tras haber consolidado una interfaz moderna y reactiva, el ecosistema **MediConnect** se enfrenta a su reto más crítico: la coherencia de datos en tiempo real y la resiliencia sistémica. En el sector salud, la fiabilidad no es una característica opcional; es el cimiento de la confianza clínica.

En este proyecto, asumirás el rol de **Full Stack Lead**, liderando la orquestación entre dos sistemas independientes: la robusta **"MediCare API" (Backend)** y la aplicación **"MediConnect Web" (Frontend)**. El desafío principal radica en la integración de estos dos mundos, garantizando que la seguridad JWT sea innegociable, la búsqueda de especialistas esté optimizada y las notificaciones críticas ocurran de forma espontánea. Todo este proceso será potenciado por el uso avanzado de **IA (Cursor)**, aplicando reglas de gobernanza técnica específicas para cada stack.

## Requerimientos Funcionales

### A. Seguridad y Túnel de Datos

- **Gestión de Identidad de Extremo a Extremo**: Implementar el flujo completo de autenticación. El frontend solicita credenciales; el backend valida mediante `POST /api/v1/auth/login` y genera un JWT firmado.
- **Mecanismos de Intercepción de Tráfico**:
  - **AuthInterceptor (Frontend)**: Desarrollo de un interceptor funcional que inyecte automáticamente el token `Bearer` en el encabezado `Authorization`.
  - **ErrorInterceptor (Frontend)**: Captura centralizada de errores **401 Unauthorized**. Ante una falla, la app debe realizar un "clean-up" de Signals y redirigir al login.
  - **Configuración CORS (Backend)**: El backend debe configurar una política CORS restrictiva que permita únicamente el origen del repositorio de frontend seleccionado.

### B. Búsqueda Reactiva y Consumo de API

- **Buscador de Especialistas**: Barra de búsqueda global con filtrado predictivo.
  - _Implicación Técnica_: Uso obligatorio de `debounceTime(300)` y `switchMap` en Angular para garantizar que solo la respuesta de la última búsqueda activa se renderice.
- **Estrategia SWR (Stale-While-Revalidate)**: Implementar caché en el cliente mediante Signals para resultados frecuentes, permitiendo visualización instantánea mientras se valida con el servidor en segundo plano.

### C. Gestión de Perfil y Optimización Multimedia

- **Módulo de Avatar Profesional**: Actualización de identidad visual con procesamiento en el "Edge" (navegador).
  - **Compresión Dinámica (Canvas API)**: El frontend debe redimensionar y transformar las imágenes a **WebP** antes de la subida.
  - **Consecuencia**: Mejora drástica del **LCP (Largest Contentful Paint)** y reducción de costes de almacenamiento en el servidor.

### D. Notificaciones Real-Time & Colaboración

- **Infraestructura WebSocket**: Configurar una conexión persistente mediante **WebSockets (STOMP)**.
- **Sistema de Telemetría de Presencia**: Rastreador que muestre si el médico está "En Línea" basándose en la conexión activa al broker de mensajes del backend.

### E. Resiliencia Avanzada y Telemetría

- **Arquitectura Offline-First**: Servicio de sincronización en Angular que encole acciones críticas (como agendar citas) ante pérdida de red y las sincronice automáticamente al recuperar la conexión.
- **Observabilidad (Telemetría de Rendimiento)**: Dashboard en el frontend que reporte métricas de la **PerformanceObserver API**, como el tiempo de respuesta (TTFB) de la API del backend.

## Especificaciones Técnicas

- **Backend (Proyecto 1)**:
  - Spring Boot 4 + Java 25.
  - Uso de `records` para DTOs inmutables.
  - Spring Security + JWT + STOMP Messaging.
- **Frontend (Proyecto 2)**:
  - Angular 21 con **Arquitectura Zoneless** pura.
  - Inyección funcional con `inject()`.
  - Control Flow nativo (`@if`, `@for`).
- **Gobernanza de IA**: Cada proyecto debe tener su propio archivo `.cursorrules` adaptado a su lenguaje y buenas prácticas específicas.

## Estructura de Repositorios

El estudiante debe entregar **dos enlaces de repositorio distintos**:

### Repositorio A: `mediconnect-backend`

```plain
/mediconnect-backend
├── src/main/java/com/mediconnect/security/   # Configuración JWT y CORS
├── src/main/java/com/mediconnect/messaging/  # WebSocket Brokers
├── src/main/resources/                       # Scripts SQL (Flyway/Liquibase)
├── .cursorrules                              # Reglas para Java 25 y Spring Boot 4
└── README.md                                 # Documentación de la API (Swagger/OpenAPI)
```

### Repositorio B: `mediconnect-frontend`

```plain
/mediconnect-frontend
├── src/app/core/                            # Interceptores y SyncService (Offline)
├── src/app/features/                        # Slices verticales de negocio
├── src/app/shared/                          # UI Library con Tailwind y Signals
├── .cursorrules                             # Reglas para Angular 21 y Zoneless
└── README.md                                # Guía de despliegue y prompts de IA
```

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio de Excelencia | Puntos |
| --- | --- | --- |
| **Integración de Seguridad** | Implementación de JWT y configuración de CORS en backend; interceptores funcionales en frontend. | 25 |
| **Arquitectura de Signals** | Uso experto de `toSignal` y manejo de estado zoneless sin fugas de memoria. | 20 |
| **Comunicación Real-Time** | Conexión WebSocket estable y sistema de presencia online funcional. | 20 |
| **Resiliencia & Offline** | Manejo de cola de reintentos y sincronización automática tras fallo de red. | 15 |
| **Gobernanza de IA Dual** | Calidad de los archivos `.cursorrules` en ambos repositorios. | 10 |
| **Documentación & Prompts** | READMEs técnicos completos y reporte de Prompt Engineering. | 10 |

## Entregables Obligatorios

1. **Enlace Repositorio Backend**: Historial de commits con uso de Cursor AI.
2. **Enlace Repositorio Frontend**: Historial de commits con uso de Cursor AI.
3. **README.md (Insights)**: Documentación del flujo de integración y los 5 prompts más complejos.
4. **Video Demo (max 5 min)**: Demostración de login, búsqueda reactiva, modo offline y notificación real-time.

> **Nota de Ingeniería**: La separación en dos proyectos independientes simula un entorno de producción real, donde el backend y el frontend escalan y se despliegan de forma autónoma.
