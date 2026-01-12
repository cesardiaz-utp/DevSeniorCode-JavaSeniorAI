# Preguntas de entrevista del Módulo 2: Java Backend Developer

Este documento recopila preguntas técnicas detalladas, diseñadas para validar una comprensión profunda de la ingeniería de backend moderna con Spring Boot 4, JPA y Seguridad. Las respuestas aquí proporcionadas buscan ir más allá de la definición de libro de texto, explorando el "por qué" y el "cómo" en escenarios del mundo real.

## Unidad 1: Spring Boot desde Cero – Arquitectura Profesional

### Q1: ¿Cómo facilita Spring Boot la configuración de un proyecto frente a Spring Framework tradicional ("Legacy Spring") y qué mecanismos internos lo hacen posible?

- **Auto-Configuración (La "Magia" explicada)**: Spring Boot utiliza un proceso sofisticado de escaneo del classpath al inicio de la aplicación. No se trata simplemente de detectar librerías, sino de aplicar lógica condicional compleja mediante anotaciones como `@ConditionalOnClass`, `@ConditionalOnMissingBean` y `@ConditionalOnProperty`. Por ejemplo, si Spring Boot detecta `h2.jar` en el classpath y no ve ninguna configuración manual de `DataSource` (bean faltante), asume automáticamente que deseas una base de datos en memoria y configura el `DataSource`, el `EntityManager` y el `TransactionManager` por ti. Esto elimina la necesidad de escribir cientos de líneas de XML o clases de configuración `JavaConfig` que eran obligatorias en versiones antiguas.
- **Opinionated Defaults (Convención sobre Configuración)**: Spring Boot toma decisiones de arquitectura por ti basadas en las mejores prácticas de la industria. Por ejemplo, configura Tomcat incrustado en el puerto 8080, establece UTF-8 como encoding predeterminado, configura Jackson para serializar fechas en formato ISO-8601 y establece niveles de log sensatos con Logback. Esto permite a los equipos centrarse en la lógica de negocio desde el minuto uno, en lugar de perder días debatiendo configuraciones de infraestructura. Sin embargo, es "opinado" pero no "dictatorial"; cualquier configuración por defecto puede ser sobrescrita fácilmente mediante propiedades.
- **Starters y Gestión de Dependencias**: Los "Starters" son descriptores de dependencias que agrupan librerías comunes para una funcionalidad específica. En lugar de buscar manualmente versiones compatibles de Hibernate, HikariCP, Spring ORM y drivers JDBC, simplemente agregas `spring-boot-starter-data-jpa`. Este starter utiliza el mecanismo de dependencias transitivas de Maven o Gradle para traer todo el árbol de dependencias necesario, garantizando que las versiones sean compatibles entre sí gracias al BOM (Bill of Materials) de Spring Boot, evitando el famoso "infierno de las versiones" (JAR hell).

### Q2: En Spring Boot 4, ¿cuál es el impacto arquitectónico y de calidad de usar Inyección por Constructor frente a `@Autowired` en campos?

- **Inmutabilidad y Thread-Safety**: La inyección por constructor es la única forma que permite declarar las dependencias de un bean como `final`. Esto garantiza que una vez que el bean es instanciado por el contenedor de Spring, su estado interno (sus dependencias) no puede ser modificado, haciendo que el componente sea inmutable y seguro para su uso en entornos concurrentes (Thread-Safe). La inyección por campo (`@Autowired private Service s;`) obliga a que el campo sea mutable y potencialmente nulo hasta que el framework intervenga mediante reflexión, lo cual es un riesgo de diseño.
- **Facilidad de Testing Unitario**: Con la inyección por constructor, tus clases son POJOs (Plain Old Java Objects) puros que no dependen de la "magia" de Spring para funcionar. Puedes instanciar la clase en un test unitario simplemente llamando al constructor y pasando dobles de prueba (mocks o stubs) manualmente: `new MyController(new MockService())`. Con la inyección de campo, estás obligado a levantar el contexto de Spring (lento) o usar librerías de reflexión como `MockitoAnnotations.openMocks(this)` para inyectar dependencias en campos privados, lo que acopla tus pruebas al framework innecesariamente.
- **Claridad y Prevención de Dependencias Circulares**: Al declarar dependencias en el constructor, haces explícito qué necesita la clase para funcionar. Si un constructor termina con 10 parámetros, es una señal visual clara de que la clase viola el Principio de Responsabilidad Única (SRP) y necesita refactorización ("Code Smell"). Además, la inyección por constructor permite que Spring detecte dependencias circulares (A depende de B, y B depende de A) en el momento del arranque, lanzando una excepción clara (`BeanCurrentlyInCreationException`), mientras que la inyección por campo puede ocultar este problema hasta que se invoca el método en tiempo de ejecución, causando un `StackOverflowError`.

