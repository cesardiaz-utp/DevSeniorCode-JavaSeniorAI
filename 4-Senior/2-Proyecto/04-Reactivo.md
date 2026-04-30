# Unidad 2 - Clase 4: Arquitectura Reactiva (Sistemas No Bloqueantes)

**Objetivo**: Comprender la filosofía de los sistemas reactivos y su implementación en la JVM mediante Project Reactor y Spring WebFlux. El estudiante dominará el modelo de ejecución no bloqueante, la gestión de hilos optimizada y la mecánica de la contrapresión (Backpressure) para construir sistemas de alta resiliencia y escalabilidad masiva. Se analizarán las implicaciones de hardware y el cambio de paradigma mental necesario para la programación declarativa.

## 1. El Paradigma Reactivo: La Crisis del Modelo "Thread-per-Request"

Para entender la arquitectura reactiva, debemos analizar la evolución de los servidores web y el famoso problema C10k (manejar 10,000 conexiones simultáneas). El modelo tradicional de servidores (como Tomcat en Spring MVC) utiliza un enfoque de concurrencia basado en hilos del sistema operativo, el cual encuentra un techo físico debido a la gestión de recursos del kernel.

### 1.1. El Costo de la Ineficiencia Imperativa

La computación imperativa tradicional se basa en el modelo de **"un hilo por petición"**. Cuando llega una solicitud, el servidor asigna un hilo que se encarga de todo el ciclo de vida: consultar la base de datos, llamar a servicios externos y devolver la respuesta.

Veamos la diferencia en la firma de un controlador:

```java
// ❌ MODELO IMPERATIVO (Spring MVC / Tomcat)
@GetMapping("/usuarios/{id}")
public Usuario obtenerUsuario(@PathVariable String id) {
    // El hilo actual se BLOQUEA totalmente aquí esperando a la base de datos
    return usuarioRepository.findById(id);
}

// ✅ MODELO REACTIVO (Spring WebFlux / Netty)
@GetMapping("/usuarios/{id}")
public Mono<Usuario> obtenerUsuarioReactivo(@PathVariable String id) {
    // Retorna inmediatamente una "Promesa" (Mono). El hilo queda libre para otra petición.
    // La BD avisará al Event Loop cuando los datos estén listos.
    return usuarioRepositoryReactivo.findById(id);
}
```

- **El Problema del Bloqueo (I/O Wait) y Syscalls**: En un sistema bloqueante, cuando realizas una llamada de red, el hilo ejecuta una "System Call" (syscall) de lectura. El kernel detiene la ejecución de ese hilo hasta que los datos llegan al buffer de red. Si una base de datos tarda 100ms, ese hilo es una unidad de computación muerta: no procesa lógica, pero sigue consumiendo memoria de stack ($\approx 1$ MB).
- **Context Switching (Cambio de Contexto)**: Con 10,000 peticiones, el planificador del sistema operativo (Scheduler) debe decidir qué hilos ejecutar en los núcleos de la CPU. El costo de salvar el estado de un hilo y cargar el de otro se vuelve prohibitivo. La CPU gasta más tiempo "gestionando hilos" que ejecutando código real, lo que se conoce como _thrashing_.
- **Desperdicio Crítico de RAM y Energía**: 10,000 hilos bloqueados implican 10 GB de RAM reservados solo para las pilas de ejecución. En entornos de nube (AWS/Azure), esto obliga a escalar a instancias con mucha memoria, aumentando los costos operativos de manera innecesaria cuando la CPU está mayormente ociosa.

### 1.2. Los Cuatro Pilares del Manifiesto Reactivo

La arquitectura reactiva no es solo "programación asíncrona"; es un diseño sistémico orientado a la robustez. Los pilares no son independientes, sino que trabajan en conjunto para garantizar la salud del sistema.

#### A. Responsivo (Responsive): La Promesa del Tiempo

Un sistema es responsivo cuando responde de manera oportuna y predecible. No se trata solo de ser "veloz" en el mejor de los casos, sino de mantener una latencia controlada bajo presión.

- **Límites de Latencia (Boundaries)**: Un sistema reactivo establece SLOs (Service Level Objectives) claros. Si una operación no puede completarse, el sistema debe responder con un error o un _fallback_ de manera rápida, en lugar de dejar al usuario esperando indefinidamente.
- **Confianza del Usuario**: La responsividad permite construir interfaces que se sienten fluidas y sistemas que pueden cumplir con acuerdos de nivel de servicio (SLAs) críticos para el negocio.

