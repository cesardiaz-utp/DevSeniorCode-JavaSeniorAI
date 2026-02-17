# Proyecto Módulo 1: Sistema de Gestión de Seguros (JavaInsure Core)

## Contexto del Proyecto

La aseguradora **"JavaInsure"** está modernizando su núcleo tecnológico. Necesitan reemplazar su sistema legado por una aplicación de consola robusta, mantenible y moderna.

El sistema debe administrar una cartera de pólizas de seguros, permitiendo la carga masiva de datos desde archivos, el cálculo dinámico de primas basado en reglas de negocio complejas y la generación de reportes financieros. Dado que este es un sistema financiero, la **calidad del código (Testing)** y la **trazabilidad (Logging)** son requisitos mandatorios.

## Visión General del Problema

"JavaInsure" enfrenta el desafío de administrar una cartera creciente y diversa de seguros. Actualmente, el cálculo de las primas (el costo anual que paga el cliente) se realiza mediante procesos manuales o dispersos, lo que genera errores financieros y lentitud en la atención al cliente.

El problema central que este sistema debe resolver es la centralización y automatización de las reglas de negocio para diferentes líneas de productos. La aseguradora maneja tres tipos de riesgos fundamentales, cada uno con su propia lógica de valoración:

1. **Seguros de Vida**: El riesgo aquí está asociado a la persona. La aseguradora necesita una forma de penalizar automáticamente las primas para clientes de edad avanzada, ya que representan un riesgo mayor de reclamo.
2. **Seguros de Automóvil**: El riesgo está asociado al bien mueble. El sistema debe ser capaz de diferenciar entre autos nuevos y autos antiguos, aplicando recargos a estos últimos debido a la mayor probabilidad de fallas mecánicas o dificultad para conseguir repuestos.
3. **Seguros de Hogar**: El riesgo está asociado a la ubicación. Es vital identificar si un inmueble se encuentra en una "Zona de Riesgo" (inundaciones, sismos) para ajustar la prima acorde a la exposición real del activo.

Finalmente, la gerencia necesita dejar de operar a ciegas. El sistema debe consolidar toda esta información dispersa para ofrecer una visión financiera clara, permitiendo saber cuánto se está recaudando en promedio y cuáles son los contratos de mayor valor para la compañía.

## Requerimientos Funcionales

### 1. Carga Inicial de Datos (Persistencia en Archivos)

- Al iniciar, el sistema debe buscar un archivo llamado `policies.csv`.
- Debe leer el archivo línea por línea usando `java.nio.file.Files.lines()` (para eficiencia de memoria).
- Debe parsear cada línea para convertirla en objetos del dominio (`Insurance`) y cargarlos en memoria.
- Formato CSV esperado: `TYPE,CLIENT_ID,AMOUNT,EXTRA_DATA...`

### 2. Gestión de Pólizas (CRUD en Memoria)

El sistema debe presentar un menú de consola que permita:

- **Listar Pólizas**: Mostrar todas las pólizas cargadas.
- **Agregar Póliza**: Solicitar datos al usuario para crear una nueva póliza (Life, Car o Home).
- **Buscar**: Encontrar una póliza por ID de cliente.

### 3. Especificaciones del Motor de Cotización

El sistema debe calcular el costo de la prima anual basándose en el tipo de seguro:

| Tipo de Seguro | Base del Cálculo | Regla Extra |
| --- | --- | --- |
| **Life Insurance** | 5% del monto asegurado | Si edad > 60, sumar 20% al total. |
| **CarInsurance** | 10% del valor del auto | Si modelo < 2015, sumar $50 USD fijos. |
| **HomeInsurance** | 2% del valor del inmueble | Si `isRiskZone` es true, la prima se duplica. |

### 4. Reportes Gerenciales

Generar estadísticas en tiempo real:

- Promedio de costo de pólizas.
- Conteo de pólizas por Tipo (cuántas de Life, cuántas de Car, etc).
- Top 3 de las pólizas más costosas.

## Requerimientos Técnicos

### A. Modelo de Dominio

