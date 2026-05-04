# Unidad 3 - Clase 2: Pruebas unitarias en Angular 21 con Vitest

- **Duración**: 2 horas
- **Objetivo**: Enseñar a construir pruebas unitarias efectivas en Angular 21 utilizando Vitest, con enfoque en servicios, componentes y reporte de cobertura.

## Introducción

Angular 21 y Vitest ofrecen una combinación moderna para pruebas unitarias veloces y confiables. En esta clase veremos cómo aislar la lógica de negocio, validar componentes y medir el impacto de las pruebas con cobertura.

### ¿Por qué Vitest en Angular?

- Ejecución rápida y feedback inmediato.
- Buen soporte para TypeScript y Vite.
- Integración sencilla con mocks, spies y coverage.
- Alternativa moderna a Jasmine/Karma para aplicaciones Angular.

## Parte 1: Teoría

### A. Fundamentos de pruebas unitarias en Angular

- Las pruebas unitarias validan unidades de código aisladas: servicios, componentes, pipes y directivas.
- Deben ser atómicas, deterministas y rápidas.
- En Angular, el objetivo es probar la lógica sin cargar la aplicación completa.
- Separamos:
  - pruebas de servicios: lógica de negocio y llamadas HTTP.
  - pruebas de componentes: interacción con la plantilla y comportamiento del DOM.

### B. Arquitectura de Vitest y configuración mínima

Vitest se configura con `vitest.config.ts` o dentro de `vite.config.ts`.

#### Ejemplo de configuración básica

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "jsdom",
    coverage: {
      reporter: ["text", "lcov"],
      reportsDirectory: "coverage",
      exclude: ["node_modules/", "test/"],
    },
  },
});
```

#### Dependencias comunes

- `vitest`
- `@testing-library/angular`
- `@angular/core-testing` (si aplica)
- `jsdom`
- `@types/jest` o definiciones necesarias para compatibilidad.

### C. Flujo de trabajo con Vitest

- `npx vitest run`: ejecutar pruebas una sola vez.
- `npx vitest watch`: ejecutar en modo observador.
- `npx vitest run --coverage`: generar reporte de cobertura.

### D. Estructura básica de una prueba: `describe()`, `it()`, `beforeEach()`, `afterEach()`

En Vitest (y en cualquier framework de testing moderno), las pruebas se organizan mediante funciones fundamentales:

#### 1. `describe()`: Agrupar pruebas relacionadas

```ts
describe("UserService", () => {
  // Todas las pruebas aquí están relacionadas con UserService
  // Los describe pueden estar anidados
});

describe("Autenticación", () => {
  describe("Login", () => {
    // Pruebas específicas de login
  });

  describe("Logout", () => {
    // Pruebas específicas de logout
  });
});
```

- Agrupa tests por tema para mejorar la legibilidad.
- El nombre debe describir qué se está probando.
- Los reportes mostrarán esta estructura jerárquica.

#### 2. `it()`: Definir una prueba individual

```ts
it("debe obtener usuarios correctamente", () => {
  // Código de la prueba
  expect(resultado).toBe(esperado);
});