#### B. Resiliente (Resilient): El Arte de Fallar con Elegancia

La resiliencia en sistemas reactivos se basa en la idea de que los fallos son inevitables. El objetivo no es evitar el fallo, sino contenerlo.

- **Aislamiento (Bulkheading)**: Al igual que los mamparos de un barco, si un componente falla, el fallo se queda ahí. No se propaga a otros servicios ni agota los recursos globales.
- **Delegación y Supervisión**: El manejo de errores se delega a un componente externo (el suscriptor o un supervisor), lo que permite que el componente fallido se recupere o se reinicie sin afectar el flujo principal.
- **Estrategias de Recuperación**: Se implementan mecanismos automáticos como reintentos exponenciales, timeouts y cortocircuitos (_Circuit Breakers_) integrados en el flujo de datos.

#### C. Elástico (Elastic): El Acordeón de Recursos

La elasticidad es la capacidad del sistema para adaptarse a los cambios en la carga de trabajo de forma dinámica, ya sea escalando hacia arriba (up) o hacia afuera (out).

- **Eficiencia de Hilos**: Dado que no bloqueamos hilos, un solo nodo puede manejar un volumen de tráfico muy variable. La elasticidad reactiva es mucho más granular que la elástica de VMs tradicionales.
- **Sin Cuellos de Botella Centrales**: Para ser elástico, el sistema debe evitar estados compartidos o bloqueos de sincronización (Locks) que se vuelvan cuellos de botella al escalar a múltiples núcleos o nodos.
- **Auto-gestión**: Los sistemas reactivos detectan aumentos en la tasa de eventos y pueden disparar el aprovisionamiento de más recursos de manera proporcional.

#### D. Orientado a Mensajes (Message Driven): El Medio para el Fin

Este es el pilar "habilitador". La comunicación mediante mensajes asíncronos es lo que permite lograr los otros tres pilares.

- **Desacoplamiento Total**: El emisor no conoce al receptor. Esto permite mover componentes entre hilos o servidores (transparencia de ubicación) sin cambiar el código.
- **Flujos No Bloqueantes**: La comunicación por mensajes elimina la necesidad de esperar activamente por una respuesta.
- **Manejo de Backpressure**: Al ser orientado a mensajes, el receptor puede enviar señales de control hacia atrás para decir "estoy saturado, envíame datos más despacio", algo que es imposible en una llamada REST directa bloqueante.

## 2. El Mecanismo Interno: Event Loop vs. Servlet Stack

La arquitectura reactiva (implementada por servidores como Netty) cambia radicalmente la forma en que el software interactúa con el hardware.

### 2.1. El Bucle de Eventos (Event Loop) y Multiplexación

A diferencia de los pools de cientos de hilos de los servlets, un sistema reactivo utiliza una cantidad mínima de hilos, usualmente fija y ligada al número de núcleos físicos de la CPU. Estos hilos se denominan Event Loops.

- **Multiplexación de E/S (epoll/kqueue)**: El Event Loop utiliza primitivas del kernel (como `epoll` en Linux) para monitorizar miles de descriptores de archivos (conexiones) simultáneamente. El hilo le dice al kernel: "Avísame cuando cualquiera de estas 10,000 conexiones tenga datos".
- **Flujo No Bloqueante Real**: Al delegar la espera al kernel, el Event Loop nunca se detiene. Procesa un evento, registra un callback y salta al siguiente. Esto permite que un solo hilo maneje miles de peticiones que están en estado "esperando datos", eliminando el desperdicio de memoria y el costo de cambio de contexto.
- **Aprovechamiento de Caché L1/L2**: Al mantener pocos hilos activos, los datos calientes permanecen en las caches de la CPU, evitando los constantes fallos de caché que ocurren en el modelo multi-hilo masivo.

### 2.2. La Analogía de la Cocina Expandida

- **Modelo Tradicional**: Un camarero toma una orden y se queda parado frente a la cocina. No puede atender otra mesa hasta que el chef termine. Si el chef tarda, el camarero es inútil. Para atender 50 mesas, necesitas 50 camareros, lo que colapsa el pasillo de la cocina.
- **Modelo Reactivo**: El camarero toma la orden, deja el ticket y atiende a la siguiente mesa. El "Evento" de que la comida está lista dispara un aviso. El camarero (que nunca dejó de trabajar) simplemente entrega el plato. Con 2 camareros ágiles puedes atender las mismas 50 mesas sin caos en los pasillos.

