# Unidad 1: MediConnect Cloud & DevOps Transformation

## Escenario de Negocio (Contexto Narrativo)

Tras blindar la calidad de MediConnect, el ecosistema se enfrenta a su último gran salto evolutivo: la **escalabilidad**, la **resiliencia** y la **disponibilidad global**. Una aplicación de salud moderna no puede estar confinada a la máquina de un desarrollador o a un entorno local inestable; debe ser capaz de desplegarse de forma idéntica y predecible en cualquier servidor del mundo, recuperarse automáticamente ante fallos críticos y actualizarse de manera transparente sin interrumpir el servicio vital que presta a médicos y pacientes.

En este proyecto final, asumirás el rol de **DevOps Engineer / SRE (Site Reliability Engineer)**. Tu misión es derribar definitivamente el histórico "muro de la confusión" entre el desarrollo y las operaciones. Transformarás los repositorios de código en artefactos inmutables y seguros mediante la **contenerización profesional con Docker**, orquestarás toda la infraestructura local mediante **Docker Compose** y automatizarás el flujo de entrega de software mediante **Pipelines de CI/CD** en GitHub Actions. Además, asegurarás que el código mantenga un estándar de "Deuda Técnica Cero" mediante el análisis estático en SonarQube, culminando con un despliegue en la nube que otorgue a MediConnect una presencia pública, profesional y preparada para el tráfico real.

## Requerimientos Funcionales y de Automatización

### A. Contenerización Profesional e Inmutabilidad

- **Inmutabilidad del Backend con Spring Boot 4**: Escribir un `Dockerfile` de grado producción para la MediCare API.
  - **Implicación Técnica**: Se debe implementar obligatoriamente el patrón **Multi-Stage Build**. Esto implica una fase inicial de compilación (usando JDK 25 y Maven/Gradle) y una fase final de ejecución utilizando un **JRE Distroless** o una imagen Alpine mínima.
  - **Consecuencia**: Esta técnica no solo reduce el tamaño de la imagen de ~800MB a menos de 200MB, sino que también elimina herramientas innecesarias (como shells o gestores de paquetes), reduciendo drásticamente la superficie de ataque y las vulnerabilidades potenciales.
- **Optimización de Frontend con Nginx**: Desarrollar un `Dockerfile` para la aplicación Angular 21 que utilice **Nginx** como servidor de contenido estático de alto rendimiento.
  - **Detalle**: Se debe configurar el archivo `nginx.conf` para manejar correctamente el enrutamiento de la SPA (Single Page Application), asegurando que todas las rutas se redirijan al `index.html` para que el router de Angular tome el control.

### B. Orquestación de Ecosistema y Resiliencia (Docker Compose)

- **Infraestructura como Código (IaC) Local**: Crear un archivo `docker-compose.yml` maestro que defina y relacione el ecosistema completo:
  - **Definición de Servicios**: `db` (Postgres 16+), `api` (Spring Boot App) y `web` (Angular App).
  - **Mecanismos de Resiliencia (Healthchecks)**: Configurar controles de salud que verifiquen el estado real de la base de datos antes de iniciar el backend. Esto evita errores de conexión prematuros y garantiza que el sistema sea capaz de auto-reiniciarse si un servicio falla.
  - **Aislamiento de Redes y Volúmenes**: Definir redes internas privadas para que la base de datos no sea accesible desde el exterior del host, y volúmenes persistentes para garantizar que los datos médicos no se pierdan al reiniciar los contenedores.

### C. Pipeline de Integración Continua (CI con GitHub Actions)

- **Automatización del Ciclo de Vida**: Configurar workflows en YAML que se disparen de forma reactiva ante eventos de `push` o `pull request` en las ramas protegidas.
  - **Estrategia de Quality Gate**: El pipeline debe ejecutar una batería de pasos que incluyan: Checkout, Setup de entornos (Java 25 / Node 22), ejecución de tests unitarios y de integración.
  - **Bloqueo de Despliegue**: El flujo debe configurarse para fallar si los tests no alcanzan el 80% de éxito o si el build de Maven/Angular genera advertencias críticas, impidiendo que código defectuoso llegue al registro de artefactos.

### D. Análisis Estático de Seguridad y Calidad (SonarQube)

- **Auditoría de Deuda Técnica**: Integrar el escaneo de **SonarQube/SonarCloud** para realizar un análisis SAST (Static Application Security Testing).
  - **Objetivo**: Identificar activamente "Code Smells", vulnerabilidades de seguridad (como inyecciones potenciales o hardcoded secrets) y duplicidad de código. El estudiante deberá refactorizar al menos tres hallazgos críticos detectados por la herramienta y documentar este proceso de mejora continua.

