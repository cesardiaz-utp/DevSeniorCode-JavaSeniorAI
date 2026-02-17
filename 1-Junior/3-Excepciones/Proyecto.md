# Unidad 3: PrestaYa Resilience

## Contexto del Proyecto

La plataforma **PrestaYa** está escalando. Ahora, el sistema debe ser capaz de manejar situaciones críticas: ingresos de datos inválidos por parte de los asesores, falta de fondos en la cuenta de desembolso y la integración con servicios externos de validación (Datacrédito).

El estudiante debe transformar el motor de créditos en un sistema robusto que utilice **JUnit 5**, **Mockito** y **Log4j 2** para garantizar la calidad del software.

## Especificaciones de Calidad y Testing

### A. Gestión de Errores

Se deben definir e implementar las siguientes excepciones de negocio (Custom Exceptions):

- `InsufficientPortfolioFundsException`: Se lanza cuando se intenta aprobar un crédito pero el monto solicitado supera el capital disponible en la plataforma.
- `InvalidApplicationException`: Se lanza si los datos de la solicitud son inconsistentes (ej: plazos negativos, nombres vacíos).

### B. Registro de Auditoría

Implementar **SLF4J con Log4j 2** para registrar eventos críticos:

- **Nivel INFO**: Registro de cada solicitud procesada (aprobada o rechazada).
- **Nivel WARN**: Intentos de registro con datos inválidos que fueron corregidos o capturados.
- **Nivel ERROR**: Excepciones de negocio lanzadas y errores inesperados del sistema.
- **Configuración**: El archivo `log4j2.xml` debe configurar un _RollingFile Appender_ que guarde los logs en una carpeta llamada `logs/`.

### C. Aislamiento con Mocks

El `PortfolioManager` ahora depende de un servicio externo de consulta: `CreditRegistryService` (Interface).

- El estudiante debe crear este servicio con un método `int getScore(String customerId)`.
- En las pruebas unitarias, se debe usar **Mockito** para simular diferentes puntajes (ej: simular un 500 para probar rechazos y un 800 para probar aprobaciones) sin necesidad de una base de datos real.

## Especificaciones de Testing

El estudiante debe entregar una suite de pruebas unitarias que cubra:

1. **Pruebas de Cálculo**: Verificar que `calculateMonthlyInstallment()` arroja el valor exacto con `BigDecimal`.
2. **Pruebas de Excepciones**: Usar `assertThrows` para verificar que el sistema lanza `InsufficientPortfolioFundsException` cuando no hay fondos.
3. **Pruebas de Flujo (Mocks)**: Verificar que si el `CreditRegistryService` devuelve un puntaje bajo, el estado de la aplicación cambia a `RejectedStatus`.

## Requerimientos Técnicos (Obligatorios)

1. **Estructura Maven**: Uso de dependencias para JUnit 5, Mockito y Log4j 2.
2. **Manejo de Recursos**: Uso de `try-with-resources` si el sistema lee configuraciones o guarda reportes en archivos `.txt`.
3. **Robustez del Código**: El menú principal (`while`) debe ser inmune a errores de teclado (ej: si el usuario ingresa una letra donde se espera un número, el programa debe capturar el error y reintentar sin cerrarse).

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Testing (JUnit 5)** | Suite de pruebas que cubre casos de éxito, bordes y lanzamiento de excepciones. Uso de aserciones correctas. | 30 |
| **Aislamiento (Mockito)** | Uso correcto de Mocks para simular dependencias externas. Verificación de interacciones (`verify`). | 20 |
| **Manejo de Excepciones** | Implementación de excepciones personalizadas y uso correcto de bloques `try-catch` para robustecer la UI de consola. | 20 |
| **Logging Profesional** | Configuración de Log4j 2 mediante XML y uso de la fachada SLF4J en las clases de negocio. | 15 |
| Calidad de Entrega | Código limpio, modular, historial de commits y video explicativo del proceso de testing. | 15 |

## Niveles de Desempeño

- **Senior (90-100)**: Logra una cobertura de código significativa (>80%). Los tests son legibles (Clean Tests), usa Mocks de forma avanzada y la aplicación es virtualmente imposible de "romper" desde la consola.
- **Semi-Senior (75-89)**: Implementa los tests y las excepciones, pero la cobertura es parcial o el logging no está configurado para rotar archivos.
- **Junior (60-74)**: Realiza tests básicos, pero no usa Mocks adecuadamente (instancia las dependencias reales) o usa printStackTrace() en lugar de un Logger.
- **Insuficiente (<60)**: El código no tiene pruebas unitarias o la aplicación se cierra ante cualquier error de entrada del usuario.

## Entregables

1. **Repositorio de GitHub**:
    - Carpeta `src/main/java` para el código y `src/test/java` para las pruebas.
    - Archivo `log4j2.xml` configurado.
2. **Video Demostrativo (Max 5 min)**:
    - Mostrar la ejecución de los tests desde el Test Runner de VS Code (todos en verde).
    - Provocar errores en la consola (inputs inválidos) para mostrar cómo el sistema los gestiona sin caerse.
    - Mostrar el contenido del archivo de log generado después de usar la aplicación.
