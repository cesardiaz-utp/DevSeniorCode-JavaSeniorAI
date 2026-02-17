# Unidad 2 - Clase 5: Transformación de Datos y Reactividad Híbrida

- **Duración**: 2 horas
- **Objetivo**: Dominar la interoperabilidad entre los flujos asíncronos de RxJS y la reactividad granular de Signals. Implementaremos un patrón de búsqueda profesional ("Type-ahead") que optimiza los recursos del servidor y ofrece una experiencia de usuario fluida, utilizando lo mejor de ambos mundos.

## Parte 1: Teoría - La Danza entre Streams y Estado (45 Minutos)

Con la llegada de Angular 21, el ecosistema ha evolucionado hacia un modelo de reactividad más refinado. Muchos desarrolladores se preguntan: _"¿Debo dejar de usar RxJS?"_. La respuesta es un rotundo **NO**. La clave del éxito en aplicaciones modernas reside en una **Arquitectura Híbrida**.

### 1. RxJS vs Signals: Comprendiendo los Roles

Para entender cuándo usar cada uno, debemos analizar su naturaleza fundamental y el problema que resuelven.

#### El Problema del Rendimiento (`Zone.js`)

Históricamente, Angular usaba `Zone.js` para detectar cambios. Si hacías clic en un botón, Angular revisaba todo el árbol de componentes para ver qué cambió. Esto es ineficiente en aplicaciones grandes. **Signals** introducen una reactividad granular: cuando una señal cambia, Angular sabe exactamente qué nodo de texto en el DOM debe actualizarse, sin revisar el resto de la aplicación. Esto es el camino hacia aplicaciones "Zone-less" (sin `Zone.js`).

#### Tabla Comparativa

| Característica | RxJS (Observables) | Signals |
| --- | --- | --- |
| **Naturaleza** | **Streams (Flujos)**: Eventos que ocurren a lo largo del tiempo (ej. clics, respuestas HTTP, sockets). | **State (Estado)**: Un contenedor que guarda un valor actual y notifica a sus dependientes cuando cambia. |
| **Tiempo** | **Asíncrono**: Los valores llegan en el futuro (Push). | **Síncrono**: El valor siempre está disponible para lectura inmediata (Pull). |
| **Evaluación** | **Lazy (Perezosa)**: No pasa nada hasta que te suscribes (`.subscribe()`). | **Eager (Ansiosa)**: Se evalúan y calculan al momento de acceder o cambiar. |
| **Domino** | Gestión de eventos complejos: Cancelación, Retraso, Reintento, Combinación. | Renderizado en UI y Estado Derivado (`computed`). |
| **Regla de Oro** | Úsalo para **llegar** al dato (La Tubería). | Úsalo para **almacenar y mostrar** el dato (El Depósito). |

### 2. Interoperabilidad: El Puente Mágico

Angular nos ofrece primitivas robustas para cruzar entre el mundo asíncrono y el síncrono.

#### 2.1. `toSignal` (De Observable a Signal)

Convierte un flujo asíncrono en un valor de lectura síncrona.

- **Suscripción Automática**: Se suscribe al Observable inmediatamente y se desuscribe automáticamente cuando el componente se destruye. ¡Adiós a los Memory Leaks por olvidar el `unsubscribe()`!
- **Opciones Críticas**:
  - `initialValue`: Define qué valor tiene la Signal antes de que el Observable emita el primer dato (vital para UX, evita pantallas en blanco).
  - `rejectErrors`: Si es true, lanza la excepción. Si es false (default), permite manejar el error de otra forma.
  - `requireSync`: Úsalo solo si estás 100% seguro de que el Observable es síncrono (ej. `BehaviorSubject`), de lo contrario lanzará error en tiempo de ejecución.

#### 2.2. `toObservable` (De Signal a Observable)

Convierte cambios de estado en un stream de eventos.

- **Contexto de Inyección**: Debe llamarse dentro de un contexto de inyección (constructor o inicialización de campos), ya que internamente usa `Effect`.
- **Caso de Uso**: Disparar efectos secundarios. Ejemplo: "Cada vez que cambie la signal `filtroUsuario`, dispara una petición al servidor".

### 3. El Arte de la Manipulación de Flujos (Operadores)