// El primer argumento es el nombre (debe ser descriptivo)
// El segundo es la función con la lógica del test
```

- Cada `it()` representa una unidad de prueba individual.
- El nombre debe ser una frase completa que describa el comportamiento esperado.
- Si falla un `it()`, el framework reportará exactamente cuál falló.

#### 3. `beforeEach()`: Configuración antes de cada prueba

```ts
describe("UserService", () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    // Este código se ejecuta ANTES de cada it()
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService],
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("test 1", () => {
    /* usa service e httpMock */
  });
  it("test 2", () => {
    /* usa service e httpMock NUEVOS */
  });
  // Cada it() obtiene su propia instancia limpia de service e httpMock
});
```

- Se ejecuta antes de cada `it()` en el mismo `describe()`.
- Ideal para:
  - configurar el `TestBed`
  - inicializar variables comunes
  - preparar datos mock
- **Crítico para el aislamiento**: cada prueba comienza con un estado limpio.

#### 4. `afterEach()`: Limpieza después de cada prueba

```ts
describe("UserService", () => {
  let httpMock: HttpTestingController;

  beforeEach(() => {
    // ... configuración
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("test 1", () => {
    /* ... */
  });

  afterEach(() => {
    // Se ejecuta DESPUÉS de cada it()
    // Verificar que no hay peticiones HTTP sin respuesta
    httpMock.verify();
  });
});
```

- Se ejecuta después de cada `it()`.
- Ideal para:
  - verificar que no quedan recursos sin liberar
  - validar que no hay efectos secundarios
  - limpiar estado global si es necesario

#### 5. Ejemplo completo: Estructura típica de una suite de pruebas

```ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { UserService } from "./user.service";
import { vi } from "vitest";

describe("UserService - Suite de Pruebas", () => {
  // Variables compartidas por todos los tests
  let service: UserService;
  let httpMock: HttpTestingController;

  // Configuración inicial (se ejecuta una sola vez antes de todos los tests)
  // Nota: Usa beforeAll() si necesitas lógica pesada una sola vez
  // beforeAll(() => { /* ... */ });

  // Configuración antes de CADA test
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService],
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  // Agrupar pruebas por funcionalidad
  describe("getUsers()", () => {
    it("debe obtener usuarios correctamente", () => {
      const mockUsers = [{ id: 1, name: "Ana" }];

      service.getUsers().subscribe((users) => {
        expect(users).toEqual(mockUsers);
      });

      const req = httpMock.expectOne("/api/users");
      req.flush(mockUsers);
    });

    it("debe manejar errores HTTP", () => {
      service.getUsers().subscribe(
        () => fail("Debería fallar"),
        (error) => {
          expect(error.status).toBe(500);
        },
      );

      const req = httpMock.expectOne("/api/users");
      req.error(new ErrorEvent("Network error"), { status: 500 });
    });
  });

  describe("getUserById()", () => {
    it("debe obtener un usuario por ID", () => {
      const mockUser = { id: 42, name: "Luis" };

      service.getUserById(42).subscribe((user) => {
        expect(user).toEqual(mockUser);
      });

      const req = httpMock.expectOne("/api/users/42");
      req.flush(mockUser);
    });

    it("debe construir la URL correctamente", () => {
      service.getUserById(99);

      const req = httpMock.expectOne("/api/users/99");
      expect(req.request.method).toBe("GET");
      req.flush({});
    });
  });

  // Limpieza después de CADA test
  afterEach(() => {
    // Verificar que no hay peticiones sin respuesta
    httpMock.verify();
  });

  // Limpieza final (se ejecuta una sola vez después de todos los tests)
  // Nota: Usa afterAll() si necesitas liberar recursos globales
  // afterAll(() => { /* ... */ });
});
```

**Salida esperada del reporte:**

```plain
✓ UserService - Suite de Pruebas
  ✓ getUsers()
  ✓ debe obtener usuarios correctamente
  ✓ debe manejar errores HTTP
  ✓ getUserById()
  ✓ debe obtener un usuario por ID
  ✓ debe construir la URL correctamente

Tests: 5 passed
Duration: 120ms
```

#### 6. Orden de ejecución

```plain
beforeAll()  ← Una sola vez, al inicio
  ↓
beforeEach()  ← Antes de CADA it()
  ↓
it("test 1")  ← Primera prueba
  ↓
afterEach()  ← Después de CADA it()
  ↓
beforeEach()  ← De nuevo, antes del siguiente it()
  ↓
it("test 2")  ← Segunda prueba
  ↓
afterEach()  ← De nuevo, después
  ↓
... (más tests)
  ↓
afterAll()  ← Una sola vez, al final
```

#### 7. Buenas prácticas en la estructura

- ✅ **Anida `describe()`** para organizar tests relacionados.
- ✅ **Usa `beforeEach()` para aislamiento**: cada test comienza limpio.
- ✅ **Nombres descriptivos**: `describe()` e `it()` forman una frase legible.
- ❌ **Evita lógica en `beforeEach()`**: mantén la preparación simple y clara.
- ❌ **Evita compartir estado**: no dependas de que un test afecte al siguiente.

### E. Aserciones con `expect()`: El corazón de las pruebas

Las aserciones determinan si una prueba pasa o falla. Vitest ofrece un catálogo extenso de matchers para validar cualquier escenario.

#### 1. Comparación de valores

```ts
const resultado = 5 + 5;

expect(resultado).toBe(10); // ✅ Igual a
expect(resultado).not.toBe(11); // ✅ No es igual a
expect(resultado).toEqual(10); // ✅ Mismo valor (para objetos/arrays)
expect(resultado).toStrictEqual(10); // ✅ Mismo valor y tipo
expect([1, 2, 3]).toEqual([1, 2, 3]); // ✅ Arrays y objetos
```

**Diferencia entre `toBe()` y `toEqual()`:**

```ts
const obj1 = { id: 1, name: "Ana" };
const obj2 = { id: 1, name: "Ana" };

expect(obj1).toBe(obj2); // ❌ Falla: son objetos diferentes en memoria
expect(obj1).toEqual(obj2); // ✅ Pasa: tienen el mismo contenido
```

#### 2. Valores nulos y indefinidos

```ts
let variable = null;
let undefinedVar = undefined;
let definedVar = "algo";

expect(variable).toBeNull(); // ✅ Es null
expect(undefinedVar).toBeUndefined(); // ✅ Es undefined
expect(definedVar).toBeDefined(); // ✅ Está definido
expect(definedVar).not.toBeNull(); // ✅ No es null
```

#### 3. Valores booleanos

```ts
const isActive = true;
const isEmpty = false;

expect(isActive).toBe(true); // ✅ Es verdadero
expect(isEmpty).toBe(false); // ✅ Es falso
expect(isActive).toBeTruthy(); // ✅ Es truthy (más flexible)
expect(isEmpty).toBeFalsy(); // ✅ Es falsy (más flexible)
```

**Truthy vs Falsy**:

En JavaScript/TypeScript, al evaluar un valor en un contexto booleano (por ejemplo dentro de un `if` o en una aserción `toBeTruthy()`), el lenguaje convierte ese valor a `true` o `false`.

- Un valor **truthy** es cualquier valor que al convertirse a booleano produce `true`.
- Un valor **falsy** es cualquiera que al convertirse a booleano produce `false`.

**Valores falsy más comunes**:

- `false`
- `0` y `-0`
- `0n` (BigInt cero)
- `""` (string vacío)
- `null`
- `undefined`
- `NaN`

**Ejemplos en tests**:

```ts
expect(1).toBeTruthy(); // 1 es truthy
expect("texto").toBeTruthy(); // string no vacío es truthy
expect(0).toBeFalsy(); // 0 es falsy
expect("").toBeFalsy(); // string vacío es falsy
expect(null).toBeFalsy(); // null es falsy
expect(undefined).toBeFalsy(); // undefined es falsy
```

**Implicaciones para las pruebas unitarias**:

- `toBeTruthy()`/`toBeFalsy()` comprueban la conversión booleana (útil para verificar presencia/ausencia o estados generales).
- `toBe(true)`/`toBe(false)` exigen el booleano estricto y deben usarse cuando la función realmente devuelve un booleano.
- Recomendación: usar `toBeTruthy()` para chequear que un valor existe/no está vacío; usar `toBe(true)` para verificar booleans explícitos.

#### 4. Números

```ts
const valor = 3.14159;

expect(valor).toBeGreaterThan(3); // ✅ Mayor que
expect(valor).toBeGreaterThanOrEqual(3.14159); // ✅ Mayor o igual
expect(valor).toBeLessThan(4); // ✅ Menor que
expect(valor).toBeLessThanOrEqual(3.14159); // ✅ Menor o igual
expect(valor).toBeCloseTo(3.14); // ✅ Cercano a (útil para floats)
```

#### 5. Strings

```ts
const mensaje = "Hola Angular";

expect(mensaje).toContain("Angular"); // ✅ Contiene substring
expect(mensaje).toMatch(/angular/i); // ✅ Coincide con regex
expect(mensaje).toHaveLength(13); // ✅ Longitud exacta
expect(mensaje).toEqual("Hola Angular"); // ✅ String exacto
```

#### 6. Arrays

```ts
const usuarios = [
  { id: 1, name: "Ana" },
  { id: 2, name: "Luis" },
];

expect(usuarios).toContain({ id: 1, name: "Ana" }); // ✅ Contiene elemento
expect(usuarios).toHaveLength(2); // ✅ Longitud exacta
expect(usuarios).toEqual([
  { id: 1, name: "Ana" },
  { id: 2, name: "Luis" },
]); // ✅ Array exacto
```

#### 7. Excepciones

```ts
const dividir = (a, b) => {
  if (b === 0) throw new Error("División por cero");
  return a / b;
};

// Capturar la excepción
expect(() => dividir(10, 0)).toThrow(); // ✅ Lanza error
expect(() => dividir(10, 0)).toThrow("División por cero"); // ✅ Error específico
expect(() => dividir(10, 0)).toThrow(Error); // ✅ Tipo de error
expect(() => dividir(10, 2)).not.toThrow(); // ✅ No lanza error
```

#### 8. Promesas y Observables

```ts
// Promesas
const promise = Promise.resolve("éxito");

expect(promise).resolves.toBe("éxito"); // ✅ Resuelve con valor
expect(Promise.reject("error")).rejects.toBe("error"); // ✅ Rechaza con valor

// Asincrónico explícito
it("debe resolver la promesa", async () => {
  const result = await promise;
  expect(result).toBe("éxito");
});

// Observables (usando subscribe)
const observable = of({ id: 1, name: "Ana" });

observable.subscribe((user) => {
  expect(user.id).toBe(1);
  expect(user.name).toBe("Ana");
});
```

#### 9. Funciones mock

```ts
const mockFn = vi.fn();
mockFn("Ana", 25);
mockFn("Luis", 30);

expect(mockFn).toHaveBeenCalled(); // ✅ Fue llamada
expect(mockFn).toHaveBeenCalledTimes(2); // ✅ Exactamente 2 veces
expect(mockFn).toHaveBeenCalledWith("Ana", 25); // ✅ Con argumentos específicos
expect(mockFn).toHaveBeenLastCalledWith("Luis", 30); // ✅ Última llamada
expect(mockFn).not.toHaveBeenCalledWith("Carlos", 35); // ✅ No fue llamada así
```

#### 10. Verificación múltiple con `expect.all()` (o el equivalente en Vitest)

```ts
// Agrupar múltiples aserciones
it("debe validar todos los campos del usuario", () => {
  const user = { id: 1, name: "Ana", email: "ana@example.com" };

  // Ejecutar todas las aserciones, reportar todas las que fallan
  expect.assertions(3); // Verificar que hay exactamente 3 expects

  expect(user.id).toBe(1);
  expect(user.name).toBe("Ana");
  expect(user.email).toBe("ana@example.com");
});

// O con múltiples afirmaciones en una:
it("debe validar el usuario completo", () => {
  const user = { id: 1, name: "Ana", email: "ana@example.com" };

  expect(user).toMatchObject({
    id: expect.any(Number),
    name: expect.any(String),
    email: expect.stringContaining("@"),
  });
});
```

#### 11. Validaciones especializadas

```ts
// Cualquier tipo
expect(valor).toEqual(expect.any(Number)); // ✅ Es un número
expect(valor).toEqual(expect.any(String)); // ✅ Es un string
expect(array).toEqual(expect.arrayContaining([2])); // ✅ Array contiene 2

// Pattern matching
const user = { id: 1, name: "Ana", role: "admin" };
expect(user).toMatchObject({
  name: "Ana",
  role: expect.stringMatching(/admin|user/),
}); // ✅ Validación parcial con regex

// Snapshot (comparar con versión anterior guardada)
expect(usuario).toMatchSnapshot(); // ✅ Útil para estructuras complejas
```

#### 12. Tabla resumen: Matchers comunes en Vitest

| Matcher                  | Uso                               | Ejemplo                                    |
| ------------------------ | --------------------------------- | ------------------------------------------ |
| `toBe()`                 | Igualdad estricta (===)           | `expect(5).toBe(5)`                        |
| `toEqual()`              | Igualdad de valor                 | `expect({a:1}).toEqual({a:1})`             |
| `toStrictEqual()`        | Igualdad estricta incluyendo tipo | `expect("5").not.toStrictEqual(5)`         |
| `toBeNull()`             | Es null                           | `expect(null).toBeNull()`                  |
| `toBeUndefined()`        | Es undefined                      | `expect(undefined).toBeUndefined()`        |
| `toBeDefined()`          | No es undefined                   | `expect(x).toBeDefined()`                  |
| `toBeTruthy()`           | Es truthy                         | `expect(1).toBeTruthy()`                   |
| `toBeFalsy()`            | Es falsy                          | `expect(0).toBeFalsy()`                    |
| `toContain()`            | Contiene elemento                 | `expect([1,2,3]).toContain(2)`             |
| `toMatch()`              | Coincide con regex                | `expect("Ana").toMatch(/^A/)`              |
| `toThrow()`              | Lanza excepción                   | `expect(() => throw()).toThrow()`          |
| `toHaveBeenCalled()`     | Mock fue invocado                 | `expect(mock).toHaveBeenCalled()`          |
| `toHaveBeenCalledWith()` | Mock llamado con args             | `expect(mock).toHaveBeenCalledWith("Ana")` |
| `toHaveLength()`         | Longitud exacta                   | `expect("Ana").toHaveLength(3)`            |
| `toBeGreaterThan()`      | Mayor que                         | `expect(10).toBeGreaterThan(5)`            |
| `resolves`               | Promesa resuelta                  | `expect(promise).resolves.toBe("ok")`      |
| `rejects`                | Promesa rechazada                 | `expect(promise).rejects.toBe("error")`    |

#### 13. Anti-patrones en aserciones

```ts
// ❌ ANTI-PATRÓN 1: Aserciones vagas
it("debería funcionar", () => {
  expect(resultado).toBeDefined(); // ¿Qué esperas exactamente?
});

// ✅ CORRECTO: Aserciones específicas
it("debería retornar un usuario válido", () => {
  expect(resultado).toEqual({ id: 1, name: "Ana" });
});
```

```ts
// ❌ ANTI-PATRÓN 2: Múltiples razones para fallar
it("debería obtener y guardar usuarios", () => {
  service.getUsers();  // ¿Qué se valida aquí? Nada explícitamente
  expect(resultado).toBe(esperado); // Una sola afirmación, pero múltiples pasos
});

// ✅ CORRECTO: Tests independientes
it("debería obtener usuarios", () => {
  const result = service.getUsers();
  expect(result).toEqual([...]);
});

it("debería guardar usuario", () => {
  service.saveUser(usuario);
  expect(localStorage.getItem("user")).toBe(JSON.stringify(usuario));
});
```

```ts
// ❌ ANTI-PATRÓN 3: Confiar en orden de ejecución
it("test 1", () => {
  globalState = "valor"; // Guarda estado global
});

it("test 2", () => {
  expect(globalState).toBe("valor"); // Depende del test anterior ¡MAL!
});

// ✅ CORRECTO: Cada test es independiente
it("test 1", () => {
  let localState = "valor";
  expect(localState).toBe("valor");
});

it("test 2", () => {
  let localState = "valor";
  expect(localState).toBe("valor");
});
```

## Parte 2: Pruebas de servicios

### A. ¿Qué se prueba en un servicio?

En Angular, los servicios contienen la lógica de negocio, acceso a datos y orquestación. Una prueba unitaria de servicio debe validar:

- **Métodos públicos**: la interfaz que otros componentes o servicios consumen.
- **Reglas de negocio**: transformaciones, validaciones y decisiones de lógica.
- **Datos retornados**: el formato y contenido esperado sin depender de infraestructura real.
- **Interacción con dependencias**: cómo el servicio invoca `HttpClient`, otros servicios o APIs externas.
- **Manejo de errores y condiciones de borde**: comportamiento cuando falla una llamada HTTP, datos inválidos, etc.

### B. Aislamiento estratégico: Mocks y Stubs

En una prueba unitaria de servicio, debemos aislar completamente la lógica del servicio sin depender de la red o bases de datos reales.

| Término  | Qué es                                        | Cuándo usarlo                                                                |
| -------- | --------------------------------------------- | ---------------------------------------------------------------------------- |
| **Mock** | Objeto simulado que reemplaza una dependencia | Cuando el servicio depende de `HttpClient`, APIs o servicios externos        |
| **Stub** | Comportamiento programado del mock            | Para definir respuestas esperadas (`resolve()`, `reject()`, errores)         |
| **Spy**  | Envuelve un método real permitiendo verificar | Cuando necesitas inspeccionar cómo se llamó sin reemplazar la implementación |

### C. Herramientas clave

- `TestBed`: Configurar el inyector de dependencias para la prueba, similar a cómo Angular crea el contexto de aplicación.

  ```ts
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
    providers: [UserService, OtherDependency],
  });
  ```

  Sin `TestBed`, Angular no podría inyectar las dependencias del servicio bajo prueba.

- **`TestBed.inject()`**: Obtener una instancia del servicio configurado.

  ```ts
  const service = TestBed.inject(UserService);
  ```

- **`HttpClientTestingModule`**: Módulo que reemplaza `HttpClientModule` para simular peticiones HTTP sin conectar a servidores reales.

  ```ts
  // En la configuración de TestBed
  imports: [HttpClientTestingModule], // NO usamos HttpClientModule

  // Ahora, cuando el servicio hace:
  constructor(private http: HttpClient) {}
  getUsers() {
  return this.http.get('/api/users');
  }

  // Las peticiones son interceptadas, no van a Internet
  ```

- **`HttpTestingController`**: Herramienta para inspeccionar y simular respuestas HTTP. Verifica que las peticiones se hayan realizado correctamente.

  ```ts
  const httpMock = TestBed.inject(HttpTestingController);

  // Esperar una petición específica
  const req = httpMock.expectOne("/api/users");

  // Verificar que el método HTTP fue correcto
  expect(req.request.method).toBe("GET");

  // Simular respuesta exitosa
  req.flush([{ id: 1, name: "Ana" }]);

  // Verificar que no hay peticiones sin respuesta
  httpMock.verify();
  ```

- **`vi.spyOn()` y `vi.fn()`**: Crear mocks y spies con Vitest para servicios inyectados o funciones.

  ```ts
  // vi.fn(): Crear una función mock desde cero
  const mockFunction = vi.fn().mockReturnValue("resultado");
  expect(mockFunction()).toBe("resultado");

  // vi.spyOn(): Envolver una función o método existente
  const service = { getName: () => "Ana" };
  vi.spyOn(service, "getName").mockReturnValue("Luis");
  expect(service.getName()).toBe("Luis");

  // Verificar cómo fue invocado
  expect(mockFunction).toHaveBeenCalled();
  expect(mockFunction).toHaveBeenCalledWith("argumento");
  expect(mockFunction).toHaveBeenCalledTimes(2);
  ```

### D. Patrones de prueba: AAA (Arrange-Act-Assert)

Toda prueba de servicio debe seguir una estructura clara:

1. **Arrange (Preparar)**: Configurar el `TestBed`, inyectar el servicio, preparar datos mock.
2. **Act (Actuar)**: Invocar el método del servicio.
3. **Assert (Verificar)**: Validar el resultado y las interacciones con dependencias.

### E. Ejemplo detallado: Prueba de obtención de usuarios

```ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { UserService } from "./user.service";

