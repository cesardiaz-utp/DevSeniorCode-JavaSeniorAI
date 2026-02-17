# Unidad 2: PrestaYa Fintech

## Contexto del Proyecto

En Colombia, el acceso a crédito formal es un reto para los emprendedores de economía popular (vendedores ambulantes, artesanos, pequeños tenderos). PrestaYa es una plataforma de microcréditos diseñada para gestionar préstamos de bajo monto con tasas de interés diferenciadas.

El estudiante debe modelar el motor de créditos de la plataforma utilizando Programación Orientada a Objetos avanzada y asegurar que cada centavo sea calculado con precisión bancaria.

## Especificaciones de Modelado

El sistema debe estar compuesto por las siguientes estructuras obligatorias:

### A. Gestión Financiera

- **Uso de `BigDecimal`**: Obligatorio para todos los campos de moneda (`requestedAmount`, `interestRate`, `monthlyInstallment`). Se debe configurar con `RoundingMode.HALF_UP`.
- **Record `Customer`**: Representa al solicitante. Debe contener: `id` (String), `name` (String), `creditScore` (int) y `city` (String).

### B. Jerarquía de Créditos

Modelar los diferentes productos financieros que ofrece la plataforma:

- **Clase Abstracta `Loan`**: Campos: `loanId`, `amount`, `termMonths`. Método abstracto `calculateMonthlyInstallment()`.
- **Subclase `EntrepreneurLoan`**: Destinado a negocios. Tiene una tasa de interés preferencial fija del **1.2%** mensual.
- **Subclase `SocialHousingLoan`**: Destinado a mejoras de hogar. Incluye un campo adicional `governmentSubsidieAmount` que se resta al monto total antes de calcular la cuota.

### C. Sistema de Evaluación

Para manejar la respuesta de una solicitud de crédito, se debe implementar una **Interfaz Sellada** llamada `ApplicationStatus`:

1. **Record `ApprovedStatus`**: Contiene el `approvedAmount` y la `calculatedInstallment`.
2. **Record `RejectedStatus`**: Contiene el `rejectionReason` (Ej: "Puntaje de crédito insuficiente").

Se debe usar un **Switch con Pattern Matching** para imprimir la respuesta adecuada al usuario en la consola.

### D. Cartera de Créditos

- **Clase `PortfolioManager`**: Debe contener un **Array de objetos** `Loan` para almacenar las solicitudes procesadas con éxito (capacidad fija de 10 elementos).
- **Métodos**: `registerLoan(Loan loan)`, `showPortfolio()` y `calculateTotalPlacedAmount()` (suma de todos los montos de créditos aprobados).

## Funcionalidades del Menú (Consola)

1. **Simular Nueva Solicitud (Simulate New Application)**:
    - Solicitar datos del cliente y tipo de crédito.
    - Evaluar según el puntaje de Datacrédito: si el puntaje es menor a 600, generar un `RejectedStatus`.
2. **Aprobar y Registrar (Approve and Register)**:
    - Si la simulación es positiva, crear el objeto (`EntrepreneurLoan` o `SocialHousingLoan`) y guardarlo en el array de `PortfolioManager`.
3. **Ver Cartera Total (View Total Portfolio)**:
    - Listar todos los créditos aprobados mostrando el detalle específico de cada uno mediante polimorfismo (`toString()`).
4. **Estado de la Plataforma (Platform Status)**:
    - Uso de un **Enum** `PlatformStatus` (`OPERATIONAL`, `UNDER_MAINTENANCE`, `OUT_OF_SERVICE`).

## Requerimientos Técnicos (Obligatorios)

1. **Precisión Bancaria**: Implementar fórmulas financieras usando los métodos de la clase `BigDecimal` (`add`, `subtract`, `multiply`, `divide`).
2. **Inmutabilidad**: Los datos del cliente y los resultados de la solicitud deben ser inmutables.
3. **Validación de Array**: Validar que no se exceda la capacidad del array de cartera y mostrar un mensaje de error profesional si sucede.
4. **Formateo de Salida**: Usar **Text Blocks** para mostrar el "Plan de Pagos" resumido al cliente.

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**, distribuidos de la siguiente manera:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Arquitectura OOP** | Uso correcto de herencia, clases abstractas y polimorfismo. El sistema de cartera usa agregación con arrays de forma segura. | 25 |
| **Modern Java Features** | Implementación de `Sealed Interfaces`, `Records` y `Pattern Matching` en el flujo de aprobación. Uso de `Text Blocks`. | 25 |
| **Precisión Financiera** | Uso estricto de `BigDecimal` para todos los cálculos. Aplicación correcta de escalas y modos de redondeo. | 20 |
| **Encapsulamiento & Enums** | Uso adecuado de modificadores de acceso y estados de la plataforma mediante Enums con comportamiento. | 15 |
| **Calidad de Entrega** | Repositorio organizado, historial de commits profesional y video demostrativo que explica la lógica técnica. | 15 |

## Niveles de Desempeño

- **Senior (90-100)**: El código sigue principios SOLID. El modelado con Records y Sealed Classes es impecable. Los cálculos financieros son exactos y el video demuestra un entendimiento profundo de la arquitectura.
- **Semi-Senior (75-89)**: Funciona correctamente y aplica las features modernas. Sin embargo, puede haber debilidades menores en el encapsulamiento o la estructura de paquetes.
- **Junior (60-74)**: Cumple los requerimientos funcionales pero tiene "code smells" (clases demasiado grandes, lógica de cálculo dentro del main o falta de validación en los arrays).
- **Insuficiente (<60)**: No compila, usa `double` para transacciones financieras o no aplica los conceptos de herencia y polimorfismo solicitados.

## Entregables

1. **Repositorio de GitHub**:
    - Organización por paquetes: `com.prestaya.model`, `com.prestaya.service`.
    - Historial de commits (mínimo 8) siguiendo buenas prácticas.
2. **Video Demostrativo (Max 5 min)**:
    - Explicar cómo se aplicó la herencia en los tipos de crédito.
    - Mostrar un caso de rechazo por bajo puntaje y un caso de aprobación exitosa usando pattern matching.
    - Mostrar en el código el uso de `BigDecimal` para asegurar la precisión.
