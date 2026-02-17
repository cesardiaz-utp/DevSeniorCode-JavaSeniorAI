# Unidad 1: SIPA - Sistema de Inteligencia de Precios Agrarios

## Contexto del Proyecto

El **SIPA** es una aplicación de consola diseñada para ayudar a los agricultores colombianos a calcular la rentabilidad de sus cosechas. Los estudiantes deben construir una herramienta que reciba datos de producción y devuelva un análisis financiero detallado.

## Especificaciones de Menú y Funcionalidad

El programa debe ejecutarse en un bucle continuo (`while`) que solo termine cuando el usuario elija la opción "Salir".

### Opción 1: Registrar Cosecha (Cálculo de Proyección)

Al ingresar a esta opción, se debe solicitar la información en el siguiente orden y aplicar las reglas de negocio descritas:

#### A. Selección de Producto

Presentar un submenú y asignar el **Precio Base por Kilo** usando un `switch` expression:

1. **Papa Capira**: $2,500 COP/kg
2. **Café Pergamino**: $15,000 COP/kg
3. **Aguacate Hass**: $8,000 COP/kg

#### B. Datos de Volumen

Solicitar cantidad en **Toneladas** (Convertir internamente a kilos: `kilos = toneladas * 1000`).

#### C. Ubicación y Flete (Costo de Transporte)

Solicitar el origen para determinar el **Costo Base de Flete por Tonelada**:

1. **Boyacá**: $120,000 por tonelada.
2. **Huila**: $180,000 por tonelada.
3. **Antioquia**: $150,000 por tonelada.

#### D. Destino y Recargos

1. **Mercado Local / Corabastos**: Sin recargos adicionales.
2. **Exportación (Puerto)**: Sumar un **15% de recargo logístico** sobre el costo total del flete calculado en el paso anterior.

#### E. Cálculo de Impuestos (DIAN)

Sobre el **Subtotal Bruto** (kilos * precio base):

- **IVA**:
  - **0%** para Papa y Aguacate (Categoría: Alimentos básicos)
  - **5%** para Café (Categoría: Procesados básicos).
- **Retención en la Fuente**: Si el Subtotal Bruto supera los **$4,000,000 COP**, aplicar una retención del **2.5%**. Si es menor, aplicar **1.5%**.

### Opción 2: Glosario de Términos (Análisis de Viabilidad)

Debe mostrar un submenú con tres conceptos. Al elegir uno, mostrar la definición usando **Text Blocks** (`"""`):

- **Punto de Equilibrio**: "Es el nivel de ventas donde los ingresos igualan a los costos totales; es decir, donde no hay pérdida ni ganancia."
- **Retención en la Fuente**: "Es un mecanismo de recaudo anticipado de impuestos que el comprador descuenta del pago total al agricultor."
- **Arancel logístico**: "Costo adicional aplicado a mercancías destinadas a la exportación para cubrir trámites portuarios."

### Opción 3: Simulador de Punto de Equilibrio

Solicitar al usuario:

1. **Costos Fijos Totales** (Ej: Arriendo, jornales, insumos).
2. **Precio de Venta por unidad**.
3. **Costo Variable por unidad**.

**Cálculo**: `Punto de Equilibrio = Costos Fijos / (Precio Venta - Costo Variable)`. Mostrar el resultado indicando cuántas unidades mínimas debe vender para no tener pérdidas.

### Opción 4: Configuración

Permitir al usuario ingresar su **Nombre** y **Nombre de la Finca**. Estos datos deben persistir en variables durante la ejecución para ser usados en el reporte final.

### Opción 5: Generar Reporte y Salir

Antes de cerrar el programa, debe imprimir un resumen formateado con **Text Blocks**:

- Nombre del Agricultor y Finca.
- Resumen del último cálculo: Subtotal, Impuestos, Flete y **Utilidad Neta** (Subtotal - Impuestos - Flete).
- Mensaje de despedida: "Gracias por usar SIPA. ¡Buen provecho con su cosecha!".

## Requerimientos Técnicos (Obligatorios)

1. **Uso de Java 25**:
    - Implementar el punto de entrada mediante `void main() { ... }` (sin `static` ni `String[] args`).
    - Uso de `var` para la declaración de variables locales.
2. **Validación**:
    - Todo ingreso numérico debe ser validado. Si el usuario ingresa un número negativo, usar un `do-while` para repetir la pregunta.
3. **Modularización (Métodos)**:
    - El código NO debe estar todo en el `main`. Crear métodos para cálculos y visualización.
4. **Manejo de Scanner**:
    - Limpiar el buffer con `scanner.nextLine()` adecuadamente.

## Rúbrica de Evaluación

El proyecto se evaluará sobre un total de **100 puntos**, distribuidos de la siguiente manera:

| Categoría | Criterio Detallado | Puntos |
| --- | --- | --- |
| **Java Moderno** | Uso correcto de `void main()` de instancia, inferencia de tipos con `var` y **Text Blocks** para reportes y glosario. | 20 |
| **Lógica de Control** | Implementación de `switch` como expresión, uso de bucles para el menú y validaciones `do-while` para entradas correctas. | 25 |
| **Modularidad** | El código está organizado en métodos con responsabilidades claras (cálculo, entrada, salida). No hay lógica repetida. | 20 |
| **Control de Versiones** | Repositorio en GitHub con historial de al menos 5 commits siguiendo convenciones y uso correcto de `.gitignore`. | 15 |
| **Calidad de Entrega** | Video demostrativo fluido, explicación técnica clara de los métodos y el código compila sin errores. | 20 |

## Niveles de Desempeño

- **Senior (90-100)**: Implementa todas las funcionalidades con validaciones exhaustivas. El código es excepcionalmente limpio, modular y utiliza todas las características de Java 25 solicitadas. El video demuestra un dominio total del problema.
- **Semi-Senior (75-89)**: El programa funciona correctamente y usa Java moderno. Los métodos están definidos, aunque la lógica podría simplificarse o faltan algunos detalles de formato en el reporte final.
- **Junior (60-74)**: El programa cumple con las funcionalidades básicas pero presenta debilidades en la modularización (métodos muy largos) o no valida correctamente todas las entradas negativas.
- **Insuficiente (<60)**: El código no compila, no hace uso de métodos, o no implementa las estructuras de control solicitadas (ej. usa `if` anidados en lugar de `switch` expressions).

## Entregables

1. **Repositorio de GitHub**:
    - Debe ser público o compartido con el instructor.
    - Mínimo **5 commits** con mensajes descriptivos (Ej: "feat: add flete calculation logic").
2. **Video Demostrativo**:
    - Duración máxima: **5 minutos**.
    - Debe mostrar:
      1. Explicación breve de la estructura del código y métodos creados.
      2. Ejecución del programa probando todas las opciones del menú (incluyendo validaciones de números negativos).
      3. El video debe subirse a YouTube.
3. **Código Fuente**:
    - El archivo `.java` principal debe estar correctamente nombrado y libre de errores de compilación.