describe("UserService", () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    // Arrange: Configurar el TestBed con dependencias simuladas
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService],
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("debe obtener usuarios correctamente cuando la API responde exitosamente", () => {
    // Arrange: Preparar datos mock
    const mockUsers = [
      { id: 1, name: "Ana", email: "ana@example.com" },
      { id: 2, name: "Luis", email: "luis@example.com" },
    ];

    // Act: Llamar al método del servicio
    const result = service.getUsers();

    // Assert: Verificar la petición HTTP
    const req = httpMock.expectOne("/api/users");
    expect(req.request.method).toBe("GET");

    // Simular respuesta exitosa
    req.flush(mockUsers);

    // Verificar que el observable devolvió los datos correctos
    result.subscribe((users) => {
      expect(users).toEqual(mockUsers);
      expect(users.length).toBe(2);
    });
  });

  it("debe manejar errores cuando la API falla", () => {
    // Arrange & Act
    const result = service.getUsers();

    // Assert: Simular error HTTP
    const req = httpMock.expectOne("/api/users");
    req.error(new ErrorEvent("Network error"), { status: 500 });

    // Verificar que el servicio propaga el error
    result.subscribe(
      () => fail("Debería haber lanzado error"),
      (error) => {
        expect(error.status).toBe(500);
      },
    );
  });

  it("debe construir la URL correcta con parámetros", () => {
    // Arrange & Act
    const result = service.getUserById(42);

    // Assert
    const req = httpMock.expectOne("/api/users/42");
    expect(req.request.method).toBe("GET");
    req.flush({ id: 42, name: "Ana" });
  });

  afterEach(() => {
    // Verificar que no hay peticiones HTTP sin respuesta
    httpMock.verify();
  });
});
```

### F. Ejemplo con servicios inyectados

Cuando un servicio depende de otro servicio (no HTTP), usamos `vi.spyOn()` o `vi.fn()`:

```ts
import { AuthService } from "./auth.service";
import { UserService } from "./user.service";
import { vi } from "vitest";

describe("UserService con AuthService", () => {
  let userService: UserService;
  let authService: AuthService;

  beforeEach(() => {
    // Arrange: Configurar TestBed con un mock de AuthService
    TestBed.configureTestingModule({
      providers: [
        UserService,
        {
          provide: AuthService,
          useValue: {
            getCurrentUser: vi.fn().mockReturnValue({ id: 1, role: "admin" }),
          },
        },
      ],
    });

    userService = TestBed.inject(UserService);
    authService = TestBed.inject(AuthService);
  });

  it("debe obtener usuarios solo si el usuario está autenticado", () => {
    // Act
    const users = userService.getSecuredUsers();

    // Assert
    expect(authService.getCurrentUser).toHaveBeenCalled();
    expect(users).toBeDefined();
  });

  it("debe lanzar error si el usuario no está autenticado", () => {
    // Arrange: Cambiar el mock para simular usuario no autenticado
    (authService.getCurrentUser as any).mockReturnValue(null);

    // Act & Assert
    expect(() => userService.getSecuredUsers()).toThrow("No autenticado");
  });
});
```

### G. Patrones de Stubbing (Programación de comportamiento mock)

Vitest ofrece múltiples formas de controlar el comportamiento de los mocks:

```ts
// 1. Respuesta simple
vi.fn().mockReturnValue({ id: 1 });

// 2. Respuesta asincrónica (Promesas)
vi.fn().mockResolvedValue({ id: 1 });

// 3. Respuesta consecutiva (múltiples llamadas devuelven diferentes valores)
vi.fn()
  .mockReturnValueOnce({ id: 1 })
  .mockReturnValueOnce({ id: 2 })
  .mockReturnValueOnce({ id: 3 });

// 4. Lanzar excepción
vi.fn().mockRejectedValue(new Error("API Error"));

// 5. Lógica personalizada
vi.fn().mockImplementation((id) => ({ id, name: `User ${id}` }));
```

### H. Verificación de comportamiento

Después de ejecutar el test, podemos verificar cómo se invocó un mock:

```ts
const mockFn = vi.fn();

// Verificar que fue llamado
expect(mockFn).toHaveBeenCalled();

// Verificar el número de veces
expect(mockFn).toHaveBeenCalledTimes(3);

// Verificar con qué argumentos
expect(mockFn).toHaveBeenCalledWith({ id: 1, name: "Ana" });

