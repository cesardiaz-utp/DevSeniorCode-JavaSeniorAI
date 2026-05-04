# Unidad 3 - Clase 3: Pruebas E2E en Angular con Cypress

- **Duración**: 2 horas
- **Objetivo**: Enseñar a construir pruebas end-to-end efectivas en Angular utilizando Cypress, con enfoque en flujos de usuario completos, integración con la aplicación real y validación de funcionalidades críticas.

## Introducción

Las pruebas E2E (End-to-End) validan la aplicación completa desde la perspectiva del usuario final. A diferencia de las pruebas unitarias que prueban unidades aisladas, las E2E prueban flujos completos: desde la interfaz de usuario hasta la base de datos.

Cypress es el framework moderno por excelencia para pruebas E2E, ofreciendo:

- Ejecución rápida y determinista
- Debugging visual integrado
- API simple y expresiva
- Soporte nativo para aplicaciones modernas (SPA, PWAs)

En esta clase combinaremos Cypress con Angular para validar que toda la aplicación funciona correctamente en conjunto.

### ¿Por qué Cypress en Angular?

- **Ejecución en navegador real**: No emula el DOM, usa el navegador real para máxima fidelidad.
- **Tiempo de viaje cero**: Hot reload de tests durante desarrollo.
- **Debugging visual**: Screenshots y videos automáticos de fallos.
- **API moderna**: Promesas nativas, sin callbacks anidados.
- **Integración perfecta**: Con Angular CLI y DevServer.

## Parte 1: Teoría de pruebas E2E

### A. ¿Qué son las pruebas E2E?

Las pruebas E2E simulan el comportamiento real del usuario en la aplicación completa, desde el momento en que abre el navegador hasta que completa una tarea específica. A diferencia de las pruebas unitarias que verifican piezas individuales, las E2E validan que todos los componentes trabajen juntos armoniosamente.

- **Alcance**: Desde la interfaz de usuario (UI) hasta la persistencia de datos (base de datos, APIs externas). Cubre la aplicación completa, incluyendo routing, formularios, validaciones, llamadas HTTP, almacenamiento local y navegación.
- **Perspectiva**: Usuario final, no desarrollador. Se enfoca en la experiencia real del usuario, simulando clicks, typing, scrolling y navegación como lo haría una persona real.
- **Objetivo**: Validar flujos críticos de negocio end-to-end, asegurando que funcionalidades complejas como registro de usuario, checkout de compras o gestión de datos funcionen correctamente en producción.
- **Entorno**: Aplicación corriendo en un navegador real con configuración de producción, incluyendo todas las dependencias, servicios externos y configuraciones de red.

**Diferencias clave con pruebas unitarias:**

| Aspecto        | Pruebas Unitarias    | Pruebas E2E                 |
| -------------- | -------------------- | --------------------------- |
| **Alcance**    | Unidad aislada       | Aplicación completa         |
| **Velocidad**  | Muy rápidas (< 1s)   | Lentas (10-60s)             |
| **Fiabilidad** | Alta (deterministas) | Media (depende de entorno)  |
| **Costo**      | Bajo                 | Alto                        |
| **Valor**      | Validar lógica       | Validar experiencia usuario |

### B. Arquitectura de Cypress

Cypress se ejecuta directamente en el navegador, paralelamente a la aplicación:

```plain
┌─────────────────┐    ┌─────────────────┐
│   Cypress Test  │    │   Your App      │
│   (JavaScript)  │◄──►│   (Angular)     │
│                 │    │                 │
│ - Commands      │    │ - Components    │
│ - Assertions    │    │ - Services      │
│ - DOM Access    │    │ - API Calls     │
└─────────────────┘    └─────────────────┘
         │                       │
         └───────────────────────┘
              Same Browser
```

**Ventajas:**

- **Sin WebDriver**: Comunicación directa con el navegador, eliminando la capa intermedia de Selenium WebDriver. Esto significa menos configuración, menos puntos de falla y mayor velocidad de ejecución, ya que no hay traducción entre protocolos.
- **Tiempo real**: Acceso inmediato al DOM, network, console y todos los objetos del navegador. Permite inspeccionar el estado de la aplicación en tiempo real durante la ejecución de tests, facilitando el debugging y la comprensión de lo que ocurre.
- **Snapshots**: Viajes en el tiempo para debugging, capturando automáticamente el estado del DOM, CSS y JavaScript en cada paso del test. Permite "rebobinar" y ver exactamente qué cambió entre pasos, haciendo el debugging visual y eficiente.

### C. Ciclo de vida de una prueba E2E

Cada prueba E2E sigue un ciclo estructurado que simula el viaje completo del usuario:

1. **Setup**: Configurar aplicación y datos de prueba. Iniciar la aplicación, preparar el estado inicial (datos en base de datos, localStorage, etc.), configurar mocks de APIs externas y establecer condiciones previas para el test.
2. **Navigate**: Ir a la página inicial. Acceder a la URL de la aplicación y navegar a la página donde comienza el flujo a probar, asegurando que la aplicación esté cargada correctamente.
3. **Interact**: Simular acciones del usuario (clicks, typing, scrolling, etc.). Ejecutar la secuencia de interacciones que un usuario real haría: llenar formularios, hacer clicks, navegar entre páginas, subir archivos, etc.
4. **Assert**: Verificar estado de la aplicación. Comprobar que el resultado esperado se cumple: elementos visibles, datos correctos, navegación exitosa, mensajes de error apropiados, etc.
5. **Teardown**: Limpiar datos si es necesario. Restaurar el estado de la aplicación a su condición original, eliminar datos de prueba temporales y preparar el entorno para el siguiente test.

### D. Configuración de Cypress en Angular

Según la documentación oficial de Angular (angular.dev/tools/cli/end-to-end), Cypress es el framework E2E recomendado para aplicaciones Angular modernas. La integración se realiza automáticamente mediante el Angular CLI.

#### Paso 1: Instalar Cypress

La forma más sencilla de agregar Cypress a un proyecto Angular es usando el comando:

```bash
ng e2e
```

Si se te pregunta qué framework E2E quieres usar, selecciona "Cypress". Esto instalará Cypress y configurará automáticamente los archivos necesarios.

Alternativamente, puedes instalar Cypress manualmente con:

```bash
ng add @cypress/schematic
```

Este comando instala Cypress y configura automáticamente:

- Dependencias necesarias (`cypress`, `@cypress/schematic`)
- Archivo de configuración `cypress.config.ts`
- Scripts en `package.json`
- Estructura de directorios básica

#### Paso 2: Verificar configuración automática

Después de instalado el CLI crea automáticamente:

- **`cypress.config.ts`**: Este archivo contiene la configuración básica para ejecutar Cypress con Angular, incluyendo la URL base y patrones de archivos de test.

  ```ts
  import { defineConfig } from "cypress";

  export default defineConfig({
    e2e: {
      baseUrl: "http://localhost:4200",
      specPattern: "cypress/e2e/**/*.cy.{js,jsx,ts,tsx}",
      supportFile: "cypress/support/e2e.ts",
    },
    component: {
      devServer: {
        framework: "angular",
        bundler: "webpack",
      },
      specPattern: "*.cy.ts",
    },
  });
  ```