## 3. Project Reactor: El Motor de la Arquitectura

**Project Reactor** es la implementación de la especificación **Reactive Streams** para la JVM. No es simplemente una librería de utilidades, sino un framework de ingeniería de flujos asíncronos que define cómo viajan los datos y las señales a través del sistema.

### 3.1. Anatomía de la Suscripción: El Modelo Push-Pull

A diferencia del modelo iterable clásico (donde tú pides datos), en Reactor el modelo es un híbrido de **Push (Empuje)** y **Pull (Tracción)**:

1. **Publisher (Publicador)**: La fuente de datos.
2. **Subscriber (Suscriptor)**: El consumidor final.
3. **Subscription (Suscripción)**: El contrato que une a ambos y gestiona la demanda (Backpressure).
4. **Signal (Señales)**: Los flujos emiten tres tipos de eventos: `onNext` (un dato llegó), `onError` (algo falló y el flujo se cierra) y `onComplete` (éxito total y cierre).

### 3.2. Mono vs. Flux: Profundización Técnica

Reactor nos obliga a ser semánticamente explícitos sobre la cardinalidad de nuestros datos:

- **Mono (0 a 1 elemento)**: Representa una promesa de un valor futuro. Es la abstracción perfecta para llamadas HTTP únicas, lecturas de base de datos por ID o cualquier operación "request-response" que no bloquea. Un `Mono` termina inmediatamente después de emitir su único valor o una señal de error.
- **Flux (0 a N elementos)**: Representa una tubería de datos abierta. Puede emitir 10 productos de una base de datos, 1 millón de registros de un sensor IoT, o incluso ser un flujo infinito que nunca termina. Flux permite componer lógica sobre "ventanas de tiempo" (ej. agrupar datos de los últimos 5 segundos).

```java
// Creación básica de flujos
Mono<String> monoVacio = Mono.empty();
Mono<String> monoUsuario = Mono.just("Juan Perez");

Flux<Integer> fluxNumeros = Flux.just(1, 2, 3, 4, 5);
Flux<String> fluxInfinito = Flux.interval(Duration.ofSeconds(1))
                                .map(tick -> "Tick: " + tick); // Emite cada segundo
```

### 3.3. Ciclo de Vida: Ensamblado vs. Suscripción

Este es el concepto más difícil de digerir para desarrolladores imperativos:

- **Tiempo de Ensamblado (Assembly Time)**: Cuando escribes `flux.map(...).filter(...)`, solo estás construyendo un plano arquitectónico. No hay datos fluyendo, no hay hilos trabajando. Es una declaración de intenciones.
- **Tiempo de Suscripción (Subscription Time)**: El flujo solo cobra vida cuando alguien invoca `.subscribe()`. En WebFlux, este "alguien" suele ser el servidor Netty al final de la cadena.
- **Inmutabilidad**: Cada operador que añades a un flujo devuelve un nuevo flujo. El flujo original nunca se modifica, lo que garantiza que no haya efectos secundarios inesperados.

Este es el error número uno de los principiantes: "**Nada sucede hasta que te suscribes**".

```java
// 1. TIEMPO DE ENSAMBLADO (Assembly Time) - Sólo construimos el plano
Flux<String> nombres = Flux.just("Ana", "Pedro", "Maria")
    .map(nombre -> {
        System.out.println("Transformando a: " + nombre.toUpperCase());
        return nombre.toUpperCase();
    });

System.out.println("Plano construido. ¿Se imprimió algo? No.");

// 2. TIEMPO DE SUSCRIPCIÓN (Subscription Time) - Abrimos la llave de paso
nombres.subscribe(nombreFinal -> {
    System.out.println("Recibido: " + nombreFinal);
});
// Resultado en consola:
// Transformando a: ANA
// Recibido: ANA
// Transformando a: PEDRO ...
```

### 3.4. Schedulers: El Control Maestro de los Hilos

En Reactor, tú no creas hilos (`new Thread()`), sino que delegas la ejecución a un **Scheduler** (Planificador). Esto es vital para evitar bloquear el Event Loop.

- `Schedulers.parallel()`: Optimizado para tareas intensivas de CPU (cálculos, criptografía).
- `Schedulers.boundedElastic()`: El "salvavidas" para tareas bloqueantes (I/O legado, JDBC). Crea un pool de hilos que puede crecer pero tiene límites para no agotar la memoria.
- `publishOn` vs. `subscribeOn`:
  - `subscribeOn` cambia el hilo desde el inicio de la cadena (donde nacen los datos).
  - `publishOn` cambia el hilo para todos los operadores que vienen después de él en la cadena.

