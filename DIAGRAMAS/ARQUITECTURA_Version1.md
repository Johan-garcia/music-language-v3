# Arquitectura Lógica y Física – Proyecto Music Language (Backend)

Este documento describe la arquitectura objetivo (estado “backend final”) para el proyecto Music Language (v3). Incluye: visión funcional (arquitectura lógica), despliegue (arquitectura física), dominios, flujos, escalabilidad, seguridad, observabilidad y roadmap evolutivo.

---

## 1. Objetivos Arquitectónicos

- Escalable: crecer desde un monolito modular hacia servicios especializados.
- Mantenible: separación clara por dominios (auth, música, letras, recomendaciones, analytics).
- Segura: protección de datos, control de acceso, gestión de secretos y monitoreo.
- Observabilidad: métricas, logs estructurados, trazas distribuidas y alertas.
- Extensible: facilitar incorporación de nuevos proveedores (YouTube, otros modelos ML).
- Eficiente: uso de caché, precomputación y batch jobs para reducir latencia y costos.

---

## 2. Arquitectura Lógica

La arquitectura lógica define los bloques funcionales (qué hace el sistema) sin atarse todavía a infraestructura específica.

### 2.1 Capas Principales

| Capa | Rol |
|------|-----|
| Presentación | Frontend Web / Mobile / SDKs / Clientes externos |
| API / Aplicación | FastAPI (REST + OpenAPI) – orquesta dominio y validación |
| Servicios de Dominio | Lógica de negocio: recomendaciones, perfiles, integración proveedores |
| Datos | PostgreSQL, Redis (caché), almacenamiento de modelos/datasets |
| Observabilidad | Logging, métricas, trazas, dashboards |
| Seguridad | Autenticación, autorización, secretos, rate limiting |
| Integraciones externas | Spotify, Genius, (YouTube futuro) |
| Procesamiento Offline | Workers/rejobs para tareas batch (recomendaciones, idioma, ML) |

### 2.2 Dominios (Bounded Contexts sugeridos)

| Dominio | Responsabilidades |
|---------|-------------------|
| Identity & Access | Usuarios, roles, JWT, scopes |
| User Profile | Preferencias, idioma preferido, ajustes |
| Music Integration | Consultas a Spotify / (YouTube) para metadata |
| Lyrics | Búsqueda y normalización (Genius) |
| Recommendations | Generación basada en seeds + heurísticas / ML |
| Analytics | Captura de eventos y KPIs |
| Language Intelligence | Detección / cache de idioma de tracks |
| Admin & Ops | Gestión avanzada, métricas, auditoría |

### 2.3 Modelo Conceptual (Simplificado)

```
User (id, email, password_hash, role, preferred_lang, created_at)
SpotifyAccount (user_id, spotify_user_id, access_token, refresh_token, scope, expires_at)
AnalyticsEvent (id, user_id, type, payload JSONB, created_at)
RecommendationCache (user_id, recommendations JSONB, updated_at)
TrackLanguageCache (track_id, lang, detected_at)
( Futuro ) Playlist (id, user_id, name, tracks[] )
( Futuro ) UserPreference (user_id, key, value)
```

### 2.4 Flujos Clave

1. Registro / Login → Genera JWT → Cliente persiste token.
2. Vincular Spotify → OAuth Code → Guardar tokens (access/refresh).
3. Obtener Top Tracks → Validar JWT → Refrescar token Spotify si vencido → Llamar API externa.
4. Recomendaciones → Leer seeds (top user) → Generar / reordenar → Cachear → Responder.
5. Búsqueda de letras → Consultar Genius → Selección mejor hit → Devolver metadata + URL.
6. Analítica → Cliente envía evento → Persistir (y en futuro: publicar a cola).
7. Detección Idioma (batch) → Worker procesa tracks nuevos → Cachea resultado.

---

## 3. Arquitectura Física

Describe cómo se despliegan e interconectan los componentes.

### 3.1 Entornos