El paradigma reactivo brilla cuando manipulamos el tiempo y el orden de los eventos.

#### Operadores de Tiempo y Filtrado

- `debounceTime(ms)`: Actúa como un portero. "No me molestes hasta que haya silencio por 300ms". Es esencial para inputs de búsqueda para no saturar al servidor con cada tecla (`K`, `Ke`, `Key`, `KeyU`, `KeyUp`).
- `distinctUntilChanged()`: Evita procesamiento redundante. Si el usuario escribe "Java", borra la última "a" y la vuelve a escribir rápido ("Java"), el valor final es idéntico. Este operador bloquea la emisión duplicada.

#### Estrategias de Aplanamiento (Flattening Operators)

Cuando un Observable emite otro Observable (ej. un cambio en un input dispara una petición HTTP), necesitamos "aplanarlos". Elegir el operador correcto es vital para la integridad de los datos.

| Operador | Comportamiento ("Mental Model") | Caso de Uso Ideal |
| --- | --- | --- |
| `switchMap` | **El Impaciente**: "Si llega una nueva petición, olvida la anterior inmediatamente". Cancela la suscripción previa. | **Búsquedas (Type-ahead)**. Solo nos importa el resultado de lo último que escribió el usuario. |
| `mergeMap` | **El Paralelo**: "Atiende a todos a la vez, no importa el orden de llegada". | **Guardar/Borrar**. Si el usuario borra 5 ítems rápido, queremos que se borren los 5, no solo el último. |
| `concatMap` | **El Ordenado**: "Ponte en la fila. No atiendo al siguiente hasta terminar con el actual". | **Operaciones secuenciales críticas** (ej. transacciones bancarias donde el orden importa). |
| `exhaustMap` | **El Ignorante**: "Estoy ocupado. Si llega alguien nuevo mientras trabajo, lo ignoro". | **Botón de Login**. Evita que el usuario haga doble clic y envíe dos peticiones de login. |

### 4. Paradigma: Imperativo vs Declarativo

En el desarrollo tradicional (Imperativo), escribimos funciones que dicen **cómo** hacer las cosas paso a paso:

```typescript
// Enfoque Imperativo (Evitar)
onSearch(texto: string) {
  this.doctorService.search(texto).subscribe(res => {
    this.doctors = res; // Mutación manual de estado
  });
}
```

En el desarrollo reactivo moderno (Declarativo), definimos **qué** son los datos en función de otros datos:

```typescript
// Enfoque Declarativo (Recomendado)
// "La lista de doctores ES el resultado de transformar los cambios del input..."
doctors = toSignal(
  this.inputChanges.pipe(
    switchMap(texto => this.doctorService.search(texto))
  )
);
```

Este cambio mental reduce bugs de estado inconsistente y hace el código más predecible y fácil de leer.

## Parte 2: Laboratorio Práctico (1h 15m)

### Paso 1: Optimización del Backend (Búsqueda Eficiente)

Antes de hacer magia en el Front, necesitamos un endpoint que responda rápido.

1. **Repositorio (`DoctorRepository`)**: Vamos a usar JPQL para buscar por nombre o apellido ignorando mayúsculas/minúsculas.

    **Prompt para Cursor (Composer)**:

    ```plain
    En `DoctorRepository`, añade un método `searchDoctors`. Debe recibir un String `query`. Usa una `@Query` JPQL personalizada.
    Debe seleccionar doctores donde el `firstName` O el `lastName` contengan el texto `query` `(LIKE %...%)`.
    Usa `LOWER()` para hacer la búsqueda case-insensitive.
    ```

    **Código esperado**:

    ```java
    @Query("SELECT d FROM Doctor d WHERE LOWER(d.firstName) LIKE LOWER(CONCAT('%', :query, '%')) OR LOWER(d.lastName) LIKE LOWER(CONCAT('%', :query, '%'))")
    List<Doctor> searchDoctors(String query);
    ```

2. **Controlador (`DoctorController`)**: Expón el endpoint `GET /api/v1/doctors/search?q={query}`.

### Paso 2: Servicio de Frontend (`DoctorService`)

Preparamos el cliente HTTP para consumir la búsqueda.

