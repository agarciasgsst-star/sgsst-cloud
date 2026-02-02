# Arquitectura Técnica
# Sistema SG-SST Multi-Tenant

**Versión:** 1.0  
**Stack Principal:** Next.js + NestJS + PostgreSQL + AWS  

---

## 1. STACK TECNOLÓGICO RECOMENDADO

### Frontend
- **Framework:** Next.js 14 (App Router)
- **Lenguaje:** TypeScript 5.x
- **UI Library:** React 18
- **Componentes:** shadcn/ui (Radix UI + Tailwind CSS)
- **Estado:** Zustand (ligero, menos boilerplate que Redux)
- **Formularios:** React Hook Form + Zod (validación)
- **Tablas:** TanStack Table v8
- **Gráficas:** Recharts (ligero) o Apache ECharts (avanzado)
- **Drag & Drop:** dnd-kit
- **Fechas:** date-fns
- **PDF Client:** react-pdf (visualización)
- **Estilos:** Tailwind CSS 3.x

**Justificación:**
- Next.js 14 ofrece SSR, ISR, y excelente DX
- TypeScript reduce bugs en producción
- shadcn/ui: componentes accesibles, customizables, sin dependencia pesada
- Zustand: menos complejo que Redux, ideal para equipos pequeños

---

### Backend
- **Framework:** NestJS 10.x (Node.js)
- **Lenguaje:** TypeScript 5.x
- **ORM:** Prisma 5.x (type-safe, excelente DX)
- **Validación:** class-validator + class-transformer
- **Auth:** Passport.js + JWT
- **File Upload:** Multer + Sharp (resize imágenes)
- **Queue:** BullMQ + Redis (jobs asíncronos)
- **Email:** Nodemailer + Handlebars (templates)
- **PDF Generation:** Puppeteer (headless Chrome)
- **Excel:** ExcelJS
- **Logging:** Winston + Morgan

**Justificación:**
- NestJS: estructura modular, fácil escalar, excelente para equipos
- Prisma: migraciones automáticas, type-safety completo
- BullMQ: manejo robusto de jobs (reportes, emails masivos)

---

### Base de Datos
- **Principal:** PostgreSQL 15 (RDS AWS)
- **Schema:** Multi-tenant con schema por empresa
- **Cache:** Redis 7.x (sesiones, rate limiting, queue)
- **Full-Text Search:** PostgreSQL tsvector (búsqueda evidencias)

**Justificación:**
- PostgreSQL: robusto, ACID, JSON support, excelente para multi-tenant
- Schema por tenant: aislamiento total, fácil backup/restore por empresa
- Redis: performance en sesiones y cache

---

### Almacenamiento
- **Archivos:** AWS S3 (estándar)
- **Estructura:** `/empresa-{id}/evidencias/{actividad-id}/{timestamp}-{filename}`
- **CDN:** CloudFront (serving rápido)
- **Backup:** S3 Versioning + Lifecycle (archivos a Glacier tras 90 días)

---

### Infraestructura y DevOps
- **Hosting:** AWS
  - Frontend: Vercel (Next.js optimizado) o AWS Amplify
  - Backend: ECS Fargate (containers) o EC2 (si budget limitado)
  - DB: RDS PostgreSQL Multi-AZ
  - Storage: S3 + CloudFront
  - Queue: ElastiCache Redis
- **CI/CD:** GitHub Actions
- **Containers:** Docker + Docker Compose (dev)
- **Monitoreo:** CloudWatch + Sentry (errores)
- **Logs:** CloudWatch Logs + structured logging (JSON)

**Alternativa Low-Budget:**
- Frontend: Vercel (gratis para 1 proyecto)
- Backend: Railway.app o Render.com (desde $7/mes)
- DB: Supabase (Postgres managed, $25/mes)
- Storage: S3 ($0.023/GB)

---