### Q3: ¿Por qué es considerado una mala práctica crítica exponer Entidades JPA directamente en el Controlador y qué rol estratégico juegan los Records aquí?

- **Acoplamiento Fuerte y Fragilidad**: Si tu API devuelve directamente una entidad JPA (ej. `UserEntity`), estás acoplando el contrato de tu API pública con el esquema de tu base de datos. Si mañana decides renombrar la columna `first_name` a `given_name` en la base de datos por optimización, romperás automáticamente a todos los clientes (Frontend, Apps móviles) que consumen tu JSON, ya que el nombre del campo cambiará. Los DTOs actúan como un amortiguador o contrato estable, permitiendo que el modelo de datos evolucione independientemente de la vista pública.
- **Seguridad y Fugas de Datos**: Las entidades suelen contener datos que nunca deben salir del backend, como hashes de contraseñas, sales de encriptación, tokens de auditoría internos o banderas de borrado lógico. Exponer la entidad completa corre el riesgo de filtrar estos datos accidentalmente. Además, las entidades JPA suelen tener relaciones bidireccionales (`User` tiene `Posts`, `Posts` tiene `User`). Serializar esto directamente con Jackson suele provocar una recursión infinita (`StackOverflowError`) o, peor aún, el problema de "Lazy Loading Exception" si se intenta serializar fuera de la transacción (anti-patrón Open Session In View).
- **Solución con Java Records**: Los DTOs (Data Transfer Objects) tradicionales requerían mucho código repetitivo. Con Java **Records** (introducidos en Java 14+), podemos definir DTOs que son inmutables, concisos y seguros por defecto. Un `record UserDto(String name, String email) {}` define automáticamente constructor, accessors y métodos `equals/hashcode`. Al usarlos, garantizamos que los datos transferidos son una copia de solo lectura, eliminando efectos secundarios y haciendo el flujo de datos unidireccional y predecible.

### Q4: Explica la diferencia semántica y técnica entre PUT y PATCH al diseñar una API RESTful

- `PUT` **(Reemplazo Completo)**: Según el estándar HTTP, PUT es idempotente. El cliente debe enviar una representación _completa_ del recurso para actualizarlo. La semántica es "toma este objeto y ponlo en esta URI". Si el recurso ya existe, se reemplaza totalmente.
  - _Implicación Idempotente_: Si tienes un usuario con `{nombre: "Juan", edad: 25}` y envías un PUT con solo `{nombre: "Pedro"}`, el campo `edad` se perderá (se volverá nulo) porque no fue incluido en el payload de reemplazo.
- `PATCH` **(Modificación Parcial)**: No es estrictamente idempotente (aunque se suele implementar así). El cliente envía un conjunto de instrucciones o _solo los campos que cambiaron_. La semántica es "aplica estos cambios al recurso existente".
  - _Implicación_: Es mucho más eficiente en ancho de banda para objetos grandes. Si envías `{nombre: "Pedro"}`, el servidor debe buscar el recurso, actualizar solo el nombre y mantener la edad original. Implementar PATCH correctamente en Java es técnicamente más complejo porque necesitas distinguir entre "el cliente envió este campo como nulo" (quiere borrar el valor) y "el cliente no envió este campo" (no quiere tocar el valor). A menudo se requiere usar `Map<String, Object>` o tipos `Optional` wrappers para manejar esta distinción.

### Q5: ¿Qué problema de gestión de configuración resuelve el uso de perfiles (`@Profile`) y archivos `application-{profile}.yml`?