Dado que trabajamos con pocos hilos (Event Loops), es vital saber cuándo delegar tareas pesadas (I/O legado o CPU intensivo) para no bloquear el sistema entero.

```java
public Flux<FacturaDTO> procesarFacturasPesadas() {
    return facturaRepository.findAll() // Nace en el Event Loop

        // CUIDADO: La generación del PDF es intensiva en CPU.
        // publishOn mueve todo lo que está DEBAJO a un pool paralelo.
        .publishOn(Schedulers.parallel())
        .map(factura -> pdfGenerator.crearPdf(factura))

        // Volvemos a cambiar de hilo para llamar a un API externa vieja (bloqueante)
        // boundedElastic() es el salvavidas para I/O bloqueante.
        .publishOn(Schedulers.boundedElastic())
        .map(pdf -> legacyEmailService.sendSync(pdf));
}
```

## 4. Backpressure: Gestión Inteligente del Caudal

Uno de los riesgos inherentes de la asincronía es la asimetría de velocidad entre servicios. Si el productor es más rápido que el consumidor, el sistema debe reaccionar.

### 4.1. El Modelo de Pull Dinámico (Demanda)

La especificación **Reactive Streams** resuelve esto con el modelo de **Demanda**: el consumidor le dice al productor exactamente cuántos elementos es capaz de procesar en ese momento mediante la señal `request(n)`. El productor tiene prohibido enviar más datos de los solicitados.

### 4.2. Estrategias de Desbordamiento y Consecuencias

Cuando el productor no puede frenar (ej. datos de la bolsa), debemos elegir una estrategia de desbordamiento mediante operadores específicos de Reactor:

- **Buffer**: `.onBackpressureBuffer()` acumula mensajes en una cola interna. Proporciona resiliencia ante ráfagas cortas pero conlleva el riesgo de `OutOfMemory` si la saturación es persistente.
- **Drop**: `.onBackpressureDrop()` descarta silenciosamente los datos que el consumidor no puede procesar. Es la estrategia ideal para telemetría IoT donde la pérdida de un dato puntual es preferible al colapso del sistema.
- **Latest**: `.onBackpressureLatest()` mantiene solo el valor más reciente emitido por el productor, descartando los intermedios. Es perfecto para indicadores de estado (como el precio actual de Bitcoin).
- **Error**: `.onBackpressureError()` cierra el flujo con una excepción si se excede la demanda. Es un enfoque de "Fallo Rápido" (Fail-Fast) para proteger la integridad del nodo.

```java
public void monitorearSensores() {
    Flux<SensorData> flujoAltaVelocidad = sensorService.streamDatosAltaFrecuencia();

    flujoAltaVelocidad
        // Estrategia 1: DROP. Ignora los datos que no alcanzo a procesar.
        .onBackpressureDrop(datoPerdido ->
            log.warn("El sistema está saturado. Descartando dato: {}", datoPerdido.getId())
        )

        // Estrategia 2: LATEST. Quédate sólo con el último valor mientras estoy ocupado.
        // .onBackpressureLatest()

        // Estrategia 3: ERROR. Falla rápido si sobrepaso la capacidad.
        // .onBackpressureError()

        // Simulamos un procesamiento lento del consumidor
        .delayElements(Duration.ofMillis(500))
        .subscribe(dato -> log.info("Procesado: {}", dato));
}
```

## 5. Resiliencia: El Arte de Fallar con Elegancia

La resiliencia en un sistema reactivo no consiste en evitar el error (algo imposible en redes distribuidas), sino en **aislarlo** y evitar que detenga la operación global del sistema. A diferencia del modelo imperativo donde un error suele terminar en una excepción que "vuela" por el stack de llamadas hasta que alguien la atrapa, en WebFlux los errores son **señales negativas** (`onError`) que viajan por el flujo, permitiéndonos reaccionar de forma declarativa.

### 5.1. El Ciclo de Vida del Fallo

Cuando un operador en la cadena lanza un error, se emite una señal `onError` hacia abajo (downstream). Si no se gestiona, esta señal cerrará la suscripción. Los operadores de resiliencia interceptan esta señal para intentar corregirla, reintentarla o transformarla.

### 5.2. Herramientas de Autocurado y Tolerancia a Fallos