| Entorno | Objetivo | Características |
|---------|----------|-----------------|
| Dev | Desarrollo local | Docker Compose, datos falsos |
| Staging | Pre-producción | Config similar a prod, pruebas integrales |
| Prod | Producción | Alta disponibilidad, observabilidad completa |

### 3.2 Diagrama (Vista Alta – Producción)

```mermaid
flowchart LR
    subgraph Clients[Clientes]
        A[Web / Frontend] -->|HTTPS| GW(API Gateway / Ingress)
        B[Mobile / SDK] -->|HTTPS| GW
    end

    GW --> API[FastAPI (Containers N)]
    API --> REDIS[(Redis Cache)]
    API --> PG[(PostgreSQL Managed)]
    API --> OBJ[(S3 / MinIO: modelos & datasets)]
    API --> Q[(Cola/Event Bus - futuro)]
    W[Worker / Batch Jobs] --> PG
    W --> REDIS
    W --> OBJ
    API --> SPOT[Spotify API]
    API --> GEN[Genius API]
    API --> YT[(YouTube API - futuro)]

    subgraph Observability
        LOG[Logs Centralizados]
        MET[Métricas (Prometheus)]
        TR[Tracing (OTel / Jaeger)]
        DASH[Dashboards (Grafana)]
    end

    API --> LOG
    API --> MET
    API --> TR
    W --> LOG
    W --> MET
    W --> TR
```

### 3.3 Componentes Físicos

| Componente | Despliegue Sugerido |
|------------|---------------------|
| FastAPI API | Contenedores (Kubernetes / ECS) detrás de Ingress / LB |
| Worker (batch) | Deployment independiente / CronJobs |
| PostgreSQL | Servicio gestionado (RDS / CloudSQL / Aurora) |
| Redis | Servicio gestionado o cluster propio |
| Object Storage | S3 / MinIO (para modelos ML y datasets) |
| API Gateway / Ingress | Nginx / Traefik / Cloud Gateway |
| Observabilidad | Prometheus + Grafana + Loki/ELK + Jaeger/Tempo |
| Cola (futuro) | Kafka / RabbitMQ / SQS (para analytics y pipelines ML) |
| Secrets Manager | Vault / AWS Secrets Manager |

---

## 4. Mapeo Lógico → Físico

| Lógico | Físico Inicial | Escalamiento Futuro |
|--------|----------------|---------------------|
| API Core (usuarios, música, letras) | Monolito FastAPI | Separar microservicios (reco, analytics) |
| Recomendaciones runtime | Dentro del monolito | Servicio dedicado + modelos ML |
| Recomendaciones batch | CronJob / Worker | Cluster de jobs / pipelines ML |
| Detección idioma | Job periódico | Servicio NLP dedicado |
| Cache tokens/queries | Redis único | Redis cluster / Multi-tier |
| Persistencia | PostgreSQL instancia única HA | Replica + partición (si escala mucho) |
| Analítica ingest | Tabla + logs | Cola + Data Lake + ETL |
| Admin panel | Integrado en frontend | Portal operativo separado |
| ML models | S3 / carpeta | Registry con versionado (MLflow, etc.) |

---

## 5. Escalabilidad

- Escalado Horizontal: múltiples réplicas del API detrás de balanceador.
- Redis para:
  - Rate limiting (token bucket).
  - Cache de respuestas (top tracks, lyrics search).
  - Precomputación de recomendaciones.
- Estrategias:
  - Cache-aside (consultar cache → si falla → generar → almacenar).
  - Precomputar recomendaciones cada X horas para usuarios activos.
- Extraer microservicio de recomendaciones cuando:
  - Se use pipeline ML intensivo.
  - Latencia de inferencia requiera recursos especializados (GPU / embeddings).

---

## 6. Estrategia de Datos y Consistencia

| Tipo | Requisito | Estrategia |
|------|-----------|------------|
| Datos críticos (usuarios, tokens) | Fuerte | PostgreSQL (ACID) |
| Recomendaciones | Eventual | Cache JSONB + Redis |
| Idioma track | Eventual | Cache en tabla + refresh batch |
| Eventos analytics | Alta ingestión | Insert rápido + streaming (futuro) |
| Modelos ML | Versionado | Naming convención + checksum |