- **Paridad de Entornos**: En el ciclo de vida del software, una aplicación pasa por múltiples entornos (Desarrollo local, Testing/QA, Staging, Producción). Cada entorno requiere configuraciones drásticamente diferentes que no deben hardcodearse en el código Java.
- **Mecanismo de Spring Profiles**: Spring Boot permite definir archivos de propiedades específicos por perfil que se cargan sobre el `application.yml` base.
- **Ejemplo Práctico**:
  - En `dev`, quieres logs en nivel `DEBUG` para ver las queries SQL, una base de datos H2 en memoria que se reinicia rápido, y credenciales falsas para servicios externos.
  - En `prod`, necesitas logs en nivel `INFO` o `WARN` (para no saturar el disco), una conexión a PostgreSQL gestionada por un pool de conexiones robusto, y credenciales reales inyectadas de forma segura (ej. vía variables de entorno del sistema operativo o Secret Managers de la nube como AWS Secrets Manager).
- Al arrancar la aplicación con `-Dspring.profiles.active=prod`, Spring fusiona las configuraciones, priorizando las del perfil activo, permitiendo generar un único artefacto (JAR) que se adapta a cualquier entorno.

### Q6: ¿Cómo aplicas el Principio de Responsabilidad Única (SRP) en una arquitectura Spring Boot y cuáles son los síntomas de su violación?

- **Síntoma "God Classes"**: El error más común es tener "Controladores Dios" o "Servicios Dios" que hacen de todo. Un controlador que valida datos, llama a la base de datos, envía emails y formatea fechas viola el SRP.
- **Aplicación en Capas**:
  - **Controller**: Su única responsabilidad es la orquestación HTTP. Debe recibir el JSON, invocar la validación básica, delegar la acción a un servicio y traducir el resultado a un código de estado HTTP adecuado (200, 404, etc.). No debe tener lógica de negocio ("if user is admin...").
  - **Service**: Debe contener la lógica de negocio pura y casos de uso. Si un servicio de "Pedidos" empieza a tener código para construir HTML de correos electrónicos, está violando SRP. Debería delegar esa tarea a un `NotificationService` dedicado o, mejor aún, publicar un evento de aplicación para desacoplarse completamente.
  - **Repository**: Solo debe saber cómo hablar con la fuente de datos. No debe validar reglas de negocio.
- **Beneficio**: Al respetar SRP, las clases son más pequeñas, fáciles de leer y, crucialmente, más fáciles de testear, ya que tienen menos dependencias que mockear.

## Unidad 2: Bases de Datos Relacionales y NoSQL

### Q7: ¿Qué son las sentencias DDL y DML y por qué es peligroso usar `ddl-auto=update` en un entorno de producción?

- **DDL (Data Definition Language)**: Son comandos que definen la estructura o esquema de la base de datos. Ejemplos: `CREATE TABLE`, `ALTER TABLE ADD COLUMN`, `DROP INDEX`, `TRUNCATE`. Estos comandos cambian el contenedor de los datos.
- **DML (Data Manipulation Language)**: Son comandos que manipulan los datos dentro de las estructuras existentes. Ejemplos: `INSERT`, `UPDATE`, `DELETE`, `SELECT`. Estos comandos cambian el contenido.
- **El Peligro de** `ddl-auto=update`: Esta configuración instruye a Hibernate para que inspeccione tus entidades Java al inicio y trate de "actualizar" el esquema de la BD para que coincida. En desarrollo es útil, pero en producción es catastrófico por varias razones:
  1. **Pérdida de Datos**: Si cambias el tipo de un atributo o lo renombras, Hibernate podría decidir borrar la columna antigua y crear una nueva, eliminando todos los datos existentes.
  2. **Bloqueo de Tablas**: En bases de datos con millones de registros, un `ALTER TABLE` puede bloquear la tabla completa durante minutos u horas, causando una denegación de servicio.
  3. **Falta de Control**: Los cambios en producción deben ser predecibles, probados y reversibles. Hibernate actúa como una "caja negra". La solución profesional es usar herramientas de migración como Flyway o Liquibase.

### Q8: ¿Qué ventaja ofrece usar Lombok (@Data, @Builder) en las Entidades JPA y qué trampas mortales debes evitar?

