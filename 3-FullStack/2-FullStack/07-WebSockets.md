# Unidad 2 - Clase 7: Comunicación en Tiempo Real con WebSockets

- **Duración**: 2 horas
- **Objetivo**: Implementar un canal de comunicación bidireccional y persistente entre el servidor y el cliente. Configuraremos un broker de mensajería en Spring Boot y consumiremos eventos en tiempo real en Angular para notificar a los usuarios sobre nuevas citas sin necesidad de recargar la página.

## Parte 1: Teoría - Profundización en Real-Time (45 Minutos)

### 1. Limitaciones de HTTP y la Solución WebSocket

El modelo clásico de la web (HTTP/1.1) es "**Request-Response**": el cliente siempre tiene la iniciativa. El servidor es pasivo; no puede hablarle al cliente si este no le pregunta primero.

- **Polling (Sondeo Clásico)**: El cliente pregunta cada 5 segundos `GET /notificaciones`.
  - _Problema_: Latencia alta y desperdicio de recursos (99% de las veces la respuesta es "nada nuevo").
- **Long Polling**: El cliente pregunta, y el servidor deja la conexión abierta hasta que tenga algo que decir.
  - _Problema_: Complejo de mantener y consume un hilo del servidor por cada cliente "esperando".
- **WebSocket**: Es un protocolo de **Capa 7 (Aplicación)**, igual que HTTP, pero diseñado para ser bidireccional.
  - **El Handshake**: Todo comienza con una petición HTTP normal que incluye el header `Upgrade: websocket`. Si el servidor acepta, responde con un código `101 Switching Protocols`.
  - **El Túnel**: A partir de ese momento, la conexión TCP se mantiene abierta y se convierte en un tubo bidireccional donde cualquiera puede enviar datos en cualquier momento.

### 2. STOMP: Poniendo orden en el caos

WebSocket por sí solo es un flujo "crudo" de bytes o texto. No tiene concepto de "rutas", "autenticación" o "errores". Si envías el string "Hola", el servidor no sabe si es un chat, un comando o un error.

**STOMP (Simple Text Oriented Messaging Protocol)** es a WebSocket lo que HTTP es a TCP. Estructura la comunicación en **Tramas (Frames)**:

```plain
COMMAND
header1:value1
header2:value2

Body del mensaje^@
```

**Ejemplo de trama de envío**:

```plain
SEND
destination:/app/nueva-cita
content-type:application/json

{"paciente": "Juan Perez", "hora": "10:00"}
```

### 3. Arquitectura del Message Broker

En Spring Boot, usamos el patrón **Pub/Sub (Publicación/Suscripción)** gestionado por un Broker.

1. `Simple Broker (In-Memory)`: Es el que usaremos hoy. Vive en la memoria RAM de la aplicación Spring Boot.
    - _Ventaja_: Configuración cero. Muy rápido.
    - _Desventaja_: **No escala horizontalmente**. Si tienes 2 instancias de tu backend tras un balanceador de carga, el Broker A no conoce a los usuarios conectados al Broker B.
2. **Full External Broker (RabbitMQ / ActiveMQ)**: Para producción a gran escala. Spring le pasa los mensajes a RabbitMQ, y este se encarga de distribuirlos a todas las instancias.

### 4. Fallback con SockJS

A veces, los firewalls corporativos o proxies antiguos bloquean el protocolo WebSocket. **SockJS** es una librería que simula la experiencia WebSocket.

1. Intenta conectar por WebSocket nativo.
2. Si falla, hace downgrade automático a _Streaming HTTP_.
3. Si falla, hace downgrade a Long Polling. Esto garantiza que tu chat funcione incluso en redes restrictivas.

### 5. Canales de Destino

- `/topic/nombre` **(Broadcast 1-a-N)**: Como una radio. Si envías un mensaje aquí, todos los clientes suscritos lo reciben. (Ej: Ticker de bolsa, Goles en vivo).
- `/queue/nombre` **(Point-to-Point 1-a-1)**: Mensajes dirigidos a un usuario específico. Spring Security y STOMP trabajan juntos para enviar mensajes a `/user/juan/queue/errors` que solo Juan puede leer.

## Parte 2: Laboratorio práctico (1h 15m)

### Paso 1: Backend (Spring Boot)

#### 1. Dependencias (pom.xml)

Necesitamos el starter de WebSocket.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

#### 2. Configuración del WebSocket (`WebSocketConfig.java`)