// Verificar que NO fue llamado
expect(mockFn).not.toHaveBeenCalled();
```

### I. Buenas prácticas en pruebas de servicios

1. **Determinismo Absoluto**: Nunca dependas de `Date.now()` o valores aleatorios dentro del test. Mockea todo lo externo.
2. **Una Razón para Fallar**: Un test debe fallar por una sola razón de negocio. No mezcles múltiples validaciones en un solo test.
3. **Nombres Descriptivos**: Usa nombres de método que expliquen la regla de negocio, no detalles técnicos.
   - ✅ `debe_obtener_usuarios_cuando_api_responde_exitosamente`
   - ❌ `test_getUsers_ok`
4. **Independencia**: Cada test debe preparar su propio estado. No dependas de orden de ejecución.
5. **Limpieza**: Usa `afterEach()` para verificar que no quedan peticiones HTTP sin respuesta (`httpMock.verify()`).

### J. Anti-patrones a evitar

```ts
// ❌ ANTI-PATRÓN 1: No simular completamente las dependencias
it("debería obtener datos", () => {
  const result = service.getUsers(); // ¿De dónde vienen los datos? Confusión.
});
```

```ts
// ❌ ANTI-PATRÓN 2: Mezclar lógica en el test
it("debería validar usuarios", () => {
  for (let i = 0; i < 10; i++) {
    const user = service.getUser(i);
    if (user) expect(user).toBeDefined(); // Lógica condicional
  }
});
```

```ts
// ❌ ANTI-PATRÓN 3: Depender de orden de ejecución
describe("suite de tests", () => {
  it("test 1", () => {
    testData = getInitialData();
  }); // Guarda estado global
  it("test 2", () => {
    expect(testData).toBeDefined();
  }); // Depende del test anterior
});
```

```ts
// ❌ ANTI-PATRÓN 4: Verificar detalles de infraestructura, no lógica
it("debería conectar a /api/users", () => {
  service.getUsers();
  expect(httpClient.request).toHaveBeenCalled(); // Verificar HTTP es correcto, pero...
  // ...no estamos validando que los datos se transforman correctamente
});
```

## Parte 3: Pruebas de componentes (40 min)

### A. Objetivos de la parte

- Entender cómo montar un componente en tests con `TestBed`.
- Validar bindings: signals `input()` y `output()`.
- Simular eventos DOM y verificar la plantilla.
- Probar interacciones con servicios mediante spies y mocks.

### B. Herramientas clave

1. **`ComponentFixture<T>`: Contenedor de prueba del componente**: Instancia que encapsula el componente y permite manipularlo en el test. Proporciona acceso a:
   - `componentInstance`: la instancia real del componente.
   - `nativeElement`: el elemento DOM raíz renderizado.
   - `debugElement`: wrapper de Angular para consultas avanzadas de DOM.

   ```ts
   describe("MyComponent", () => {
     let fixture: ComponentFixture<MyComponent>;
     let component: MyComponent;

     beforeEach(async () => {
       await TestBed.configureTestingModule({
         declarations: [MyComponent],
       }).compileComponents();

       fixture = TestBed.createComponent(MyComponent);
       component = fixture.componentInstance;
     });

     it("accede al componente y su DOM", () => {
       // Acceder a la instancia del componente
       expect(component).toBeTruthy();

       // Acceder al elemento DOM raíz
       const rootElement = fixture.nativeElement;
       expect(rootElement.querySelector("h1")).toBeTruthy();
     });
   });
   ```

2. **`TestBed.configureTestingModule()` + `TestBed.createComponent()`**:
   - `configureTestingModule()` configura el inyector de Angular para la prueba, similar a cómo Angular bootstraps la aplicación.
   - `createComponent()` monta el componente en ese contexto.

   ```ts
   beforeEach(async () => {
     await TestBed.configureTestingModule({
       declarations: [MyComponent, ChildComponent], // Componentes a probar
       imports: [HttpClientTestingModule, ReactiveFormsModule], // Módulos necesarios
       providers: [UserService], // Servicios inyectables
     }).compileComponents(); // Compila plantillas y estilos CSS externos

     fixture = TestBed.createComponent(MyComponent); // Crea la instancia
   });
   ```

3. **`fixture.detectChanges()` y `fixture.whenStable()`: Control de detección de cambios**:
   - `detectChanges()`: Ejecuta la detección de cambios Angular (OnInit, property binding updates, etc.). Debe llamarse explícitamente antes de asserts sobre la plantilla.
   - `whenStable()`: Espera hasta que todas las tareas asíncronas pendientes (Promesas, timers) se resuelvan. Devuelve una Promesa.

   ```ts
   it("renderiza el título después de detectChanges", () => {
     component.title = "Mi Título";
     fixture.detectChanges(); // Aplica el binding
     const heading = fixture.nativeElement.querySelector("h1");
     expect(heading.textContent).toContain("Mi Título");
   });

   it("espera operaciones asíncronas antes de verificar", async () => {
     component.loadData(); // Inicia una petición HTTP
     fixture.detectChanges();
     await fixture.whenStable(); // Espera a que termine
     fixture.detectChanges(); // Re-renderiza con los nuevos datos
     expect(component.data).toBeTruthy();
   });
   ```

4. **`By.css()` / `debugElement.query()`: Selección de elementos DOM**: Utilidades de Angular para localizar elementos en la plantilla sin depender de selectores frágiles.

   ```ts
   import { By } from "@angular/platform-browser";

   it("selecciona elementos del DOM", () => {
     fixture.detectChanges();

     // Buscar un elemento por selector CSS
     const buttonDebugElement = fixture.debugElement.query(
       By.css("button.submit"),
     );
     const buttonElement = buttonDebugElement.nativeElement;
     expect(buttonElement.textContent).toContain("Guardar");

     // Buscar por directiva
     const childComponentDebugElement = fixture.debugElement.query(
       By.directive(ChildComponent),
     );
     expect(childComponentDebugElement.componentInstance).toBeTruthy();

     // QueryAll para múltiples elementos
     const allButtons = fixture.debugElement.queryAll(By.css("button"));
     expect(allButtons.length).toBe(3);
   });
   ```

5. **`vi.spyOn()` / `vi.fn()`: Mocks y spies**: Herramientas de Vitest para interceptar llamadas a métodos y crear funciones mock.
   - `vi.fn()`: Crear una función mock desde cero.
   - `vi.spyOn(object, 'method')`: Envolver un método existente sin perder su implementación original (a menos que lo mockees).

   ```ts
   import { vi } from "vitest";

   it("usa vi.fn() para crear un mock de callback", () => {
     const handler = vi.fn(); // Mock function
     component.onSave = handler;

     const button = fixture.debugElement.query(By.css("button")).nativeElement;
     button.click();

     expect(handler).toHaveBeenCalled();
     expect(handler).toHaveBeenCalledTimes(1);
   });

   it("usa vi.spyOn() para espiar un servicio", () => {
     const service = TestBed.inject(UserService);
     vi.spyOn(service, "getUser").mockResolvedValue({ id: 1, name: "Ana" });

     component.ngOnInit(); // Llama al servicio
     fixture.detectChanges();

     expect(service.getUser).toHaveBeenCalledWith(1);
   });
   ```

6. **`NO_ERRORS_SCHEMA`: Shallow testing (pruebas superficiales)**: Permite ignorar componentes desconocidos o elementos no reconocidos. Útil para aislar un componente de sus hijos y acelerar las pruebas.

   ```ts
   import { NO_ERRORS_SCHEMA } from "@angular/core";

   beforeEach(async () => {
     await TestBed.configureTestingModule({
       declarations: [ParentComponent], // Solo probamos el padre
       schemas: [NO_ERRORS_SCHEMA], // Ignorar <app-child>, <custom-element>, etc.
     }).compileComponents();
   });

   it("prueba el padre sin compilar componentes hijos", () => {
     fixture.detectChanges();
     expect(component).toBeTruthy();
     // No fallará aunque <app-child> no esté declarado
   });
   ```

### C. Montaje básico y comprobación de plantilla

Este ejemplo demuestra cómo montar un componente simple en una prueba unitaria usando `TestBed`. Primero, configuramos el módulo de prueba declarando el componente y compilándolo. Luego, creamos una instancia del componente con `TestBed.createComponent()`, obtenemos su instancia y aplicamos la detección de cambios inicial con `fixture.detectChanges()`. Finalmente, verificamos que el componente se creó correctamente y que su plantilla renderiza el contenido esperado (en este caso, un título basado en una propiedad del componente).

```ts
import { TestBed } from "@angular/core/testing";
import { By } from "@angular/platform-browser";
import { ComponentFixture } from "@angular/core/testing";
import { HeroComponent } from "./hero.component";

