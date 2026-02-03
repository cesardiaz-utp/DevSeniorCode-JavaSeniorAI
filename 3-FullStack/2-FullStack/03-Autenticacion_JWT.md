# Unidad 2 - Clase 3: Estrategias de Almacenamiento y Autenticación con JWT

- **Duración**: 2 horas
- **Objetivo**: Comprender profundamente el ciclo de vida de la autenticación moderna, implementar una estrategia de persistencia segura en el cliente (Storage) y sincronizar el estado de la aplicación utilizando la reactividad de **Signals** para una experiencia de usuario fluida.

## Parte 1: Teoría - ¿Dónde guardo mis secretos? (45 Minutos)

### 1. Fundamentos de Persistencia en el Cliente

En el desarrollo de Single Page Applications (SPA), una vez que el servidor valida las credenciales del usuario, el cliente debe ser capaz de "recordar" esa identidad. A diferencia de las aplicaciones web antiguas que usaban sesiones en el servidor (JSESSIONID), las arquitecturas modernas son _Stateless_ (sin estado).

#### 1.1. Comparativa de Mecanismos de Almacenamiento

El navegador ofrece tres bóvedas principales para guardar datos. Elegir la correcta es una decisión de seguridad crítica.

| Característica | LocalStorage | SessionStorage | Cookies (`httpOnly`) |
| --- | --- | --- | --- |
| **Persistencia** | **Largo Plazo**: Los datos persisten incluso si se cierra el navegador o se reinicia el equipo. Deben borrarse manualmente. | **Volátil**: Los datos viven solo mientras la _pestaña_ está abierta. Al cerrar la pestaña, los datos mueren. | **Configurable**: Se define una fecha de expiración (`Expires`) o duración (`Max-Age`). |
| **Ámbito (Scope)** | Compartido entre todas las pestañas y ventanas del mismo origen (dominio). | Aislado por pestaña. Una pestaña no puede leer el SessionStorage de otra, aunque sea el mismo sitio. | Compartido por dominio. Se envían automáticamente al servidor en cada petición HTTP. |
| **Acceso JavaScript** | Total acceso mediante `window.localStorage`. | Total acceso mediante `window.sessionStorage`. | **Restringido**: Si la bandera `HttpOnly` está activa, JS **no** puede leerlas (`document.cookie` devuelve vacío). |
| **Tamaño Aprox.** | ~5MB - 10MB | ~5MB | ~4KB (Muy limitado). |
| **Vector de Ataque** | **XSS (Cross-Site Scripting)**: Vulnerable si el atacante inyecta JS. | **XSS**: Vulnerable. | **CSRF (Cross-Site Request Forgery)**: Vulnerable si no se usan tokens anti-CSRF o SameSite policies |

#### 1.2. Profundizando en los Riesgos de Seguridad

##### XSS (Cross-Site Scripting)

Es el ataque donde un actor malicioso logra ejecutar JavaScript ajeno en tu aplicación (ej. a través de un input no sanitizado o una librería infectada).

- **Impacto en Storage**:

  ```typescript
  const token = localStorage.getItem('token');
  sendToHacker(token);
  ```

- **Mitigación**: Sanitización estricta de datos (Angular lo hace por defecto), Content Security Policy (CSP) y auditoría de dependencias (npm audit).

##### CSRF (Cross-Site Request Forgery)

Ocurre cuando un sitio malicioso fuerza al navegador del usuario a realizar acciones no deseadas en una web donde el usuario ya está autenticado.

- **Mecanismo**: Como las Cookies se envían automáticamente, el servidor cree que la petición es legítima.
- **Mitigación**: Uso de Cookies `SameSite=Strict` y tokens anti-CSRF.

### 2. JSON Web Tokens (JWT)

El JWT es el estándar de facto (RFC 7519) para la transmisión segura de información entre partes como un objeto JSON.

#### 2.1. Anatomía de un Token

Un JWT son tres partes separadas por puntos (`.`): `aaaaaa.bbbbbb.cccccc`

1. **Header (Cabecera)**: Define el algoritmo de firma (ej. HS256 o RS256) y el tipo (JWT).

    ```json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```

