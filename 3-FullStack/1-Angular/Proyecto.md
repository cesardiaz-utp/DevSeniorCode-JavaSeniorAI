# Unidad 1: MediConnect Web (Lite)

## Escenario de Negocio: La Nueva Frontera de MediConnect

Tras haber consolidado un backend empresarial robusto en las unidades anteriores, la red hospitalaria **MediConnect** se enfrenta a su desafío más visible: la experiencia del usuario final. En esta Unidad, construiremos la base del ecosistema frontend utilizando **Angular 21+**, posicionando a la institución como líder en salud digital.

El objetivo central es construir una Single Page Application (SPA) de alto rendimiento que priorice la accesibilidad y velocidad de respuesta. Aprovecharemos las últimas innovaciones de la plataforma,  especialmente la **arquitectura zoneless**, que elimina la dependencia de `zone.js` para lograr aplicaciones mas ligeras y predecibles. MediConnect Web (Lite) servirá como el portal principal donde los pacientes podrán descubrir especialistas de clase mundial y los doctores podrán gestionar su presencia digital de manera eficiente.

## Requerimientos Funcionales

### A. Catálogo Maestro-Detalle de Doctores

- **Listado Dinámico de Especialistas**: La pantalla principal (`DoctorListComponent`) debe recuperar una colección de datos desde un servicio y renderizar tarjetas interactivas. Se debe utilizar exclusivamente el nuevo flujo de control `@for` con la cláusula `track` obligatoria. Esto no solo mejora el rendimiento del DOM, sino que permite animaciones mas fluidas al reordenar o filtrar elementos.
  - `Implicación`: Al ser una app zoneless, el programador debe asegurar que el estado se gestione mediante señales para que Angular sepa exactamente qué parte de la vista actualizar sin ciclos de detección globales.
- **Filtros Reactivos en Tiempo Real**: Implementar una barra de búsqueda inteligente. Al escribir, la lista de doctores debe filtrarse instantáneamente. Esto se logrará mediante una señal de búsqueda (`searchInput = signal('')`) y una señal computada (`filteredDoctors = computed(...)`) que que derive los datos basándose en el termino ingresado.
  - _Ejemplo_: Si el usuario busca "Cardio", la interfaz debe omitir a los pediatras sin necesidad de disparar peticiones HTTP adicionales si los datos residen en memoria.
- **Navegación al Detalle**: Al seleccionar un doctor, el sistema debe realizar una transición hacia la vista `DoctorDetailComponent`. Utilizaremos el enrutamiento basado en señales para capturar el ID del doctor y mostrar su perfil completo, incluyendo años de experiencia y horarios disponibles.

### B. Registro de Profesionales

- **Formulario de Alta de Doctores**: Crear la interfaz `DoctorRegisterComponent`. El formulario debe implementar `ReactiveFormsModule` con tipado estricto (**Strongly Typed Forms**), lo que garantiza que los errores de desarrollo se detecten en tiempo de compilación y no en tiempo de ejecución.
  - **Validación de Identidad**: Implementar validadores para nombre, apellido y numero de colegiado (obligatorios).
  - **Validación de Contacto**: El correo electrónico debe seguir el estándar RFC 5322 y el teléfono debe cumplir un patron numérico específico.
  - **Integridad de Datos**: Uso de `select` o `radio buttons` para especialidades predefinidas (ej. Oncología, Neurología, Medicina General).
- **Gestión de Errores Visuales**: Utilizar el flujo de control `@if` para gestionar la visibilidad de los mensajes de error. Estos solo deben aparecer cuando el campo es inválido Y ha sido tocado por el usuario (`touched`), evitando el "ruido visual" mientras el usuario apenas comienza a escribir..

### C. Gestión de Estado Global y Notificaciones

- **Servicio de Notificaciones Híbrido**: Crear un `NotificationService` que permita comparar paradigmas. Se implementará un sistema donde el componente `HeaderComponent` se suscriba a una señal de mensajes.
  - _Modernización_: A diferencia de RxJS, las Signals en el header permitirán que la burbuja de notificación se actualice de forma aislada, sin disparar verificaciones en toda la página de listado de doctores.

## Especificaciones Técnicas

### 1. Componentes de Nueva Generación (Standalone por Defecto)

