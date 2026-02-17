# Unidad 2 - Clase 6: Subida de Archivos y Optimización de Imágenes

- **Duración**: 2 horas
- **Objetivo**: Dominar la gestión eficiente de archivos binarios en aplicaciones web modernas. Implementaremos un pipeline de subida de imágenes que optimiza el rendimiento (LCP) mediante compresión en el cliente (Canvas API) y asegura la integridad y seguridad de los datos en el backend (Spring Boot), reduciendo drásticamente el consumo de ancho de banda.

## Parte 1: Teoría - Estrategias de Carga y Optimización (30 Minutos)

### 1 El Protocolo: `multipart/form-data`

A diferencia de los envíos de texto estándar, la transferencia de archivos binarios requiere un tratamiento especial en el protocolo HTTP.

- **El Problema del JSON**: `application/json` no es eficiente para datos binarios grandes (requeriría codificación Base64, aumentando el peso un 33%).
- **La Solución Multipart**: El `Content-Type` define un **boundary** (límite), una cadena única de caracteres que actúa como separador entre las distintas partes del cuerpo de la petición.

**Anatomía de una Petición Multipart**:

```plain
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

juanperez
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="foto.png"
Content-Type: image/png

[...DATOS BINARIOS DE LA IMAGEN...]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

- **En Spring Boot**: El framework detecta este header y activa el `MultipartResolver`, que procesa el stream de entrada, separa las partes basándose en el boundary y te entrega un objeto `MultipartFile` limpio y listo para usar.

### 2 Performance y Core Web Vitals (LCP)

El **LCP (Largest Contentful Paint)** es una métrica crítica de Google que mide el tiempo de renderizado del elemento más grande visible en el viewport.

- **Cuello de Botella**: Los navegadores tienen un límite estricto de conexiones paralelas por dominio (~6). Una imagen de avatar de 5MB sin optimizar bloquea el canal de descarga, retrasando scripts y estilos críticos.
- **Formatos de Nueva Generación**:
  - **JPEG/PNG**: Estándares antiguos. PNG es ineficiente para fotos complejas.
  - **WebP**: El estándar actual. Ofrece compresión con pérdida y transparencia (canal alpha). Reduce el peso un **30-50%** comparado con JPEG/PNG con calidad visual idéntica.
  - **AVIF**: Sucesor de WebP con mayor compresión, pero con un coste de CPU muy alto para la codificación (generación) en tiempo real en dispositivos móviles.

### 3 Arquitectura: ¿Dónde optimizamos?

| Estrategia | Descripción | Ventajas | Desventajas |
| --- | --- | --- | --- |
| **Procesamiento en Servidor** | El cliente sube el archivo "crudo" (RAW/MBs). El backend redimensiona. | Lógica de frontend simple. Control centralizado. | Alto consumo de ancho de banda y CPU del servidor. Mala UX (espera larga). |
| **Procesamiento en Cliente** | JS usa Canvas para redimensionar y comprimir a WebP antes de enviar. | Ahorro masivo de ancho de banda. UX instantánea. Backend ligero. | Requiere lógica JS intermedia. Se pierde el original de alta calidad si no se guarda copia. |
| **Direct Upload (S3)** | El backend firma una URL temporal y el frontend sube directo a la nube. | Serverless para archivos estáticos. Escalabilidad infinita. | Alta complejidad de implementación y gestión de permisos. |

### 4 Seguridad en la Subida (Defensa en Profundidad)

La subida de archivos es uno de los vectores de ataque más peligrosos. Reglas de oro:

- **Renombramiento Obligatorio**: JAMÁS guardar con el nombre original. Usar UUIDs (a0eebc99-9c0b.webp) para evitar colisiones y sobrescrituras malintencionadas.
- **Validación de "Magic Numbers"**: No confiar en la extensión `.jpg`. En producción, se deben leer los primeros bytes del archivo (header hexadecimal) para confirmar que es realmente una imagen (ej. JPEG inicia con `FF D8 FF`).
- **Path Traversal**: Evitar nombres que contengan `../` para prevenir que un atacante escriba archivos en directorios del sistema operativo (`/etc/passwd`).

## Parte 2: Práctica - Implementación Full Stack (1 Hora 30 Minutos)

### Paso 1 Backend: Endpoint Seguro con Spring Boot

Implementaremos un controlador robusto que reciba, valide y simule el almacenamiento del archivo.

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@RestController
@RequestMapping("/api/v1/files")
@CrossOrigin(origins = "*") 
public class FileController {

    @PostMapping("/upload-avatar")
    public ResponseEntity<Map<String, String>> uploadAvatar(
            @RequestParam("file") MultipartFile file
    ) {
        Map<String, String> response = new HashMap<>();
        
        // 1. Validar existencia
        if (file.isEmpty()) {
            response.put("error", "El archivo está vacío");
            return ResponseEntity.badRequest().body(response);
        }

        // 2. Validar tipo MIME (Primera capa de seguridad)
        String contentType = file.getContentType();
        if (contentType == null || !contentType.startsWith("image/")) {
             response.put("error", "Formato no permitido. Solo imágenes.");
             return ResponseEntity.badRequest().body(response);
        }

        try {
            // Log de auditoría básico
            System.out.println("Procesando archivo: " + file.getOriginalFilename());
            System.out.println("Tamaño recibido: " + file.getSize() + " bytes");

            // 3. Sanitización y Renombramiento (UUID)
            // Asumimos extensión .webp si viene de nuestro frontend optimizado
            String extension = "webp"; 
            String fileName = UUID.randomUUID().toString() + "." + extension;

            // 4. ALMACENAMIENTO (Simulación)
            // En producción: Files.copy(inputStream, path) o s3Client.putObject(...)
            // Files.copy(file.getInputStream(), Paths.get("uploads").resolve(fileName));
            
            // Retornamos URL pública simulada
            String url = "[https://cdn.tuapp.com/avatars/](https://cdn.tuapp.com/avatars/)" + fileName;
            
            response.put("url", url);
            response.put("message", "Recurso creado exitosamente");
            
            return ResponseEntity.ok(response);

        } catch (Exception e) {
            response.put("error", "Error interno de E/S: " + e.getMessage());
            return ResponseEntity.internalServerError().body(response);
        }
    }
}
```