- **Ventaja**: Reduce drásticamente el "boilerplate code". Una entidad con 10 campos requeriría ~80 líneas de código visualmente ruidoso para getters, setters, `equals`, `hashCode`, `toString` y constructores. Lombok permite mantener la clase limpia y enfocada en el modelo.
- **Trampas Mortales con JPA**:
  - `@Data` **y el HashCode**: La anotación `@Data` genera un método `hashCode()` que incluye todos los campos. Si tu entidad tiene relaciones Lazy (`@OneToMany`), invocar el `hashCode()` (que ocurre automáticamente si pones la entidad en un `HashSet` o `HashMap`) forzará la carga de todas las relaciones, disparando consultas SQL innecesarias o causando un `LazyInitializationException` si no hay sesión activa.
  - `@ToString` **y Recursión**: Si tienes una relación bidireccional (Usuario <-> Pedidos) y ambos tienen `@ToString` (incluido en `@Data`), al imprimir un Usuario se imprimirán sus Pedidos, y cada Pedido imprimirá su Usuario, causando un bucle infinito y un `StackOverflowError`.
  - **Recomendación**: En Entidades JPA, evita `@Data`. Usa `@Getter`, `@Setter` por separado. Sobrescribe `equals` y `hashCode` manualmente para usar solo la Clave Primaria (ID), y usa `@ToString.Exclude` en las relaciones Lazy.

### Q9: Explica en detalle el problema "N+1 Selects" en Hibernate y describe las estrategias para solucionarlo

- **El Problema**: Es el problema de rendimiento más común en ORMs. Ocurre cuando cargas una lista de N entidades padre (ej. 10 Autores) y luego, dentro de un bucle, accedes a una relación Lazy (ej. `autor.getLibros()`).
  - Hibernate ejecutará **1** consulta inicial para traer los 10 autores.
  - Luego, al iterar, detectará que los libros no están en memoria y ejecutará N consultas adicionales (una por cada autor) para traer sus libros.
  - Total: 1 + 10 = 11 consultas. Si N es 1000, son 1001 consultas, lo que puede tumbar la base de datos.
- **Soluciones**:
  - `JOIN FETCH` **en JPQL**: Escribir la consulta manualmente: `SELECT a FROM Autor a JOIN FETCH a.libros`. Esto obliga a la base de datos a realizar un JOIN y traer toda la información (Autores y Libros) en una sola consulta SQL optimizada.
  - `@EntityGraph`: Una anotación de Spring Data que permite definir declarativamente qué relaciones cargar ansiosamente (Eagerly) para un método específico del repositorio, sin escribir JPQL.
  - **Batch Fetching**: Configurar Hibernate para que, si necesita cargar libros, no cargue 1, sino un lote (ej. 50) usando `WHERE id IN (...)`. Esto reduce el problema de N+1 a (N/TamañoLote)+1.

### Q10: ¿Cómo funciona la propagación de transacciones (`@Transactional`) por defecto en Spring y qué implicaciones tiene para el manejo de errores?

- **Propagación `REQUIRED` (Default)**: Si el Método A (transaccional) llama al Método B (también transaccional):
  - Spring verifica si ya existe una transacción activa (iniciada por A). Si existe, B se une a esa misma transacción lógica y física. No se crea una nueva.
  - Si no existe, B crea una nueva.
- **Implicación de Rollback**: Dado que comparten la misma transacción física, si el Método B falla y lanza una excepción `RuntimeException` (Unchecked), marcará la transacción global como `rollback-only`.
  - Incluso si el Método A tiene un bloque `try-catch` para capturar el error de B e intentar recuperarse, **no podrá hacer commit**. Al intentar finalizar, Spring detectará la marca de rollback y lanzará una `UnexpectedRollbackException`. Esto garantiza la integridad atómica: o todo se guarda, o nada se guarda. Si necesitas que B falle sin afectar a A, debes usar propagación `REQUIRES_NEW`.

### Q11: ¿Para qué sirve Flyway y cómo se diferencia fundamentalmente de la generación automática de esquemas de Hibernate?

