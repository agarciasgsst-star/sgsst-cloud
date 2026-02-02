# Backlog MVP Priorizado
# Sistema SG-SST Multi-Tenant

**Duración MVP:** 4-6 semanas (3 sprints de 2 semanas)  
**Equipo:** 2 desarrolladores full-stack  
**Metodología:** Scrum/Kanban híbrido

---

## 1. PRIORIZACIÓN Y CRITERIOS

### Criterios de Priorización (MoSCoW)
- **MUST:** Funcionalidad crítica sin la cual el MVP no es viable
- **SHOULD:** Funcionalidad importante que agrega valor significativo
- **COULD:** Funcionalidad deseable si hay tiempo
- **WON'T:** Fuera de scope del MVP (fase 2+)

### Definición de "Done"
- ✅ Código desarrollado y en repositorio
- ✅ Tests unitarios (coverage >70%)
- ✅ Code review aprobado
- ✅ Documentación de API actualizada
- ✅ Funcionalidad probada en staging
- ✅ Sin bugs críticos o blockers abiertos

---

## 2. ROADMAP MVP - 3 SPRINTS

### SPRINT 1 (Semanas 1-2): Fundación + Auth + Multi-Tenant
**Objetivo:** Infraestructura base, autenticación y gestión de empresas funcionando

**Entregables:**
- Setup de infraestructura (repos, CI/CD, DB, S3)
- Autenticación JWT completa
- CRUD de empresas y usuarios
- Dashboard básico (sin métricas aún)
- Creación automática de schema por empresa

---

### SPRINT 2 (Semanas 3-4): Core PHVA + Evidencias + Autoevaluación
**Objetivo:** Funcionalidad operativa central del SG-SST

**Entregables:**
- Gestión completa de actividades (CRUD + estados)
- Carga y gestión de evidencias (S3)
- Autoevaluación 0312 completa (checklist dinámico)
- Generación de plan de mejora automático
- Bitácora de auditoría

---

### SPRINT 3 (Semanas 5-6): Reportes + Alertas + Polish
**Objetivo:** Cierre del MVP con reportes, notificaciones y UX refinado

**Entregables:**
- Generación de reportes PDF/Excel (asíncrono)
- Sistema de notificaciones (email + in-app)
- Dashboard con KPIs principales
- Calendario de actividades
- Testing end-to-end
- Deploy a producción

---

## 3. BACKLOG DETALLADO POR SPRINT

---

## SPRINT 1: Fundación + Auth + Multi-Tenant

### 📦 US-1.1: Setup de Infraestructura
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** DevOps/Backend Lead

**Descripción:**
Como desarrollador, necesito configurar la infraestructura base del proyecto para poder comenzar el desarrollo.

**Tareas:**
1. Crear repos en GitHub (frontend, backend)
2. Configurar Docker Compose para desarrollo local (Postgres + Redis + Minio)
3. Setup CI/CD con GitHub Actions (lint, test, build)
4. Configurar AWS (RDS Postgres, ElastiCache Redis, S3, ECS/EC2)
5. Setup Vercel para frontend
6. Configurar Sentry (monitoreo de errores)
7. Configurar CloudWatch (logs, métricas)

**Criterios de Aceptación:**
- ✅ Desarrolladores pueden levantar stack completo con `docker-compose up`
- ✅ Pipeline CI/CD ejecuta tests automáticamente en cada push
- ✅ Ambientes staging y producción configurados en AWS
- ✅ Logs centralizados en CloudWatch

---

### 📦 US-1.2: Configuración Inicial de Frontend y Backend
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Full-Stack

**Descripción:**
Como desarrollador, necesito el scaffolding inicial de los proyectos frontend y backend.

**Tareas:**
1. Setup Next.js 14 con TypeScript, Tailwind, shadcn/ui
2. Setup NestJS con Prisma, Passport.js
3. Configurar estructura de módulos (Auth, Companies, Users, Activities, etc.)
4. Configurar variables de entorno (.env)
5. Setup de Prisma con PostgreSQL (conexión inicial)
6. Configurar CORS, Helmet.js (seguridad)

