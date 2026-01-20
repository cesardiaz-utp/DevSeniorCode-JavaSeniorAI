# Unidad 2 - Clase 2: Estilos Modernos, Conexión HTTP y Seguridad CORS

- **Duración**: 2 horas
- **Objetivo**: Romper la barrera técnica y operativa entre el Frontend y el Backend. En esta sesión, no solo conectaremos dos aplicaciones; configuraremos un ecosistema de desarrollo profesional. Transformaremos la interfaz de usuario con **Tailwind CSS**, implementaremos una comunicación asíncrona robusta con el nuevo **HttpClient** de Angular y desmitificaremos y resolveremos el mecanismo de seguridad **CORS**, el obstáculo más frecuente en arquitecturas desacopladas.

## Parte 1: Teoría - La Autopista de la Información (40 Minutos)

### 1. Tailwind CSS: El fin de las hojas de estilo monolíticas

La filosofía "Utility-First" de Tailwind CSS representa un cambio radical respecto a metodologías tradicionales como BEM (Block Element Modifier) o frameworks de componentes como Bootstrap. No es simplemente una librería de estilos; es un motor para crear sistemas de diseño.

- **El problema del CSS Tradicional ("Append-Only")**: En flujos de trabajo convencionales, cada nueva funcionalidad suele requerir nuevas clases CSS. Esto provoca que las hojas de estilo crezcan indefinidamente (archivos de miles de líneas), volviéndose inmantibles. Nadie se atreve a borrar una clase antigua por miedo a romper algo en una página olvidada.
  - _Ejemplo de dolor_: Crear nombres semánticos forzados como `.user-profile-card-container-wrapper--active`.
- **La solución Utility-First y el motor JIT**: Tailwind propone componer interfaces complejas utilizando clases utilitarias primitivas y de bajo nivel directamente en el HTML.
  - **Composición sobre Herencia**: En lugar de heredar estilos, compones el botón usando `bg-blue-500`, `text-white`, `rounded`, `py-2`, `px-4`.
  - **Just-In-Time (JIT) Engine**: A diferencia de versiones antiguas que cargaban todo el CSS posible, el motor moderno de Tailwind observa tus archivos HTML en tiempo real y genera _exclusivamente_ el CSS que estás utilizando. Esto resulta en bundles de producción increíblemente pequeños (a menudo menos de 10kb de CSS).
  - **Consistencia de Diseño**: Al usar una escala predefinida (ej. `p-4` es siempre `1rem`), evitas los "números mágicos" y aseguras que todos los espaciados y tipografías sean matemáticamente consistentes.

### 2. El Cliente HTTP Moderno de Angular: Adiós a los Módulos

Angular 21 ha completado su transición hacia una arquitectura funcional y ligera ("Standalone"), eliminando la necesidad de `NgModules` y clases pesadas para configuraciones simples.

- **Evolución del** `HttpClient`: Anteriormente, dependíamos de importar `HttpClientModule` en el módulo raíz. Esto acoplaba la aplicación y dificultaba el _Tree Shaking_ (eliminación de código muerto).
  - `provideHttpClient(withFetch())`: Esta es la nueva norma. Se configura en el `app.config.ts` como un proveedor global.
  - **La magia de** `withFetch`: Al activar esta función, Angular deja de usar `XMLHttpRequest` (la tecnología antigua de AJAX) y pasa a usar la API nativa fetch del navegador. Esto trae ventajas críticas:
    1. **Soporte nativo para Server-Side Rendering (SSR)**: `fetch` funciona en entornos Node.js, lo que facilita la hidratación de la aplicación en el servidor.
    2. **Mejor manejo de Streams**: Permite procesar respuestas en streaming de manera más eficiente.

### 3. Integración y el Muro de CORS (Cross-Origin Resource Sharing)

CORS es, sin duda, la fuente de frustración número uno para los desarrolladores FullStack junior, pero es un guardián necesario de la seguridad en la web.

