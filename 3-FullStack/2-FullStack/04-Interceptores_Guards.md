# Unidad 2 - Clase 4: Seguridad en el Cliente con Interceptores y Guards

- **Duración**: 2 horas
- **Objetivo**: Completar el ciclo de seguridad implementando vigilancia activa en el Frontend. En esta sesión, configuraremos **Interceptores Funcionales** para inyectar tokens JWT automáticamente en cada petición, manejaremos errores críticos de sesión (401/403) de forma centralizada y blindaremos las rutas de la aplicación mediante **Guards Funcionales**, asegurando que solo los usuarios autorizados accedan a recursos sensibles.

## Parte 1: Teoría - La Defensa en Profundidad en el Frontend (40 Minutos)

Una vez que tenemos un JWT válido almacenado en el `StorageService`, el siguiente desafío es utilizarlo de forma transparente, eficiente y segura. No es viable repetir código en cada uno de nuestros servicios para añadir manualmente el header `Authorization`. Para resolver esto, Angular nos proporciona mecanismos de interceptación y protección que actúan como "Middleware" en el navegador.

### 1. Interceptores Funcionales (`HttpInterceptorFn`)

Los interceptores son el mecanismo más potente de Angular para manipular el tráfico HTTP. En versiones anteriores (Angular 14 e inferiores), se definían como Clases inyectables complejas (`@Injectable`) que requerían mucho código repetitivo. Desde Angular 15+, hemos transicionado hacia **Interceptores Funcionales**, alineándonos con el paradigma de programación funcional.

#### Concepto Técnico: El Patrón "Cadena de Responsabilidad"

Un interceptor es una función que se ubica entre tu componente y la red. Captura **toda** petición HTTP saliente (Request) y **toda** respuesta entrante (Response).
Funciona como una tubería: la petición entra, el interceptor la modifica y la pasa al siguiente eslabón (`next`).

#### Anatomía y Retos de Implementación

La firma de un interceptor funcional es: `(req: HttpRequest<unknown>, next: HttpHandlerFn) => Observable<HttpEvent<unknown>>`.

1. **Inmutabilidad del Request**: Un concepto crítico es que los objetos `HttpRequest` en Angular son inmutables. No puedes hacer simplemente `req.headers.set('Authorization', token)`. Esto fallaría.
    - _Solución_: Debes usar el método `req.clone()`. Esto crea una copia exacta de la petición original donde puedes aplicar modificaciones seguras.
2. **Inyección de Dependencias (DI) en Funciones**: Al no ser clases, no tenemos constructor. Para acceder a servicios (como `StorageService` o `Router`), utilizamos la función `inject()` de Angular. Esto debe hacerse en el cuerpo principal de la función interceptora.
3. **Registro**: A diferencia de los módulos antiguos, estos interceptores se registran directamente en la configuración del cliente HTTP (`provideHttpClient`) usando la función `withInterceptors([...])`. El orden en el array importa: se ejecutan secuencialmente.

### 2. Gestión Centralizada de Errores HTTP

El manejo de errores es una preocupación transversal. Si el token expira mientras el usuario navega, el Backend responderá con un error HTTP 401 Unauthorized o 403 Forbidden.

#### ¿Por qué un Interceptor de Errores?