**Criterios de Aceptación:**
- ✅ Frontend corriendo en `localhost:3000`
- ✅ Backend corriendo en `localhost:3001`
- ✅ Conexión a DB exitosa
- ✅ Estructura de carpetas según arquitectura modular

---

### 📦 US-1.3: Modelo de Datos y Migraciones
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Backend

**Descripción:**
Como backend developer, necesito crear el modelo de datos completo en Prisma.

**Tareas:**
1. Definir schema Prisma para tablas en `public` (users, companies, audit_logs_global)
2. Crear función PL/pgSQL para creación de schema por tenant
3. Definir schema Prisma para tablas de empresa (activities, evidences, etc.)
4. Crear migraciones iniciales
5. Crear seeds para datos de prueba (1 super admin, 2 empresas demo)
6. Configurar triggers (updated_at, audit_logs)

**Criterios de Aceptación:**
- ✅ Comando `prisma migrate dev` ejecuta sin errores
- ✅ Comando `prisma db seed` carga datos de prueba
- ✅ Función `create_company_schema()` crea schema completo
- ✅ Triggers funcionando (updated_at automático)

---

### 📦 US-1.4: Autenticación JWT
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero poder iniciar sesión y mantener mi sesión activa de forma segura.

**Tareas:**
1. Implementar endpoint `POST /auth/login` con bcrypt
2. Implementar generación de access token (JWT, 15min) y refresh token (7 días)
3. Implementar endpoint `POST /auth/refresh`
4. Implementar endpoint `POST /auth/logout` (invalidar refresh token en Redis)
5. Implementar endpoint `GET /auth/me`
6. Crear guards de autenticación (JwtAuthGuard, RolesGuard)

**Criterios de Aceptación:**
- ✅ Login exitoso retorna tokens válidos
- ✅ Access token expira en 15 minutos
- ✅ Refresh token permite renovar access token
- ✅ Logout invalida refresh token
- ✅ Endpoints protegidos rechazan requests sin token válido

---

### 📦 US-1.5: Frontend - Páginas de Autenticación
**Prioridad:** MUST | **Estimación:** 3 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero una interfaz para iniciar sesión.

**Tareas:**
1. Crear página `/login` con formulario
2. Integrar con endpoint `POST /auth/login`
3. Guardar tokens en localStorage/cookies (httpOnly)
4. Crear contexto de autenticación (AuthContext con Zustand)
5. Implementar refresh automático de token
6. Implementar logout

**Criterios de Aceptación:**
- ✅ Formulario de login funcional con validación
- ✅ Login exitoso redirige a `/dashboard`
- ✅ Token se renueva automáticamente antes de expirar
- ✅ Logout limpia sesión y redirige a `/login`

---

### 📦 US-1.6: CRUD de Empresas (Backend)
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Backend

**Descripción:**
Como Super Admin, quiero gestionar empresas (crear, editar, listar).

**Tareas:**
1. Implementar endpoints CRUD de companies
2. Al crear empresa: ejecutar `create_company_schema()`
3. Seed de actividades PHVA base según parametrización
4. Implementar endpoint de settings por empresa
5. Crear servicio para carga de logo a S3

**Criterios de Aceptación:**
- ✅ `POST /companies` crea empresa y schema en DB
- ✅ Schema creado contiene ~60 actividades precargadas
- ✅ `GET /companies` lista empresas del super admin
- ✅ `PATCH /companies/:id` actualiza información
- ✅ Logo se sube a S3 y URL se guarda en DB

---

### 📦 US-1.7: Frontend - Gestión de Empresas
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como Super Admin, quiero ver y gestionar mis empresas desde el dashboard.

**Tareas:**
1. Crear página `/empresas` con tabla de empresas
2. Crear wizard modal para crear nueva empresa (5 pasos)
3. Formulario de edición de empresa
4. Integrar con endpoints del backend
5. Upload de logo con preview

**Criterios de Aceptación:**
- ✅ Tabla muestra todas las empresas con info clave
- ✅ Wizard guía al usuario en 5 pasos (datos básicos, parametrización, admin inicial, etc.)
- ✅ Creación exitosa muestra mensaje de confirmación
- ✅ Upload de logo funcional con preview