- **El Modelo de Seguridad (Same-Origin Policy)**: Por defecto, los navegadores aplican la "Política del Mismo Origen". Esto significa que un script cargado desde `http://localhost:4200` solo puede leer respuestas de `http://localhost:4200`. Si intenta leer de `http://localhost:8080` (donde vive nuestra API Spring Boot), el navegador bloquea la lectura de la respuesta para prevenir ataques como el robo de sesiones.
- **La Negociación (Handshake)**: CORS no es un error, es un protocolo de negociación.
  1. **Peticiones Simples (`GET`/`POST` sin cabeceras raras)**: El navegador envía la petición con una cabecera `Origin`. Si el servidor responde con `Access-Control-Allow-Origin: *` o el dominio específico, la petición pasa.
  2. **Peticiones Complejas y el "Preflight"**: Si añades una cabecera como `Authorization` (para JWT) o usas `PUT`/`DELETE`, el navegador es cauteloso. Antes de enviar la petición real, envía una petición tipo `OPTIONS` ("Preflight").
      - _Navegador_: "Hola servidor 8080, soy el origen 4200. ¿Me dejas enviar un DELETE con un Token?"
      - _Servidor (si está bien configurado)_: "Sí, permito al origen 4200 usar DELETE y la cabecera Authorization".
      - _Navegador_: Procede a enviar la petición real.

      ```mermaid
      sequenceDiagram
          autonumber
          actor User as 👤 Usuario
          participant Browser as 🌐 Navegador (Angular:4200)
          participant Server as ⚙️ Servidor (Spring Boot:8080)

          Note over Browser, Server: 🛑 INICIO DEL BLOQUEO CORS 🛑

          User->>Browser: Clic en "Eliminar Datos"
          
          rect rgb(255, 240, 240)
              Note right of Browser: El navegador detecta una "Petición Compleja"<br/>(Usa método DELETE o Headers personalizados)
              
              %% Paso 1: Preflight (El "Espía")
              Browser->>Server: 🕵️ Petición OPTIONS (Preflight)
              Note right of Browser: Headers enviados:<br/>Origin: http://localhost:4200<br/>Access-Control-Request-Method: DELETE
              
              activate Server
              Note right of Server: 🛡️ Spring Security verifica:<br/>1. ¿Está el origen 4200 en la lista blanca?<br/>2. ¿Está el método DELETE permitido?
              
              %% Paso 2: Respuesta del Servidor
              Server-->>Browser: ✅ Respuesta 200 OK
              Note left of Server: Headers de respuesta:<br/>Access-Control-Allow-Origin: http://localhost:4200<br/>Access-Control-Allow-Methods: DELETE, OPTIONS
              deactivate Server
          end

          %% Paso 3: Verificación del Navegador
          Note right of Browser: 👮 El navegador inspecciona los permisos.<br/>Si coinciden, levanta la barrera.

          rect rgb(240, 255, 240)
              Note over Browser, Server: 🟢 PETICIÓN REAL PERMITIDA 🟢
              
              %% Paso 4: Petición Real
              Browser->>Server: 🚀 DELETE /api/v1/doctors/123
              Note right of Browser: Headers:<br/>Authorization: Bearer token123...
              
              activate Server
              Note right of Server: Ejecuta lógica de negocio<br/>(Borrar registro en DB)
              Server-->>Browser: Respuesta 200 OK (JSON)
              deactivate Server
          end

          Browser-->>User: Muestra "Eliminado con éxito"
      ```

- **La Solución Correcta**: Configurar Spring Security para que intercepte estas peticiones `OPTIONS` y añada automáticamente las cabeceras de permiso requeridas.

## Parte 2: Laboratorio Práctico (1h 20m)

### Paso 1: Instalación y Configuración Profunda de Tailwind CSS (Angular 21)

En Angular 21, la integración se ha simplificado gracias a los nuevos builders y plugins de PostCSS. Usaremos el comando oficial automatizado que configura todo el entorno por nosotros.

1. **Instalación Automatizada (`ng add`)**: Abre la terminal integrada en Cursor (`Ctrl + ~`) en la raíz de tu proyecto Frontend y ejecuta el siguiente comando. Este es el estándar actual recomendado por el equipo de Angular:

    ```bash
    ng add tailwindcss
    ```

2. **Entendiendo la "Magia" (Lo que ocurre bajo el capó)**: Es importante saber qué cambios aplicó este comando en tu proyecto, por si necesitas ajustarlo manualmente en el futuro:
    - **Instalación de Paquetes**: Se instalaron `tailwindcss`, `postcss` y el nuevo plugin `@tailwindcss/postcss`.
    - **Configuración de PostCSS**: Se creó el archivo `.postcssrc.json` en la raíz. Este archivo conecta el compilador de estilos de Angular con el motor de Tailwind.

      ```json
      {
        "plugins": {
          "@tailwindcss/postcss": {}
        }
      }
      ```

    - **Importación en Estilos Globales**: Se actualizó tu archivo `src/styles.css` (o `.scss`). Observa que ya no usamos las antiguas directivas `@tailwind base;`. Ahora se usa una importación estándar mucho más limpia:

      ```css
      @import 'tailwindcss';
      ```

    - **Verificación**: Abre `app.component.html` y añade una clase de prueba: `<h1 class="text-3xl font-bold underline text-blue-600">Hola Tailwind</h1>`. Ejecuta `ng serve` y verifica que los estilos se apliquen.

### Paso 2: Refactorización Visual - De CSS Vainilla a Utilidades

Transformaremos componentes "feos" o con CSS legado en componentes modernos y responsivos.

1. **Navbar Responsive**: Abre `NavbarComponent`. Vamos a usar Flexbox y modificadores de estado. **Prompt Inline**:

    ```plain
    "Reemplaza el CSS manual. Crea un `nav` con:

    - Fondo índigo oscuro (`bg-indigo-600`).
    - Padding vertical y horizontal (`px-4 py-3`).
    - Flexbox para separar logo y menú (`flex justify-between items-center`).
    - Sombra elevada (`shadow-lg`).
    - Texto blanco y seminegrita (`text-white font-semibold`)."
    ```