### Autenticación y Seguridad
- **Auth:** JWT (access token 15min, refresh token 7 días)
- **Hash:** bcrypt (passwords)
- **Encryption:** AES-256-GCM (datos sensibles en DB)
- **HTTPS:** TLS 1.3 obligatorio
- **CORS:** Whitelist de dominios
- **Rate Limiting:** 100 req/min por IP (Redis)
- **OWASP:** Helmet.js (headers security)

---

## 2. ARQUITECTURA DE ALTO NIVEL

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND (Next.js)                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Dashboard│  │ Empresas │  │ PHVA     │  │ Reportes │       │
│  │          │  │ Multi    │  │ Activities│  │ PDF/Excel│       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
│                                                                 │
│  - Server Components (SSR)                                      │
│  - Client Components (interactividad)                           │
│  - API Routes (BFF pattern para auth)                           │
└─────────────────────────────────────────────────────────────────┘
                              ↕ HTTPS/REST
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY / BACKEND                      │
│                         (NestJS)                                │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Auth      │  │  Companies  │  │  Activities │            │
│  │  Module     │  │   Module    │  │   Module    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  Evidences  │  │  Reports    │  │  Audits     │            │
│  │   Module    │  │   Module    │  │   Module    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
│  - Guards (RBAC)                                                │
│  - Interceptors (logging, transform)                            │
│  - Pipes (validation)                                           │
│  - Services (business logic)                                    │
└─────────────────────────────────────────────────────────────────┘
           ↕                    ↕                    ↕
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │PostgreSQL│        │  Redis   │        │   S3     │
    │  (RDS)   │        │ (Cache + │        │ (Files)  │
    │          │        │  Queue)  │        │          │
    └──────────┘        └──────────┘        └──────────┘
           ↕                                       ↕
    ┌──────────┐                           ┌──────────┐
    │ Scheduled│                           │CloudFront│
    │  Backups │                           │  (CDN)   │
    └──────────┘                           └──────────┘

    ┌─────────────────────────────────────────────┐
    │         BACKGROUND JOBS (BullMQ)            │
    │  - Email notifications (daily/weekly)       │
    │  - PDF generation (async)                   │
    │  - Excel exports                            │
    │  - Alerts processing                        │
    └─────────────────────────────────────────────┘
