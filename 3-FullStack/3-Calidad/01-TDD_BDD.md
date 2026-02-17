# Unidad 3 - Clase 1: Principios de Calidad: TDD y BDD como Pilares de Arquitectura

- **Duración**: 2 horas
- **Objetivo**: En el ecosistema de **Spring Boot 4** y **Angular 21**, la calidad no es una capa externa, sino un atributo intrínseco de la arquitectura. Esta clase profundiza en cómo el **Test-Driven Development (TDD)** y el **Behavior-Driven Development (BDD)** eliminan la brecha entre los requisitos de negocio y la implementación técnica, garantizando sistemas que pueden evolucionar sin miedo al colapso por regresiones..

## Parte 1: Teoría (60 min)

### 1. La Pirámide de Pruebas: Estrategia y Balance

![La Pirámide de Testing](https://bluebirdinternational.com/wp-content/uploads/2023/08/agile-testing-pyramid-bluebird-scaled.jpg)

La pirámide de pruebas es un modelo que nos indica cómo distribuir nuestros esfuerzos de automatización para obtener la máxima confianza al menor costo de mantenimiento, abarcando desde el backend en Java hasta el frontend reactivo.

#### A. Pruebas Unitarias (La Base - 70%)

Son el cimiento de la calidad. Validan la unidad más pequeña de código de forma aislada, garantizando que los bloques de construcción individuales funcionen correctamente.

- **Objetivo**: Verificar lógica de dominio, algoritmos de cálculo y transformaciones de datos.
- **Enfoque Spring Boot 4**: Uso intensivo de **JUnit 5** y **Mockito**. Se prueban POJOs, Records y Servicios aislados del contexto de Spring para asegurar ejecuciones en milisegundos. Se aprovechan los _Text Blocks_ de Java 25 para documentar casos de prueba complejos.
- **Enfoque Angular 21**: Al ser **Zoneless** (sin Zone.js), las pruebas de **Signals** son atómicas y puramente sincrónicas. Se prueba la lógica de los servicios y componentes sin necesidad de manejar ciclos de detección de cambios pesados.
- **Aislamiento**: Mocks para dependencias externas (bases de datos en el back, servicios HTTP en el front).

#### B. Pruebas de Integración (El Cuerpo - 20%)

Verifican la comunicación y el ensamblaje entre diferentes módulos o sistemas externos.

- **Objetivo**: Validar persistencia, seguridad, contratos de API y flujos de datos entre capas.
- **Enfoque Spring Boot 4**: El estándar es **Testcontainers**. En lugar de usar bases de datos en memoria (H2), se levantan contenedores Docker reales (PostgreSQL, Kafka, Redis) para garantizar que el código se comporte exactamente como en producción. Se utiliza `@SpringBootTest` o `@DataJpaTest`.
- **Enfoque Angular 21**: Pruebas de integración de componentes utilizando el `TestBed` para verificar la interacción entre el template (DOM) y la lógica del componente, asegurando que los eventos disparen los Signals correctos y la UI responda.
- **Compromiso**: Son más lentas pero detectan errores de configuración (SQL incorrecto, errores de inyección) que las unitarias ignoran.

#### C. Pruebas E2E / UI (La Cúspide - 10%)

Simulan el comportamiento de un usuario real interactuando con la aplicación completa "viva".

- **Objetivo**: Validar flujos críticos de negocio de principio a fin (ej. registro de paciente -> diagnóstico -> cobro).
- **Stack Tecnológico**: Uso de **Playwright** o **Cypress** para orquestar la navegación.
- **Alcance**: Se atraviesan todas las capas: desde el click en el botón de Angular 21, pasando por el controlador REST de Spring 4, hasta la persistencia en la base de datos y la respuesta de vuelta a la UI.
- **Desventaja**: Son frágiles y lentas; deben limitarse a los "Happy Paths" más importantes para no ralentizar el pipeline de CI/CD.

### 2. TDD: Más allá de "Escribir Tests antes"

El **Test-Driven Development (TDD)** no es simplemente una técnica de "probar antes de programar"; es una metodología de **Ingeniería de Software** que utiliza las pruebas para guiar el diseño y asegurar la calidad desde el primer segundo.

#### ¿Por qué es Vital para la Alta Calidad?

1. **Código Justo y Necesario**: TDD evita la "sobre-ingeniería" (Gold Plating). Solo se escribe el código que satisface un requerimiento probado.
2. **Arquitectura Desacoplada**: Para que una clase sea testable antes de existir, sus dependencias deben estar bien definidas. Esto obliga al uso de inyección de dependencias e interfaces claras (SOLID).
3. **Documentación Ejecutable**: Los tests se convierten en la fuente de verdad sobre lo que el sistema hace, eliminando la obsolescencia de la documentación técnica tradicional.
4. **Confianza Total en el Refactoring**: Permite mejorar el código continuamente (limpiar deuda técnica) con la seguridad de que una red de seguridad detectará cualquier error al instante.

#### Estilos de TDD

La comunidad de TDD se divide principalmente en dos escuelas de pensamiento que abordan el desarrollo desde ángulos opuestos:

| Característica | Estilo Detroit (Clásico / Sociable) | Estilo London (Mockista / Solitario) |
| --- | --- | --- |
| **Origen** | Kent Beck (Extreme Programming) | Steve Freeman / Nat Pryce (GOOS) |
| **Enfoque** | Estado: Verifica que el resultado sea correcto. | Interacción: Verifica cómo colaboran los objetos. |
| **Dirección** | **Inside-Out**: Empieza por el dominio (núcleo). | **Outside-In**: Empieza por la periferia (API/UI). |
| **Uso de Mocks** | Mínimo. Se usan objetos reales siempre que se pueda. | Intensivo. Se "mockean" todas las dependencias. |
| **Arquitectura** | Favorece el diseño basado en valores y lógica pura. | Favorece el diseño basado en interfaces y roles. |

#### Estilo Detroit (Inside-Out)

Se centra en construir el corazón de la aplicación primero. Escribes tests para las clases del dominio y vas subiendo hacia las capas exteriores.

- **Cómo aborda el desarrollo**: Se enfoca en que la lógica sea correcta. Si una clase `A` usa a `B`, el test de `A` usa la implementación real de `B`.
- **Riesgo**: Un error en una clase base puede hacer fallar cientos de tests (efecto dominó).

#### Estilo London (Outside-In)

Se centra en la necesidad del usuario final. Empiezas definiendo el controlador o la interfaz externa y usas "Mocks" para representar lo que aún no has construido.

- **Cómo aborda el desarrollo**: Define el "protocolo" de comunicación. "Yo necesito que este servicio me devuelva un dato, no me importa cómo lo haga todavía".
- **Riesgo**: Si no se tiene cuidado, se puede terminar con tests "frágiles" que están demasiado acoplados a la implementación interna (los métodos que llamas) en lugar del resultado.

#### La técnica de Triangulación

![Test Driven Development](https://www.icterra.com/wp-content/uploads/2020/01/Test_01-1.png)

La triangulación es una estrategia de diseño que evita que el desarrollador cree implementaciones "accidentales" o demasiado específicas. Se basa en la premisa de que **necesitas al menos dos puntos de datos para definir una regla general**.

1. **El Problema del Primer Test**: Cuando escribes el primer test (Fase RED) y pasas a GREEN, la forma más rápida suele ser "hardcodear" (quemar) el resultado. Esto es correcto en TDD, pero es una solución incompleta.
2. **La Intervención del Segundo Test**: Al añadir un segundo test con datos diferentes, la solución "hardcodeada" falla.
3. **La Transformación Algorítmica**: La necesidad de que ambos tests pasen simultáneamente obliga al cerebro (y al código) a abandonar la constante y buscar la **variable** o la **fórmula**.

##### ¿Por qué es vital para la calidad?

- **Elimina Falsos Positivos**: Un solo test puede pasar por coincidencia. Dos tests con valores distintos prueban la veracidad del algoritmo.
- **Inducción Matemática**: Nos permite movernos de lo concreto (ej. `2+2=4`) a lo abstracto (ej. `a+b=c`) con seguridad.
- **Refactorización Segura**: Proporciona los "raíles" necesarios para que, al limpiar el código, estemos seguros de que la lógica no se rompe para casos variados.

### 3. BDD: Behavior-Driven Development (Diseño Orientado al Comportamiento)

BDD es una evolución de TDD que desplaza el foco de "probar el código" a "especificar el comportamiento". Fue creado por Dan North para resolver la pregunta: _¿Por dónde empezar a testear y qué testear?_

#### A. La Filosofía: "Especificación mediante Ejemplos"

En lugar de documentos de requisitos ambiguos, BDD propone ejemplos concretos que se convierten en pruebas automatizadas. Esto garantiza que lo que se construye es exactamente lo que el negocio necesita.

#### B. El Proceso de los "Tres Amigos"

BDD fomenta la reunión de tres roles clave antes de escribir una sola línea de código:

1. **Negocio (Product Owner)**: Define el _qué_ y el _para qué_.
2. **Desarrollo**: Define el _cómo_ técnico.
3. **QA/Tester**: Define los casos de borde y escenarios de falla.

#### C. Gherkin: El Lenguaje de la Ubicuidad

Para que todos se entiendan, se utiliza Gherkin, un lenguaje natural estructurado:

- **Feature**: Describe la funcionalidad global.
- **Scenario**: Un ejemplo específico de uso.
- **Given (Dado)**: Contexto inicial o estado del sistema.
- **When (Cuando)**: La acción o evento clave.
- **Then (Entonces)**: El resultado observable (la verificación).

#### D. Beneficios en el Stack Moderno

- **Spring Boot 4**: Permite usar Cucumber o JBehave para que los tests de integración lean directamente archivos `.feature`, asegurando que el backend cumple el contrato de negocio.
- **Angular 21**: Facilita la prueba de la experiencia de usuario (UX) validando que el flujo de **Signals** responda a las acciones del lenguaje natural descrito en los escenarios.

## Parte 2: Laboratorio TDD y Refactorización (45 min)

### 1. El Escenario de Negocio: Gestión de Pólizas "Vida Sana"

Trabajamos para una aseguradora que está migrando su núcleo a **Spring Boot 4**. El departamento de riesgos necesita un **Motor de Reglas de Beneficios** que determine automáticamente los descuentos aplicables a los asegurados según su perfil demográfico.

**El Problema**: Actualmente, las reglas se aplican manualmente o mediante hojas de cálculo, lo que genera errores en la facturación. El negocio requiere una lógica centralizada donde la regla principal es el **"Beneficio de Longevidad"**.

#### Reglas de Negocio (Requerimientos)

1. **Segmento General**: Asegurados menores de 65 años no tienen descuento adicional (Tarifa base).
2. **Segmento Senior**: Asegurados con 65 años o más deben recibir un descuento del 20% sobre su prima mensual.
3. **Restricción Técnica**: El motor debe ser capaz de evolucionar para incluir reglas por enfermedades preexistentes en el futuro sin romper lo anterior.

### 2. Especificación BDD (Gherkin)

Antes de programar, los "Tres Amigos" (PO, Dev, QA) definen el contrato con los dos escenarios principales:

```plain
Feature: Descuento por Edad en Pólizas
  Como Gerente de Riesgos
  Quiero que el sistema identifique el perfil del asegurado
  Para aplicar o denegar el descuento de longevidad correctamente.

  Scenario: Cliente en segmento general no recibe descuento
    Given un asegurado con 60 años
    When el sistema calcula el beneficio
    Then el descuento aplicado debe ser del 0%

  Scenario: Aplicar descuento a ciudadanos senior
    Given un asegurado con 65 años
    When el sistema calcula el beneficio
    Then el descuento aplicado debe ser del 20%
```

### 3. Implementación Técnica (TDD)

#### Paso 1: Fase RED (Caso Base - Segmento General)

Usamos **Text Blocks** para que el test sea nuestra documentación técnica basada en el primer escenario.

```java
@Test
@DisplayName("""
    Scenario: Cliente en segmento general no recibe descuento
      Given un asegurado con 60 años
      When el sistema calcula el beneficio
      Then el descuento aplicado debe ser del 0%
    """)
void age60NoDiscount() {
    var engine = new InsuranceEngine();
    assertEquals(0.0, engine.getDiscount(60));
}
```

#### Paso 2: Fase GREEN (Minimalista)

```java
public class InsuranceEngine {
    public double getDiscount(int age) {
        return 0.0; // Satisface el requerimiento del primer test
    }
}
```

#### Paso 3: Triangulación (Forzando la Abstracción con el Segundo Escenario)

Añadimos el caso Senior del segundo escenario BDD para romper la constante `0.0`.

```java
@Test
@DisplayName("""
    Scenario: Aplicar descuento a ciudadanos senior
      Given un asegurado con 65 años
      When el sistema calcula el beneficio
      Then el descuento aplicado debe ser del 20%
    """)
void age65SeniorDiscount() {
    var engine = new InsuranceEngine();
    assertEquals(0.20, engine.getDiscount(65));
}
```

#### Paso 4: Fase REFACTOR (Hacia el Algoritmo Real)

```java
public class InsuranceEngine {
    public double getDiscount(int age) {
        // La triangulación forzada por los dos escenarios BDD 
        // nos obligó a implementar la lógica de negocio genérica
        return (age >= 65) ? 0.20 : 0.0;
    }
}
```

## Parte 3: Resumen y discusión arquitectónica (15 min)

### Debate de cierre

1. ¿Cómo el haber definido ambos escenarios en Gherkin evitó ambigüedades en el código?
2. ¿Qué pasaría si el negocio pide que a los 80 años el descuento sea del 30%? (Necesidad de un tercer escenario y nueva triangulación).

### Recursos y referencias

#### Bibliografía Fundamental

- **Kent Beck - TDD by Example**: El libro que definió la metodología y la técnica de triangulación.
- **Steve Freeman & Nat Pryce - Growing Object-Oriented Software, Guided by Tests**: La "biblia" del Estilo London y el uso de Mocks.
- **Robert C. Martin - Clean Code**: Capítulo sobre pruebas unitarias y las reglas F.I.R.S.T.
- **Gojko Adzic - Specification by Example**: Cómo BDD ayuda a construir el software correcto.

#### Documentación y Herramientas

- **Dan North**: [Introducing BDD](https://dannorth.net/blog/introducing-bdd/) - El artículo fundacional de BDD.
- **Cucumber.io**: [Gherkin Reference](https://cucumber.io/docs/gherkin/reference/) - Guía completa de sintaxis para escenarios.
- **JUnit 6 User Guide**: [Writing Tests](https://docs.junit.org/6.0.3/writing-tests/intro.html) - Documentación oficial para Java moderno.
- **Angular.dev**: [Testing Guide](https://angular.dev/guide/testing) - Mejores prácticas para testear componentes y Signals.

#### Artículos y Blogs de Arquitectura

- **Martin Fowler**: [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html): Una explicación detallada sobre cómo estructurar las pruebas en un proyecto moderno.
- **Martin Fowler**: [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html): El artículo definitivo para entender la diferencia entre los estilos de TDD (Detroit vs. London).
- **Dan North**: [Introducing BDD](https://dannorth.net/introducing-bdd/): El artículo original donde se presentó el concepto de BDD por primera vez.