---

### 📦 US-1.8: CRUD de Usuarios (Backend)
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Backend

**Descripción:**
Como Admin de Empresa, quiero gestionar usuarios (invitar, editar roles, desactivar).

**Tareas:**
1. Implementar endpoints CRUD de users
2. Implementar invitación por email (generar token único)
3. Implementar endpoint de activación de cuenta
4. Implementar RBAC (verificación de permisos por rol)

**Criterios de Aceptación:**
- ✅ `POST /companies/:id/users` envía email de invitación
- ✅ Link de invitación válido por 7 días
- ✅ Usuario invitado puede activar cuenta y definir contraseña
- ✅ RBAC bloquea acciones no permitidas por rol

---

### 📦 US-1.9: Frontend - Gestión de Usuarios
**Prioridad:** SHOULD | **Estimación:** 3 pts | **Responsable:** Frontend

**Descripción:**
Como Admin de Empresa, quiero gestionar los usuarios de mi empresa.

**Tareas:**
1. Crear página `/usuarios` con tabla
2. Modal para invitar usuario (email, rol)
3. Editar rol de usuario existente
4. Activar/desactivar usuario

**Criterios de Aceptación:**
- ✅ Tabla muestra usuarios con rol, estado, último login
- ✅ Modal de invitación envía email correctamente
- ✅ Cambios de rol se aplican inmediatamente

---

### 📦 US-1.10: Dashboard Básico (Frontend)
**Prioridad:** SHOULD | **Estimación:** 3 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver un dashboard al iniciar sesión.

**Tareas:**
1. Crear página `/dashboard` con layout base
2. Sidebar con navegación (Empresas, Dashboard, Actividades, etc.)
3. Header con nombre de usuario, empresa, logout
4. Placeholder para KPIs (se implementan en Sprint 3)

**Criterios de Aceptación:**
- ✅ Dashboard carga después del login
- ✅ Sidebar navegable con iconos y labels
- ✅ Header muestra usuario actual y botón de logout
- ✅ Responsive (mobile-first)

---

### **Sprint 1 - Total Story Points:** 53 pts

---

## SPRINT 2: Core PHVA + Evidencias + Autoevaluación

### 📦 US-2.1: Backend - CRUD de Actividades
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero gestionar las actividades del SG-SST (crear, editar, eliminar, cambiar estado).

**Tareas:**
1. Implementar endpoints CRUD de activities
2. Implementar cambio de estado con validaciones
3. Implementar bitácora automática en cada cambio
4. Filtros avanzados (módulo PHVA, estado, responsable, fechas)
5. Endpoint de historial de cambios

**Criterios de Aceptación:**
- ✅ `GET /companies/:id/activities` con filtros funcionales
- ✅ Cambio a estado CLOSED valida evidencia obligatoria
- ✅ Cambio a NOT_APPLICABLE requiere justificación
- ✅ Toda modificación crea entrada en audit_logs
- ✅ Endpoint de historial retorna timeline de cambios

---

### 📦 US-2.2: Frontend - Vista de Actividades (Tabla)
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver y filtrar las actividades del SG-SST en una tabla.

**Tareas:**
1. Crear página `/actividades` con tabla (TanStack Table)
2. Implementar filtros (módulo, estado, responsable, fecha)
3. Columnas: código, nombre, responsable, vencimiento, estado, prioridad
4. Indicadores visuales (semáforos por estado, badges)
5. Paginación

**Criterios de Aceptación:**
- ✅ Tabla muestra actividades con columnas configurables
- ✅ Filtros actualizan resultados en tiempo real
- ✅ Color por estado (verde=cerrado, rojo=vencido, amarillo=pendiente)
- ✅ Paginación funcional (20 items por página)

---

### 📦 US-2.3: Frontend - Detalle de Actividad
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver el detalle completo de una actividad y poder editarla.

**Tareas:**
1. Crear modal o página `/actividades/:id`
2. Formulario de edición (nombre, descripción, responsable, fechas, etc.)
3. Botones de cambio de estado (con confirmación)
4. Sección de evidencias asociadas (lista + botón "Subir")
5. Timeline de historial de cambios