- **Estructura Simplificada**: En Angular 21+, los componentes son **standalone por defecto**. Ya no se requiere declarar `standalone: true`. Esto reduce el código repetitivo y permite que cada componente gestione sus propias dependencias a través del array `imports`.
- **Comunicación Basada en Señales**:
  - **Inputs Reactivos**: Reemplazar el decorador legacy `@Input()` por la función `input()` o `input.required()`. Esto permite que el componente hijo trate a sus propiedades como señales, facilitando la composición lógica mediante `computed`.
  - **Outputs Reactivos**: Utilizar la función `output()` para la emisión de eventos (ej. `onDoctorSelected = output<Doctor>()`), lo que ofrece una sintaxis más limpia y un mejor tipado que el antiguo `EventEmitter`.
- **Inyección de Dependencias**: Priorizar el uso de la función `inject()` dentro del cuerpo del componente en lugar de la inyección por constructor. Esto facilita la reutilización de lógica mediante funciones de utilidad y mejora la legibilidad.

### 2. Reactividad Nativa y Optimización

- **Signals vs Zone.js**: La aplicación se configurará como `provideExperimentalZonelessChangeDetection()`. Esto significa que Angular solo actualizará la UI cuando una señal cambie o se emita un evento, reduciendo drásticamente la carga sobre el procesador, especialmente en dispositivos móviles.
- **Efectos Secundarios (`effect`)**: Implementar un `effect` para persistir las preferencias del usuario. Por ejemplo, si el usuario cambia al "Modo Oscuro", el efecto debe guardar automáticamente esta configuración en el `localStorage` sin necesidad de intervención manual en cada método de la interfaz.

### 3. Layout y Diseño (Modern CSS)

- **Arquitectura de Estilos**: Se prohíbe el uso de frameworks pesados (Bootstrap/Material) para evaluar el dominio de CSS puro.
  - **CSS Grid**: Para el layout principal y la cuadrícula de doctores.
  - **Flexbox**: Para componentes de menor escala como la barra de navegación y las tarjetas de información.
  - **Variables CSS**: Definir una paleta de colores institucional (`--mediconnect-primary`, `--mediconnect-accent`) para facilitar cambios temáticos globales.

## Estructura del Proyecto

```plain
src/app/
├── core/                # Singleton Services (Auth, Notifications)
│   └── services/        # Lógica de negocio pura
├── shared/              # Componentes UI reutilizables (Card, Loader, Nav)
├── features/            # Vistas principales de la aplicación
│   ├── doctors/         # Lista y detalle de especialistas
│   └── register/        # Formulario de alta con lógica de validación
├── models/              # Interfaces y tipos de TypeScript
├── app.routes.ts        # Rutas con Lazy Loading (loadComponent)
└── app.config.ts        # Configuración de Zoneless y Providers
```

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Arquitectura Angular 21** | Correcta implementación de componentes (sin standalone: true), flujo `@if`/`@for` y configuración Zoneless. | 25 |
| **Dominio de Signals** | Uso avanzado de `signal`, `computed` y `input()` para manejar el estado y la comunicación. | 25 |
| **Formularios y Tipado Estricto** | Implementación de `Typed Forms` con validaciones personalizadas y feedback visual asíncrono. | 20 |
| **Servicios e Inyección de Dependencias** | Uso consistente de **inject()** y servicios compartidos para la persistencia de datos. | 15 |
| **Diseño y UX** | Calidad del CSS (Grid/Flexbox), diseño responsive y manejo de estados de carga/error. | 15 |

## Entregables Obligatorios

1. **Repositorio de Código**: Debe incluir un historial de commits claro que demuestre la evolución del proyecto.
2. **Documentación Técnica**: Un archivo `README.md` que explique cómo se implementó la arquitectura zoneless y los beneficios observados en el rendimiento.
3. **Demo en Video**: Máximo 5 minutos mostrando:
    - Filtrado instantáneo mediante Signals.
    - Validación robusta del formulario de registro.
    - Navegación fluida entre componentes.

>**Nota**: El cumplimiento de los estándares de código limpio (Clean Code) y el tipado estricto es fundamental para alcanzar el nivel "Senior" en este proyecto.