- **Scripts en `package.json`:**: El CLI agrega scripts para ejecutar Cypress en modo interactivo y headless.

  ```json
  {
    "scripts": {
      "cy:open": "cypress open",
      "cy:run": "cypress run",
      "test:e2e": "start-server-and-test start http://localhost:4200 cy:run"
    }
  }
  ```

#### Paso 3: Ejecutar Cypress

Para abrir el Test Runner interactivo:

```bash
# Abrir Test Runner interactivo
npm run cy:open

# Ejecutar tests headless
npm run cy:run

# Ejecutar con servidor de desarrollo
npm run test:e2e
```

#### Paso 4: Estructura de archivos generada

```plain
cypress/
├── e2e/
│   └── example.cy.ts  # Test de ejemplo
├── fixtures/
│   └── example.json
├── support/
│   ├── commands.ts
│   └── e2e.ts
└── config.ts
```

## Parte 2: API de Cypress - Comandos y Assertions

### A. Comandos básicos

Cypress ofrece una API fluida para interactuar con la aplicación:

#### 1. Navegación

La navegación es el primer paso en cualquier test E2E. Cypress proporciona comandos simples para acceder a diferentes rutas de la aplicación.

```ts
// Visitar la página principal (siempre es el primer paso)
cy.visit("/");

// Navegar a una ruta específica con parámetros
cy.visit("/users/123");
cy.visit("/products?category=electronics&sort=price");

// Visitar con opciones avanzadas
cy.visit("/", {
  onBeforeLoad: (win) => {
    // Ejecutar código antes de cargar la página
  },
});

// Recargar la página actual (útil después de cambios)
cy.reload();

// Navegar en el historial del navegador
cy.go("back"); // Botón atrás
cy.go("forward"); // Botón adelante
cy.go(-1); // Ir atrás 1 página
cy.go(2); // Ir adelante 2 páginas
```

**Notas importantes:**

- `cy.visit()` debe ser el primer comando en un test para establecer el estado inicial
- Cypress espera automáticamente a que la página esté lista antes de continuar
- Usa `cy.visit()` en `beforeEach()` para establecer un punto de partida limpio

#### 2. Seleccionar elementos

La selección de elementos es crucial para interactuar con la UI. Cypress proporciona múltiples estrategias, siendo `data-testid` la más recomendada para tests E2E.

```ts
// ✅ MEJOR: Por data-testid (menos frágil a cambios de CSS)
cy.get("[data-testid='submit-btn']").click();
cy.get("[data-testid='user-email']").type("test@example.com");

// Por selectores CSS (evitar si es posible)
cy.get(".btn-primary"); // ❌ Frágil: si cambia la clase, falla
cy.get("#user-form");

// Por texto (útil para botones y enlaces)
cy.contains("Guardar"); // Encuentra elemento con este texto
cy.contains("button", "Eliminar"); // Encuentra botón con este texto

// Por atributos
cy.get("input[name='email']");
cy.get("input[type='password']");

// Combinaciones de selectores
cy.get(".form-group input[type='email']");
cy.get("form#login input[type='email']");

// Seleccionar el primer o último elemento
cy.get("li").first();
cy.get("li").last();

// Seleccionar por índice
cy.get("li").eq(2); // Tercer elemento (0-indexed)
```

**Estrategia recomendada:**

1. Usa `data-testid` en componentes Angular para selectores estables
2. Usa `cy.contains()` para elementos con texto visible
3. Evita selectores de clase (`.btn-primary`) que cambian con refactoring CSS

#### 3. Interacciones

Las interacciones simulan las acciones del usuario: clicks, escritura, selecciones, etc.

```ts
// CLICKS - Diferentes tipos de clicks
cy.get("[data-testid='save-btn']").click(); // Click normal
cy.get("[data-testid='menu']").rightclick(); // Click derecho
cy.get("[data-testid='item']").dblclick(); // Doble click

// TYPING - Escribir en inputs
cy.get("input[name='email']").type("user@example.com"); // Tecleo rápido
cy.get("textarea").type("Mensaje largo", { delay: 100 }); // Tecleo lento (simula usuario)

// Limpiar y luego escribir
cy.get("input[name='search']").clear().type("nuevo texto");

// Presionar teclas especiales
cy.get("input").type("{enter}"); // Enter
cy.get("input").type("{backspace}"); // Backspace
cy.get("input").type("{tab}"); // Tab
cy.get("input").type("{esc}"); // Escape

// SELECT - Seleccionar opciones en dropdowns
cy.get("select[name='country']").select("mexico"); // Por valor
cy.get("select").select("Option 1"); // Por texto visible

// CHECKBOXES y RADIO BUTTONS
cy.get("input[type='checkbox']").check(); // Marcar checkbox
cy.get("input[type='checkbox']").uncheck(); // Desmarcar checkbox
cy.get("input[type='radio'][value='yes']").check(); // Seleccionar radio

// Verificar si están marcados
cy.get("input[type='checkbox']").should("be.checked");

// FORM SUBMISSION
cy.get("form").submit(); // Enviar formulario
cy.get("button[type='submit']").click(); // O simplemente hacer click

// HOVER y FOCUS
cy.get("[data-testid='dropdown-trigger']").trigger("mouseover"); // Simular hover
cy.get("input").focus(); // Dar focus a elemento

// SCROLL
cy.get("[data-testid='bottom-section']").scrollIntoView(); // Scroll hacia elemento
```

**Buenas prácticas:**

- Usa `{ delay: 100 }` para simular typing humano (más lento)
- Cypress espera automáticamente a que elementos puedan ser interactuados (visible, enabled)
- Encadena acciones: `cy.get(...).clear().type(...).submit()`

#### 4. Assertions

Las aserciones verifican que el estado de la aplicación es el esperado. Son el corazón de los tests E2E.

```ts
// VISIBILIDAD Y EXISTENCIA
cy.get(".alert").should("be.visible"); // Elemento visible en pantalla
cy.get(".error-msg").should("exist"); // Elemento existe en DOM
cy.get(".modal").should("not.exist"); // Elemento NO existe
cy.get("input").should("be.hidden"); // Elemento oculto
cy.get("input").should("not.be.visible"); // Elemento no visible

// CLASES CSS
cy.get(".loading").should("have.class", "active");
cy.get("button").should("not.have.class", "disabled");

// CONTENIDO DE TEXTO
cy.get(".title").should("contain", "Bienvenido"); // Contiene texto
cy.get(".title").should("have.text", "Bienvenido"); // Texto exacto
cy.get(".error").should("include.text", "Campo requerido");

// VALORES DE INPUT
cy.get("input[name='email']").should("have.value", "test@example.com");
cy.get("input[name='password']").should("have.value", ""); // Vacío

// ATRIBUTOS
cy.get("img").should("have.attr", "src", "/logo.png");
cy.get("a[href]").should("have.attr", "href"); // Tiene atributo
cy.get("a").should("have.attr", "href").and("include", "/users"); // Encadenado

// ESTADO DE ELEMENTOS
cy.get("input").should("be.enabled"); // Input habilitado
cy.get("button").should("be.disabled"); // Botón deshabilitado
cy.get("input[type='checkbox']").should("be.checked"); // Checkbox marcado
cy.get("input[type='checkbox']").should("not.be.checked"); // No marcado
cy.get("input").should("be.focused"); // Tiene focus

// LONGITUD (cantidad de elementos)
cy.get(".user-list li").should("have.length", 5); // Exactamente 5
cy.get(".user-list li").should("have.length.greaterThan", 3);
cy.get(".user-list li").should("have.length.lessThan", 10);

// URL Y NAVEGACIÓN
cy.url().should("include", "/dashboard"); // URL contiene
cy.url().should("eq", "http://localhost:4200/users"); // URL exacta
cy.location("pathname").should("eq", "/users"); // Solo pathname
cy.location("search").should("include", "sort=asc"); // Query string

// MÚLTIPLES ASERCIONES (encadenado)
cy.get("button")
  .should("be.visible")
  .and("be.enabled")
  .and("have.text", "Guardar")
  .and("have.class", "primary");

// ASERCIONES CON RETINTOS (útil para condiciones dinámicas)
cy.get(".counter").should("have.text", "5"); // Reintenta hasta 4s
```

