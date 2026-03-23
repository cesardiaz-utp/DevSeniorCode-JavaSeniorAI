# Unidad 1 - Clase 8: Laboratorio de Integración

Objetivo Estratégico: Durante estas 2 horas, el objetivo es transicionar del pensamiento fragmentado (ejercicios sueltos) al Pensamiento Arquitectónico. Desarrollarás una terminal financiera robusta utilizando las funciones de Java 25, aplicando modularidad, validación de datos y control de flujo avanzado sin el ruido de la programación orientada a objetos tradicional.

## 💻 El Escenario: "Fintech Edge Terminal v1.0"

Tu empresa ha sido contratada para desplegar una terminal de operaciones financieras en zonas rurales con nula conectividad a internet (Edge Computing). El software debe ejecutarse localmente, ser extremadamente ligero, rápido y resiliente frente a los errores de tecleo de los usuarios.

Para cumplir con los requerimientos del cliente, la terminal debe implementar la siguiente **funcionalidad core**, la cual construiremos paso a paso durante este laboratorio:

1. **Inicio de Sesión Dinámico**: Al abrir la aplicación, el sistema debe solicitar el "Identificador de Agente" para personalizar los mensajes de la sesión activa.
2. **Motor de Decisiones Ininterrumpido**: Un menú principal que se mantenga en ejecución constante (bucle) hasta que el agente decida salir explícitamente. Debe soportar comandos numéricos y de texto (ej. "1" o "interés").
3. **Simulador de Crédito Seguro**: Una herramienta que pida un monto a financiar y calcule el interés basado en una tasa fija (4.5%). Debe ser capaz de detectar si el usuario ingresa texto en lugar de números y evitar que la aplicación colapse.
4. **Registro Rápido de Ventas**: Un módulo ágil para registrar el nombre y precio de un producto vendido localmente.
5. **Sistema de Descuentos con Escalado de Privilegios**: Una calculadora de descuentos que, como medida de seguridad, solicite una clave de autorización especial (`SUPER25`) si el agente intenta aplicar un descuento superior al 50%.
6. **Identidad Corporativa Visual**: Una opción para renderizar el logo de la empresa directamente en la consola usando patrones geométricos (Arte ASCII).

### Estructura del Taller (120 Minutos)

- **Fase 1 (20m)**: Construcción del núcleo de sesión y saludo dinámico.
- **Fase 2 (30m)**: Implementación del Motor de Decisiones (Menu Loop + Switch Expressions).
- **Fase 3 (40m)**: Módulos de Cálculo y Lógica de Negocio (Validaciones y Try-Catch).
- **Fase 4 (30m)**: El Reto de Seguridad (Módulo de Descuento Protegido y Refactorización).

## 🛠️ Guía de Implementación Paso a Paso

### 1. El Núcleo de la Aplicación

Define las variables globales y el punto de entrada. Nota cómo en Java 25 podemos tener variables de "instancia" fuera del `main` en archivos sin clase declarada.

```java
// Variables de configuración de la Terminal (Campos de la clase implícita)
final double TASA_INTERES_BASE = 0.045; // 4.5%
final String VERSION = "1.0-STABLE";
String nombreUsuario;

void main() {
    configurarSesion();
    ejecutarBuclePrincipal();
}
```

### 2. Implementación de Referencia Completa

A continuación, el código base que debes construir y expandir. Lee los comentarios para entender la jerarquía de llamadas y cómo resuelven los requerimientos del escenario.