Habilitamos el broker y definimos los puntos de entrada. Observa el uso de `withSockJS()`.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // Habilita un broker simple en memoria
        // Prefijo para enviar mensajes DESDE el servidor AL cliente: /topic
        config.enableSimpleBroker("/topic");
        
        // Prefijo para mensajes DESDE el cliente AL servidor (si fuera necesario)
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // Punto de conexión inicial (Handshake)
        // setAllowedOriginPatterns("*") es vital para evitar errores de CORS en desarrollo
        registry.addEndpoint("/ws-citas")
                .setAllowedOriginPatterns("*")
                .withSockJS(); // Habilita fallback para navegadores antiguos/proxies restrictivos
    }
}
```

#### 3. Controlador de Mensajería (`NotificationController.java`)

Este controlador recibe la orden de notificar y la distribuye.

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;

// DTO simple para el ejemplo
class CitaNotification {
    public String mensaje;
    public String paciente;
    // getters & setters
}

@Controller
public class NotificationController {

    // 1. Recibe mensajes en /app/nueva-cita (desde el cliente o servicio interno)
    // 2. Reenvía automáticamente lo que retorne a /topic/appointments
    @MessageMapping("/nueva-cita")
    @SendTo("/topic/appointments")
    public CitaNotification notificarCita(CitaNotification notificacion) {
        // Aquí podrías guardar logs o procesar datos
        System.out.println("Notificando nueva cita de: " + notificacion.paciente);
        return notificacion;
    }
    
    // NOTA AVANZADA: Para enviar desde un @Service normal (ej. cuando se guarda en DB),
    // debes inyectar 'SimpMessagingTemplate' y usar:
    // template.convertAndSend("/topic/appointments", objeto);
}
```

### Paso 2: Frontend (Angular)

Para conectar Angular con Spring Boot vía STOMP, el estándar moderno es la librería @stomp/rx-stomp.

#### 1. Instalación

```bash
npm install @stomp/rx-stomp
```

#### 2. Implementación

Crearemos un servicio que:

1. Configure la conexión al endpoint `ws://localhost:8080/ws-citas`.
2. Exponga un `Observable` al que los componentes se puedan suscribir.
3. Muestre un "Toast" visual cuando llegue un mensaje.

```typescript
/**
 * INTERFACES
 */
interface CitaNotification {
  mensaje: string;
  paciente: string;
  fecha: Date;
}
```

```typescript
/**
 * SERVICIO WEBSOCKET (Simulado para Demo / Estructura Real Comentada)
 */
@Injectable({
  providedIn: 'root',
})
export class WebSocketService {
  // private rxStomp: RxStomp; // <--- Descomentar con librería real

  // Simulamos el stream de datos
  private notificationSubject = new BehaviorSubject<CitaNotification | null>(null);
  
  constructor() {
    // --- CÓDIGO REAL CON @stomp/rx-stomp ---
    /*
    this.rxStomp = new RxStomp();
    this.rxStomp.configure({
      brokerURL: 'ws://localhost:8080/ws-citas/websocket', // Endpoint definido en Spring
      connectHeaders: {
        login: 'guest',
        passcode: 'guest',
      },
      heartbeatIncoming: 0,
      heartbeatOutgoing: 20000,
      reconnectDelay: 200,
      debug: (msg: string) => {
        console.log(new Date(), msg);
      },
    });
    this.rxStomp.activate();
    */

    // --- SIMULACIÓN PARA DEMO VISUAL ---
    // (Simula que llega una notificación del backend cada 5 segundos para que no esperes tanto)
    interval(5000).subscribe((i) => {
      const pacientes = ['Ana García', 'Carlos López', 'Maria Rodriguez', 'Juan Perez'];
      const paciente = pacientes[Math.floor(Math.random() * pacientes.length)];
      
      this.simulateIncomingMessage({
        mensaje: `Nueva cita agendada #${100 + i}`,
        paciente: paciente,
        fecha: new Date()
      });
    });
  }

  // Método para suscribirse al tópico
  watchAppointments(): Observable<CitaNotification> {
    // --- CÓDIGO REAL ---
    /*
    return this.rxStomp.watch('/topic/appointments').pipe(
      map((message) => JSON.parse(message.body) as CitaNotification)
    );
    */

    // --- CÓDIGO SIMULADO ---
    return this.notificationSubject.asObservable().pipe(
      // Filtramos nulos iniciales y aseguramos el tipo
      map(val => val as CitaNotification) 
    );
  }

  // Método auxiliar para la simulación
  private simulateIncomingMessage(notification: CitaNotification) {
    console.log('WS: Mensaje recibido', notification);
    this.notificationSubject.next(notification);
  }
}
```

```typescript
/**
 * COMPONENTE TOAST (Notificación Visual)
 */
@Component({
  selector: 'app-toast',
  standalone: true,
  imports: [CommonModule],
  template: `
    @if (visible()) {
      <div class="fixed top-5 right-5 z-50 animate-slide-in">
        <div class="bg-white border-l-4 border-indigo-500 shadow-xl rounded-lg p-4 max-w-sm flex items-start gap-3">
          <!-- Icono -->
          <div class="bg-indigo-100 p-2 rounded-full text-indigo-600">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
            </svg>
          </div>
          
          <!-- Contenido -->
          <div class="flex-1">
            <h4 class="font-bold text-gray-800 text-sm">Notificación de Sistema</h4>
            <p class="text-sm text-gray-600 mt-1">{{ data()?.mensaje }}</p>
            <p class="text-xs text-indigo-500 mt-1 font-medium">Paciente: {{ data()?.paciente }}</p>
          </div>

          <!-- Cerrar -->
          <button (click)="close()" class="text-gray-400 hover:text-gray-600">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
      </div>
    }
  `,
  styles: [`
    .animate-slide-in {
      animation: slideIn 0.5s ease-out forwards;
    }
    @keyframes slideIn {
      from { transform: translateX(100%); opacity: 0; }
      to { transform: translateX(0); opacity: 1; }
    }
  `]
})
export class ToastComponent {
  visible = signal(false);
  data = signal<CitaNotification | null>(null);
  