**Estrategia de aserciones:**

- Verifica el resultado visible (lo que ve el usuario)
- Encadena aserciones cuando sea posible para más claridad
- Cypress reintenta automáticamente aserciones hasta timeout (perfecto para async)
- No abuses de aserciones: cada test debe tener 3-5 aserciones principales

### B. Comandos avanzados

#### 1. Esperas y timing

Cypress automáticamente espera a que los elementos sean interactuables y las condiciones se cumplan. Sin embargo, en casos especiales puedes controlar explícitamente las esperas.

```ts
// ✅ MEJOR: Espera automática (recomendado)
// Cypress espera hasta 4s (timeout por defecto) a que el elemento esté listo
cy.get(".async-content");
cy.get("[data-testid='submit-btn']").should("be.visible");

// Espera de aserciones (Cypress reintenta la aserción hasta timeout)
cy.get(".counter").should("have.text", "5"); // Reintenta hasta 4s
cy.get(".loading").should("not.exist"); // Espera a que desaparezca

// ⚠️ Espera explícita (úsala solo cuando sea necesario)
cy.wait(1000); // Esperar 1 segundo (es una "espera dura")

// Esperar un request específico (por alias)
cy.intercept("GET", "/api/users").as("getUsers");
cy.visit("/users");
cy.wait("@getUsers"); // Espera a que termine este request

// Esperar múltiples requests
cy.wait("@getUsers", "@getCount"); // Espera ambos

// Esperar con validación
cy.wait("@getUsers").then((interception) => {
  expect(interception.response.statusCode).to.equal(200);
  expect(interception.response.body).to.have.length.greaterThan(0);
});
```

**Cuándo usar cada uno:**

- ✅ Aserciones: para condiciones que cambiarán (async, animaciones)
- ✅ `cy.wait("@alias")`: para esperar requests HTTP completados
- ❌ `cy.wait(1000)`: evita esperas duras, son frágiles e impredecibles

#### 2. Aliases y reusabilidad

Los aliases permiten guardar referencias a elementos o requests para reutilizarlos múltiples veces en el test.

```ts
// ALIASES PARA ELEMENTOS
// Guardar referencia a un elemento para usarlo después
cy.get("input[name='email']").as("emailInput");
cy.get("@emailInput").type("test@example.com");
cy.get("@emailInput").should("have.value", "test@example.com");

// Útil para elementos que se repiten
cy.get("[data-testid='form']").as("userForm");
cy.get("@userForm").find("input[name='name']").type("Juan");
cy.get("@userForm").find("input[name='email']").type("juan@example.com");
cy.get("@userForm").submit();

// ALIASES PARA REQUESTS (más común)
// Interceptar y nombrar un request
cy.intercept("GET", "/api/users").as("getUsers");
cy.intercept("POST", "/api/users").as("createUser");
cy.intercept("DELETE", "/api/users/*").as("deleteUser");

// Visitar página (esto dispara los requests)
cy.visit("/users");

// Esperar que el request se complete
cy.wait("@getUsers");

// Verificar detalles del request y response
cy.wait("@createUser").then((interception) => {
  // Request: lo que se envió
  expect(interception.request.body).to.deep.equal({
    name: "Juan",
    email: "juan@example.com",
  });

  // Response: lo que se recibió
  expect(interception.response.statusCode).to.equal(201);
  expect(interception.response.body.id).to.exist;
});

// Usar alias en múltiples lugares
cy.wait("@getUsers");
cy.contains("Juan").should("be.visible");
cy.get("[data-testid='delete-btn']").click();
cy.wait("@deleteUser").its("response.statusCode").should("equal", 204);
```

**Ventajas de usar aliases:**

- Evita repetir selectores complejos
- Documenta qué requests esperas
- Permite verificar detalles del request/response

#### 3. Interceptar requests

La interceptación de requests permite mockear APIs externas, controlar respuestas y verificar que se realicen las llamadas correctas.

```ts
// MOCKEAR RESPUESTAS
// Usar datos de un fixture
cy.intercept("GET", "/api/users", { fixture: "users.json" }).as("getUsers");

// Respuesta inline
cy.intercept("GET", "/api/users", {
  statusCode: 200,
  body: [
    { id: 1, name: "Ana", email: "ana@example.com" },
    { id: 2, name: "Juan", email: "juan@example.com" },
  ],
}).as("getUsers");

// RESPUESTAS DINÁMICAS (basadas en el request)
cy.intercept("POST", "/api/users", (req) => {
  // Inspeccionar el request
  const body = req.body;

  // Responder con datos derivados del request
  req.reply({
    statusCode: 201,
    body: {
      id: Math.random(),
      ...body, // Incluye los datos que se enviaron
      createdAt: new Date().toISOString(),
    },
  });
}).as("createUser");

// SIMULAR ERRORES
cy.intercept("POST", "/api/users", (req) => {
  req.reply({
    statusCode: 400,
    body: {
      error: "Email ya existe",
      code: "DUPLICATE_EMAIL",
    },
  });
}).as("createUserError");

// SIMULAR PROBLEMAS DE RED
cy.intercept("GET", "/api/data", { forceNetworkError: true }); // Desconexión

// MODIFICAR REQUESTS
cy.intercept("POST", "/api/users", (req) => {
  // Modificar el body antes de enviarlo (si no está simulado)
  req.body.addedBy = "cypress-test";
  req.headers["X-Test"] = "true";

  // Continuar con la request modificada
  req.continue();
});

// MÚLTIPLES INTERCEPTACIONES
cy.intercept("GET", "/api/users", { fixture: "users.json" }).as("getUsers");
cy.intercept("GET", "/api/users/*/details", (req) => {
  req.reply({
    statusCode: 200,
    body: {
      /* detalles del usuario */
    },
  });
}).as("getUserDetails");

// VERIFICAR REQUESTS
cy.wait("@getUsers").then((interception) => {
  // Verificar que se realizo el request
  expect(interception.request.method).to.equal("GET");
  expect(interception.request.url).to.include("/api/users");

  // Verificar parámetros de query
  expect(interception.request.url).to.include("sort=name");

  // Verificar respuesta
  expect(interception.response.statusCode).to.equal(200);
  expect(interception.response.body).to.have.length(10);
});

// CASOS COMUNES
// 1. Mock de login
cy.intercept("POST", "/api/auth/login", {
  statusCode: 200,
  body: {
    token: "fake-jwt-token",
    user: { id: 1, name: "Test User", role: "admin" },
  },
}).as("login");

// 2. Mock de lista con paginación
cy.intercept("GET", "/api/users*", (req) => {
  const page = new URLSearchParams(req.url.split("?")[1]).get("page");
  req.reply({
    body: {
      data: [], // Los datos cambiarían según la página
      total: 100,
      page: parseInt(page) || 1,
    },
  });
}).as("getPagedUsers");
```