1. **Prompt Inline**:

    ```plain
    Añade el método `search(query: string)` al `DoctorService`. Retorna `Observable<DoctorResponse[]>`. Si el query está vacío, retorna un array vacío inmediatamente (`of([])`) para no llamar al backend innecesariamente."
    ```

### Paso 3: El Buscador Reactivo ("Type-ahead")

Aquí implementaremos el patrón moderno de Angular.

1. **Setup del Componente (`DoctorsComponent`)**: Necesitamos un `FormControl` para el input y una Signal para los resultados.

    _Nota_: Asegúrate de importar `ReactiveFormsModule` en el componente standalone.

2. **Implementación con Cursor (Composer)**:

    **Prompt**:

    ```plain
    Implementa un buscador reactivo en `DoctorsComponent`.

    1. Crea un `searchControl = new FormControl('')`.
    2. Crea una propiedad `doctors` que sea una Signal (`toSignal`).
    3. La fuente de la Signal debe ser `searchControl.valueChanges`.
    4. Aplica el pipe de RxJS:
        - `debounceTime(300)`
        - `distinctUntilChanged()`
        - `filter` para evitar queries nulos.
        - `switchMap` llamando a `doctorService.search(query)`.
    5. Maneja errores con `catchError` retornando un array vacío.
    6. Define `{ initialValue: [] }` en las opciones de `toSignal`.
    ```

    **Código de Referencia**:

    ```typescript
    // doctors.component.ts
    searchControl = new FormControl('');

    doctors = toSignal(
      this.searchControl.valueChanges.pipe(
        debounceTime(300),
        distinctUntilChanged(),
        switchMap(query => {
          if (!query || query.length < 2) return of([]);
          return this.doctorService.search(query);
        }),
        catchError(err => {
          console.error(err);
          return of([]);
        })
      ),
      { initialValue: [] }
    );
    ```

3. **Template (HTML)**: Vincula el input y muestra los resultados iterando sobre la Signal.

    ```html
    <!-- Input vinculado al FormControl -->
    <input [formControl]="searchControl" placeholder="Buscar médico..." 
          class="input-tailwind-styles..." />

    <!-- Iteración sobre la Signal (sin async pipe!) -->
    @for (doctor of doctors(); track doctor.id) {
      <div class="card">
        <h3>{{ doctor.firstName }} {{ doctor.lastName }}</h3>
        <p>{{ doctor.specialty }}</p>
      </div>
    } @empty {
      <p class="text-gray-500">No se encontraron resultados o inicie una búsqueda.</p>
    }
    ```

### Paso 4: Visualización de Estado de Carga (Loading)

El usuario necesita saber cuándo se está buscando. Vamos a combinar RxJS con Signals para esto.

1. Añade una signal `isSearching = signal(false)`.
2. Usa el operador `tap` en el pipe de RxJS:
    - Al inicio (antes del `switchMap`): `tap(() => this.isSearching.set(true))`
    - Al final (después del `switchMap`): Esto es truculento en pipes reusables. Una mejor opción es usar `finalize` dentro del observable interno del switchMap o simplemente asignar `false` en el tap de respuesta.

    _Mejor aproximación para este nivel_:

    ```typescript
    switchMap(query => 
      this.service.search(query).pipe(
        tap(() => this.isSearching.set(false)) // Apaga al recibir respuesta
      )
    ),
    tap(() => this.isSearching.set(true)) // Enciende al cambiar el input (ojo con el orden)
    ```

    _(Nota: Este es un excelente punto de discusión en clase sobre la complejidad de los efectos secundarios en RxJS)._

### Verificación

1. Abre la pestaña **Network**.
2. Escribe "Cardio" lentamente: `C... a... r... d... i... o`.
3. Deberías ver solo **una o dos peticiones** al backend, no 6.
4. Si borras rápido y escribes otra cosa, verifica que las peticiones anteriores se cancelen (en rojo en Chrome DevTools).

## Desafío (Homework)

### Búsqueda con Reintento Inteligente

A veces la red falla. Mejora tu pipe de búsqueda:

1. Usa el operador `retry(2)` antes del `catchError`.
    - Si la petición falla (ej. error 500 temporal), RxJS reintentará inmediatamente 2 veces más antes de rendirse.
2. **Avanzado**: Usa `retry({ count: 3, delay: 1000 })` para un reintento con espera (backoff).
