# Unidad 2 - Clase 1: Inteligencia Artificial en el Desarrollo y Cursor AI

- **Duración**: 2 horas
- **Objetivo**: Configurar un entorno de desarrollo de "siguiente generación" asistido por IA, comprender el cambio de paradigma en la codificación y establecer las reglas de juego (`.cursorrules`) para garantizar la generación de código moderno, seguro, mantenible y limpio.

## Parte 1: Teoría Profunda (45 Minutos)

### 1. El Nuevo Rol del Desarrollador: De "Picapedrero" a Arquitecto

La llegada de la Inteligencia Artificial Generativa al flujo de trabajo del desarrollo de software no marca el fin de la programación, sino una **evolución en la abstracción**. La IA no está aquí para reemplazar a los desarrolladores, sino para potenciar sus capacidades y eliminar la fricción de la sintaxis repetitiva.

- **El Pasado (Albañiles del Código)**: Antes, nuestra carga cognitiva estaba saturada por la sintaxis. Pasábamos horas escribiendo _boilerplate_, configurando inyecciones de dependencias manuales, cerrando llaves y buscando en Google cómo iterar un mapa en Java. El valor se medía en líneas de código producidas.
- **El Presente (Arquitectos y Auditores)**: Ahora, nuestro foco se desplaza hacia el diseño de sistemas, la seguridad y la lógica de negocio.
  - **Arquitectura**: Definimos qué debe hacer el sistema, cómo se comunican los microservicios y qué patrones de diseño aplicar.
  - **Auditoría**: La IA escribe la implementación, pero nosotros somos los responsables finales. Debemos tener la capacidad técnica para leer el código generado, identificar vulnerabilidades y asegurar que cumple con los estándares del equipo.
- **La Regla de Oro de la IA**: _Nunca aceptes código que no entiendas o no puedas explicar_.
  - **El peligro de la Alucinación**: Los LLMs son probabilísticos, no deterministas. Pueden inventar librerías que no existen, usar métodos depreciados de Spring Boot 2 en un proyecto de Spring Boot 4, o sugerir patrones inseguros. Tu responsabilidad es validar cada sugerencia con tu conocimiento experto.

### 2. Introducción a Cursor AI: Más que un Editor

Cursor es un _fork_ directo de VS Code. Esto ofrece una ventaja táctica inmediata: la migración es indolora porque todas tus extensiones, atajos de teclado y temas favoritos funcionan nativamente. Sin embargo, la diferencia radica en que el motor de IA no es un plugin externo; está **integrado en el núcleo** del editor, lo que le permite "ver" y "entender" tu proyecto de una manera que los plugins tradicionales no pueden.

#### Las Herramientas Clave del Ecosistema Cursor

1. **Chat Lateral (`Cmd+L` / `Ctrl+L`): El Copiloto Conversacional**.
    - **Ideal para**: Preguntas exploratorias ("¿Cuál es la mejor forma de manejar errores globales en Spring Boot 4?"), dudas de arquitectura, explicación de código legado o debugging de errores complejos pegando el stack trace.
    - _Ventaja_: Mantiene el historial de la conversación, permitiendo iterar sobre una solución.
2. **Composer (`Cmd+I` / `Ctrl+I`): El Constructor Multi-archivo**.
    - Esta es la característica diferenciadora más potente. A diferencia de un chat normal que te da fragmentos para copiar y pegar, Composer tiene permisos de escritura en tu sistema de archivos.
    - _Caso de Uso Real_: "Crea un CRUD completo para la entidad `MedicalAppointment`. Genera el Entity, el Repository, el Service Interface, la implementación del Service y el REST Controller. Asegúrate de crear también los DTOs correspondientes". Composer creará o editará esos 5 archivos simultáneamente.
3. **Inline Edit (`Cmd+K` / `Ctrl+K`): La Micro-cirugía**.
    - Diseñado para iteraciones rápidas dentro de un archivo abierto sin perder el foco.
    - _Ejemplo_: Seleccionas una función y escribes: "Añade manejo de excepciones y loguea el error". La IA mostrará un "diff" (diferencia) visual que puedes aceptar o rechazar.

#### El Contexto es el Rey (RAG - Retrieval Augmented Generation)

Los LLMs (Modelos de Lenguaje) son "cerebros en un frasco"; no conocen tu proyecto a menos que se lo muestres. Cursor utiliza una técnica avanzada para inyectar contexto automáticamente mediante símbolos (`@`):