```

---

## 3. MÓDULOS PRINCIPALES DEL BACKEND

### 3.1 Auth Module
**Responsabilidades:**
- Login/Logout
- Registro de usuarios (por invitación)
- JWT generation (access + refresh tokens)
- Password reset
- MFA (opcional, fase 2)

**Endpoints:**
```
POST /auth/login
POST /auth/logout
POST /auth/refresh
POST /auth/forgot-password
POST /auth/reset-password
GET  /auth/me (usuario actual)
```

---

### 3.2 Companies Module
**Responsabilidades:**
- CRUD de empresas
- Parametrización (tamaño, riesgo, trabajadores)
- Gestión de sedes y procesos
- Invitación de usuarios a empresa

**Endpoints:**
```
GET    /companies (lista para super admin)
POST   /companies (crear empresa)
GET    /companies/:id
PATCH  /companies/:id
DELETE /companies/:id
POST   /companies/:id/users (invitar usuario)
GET    /companies/:id/settings
PATCH  /companies/:id/settings
```

---

### 3.3 Users Module
**Responsabilidades:**
- Gestión de usuarios
- Asignación de roles (RBAC)
- Perfiles

**Endpoints:**
```
GET    /users (filtrable por empresa)
POST   /users
GET    /users/:id
PATCH  /users/:id
DELETE /users/:id (soft delete)
PATCH  /users/:id/roles
```

---

### 3.4 Activities Module
**Responsabilidades:**
- CRUD de actividades del SG-SST
- Estados (workflow)
- Asignación de responsables
- Vencimientos y alertas
- Bitácora de cambios

**Endpoints:**
```
GET    /companies/:companyId/activities (con filtros)
POST   /companies/:companyId/activities
GET    /companies/:companyId/activities/:id
PATCH  /companies/:companyId/activities/:id
DELETE /companies/:companyId/activities/:id
PATCH  /companies/:companyId/activities/:id/status
GET    /companies/:companyId/activities/:id/history
```

---

### 3.5 Evidences Module
**Responsabilidades:**
- Upload de archivos a S3
- Metadatos y versionado
- Búsqueda y filtrado
- Expiración de documentos

**Endpoints:**
```
POST   /companies/:companyId/evidences (upload)
GET    /companies/:companyId/evidences (lista con filtros)
GET    /companies/:companyId/evidences/:id
PATCH  /companies/:companyId/evidences/:id (metadata)
DELETE /companies/:companyId/evidences/:id
GET    /companies/:companyId/evidences/:id/download
GET    /companies/:companyId/evidences/search?q=...
```

---

### 3.6 Assessments Module (Autoevaluación 0312)
**Responsabilidades:**
- Generación de checklist según parametrización
- Cálculo de puntaje
- Identificación de brechas
- Generación automática de plan de mejora

**Endpoints:**
```
GET    /companies/:companyId/assessments
POST   /companies/:companyId/assessments (nueva autoevaluación)
GET    /companies/:companyId/assessments/:id
PATCH  /companies/:companyId/assessments/:id/items/:itemId (respuesta)
POST   /companies/:companyId/assessments/:id/calculate
POST   /companies/:companyId/assessments/:id/generate-improvement-plan
GET    /companies/:companyId/assessments/:id/export (PDF)
```

---

### 3.7 Improvement Plans Module
**Responsabilidades:**
- Gestión de acciones de mejora
- Seguimiento de progreso
- Cierre con evidencias

**Endpoints:**
```
GET    /companies/:companyId/improvement-plans
POST   /companies/:companyId/improvement-plans
GET    /companies/:companyId/improvement-plans/:id/actions
POST   /companies/:companyId/improvement-plans/:id/actions
PATCH  /companies/:companyId/improvement-plans/:id/actions/:actionId
```

---

### 3.8 Reports Module
**Responsabilidades:**
- Generación de reportes en PDF/Excel
- Templates personalizables
- Queue de generación asíncrona

**Endpoints:**
```
POST   /companies/:companyId/reports/executive (genera PDF mensual)
POST   /companies/:companyId/reports/assessment (autoevaluación 0312)
POST   /companies/:companyId/reports/calendar (Excel cronograma)
GET    /companies/:companyId/reports/:jobId/status
GET    /companies/:companyId/reports/:jobId/download
```

---

### 3.9 Notifications Module
**Responsabilidades:**
- Email notifications (SMTP)
- Alertas en app
- Digest semanal

**Endpoints:**
```
GET    /notifications (usuario actual)
PATCH  /notifications/:id/read
POST   /notifications/mark-all-read
```

---

### 3.10 Audit Logs Module
**Responsabilidades:**
- Registro de todas las acciones críticas
- Consulta de bitácora

**Endpoints:**
```
GET    /companies/:companyId/audit-logs (filtros: usuario, fecha, entidad)
GET    /audit-logs (super admin: todas las empresas)
```

---

## 4. MODELO MULTI-TENANT

### Estrategia: Schema-per-Tenant (PostgreSQL Schemas)

**Ventajas:**
- Aislamiento total entre empresas
- Backup/restore por empresa fácil
- Escalabilidad moderada (hasta ~1000 tenants)
- Queries simples (no filtro tenant_id en todos los queries)

**Estructura:**
```
Database: sgsst_platform

Schemas:
- public (tablas compartidas: users, super_admin_config)
- company_1 (tablas de la empresa 1)
- company_2 (tablas de la empresa 2)
- ...
- company_N