**Configuración (`application.properties`)**: Establecemos límites duros para proteger la memoria del servidor ante ataques DoS por subida de archivos gigantes.

```properties
spring.servlet.multipart.max-file-size=2MB
spring.servlet.multipart.max-request-size=5MB
```

### Paso 2 Frontend: El Pipeline de Optimización (Angular)

Implementaremos un servicio o utilidad en Angular que intercepte la selección del archivo y lo transforme antes de que llegue a la red.

**Flujo de Datos**:

1. **Input Change**: El usuario selecciona `foto_vacaciones.jpg` (4MB).
2. **FileReader**: Generamos una URL temporal para mostrar el preview instantáneo (UX).
3. **Canvas Processing**:
    - Dibujamos la imagen en un Canvas HTML5 en memoria.
    - Redimensionamos a máximo 800px de ancho (suficiente para avatares).
    - Ejecutamos `canvas.toBlob(callback, 'image/webp', 0.8)`.
4. **Network Request**: Obtenemos un `Blob` de ~40KB y lo enviamos vía `FormData`.

```typescript
/**
 * UTILIDAD DE IMAGEN (Normalmente esto iría en un archivo separado utils/image-compressor.ts)
 * Se encarga de redimensionar y comprimir la imagen en el cliente.
 */
class ImageCompressor {
  static async compressAndResize(
    file: File, 
    maxWidth: number = 800, 
    quality: number = 0.8
  ): Promise<File> {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      
      reader.onload = (event: any) => {
        const img = new Image();
        img.src = event.target.result;
        
        img.onload = () => {
          // Calcular nuevas dimensiones manteniendo aspecto
          let width = img.width;
          let height = img.height;

          if (width > maxWidth) {
            height = Math.round((height * maxWidth) / width);
            width = maxWidth;
          }

          // Crear canvas para dibujar la imagen redimensionada
          const canvas = document.createElement('canvas');
          canvas.width = width;
          canvas.height = height;
          
          const ctx = canvas.getContext('2d');
          if (!ctx) {
            reject('No se pudo obtener el contexto del canvas');
            return;
          }

          // Dibujar imagen
          ctx.drawImage(img, 0, 0, width, height);

          // Convertir a Blob (WebP para mejor compresión)
          canvas.toBlob(
            (blob) => {
              if (blob) {
                // Reconstruir como File para enviar al backend
                const newFile = new File([blob], file.name.replace(/\.[^/.]+$/, ".webp"), {
                  type: 'image/webp',
                  lastModified: Date.now(),
                });
                resolve(newFile);
              } else {
                reject('Error al comprimir la imagen');
              }
            },
            'image/webp',
            quality
          );
        };
        
        img.onerror = (error) => reject(error);
      };
      
      reader.onerror = (error) => reject(error);
    });
  }
}
```