- **Sealed Interface**: `Insurance` que permita (`permits`) las implementaciones: `LifeInsurance`, `CarInsurance` y `HomeInsurance`.
- **Records**: Todas las pólizas y la clase `Client` deben ser implementadas como **records** para garantizar inmutabilidad.
- **Pattern Matching**: El cálculo de primas **DEBE** realizarse en una clase `InsuranceQuoter` utilizando un `switch expression` con **Pattern Matching**. Queda prohibido el uso de `instanceof` con cadenas `if-else`.

### B. Persistencia y Colecciones

- **File Loading**: Uso obligatorio de `java.nio.file.Files.lines()` para leer el archivo `policies.csv` de forma perezosa (Lazy Loading).
- **Sequenced Collections**: Uso de `ArrayList` (vía la interfaz `SequencedCollection`) para mantener el orden de registro.
- **Streams API**: Todos los reportes, búsquedas y filtros deben realizarse exclusivamente con Streams. No se permite el uso de bucles `for` o `while` para procesar colecciones.

### C. Calidad y Auditoría

- **Custom Exceptions**:
  - Crear `InvalidPolicyDataException` (Checked) para errores en el CSV.
  - Crear `BusinessRuleException` (Unchecked) para violaciones lógicas (ej. edad negativa).
- **Logging**:
  - **NO usar** `System.out.println` para informar errores o flujo interno.
  - Usar `logger.info()` para registrar "Policy created successfully".
  - Usar `logger.error()` con el stack trace cuando falle la lectura del CSV.
  - Usar `System.out` **solo** para la interacción directa del menú con el usuario.
- **Testing (JUnit 5 + Mockito)**:
  - Crear `InsuranceQuoterTest`: Verificar que las matemáticas de las primas sean exactas para cada tipo de seguro y sus casos bordes.
  - Crear `FileHandlerTest`: Usar **Mockito** o archivos temporales para probar la lectura del CSV sin depender de un archivo real en disco duro fijo (opcional: o testear lógica de parseo aislada).

## Rúbrica de Evaluación

El proyecto se evaluará sobre **100 puntos**:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Modelado Moderno** | Uso correcto de `sealed interface`, `permits` y `records`. El código es conciso y usa inmutabilidad por defecto. | 15 |
| **Lógica & Pattern Matching** | Implementación correcta del `switch` expression con pattern matching para el cálculo de costos. Lógica de negocio libre de errores. | 20 |
| **Streams & NIO** | Lectura eficiente del archivo con `Files.lines()`. Uso elegante de Streams para los reportes (`filter`, `map`, `collect`, `groupingBy`). | 20 |
| **Calidad & Logging** | Uso correcto de **Log4j 2** (niveles info/error). Manejo apropiado de Excepciones Custom (`try-catch` donde corresponde). Código limpio y organizado. | 15 |
| **Testing (JUnit)** | Suite de pruebas unitarias que cubre al menos los casos de éxito y un caso de error (excepción) para el cálculo de costos. Uso de aserciones correctas. | 30 |

## Niveles de Desempeño

- **Senior (90-100)**: Cumple todos los requisitos. Incluye tests exhaustivos (cobertura > 80%), utiliza `Collectors.teeing` o agrupamientos complejos en Streams. El código es impecable (Clean Code).
- **Semi-Senior (75-89)**: Funciona correctamente y usa Java moderno. Faltan algunos casos borde en los tests o el manejo de excepciones no es totalmente exhaustivo.
- **Junior (60-74)**: Funciona, pero utiliza bucles tradicionales en lugar de Streams en algunas partes o el logging es muy básico.
- **Insuficiente (<60)**: No compila, no lee el archivo CSV o no utiliza el paradigma de Objetos solicitado.

## Entregables

1. **Repositorio de GitHub**: Código fuente organizado por paquetes (`model`, `service`, `exception`, `util`).
2. **README.md**: Instrucciones claras de ejecución.
3. **Archivo `policies.csv`**: Ejemplo con al menos 10 registros de diferentes tipos.
4. **Video Demostrativo (Max 5 min)**:
    - Explicación de la jerarquía de `Insurance`.
    - Ejecución de los reportes (Promedio, conteo por tipo, Top 3).
    - Demostración de los Tests en verde.