Cada schema tiene tablas:
- activities
- evidences
- assessments
- improvement_plans
- audits
- ...
```

**Conexión Dinámica:**
```typescript
// En cada request, se determina el schema según empresa del usuario
const companySchema = `company_${user.companyId}`;
await prisma.$executeRaw`SET search_path TO ${companySchema}`;
```

**Alternativa (si >1000 empresas):** Row-Level Security (RLS) con tenant_id en todas las tablas

---

## 5. SEGURIDAD Y RBAC

### Roles y Permisos

| Rol | Nivel | Permisos |
|-----|-------|----------|
| **SUPER_ADMIN** | Plataforma | Full access a todas las empresas |
| **COMPANY_ADMIN** | Empresa | Gestión completa de su empresa |
| **AUDITOR** | Empresa | Lectura todo, escritura en auditorías/hallazgos |
| **AREA_LEADER** | Empresa | Actividades asignadas, evidencias de su área |
| **EMPLOYEE** | Empresa | Solo lectura (fase 2: reportar incidentes) |
| **CLIENT_READONLY** | Empresa | Dashboard y reportes, sin edición |

**Implementación:**
```typescript
// Guard en NestJS
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('COMPANY_ADMIN', 'AUDITOR')
@Get('companies/:id/activities')
async getActivities() { ... }

// Middleware verifica:
1. Token JWT válido
2. Usuario tiene rol permitido
3. Usuario tiene acceso a esa empresa (tenant check)
```

---

## 6. FLUJO DE DATOS CRÍTICOS

### 6.1 Carga de Evidencias
```
1. Frontend: Usuario selecciona archivo
2. Frontend → Backend: POST /evidences (multipart/form-data)
3. Backend: Valida (tipo, tamaño <25MB, virus scan opcional)
4. Backend → S3: Upload con key único
5. Backend → DB: Inserta metadata (filename, S3 key, autor, fecha)
6. Backend → Frontend: Retorna evidencia creada
7. Si actividad requiere evidencia obligatoria → actualiza estado
```

---

### 6.2 Generación de Reportes
```
1. Frontend: Usuario solicita reporte (POST /reports/executive)
2. Backend: Crea job en BullMQ queue
3. Backend → Frontend: Retorna job_id
4. Worker: Procesa job
   - Query datos desde DB
   - Renderiza HTML con Handlebars
   - Genera PDF con Puppeteer
   - Sube PDF a S3
   - Actualiza job status = 'completed'
5. Frontend: Polling GET /reports/:jobId/status
6. Cuando completed → GET /reports/:jobId/download (signed URL S3)
```

---

### 6.3 Autoevaluación 0312
```
1. Admin crea nueva autoevaluación
2. Backend:
   - Lee parametrización de empresa (tamaño, riesgo)
   - Genera checklist dinámico con 21 estándares y N ítems según matriz 0312
   - Inserta assessment + items en DB
3. Usuario responde items (SI/NO/PARCIAL/NO_APLICA)
4. Usuario solicita cálculo (POST /assessments/:id/calculate)
5. Backend:
   - Calcula puntaje por estándar
   - Aplica ponderación según riesgo/tamaño
   - Identifica brechas (items NO o PARCIAL)
   - Genera plan de mejora automático con acciones sugeridas
6. Frontend muestra resultados con gráficas
7. Usuario puede exportar a PDF
```

---

## 7. ESTRATEGIA DE DESPLIEGUE

### Entornos

| Entorno | Propósito | Stack |
|---------|-----------|-------|
| **Development** | Local development | Docker Compose (Postgres + Redis + Minio) |
| **Staging** | QA y demos | AWS (RDS + ElastiCache + S3) |
| **Production** | Clientes reales | AWS (Multi-AZ, backups, monitoring) |

---

### Docker Setup (Development)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: sgsst_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  minio: # S3 local
    image: minio/minio
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio_data:/data

  backend:
    build: ./backend
    ports:
      - "3001:3001"
    environment:
      DATABASE_URL: postgresql://dev:dev123@postgres:5432/sgsst_dev
      REDIS_URL: redis://redis:6379
      S3_ENDPOINT: http://minio:9000
    depends_on:
      - postgres
      - redis
      - minio

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:3001
    depends_on:
      - backend

volumes:
  postgres_data:
  minio_data:
```