- `@Codebase`: Realiza una búsqueda vectorial sobre todo tu proyecto. Útil cuando haces preguntas globales como "¿Dónde se define la autenticación?" o "¿Cómo se relacionan el módulo de Pagos con el de Usuarios?".
- `@Files`: Te permite referenciar archivos específicos manualmente. Es la forma más precisa de dar contexto. Ejemplo: "Usa `@UserDto.java` como base para crear `@DoctorDto.java`".
- `@Docs`: Permite indexar documentación externa. Puedes añadir la URL de la documentación oficial de Spring Boot 4 o Angular 21, y Cursor la leerá antes de responder, reduciendo alucinaciones sobre versiones antiguas.
- `@Git`: Contexto de tus cambios actuales (staged/unstaged) o commits previos, ideal para generar mensajes de commit o changelogs.

### 3. Archivos `.cursorrules`: El Contrato de Comportamiento

El archivo `.cursorrules` soluciona el problema de la "memoria a corto plazo" y la repetitividad. Es un archivo de configuración en lenguaje natural situado en la raíz del proyecto que actúa como un _System Prompt_ persistente.

- **¿Por qué es vital?** Sin él, tendrías que repetir "Usa Java 25 y Signals de Angular" en cada prompt. Con este archivo, la IA "sabe" cómo comportarse antes de que escribas una sola palabra.
- **Contenido Típico**:
  - **Estilo de Código**: Tabs vs Spaces, nomenclatura (camelCase, snake_case), uso de `final` o `var`.
  - **Stack Tecnológico Permitido**: Definir versiones exactas (ej: "No uses RxJS para estado síncrono, usa Signals").
  - **Decisiones Arquitectónicas**: (ej: "Siempre usa Interfaces para los Servicios", "Patrón Hexagonal").

### 4. Prompt Engineering Avanzado para Developers

Escribir buenos prompts es la nueva habilidad blanda (soft skill) esencial. Una petición vaga genera código mediocre. Para obtener resultados de nivel senior, utilizamos la estructura **C-I-R-F**:

#### C (Contexto): Define el rol y el escenario

"Actúa como un arquitecto de software experto en sistemas médicos y Angular 21..."

#### I (Instrucción): El verbo de acción claro y directo

"Crea un componente reutilizable de tabla dinámica..."

#### R (Restricciones): Las reglas que no debe romper

"Usa estrictamente Tailwind CSS para los estilos. No uses ngStyle. Usa input.required() para las propiedades. Debe ser accesible (ARIA)."

#### F (Formato): Cómo quieres la respuesta

"Solo dame el código del archivo TypeScript y el HTML, no necesito explicaciones de texto."

## Parte 2: Práctica y Laboratorio (1h 15m)

### Paso 1: Setup Inicial y Verificación