```typescript
/**
 * SERVICIO DE SUBIDA (Backend Connection)
 * Se encarga de la lógica HTTP.
 */
@Injectable({
  providedIn: 'root'
})
export class AvatarUploadService {
  private http = inject(HttpClient);

  uploadAvatar(file: File): Observable<any> {
    // --- LÓGICA REAL DE SUBIDA (Comentada) ---
    /*
    const formData = new FormData();
    formData.append('file', file);
    // Retornamos el observable directamente
    return this.http.post('http://localhost:8080/api/v1/files/upload-avatar', formData);
    */

    // --- SIMULACIÓN DE SUBIDA ---
    console.log('Servicio: Enviando FormData al backend...');
    console.log('Archivo procesado:', file.name, file.type, file.size);
    
    // Simulamos delay de red y respuesta exitosa
    return of({ success: true, url: 'https://fake-url.com/avatar.webp' }).pipe(delay(2000));
  }
}
```

```typescript
/**
 * COMPONENTE PRINCIPAL
 */
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule], 
  providers: [provideHttpClient()],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="min-h-screen bg-gray-50 flex flex-col items-center justify-center p-4 font-sans text-slate-800">
      
      <div class="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full border border-gray-100">
        <h1 class="text-2xl font-bold mb-2 text-center text-indigo-700">Subir Avatar</h1>
        <p class="text-sm text-gray-500 mb-6 text-center">Optimización automática a WebP</p>

        <!-- Área de Previsualización -->
        <div class="flex justify-center mb-6 relative group">
          <div class="relative w-40 h-40 rounded-full overflow-hidden border-4 border-indigo-100 shadow-inner bg-gray-100">
            @if (previewUrl()) {
              <img [src]="previewUrl()" class="w-full h-full object-cover" alt="Avatar preview">
            } @else {
              <div class="flex items-center justify-center w-full h-full text-gray-300">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-20 w-20" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                </svg>
              </div>
            }
            
            <!-- Overlay de carga -->
            @if (isCompressing()) {
              <div class="absolute inset-0 bg-black/50 flex items-center justify-center backdrop-blur-sm">
                <span class="text-white text-xs font-semibold animate-pulse">Comprimiendo...</span>
              </div>
            }
          </div>

          <!-- Botón flotante para editar -->
          <button (click)="fileInput.click()" 
            class="absolute bottom-0 right-1/3 translate-x-8 bg-indigo-600 hover:bg-indigo-700 text-white p-2 rounded-full shadow-lg transition-all active:scale-95"
            [disabled]="isUploading() || isCompressing()">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z" />
            </svg>
          </button>
        </div>

        <!-- Input Oculto -->
        <input #fileInput type="file" (change)="onFileSelected($event)" accept="image/*" class="hidden">

        <!-- Estadísticas de Compresión -->
        @if (originalSize() && compressedSize()) {
          <div class="bg-indigo-50 rounded-lg p-4 mb-6 border border-indigo-100">
            <h3 class="text-xs font-bold text-indigo-400 uppercase tracking-wider mb-2">Resultados de Optimización</h3>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <p class="text-xs text-gray-500">Original ({{originalFormat()}})</p>
                <p class="font-mono text-sm font-bold text-gray-700">{{ originalSize() }}</p>
              </div>
              <div class="text-right">
                <p class="text-xs text-gray-500">Optimizado (WebP)</p>
                <p class="font-mono text-sm font-bold text-green-600">{{ compressedSize() }}</p>
              </div>
            </div>
            <div class="mt-2 text-center">
              <span class="inline-block bg-green-200 text-green-800 text-xs px-2 py-1 rounded-full font-bold">
                ¡Ahorro del {{ savingsPercentage() }}%!
              </span>
            </div>
          </div>
        }

        <!-- Acciones -->
        <div class="flex flex-col gap-3">
          <button 
            (click)="upload()" 
            [disabled]="!compressedFile() || isUploading()"
            class="w-full py-3 px-4 bg-indigo-600 hover:bg-indigo-700 disabled:bg-gray-300 disabled:cursor-not-allowed text-white rounded-lg font-medium transition-colors shadow-sm flex items-center justify-center gap-2">
            @if (isUploading()) {
              <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              <span>Subiendo...</span>
            } @else {
              <span>Guardar Cambios</span>
            }
          </button>

          @if (uploadSuccess()) {
             <div class="p-3 bg-green-100 text-green-700 text-sm rounded-lg text-center animate-pulse">
               ✅ ¡Imagen subida al servidor con éxito!
             </div>
          }
        </div>

      </div>
      
      <p class="mt-8 text-xs text-gray-400 max-w-xs text-center">
        Nota: Esta demo redimensiona a max 800px y convierte a WebP en el navegador antes de subir.
      </p>
    </div>
  `,
  styles: []
})
export class App {
  previewUrl = signal<string | null>(null);
  compressedFile = signal<File | null>(null);
  
