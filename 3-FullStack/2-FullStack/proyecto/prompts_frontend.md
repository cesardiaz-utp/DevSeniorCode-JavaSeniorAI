# Secuencia de Prompts para el Frontend de BiblioKeep (Metodología CIFR)

> **Nota**: Se asume que el proyecto ya fue creado con Angular CLI y Tailwind CSS está configurado.

## Prompt 1: Arquitectura de Carpetas e Interfaces

El proyecto Angular con Tailwind ya está inicializado. Necesitamos establecer la estructura profesional basada en `@especificaciones_detalladas.md`. Crea la estructura de carpetas sugerida: `app/core` (modelos, servicios, interceptores), `app/shared` (componentes atómicos) y `app/features` (dashboard, library, loans). Además, genera las interfaces de TypeScript (User, Book, Loan, AuthResponse) en `app/core/models` basándote exactamente en la Sección 3 del archivo de especificaciones. Usa `export interface`. Asegúrate de que los tipos coincidan con el contrato del backend.

## Prompt 2: Layout Maestro y Navegación con Tailwind

Implementaremos el marco visual de la aplicación antes de la lógica de negocio (Sección 6). Crea un `MainLayoutComponent` (Standalone) que sirva como contenedor principal. Debe incluir:

1. Un Sidebar colapsable con enlaces de navegación (`routerLink`).
2. Un Header responsivo con el nombre del app "BiblioKeep".
3. Un área de contenido principal con un `router-outlet`.
Usa exclusivamente clases de Tailwind CSS y `Lucide Angular` para los iconos. El Sidebar debe ser amigable para móviles (tipo "drawer"). No uses librerías de componentes externas.

## Prompt 3: Gestión de Biblioteca y Búsqueda Híbrida (Signals)

Implementaremos el núcleo: la gestión de libros (Sección 5 y 6) sin seguridad aún. Crea el `BookService` para consumir `/api/books/search`. Implementa un `BookStoreService` basado en **Signals** que maneje:

1. `books`: La lista de la colección.
2. `isLoading`: Estado de carga.
3. `filteredBooks`: Un `computed` signal que filtre la lista local por `status`. Crea el componente `LibraryComponent` con una barra de búsqueda y una cuadrícula de `BookCard` usando Tailwind. Servicio, Store y Componentes de la feature `library`. Aplica **Optimistic UI**: si el usuario cambia el rating o status, actualiza el Signal localmente antes de la petición HTTP.

## Prompt 4: Préstamos, Formularios Reactivos y Dashboard

Ahora trabajaremos en las funcionalidades de préstamos y estadísticas visuales (Sección 6). Implementa la feature de préstamos usando **Reactive Forms** con validaciones. Además, crea el `DashboardComponent` que consuma `/api/stats/dashboard` y muestre:

1. Un gráfico de progreso hacia la `annualGoal` (puedes usar un gráfico circular simple con CSS/Tailwind).
2. Widgets de estadísticas con gradientes de Tailwind y bordes redondeados.

Valida que la fecha de devolución no sea anterior a la de inicio. Estiliza los widgets para que se vean modernos.

## Prompt 5: Pulido de UX, Skeletons y Modo Enfoque

Ahora haremos el refinamiento de la experiencia de usuario (Sección 6).Añade detalles de alta calidad:

1. **Skeletons**: Crea un `BookSkeletonComponent` con la animación `animate-pulse` de Tailwind.
2. **Modo Enfoque**: Implementa un estado en el layout que oculte el sidebar para lectura de detalles.
3. **Notificaciones**: Crea un servicio simple de "Toast" para confirmar acciones.

Usa `takeUntilDestroyed` de Angular para la limpieza de suscripciones.

## Prompt 6: Seguridad, JWT y Protección de Rutas (Final)

Con la app funcional, ahora cerraremos la seguridad conectando con el backend.
Implementa el `AuthService` en `core/services` para manejar login y registro. Utiliza **Signals** para gestionar `currentUser` e `isAuthenticated`. Crea un `auth.interceptor.ts` para adjuntar el token JWT de `localStorage` a todas las peticiones y un `auth.guard.ts` para proteger las rutas. Maneja errores 401 para limpiar el estado y redirigir al login. Asegúrate de que la navegación solo sea posible si el usuario está autenticado.