**Mejores prácticas:**

- Mockea APIs externas para tests independientes de la red
- Verifica que se realicen los requests correctos
- Usa fixtures para datos complejos o reutilizables

#### 4. Manejo de archivos y fixtures

Los fixtures son archivos de datos de prueba reutilizables. Cypress también permite subir archivos en formularios.

```ts
// FIXTURES - Cargar datos de prueba
// Crear archivo: cypress/fixtures/users.json
// Contenido:
// [
//   { id: 1, name: "Ana García", email: "ana@example.com" },
//   { id: 2, name: "Juan Pérez", email: "juan@example.com" }
// ]

// Cargar fixture en test
cy.fixture("users.json").then((users) => {
  // users es un array con los datos del archivo
  expect(users).to.have.length(2);
  expect(users[0].name).to.equal("Ana García");
});

// Usar fixture en interceptación
cy.intercept("GET", "/api/users", { fixture: "users.json" });
cy.visit("/users");
cy.contains("Ana García").should("be.visible");

// Fixtures con diferentes formatos
cy.fixture("data.json"); // JSON
cy.fixture("response.txt"); // Texto plano
// cy.fixture("image.png"); // Binarios

// SUBIR ARCHIVOS EN FORMULARIOS
// HTML:
// <input type="file" id="file-upload" />
// <button type="submit">Subir</button>

// Método 1: selectFile (recomendado en versiones recientes)
cy.get("input[type='file']").selectFile("cypress/fixtures/document.pdf");
cy.get("button[type='submit']").click();

// Método 2: selectFile con múltiples archivos
cy.get("input[type='file']").selectFile([
  "cypress/fixtures/file1.txt",
  "cypress/fixtures/file2.txt",
]);

// Verificar que el archivo fue seleccionado
cy.get("input[type='file']").should((input) => {
  expect(input[0].files.length).to.equal(1);
  expect(input[0].files[0].name).to.equal("document.pdf");
});

// CASOS COMUNES
// 1. Subir imagen de perfil
cy.get("[data-testid='profile-picture-input']").selectFile(
  "cypress/fixtures/avatar.jpg",
);
cy.get("[data-testid='save-profile']").click();
cy.get("[data-testid='profile-picture']").should("have.attr", "src");

// 2. Cargar datos de un fixture para verificar
cy.fixture("products.json").then((products) => {
  products.forEach((product) => {
    cy.contains(product.name).should("exist");
  });
});

// 3. Crear fixture dinámicamente en test
const testUser = {
  id: 1,
  name: "Test User",
  email: "test@example.com",
};

// Usar como si fuera un fixture
cy.wrap(testUser).then((user) => {
  cy.get("input[name='email']").type(user.email);
});
```

**Estructura recomendada de fixtures:**

```plain
cypress/
├── fixtures/
│   ├── users.json
│   ├── products.json
│   ├── error-responses.json
│   ├── document.pdf
│   └── avatar.jpg
```

**Buenas prácticas:**

- Usa fixtures para datos complejos que se reutilizan
- Mantén fixtures actualizadas con cambios en la API
- Crea fixtures específicas para casos de error

### C. Patrones de prueba E2E

#### Patrón AAA (Arrange-Act-Assert) en E2E

Para mantener tests claros y estructurados, el patrón AAA es muy útil:

```ts
describe("User Management", () => {
  beforeEach(() => {
    // Arrange: Preparar estado inicial
    cy.visit("/");
    cy.intercept("GET", "/api/users", { fixture: "users.json" });
  });

  it("debe crear un nuevo usuario", () => {
    // Act: Interactuar con la aplicación
    cy.get("[data-testid='add-user-btn']").click();
    cy.get("input[name='name']").type("Juan Pérez");
    cy.get("input[name='email']").type("juan@example.com");
    cy.get("button[type='submit']").click();

    // Assert: Verificar resultado
    cy.contains("Usuario creado exitosamente").should("be.visible");
    cy.get(".user-list").should("contain", "Juan Pérez");
  });
});
```

#### Patrón Page Object

El patrón Page Object ayuda a abstraer la lógica de interacción con la UI, haciendo los tests más legibles y mantenibles.

```ts
// cypress/support/page-objects/LoginPage.ts
export class LoginPage {
  visit() {
    cy.visit("/login");
    return this;
  }

  fillEmail(email: string) {
    cy.get("input[name='email']").type(email);
    return this;
  }

  fillPassword(password: string) {
    cy.get("input[name='password']").type(password);
    return this;
  }

  submit() {
    cy.get("button[type='submit']").click();
    return this;
  }

  login(email: string, password: string) {
    return this.fillEmail(email).fillPassword(password).submit();
  }
}
```

```ts
// Uso en test
import { LoginPage } from "../support/page-objects/LoginPage";

it("debe loguear usuario correctamente", () => {
  const loginPage = new LoginPage();
  loginPage.visit().login("user@example.com", "password123");

  cy.url().should("include", "/dashboard");
});
```

## Parte 3: Ejemplos detallados de pruebas E2E

### A. Ejemplo 1: Flujo de autenticación

El flujo de autenticación es crítico en cualquier aplicación. Este ejemplo demuestra cómo probar los tres escenarios principales: login exitoso, credenciales inválidas, y validación de campos requeridos.