  // Estados de UI
  isCompressing = signal(false);
  isUploading = signal(false);
  uploadSuccess = signal(false);

  // Estadísticas
  originalSize = signal<string | null>(null);
  originalFormat = signal<string | null>(null);
  compressedSize = signal<string | null>(null);
  savingsPercentage = signal<number>(0);

  // Inyección del servicio
  private uploadService = inject(AvatarUploadService);

  async onFileSelected(event: Event) {
    const input = event.target as HTMLInputElement;
    if (!input.files?.length) return;

    const file = input.files[0];
    this.uploadSuccess.set(false);
    
    // 1. Mostrar preview inmediato (UX)
    const objectUrl = URL.createObjectURL(file);
    this.previewUrl.set(objectUrl);

    // Guardar datos originales para comparar
    this.originalSize.set(this.formatBytes(file.size));
    this.originalFormat.set(file.type.split('/')[1]?.toUpperCase() || 'FILE');

    // 2. Comprimir en cliente (Optimización)
    this.isCompressing.set(true);
    
    try {
      // Simulamos un pequeño delay artificial para que se vea el loader si la imagen es muy pequeña
      // En la vida real, ImageCompressor tarda un poco con imágenes grandes.
      const compressed = await ImageCompressor.compressAndResize(file, 800, 0.85);
      
      this.compressedFile.set(compressed);
      this.compressedSize.set(this.formatBytes(compressed.size));
      
      // Calcular ahorro
      const saving = ((file.size - compressed.size) / file.size) * 100;
      this.savingsPercentage.set(Math.round(saving));
      
    } catch (error) {
      console.error('Error al comprimir', error);
      alert('Error al procesar la imagen');
    } finally {
      this.isCompressing.set(false);
    }
  }

  upload() {
    const fileToUpload = this.compressedFile();
    if (!fileToUpload) return;

    this.isUploading.set(true);

    // Llamada al servicio
    this.uploadService.uploadAvatar(fileToUpload).subscribe({
      next: (response) => {
        console.log('Respuesta del servicio:', response);
        this.isUploading.set(false);
        this.uploadSuccess.set(true);
      },
      error: (error) => {
        console.error('Error en subida:', error);
        this.isUploading.set(false);
      }
    });
  }

  private formatBytes(bytes: number, decimals = 2) {
    if (!+bytes) return '0 Bytes';
    const k = 1024;
    const dm = decimals < 0 ? 0 : decimals;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return `${parseFloat((bytes / Math.pow(k, i)).toFixed(dm))} ${sizes[i]}`;
  }
}
```

## Desafío (Homework)

1. Integrar el `FileController` en su proyecto backend.
2. Configurar una petición en **Postman** seleccionando `Body` -> `form-data`, clave file (tipo `File`) y adjuntando una imagen real.
3. Verificar en la consola de Spring Boot que el archivo llega y se genera un UUID único.