**Criterios de Aceptación:**
- ✅ Detalle muestra toda la información de la actividad
- ✅ Formulario editable (solo si usuario tiene permisos)
- ✅ Cambio de estado requiere confirmación
- ✅ Timeline muestra cambios ordenados cronológicamente

---

### 📦 US-2.4: Backend - Upload de Evidencias a S3
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero subir evidencias (archivos) asociadas a actividades.

**Tareas:**
1. Configurar Multer para upload multipart/form-data
2. Implementar validaciones (tipo, tamaño <25MB)
3. Subir archivo a S3 con key único
4. Guardar metadata en DB (evidences table)
5. Generar pre-signed URL para descarga (válida 1h)
6. Versionado: si se sube archivo con mismo nombre → nueva versión

**Criterios de Aceptación:**
- ✅ `POST /companies/:id/evidences` sube archivo a S3
- ✅ Archivos >25MB son rechazados (422 Error)
- ✅ Solo formatos permitidos: PDF, JPG, PNG, DOCX, XLSX
- ✅ Metadata guardada incluye: filename, size, uploader, timestamp
- ✅ Endpoint de download retorna pre-signed URL

---

### 📦 US-2.5: Frontend - Upload y Gestión de Evidencias
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero subir evidencias y asociarlas a actividades.

**Tareas:**
1. Crear modal de upload con drag & drop
2. Preview de archivos antes de subir
3. Progress bar durante upload
4. Vista de evidencias asociadas a actividad (thumbnails)
5. Botón de descarga (pre-signed URL)

**Criterios de Aceptación:**
- ✅ Drag & drop funcional
- ✅ Progress bar muestra % de upload
- ✅ Upload exitoso actualiza lista de evidencias
- ✅ Thumbnails para imágenes, iconos para otros tipos
- ✅ Descarga funcional con un clic

---

### 📦 US-2.6: Backend - Autoevaluación 0312
**Prioridad:** MUST | **Estimación:** 13 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero realizar autoevaluación según Res. 0312/2019 y obtener puntaje.

**Tareas:**
1. Crear matriz de estándares e ítems por parametrización (JSON config)
2. Implementar endpoint `POST /assessments` (genera checklist dinámico)
3. Implementar endpoint `PATCH /assessments/:id/items/:itemId` (responder ítem)
4. Implementar lógica de cálculo de puntaje (con ponderación)
5. Implementar identificación de brechas (ítems NO/PARCIAL)
6. Implementar generación automática de plan de mejora

**Criterios de Aceptación:**
- ✅ Crear autoevaluación genera 60 ítems (aprox) según empresa
- ✅ Responder todos los ítems permite calcular puntaje
- ✅ Puntaje calculado coincide con fórmula de Res. 0312
- ✅ Brechas identificadas correctamente (ítems no cumplidos)
- ✅ Plan de mejora generado con 1 acción por brecha

---

### 📦 US-2.7: Frontend - Autoevaluación 0312 (Wizard)
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero responder la autoevaluación 0312 en una interfaz guiada.

**Tareas:**
1. Crear wizard multi-paso (21 estándares)
2. Por cada estándar, mostrar ítems con radio buttons (SÍ/NO/PARCIAL/NO APLICA)
3. Campo opcional para asociar evidencia
4. Barra de progreso
5. Paso final: calcular puntaje y mostrar resultados
6. Gráficas de puntaje por estándar (Recharts)

**Criterios de Aceptación:**
- ✅ Wizard navegable (anterior/siguiente)
- ✅ Respuestas se guardan automáticamente
- ✅ Resultados muestran puntaje total, % cumplimiento, gráficas
- ✅ Botón "Generar plan de mejora" funcional

---

### 📦 US-2.8: Backend - Plan de Mejora Automático
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Backend

**Descripción:**
Como sistema, quiero generar automáticamente un plan de mejora desde autoevaluación.

**Tareas:**
1. Implementar `POST /assessments/:id/generate-improvement-plan`
2. Crear plan con acciones (1 por brecha identificada)
3. Sugerir descripción de acción basada en ítem incumplido
4. Asignar fechas por defecto (90 días desde hoy)
5. Estado inicial: PENDING