1. Descargar e instalar [Cursor](https://cursor.com/) desde el sitio oficial.
2. Durante la instalación, selecciona la opción "Importar extensiones de VS Code" para mantener tu entorno familiar.
3. Abre la carpeta raíz del proyecto, asegurándote de tener acceso a `MediCare-Backend` y `MediCare-Frontend`.
4. **Verificación**: Abre la paleta de comandos (`Cmd+Shift+P` / `Ctrl+Shift+P`) y escribe "Cursor: Settings" para verificar que la IA está activa.

### Paso 2: Configuración de la IA (Backend - Spring Boot 4)

El ecosistema de Java evoluciona rápidamente. Spring Boot 4 introduce cambios significativos y Java 25 trae características sintácticas que reducen la verbosidad.

En la raíz de tu proyecto **Spring Boot**, crea el archivo `.cursorrules`. Este "contrato" forzará a la IA a olvidar las prácticas de Java 8 o 11 y a utilizar inglés en todo el código.

```markdown
# Reglas de Comportamiento para MediCare API (Backend)

You are an expert in Java 25, Spring Boot 4, and Modern REST API development.

## Code Style & Standards
- **Java 25 Modern Features**: 
  - ALWAYS use `var` for local variables to improve readability.
  - Use Records (`record`) for all DTOs and immutable data carriers.
  - Use Pattern Matching for `switch` and `instanceof`.
  - Use Text Blocks (`"""`) for SQL queries or JSON strings inside code.
- **Lombok Usage**: Aggressively use `@Data`, `@RequiredArgsConstructor`, and `@Builder` to minimize boilerplate code.
- **Dependency Injection**: PREFER **Constructor Injection** via `@RequiredArgsConstructor`. AVOID field injection (`@Autowired` on fields is forbidden).
- **Collections**: Always use immutable factory methods: `List.of()`, `Set.of()`, `Map.of()`.

## Spring Boot 4 Best Practices
- **HTTP Client**: Use `RestClient` (the modern, fluent API) instead of the legacy `RestTemplate` or reactive `WebClient` (unless specifically doing streaming).
- **Security**: Use Spring Security 6+ Lambda DSL configuration (e.g., `.authorizeHttpRequests(auth -> ...)`). Avoid extending `WebSecurityConfigurerAdapter`.
- **Validation**: Use `jakarta.validation` annotations (`@NotNull`, `@Email`, `@PastOrPresent`) strictly in DTOs, never in Entities.
- **Error Handling**: Implement global handling using `@ControllerAdvice` and strict `ProblemDetails` format (RFC 7807).

## Architecture & Layers
- Flow: Controller -> Service Interface -> Service Implementation -> Repository.
- **Separation of Concerns**: NEVER put business logic in Controllers. Controllers should only handle HTTP request/response mapping.
- **Data Exposure**: Always return DTOs. NEVER return JPA Entities directly to the client to prevent infinite recursion and leakage of internal schema.

## Testing Strategy
- Use JUnit 5 and AssertJ for fluent assertions.
- Use `@MockBean` for mocking dependencies in integration tests.
- Prefer `Testcontainers` for database integration tests.
```

### Paso 3: Configuración de la IA (Frontend - Angular 21)

Angular ha sufrido un renacimiento. La versión 21 consolida la arquitectura "Zone-less" y basada en Signals. Es crítico configurar esto para evitar que la IA genere código estilo "Angular 14" con módulos y decoradores obsoletos, y asegurar que los nombres estén en inglés.

En la raíz de tu proyecto **Angular**, crea el archivo `.cursorrules`.

```markdown
# Reglas de Comportamiento para MediCare Frontend (Angular)

You are an expert in Angular 21, TypeScript 5.x, and Tailwind CSS.

## Angular 21 Core Concepts
- **Strictly Standalone Components**: NgModules are FORBIDDEN. Always use `imports: []` in the `@Component` decorator.
- **Signals First Architecture**: 
  - Use `signal()`, `computed()`, and `effect()` for all local state management.
  - Use `input()` and `output()` (Signal inputs/outputs) instead of the legacy `@Input` and `@Output` decorators.
  - **State Management**: Avoid RxJS for synchronous component state; keep RxJS only for complex asynchronous streams (HTTP) or use `rxResource` / `toSignal` for interoperability.
- **Dependency Injection**: Use the modern `inject()` function. DO NOT use constructor injection for services.

## Styling (Tailwind CSS)
- **Utility First**: Use Tailwind utility classes directly in HTML.
- **No Custom CSS**: Avoid creating `.css` or `.scss` files unless creating complex custom animations or `@apply` abstractions.
- **Responsive Design**: Ensure mobile-first design using `sm:`, `md:`, `lg:`, `xl:` prefixes standardly.

## HTML & Templates
- **Control Flow Syntax**: Use the new built-in syntax: `@if`, `@for`, `@switch`, `@defer`. Do NOT use legacy structural directives like `*ngIf` or `*ngFor`.
- **Image Optimization**: Always use `ngSrc` (NgOptimizedImage directive) instead of `src` for external images to ensure lazy loading and performance.

## Specific Project Context
- App name: MediCare.
- Authentication strategy: JWT tokens stored in a generic storage service (Abstract).
- UI Library: Use pure Tailwind + Headless UI concepts (no heavy component libraries unless specified).
```

### Paso 4: Generación de Documentación Automática

Vamos a probar la capacidad de la IA para entender el contexto global (`@Codebase`) y generar documentación útil para el onboarding de nuevos desarrolladores.

1. Abre el Chat (`Cmd+L` / `Ctrl+L`).
2. Escribe el siguiente prompt (observa el uso de contexto):

    ```plain
    "Analiza @Codebase. Actúa como un Tech Lead y crea un archivo README.md profesional para el repositorio Frontend. Debe incluir:
    
    1. Tecnologías usadas (resaltando Angular 21 y Signals).
    2. Estructura de carpetas actual explicada.
    3. Guía de instalación y ejecución.
    4. Una sección de 'Reglas de Desarrollo' generada automáticamente basada en el contenido del archivo .cursorrules."
    ```

3. Revisa el resultado. Observa cómo la IA extrae las reglas que acabamos de definir y las presenta en formato legible para humanos. Guárdalo en la raíz.

### Paso 5: Refactorización Inteligente y Modernización

_Escenario_: En la Unidad 1, creamos prototipos rápidos de componentes (ej. `PatientListComponent`) que muestran tablas. Probablemente usamos `*ngFor` y CSS tradicional. Vamos a modernizarlo.

1. **Refactorización Inline**:
    - Abre `PatientListComponent`.
    - Selecciona todo el bloque `<table>` o `<div>` que contiene la lista.
    - Presiona `Cmd+K` / `Ctrl+K` (Inline Edit) y escribe:

      ```markdown
      "Refactoriza esta vista para usar el nuevo Control Flow de Angular 21 (@for iterando sobre la signal de pacientes). Asegúrate de aplicar clases de Tailwind para un diseño de tabla moderno ("zebra striping") y soporte para modo oscuro."
      ```

    - Acepta los cambios y observa la limpieza del código (`@for (patient of patients(); track patient.id)`).
2. **Desafío Avanzado con Composer (Arquitectura de Componentes)**:

- Vamos a elevar el nivel de abstracción creando un componente genérico.
- Presiona `Cmd+I` / `Ctrl+I` (Composer) para abrir el editor multi-archivo.
- Prompt:

  ```markdown
  "Crea un componente reutilizable llamado `GenericTableComponent`.
  Requisitos:

  1. Debe usar Inputs basados en Signals (`input.required<T[]>()`) para recibir los datos (data) y la configuración de columnas (cols).
  2. Debe ser totalmente tipado usando Generics de TypeScript.
  3. Una vez creado, modifica el archivo `PatientListComponent` para eliminar la tabla manual e implementar este nuevo `GenericTableComponent`."
  ```

- Acepta los cambios yObserva cómo Composer crea el nuevo archivo `.ts` y actualiza el archivo existente simultáneamente, manteniendo la consistencia definida en `.cursorrules`.

## Resumen de Comandos Cursor y Buenas Prácticas

| Comando | Nombre | Función Principal | Cuándo usarlo (Pro Tip) |
| --- | --- | --- | --- |
| `Cmd + L` / `Ctrl + L` | Chat | Conversación y contexto | Para preguntas conceptuales, explicaciones de errores ("¿Qué hace este código?") o planificación antes de codificar. |
| `Cmd + K` / `Ctrl + K` | Inline Edit | Edición focalizada | Para modificar un bloque de código específico sin perder el contexto visual del archivo. Ideal para refactorizaciones rápidas. |
| `Cmd + I` / `Ctrl + I` | Composer | Edición Multi-archivo | Para crear nuevas features que tocan Model, View y Controller a la vez, o refactorizaciones masivas. |
| `@Codebase` | Contexto Global | Indexado Vectorial | Cuando la IA necesita "entender" todo el proyecto para responder (ej: "¿Dónde se usa esta clase?"). |
| `@Docs` | Documentación | Referencia Externa | Imprescindible para librerías nuevas o específicas (ej: Docs de una librería de pasarela de pagos). |

## Errores Comunes y Cómo Mitigarlos

1. **Contexto Insuficiente (La "Amnesia" del LLM)**:
    - _Síntoma_: La IA genera código genérico que no usa tus clases o DTOs existentes.
    - _Solución_: No asumas que la IA lo sabe todo. Usa explícitamente `@Files` para apuntar a los archivos relacionados (ej: "Crea un test para `@UserService.java`").
2. **Confianza Ciega (El "Piloto Automático")**:
    - _Síntoma_: Aceptas código con imports erróneos o lógica insegura.
    - _Solución_: Angular 21 y Spring Boot 4 cambian rápido. Si Cursor importa `CommonModule` en Angular, es una señal de alerta (ya no es necesario). Audita siempre los imports.
3. **Sobrecarga de Archivos (Context Window Overflow)**:
    - _Síntoma_: Si un archivo tiene más de 500-600 líneas, la IA empieza a "olvidar" el principio o el final al hacer ediciones.
    - _Solución_: Aplica el principio de responsabilidad única. Pide a Cursor: _"Analiza este archivo gigante y propón una estrategia para dividirlo en sub-componentes lógicos o servicios auxiliares"_.
4. **Ambigüedad en el Prompt**:
    - _Error_: "Arregla esto".
    - _Corrección_: "Arregla el error de NullPointer en la línea 45 validando la entrada antes de procesarla". Sé específico con el qué y el _cómo_.