  show(notification: CitaNotification) {
    this.data.set(notification);
    this.visible.set(true);
    
    // Auto-ocultar después de 5 segundos
    setTimeout(() => this.close(), 5000);
  }

  close() {
    this.visible.set(false);
  }
}
```

```typescript
/**
 * COMPONENTE PRINCIPAL (App)
 */
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, ToastComponent],
  template: `
    <div class="min-h-screen bg-slate-50 flex flex-col items-center justify-center p-6 font-sans">
      
      <!-- Cabecera -->
      <div class="text-center mb-10">
        <h1 class="text-3xl font-bold text-slate-800 mb-2">Clase 7: WebSockets en Acción</h1>
        <p class="text-slate-500">Sistema de Citas Médicas - Panel de Recepción</p>
      </div>

      <!-- Panel Principal -->
      <div class="bg-white rounded-2xl shadow-lg w-full max-w-4xl p-8 border border-slate-100">
        
        <div class="flex justify-between items-center mb-6 pb-6 border-b border-slate-100">
          <div>
            <h2 class="text-xl font-bold text-slate-700">Citas del Día</h2>
            <p class="text-sm text-green-600 flex items-center gap-2 mt-1">
              <span class="relative flex h-3 w-3">
                <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-green-400 opacity-75"></span>
                <span class="relative inline-flex rounded-full h-3 w-3 bg-green-500"></span>
              </span>
              Conectado al servidor en tiempo real (/topic/appointments)
            </p>
          </div>
          
          <button class="bg-slate-100 hover:bg-slate-200 text-slate-600 px-4 py-2 rounded-lg text-sm font-medium transition-colors">
            Filtrar Vista
          </button>
        </div>

        <!-- Lista de Citas (Se llena dinámicamente) -->
        <div class="space-y-3">
          @if (citas().length === 0) {
            <div class="text-center py-10 text-slate-400 italic">
              Esperando nuevas citas...
            </div>
          }

          @for (cita of citas(); track cita) {
            <div class="flex items-center p-4 bg-slate-50 rounded-xl border border-slate-100 animate-fade-in hover:shadow-md transition-shadow">
              <div class="h-10 w-10 rounded-full bg-indigo-100 flex items-center justify-center text-indigo-600 font-bold mr-4">
                {{ cita.paciente.charAt(0) }}
              </div>
              <div class="flex-1">
                <h3 class="font-bold text-slate-800">{{ cita.paciente }}</h3>
                <p class="text-xs text-slate-500">{{ cita.mensaje }}</p>
              </div>
              <div class="text-right">
                <span class="block text-xs font-mono text-slate-400">
                  {{ cita.fecha | date:'HH:mm:ss' }}
                </span>
                <span class="inline-block mt-1 px-2 py-0.5 bg-blue-100 text-blue-700 text-[10px] font-bold rounded uppercase">
                  Agendada
                </span>
              </div>
            </div>
          }
        </div>

      </div>

      <!-- Componente Toast para notificaciones emergentes -->
      <app-toast #toast></app-toast>

      <p class="mt-8 text-xs text-gray-400 max-w-md text-center">
        * En esta demo, las notificaciones son simuladas. En producción, conectarías el 
        <code>WebSocketService</code> a tu backend Spring Boot real.
      </p>
    </div>
  `,
  styles: [`
    .animate-fade-in {
      animation: fadeIn 0.5s ease-out;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
  `]
})
export class App implements OnInit, OnDestroy {
  citas = signal<CitaNotification[]>([]);
  private wsSubscription?: Subscription;

  // Obtenemos referencia al componente Toast del template
  @ViewChild('toast') toast!: ToastComponent; 
  
  constructor(private wsService: WebSocketService) {}

  ngOnInit() {
    // Suscribirse a los eventos del WebSocket
    this.wsSubscription = this.wsService.watchAppointments().subscribe((notificacion) => {
      if (notificacion) {
        // 1. Agregar a la lista (Datos)
        this.citas.update(lista => [notificacion, ...lista]);
        
        // 2. Mostrar Toast (UI)
        // Verificamos que el componente hijo esté inicializado
        if (this.toast) {
          this.toast.show(notificacion);
        }
      }
    });
  }

  ngOnDestroy() {
    this.wsSubscription?.unsubscribe();
  }
}
```

## Desafío de la Clase (Homework)

1. Levantar el backend con la configuración WebSocket.
2. Implementar el frontend (puedes usar el código provisto).
3. Abrir la aplicación en dos pestañas diferentes del navegador.
4. Generar una cita en una pestaña y ver cómo aparece la notificación instantáneamente en la otra.