---

### CI/CD Pipeline (GitHub Actions)

**Workflow:**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test
      - run: npm run lint

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker images
        run: |
          docker build -t sgsst-backend ./backend
          docker build -t sgsst-frontend ./frontend
      
      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
          docker push $ECR_REGISTRY/sgsst-backend:latest
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service --cluster sgsst-prod --service backend --force-new-deployment
      
      - name: Deploy Frontend to Vercel
        run: vercel --prod
```

---

## 8. MONITOREO Y OBSERVABILIDAD

### Logs Estructurados
```typescript
// Ejemplo con Winston
logger.info('Activity updated', {
  userId: user.id,
  companyId: company.id,
  activityId: activity.id,
  previousStatus: 'IN_PROGRESS',
  newStatus: 'CLOSED',
  timestamp: new Date().toISOString()
});
```

**Enviado a CloudWatch Logs → Consulta con CloudWatch Insights**

---

### Métricas Clave
- **Performance:** P50, P95, P99 de response time (por endpoint)
- **Errores:** Rate de 4xx, 5xx
- **Negocio:** Actividades creadas/día, reportes generados, usuarios activos
- **Infraestructura:** CPU, RAM, DB connections, queue length

**Herramienta:** CloudWatch Metrics + Dashboards

---

### Alertas
```
- Response time P95 > 1s por 5min → Slack
- Error rate > 1% por 5min → Email + Slack
- Queue length > 1000 → Investigar
- DB connections > 80% → Escalar
```

---

## 9. RIESGOS TÉCNICOS Y MITIGACIONES

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| **Performance DB con >100 empresas** | Media | Alto | Indexación agresiva, pool de conexiones, read replicas |
| **S3 costs con muchos archivos** | Media | Medio | Lifecycle policies (Glacier tras 90 días), compresión |
| **PDF generation lenta** | Alta | Medio | Queue asíncrona, cache de templates, workers dedicados |
| **Seguridad: acceso cruzado entre tenants** | Baja | Crítico | Tests automatizados de RBAC, audit logs completos |
| **Escalabilidad del schema-per-tenant** | Baja | Alto | Monitorear límite (~1000 schemas), plan B con RLS |
| **Email delivery issues** | Media | Bajo | SES con retry policy, fallback a SendGrid |
| **Falta de expertise en normatividad** | Alta | Medio | Validación con consultor SG-SST real, iteración con feedback |

---

## 10. CRITERIOS DE ÉXITO ARQUITECTURA

### Performance
- ✅ Response time P95 < 500ms
- ✅ Carga inicial frontend < 2s
- ✅ Upload de archivo 10MB < 3s
- ✅ Generación de PDF < 30s (asíncrona)

### Escalabilidad
- ✅ Soporta 100 empresas concurrentes en MVP
- ✅ Horizontal scaling sin cambios de código
- ✅ DB puede crecer a 500GB sin degradación

### Seguridad
- ✅ OWASP Top 10 mitigado
- ✅ Audit log completo
- ✅ Backups diarios automáticos
- ✅ Encryption at rest + in transit

### Mantenibilidad
- ✅ Coverage de tests > 80%
- ✅ Documentación de APIs (Swagger/OpenAPI)
- ✅ Logs estructurados consultables
- ✅ Rollback en <5min

---

**FIN DE ARQUITECTURA TÉCNICA**

---

## PRÓXIMO ENTREGABLE
1. ✅ PRD  
2. ✅ Arquitectura Técnica  
3. 🔄 Modelo de Datos (ERD)  
4. ⏳ API Design  
5. ⏳ UX/Wireframes  
6. ⏳ Backlog MVP Priorizado