```ts
describe("Authentication Flow", () => {
  beforeEach(() => {
    // Establecer punto de partida limpio antes de cada test
    cy.visit("/login");
  });

  it("debe loguear usuario con credenciales válidas", () => {
    // PASO 1: Interceptar la API de login antes de que se llame
    // Esto nos permite:
    // - Controlar la respuesta (no depender de servidor real)
    // - Verificar qué datos se envían
    // - Simular diferentes respuestas (éxito, error, timeout)
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 200, // HTTP 200 = éxito
      body: {
        token: "fake-jwt-token", // Token que se guardará en localStorage
        user: { id: 1, name: "Juan", role: "admin" }, // Datos del usuario
      },
    }).as("loginRequest"); // Nombramos la interceptación para esperar después

    // PASO 2: Llenar el formulario (simular acciones del usuario)
    cy.get("input[name='email']").type("juan@example.com"); // Escribir email
    cy.get("input[name='password']").type("password123"); // Escribir password
    cy.get("button[type='submit']").click(); // Hacer click en Enviar

    // PASO 3: Verificar que se envió el request correcto
    // Esto valida que el frontend envíe los datos esperados al backend
    cy.wait("@loginRequest") // Esperar a que complete el request
      .its("request.body") // Obtener el body del request que se envió
      .should("deep.equal", {
        // Verificar que los datos sean exactos
        email: "juan@example.com",
        password: "password123",
      });

    // PASO 4: Verificar redirección a dashboard
    // El browser debería redirigir automáticamente después del login
    cy.url().should("include", "/dashboard");

    // PASO 5: Verificar que el token se guardó en localStorage
    // Angular típicamente guarda el JWT en localStorage para requests futuros
    cy.window() // Acceder al objeto window del navegador
      .its("localStorage.token") // Obtener el valor del token
      .should("equal", "fake-jwt-token"); // Verificar que es el token correcto

    // ✅ Este test valida:
    // - Usuario puede escribir credenciales
    // - Request se envía con datos correctos
    // - API responde correctamente
    // - Usuario es redirigido después de login
    // - Token se almacena para requests futuros
  });

  it("debe mostrar error con credenciales inválidas", () => {
    // Interceptar la API pero simular error de autenticación
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 401, // HTTP 401 = Unauthorized (credenciales inválidas)
      body: { message: "Credenciales inválidas" }, // Mensaje de error
    });

    // Usuario intenta loguear con datos incorrectos
    cy.get("input[name='email']").type("wrong@example.com");
    cy.get("input[name='password']").type("wrongpass");
    cy.get("button[type='submit']").click();

    // Verificar que se muestra mensaje de error
    cy.contains("Credenciales inválidas").should("be.visible"); // El mensaje debe ser visible al usuario

    // ⚠️ IMPORTANTE: Verificar que NO redirigió
    // Si redirige a dashboard sin loguear = BUG crítico
    cy.url().should("include", "/login"); // Debe permanecer en página de login

    // ✅ Cambios vs test anterior:
    // - statusCode 401 en lugar de 200
    // - No se guarda token (porque falló)
    // - No redirige a dashboard
    // - Se muestra mensaje de error visible
  });

  it("debe validar campos requeridos", () => {
    // ⚠️ IMPORTANTE: Este es un test de VALIDACIÓN DEL FRONTEND
    // No hace request al servidor, solo verifica que Angular valide antes de enviar

    // Usuario hace click en Enviar sin llenar campos
    cy.get("button[type='submit']").click();

    // Verificar que Angular muestra errores de validación
    // (Angular agrega clase CSS "error" a campos inválidos)
    cy.get("input[name='email']").should("have.class", "error");
    cy.get("input[name='password']").should("have.class", "error");

    // Verificar mensajes de validación específicos
    cy.contains("Email es requerido").should("be.visible");
    cy.contains("Password es requerido").should("be.visible");

    // ✅ Este test valida:
    // - Formulario requiere email y password
    // - Mensajes de error son claros para usuario
    // - No permite enviar datos vacíos
    // - Validación ocurre en cliente (UX rápida)
  });
});
```

**Notas importantes sobre este ejemplo:**

1. **Interceptaciones diferentes**: Cada test intercepta la API diferente:
   - ✅ Test 1: statusCode 200 (éxito)
   - ❌ Test 2: statusCode 401 (error)
   - Test 3: No intercepta (valida cliente)

2. **AAA Pattern**: Cada test sigue Arrange-Act-Assert:
   - **Arrange**: `beforeEach()` y setup de intercept
   - **Act**: Llenar formulario y hacer click
   - **Assert**: Verificar URL, token, mensajes

3. **Verificaciones en 3 niveles**:
   - **Request**: Verificar QUÉ se envía al servidor
   - **Response**: Verificar QUÉ responde el servidor
   - **UI**: Verificar QUÉ ve el usuario

4. **Independencia**: Cada test es completamente independiente, `beforeEach()` reinicia en `/login`

5. **Anti-patrones evitados**:
   - ❌ NO usar `cy.wait(2000)` esperando que "eventualmente" ocurra algo
   - ❌ NO asumir que login exitoso ocurrió en otro test
   - ❌ NO verificar solo el DOM sin verificar requests

### B. Ejemplo 2: Gestión de TODOs (CRUD completo)

Este ejemplo es más complejo porque prueba operaciones CRUD (Create-Read-Update-Delete) completas en un solo test. Demuestra cómo un usuario típico interactuaría con una app de TODOs.

```ts
describe("Todo Management", () => {
  beforeEach(() => {
    // SETUP: Preparar estado limpio antes de cada test

    // Limpiar localStorage (remove tokens, preferences, etc)
    // Asegura que cada test comienza en estado "nuevo usuario"
    cy.clearLocalStorage();

    // Interceptar la llamada inicial que obtiene TODOs
    // Empezamos con lista vacía [] para un estado determinista
    cy.intercept("GET", "/api/todos", []).as("getTodos");

    // Visitar página inicial (donde se carga la lista de TODOs)
    cy.visit("/");

    // Esperar a que se complete la carga de TODOs
    // Sin esto, los tests serían frágiles (a veces pasa, a veces no)
    cy.wait("@getTodos");
  });

  it("debe crear, editar y eliminar un TODO", () => {
    // ===== OPERACIÓN CREATE =====
    // Escribir título del nuevo TODO
    // Nota: {enter} simula presionar la tecla Enter (typical para input de TODOs)
    cy.get("[data-testid='new-todo-input']").type("Aprender Cypress{enter}");

    // Interceptar la respuesta del POST (crear TODO)
    // El servidor devuelve el TODO creado con su ID asignado
    cy.intercept("POST", "/api/todos", {
      id: 1, // Server genera el ID
      title: "Aprender Cypress",
      completed: false, // Nuevo TODO siempre inicia como no completado
    }).as("createTodo");

    // Esperar a que se complete la creación
    cy.wait("@createTodo");

    // Verificar que el TODO aparece en la UI
    cy.contains("Aprender Cypress").should("be.visible");

    // ===== OPERACIÓN UPDATE (marcar completado) =====
    // Localizar el checkbox del TODO que creamos
    // (data-testid='todo-1' viene del ID que server asignó)
    cy.get("[data-testid='todo-1'] input[type='checkbox']").check();

    // Interceptar la actualización (PATCH = actualización parcial)
    cy.intercept("PATCH", "/api/todos/1", {
      id: 1,
      title: "Aprender Cypress",
      completed: true, // Ahora está completado
    }).as("updateTodo");

    // Esperar a que complete
    cy.wait("@updateTodo");

    // Verificar que se aplicó visualmente (Angular agrega clase CSS "completed")
    // Típicamente esto agrega strike-through text
    cy.get("[data-testid='todo-1']").should("have.class", "completed");

    // ===== OPERACIÓN UPDATE (editar título) =====
    // Hacer click en botón de editar
    cy.get("[data-testid='todo-1'] [data-testid='edit-btn']").click();

    // Encontrar el input de edición y actualizar el texto
    cy.get("[data-testid='todo-1'] input[type='text']") // El input ahora es editable
      .clear() // Limpiar texto anterior
      .type("Aprender Cypress avanzado{enter}"); // Escribir nuevo texto y Enter

    // Interceptar el PATCH de actualización
    cy.intercept("PATCH", "/api/todos/1", {
      id: 1,
      title: "Aprender Cypress avanzado", // Título actualizado
      completed: true, // Sigue completado
    }).as("editTodo");

    // Esperar actualización
    cy.wait("@editTodo");

    // Verificar que el nuevo título aparece
    cy.contains("Aprender Cypress avanzado").should("be.visible");

    // ✅ Verificar que el título anterior NO aparece
    cy.contains("Aprender Cypress").should("not.exist");

    // ===== OPERACIÓN DELETE =====
    // Hacer click en botón de eliminar
    cy.get("[data-testid='todo-1'] [data-testid='delete-btn']").click();

    // Interceptar la respuesta del DELETE
    // El servidor devuelve {} vacío (confirmación de eliminación)
    cy.intercept("DELETE", "/api/todos/1", {}).as("deleteTodo");

    // Esperar a que complete la eliminación
    cy.wait("@deleteTodo");

    // ✅ VERIFICACIÓN FINAL: El TODO debe desaparecer completamente
    cy.contains("Aprender Cypress avanzado").should("not.exist");

    // ✅ Este test demuestra:
    // - Crear TODO (POST)
    // - Marcar como completado (PATCH - cambio booleano)
    // - Editar título (PATCH - cambio de string)
    // - Eliminar TODO (DELETE)
    // - Verificar que cada operación actualiza la UI correctamente
  });

  it("debe filtrar TODOs por estado", () => {
    // ===== SETUP: Multiple TODOs para probar filtros =====

    // Interceptar GET para devolver 3 TODOs con diferentes estados
    cy.intercept("GET", "/api/todos", [
      { id: 1, title: "TODO 1", completed: false }, // Activo
      { id: 2, title: "TODO 2", completed: true }, // Completado
      { id: 3, title: "TODO 3", completed: false }, // Activo
    ]);

    // Recargar página para obtener los nuevos datos
    cy.reload();
    cy.wait("@getTodos"); // Esperar que la carga complete

    // ===== PRUEBA 1: Filtrar por ACTIVOS (no completados) =====
    cy.get("[data-testid='filter-active']").click();

    // Verificar cantidad correcta (2 activos: TODO 1 y TODO 3)
    cy.get("[data-testid='todo-list'] li").should("have.length", 2);

    // Verificar que se muestran los correctos
    cy.contains("TODO 1").should("be.visible");
    cy.contains("TODO 3").should("be.visible"); // Nota: puede no estar visible en output

    // ✅ Verificar que el completado está OCULTO
    cy.contains("TODO 2").should("not.exist"); // Importante!

    // ===== PRUEBA 2: Filtrar por COMPLETADOS =====
    cy.get("[data-testid='filter-completed']").click();

    // Verificar cantidad (1 completado: TODO 2)
    cy.get("[data-testid='todo-list'] li").should("have.length", 1);

    // Verificar que se muestra el completado
    cy.contains("TODO 2").should("be.visible");

    // ✅ Verificar que los activos están ocultos
    cy.contains("TODO 1").should("not.exist");

    // ===== PRUEBA 3: Mostrar TODOS (sin filtro) =====
    cy.get("[data-testid='filter-all']").click();

    // Verificar que se muestran los 3
    cy.get("[data-testid='todo-list'] li").should("have.length", 3);

    // ✅ Este test demuestra:
    // - Los filtros funcionan correctamente
    // - Cada estado de filtro muestra elementos adecuados
    // - El conteo es exacto
    // - Los elementos ocultos no "fantasmas" en el DOM
  });
});
```