TTL sugeridos:
- Top tracks usuario: 10–15 min.
- Búsquedas lyrics: 30–60 min.
- Recomendaciones: 1–3 h (según frecuencia uso).
- Detección idioma: persistente (solo recálculo si heurística cambia).

---

## 7. Seguridad

| Aspecto | Medidas |
|---------|---------|
| Autenticación | JWT (exp corto 15–60 min) + refresh tokens (futuro) |
| Autorización | Roles (user/admin) + scopes |
| Secretos | Vault / Secrets Manager (no en repositorio) |
| Comunicación | HTTPS en todos los hops externos |
| Rate limiting | Por IP y por user_id |
| Hardening | Headers: HSTS, X-Frame-Options, CSP (frontend) |
| Dependencias | Escaneo (pip-audit / Trivy) en CI |
| Tokens externos | Rotación programada (Spotify/Genius) |
| Auditoría | Log de acciones sensibles (admin, cambios credenciales) |
| Protección DoS | WAF + límites + backpressure |
| Validación | Pydantic y sanitización de datos externos |

---

## 8. Observabilidad

### 8.1 Métricas Recomendadas

| Categoría | Métricas |
|-----------|----------|
| API | Latencia p50/p95/p99 por endpoint, tasa 4xx/5xx |
| Integraciones | Latencia Spotify/Genius, tasa de fallos, reintentos |
| Cache | Hit ratio, expiraciones |
| Recomendaciones | Tiempo generación, tamaño respuesta |
| Workers | Jobs procesados, fallos, cola pendiente |
| DB | Conexiones activas, locks, slow queries |

### 8.2 Logs

Formato JSON estructurado:
```
{
  "timestamp": "...",
  "level": "INFO",
  "trace_id": "...",
  "user_id": "...",
  "endpoint": "/music/me/top/tracks",
  "status_code": 200,
  "latency_ms": 123
}
```

### 8.3 Trazas

- OpenTelemetry SDK en FastAPI.
- Spans para: request entrante, llamada a Spotify, acceso a DB, caché.
- Exportación a Jaeger o Tempo.

### 8.4 Alertas (Ejemplos)

| Alerta | Condición |
|--------|-----------|
| Error rate alta | >5% 5xx en 5 min |
| Latencia elevada | p95 > 800ms sostenido |
| Fallo integraciones | >10% errores Spotify/Genius |
| Cache ineficiente | Hit ratio < 40% |
| Tokens Spotify expiran | Muchos refresh fallidos consecutivos |

---

## 9. CI/CD

Pipeline sugerido (GitHub Actions):

1. Lint + formato (ruff / black).
2. Tests (pytest + coverage).
3. Escaneo dependencias (pip-audit).
4. Build imagen Docker (multi-stage).
5. Escaneo imagen (Trivy).
6. Push a registry.
7. Deploy automático a Staging (Helm/Kustomize).
8. Smoke tests.
9. Deploy manual (aprobación) a Producción.

Control de versiones semántico (SemVer):
- PATCH: bugfix
- MINOR: nuevas features compatibles
- MAJOR: cambios incompatibles

---

## 10. Flujos de Secuencia (Simplificados)

### 10.1 Recomendaciones por Idioma

```
Cliente -> API /reco/by-language
API -> PostgreSQL (lee tokens Spotify)
API -> ensure_access_token (posible refresh)
API -> Spotify /me/top/tracks & /me/top/artists
API -> Spotify /recommendations (seeds + market)
API -> (cache opcional) -> Respuesta JSON
```

### 10.2 Búsqueda de Letras

```
Cliente -> /lyrics?artist=...&title=...
API -> Genius /search
API -> Heurística selección mejor hit
API -> Respuesta (genius_url + metadata)
```

### 10.3 Refresh Token Spotify

```
API -> Lee token + expires_at
Si expirado:
  API -> POST /api/token (Spotify)
  Spotify -> Nuevo access_token
  API -> Guarda en DB
Continuar operación original
```

---