**Criterios de Aceptación:**
- ✅ Plan creado con N acciones (N = brechas)
- ✅ Cada acción tiene descripción sugerida
- ✅ Fechas por defecto asignadas (inicio=hoy, vencimiento=+90 días)

---

### 📦 US-2.9: Frontend - Vista de Plan de Mejora
**Prioridad:** SHOULD | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver y gestionar el plan de mejora generado.

**Tareas:**
1. Crear página `/planes-de-mejora` con lista de planes
2. Crear página `/planes-de-mejora/:id` con detalle
3. Tabla de acciones con estado, responsable, vencimiento
4. Editar acción (asignar responsable, ajustar fechas)
5. Cambiar estado de acción (PENDING → IN_PROGRESS → COMPLETED)
6. Progress bar del plan (% acciones completadas)

**Criterios de Aceptación:**
- ✅ Lista de planes con progress % visible
- ✅ Detalle muestra todas las acciones
- ✅ Edición de acción funcional
- ✅ Cambio de estado actualiza progress bar

---

### 📦 US-2.10: Backend - Bitácora de Auditoría
**Prioridad:** MUST | **Estimación:** 3 pts | **Responsable:** Backend

**Descripción:**
Como sistema, quiero registrar automáticamente todas las acciones críticas.

**Tareas:**
1. Crear interceptor global para capturar CREATE/UPDATE/DELETE
2. Guardar en audit_logs_company (user, action, entity, changes)
3. Guardar IP y user agent
4. Implementar endpoint `GET /audit-logs` (filtros: user, date, entity)

**Criterios de Aceptación:**
- ✅ Toda acción CUD genera entrada en audit_logs
- ✅ Cambios incluyen before/after (JSON diff)
- ✅ Endpoint de consulta retorna logs filtrados

---

### **Sprint 2 - Total Story Points:** 65 pts

---

## SPRINT 3: Reportes + Alertas + Polish

### 📦 US-3.1: Backend - Generación de Reportes PDF
**Prioridad:** MUST | **Estimación:** 13 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero generar reportes en PDF (autoevaluación, informe ejecutivo).

**Tareas:**
1. Configurar Puppeteer (headless Chrome)
2. Crear templates HTML con Handlebars (3 templates: autoevaluación, ejecutivo, auditoría)
3. Implementar queue con BullMQ para generación asíncrona
4. Worker: renderizar HTML → PDF (Puppeteer) → subir a S3
5. Endpoints: `POST /reports/executive`, `GET /reports/:jobId/status`, `GET /reports/:jobId/download`

**Criterios de Aceptación:**
- ✅ Request de reporte retorna job_id inmediatamente
- ✅ Worker genera PDF en <30 segundos
- ✅ PDF incluye logo de empresa, datos, gráficas
- ✅ Endpoint de download retorna pre-signed URL

---

### 📦 US-3.2: Frontend - Solicitud de Reportes
**Prioridad:** MUST | **Estimación:** 3 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero solicitar y descargar reportes.

**Tareas:**
1. Crear página `/reportes` con botones para generar
2. Modal para configurar reporte (mes, tipo, opciones)
3. Progress indicator (polling de status cada 2s)
4. Botón de descarga cuando status = COMPLETED

**Criterios de Aceptación:**
- ✅ Botones generan reporte y muestran progress
- ✅ Al completar, aparece botón de descarga
- ✅ Descarga funcional con un clic

---

### 📦 US-3.3: Backend - Sistema de Notificaciones
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero recibir alertas de actividades próximas a vencer.

**Tareas:**
1. Crear job programado (cron) que se ejecuta diariamente
2. Identificar actividades con due_date en 7 días
3. Crear notificación in-app (tabla notifications)
4. Enviar email con Nodemailer (template HTML)
5. Crear job semanal para digest (resumen de actividades)

**Criterios de Aceptación:**
- ✅ Job corre diariamente a las 8am
- ✅ Usuarios reciben email 7 días antes de vencimiento
- ✅ Notificaciones in-app creadas en DB
- ✅ Digest semanal enviado los lunes