**Patrones importantes en este ejemplo:**

1. **Múltiples interceptaciones**: No interceptas una sola vez:

   ```ts
   // ❌ INCORRECTO: Interceptar al inicio (se usa solo una vez)
   cy.intercept("POST", "/api/todos", {...}).as("createTodo");
   cy.visit("/");

   // ✅ CORRECTO: Interceptar justo antes de que ocurra (o en beforeEach)
   cy.get("input").type("...");
   cy.intercept("POST", "/api/todos", {...}).as("createTodo");
   cy.get("button").click(); // Esto dispara el POST
   cy.wait("@createTodo");
   ```

2. **Verificaciones encadenadas**: Después de cada operación, verifica:
   - Que el request se hizo correctamente
   - Que la UI se actualizó
   - Que los datos son correctos

3. **Uso de data-testid**: Más estable que clases CSS:
   - ✅ `[data-testid='todo-1']` - No cambia si refactorizar CSS
   - ❌ `[data-testid='todo-1'] .completed-item` - Frágil

4. **Estados deterministas**: Cada test parte de estado conocido (lista vacía) gracias a `beforeEach()`

### C. Ejemplo 3: Manejo de errores y edge cases

Los tests de error son críticos pero a menudo ignorados. Este ejemplo demuestra cómo probar situaciones reales de fallo: desconexiones, timeouts, y confirmaciones del usuario.