2. **Payload (Carga Útil)**: Contiene los "Claims" (afirmaciones). Es información **no cifrada** (solo codificada en Base64), por lo que **nunca** se debe guardar información sensible como contraseñas aquí.

    - _Registered Claims_: `sub` (sujeto/ID), `iss` (emisor), `exp` (expiración).
    - _Public/Private Claims_: `"role": "ADMIN"`, `"email": "user@test.com"`.

3. **Signature (Firma)**: Es la garantía de integridad. Se genera combinando el Header + Payload + una **Clave Secreta** (que solo tiene el Backend). Si alguien modifica el Payload en el navegador (ej. cambia rol de USER a ADMIN), la firma ya no coincidirá y el servidor rechazará el token.

#### 2.2. Decodificación en el Cliente

Aunque el Frontend no puede verificar la firma (no tiene la clave secreta), sí puede y debe decodificar el Payload para mejorar la UX.

- **Caso de Uso**: Ocultar el botón "Panel de Administración" si el claim `roles` no contiene `ADMIN`.
- **Librería Recomendada**: `jwt-decode` (ligera y estándar).

### 3. Arquitectura de Estado en Angular (State Management)

Uno de los desafíos más grandes en el desarrollo Frontend es la Sincronización de Estado.

- **Problema**: El usuario inicia sesión en el componente `LoginComponent`. El token se guarda en `LocalStorage`. Sin embargo, el `NavbarComponent` (que es un componente "hermano" o "tío") no se entera de que el usuario ya entró y sigue mostrando el botón "Login".

#### 3.1. La Solución Reactiva: Signals

Angular 21 promueve el uso de Signals para gestionar este estado globalmente sin la complejidad de librerías externas como NgRx para casos simples.

##### Patrón de Servicio de Autenticación (`AuthService`)

El servicio actúa como la "fuente de la verdad".

```typescript
// Ejemplo conceptual del patrón
@Injectable({ providedIn: 'root' })
export class AuthService {
  // 1. Estado Reactivo (Signal)
  // Inicialmente null, luego contendrá el objeto usuario
  readonly currentUser = signal<User | null>(null);

  constructor(private storage: StorageService) {
    // 2. Hidratación al inicio
    // Al recargar la app, verificamos si hay token guardado
    const token = this.storage.getToken();
    if (token && !this.isExpired(token)) {
      const user = this.decodeToken(token);
      this.currentUser.set(user); // Restauramos la sesión
    }
  }

  login(credentials: any) {
    return this.http.post('/api/login', credentials).pipe(
      tap((response: any) => {
        // 3. Actualización del Estado
        this.storage.setToken(response.token);
        const user = this.decodeToken(response.token);
        this.currentUser.set(user); // ¡El Navbar se actualiza automáticamente!
      })
    );
  }

  logout() {
    this.storage.removeToken();
    this.currentUser.set(null); // La UI reacciona y vuelve a estado anónimo
  }
}
```

## Parte 2: Laboratorio Práctico (1h 15m)

### Paso 1: Ajuste Backend: Preparando la API de Autenticación

Antes de trabajar en el Frontend, es imperativo asegurar que nuestro Backend exponga correctamente los mecanismos de autenticación.