- **Propósito**: Flyway es una herramienta de "Control de Versiones para Base de Datos". Permite evolucionar el esquema de manera segura, reproducible y colaborativa.
- **Funcionamiento**: Gestiona el esquema mediante scripts SQL inmutables y estrictamente versionados (`V1__init.sql`, `V2__add_index.sql`). Flyway mantiene una tabla interna especial (`flyway_schema_history`) en tu base de datos para rastrear qué scripts ya se han ejecutado y calcular checksums para asegurar que nadie modificó un script ya aplicado.
- **Diferencia con Hibernate**: Hibernate intenta "adivinar" el estado deseado de la BD basándose en el código Java actual, lo cual es no determinista y arriesgado. Flyway aplica cambios explícitos escritos por el desarrollador. Flyway es imperativo ("haz esto, luego esto"), mientras que Hibernate ddl-auto es declarativo pero incontrolable en producción. Con Flyway, garantizas que el entorno de Desarrollo, QA y Producción tengan matemáticamente la misma estructura de base de datos.

### Q12: ¿Cuándo elegirías MongoDB sobre PostgreSQL en un proyecto Spring Boot, considerando el Teorema CAP?

- **Elección de MongoDB (NoSQL Documental - CP/AP según config)**:
  1. **Esquema Dinámico**: Cuando los datos no tienen una estructura fija o esta cambia frecuentemente (ej. un catálogo de productos donde una "Camisa" tiene talla y color, pero un "Laptop" tiene CPU y RAM). En SQL esto requeriría muchas tablas o columnas nulas; en Mongo es un documento JSON natural.
  2. **Alta Velocidad de Escritura/Lectura**: Para casos de uso como logs, analítica en tiempo real o perfiles de usuario, donde se necesita insertar o leer el objeto completo rápidamente sin el costo de hacer múltiples JOINs complejos.
  3. **Escalabilidad Horizontal**: Mongo está diseñado para sharding (distribuir datos en múltiples servidores) de forma nativa.
- **Elección de PostgreSQL (SQL Relacional - CA/CP)**:
  1. **Integridad Referencial**: Cuando la consistencia de los datos y las relaciones son críticas (ej. transacciones financieras, inventarios). SQL garantiza ACID estricto.
  2. **Consultas Complejas**: Cuando el patrón de acceso requiere cruzar datos de muchas formas diferentes (Reportes, BI).
- En sistemas modernos ("Polyglot Persistence"), es común usar ambos: Postgres para el núcleo transaccional y Mongo para catálogos o bitácoras.

## Unidad 3: Seguridad, JWT y APIs Avanzadas

### Q13: ¿Cómo funciona detalladamente el flujo de autenticación Stateless con JWT y Spring Security 7?

1. **Login**: El usuario envía sus credenciales (`username`, `password`) vía HTTPS a un endpoint público `/login`.
2. **Generación**: El servidor valida las credenciales contra la BD. Si son correctas, crea un objeto JSON (Payload) con la identidad del usuario (Claims) y sus roles. Firma este JSON criptográficamente usando una clave secreta (HMAC) o privada (RSA) para crear el JWT. Importante: No se crea sesión en el servidor (`HttpSession` es `null`).
3. **Entrega**: El token se devuelve al cliente.
4. **Uso**: Para cualquier petición subsiguiente a un recurso protegido, el cliente debe adjuntar el token en el encabezado HTTP `Authorization: Bearer <token>`.
5. **Validación (Filtro)**: Un filtro personalizado (`OncePerRequestFilter`) en Spring Security intercepta cada petición. Extrae el token, verifica su firma matemática (asegurando que no fue alterado) y su fecha de expiración. Si es válido, extrae los roles y construye un objeto `Authentication` que se inyecta en el `SecurityContext` de ese hilo, permitiendo el acceso.

### Q14: ¿Cuál es la diferencia conceptual y práctica entre Autenticación y Autorización?

- **Autenticación (AuthN)**: Responde a la pregunta "¿Quién eres?". Es el proceso de verificar la identidad del usuario (password, OTP, biometría). Si falla, el sistema devuelve `401 Unauthorized` (aunque semánticamente significa "No autenticado").
- **Autorización (AuthZ)**: Responde a la pregunta "¿Qué tienes permiso para hacer?". Ocurre después de la autenticación. Verifica si el usuario identificado tiene los privilegios o roles necesarios para acceder a un recurso específico.
  - **Ejemplo**: Un usuario puede estar logueado correctamente (Autenticado), pero si intenta acceder a `/admin/delete-users` sin ser administrador, el sistema le denegará el paso devolviendo `403 Forbidden` (Prohibido: sé quién eres, pero no puedes pasar).