```ts
describe("Error Handling", () => {
  it("debe manejar desconexión de red", () => {
    // ===== SIMULACIÓN DE ERROR DE RED =====
    // En la vida real, Internet puede fallar:
    // - Usuario sin WiFi
    // - Servidor inaccesible
    // - DNS no responde
    // Cypress simula esto con forceNetworkError

    cy.intercept("GET", "/api/data", {
      forceNetworkError: true, // Simula conexión rechazada
    });

    // Visitar página que hace request al cargar
    cy.visit("/dashboard");

    // Angular debería mostrar mensaje de error al usuario
    cy.contains("Error de conexión").should("be.visible"); // Error visible = UX buena

    // ✅ IMPORTANTE: Debe haber botón de Reintentar
    // No dejar usuario sin opciones después de error
    cy.get("[data-testid='retry-btn']").should("be.visible");

    // ✅ Este test valida:
    // - Aplicación no se rompe con errores de red
    // - Usuario ve mensaje claro de qué sucedió
    // - Usuario puede reintentar la operación
    // - No aparecen errors en consola (stack trace al usuario = malo)
  });

  it("debe manejar timeouts de API", () => {
    // ===== SIMULACIÓN DE API LENTA =====
    // A veces el servidor es lento:
    // - Base de datos responde lentamente
    // - Servidor bajo alta carga
    // - Computación pesada en servidor

    cy.intercept("GET", "/api/slow-endpoint", (req) => {
      // req.reply permite controlar totalmente la respuesta
      req.reply((res) => {
        // Simular delay de 10 segundos
        // (Cypress timeout por defecto es 4s, así que fallará si no aumentamos)
        setTimeout(() => {
          res.send({ data: "ok" }); // Enviar respuesta después del delay
        }, 10000); // 10 segundos = 10,000 ms
      });
    });

    // Visitar página que dispara request lento
    cy.visit("/slow-page");

    // Mientras carga, usuario debería ver indicador de carga
    cy.contains("Cargando...").should("be.visible");

    // ⚠️ IMPORTANTE: Aumentar timeout para este comando
    // Cypress espera máximo 4 segundos por defecto
    // Pero nuestro request dura 10 segundos
    cy.contains("Datos cargados", { timeout: 15000 }) // 15 segundos = espacio suficiente
      .should("be.visible"); // Después que se complete, aparecer datos

    // ✅ Este test valida:
    // - Aplicación muestra indicador de carga (UX responsive)
    // - Aplicación aguanta requests lentos
    // - Usuario ve datos cuando finalmente llegan
    // - Timeout configurable por comando (flexibilidad)

    // ⚠️ ANTI-PATRÓN: NO hacer esto
    // ❌ cy.wait(15000); // Espera dura de 15 segundos
    // ✅ cy.contains(..., { timeout: 15000 }) // Espera inteligente
  });

  it("debe manejar navegación con datos no guardados", () => {
    // ===== CONFIRMACIÓN DE NAVEGACIÓN =====
    // Patrón común: Prevenir que usuario pierda datos
    // Si está editando un formulario sin guardar y intenta navegar,
    // mostrar "¿Está seguro? Perderá sus cambios"

    // Visitar página con formulario editable
    cy.visit("/form");

    // Usuario escribe datos pero NO hace click en Guardar
    cy.get("input[name='title']").type("Título no guardado");

    // Usuario intenta navegar a otra página
    // (haciendo click en enlace de navegación)
    cy.get("a[href='/other-page']").click();

    // Angular intercepta la navegación y muestra dialog de confirmación
    // Usamos cy.on() para capturar y responder al dialog
    cy.on("window:confirm", (text) => {
      // Verificar que el mensaje es correcto
      // (para saber que se ejecutó la lógica esperada)
      expect(text).to.include("¿Guardar cambios?");

      // Simular que usuario hace click en "Cancelar" (return false)
      // Si retorna true = "Aceptar" y navega
      // Si retorna false = "Cancelar" y permanece
      return false; // Cancelar navegación
    });

    // ✅ VERIFICACIÓN: Usuario debe permanecer en página de form
    // (porque canceló la navegación)
    cy.url().should("include", "/form");

    // ✅ Datos que escribió deben permanecer
    cy.get("input[name='title']").should("have.value", "Título no guardado");

    // ===== VARIACIÓN: Guardar cambios y luego navegar =====
    // Esto es más complejo en E2E porque requiere:
    // 1. Click en Guardar
    // 2. Esperar POST/PUT al servidor
    // 3. Luego intentar navegar
    // 4. Verificar que navega SIN mostrar dialog

    cy.get("button[type='submit']").click(); // Guardar datos
    cy.intercept("POST", "/api/form", { success: true }).as("saveForm");
    cy.wait("@saveForm");

    // Ahora si intenta navegar, NO debería mostrar confirmación
    cy.get("a[href='/other-page']").click();

    // ✅ Debería navegar directamente (sin dialog)
    cy.url().should("include", "/other-page");

    // ✅ Este test valida:
    // - Confirmación de navegación protege datos del usuario
    // - Mensaje es claro y específico
    // - Usuario puede optar por guardar o descartar
    // - Después de guardar, no repite la confirmación
  });

  it("debe manejar errores de validación del servidor", () => {
    // ===== VALIDACIÓN EN SERVIDOR =====
    // A veces validación falla en servidor (no solo cliente):
    // - Email ya existe
    // - Constraint de BD violado
    // - Regla de negocio no cumplida

    cy.visit("/register");

    // Llenar formulario
    cy.get("input[name='email']").type("existing@example.com");
    cy.get("input[name='password']").type("password123");
    cy.get("button[type='submit']").click();

    // Interceptar error del servidor
    cy.intercept("POST", "/api/auth/register", {
      statusCode: 400, // Bad request
      body: {
        error: "Email ya está registrado",
        field: "email", // Campo específico
      },
    }).as("registerError");

    cy.wait("@registerError");

    // Verificar que se muestra error específico
    cy.contains("Email ya está registrado").should("be.visible");

    // ✅ Verificar que el campo se marca como error
    cy.get("input[name='email']").should("have.class", "error"); // CSS class para resaltado

    // ✅ Este test valida:
    // - Errores del servidor se muestran al usuario
    // - Errores no cierran la aplicación
    // - Usuario puede corregir y reintentar
  });
});
```

**Por qué estos tests son cruciales:**

1. **Errores reales ocurren**: No es "si" sino "cuándo" falla la red
2. **UX en errores**: Cómo manejamos errores define la experiencia
3. **Confirmaciones**: Protegen usuarios de perder datos accidentalmente
4. **Timeouts variables**: Diferentes servidores tienen diferentes velocidades

**Mejores prácticas para tests de error:**

```ts
// ✅ BUEN ERROR HANDLING
it("debe mostrar error y permitir reintentar", () => {
  cy.intercept("GET", "/api/data", { forceNetworkError: true }).as("initialError");
  cy.visit("/page");
  cy.wait("@initialError");
  cy.contains("Error de conexión").should("be.visible");

  // Cambiar interceptación para simular recovery
  cy.intercept("GET", "/api/data", { statusCode: 200, body: {...} }).as("success");

  // Usuario hace click en reintentar
  cy.get("[data-testid='retry-btn']").click();
  cy.wait("@success");
  cy.contains("Datos cargados").should("be.visible");
});
```

**Anti-patrones a evitar:**

- ❌ Ignorar errores en tests ("si funciona en happy path, funciona")
- ❌ Hacer esperas duras en lugar de esperar eventos
- ❌ No verificar mensajes de error específicos
- ❌ Asumir que servidor siempre responde rápido

## Parte 4: Mejores prácticas y debugging

### A. Estrategia de pruebas E2E

1. **Enfocarse en flujos críticos**: No probar todo, solo lo esencial.
2. **Datos de prueba consistentes**: Usar fixtures y seeds.
3. **Independencia**: Cada test debe ser autónomo.
4. **Paralelización**: Ejecutar tests en paralelo para velocidad.

### B. Debugging en Cypress

#### 1. Modo interactivo

```bash
npm run cy:open
```

- **Time Travel**: Ver snapshots de cada paso.
- **Console**: Acceder a logs del navegador.
- **Network**: Inspeccionar requests/responses.
- **Elements**: Ver estado del DOM.

#### 2. Screenshots y videos

Puedes configurar Cypress para tomar screenshots automáticamente en fallos o manualmente en cualquier paso.

```ts
// Tomar screenshot manual
cy.screenshot("estado-actual");

// En configuración global
export default defineConfig({
  e2e: {
    screenshotOnRunFailure: true,
    video: true,
  },
});
```

#### 3. Logs y debugging

```ts
// Logs en consola
cy.log("Paso actual: verificando formulario");

// Pausar ejecución
cy.pause();

// Debug en browser console
cy.window().then((win) => {
  console.log("Window object:", win);
});

// Ver estado de aplicación
cy.window()
  .its("store")
  .then((store) => {
    console.log("Redux state:", store.getState());
  });
```

### C. Anti-patrones a evitar

```ts
// ❌ ANTI-PATRÓN 1: Tests frágiles
it("debe hacer click en el botón azul", () => {
  cy.get(".btn-primary").click(); // ¿Qué pasa si cambia el CSS?
});

// ✅ MEJOR: Usar data-testid
it("debe guardar el formulario", () => {
  cy.get("[data-testid='save-btn']").click();
});
```

```ts
// ❌ ANTI-PATRÓN 2: Esperas fijas
it("debe cargar datos", () => {
  cy.visit("/page");
  cy.wait(3000); // Espera arbitraria
  cy.contains("Datos").should("be.visible");
});

// ✅ MEJOR: Esperas inteligentes
it("debe cargar datos", () => {
  cy.intercept("GET", "/api/data").as("getData");
  cy.visit("/page");
  cy.wait("@getData");
  cy.contains("Datos").should("be.visible");
});
```

