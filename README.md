# Backend — API REST Despachos 🚚

API REST desarrollada con **Spring Boot 3.4.4** y **Java 17** para la gestión de despachos de Innovatech Chile.

---

## 🏗️ Arquitectura de despliegue (EP3/EFT — ECS Fargate)

```
Internet
   │
   ▼
Application Load Balancer (innovatech-alb)
   │  ruta: /api/v1/despachos*
   ▼
ECS Fargate — Service: innovatech-svc-despachos
   │  Task Definition: innovatech-despachos (CPU 512 / 1024 MB)
   │  Container: despachos (puerto 8081)
   ▼
Amazon RDS MySQL (innovatech-db) — esquema despachos_db
```

- **Orquestación:** Amazon ECS con capacity provider Fargate (sin servidores que administrar).
- **Autoscaling:** Target Tracking por CPU (objetivo 50%), min 1 / max 3 tasks.
- **Imagen:** publicada en Amazon ECR (`innovatech-despachos`), con escaneo de vulnerabilidades automático (`scanOnPush`).
- **Secrets:** credenciales de base de datos gestionadas por AWS Secrets Manager, inyectadas como `secrets` en la Task Definition (nunca en texto plano).
- **Observabilidad:** logs en CloudWatch (`/ecs/innovatech-despachos`).
- **Health check:** Spring Boot Actuator expuesto en `/actuator/health`.

> Esta arquitectura reemplaza el despliegue manual en EC2 de la EP2. El historial de cómo se migró está en los commits de la rama `deploy`.

---

## ⚙️ Variables de entorno

| Variable | Origen | Descripción |
|---|---|---|
| `DB_ENDPOINT` | Task Definition (`environment`) | Endpoint de la instancia RDS |
| `DB_PORT` | Task Definition (`environment`) | Puerto de MySQL (3306) |
| `DB_NAME` | Task Definition (`environment`) | `despachos_db` |
| `DB_USERNAME` | Task Definition (`secrets`) | Desde AWS Secrets Manager |
| `DB_PASSWORD` | Task Definition (`secrets`) | Desde AWS Secrets Manager |

---

## 🐳 Desarrollo local

```bash
docker compose up --build
```

Esto levanta el backend junto a una instancia MySQL local para desarrollo, sin necesidad de credenciales de AWS.

---

## 🔄 CI/CD

El pipeline (`.github/workflows/deploy.yml`) se dispara en cada push a la rama `deploy`:

1. Tests unitarios con Maven (excluye el test de carga de contexto, que requiere conexión real a RDS).
2. Build de la imagen Docker.
3. Push a Amazon ECR (tags `latest` y hash del commit, para trazabilidad).
4. Registro de nueva revisión de la Task Definition.
5. Deploy automático al servicio ECS, esperando estabilidad (`wait-for-service-stability`).

**Secrets requeridos en GitHub:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` (credenciales temporales de AWS Academy Learner Lab — deben actualizarse cada vez que el laboratorio rota la sesión, aprox. cada 4 horas).

---

## 📚 Endpoints principales

Base path: `/api/v1/despachos`

Documentación interactiva (Swagger/OpenAPI) disponible en `/swagger-ui.html`.