2. **Diseño de Cards Interactivas**: Para la lista de doctores o especialidades. Observa el uso de `group` y `hover`. **Prompt**:

    ```plain
    "Estiliza este contenedor como una tarjeta moderna:

    - Bordes redondeados grandes (`rounded-xl`).
    - Fondo blanco con borde sutil (`bg-white border border-gray-100`).
    - Sombra suave que crece al pasar el mouse (`shadow-md hover:shadow-xl)`.
    - Transición suave de movimiento (`transition-all duration-300 hover:-translate-y-1`).
    - Padding interno (`p-6`)."
    ```

### Paso 3: Configuración de Seguridad Global en Spring Boot

Vamos a configurar CORS a nivel de infraestructura, no controlador por controlador (evitando el anti-patrón de llenar todo de `@CrossOrigin`).

1. Abre el Backend. Localiza tu clase `SecurityConfig`.
2. **Implementación Robusta con Composer (`Cmd+I` / `Ctrl+I`)**: Pide a la IA que genere una configuración que cubra todos los verbos HTTP.

    ```markdown
    "Configura CORS globalmente dentro del `SecurityFilterChain`.

    1. Crea un `@Bean` llamado `corsConfigurationSource`.
    2. Usa `UrlBasedCorsConfigurationSource` para registrar la configuración.
    3. Define explícitamente:
        - `setAllowedOrigins`: Lista `http://localhost:4200` (Evita * por seguridad).
        - `setAllowedMethods`: GET, POST, PUT, DELETE, OPTIONS.
        - `setAllowedHeaders`: * (o especifica `Authorization`, `Content-Type`, `X-Requested-With`).
        - `setAllowCredentials(true)`: Necesario si enviamos cookies o headers de autenticación en el futuro."
    ```

3. **Ciclo de Vida**: Recuerda reiniciar el servidor Spring Boot (`Stop` -> `Run`) para que la configuración de seguridad se aplique al arranque.

### Paso 4: Consumo de API Real y Diagnóstico

Conectaremos las piezas y verificaremos el flujo de red.

1. **Habilitar Cliente HTTP (Frontend)**: En `app.config.ts`, asegúrate de tener:

    ```typescript
    export const appConfig: ApplicationConfig = {
      providers: [
        provideRouter(routes),
        provideHttpClient(withFetch()) // ¡Crucial!
      ]
    };
    ```

2. **Servicio de Datos (`DoctorApiService`)**: Usa la inyección de dependencias moderna. **Prompt para Composer**:

    ```markdown
    "Genera un servicio `DoctorApiService`.

    - Usa `private http = inject(HttpClient)` para la inyección.
    - Define una interfaz `Specialty` { id: number, name: string }.
    - Crea un método `getAllSpecialties()` que retorne `Observable<Specialty[]>`.
    - La URL base debe ser `http://localhost:8080/api/v1/specialties`.
    - Añade manejo de errores básico con `.pipe(catchError(...))`."
    ```

3. **Renderizado Reactivo en el Componente**: En lugar de usar `.subscribe()` manualmente y gestionar la desuscripción (`ngOnDestroy`), usaremos el `AsyncPipe` en el template o Signals.
    - _En el TS_: `specialties$ = this.doctorService.getAllSpecialties();`
    - _En el HTML_: `@for (item of specialties$ | async; track item.id) { ... }`
4. **Prueba de Fuego (Debugging)**: Abre Chrome DevTools (F12) -> Pestaña Network. Al cargar la página, observa la columna "Type".
    - Si ves una petición `xhr` o `fetch` en rojo: Revisa la consola para ver si es error de CORS o error `500` del servidor.
    - Si ves una petición `preflight` (método OPTIONS) seguida de la petición `GET` (`200 OK`), ¡felicidades! Has configurado correctamente la seguridad.

## Desafío de la Clase (Homework)

### Implementación de un Buscador "Live" con Debounce

Este desafío pondrá a prueba tu manejo de RxJS, eventos del DOM y seguridad Backend.

1. **UI de Búsqueda**: Añade un `<input type="text">` en el Navbar. Estilízalo con Tailwind para que parezca una píldora (`rounded-full`, `bg-gray-100`, `focus:ring-2`, `focus:ring-indigo-500`).
2. **Lógica Reactiva (Frontend)**: No envíes una petición por cada letra que el usuario escribe (eso mataría al servidor).
    - Usa `FormControl` o un `Subject` para capturar el input.
    - Aplica operadores de RxJS:
      - `debounceTime(300)`: Espera 300ms a que el usuario deje de escribir.
      - `distinctUntilChanged()`: No busques si el texto es igual al anterior.
      - `switchMap(query => service.search(query))`: Cancela la petición anterior si llega una nueva.
3. **Endpoint Backend**: Crea `GET /api/v1/doctors/search?query={texto}` en Spring Boot. Debe filtrar por nombre o especialidad usando JPQL o Specifications.
4. **Reto CORS Avanzado (Custom Headers)**:
    - Intenta enviar una cabecera personalizada desde el servicio Angular: `headers: { 'X-Source-App': 'MediCare-Web' }`.
    - Observarás que la petición falla por CORS.
    - **Misión**: Modifica la configuración de Spring Security (`setAllowedHeaders`) para permitir explícitamente `X-Source-App`. Esto demuestra control total sobre la "frontera" de tu API.