- **Objetivo**: Exponer un endpoint público que reciba credenciales, valide el usuario y retorne un JWT firmado.
- **Prompt para Cursor (Composer)**:

  ```plain
  1. Verifica o crea el AuthController en el paquete auth.
  2. Implementa el endpoint POST `/auth/login`. Debe recibir un `LoginRequest` (email, password) y retornar un `AuthResponse` (token, tipo).
  3. **Seguridad**: Asegúrate de que en **SecurityConfig**, la ruta `/auth/**` esté configurada con `.permitAll()` para que los usuarios no autenticados puedan acceder.
  4. **Maneja las excepciones**: Si las credenciales son inválidas (`BadCredentialsException`), retorna un 401 Unauthorized claro."
  ```

### Paso 2: Abstracción del Almacenamiento (`StorageService`)

No usaremos `localStorage.setItem()` directamente en los componentes.

- **Objetivo**: Crear una capa de abstracción que nos permita cambiar de estrategia (ej. a Cookies o SessionStorage) sin tocar el resto de la app.
- **Prompt para Cursor (Composer)**:

  ```plain
  Crea un servicio `StorageService` en Angular.
  Debe tener métodos para:

  1. `setToken(token: string)`: Guarda en localStorage.
  2. `getToken()`: Retorna string | null.
  3. `removeToken()`: Limpia el storage.
  4. `isAuthenticated()`: Retorna true si existe token.
  
  Restricción: Usa un bloque `try-catch` al acceder a localStorage para evitar crashes si el usuario tiene las cookies/storage bloqueados en su navegador.
  ```

### Paso 3: Gestión de Estado Global (`AuthService`)

Este servicio será el cerebro de la identidad en nuestra aplicación.

- **Objetivo**: Conectar con la API, persistir el token y notificar a toda la UI mediante Signals.
- **Prompt para Cursor (Composer)**:

  ```plain
  Crea/Actualiza `AuthService`.

  1. Inyecta `StorageService` y `HttpClient`.
  2. Declara una signal pública de lectura `currentUser = signal<User | null>(null)`.
  3. Método `login(credentials)`:
      - Hace POST a `/auth/login`.
      - Al recibir éxito: Guarda token en `StorageService` y actualiza `currentUser.set(decodedUser)`.
      - Usa la librería `jwt-decode` (o una función helper manual) para leer el payload del token.
  4. Método `logout()`: Borra token y hace `currentUser.set(null)`.
  ```

### Paso 4: Diseño de Login con Tailwind (`LoginComponent`)

- **Objetivo**: Crear una interfaz de usuario limpia y moderna.
- **Instrucciones**:
  1. Genera el componente: `ng g c pages/login`.
  2. Usa **Inline Edit (`Cmd+K`)** en el HTML con el siguiente prompt:

      ```plain
      Crea un formulario de login centrado vertical y horizontalmente en la pantalla (h-screen).

      - Card blanca con sombra suave (shadow-xl).
      - Input para Email y Password con estilos de 'floating label' o borde moderno (focus:ring).
      - Botón de 'Ingresar' con gradiente (bg-gradient-to-r from-blue-500 to-indigo-600).
      - Muestra un mensaje de error en rojo si la autenticación falla.
      ```

### Paso 5: Conexión Reactiva en el Navbar

- **Objetivo**: Que la barra de navegación cambie automáticamente cuando el usuario inicia sesión.
- **Implementación**: Inyecta `AuthService` en `NavbarComponent` y usa el control de flujo de Angular:

  ```html
  <nav class="...">
    <!-- Logo ... -->
    <div class="menu">
      @if (authService.currentUser()) {
        <span class="text-gray-700">Hola, {{ authService.currentUser()?.sub }}</span>
        <button (click)="authService.logout()" class="btn-danger">Salir</button>
      } @else {
        <a routerLink="/login" class="btn-primary">Iniciar Sesión</a>
      }
    </div>
  </nav>
  ```

### Buenas Prácticas de Implementación

1. **Manejo de Excepciones**: El acceso a `localStorage` puede fallar (ej. modo incógnito estricto o almacenamiento lleno). Siempre envuelve las llamadas en bloques `try-catch`.
2. **No Confiar en el Cliente**: Recuerda que cualquier validación en el Frontend (`currentUser`, `if(admin)`) es solo cosmética para mejorar la experiencia del usuario. La seguridad real siempre reside en los endpoints del Backend validando el token en cada petición.

## Desafío (Homework)

Mejora la experiencia de usuario (UX) implementando una redirección automática al cerrar sesión.

1. Modifica el método `logout()` en el `AuthService`.
2. Inyecta el Router de Angular.
3. Al hacer logout, navega programáticamente a la ruta raíz `/` o `/login`.
4. **Bonus**: Añade un "Toast" o notificación flotante usando una librería ligera (o un componente propio con Tailwind) que diga "Sesión cerrada correctamente".
