# Proyecto Integrador Módulo 1: "Sistema de Gestión de Seguros (JavaInsure Core)"

## Descripción del Escenario

La aseguradora "JavaInsure" está modernizando su núcleo tecnológico. Necesitan reemplazar su sistema legado por una aplicación de consola robusta, mantenible y moderna.

El sistema debe administrar una cartera de pólizas de seguros, permitiendo la carga masiva de datos desde archivos, el cálculo dinámico de primas basado en reglas de negocio complejas y la generación de reportes financieros. Dado que este es un sistema financiero, la **calidad del código (Testing)** y la **trazabilidad (Logging)** son requisitos no negociables, no características opcionales.

## Objetivos de Aprendizaje Evaluados

Este proyecto valida la competencia del estudiante en:

1. **Java Moderno (21-25)**: Uso de _Records_, _Sealed Classes_ y _Pattern Matching_ para un modelo de dominio expresivo.
2. **Manipulación de Datos**: Uso avanzado de la **Streams API** y **Java NIO** para procesar archivos y colecciones.
3. **Calidad de Software**: Implementación de Pruebas Unitarias con **JUnit 5** y aislamiento con **Mockito**.
4. **Operaciones Empresariales**: Uso de **Log4j 2** para auditoría y manejo profesional de Excepciones.

## Visión General del Sistema

"JavaInsure" enfrenta el desafío de administrar una cartera creciente y diversa de seguros. Actualmente, el cálculo de las primas (el costo anual que paga el cliente) se realiza mediante procesos manuales o dispersos, lo que genera errores financieros y lentitud en la atención al cliente.

El problema central que este sistema debe resolver es la centralización y automatización de las reglas de negocio para diferentes líneas de productos. La aseguradora maneja tres tipos de riesgos fundamentales, cada uno con su propia lógica de valoración:

1. **Seguros de Vida**: El riesgo aquí está asociado a la persona. La aseguradora necesita una forma de penalizar automáticamente las primas para clientes de edad avanzada, ya que representan un riesgo mayor de reclamo.
2. **Seguros de Automóvil**: El riesgo está asociado al bien mueble. El sistema debe ser capaz de diferenciar entre autos nuevos y autos antiguos, aplicando recargos a estos últimos debido a la mayor probabilidad de fallas mecánicas o dificultad para conseguir repuestos.
3. **Seguros de Hogar**: El riesgo está asociado a la ubicación. Es vital identificar si un inmueble se encuentra en una "Zona de Riesgo" (inundaciones, sismos) para ajustar la prima acorde a la exposición real del activo.

Finalmente, la gerencia necesita dejar de operar a ciegas. El sistema debe consolidar toda esta información dispersa para ofrecer una visión financiera clara, permitiendo saber cuánto se está recaudando en promedio y cuáles son los contratos de mayor valor para la compañía.

## Requerimientos Funcionales (Lo que hace el sistema)

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

### 3. Motor de Cotización (Reglas de Negocio)

El sistema debe calcular el costo de la prima anual basándose en el tipo de seguro:

- **Seguro de Vida (Life Insurance)**:
  - **Base**: 5% del monto asegurado.
  - **Regla**: Si la edad del cliente > 60, sumar 20% extra.
- **Seguro de Auto (Car Insuranc)**:
  - **Base**: 10% del valor del auto.
  - **Regla**: Si el modelo es anterior a 2015, sumar 50 USD de "impuesto por antigüedad".
- **Seguro de Hogar (Home Insurance)**:
  - **Base**: 2% del valor del inmueble.
  - **Regla**: Si está en "Zona de Riesgo", la prima se duplica.

### 4. Reportes Gerenciales

Generar estadísticas en tiempo real:

- Promedio de costo de pólizas.
- Conteo de pólizas por Tipo (cuántas de Life, cuántas de Car, etc).
- Top 3 de las pólizas más costosas.

## Requerimientos Técnicos (Cómo se construye)

### 1. Modelo de Dominio

- **Interfaces Selladas**: Crear `sealed interface Insurance permits LifeInsurance, CarInsurance, HomeInsurance`.
- **Records**: Todas las implementaciones (`LifeInsurance`, etc.) y la clase `Client` deben ser **records**.
- **Pattern Matching**: El cálculo de costos **DEBE** realizarse en una clase `InsuranceQuoter` utilizando un `switch` expression con `Pattern Matching` sobre la interfaz `Insurance`. No se permite el uso de `instanceof` en cadenas `if-else`.

### 2. Calidad y Robustez

- **Excepciones**:
  - Crear `InvalidPolicyDataException` (Checked) para errores en el CSV.
  - Crear `BusinessRuleException` (Unchecked) para violaciones lógicas (ej. edad negativa).
- **Logging (Log4j 2)**:
  - **NO usar** `System.out.println` para informar errores o flujo interno.
  - Usar `logger.info()` para registrar "Policy created successfully".
  - Usar `logger.error()` con el stack trace cuando falle la lectura del CSV.
  - Usar `System.out` **solo** para la interacción directa del menú con el usuario.
- **Testing (JUnit 5 + Mockito)**:
  - Crear `InsuranceQuoterTest`: Verificar que las matemáticas de las primas sean exactas para cada tipo de seguro y sus casos bordes.
  - Crear `FileHandlerTest`: Usar **Mockito** o archivos temporales para probar la lectura del CSV sin depender de un archivo real en disco duro fijo (opcional: o testear lógica de parseo aislada).

### 3. Colecciones

- Usar `SequencedCollection` (como `ArrayList`) para mantener el orden de inserción.
- Todas las búsquedas y reportes deben hacerse exclusivamente con **Streams API** (prohibido usar bucles `for` tradicionales para los reportes).

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**, distribuidos de la siguiente manera:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Modelado Moderno** | Uso correcto de `sealed interface`, `permits` y `records`. El código es conciso y usa inmutabilidad por defecto. | 15 |
| **Lógica & Pattern Matching** | Implementación correcta del `switch` expression con pattern matching para el cálculo de costos. Lógica de negocio libre de errores. | 20 |
| **Streams & NIO** | Lectura eficiente del archivo con `Files.lines()`. Uso elegante de Streams para los reportes (`filter`, `map`, `collect`, `groupingBy`). | 20 |
| **Calidad & Logging** | Uso correcto de **Log4j 2** (niveles info/error). Manejo apropiado de Excepciones Custom (`try-catch` donde corresponde). Código limpio y organizado. | 15 |
| **Testing (JUnit)** | Suite de pruebas unitarias que cubre al menos los casos de éxito y un caso de error (excepción) para el cálculo de costos. Uso de aserciones correctas. | 30 |

## Niveles de Desempeño

- **Senior (90-100)**: Cumple todo, incluye tests exhaustivos con Mockito, usa Streams complejos y el código es impecable.
- **Semi-Senior (75-89)**: Funciona correctamente, usa las features modernas, pero faltan algunos tests o el logging es básico.
- **Junior (60-74)**: Funciona pero usa bucles `for` en lugar de Streams, o no implementa tests, o usa `System.out` para errores.
- **Insuficiente (<60)**: No compila, no lee el archivo o no usa conceptos de Objetos.

## Entregables

1. Enlace al repositorio de GitHub.
2. Archivo `README.md` detallado con instrucciones de ejecución y pre-requisitos.
3. Archivo `policies.csv` de ejemplo incluido en la raíz del proyecto.
4. Captura de pantalla de la ejecución de los Tests en verde.