---

### 📦 US-3.4: Frontend - Centro de Notificaciones
**Prioridad:** SHOULD | **Estimación:** 3 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver mis notificaciones en la app.

**Tareas:**
1. Crear dropdown de notificaciones en header (icono campana)
2. Badge con número de no leídas
3. Lista de notificaciones con botón "Marcar como leída"
4. Click en notificación navega a entidad relacionada

**Criterios de Aceptación:**
- ✅ Badge muestra número correcto de no leídas
- ✅ Click en notificación marca como leída y navega
- ✅ Botón "Marcar todas como leídas" funcional

---

### 📦 US-3.5: Backend - Dashboard KPIs
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Backend

**Descripción:**
Como usuario, quiero ver métricas clave en el dashboard.

**Tareas:**
1. Implementar endpoint `GET /dashboard` con agregaciones
2. Calcular: compliance %, actividades vencidas, evidencias expiradas, etc.
3. Calcular tendencia de cumplimiento (últimos 6 meses)
4. Distribución por módulo PHVA

**Criterios de Aceptación:**
- ✅ Endpoint retorna 6 KPIs principales
- ✅ Tendencia calculada correctamente
- ✅ Distribución PHVA con actividades completadas/totales

---

### 📦 US-3.6: Frontend - Dashboard con KPIs
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver un dashboard visual con métricas clave.

**Tareas:**
1. Crear tarjetas de KPIs (6 principales) con iconos
2. Gráfica de tendencia de cumplimiento (line chart)
3. Gráfica de distribución PHVA (bar chart o donut)
4. Tabla de actividades vencidas (top 5)
5. Semáforos por estado (verde/amarillo/rojo)

**Criterios de Aceptación:**
- ✅ KPIs muestran valores en tiempo real
- ✅ Gráficas responsive y visualmente atractivas
- ✅ Tabla de vencidas clickeable (navega a detalle)
- ✅ Semáforos según umbrales (<70% rojo, 70-85% amarillo, >85% verde)

---

### 📦 US-3.7: Frontend - Calendario de Actividades
**Prioridad:** SHOULD | **Estimación:** 5 pts | **Responsable:** Frontend

**Descripción:**
Como usuario, quiero ver un calendario con actividades del mes.

**Tareas:**
1. Crear página `/calendario` con vista mensual
2. Integrar librería de calendario (react-big-calendar o similar)
3. Mostrar actividades según due_date
4. Color por estado
5. Click en actividad abre modal de detalle

**Criterios de Aceptación:**
- ✅ Calendario muestra actividades del mes actual
- ✅ Navegación anterior/siguiente mes funcional
- ✅ Actividades coloreadas por estado
- ✅ Click abre modal con detalle

---

### 📦 US-3.8: Testing End-to-End
**Prioridad:** MUST | **Estimación:** 8 pts | **Responsable:** QA/Full-Stack

**Descripción:**
Como equipo, queremos asegurar que los flujos críticos funcionen correctamente.

**Tareas:**
1. Configurar Playwright o Cypress
2. Tests E2E para flujos principales:
   - Login → Dashboard
   - Crear empresa → Crear usuario → Actividades
   - Autoevaluación completa → Plan de mejora
   - Upload evidencia → Cerrar actividad
3. Tests de integración de APIs (Postman/Newman)

**Criterios de Aceptación:**
- ✅ 10 tests E2E pasando
- ✅ Coverage >80% de flujos críticos
- ✅ Tests corriendo en CI/CD

---

### 📦 US-3.9: Optimización y Performance
**Prioridad:** SHOULD | **Estimación:** 5 pts | **Responsable:** Full-Stack

**Descripción:**
Como usuario, quiero que la aplicación cargue rápido.

**Tareas:**
1. Optimizar queries lentas (añadir índices faltantes)
2. Implementar cache Redis en endpoints más consultados
3. Lazy loading de componentes pesados (frontend)
4. Compresión de imágenes en frontend
5. Minificación y tree-shaking (webpack)