A continuación, profundizamos en los mecanismos que WebFlux pone a nuestra disposición para construir sistemas "a prueba de balas":

- **Timeouts: El escudo contra la lentitud**. En redes inestables, un servicio que no responde es peor que un servicio que falla rápido. El operador `.timeout()` asegura que no mantengamos recursos (memoria, suscripciones) abiertos indefinidamente esperando por una respuesta que quizá nunca llegue, evitando el agotamiento de hilos por espera latente.
- **Retries: Gestión de la transitoriedad**. Muchos fallos en microservicios son efímeros (un micro-corte de red o una instancia reiniciándose). El operador `.retry()` permite automatizar los reintentos. La mejor práctica es usar **Exponential Backoff** (esperas cada vez más largas) para no saturar al servicio que ya está sufriendo.
- **Graceful Degradation (Degradación Controlada)**. Mediante `.onErrorResume()`, podemos implementar estrategias de "Plan B". Si el servicio principal falla, el usuario puede recibir una respuesta de una caché local, un valor estático seguro o una llamada a un servicio secundario de respaldo.

### 5.3. Implementación de Resiliencia Multicapa

```java
public Mono<PerfilUsuario> obtenerPerfilConResiliencia(String id) {
    return servicioExternoApi.llamar(id)

        // 1. Timeout: Si no responde en 2 segundos, lanza excepción.
        // Evita que el cliente se quede esperando eternamente.
        .timeout(Duration.ofSeconds(2))

        // 2. Retry con Backoff: Si hay un error de red, reintenta 3 veces.
        // Empezamos con 1s de espera y aumentamos para dar aire al proveedor.
        .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
            .filter(throwable -> isNetworkError(throwable))) // Solo reintentar si es error de red

        // 3. Fallback: Si después de los reintentos sigue fallando, degradamos el servicio.
        .onErrorResume(ex -> {
            log.error("API externa caída tras reintentos. Error: {}", ex.getMessage());
            // Retornamos un perfil básico de la caché para no romper la UI
            return cacheLocal.obtener(id);
        });
}
```

### 5.4. El Patrón Circuit Breaker (Cortocircuito)

Para una resiliencia de grado industrial, WebFlux se integra con librerías como **Resilience4j**. Un **Circuit Breaker** monitoriza la tasa de fallos: si supera un umbral (ej. 50% de fallos en la última ventana de tiempo), el circuito se "abre" y todas las llamadas fallan instantáneamente sin intentar contactar al servicio externo. Esto protege tanto a tu infraestructura como al servicio externo, permitiéndole recuperarse sin recibir ráfagas de peticiones destinadas al fracaso.

## 6. WebFlux vs. Spring MVC: Matriz de Decisión y Costos

La arquitectura reactiva no es un reemplazo gratuito para MVC; es una inversión técnica con pros y contras claros.

| Criterio                 | Spring MVC (Imperativo)                               | Spring WebFlux (Reactivo)                                              |
| ------------------------ | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| **Curva de Aprendizaje** | **Suave**. Pensamiento lineal y secuencial.           | **Empinada**. Requiere dominio de programación funcional.              |
| **Hardware Requerido**   | Mayor RAM para gestionar hilos.                       | Mínima RAM y CPU; ideal para contenedores pequeños.                    |
| **Depuración (Debug)**   | **Sencilla**. El Stack Trace muestra el error exacto. | **Compleja**. El error puede ocurrir en un hilo distinto al de origen. |
| **Ecosistema**           | Compatible con todo (JPA, JDBC, Hibernate).           | Requiere drivers específicos (R2DBC, MongoDB Reactive).                |
| **Latencia de Red**      | Bloquea el hilo durante llamadas externas.            | Libera el hilo; ideal para orquestación de microservicios.             |

### Implicaciones de Costo en la Nube

Adoptar WebFlux en entornos como Kubernetes permite definir _resource limits_ mucho más bajos. Mientras que un microservicio MVC podría necesitar 1GB de RAM para manejar carga concurrente, el mismo servicio en WebFlux podría funcionar con 256MB, reduciendo la factura de infraestructura en un 60-70% en despliegues masivos.

---

La arquitectura reactiva es la respuesta a la necesidad de sistemas ultra-eficientes y elásticos. Al aceptar que el tiempo y los fallos son ciudadanos de primera clase en nuestro código, construimos aplicaciones que no solo escalan más, sino que son más baratas de operar y más resistentes a la incertidumbre de las redes distribuidas.