### E. Entrega Continua y Despliegue en la Nube (CD)

- **Gestión de Artefactos y Versionado Semántico**: Extender el pipeline para que, tras un build exitoso, la imagen Docker sea construida, etiquetada con **SemVer (v1.0.x)** y publicada automáticamente en **Docker Hub** o **GitHub Container Registry (GHCR)**.
- **Despliegue en Entorno de Producción (Cloud)**: Utilizar un servicio PaaS (como Railway, Render o AWS Free Tier) para desplegar los contenedores. Se debe demostrar que la comunicación entre el frontend y el backend funciona a través de URLs públicas seguras (HTTPS).

## Especificaciones Técnicas

- **Runtime de Contenedores**: Docker Engine con optimizaciones para la JVM (Container Awareness).
- **Infraestructura Cloud**: Plataformas PaaS para despliegue rápido y escalable.
- **Automatización**: GitHub Actions utilizando Secretos de GitHub para manejar credenciales de Docker y la Nube.
- **Calidad**: SonarCloud integrado con el repositorio para feedback en tiempo real sobre los Pull Requests.
- **Seguridad**: Implementación de políticas de red básicas y escaneo de vulnerabilidades en imágenes Docker.

## Estructura de Repositorios

La separación en dos repositorios permite pipelines independientes y despliegues granulares:

### Repositorio A: `mediconnect-backend`

```plain
/mediconnect-backend
├── .github/workflows/ci-backend.yml  # Pipeline completo de build, test y push
├── Dockerfile                        # Multi-stage: Build (Maven) -> Run (JRE)
├── docker-compose.yml                # Configuración local con Postgres
├── scripts/healthcheck.sh            # Script para validar disponibilidad de DB
├── .cursorrules                      # Regla: "Optimizar capas de Docker para caché"
└── README.md                         # Documentación de la API y tags de imagen
```

### Repositorio B: `mediconnect-frontend`

```plain
/mediconnect-frontend
├── .github/workflows/ci-frontend.yml # Pipeline de lint, build y despliegue
├── Dockerfile                        # Nginx con optimizaciones de caché
├── nginx.conf                        # Configuración de seguridad para producción
├── .cursorrules                      # Regla: "Garantizar build de producción minificado"
└── README.md                         # Enlace a la URL de producción activa
```

## Rúbrica de Evaluación Expandida

El proyecto se evaluará sobre un total de **100 puntos**:

| Categoría | Criterio de Excelencia (Nivel DevOps) | Puntos |
| --- | --- | --- |
| **Estrategia Docker** | Uso profesional de Multi-stage, imágenes Distroless/Alpine y Nginx optimizado. | 25 |
| **Orquestación & Resiliencia** | Docker Compose funcional con healthchecks, redes aisladas y volúmenes persistentes. | 20 |
| **Automatización CI/CD** | Pipelines en GitHub Actions que incluyen tests, calidad de código y publicación automática. | 25 |
| **Calidad & Seguridad** | Integración de SonarQube con corrección demostrable de deuda técnica y vulnerabilidades. | 15 |
| **Disponibilidad Cloud** | Aplicación operativa en URL pública con despliegue automatizado desde la rama main. | 15 |

## Entregables Obligatorios y Defensa del Proyecto

1. **Enlaces a Repositorios**: Con los flujos de GitHub Actions visibles y en estado exitoso (`green builds`).
2. **Registro de Imágenes**: Enlaces a las imágenes en Docker Hub con tags de versión (`v1.0.0`, `latest`).
3. **Reporte de Calidad SonarQube**: Captura del dashboard mostrando el cumplimiento del "Quality Gate".
4. **URL Pública de Producción**: Enlace a la plataforma MediConnect funcionando en la nube.
5. **Video de "The Pipeline Challenge" (max 5 min)**: El estudiante debe realizar un cambio en vivo (ej. actualizar un mensaje de bienvenida), hacer commit/push, y narrar cómo el sistema automatizado detecta el cambio, valida el código, construye la imagen y actualiza el entorno de producción sin intervención manual.

> **Nota de Ingeniería**: Tu código deja de ser un simple software para convertirse en un servicio resiliente. La automatización no es un lujo, es el único camino para garantizar que MediConnect pueda escalar y salvar vidas sin interrupciones técnicas.