**Criterios de Aceptación:**
- ✅ P95 de response time <500ms
- ✅ Lighthouse score >90 (performance)
- ✅ Queries lentas (<100ms después de optimización)

---

### 📦 US-3.10: Documentación y Deploy
**Prioridad:** MUST | **Estimación:** 5 pts | **Responsable:** Full-Stack

**Descripción:**
Como equipo, queremos documentar la API y deployar a producción.

**Tareas:**
1. Generar documentación Swagger/OpenAPI (NestJS)
2. Crear README.md con instrucciones de instalación
3. Configurar ambientes de staging y producción
4. Deploy inicial a producción (AWS + Vercel)
5. Configurar backups automáticos de DB
6. Configurar alertas de monitoreo (Sentry, CloudWatch)

**Criterios de Aceptación:**
- ✅ API documentada en Swagger (/api/docs)
- ✅ README completo con instrucciones
- ✅ Producción funcionando sin errores críticos
- ✅ Backups diarios configurados
- ✅ Alertas de errores llegando a Slack

---

### **Sprint 3 - Total Story Points:** 63 pts

---

## 4. RESUMEN DE STORY POINTS

| Sprint | Story Points | Capacidad Estimada (2 devs) |
|--------|--------------|------------------------------|
| Sprint 1 | 53 pts | 50-60 pts (OK) |
| Sprint 2 | 65 pts | 50-60 pts (ligeramente sobrecargado) |
| Sprint 3 | 63 pts | 50-60 pts (ligeramente sobrecargado) |
| **TOTAL** | **181 pts** | **150-180 pts** |

**Nota:** Sprint 2 y 3 están ligeramente por encima de capacidad. Opciones:
1. Mover US-2.9 (Vista Plan de Mejora) a Sprint 3
2. Reducir scope de US-3.7 (Calendario) a fase 2 si es necesario
3. Extender a 7 semanas si el timeline lo permite

---

## 5. RIESGOS Y MITIGACIONES

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| **Complejidad de autoevaluación 0312** | Alta | Crítico | Validar lógica con consultor SG-SST real en Sprint 2 semana 1 |
| **Performance de generación de PDFs** | Media | Alto | Implementar queue asíncrona desde el inicio (US-3.1) |
| **Curva de aprendizaje en tecnologías nuevas** | Media | Medio | Pair programming, code reviews diarios |
| **Scope creep** | Alta | Alto | Product Owner debe proteger el backlog, decir NO a features extra |
| **Bugs en producción por falta de testing** | Media | Crítico | Priorizar US-3.8 (Testing E2E) al inicio de Sprint 3 |

---

## 6. DEFINICIÓN DE MVP COMPLETADO

El MVP se considera **completo** cuando:

✅ Un consultor puede:
- Crear 5 empresas y gestionarlas
- Invitar usuarios a cada empresa
- Ver dashboard consolidado de todas sus empresas

✅ Un administrador de empresa puede:
- Ver 60 actividades precargadas del SG-SST
- Asignar responsables y fechas
- Cambiar estados de actividades
- Subir evidencias

✅ Un auditor puede:
- Ejecutar autoevaluación 0312 completa
- Obtener puntaje y % cumplimiento
- Generar plan de mejora automático con N acciones

✅ Sistema automatizado:
- Envía alertas 7 días antes de vencimientos
- Genera reportes PDF en <30s
- Registra bitácora de todas las acciones

✅ Métricas técnicas:
- Uptime >99% (staging validado)
- P95 response time <500ms
- Coverage de tests >70%
- Sin bugs críticos abiertos

---

## 7. POST-MVP (Fase 2+)

Features para siguientes iteraciones:
- ❌ Auditorías internas con hallazgos y no conformidades
- ❌ Matriz IPER (identificación de peligros y riesgos)
- ❌ Gestión de incidentes/accidentes
- ❌ Capacitaciones con LMS
- ❌ Firma electrónica de documentos
- ❌ App móvil nativa
- ❌ Integraciones con nómina/ERP
- ❌ Reportes personalizables con query builder

---

**FIN DEL BACKLOG MVP**

¡Listo para arrancar el desarrollo! 🚀