```java
/**
 * FASE 1: Configuración de Sesión
 */
void configurarSesion() {
    println("--- SISTEMA INICIADO: " + VERSION + " ---");
    nombreUsuario = readln("Introduzca su Identificador de Agente: ");
    println("Bienvenido, Agente " + nombreUsuario + ". Conexión segura establecida.");
}

/**
 * FASE 2: Motor de Decisiones
 */
void ejecutarBuclePrincipal() {
    boolean sistemaActivo = true;

    while (sistemaActivo) {
        imprimirInterfaz();
        String comando = readln(">> ").toLowerCase().trim();

        sistemaActivo = switch (comando) {
            case "1", "interes"     -> { calcularInteres(); yield true; }
            case "2", "transaccion" -> { registrarVenta(); yield true; }
            case "3", "promo"       -> { aplicarDescuentoEspecial(); yield true; }
            case "4", "logo"        -> { renderizarLogo(); yield true; }
            case "0", "salir", "exit" -> { 
                println("Cerrando sesión para " + nombreUsuario + "..."); 
                yield false; 
            }
            default -> {
                println("⚠️ Comando desconocido. Escriba 'help' para ver opciones.");
                yield true;
            }
        };
    }
}

void imprimirInterfaz() {
    println("\n" + "=".repeat(30));
    println("OPERACIONES DISPONIBLES");
    println("[1] Simular Interés");
    println("[2] Registrar Venta");
    println("[3] Descuento Especial (Nivel Supervisor)");
    println("[4] Ver Identidad Corporativa");
    println("[0] Salir");
    println("=".repeat(30));
}

/**
 * FASE 3: Lógica de Negocio y Robustez
 */
void calcularInteres() {
    println("\n-- Calculadora de Interés Edge --");
    String input = readln("Monto a financiar: ");
    
    try {
        double monto = Double.parseDouble(input);
        if (monto <= 0) throw new IllegalArgumentException("Monto negativo");
        
        double interes = monto * TASA_INTERES_BASE;
        println("Resultado: Interés estimado: $" + interes);
        println("Total a devolver: $" + (monto + interes));
    } catch (Exception e) {
        println("❌ Error: Entrada financiera inválida. Abortando operación.");
    }
}

void registrarVenta() {
    String producto = readln("Nombre del Producto: ");
    String precio = readln("Precio de Venta: ");
    // Registro simplificado en consola
    println("✅ Venta de '" + producto + "' por $" + precio + " registrada.");
}

/**
 * FASE 4: Módulo de Seguridad y Arte ASCII
 */
void aplicarDescuentoEspecial() {
    println("\n-- Módulo de Descuentos Críticos --");
    double precio = Double.parseDouble(readln("Precio Base: "));
    double porcentaje = Double.parseDouble(readln("Porcentaje (%): "));

    if (porcentaje > 50) {
        String pass = readln("⚠️ ALTA PRIORIDAD. Ingrese Clave de Supervisor: ");
        if (!pass.equals("SUPER25")) {
            println("❌ Acceso Denegado. Operación bloqueada.");
            return;
        }
    }
    
    double finalPrice = precio - (precio * (porcentaje / 100));
    println("✅ Descuento Aplicado. Precio Final: $" + finalPrice);
}

void renderizarLogo() {
    // Uso de bucles anidados para generar el logo de la empresa (Triángulo)
    int niveles = 5;
    for (int i = 1; i <= niveles; i++) {
        println(" ".repeat(niveles - i) + "*".repeat(2 * i - 1));
    }
    println("  FINTECH EDGE  ");
}
```

## 💪 Reto de Consolidación (Últimos 30 Minutos)

Para obtener la calificación de "Senior Grade" en este laboratorio, añade las siguientes funcionalidades:

1. **Persistencia de Sesión**: Crea un contador de operaciones. Cada vez que el usuario use una opción (1, 2, 3 o 4), incrementa un contador global `int totalOperaciones`. Muestra este número antes de cerrar la aplicación al elegir la opción 0.
2. **Manejo de Errores Avanzado**: En la función `aplicarDescuentoEspecial`, asegúrate de que si el usuario escribe letras en lugar de números para el precio, el programa no se detenga, sino que capture el error y regrese al menú principal.
3. **Formato de Salida**: Investiga el método `String.format()` o usa concatenación avanzada para que todos los montos de dinero se muestren siempre con dos decimales (ej. `$ 100.50`).

## 💡 VS Code Pro-Tips para este Lab

- **Multi-Cursor**: Si quieres cambiar el nombre de una variable en varios sitios a la vez, selecciónala y presiona `Ctrl + D` repetidamente.
- **Extract Method**: Si una parte de tu `main` se vuelve muy larga, selecciona el código, haz clic derecho y elige `Refactor... > Extract to method`. VS Code creará la estructura en el archivo de clase implícita automáticamente.
- **Debug Console**: No uses solo `println`. Pon un _Breakpoint_ (punto rojo a la izquierda del número de línea) y usa la tecla `F5` para inspeccionar los valores de las variables en tiempo real.
