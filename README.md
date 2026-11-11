# Users Management — Despliegue en Render (PaaS) y CD/CI (Guía 3)

Microservicio backend desarrollado con **Java 17** y **Spring Boot 3.3.5** para la gestión de usuarios, autenticación con **JWT** y control de acceso basado en roles (RBAC), siguiendo **Arquitectura Hexagonal** y **Domain-Driven Design (DDD)**.

Esta versión corresponde a la **Guía 3**, enfocada en desplegar el contenedor Docker en **Render (PaaS)** conectado a MySQL remoto en **Aiven (SSL REQUIRED)** y habilitar el flujo completo de **CI/CD**.

---

## Arquitectura y Flujo CI/CD

- **Integración Continua (CI):** El workflow [`.github/workflows/ci.yml`](.github/workflows/ci.yml) ejecuta `./mvnw clean verify` (compilación, 194 pruebas unitarias/web con JUnit/Mockito y empaquetado) en cada `push` o `pull_request` hacia `main`.
- **Despliegue Continuo (CD):** El Blueprint [`render.yaml`](render.yaml) construye la imagen usando [`Dockerfile`](Dockerfile), configura `autoDeployTrigger: commit` sobre la rama `main` y monitorea el servicio mediante `GET /health`.
- **Spring Composition Root:** `Main` inicia el contexto y las dependencias se resuelven mediante configuración e inyección de Spring.

---

## Verificación Local

Copie `.env.example` a `.env` con sus credenciales reales (nunca suba `.env` al repositorio) y ejecute:

```bash
# Windows
mvnw.cmd clean verify
mvnw.cmd spring-boot:run

# Linux / macOS
./mvnw clean verify
./mvnw spring-boot:run
```

---

## Despliegue en Render (PaaS)

1. Crear una cuenta o iniciar sesión en **Render** mediante **GitHub (SSO)** con autenticación de dos factores (**2FA**) habilitada.
2. Instalar/autorizar la aplicación **Render** en GitHub y conectar este repositorio.
3. Crear un **Blueprint** (detecta [`render.yaml`](render.yaml)) o **Web Service** con:
   - **Runtime:** `Docker`
   - **Branch:** `main`
   - **Region:** `Virginia (US East)` (recomendado por cercanía a la BD)
   - **Plan:** `Free ($0/month)`
4. Configurar en el panel **Environment** las variables secretas (`sync: false`):
   - `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD` (instancia MySQL en Aiven con `DB_SSL_MODE=REQUIRED`)
   - `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_FROM_ADDRESS` (servidor SMTP Gmail TLS 587)
   - `JWT_SECRET` (autogenerado en Base64 por `render.yaml` o definido manualmente con mínimo 32 bytes Base64)
5. Pulsar **Deploy** y verificar los logs hasta confirmar `Your service is live`.

> **Buenas prácticas DevSecOps:** Ninguna credencial o secreto debe guardarse en `application.properties` ni en `render.yaml`.

---

## Catálogo de Endpoints y Verificación en la Nube

| Método | Ruta | Seguridad | Propósito |
| :---: | :--- | :---: | :--- |
| **GET** | `/` | Protegido | Retorna `401 Unauthorized` confirmando el filtro JWT (`RestAuthenticationEntryPoint`). |
| **GET** | `/health` | Público | Retorna `{"status":"UP","name":"users-management","version":"2.2.0"}`. |
| **GET** | `/swagger-ui/index.html` | Público | Documentación interactiva OpenAPI 3.0. |
| **POST** | `/api/auth/login` | Público | Autentica usuario activo y emite el token Bearer JWT. |
| **POST** | `/api/users` | Público | Crea usuario en estado inicial `PENDING` y envía correo asíncrono. |
| **GET** | `/api/users` | JWT (`ADMIN`, `REVIEWER`) | Lista todos los usuarios registrados. |
| **GET** | `/api/users/{id}` | JWT (`ADMIN`, `REVIEWER`) | Consulta un usuario por su identificador. |
| **PUT** | `/api/users/{id}` | JWT (`ADMIN`) | Actualiza datos, estado o contraseña (útil para rotar el admin inicial). |
| **DELETE** | `/api/users/{id}` | JWT (`ADMIN`) | Elimina un usuario por su ID. |

La colección lista para importar en Postman, Insomnia o Boomerang se encuentra en [`collections/Taller de MicroServicios Spring Boot Hexagonal.json`](collections/Taller%20de%20MicroServicios%20Spring%20Boot%20Hexagonal.json).