Sin un interceptor, tendrías que escribir un `try-catch` o un `.subscribe({ error: ... })` en cada llamada al servidor para redirigir al login si falla la sesión. Esto viola el principio **DRY** (Don't Repeat Yourself).

#### Flujo de Manejo con RxJS

El interceptor de errores utiliza el poder de los operadores reactivos:

1. **Interceptar la respuesta**: Usamos next(req) que retorna un Observable.
2. **Operador `catchError`**: Interceptamos el flujo solo si ocurre un error.
3. **Análisis del `HttpErrorResponse`**: Angular envuelve el error de red en este objeto. Verificamos su propiedad `status`.
    - **Errores 4xx (Cliente)**: 401 (Token inválido/expirado) o 403 (Rol insuficiente). Aquí ejecutamos la lógica de limpieza de sesión (`logout`) y redirección.
    - **Errores 5xx (Servidor)**: Podríamos mostrar un mensaje global tipo "Servidor en mantenimiento".
4. **Propagación (`throwError`)**: Es vital relanzar el error al final. Si no lo hacemos, el componente que inició la petición pensará que todo salió bien y podría romperse al intentar leer datos inexistentes.

### 3. Guards Funcionales (`CanActivateFn`)

Mientras los interceptores protegen los datos (la API), los Guards protegen la experiencia de usuario (la navegación).

#### Evolución hacia Funciones

Anteriormente implementábamos la interfaz `CanActivate`. Ahora, un Guard es simplemente una función que retorna:

- `boolean`: `true` (pasa) o `false` (bloqueado).
- `UrlTree`: Un objeto que representa una redirección inmediata.
- `Observable<boolean | UrlTree>`: Para lógica asíncrona (ej. preguntar al backend si el token es válido antes de dejar pasar).

#### Estrategia de Implementación

1. **Contexto de Ejecución**: El Guard recibe `ActivatedRouteSnapshot` (dónde quiere ir el usuario) y `RouterStateSnapshot` (dónde está ahora).
2. **Lógica Síncrona vs Asíncrona**:
    - Si solo verificamos la existencia de un token en LocalStorage, la respuesta es inmediata (Síncrona).
    - Si necesitamos validar el token contra el servidor, debemos retornar un `Observable` y usar operadores como `map` para transformar la respuesta del backend en un booleano.
3. **Redirección Inteligente (`UrlTree`)**: En lugar de inyectar el Router y navegar manualmente dentro del Guard (lo cual puede causar condiciones de carrera), lo moderno es retornar un `UrlTree`.
    - _Ejemplo_: `return isAuthenticated ? true : router.createUrlTree(['/login']);`

## Parte 2: Laboratorio Práctico (1h 20m)

### Paso 1: El Interceptor de Autenticación (`authInterceptor`)

Este interceptor inyectará automáticamente el token en todas las peticiones a nuestra API.

1. **Generación**: No hay comando CLI específico para funciones, así que crea el archivo manual o pide a Cursor:
    - _Ruta_: `src/app/core/interceptors/auth.interceptor.ts`
2. **Prompt para Cursor (Composer)**:

    ```plain
    Crea un interceptor funcional llamado `authInterceptor` (`HttpInterceptorFn`).
    Lógica:

    1. Inyecta StorageService.
    2. Obtén el token.
    3. Si existe, clona la petición (`req.clone`) añadiendo el header `Authorization` con el valor `Bearer {token}`.
    4. Retorna `next(req)` con la petición clonada (o la original si no hay token).
    ```

### Paso 2: El "Salvavidas" (`errorInterceptor`)

Este interceptor nos protege de quedarnos en un estado inconsistente cuando el token muere.

1. **Prompt para Cursor**:

```plain
Crea `errorInterceptor` (`HttpInterceptorFn`).

Lógica:

1. Usa `next(req).pipe(...)` para capturar la respuesta.
2. Usa el operador `catchError`.
3. Si el error es `HttpErrorResponse` y el status es `401` (Unauthorized) o `403` (Forbidden):
    - Inyecta `AuthService` y llama a `logout()`.
    - Inyecta `Router` y redirige a `/login`.
4. Siempre relanza el error (`throwError`) para que el componente también se entere si es necesario.
```

### Paso 3: Registro Global (`app.config.ts`)

Los interceptores no funcionan mágicamente; hay que registrarlos.

1. Abre `src/app/app.config.ts`.
2. Busca `provideHttpClient(withFetch())`.
3. Modifícalo para incluir `withInterceptors`:

    ```typescript
    ...
    provideHttpClient(
      withFetch(),
      withInterceptors([authInterceptor, errorInterceptor]) // El orden importa
    )
    ...
    ```

### Paso 4: Protección de Rutas (`authGuard` y `adminGuard`)

Vamos a crear los porteros de la navegación.

1. **Generación**: `ng g g core/guards/auth --implements CanActivateFn` (O usa el chat).
2. **Prompt para AuthGuard**:

    ```plain
    Implementa `authGuard`. Si `AuthService.isAuthenticated()` es true, retorna true. Si no, redirige a `/login` y retorna false.
    ```

3. **Prompt para AdminGuard (Desafío)**:

    ```plain
    Implementa adminGuard.

    1. Verifica si está autenticado.
    2. Verifica si el usuario tiene el rol 'ADMIN' (necesitarás un método `hasRole('ADMIN')` en tu `AuthService` que lea el payload del token).
    3. Si falla, redirige a `/forbidden` o al home."
    ```

4. **Aplicación en Rutas (`app.routes.ts`)**:

```typescript
{
  path: 'dashboard',
  component: DashboardComponent,
  canActivate: [authGuard] // Protección básica
},
{
  path: 'admin',
  loadComponent: () => ...,
  canActivate: [authGuard, adminGuard] // Protección doble
}
```

### Paso 5: Verificación Integral (Backend + Frontend)

Para asegurar que el sistema funciona, haremos una prueba de estrés.

1. **Prueba de Inyección (Happy Path)**:
    - Haz login en la app.
    - Abre la pestaña **Network** del navegador.
    - Realiza una acción que consuma la API (ej. cargar lista de doctores).
    - Haz clic en la petición y verifica en **Request Headers** que aparezca: `Authorization: Bearer eyJhbGci...`
2. **Prueba de Expiración (Edge Case)**:
    - Detén el Backend.
    - Configura la expiración de tokens en Spring Boot a 1 minuto (temporalmente) o borra manualmente unas letras del token en el LocalStorage del navegador.
    - Intenta navegar.
    - **Resultado esperado**: El Backend rechazará la petición (`401 Unauthorized`). El `errorInterceptor` capturará ese error e inmediatamente te redirigirá al Login.

## Desafío (Homework)

Mejora la UX implementando un indicador de carga visual que funcione automáticamente para **toda** la aplicación.

1. Crea un `UiService` con una signal `isLoading`.
2. Crea un `loadingInterceptor`.
    - Al inicio de la petición: `uiService.isLoading.set(true)`.
    - En el `finalize` del pipe (RxJS): `uiService.isLoading.set(false)`.
3. En `AppComponent`, muestra una barra de progreso o spinner si `isLoading()` es true.
4. ¡Nunca más tendrás que poner `loading = true` manualmente en tus componentes!