describe("HeroComponent", () => {
  let fixture: ComponentFixture<HeroComponent>;
  let component: HeroComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [HeroComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(HeroComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("crea el componente y renderiza el título", () => {
    expect(component).toBeTruthy();
    const title = fixture.debugElement.query(By.css(".title")).nativeElement;
    expect(title.textContent).toContain(component.title);
  });
});
```

### D. Probar `input()` (signals)

Opciones:

- Asignar directamente al signal usando `component.inputSignal.set(value);` y `fixture.detectChanges()`.
- Usar un Host Test Component para simular binding desde el template padre; la sintaxis de binding sigue siendo `[prop]="value"`.

Ejemplo directo:

```ts
it("muestra el input() recibido (signal)", () => {
  // Si 'item' es un WritableSignal en el componente
  component.item.set({ id: 1, name: "Test" });
  fixture.detectChanges();

  const el = fixture.debugElement.query(By.css(".item-name")).nativeElement;
  expect(el.textContent).toContain("Test");
});
```

Ejemplo con Host:

```ts
@Component({ template: `<app-item [item]="item"></app-item>` })
class HostComponent {
  item = { id: 1, name: "Host" };
}

beforeEach(async () => {
  await TestBed.configureTestingModule({
    declarations: [ItemComponent, HostComponent],
  }).compileComponents();
  const hostFixture = TestBed.createComponent(HostComponent);
  hostFixture.detectChanges();
  const child = hostFixture.debugElement.query(By.directive(ItemComponent));
  // Si 'item' en el child es un Signal, leer su valor para la comprobación
  const value =
    child.componentInstance.item?.() ?? child.componentInstance.item;
  expect(value.name).toBe("Host");
});
```

### E. Probar `output()` (signals)

En Angular 21, `output()` es un signal que reemplaza a `@Output()` EventEmitter para emitir eventos desde el componente. El signal `output()` permite suscribirse a eventos y emitir valores de manera reactiva.

Para probar un `output()` signal:

- Suscríbete al signal en el test usando `component.outputSignal.subscribe(spy)`.
- Simula la interacción que dispara el evento (ej. click en un botón).
- Verifica que el spy haya sido llamado con `expect(spy).toHaveBeenCalled()`.

Ejemplo:

```ts
it("emite evento cuando se pulsa el botón", () => {
  const spy = vi.fn();
  component.clicked.subscribe(spy); // 'clicked' es un signal output()

  const btn = fixture.debugElement.query(By.css("button")).nativeElement;
  btn.click();
  fixture.detectChanges();

  expect(spy).toHaveBeenCalled();
});
```

En el componente, para emitir un valor: `this.clicked.emit(value);`. El signal maneja la emisión de manera reactiva, similar a EventEmitter pero integrado con el sistema de signals.

### F. Simular eventos DOM y formularios

- Usar `nativeElement.click()` o `dispatchEvent(new Event('input'))`.
- Para formularios, asignar valores a inputs, disparar `input`/`change` y enviar el formulario.

Ejemplo de input:

```ts
const input = fixture.debugElement.query(By.css("input")).nativeElement;
input.value = "hola";
input.dispatchEvent(new Event("input"));
fixture.detectChanges();
expect(component.form.value.name).toBe("hola");
```

### G. Componentes que usan servicios: spies y mocks

Inyecta el servicio real o un mock en `TestBed` y usa `vi.spyOn()` para controlar el comportamiento.

```ts
const service = TestBed.inject(HeroService);
vi.spyOn(service, "getHero").mockResolvedValue({ id: 1, name: "X" });

fixture.detectChanges();
await fixture.whenStable();
expect(service.getHero).toHaveBeenCalled();
```

Para Observables, mockear con `of()`:

```ts
vi.spyOn(service, "getHeroes").mockReturnValue(of([{ id: 1, name: "A" }]));
fixture.detectChanges();
```

### H. Async: `fixture.whenStable()` y `await` patterns

- Si el componente espera Promises o tareas micro/macro, usar `await fixture.whenStable()` antes de assertions.
- Alternativa: usar utilities como `fakeAsync` / `tick()` si se necesita controlar timers (no común con Vitest).

Ejemplo:

```ts
vi.spyOn(service, "getHero").mockResolvedValue({ id: 1, name: "Async" });
fixture.detectChanges();
await fixture.whenStable();
fixture.detectChanges();
expect(
  fixture.debugElement.query(By.css(".name")).nativeElement.textContent,
).toContain("Async");
```

### I. Shallow testing y `NO_ERRORS_SCHEMA`

- Para pruebas centradas en el componente sin compilar hijos, declarar `schemas: [NO_ERRORS_SCHEMA]` en `TestBed`.
- Útil para aislar, pero pierde garantías sobre la integración con componentes hijos.

```ts
TestBed.configureTestingModule({
  declarations: [ParentComponent],
  schemas: [NO_ERRORS_SCHEMA],
});
```

### J. Buenas prácticas y anti-patrones

- Buenas prácticas:
  - Testear comportamiento observable y efectos en la plantilla, no detalles privados.
  - Preferir pequeños tests independientes (AAA / GWT).
  - Mockear dependencias externas (servicios, APIs) y controlar sus respuestas.
  - Usar nombres de test descriptivos: `debería emitir evento al hacer click en Guardar`.

- Anti-patrones:
  - No interactuar con la DOM real (evitar acceso a servicios de plataforma en unit tests).
  - Probar demasiado estado interno (private properties) en lugar de salida observable.
  - Depender del orden de ejecución de tests.

### K. Lista rápida de comprobaciones al terminar una prueba de componente

- ¿Se renderiza lo esperado en la plantilla?
- ¿Se llamó a los servicios esperados (spies)?
- ¿Se emiten `output()` cuando corresponde?
- ¿Se manejan correctamente flujos async (await/whenStable)?
- ¿No quedan timers o peticiones sin controlar?

## Parte 4: Cobertura de código con Vitest (20 min)

### A. ¿Qué es la cobertura de código?

La cobertura de código mide qué porcentaje de tu código fuente es ejecutado durante las pruebas. Es una métrica útil para identificar áreas que no están siendo testadas, aunque **no es garantía de calidad**.

Vitest registra automáticamente qué líneas, funciones, ramas y declaraciones se ejecutan durante los tests y genera un reporte detallado.

### B. Tipos de cobertura

Existen cuatro dimensiones de cobertura que Vitest mide:

#### 1. **Statement Coverage (Cobertura de Sentencias)**

Mide el porcentaje de sentencias ejecutadas en el código.

```ts
// Ejemplo de servicio
export class UserService {
  constructor(private http: HttpClient) {}

  getUser(id: number) {
    return this.http.get(`/api/users/${id}`); // ← Sentencia 1
  }

  deleteUser(id: number) {
    return this.http.delete(`/api/users/${id}`); // ← Sentencia 2
  }
}

// Test que cubre solo getUser
it("debe obtener usuario", () => {
  service.getUser(1);
  // Statement coverage: 50% (solo se ejecutó la sentencia 1)
  // deleteUser nunca fue llamado
});
```

#### 2. **Branch Coverage (Cobertura de Ramas)**

Mide el porcentaje de caminos condicionales (if/else, switch, etc.) que se ejecutan.

```ts
export class AuthService {
  login(username: string, password: string): boolean {
    if (!username || !password) {
      // ← Rama 1: Si no hay usuario o contraseña
      throw new Error("Credenciales faltantes");
    }

    if (username === "admin" && password === "123") {
      // ← Rama 2: Si credenciales son correctas
      return true;
    }

    // ← Rama 3: Si credenciales son incorrectas
    return false;
  }
}

// Test que solo cubre el caso exitoso
it("debe loguear con credenciales correctas", () => {
  const result = AuthService.login("admin", "123");
  expect(result).toBe(true);
  // Branch coverage: 33% (solo rama 2)
  // Faltan pruebas para rama 1 (credenciales faltantes) y rama 3 (credenciales incorrectas)
});
```

#### 3. **Function Coverage (Cobertura de Funciones)**

Mide el porcentaje de funciones que fueron invocadas al menos una vez.

```ts
export class CalculatorService {
  add(a: number, b: number) {
    return a + b; // ← Función 1
  }

  subtract(a: number, b: number) {
    return a - b; // ← Función 2
  }

  multiply(a: number, b: number) {
    return a * b; // ← Función 3
  }
}

// Test que solo prueba add
it("debe sumar dos números", () => {
  expect(CalculatorService.add(2, 3)).toBe(5);
  // Function coverage: 33% (solo se llamó a add)
  // subtract y multiply nunca fueron invocadas
});
```

#### 4. **Line Coverage (Cobertura de Líneas)**

Mide el porcentaje de líneas de código que fueron ejecutadas.

```ts
export class DataProcessor {
  process(data: any[]) {
    if (!data || data.length === 0) {
      // ← Línea 1-3
      console.log("No hay datos");
      return null;
    }

    // ← Línea 5-7
    const result = data.map((item) => item * 2);
    return result;
  }
}

// Test que solo cubre el caso exitoso
it("debe procesar datos", () => {
  const result = DataProcessor.process([1, 2, 3]);
  expect(result).toEqual([2, 4, 6]);
  // Line coverage: ~50% (no se ejecutó el bloque de error)
});
```

**Tabla comparativa:**

| Tipo          | Qué mide                             | Útil para                                            |
| ------------- | ------------------------------------ | ---------------------------------------------------- |
| **Statement** | Sentencias individuales ejecutadas   | Identificar código muerto o no alcanzado             |
| **Branch**    | Caminos condicionales tomados        | Asegurar que if/else/switch se prueban completamente |
| **Function**  | Funciones invocadas al menos una vez | Detectar funciones nunca llamadas                    |
| **Line**      | Líneas de código ejecutadas          | Visión general de cobertura                          |

### C. Configuración de coverage en Vitest

Vitest usa la librería `v8` por defecto para medir cobertura. Configúralo en `vitest.config.ts`:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "jsdom",
    coverage: {
      // Proveedor de coverage (v8 es el recomendado)
      provider: "v8",

      // Reportes a generar
      reporter: [
        "text", // Imprime en terminal
        "text-summary", // Resumen en terminal
        "lcov", // Formato LCOV (compatible con editores y CI/CD)
        "html", // Reporte HTML interactivo
        "json", // Formato JSON para procesamiento automático
      ],

      // Directorio donde se guardarán los reportes
      reportsDirectory: "./coverage",

      // Archivos a excluir del análisis
      exclude: [
        "node_modules/",
        "dist/",
        "coverage/",
        "**/*.spec.ts", // Excluir archivos de test
        "**/index.ts", // Excluir barrel exports
      ],

      // Umbrales mínimos de cobertura (opcional)
      lines: 70, // Al menos 70% de cobertura de líneas
      functions: 70,
      branches: 70,
      statements: 70,

      // Forzar falla si no se cumplen los umbrales
      // Si enabled es true, el build falla si no se alcanzan los umbrales
      thresholds: {
        lines: 70,
        functions: 70,
        branches: 60, // Ramificaciones son más complejas, umbral más bajo
        statements: 70,
      },
    },
  },
});
```

### D. Comandos para generar reportes

#### Opción 1: Ejecutar una sola vez con coverage

```bash
npx vitest run --coverage
```

Salida esperada en terminal:

```plain
 ✓ src/services/user.service.spec.ts (5)
 ✓ src/components/user-list.component.spec.ts (8)

Test Files  2 passed (2)
  Tests  13 passed (13)
  Start at  15:32:40
  Duration  1.24s

———————————|—————————–|—————————–|—————————–|—————————–|
File | % Stmts | % Branch | % Funcs | % Lines |
———————————|—————————–|—————————–|—————————–|—————————–|
All files |  78.5 |  72.3 |  81.0 |  78.5 |
 user.service.ts |  85.0 |  80.0 |  90.0 |  85.0 |
 user-list.component |  72.0 |  65.0 |  72.0 |  72.0 |
———————————|—————————–|—————————–|—————————–|—————————–|
```

#### Opción 2: Modo watch con coverage

```bash
npx vitest watch --coverage
```

Ejecuta las pruebas de nuevo cada vez que guardas un archivo, actualizando el coverage en tiempo real.

#### Opción 3: Coverage solo de archivos específicos

```bash
npx vitest run --coverage -- src/services
```

### E. Interpretación del reporte

Vitest genera reportes en múltiples formatos. El más útil para desarrollo es el reporte HTML.

#### 1. Abrir el reporte HTML

Después de ejecutar `npx vitest run --coverage`, abre:

```plain
coverage/index.html
```

En tu navegador verás un dashboard con:

- Resumen global de cobertura (% Stmts, % Branch, % Funcs, % Lines).
- Lista de archivos con sus porcentajes individuales.
- Color coding: verde (>80%), amarillo (60-80%), rojo (<60%).

#### 2. Profundizar en un archivo

Haz clic en un archivo (ej. `user.service.ts`) para ver:

- Código fuente línea por línea.
- Líneas rojas: no fueron ejecutadas.
- Líneas amarillas: parcialmente ejecutadas (solo algunas ramas).
- Líneas verdes: completamente ejecutadas.

Ejemplo visual:

```ts
// ✅ Verde - Ejecutada
export class UserService {
  constructor(private http: HttpClient) {}

  // ✅ Verde - Ejecutada
  getUser(id: number) {
    return this.http.get(`/api/users/${id}`);
  }

  // ❌ Rojo - No ejecutada
  deleteUser(id: number) {
    return this.http.delete(`/api/users/${id}`);
  }

  // 🟡 Amarillo - Parcialmente ejecutada (solo rama 1 de 2)
  validateUser(user: any) {
    if (!user) {
      // ✅ Ejecutada
      throw new Error("Usuario inválido");
    }
    // ❌ No ejecutada
    return user.isActive;
  }
}
```

#### 3. Interpretación de umbrales

- **>80%**: Excelente cobertura, código bien testado.
- **70-80%**: Bueno, pero hay áreas con poca cobertura.
- **60-70%**: Aceptable, pero requiere mejora.
- **<60%**: Crítico, falta mucha cobertura.

### F. Estrategias para mejorar la cobertura

#### 1. Identificar código rojo (no ejecutado)

Revisa el reporte HTML y busca:

- Funciones completas no llamadas → Agrega tests que las invoquen.
- Bloques `if`/`else` no ejecutados → Crea casos de test para ambas ramas.
- Manejo de errores no probado → Simula fallos y verifica el comportamiento.

#### 2. Ejemplo: Mejorar cobertura de un servicio

Servicio sin tests:

```ts
export class PaymentService {
  constructor(private http: HttpClient) {}

  processPayment(amount: number, cardToken: string) {
    if (amount <= 0) {
      // ❌ No probado
      throw new Error("Monto inválido");
    }

    if (!cardToken) {
      // ❌ No probado
      throw new Error("Token de tarjeta requerido");
    }

    // ❌ No probado
    return this.http.post("/api/payments", { amount, cardToken });
  }

  refund(paymentId: string) {
    // ❌ No probado
    return this.http.post(`/api/payments/${paymentId}/refund`, {});
  }
}
```

Tests mejorados:

```ts
describe("PaymentService - Cobertura mejorada", () => {
  let service: PaymentService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [PaymentService],
    });

    service = TestBed.inject(PaymentService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  // Rama: monto válido
  it("debe procesar pago con monto válido", () => {
    service.processPayment(100, "token123").subscribe();
    const req = httpMock.expectOne("/api/payments");
    req.flush({ success: true });
  });

  // Rama: monto inválido ← Ahora probado
  it("debe lanzar error si el monto es inválido", () => {
    expect(() => service.processPayment(-50, "token123")).toThrow(
      "Monto inválido",
    );
  });

  // Rama: sin token ← Ahora probado
  it("debe lanzar error si no hay token", () => {
    expect(() => service.processPayment(100, "")).toThrow(
      "Token de tarjeta requerido",
    );
  });

  // Función refund ← Ahora probada
  it("debe procesar reembolso", () => {
    service.refund("pay_123").subscribe();
    const req = httpMock.expectOne("/api/payments/pay_123/refund");
    req.flush({ status: "refunded" });
  });

  afterEach(() => {
    httpMock.verify();
  });
});
```

**Impacto**: La cobertura de `PaymentService` pasa de ~10% a ~90%.

#### 3. Priorización

No todas las líneas de código valen igual. Prioriza:

1. **Lógica crítica**: Autenticación, pagos, validaciones.
2. **Lógica condicional**: if/else, switch, ternarios.
3. **Manejo de errores**: try/catch, validaciones.
4. **Getters/setters simples**: Menor prioridad si no tienen lógica.

### G. Buenas prácticas de cobertura

1. **Usa la cobertura como guía, no como objetivo**: Una cobertura del 100% puede esconder tests débiles.

   ```ts
   // ❌ Test débil que sube cobertura pero no valida nada
   it("debe llamar al servicio", () => {
     service.getUser(1); // Solo llama, no verifica nada
   });

   // ✅ Test fuerte que valida comportamiento
   it("debe obtener usuario y retornar datos correctos", () => {
     const result = service.getUser(1);
     result.subscribe((user) => {
       expect(user).toEqual({ id: 1, name: "Ana" });
     });
   });
   ```

2. **Cubre ramas, no solo líneas**: Una rama mal cubierta puede esconder bugs.

   ```ts
   // ❌ Solo cubre la rama "feliz"
   it("debe procesar datos", () => {
     const result = service.process([1, 2, 3]);
     expect(result).toEqual([2, 4, 6]);
   });

   // ✅ Cubre rama de error también
   it("debe procesar datos válidos", () => {
     const result = service.process([1, 2, 3]);
     expect(result).toEqual([2, 4, 6]);
   });

   it("debe lanzar error si no hay datos", () => {
     expect(() => service.process([])).toThrow();
   });
   ```

3. **Evita código que no pueda ser probado**: Si el código es difícil de testear, probablemente sea mal diseño.

   ```ts
   // ❌ Difícil de testear (dependencias acopladas)
   export class UserComponent {
     ngOnInit() {
       // HTTP y lógica mezcladas
       fetch("/api/users")
         .then((res) => res.json())
         .then((users) => {
           this.users = users.filter((u) => u.isActive);
         });
     }
   }

   // ✅ Fácil de testear (separación de concerns)
   export class UserComponent {
     constructor(private userService: UserService) {}

     ngOnInit() {
       this.userService.getActiveUsers().subscribe((users) => {
         this.users = users;
       });
     }
   }
   ```

4. **Configura umbrales realistas**: No todos los proyectos necesitan 100%.

   ```ts
   // Realista para un proyecto en desarrollo
   thresholds: {
    lines: 70,
    functions: 70,
    branches: 60, // Ramas son complejas, umbral más bajo
    statements: 70,
   }

   // Estricto para código crítico (pagos, autenticación)
   thresholds: {
    lines: 85,
    functions: 85,
    branches: 80,
    statements: 85,
   }
   ```

### H. Anti-patrones en cobertura

```ts
// ❌ ANTI-PATRÓN 1: Tests que solo suben cobertura
it("existe el servicio", () => {
  expect(service).toBeDefined(); // No valida comportamiento
});
```

```ts
// ❌ ANTI-PATRÓN 2: Ignorar código no testeable
// Archivo .spec.ts
/* v8 ignore next 5 */
export class UntestableClass {}
// Esto oculta el problema en lugar de solucionarlo
```

```ts
// ❌ ANTI-PATRÓN 3: Excluir demasiados archivos
coverage: {
  exclude: [
  "node_modules/",
  "src/**/*.mock.ts", // OK
  "src/**/*.component.ts", // MAL: Excluye componentes legítimos
  ],
}
```

```ts
// ❌ ANTI-PATRÓN 4: Umbrales imposibles
thresholds: {
  lines: 100,
  branches: 100,
  functions: 100,
  statements: 100,
}
// Causa fallos constantes y frustración en el equipo
```

### I. Integración con CI/CD

Asegura que el CI/CD falle si no se cumple la cobertura:

```bash
# En tu pipeline (GitHub Actions, GitLab CI, etc.)
npx vitest run --coverage

# Si falla, el pipeline se detiene
# (Si los umbrales en vitest.config.ts no se alcanzan)
```

Ejemplo GitHub Actions:

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
  runs-on: ubuntu-latest
  steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-node@v3
  with:
  node-version: "18"
  - run: npm install
  - run: npm run test:coverage
  # Si falla, la PR no puede ser mezclada (merge)
```

### J. Checklist final de cobertura

- ✅ Cobertura > 70% en el proyecto completo.
- ✅ Cobertura > 80% en código crítico (servicios, lógica de negocio).
- ✅ Todas las ramas condicionales probadas.
- ✅ Manejo de errores validado en tests.
- ✅ No hay archivos con <50% de cobertura.
- ✅ Los umbrales están ajustados al contexto del proyecto.
- ✅ El CI/CD valida cobertura antes de mezclar PRs.

## Parte 5: Laboratorio práctico (25 min)

### A. Objetivo del laboratorio

Aplicar en la práctica todo lo aprendido en las partes anteriores: crear pruebas unitarias reales para un servicio y componente, validar que funcionan correctamente, y analizar la cobertura generada.

**Tiempo estimado**: 25 minutos  
**Dificultad**: Intermedia  
**Requisitos**: Node.js, npm, Vitest configurado

### B. Ejercicio 1: Servicio REST con pruebas unitarias (10 min)

#### Paso 1: Crear el servicio

Crea un archivo `src/services/todo.service.ts`:

```ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable } from "rxjs";

export interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

@Injectable({ providedIn: "root" })
export class TodoService {
  private apiUrl = "/api/todos";

  constructor(private http: HttpClient) {}

  // Obtener todos los TODOs
  getTodos(): Observable<Todo[]> {
    return this.http.get<Todo[]>(this.apiUrl);
  }

  // Obtener un TODO por ID
  getTodoById(id: number): Observable<Todo> {
    return this.http.get<Todo>(`${this.apiUrl}/${id}`);
  }

  // Crear un nuevo TODO
  createTodo(title: string): Observable<Todo> {
    if (!title || title.trim().length === 0) {
      throw new Error("El título no puede estar vacío");
    }
    return this.http.post<Todo>(this.apiUrl, { title, completed: false });
  }

  // Actualizar un TODO
  updateTodo(id: number, completed: boolean): Observable<Todo> {
    if (id <= 0) {
      throw new Error("ID inválido");
    }
    return this.http.patch<Todo>(`${this.apiUrl}/${id}`, { completed });
  }

  // Eliminar un TODO
  deleteTodo(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

#### Paso 2: Crear el archivo de pruebas

Crea un archivo `src/services/todo.service.spec.ts`:

```ts
import { TestBed } from "@angular/core/testing";
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";
import { TodoService, Todo } from "./todo.service";
import { describe, it, beforeEach, afterEach, expect } from "vitest";

describe("TodoService", () => {
  let service: TodoService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [TodoService],
    });

    service = TestBed.inject(TodoService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  // ✅ PRUEBA 1: Obtener todos los TODOs
  it("debe obtener la lista de TODOs correctamente", () => {
    const mockTodos: Todo[] = [
      { id: 1, title: "Aprender Vitest", completed: false },
      { id: 2, title: "Hacer pruebas", completed: true },
    ];

    service.getTodos().subscribe((todos) => {
      expect(todos).toEqual(mockTodos);
      expect(todos.length).toBe(2);
    });

    const req = httpMock.expectOne("/api/todos");
    expect(req.request.method).toBe("GET");
    req.flush(mockTodos);
  });

  // ✅ PRUEBA 2: Manejar error al obtener TODOs
  it("debe manejar error cuando la API falla al obtener TODOs", () => {
    service.getTodos().subscribe(
      () => fail("Debería haber lanzado error"),
      (error) => {
        expect(error.status).toBe(500);
      },
    );

    const req = httpMock.expectOne("/api/todos");
    req.error(new ErrorEvent("Server error"), { status: 500 });
  });

  // ✅ PRUEBA 3: Obtener TODO por ID
  it("debe obtener un TODO específico por su ID", () => {
    const mockTodo: Todo = {
      id: 1,
      title: "Aprender Vitest",
      completed: false,
    };

    service.getTodoById(1).subscribe((todo) => {
      expect(todo).toEqual(mockTodo);
      expect(todo.id).toBe(1);
    });

    const req = httpMock.expectOne("/api/todos/1");
    expect(req.request.method).toBe("GET");
    req.flush(mockTodo);
  });

  // ✅ PRUEBA 4: Crear TODO con título válido
  it("debe crear un nuevo TODO con título válido", () => {
    const newTodo: Todo = { id: 3, title: "Nuevo TODO", completed: false };

    service.createTodo("Nuevo TODO").subscribe((todo) => {
      expect(todo).toEqual(newTodo);
      expect(todo.title).toBe("Nuevo TODO");
    });

    const req = httpMock.expectOne("/api/todos");
    expect(req.request.method).toBe("POST");
    expect(req.request.body).toEqual({ title: "Nuevo TODO", completed: false });
    req.flush(newTodo);
  });

  // ✅ PRUEBA 5: Lanzar error si título está vacío
  it("debe lanzar error si el título está vacío", () => {
    expect(() => service.createTodo("")).toThrow(
      "El título no puede estar vacío",
    );
  });

  // ✅ PRUEBA 6: Actualizar TODO
  it("debe actualizar el estado de un TODO", () => {
    const updatedTodo: Todo = {
      id: 1,
      title: "Aprender Vitest",
      completed: true,
    };

    service.updateTodo(1, true).subscribe((todo) => {
      expect(todo.completed).toBe(true);
    });

    const req = httpMock.expectOne("/api/todos/1");
    expect(req.request.method).toBe("PATCH");
    expect(req.request.body).toEqual({ completed: true });
    req.flush(updatedTodo);
  });

  // ✅ PRUEBA 7: Lanzar error si ID es inválido
  it("debe lanzar error si el ID es inválido", () => {
    expect(() => service.updateTodo(-1, true)).toThrow("ID inválido");
    expect(() => service.updateTodo(0, true)).toThrow("ID inválido");
  });

  // ✅ PRUEBA 8: Eliminar TODO
  it("debe eliminar un TODO por su ID", () => {
    service.deleteTodo(1).subscribe(() => {
      expect(true).toBe(true); // Si no lanza error, está bien
    });

    const req = httpMock.expectOne("/api/todos/1");
    expect(req.request.method).toBe("DELETE");
    req.flush(null);
  });

  afterEach(() => {
    httpMock.verify(); // Verificar que no hay peticiones sin respuesta
  });
});
```

#### Paso 3: Ejecutar los tests

```bash
npm run test
# O con watch:
npm run test:watch
```

**Resultado esperado**: 8 pruebas pasando ✅

### C. Ejercicio 2: Componente principal con pruebas unitarias

Crea un archivo `src/components/todo-list.component.ts`:

```ts
import { Component, OnInit } from "@angular/core";
import { TodoService, Todo } from "../services/todo.service";
@Component({
  selector: "app-todo-list",
  template: `
    <h2>Lista de TODOs</h2>
    <ul>
      <li *ngFor="let todo of todos">
        {{ todo.title }} - {{ todo.completed ? "Hecho" : "Pendiente" }}
      </li>
    </ul>
  `,
})
export class TodoListComponent implements OnInit {
  todos: Todo[] = [];

  constructor(private todoService: TodoService) {}

  ngOnInit() {
    this.todoService.getTodos().subscribe((todos) => {
      this.todos = todos;
    });
  }
}
```

Crea un archivo `src/components/todo-list.component.spec.ts`:

```ts
import { TestBed, ComponentFixture } from "@angular/core/testing";
import { By } from "@angular/platform-browser";
import { TodoListComponent } from "./todo-list.component";
import { TodoService } from "../services/todo.service";
import { of } from "rxjs";
import { describe, it, beforeEach, expect } from "vitest";
describe("TodoListComponent", () => {
  let component: TodoListComponent;
  let fixture: ComponentFixture<TodoListComponent>;
  let mockTodoService: Partial<TodoService>;

  beforeEach(() => {
    // Mock del servicio con datos de prueba
    mockTodoService = {
      getTodos: () =>
        of([
          { id: 1, title: "Aprender Vitest", completed: false },
          { id: 2, title: "Hacer pruebas", completed: true },
        ]),
    };

    TestBed.configureTestingModule({
      declarations: [TodoListComponent],
      providers: [{ provide: TodoService, useValue: mockTodoService }],
    }).compileComponents();

    fixture = TestBed.createComponent(TodoListComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  // ✅ PRUEBA 1: El componente se crea correctamente
  it("debe crear el componente", () => {
    expect(component).toBeTruthy();
  });

  // ✅ PRUEBA 2: Renderizar la lista de TODOs
  it("debe renderizar la lista de TODOs obtenida del servicio", () => {
    const todoItems = fixture.debugElement.queryAll(By.css("li"));
    expect(todoItems.length).toBe(2);
    expect(todoItems[0].nativeElement.textContent).toContain("Aprender Vitest");
    expect(todoItems[0].nativeElement.textContent).toContain("Pendiente");
    expect(todoItems[1].nativeElement.textContent).toContain("Hacer pruebas");
    expect(todoItems[1].nativeElement.textContent).toContain("Hecho");
  });
});
```

### D. Ejercicio 3: Componente con signals `input()` y `output()` (10 min)

#### Paso 1: Crear el componente

Crea un archivo `src/components/todo-item.component.ts`:

```ts
import { Component, input, output, OnInit } from "@angular/core";

export interface TodoItem {
  id: number;
  title: string;
  completed: boolean;
}

@Component({
  selector: "app-todo-item",
  template: `
    <div class="todo-item" [class.completed]="item().completed">
      <input
        type="checkbox"
        [checked]="item().completed"
        (change)="onToggle()"
        data-testid="toggle-checkbox"
      />
      <span>{{ item().title }}</span>
      <button
        (click)="onDelete()"
        data-testid="delete-button"
        class="delete-btn"
      >
        ✕
      </button>
    </div>
  `,
  styles: [
    `
      .todo-item {
        display: flex;
        gap: 10px;
        padding: 8px;
        border: 1px solid #ddd;
      }
      .todo-item.completed span {
        text-decoration: line-through;
        color: #999;
      }
      .delete-btn {
        background: red;
        color: white;
        border: none;
        cursor: pointer;
      }
    `,
  ],
})
export class TodoItemComponent implements OnInit {
  // Input signal: recibe datos del componente padre
  item = input<TodoItem>({ id: 0, title: "", completed: false });

  // Output signals: emiten eventos al componente padre
  toggled = output<number>(); // Emite el ID cuando se marca/desmarca
  deleted = output<number>(); // Emite el ID cuando se elimina

  ngOnInit() {
    console.log("TodoItemComponent inicializado con:", this.item());
  }

  // Método llamado cuando se hace toggle del checkbox
  onToggle() {
    const id = this.item().id;
    this.toggled.emit(id);
  }

  // Método llamado cuando se hace click en eliminar
  onDelete() {
    const id = this.item().id;
    this.deleted.emit(id);
  }
}
```

#### Paso 2: Crear el archivo de pruebas del componente

Crea un archivo `src/components/todo-item.component.spec.ts`:

```ts
import { TestBed, ComponentFixture } from "@angular/core/testing";
import { By } from "@angular/platform-browser";
import { TodoItemComponent, TodoItem } from "./todo-item.component";
import { describe, it, beforeEach, expect, vi } from "vitest";

describe("TodoItemComponent", () => {
  let component: TodoItemComponent;
  let fixture: ComponentFixture<TodoItemComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [TodoItemComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(TodoItemComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  // ✅ PRUEBA 1: El componente se crea correctamente
  it("debe crear el componente", () => {
    expect(component).toBeTruthy();
  });

  // ✅ PRUEBA 2: Renderizar el título del TODO
  it("debe renderizar el título del TODO recibido", () => {
    const mockTodo: TodoItem = {
      id: 1,
      title: "Aprender Vitest",
      completed: false,
    };
    component.item.set(mockTodo);
    fixture.detectChanges();

    const titleSpan = fixture.debugElement.query(By.css("span"));
    expect(titleSpan.nativeElement.textContent).toContain("Aprender Vitest");
  });

  // ✅ PRUEBA 3: Aplicar clase 'completed' cuando TODO está completado
  it("debe agregar clase 'completed' cuando el TODO está completado", () => {
    const completedTodo: TodoItem = {
      id: 1,
      title: "Hecho",
      completed: true,
    };
    component.item.set(completedTodo);
    fixture.detectChanges();

    const todoDiv = fixture.debugElement.query(By.css(".todo-item"));
    expect(todoDiv.nativeElement.classList.contains("completed")).toBe(true);
  });

  // ✅ PRUEBA 4: No aplicar clase 'completed' cuando TODO está pendiente
  it("no debe agregar clase 'completed' cuando el TODO está pendiente", () => {
    const pendingTodo: TodoItem = {
      id: 1,
      title: "Pendiente",
      completed: false,
    };
    component.item.set(pendingTodo);
    fixture.detectChanges();

    const todoDiv = fixture.debugElement.query(By.css(".todo-item"));
    expect(todoDiv.nativeElement.classList.contains("completed")).toBe(false);
  });

  // ✅ PRUEBA 5: Emitir evento 'toggled' cuando se hace click en el checkbox
  it("debe emitir evento 'toggled' al hacer click en el checkbox", () => {
    const mockTodo: TodoItem = { id: 42, title: "Test", completed: false };
    component.item.set(mockTodo);
    fixture.detectChanges();

    const spy = vi.fn();
    component.toggled.subscribe(spy);

    const checkbox = fixture.debugElement.query(
      By.css("input[type='checkbox']"),
    );
    checkbox.nativeElement.click();

    expect(spy).toHaveBeenCalled();
    expect(spy).toHaveBeenCalledWith(42);
  });

  // ✅ PRUEBA 6: Emitir evento 'deleted' cuando se hace click en botón eliminar
  it("debe emitir evento 'deleted' al hacer click en el botón eliminar", () => {
    const mockTodo: TodoItem = { id: 99, title: "Eliminar", completed: false };
    component.item.set(mockTodo);
    fixture.detectChanges();

    const spy = vi.fn();
    component.deleted.subscribe(spy);

    const deleteButton = fixture.debugElement.query(By.css(".delete-btn"));
    deleteButton.nativeElement.click();

    expect(spy).toHaveBeenCalled();
    expect(spy).toHaveBeenCalledWith(99);
  });

  // ✅ PRUEBA 7: El checkbox debe estar marcado cuando el TODO está completado
  it("debe tener el checkbox marcado cuando el TODO está completado", () => {
    const completedTodo: TodoItem = {
      id: 1,
      title: "Hecho",
      completed: true,
    };
    component.item.set(completedTodo);
    fixture.detectChanges();

    const checkbox = fixture.debugElement.query(
      By.css("input[type='checkbox']"),
    ).nativeElement;
    expect(checkbox.checked).toBe(true);
  });

  // ✅ PRUEBA 8: El checkbox debe estar desmarcado cuando el TODO está pendiente
  it("debe tener el checkbox desmarcado cuando el TODO está pendiente", () => {
    const pendingTodo: TodoItem = {
      id: 1,
      title: "Pendiente",
      completed: false,
    };
    component.item.set(pendingTodo);
    fixture.detectChanges();

    const checkbox = fixture.debugElement.query(
      By.css("input[type='checkbox']"),
    ).nativeElement;
    expect(checkbox.checked).toBe(false);
  });
});
```

#### Paso 3: Ejecutar los tests del componente

```bash
npm run test src/components/todo-item.component.spec.ts
```

**Resultado esperado**: 8 pruebas pasando ✅

### E. Ejercicio 4: Analizar cobertura de código (5 min)

#### Paso 1: Generar el reporte de cobertura

```bash
npm run test:coverage
```

Esto generará un reporte en `coverage/` con archivos HTML.

#### Paso 2: Abrir el reporte HTML

```bash
# En Windows
start coverage/index.html

# En macOS
open coverage/index.html

# En Linux
xdg-open coverage/index.html
```

#### Paso 3: Analizar el reporte

**En el dashboard principal (coverage/index.html)**, busca:

1. **Línea "All files"**: Muestra la cobertura global
   - `% Stmts` (sentencias)
   - `% Branch` (ramas)
   - `% Funcs` (funciones)
   - `% Lines` (líneas)

2. **Haz clic en `todo.service.ts`** para ver:
   - Qué líneas están verdes (cubiertas): ✅
   - Qué líneas están rojas (no cubiertas): ❌
   - Qué líneas están amarillas (parcialmente cubiertas): 🟡

3. **Busca líneas sin cubrir** (rojas):
   - Métodos no probados
   - Ramas de error sin validar
   - Código muerto

#### Paso 4: Documentar hallazgos

Crea un archivo `COVERAGE_ANALYSIS.md`:

```markdown
# Análisis de Cobertura

## TodoService

**Cobertura total**: 92% (ejemplo)

### Líneas cubiertas ✅

- `getTodos()`: Completamente testado
- `getTodoById()`: Completamente testado
- `createTodo()`: Completamente testado
- Validaciones de entrada

### Líneas NO cubiertas ❌

- `deleteTodo()`: No se probó el caso de error
  **Razón**: Falta un test para error 404

### Líneas parcialmente cubiertas 🟡

- `updateTodo()`: Se probó exitoso, pero no el error de validación
  **Razón**: Agregar test para ID inválido

## TodoItemComponent

**Cobertura total**: 95% (ejemplo)

### Líneas cubiertas ✅

- Template rendering
- Event emissions
- Class binding

### Acciones recomendadas

1. Agregar test para casos de error en el servicio
2. Probar integración entre componente y servicio
3. Mantener cobertura > 80% en producción
```

#### Paso 5: Mejorar la cobertura

Si hay líneas sin cubrir, agrega tests específicos. Ejemplo:

```ts
// Nuevo test para deleteTodo
it("debe manejar error al eliminar TODO", () => {
  service.deleteTodo(999).subscribe(
    () => fail("Debería fallar"),
    (error) => {
      expect(error.status).toBe(404);
    },
  );

  const req = httpMock.expectOne("/api/todos/999");
  req.error(new ErrorEvent("Not found"), { status: 404 });
});
```

### F. Recursos adicionales

- 📖 [Documentación oficial de Vitest](https://vitest.dev)
- 📖 [Angular Testing Guide](https://angular.io/guide/testing)
- 📖 [Patterns de Testing (AAA, GWT)](https://martinfowler.com/bliki/GivenWhenThen.html)
- 🛠️ Herramientas: VS Code extension "Coverage Gutters" para visualizar cobertura en el editor