### Q15: ¿Qué son los Virtual Threads (Project Loom) y qué ventaja masiva de escalabilidad traen a Spring Boot 4?

- **El Problema de los Hilos del OS**: En el modelo clásico (Thread-per-request), cada petición HTTP asigna un hilo del Sistema Operativo (Platform Thread). Estos hilos son caros en memoria (aprox 1-2MB de stack) y limitados en cantidad. Si tu servidor tiene 8GB de RAM, solo puedes manejar unos pocos miles de hilos concurrentes. Si estos hilos se bloquean esperando una base de datos (I/O), estás desperdiciando recursos.
- **La Solución Virtual**: Los Virtual Threads son hilos ligeros gestionados por la JVM, no por el OS. Son extremadamente baratos (bytes de memoria). Puedes crear millones de ellos.
- **La Magia**: Cuando un Virtual Thread realiza una operación bloqueante (ej. llamar a una API externa), la JVM "desmonta" el Virtual Thread del hilo del OS, liberando el hilo del OS para procesar otra petición inmediatamente. Cuando la operación de I/O termina, la JVM "remonta" el Virtual Thread y continúa.
- **Impacto**: Permite que aplicaciones Spring Boot tradicionales (imperativas/bloqueantes) manejen una concurrencia masiva (similar a la programación reactiva) sin la complejidad de aprender WebFlux o programación asíncrona, usando el mismo hardware.

### Q16: ¿Para qué sirve el patrón Circuit Breaker al consumir APIs externas y cuáles son sus estados?

- **Propósito**: Protege a tu sistema de fallos en cascada. Si tu API "A" depende de un servicio externo "B" (ej. Pasarela de Pagos) y "B" se cae o se vuelve lentísimo, los hilos de "A" se quedarán esperando (bloqueados) hasta el timeout. Si llegan muchas peticiones, agotarás todos los hilos de "A" y tu sistema colapsará también.
- **Funcionamiento**: El Circuit Breaker envuelve la llamada a "B" y monitorea los fallos.
- **Estados**:
  1. **Cerrado (Closed)**: Todo funciona bien. Las llamadas pasan.
  2. **Abierto (Open)**: Se detectaron demasiados fallos consecutivos. El circuito "salta" y corta la conexión. Las nuevas llamadas fallan inmediatamente (Fail Fast) sin esperar timeout, o devuelven una respuesta por defecto (Fallback), protegiendo tus recursos.
  3. **Semi-Abierto (Half-Open)**: Después de un tiempo, el circuito deja pasar algunas peticiones de prueba. Si tienen éxito, se cierra de nuevo (recuperación). Si fallan, vuelve a abrirse.

## Coding Challenge (Live Coding)

**Problema**: "Crea un servicio en Spring Boot que consulte una API externa de Clima. Si la API falla o tarda más de 1 segundo, debe devolver un clima por defecto ('Soleado') para no romper la experiencia del usuario. Debes usar el nuevo `RestClient` y anotaciones de resiliencia."

```Java
@Service
public class WeatherService {

    private final RestClient restClient;

    // Inyectamos el Builder pre-configurado de Spring Boot
    public WeatherService(RestClient.Builder builder) {
        this.restClient = builder.baseUrl("[https://api.weather.com](https://api.weather.com)").build();
    }

    // @CircuitBreaker: Si la tasa de fallos supera el umbral, abre el circuito.
    // @TimeLimiter: Si la llamada tarda más de 1s, lanza TimeoutException.
    // fallbackMethod: Método a invocar en caso de cualquier error (circuito abierto, timeout, 500).
    @CircuitBreaker(name = "weatherService", fallbackMethod = "getDefaultWeather")
    @TimeLimiter(name = "weatherService") 
    public String getCurrentWeather(String city) {
        // Uso de la API fluida de RestClient (Spring 6+)
        return restClient.get()
                .uri("/current?city={city}", city)
                .retrieve()
                .body(String.class);
    }

    // Método Fallback: Debe tener la misma firma que el original + la Excepción
    public String getDefaultWeather(String city, Throwable t) {
        // Es vital loguear el error real (t) para monitoreo y alertas, 
        // aunque al usuario le mostremos un dato degradado.
        System.err.println("Error fetching weather for " + city + ": " + t.getMessage());
        return "Sunny (Fallback data)"; 
    }
}
```