```ts
// ❌ ANTI-PATRÓN 3: Tests que dependen del orden
describe("Suite dependiente", () => {
  it("test 1", () => {
    cy.get(".counter").should("contain", "0");
    cy.get(".increment").click();
  });

  it("test 2", () => {
    cy.get(".counter").should("contain", "1"); // Depende de test 1
  });
});

// ✅ MEJOR: Tests independientes
describe("Suite independiente", () => {
  it("debe incrementar contador", () => {
    cy.visit("/counter");
    cy.get(".counter").should("contain", "0");
    cy.get(".increment").click();
    cy.get(".counter").should("contain", "1");
  });

  it("debe decrementar contador", () => {
    cy.visit("/counter");
    cy.get(".counter").should("contain", "0");
    cy.get(".decrement").click();
    cy.get(".counter").should("contain", "-1");
  });
});
```

## Parte 5: Laboratorio práctico (25 min)

### A. Objetivo del laboratorio

Aplicar Cypress para crear pruebas E2E de una aplicación Angular completa, incluyendo autenticación, navegación y operaciones CRUD.

**Tiempo estimado**: 25 minutos  
**Dificultad**: Intermedia  
**Requisitos**: Angular CLI, Cypress instalado

### B. Ejercicio 1: Configuración del proyecto (5 min)

#### Paso 1: Crear aplicación Angular

```bash
ng new cypress-demo --routing --style=css
cd cypress-demo
```

#### Paso 2: Instalar Cypress

```bash
ng add @cypress/schematic
```

#### Paso 3: Configurar aplicación básica

Crear componentes básicos (login, dashboard, user-list) con formularios y navegación.

#### Paso 4: Ejecutar Cypress

```bash
npm run cy:open
```

### C. Ejercicio 2: Pruebas de autenticación (10 min)

#### Paso 1: Crear test de login exitoso

```ts
// cypress/e2e/auth.cy.ts
describe("Authentication", () => {
  beforeEach(() => {
    cy.visit("/login");
  });

  it("debe loguear usuario correctamente", () => {
    // Mock API response
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 200,
      body: {
        token: "fake-jwt-token",
        user: { id: 1, name: "Test User" },
      },
    }).as("loginRequest");

    // Fill form
    cy.get("input[name='email']").type("test@example.com");
    cy.get("input[name='password']").type("password123");
    cy.get("button[type='submit']").click();

    // Verify API call
    cy.wait("@loginRequest").its("request.body").should("deep.equal", {
      email: "test@example.com",
      password: "password123",
    });

    // Verify redirect
    cy.url().should("include", "/dashboard");

    // Verify token stored
    cy.window().its("localStorage.token").should("equal", "fake-jwt-token");
  });

  it("debe mostrar error con credenciales inválidas", () => {
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 401,
      body: { message: "Invalid credentials" },
    });

    cy.get("input[name='email']").type("wrong@example.com");
    cy.get("input[name='password']").type("wrongpass");
    cy.get("button[type='submit']").click();

    cy.contains("Invalid credentials").should("be.visible");
    cy.url().should("include", "/login");
  });
});
```

#### Paso 2: Ejecutar y verificar

```bash
npm run cy:run -- --spec cypress/e2e/auth.cy.ts
```

### D. Ejercicio 3: Pruebas de navegación y CRUD (10 min)

#### Paso 1: Crear test de gestión de usuarios

```ts
// cypress/e2e/user-management.cy.ts
describe("User Management", () => {
  beforeEach(() => {
    // Login first
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 200,
      body: { token: "fake-token", user: { id: 1, role: "admin" } },
    });
    cy.visit("/login");
    cy.get("input[name='email']").type("admin@example.com");
    cy.get("input[name='password']").type("admin123");
    cy.get("button[type='submit']").click();

    // Navigate to user management
    cy.visit("/users");
  });

  it("debe crear, editar y eliminar usuario", () => {
    // Create user
    cy.get("[data-testid='add-user-btn']").click();
    cy.get("input[name='name']").type("Juan Pérez");
    cy.get("input[name='email']").type("juan@example.com");
    cy.get("button[type='submit']").click();

    cy.intercept("POST", "/api/users", {
      id: 123,
      name: "Juan Pérez",
      email: "juan@example.com",
    }).as("createUser");

    cy.wait("@createUser");
    cy.contains("Juan Pérez").should("be.visible");

    // Edit user
    cy.get("[data-testid='edit-user-123']").click();
    cy.get("input[name='name']").clear().type("Juan P. Actualizado");
    cy.get("button[type='submit']").click();

    cy.intercept("PUT", "/api/users/123", {
      id: 123,
      name: "Juan P. Actualizado",
      email: "juan@example.com",
    }).as("updateUser");

    cy.wait("@updateUser");
    cy.contains("Juan P. Actualizado").should("be.visible");

    // Delete user
    cy.get("[data-testid='delete-user-123']").click();
    cy.get("[data-testid='confirm-delete']").click();

    cy.intercept("DELETE", "/api/users/123", {}).as("deleteUser");
    cy.wait("@deleteUser");

    cy.contains("Juan P. Actualizado").should("not.exist");
  });

  it("debe filtrar usuarios por nombre", () => {
    // Setup multiple users
    cy.intercept("GET", "/api/users", [
      { id: 1, name: "Ana García", email: "ana@example.com" },
      { id: 2, name: "Juan Pérez", email: "juan@example.com" },
      { id: 3, name: "María López", email: "maria@example.com" },
    ]);

    cy.reload();

    // Filter by name
    cy.get("input[name='search']").type("Juan");
    cy.get(".user-list li").should("have.length", 1);
    cy.contains("Juan Pérez").should("be.visible");
    cy.contains("Ana García").should("not.exist");
  });
});
```

#### Paso 2: Ejecutar tests completos

```bash
npm run cy:run
```

### E. Checklist de finalización

- ✅ **Configuración**: Cypress instalado y configurado
- ✅ **Tests de auth**: Login exitoso y manejo de errores
- ✅ **Tests de navegación**: Redirecciones y protección de rutas
- ✅ **Tests CRUD**: Crear, leer, actualizar, eliminar usuarios
- ✅ **Debugging**: Screenshots y videos generados
- ✅ **CI/CD**: Tests ejecutándose en pipeline

### F. Recursos adicionales

- 📖 [Documentación oficial de Cypress](https://docs.cypress.io)
- 📖 [Cypress con Angular](https://docs.cypress.io/guides/component-testing/angular/overview)
- 📖 [Best Practices](https://docs.cypress.io/guides/references/best-practices)
- 🛠️ [Cypress Dashboard](https://cloud.cypress.io) para CI/CD
- 📚 [Real World Testing with Cypress](https://www.cypress.io/blog/)

## Resumen final

Esta clase ofrece un flujo completo de aprendizaje para pruebas E2E con Cypress en Angular:

- **Teoría**: Fundamentos de E2E vs unitarias
- **Configuración**: Setup completo de Cypress
- **API**: Comandos, assertions, intercept
- **Ejemplos**: Flujos reales de autenticación y CRUD
- **Laboratorio**: Práctica hands-on con aplicación real
- **Best practices**: Estrategias para tests mantenibles

Al finalizar, los estudiantes podrán crear suites de pruebas E2E robustas que validen la experiencia completa del usuario.
