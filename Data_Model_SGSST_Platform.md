# Modelo de Datos (ERD)
# Sistema SG-SST Multi-Tenant

**Versión:** 1.0  
**Base de Datos:** PostgreSQL 15  
**Estrategia Multi-Tenant:** Schema per tenant

---

## 1. ESTRUCTURA GENERAL

### Schemas en PostgreSQL

```
Database: sgsst_platform

├── public (tablas compartidas globales)
│   ├── users
│   ├── companies
│   ├── super_admin_settings
│   └── audit_logs_global
│
├── company_1 (schema para empresa ID 1)
│   ├── activities
│   ├── evidences
│   ├── assessments
│   ├── assessment_items
│   ├── improvement_plans
│   ├── improvement_actions
│   ├── audits (fase 2)
│   ├── findings (fase 2)
│   └── audit_logs_company
│
├── company_2 (schema para empresa ID 2)
│   └── [mismas tablas que company_1]
│
└── company_N
    └── [...]
```

---

## 2. TABLAS EN SCHEMA PUBLIC (Compartidas)

### 2.1 Tabla: users

**Descripción:** Todos los usuarios de la plataforma (consultores y usuarios de empresas)

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único del usuario |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email (username) |
| password_hash | VARCHAR(255) | NOT NULL | Hash bcrypt |
| first_name | VARCHAR(100) | NOT NULL | Nombre |
| last_name | VARCHAR(100) | NOT NULL | Apellido |
| role | ENUM | NOT NULL | SUPER_ADMIN, COMPANY_ADMIN, AUDITOR, AREA_LEADER, EMPLOYEE, CLIENT_READONLY |
| company_id | UUID | FK → companies.id, NULL for SUPER_ADMIN | Empresa a la que pertenece |
| is_active | BOOLEAN | DEFAULT true | Usuario activo |
| last_login | TIMESTAMP | NULL | Última sesión |
| created_at | TIMESTAMP | DEFAULT now() | Fecha creación |
| updated_at | TIMESTAMP | DEFAULT now() | Fecha última actualización |

**Índices:**
- `idx_users_email` (email)
- `idx_users_company_id` (company_id)

**Relaciones:**
- users.company_id → companies.id (muchos a uno)

---

### 2.2 Tabla: companies

**Descripción:** Empresas/clientes en la plataforma

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la empresa |
| legal_name | VARCHAR(255) | NOT NULL | Razón social |
| trade_name | VARCHAR(255) | NULL | Nombre comercial |
| nit | VARCHAR(20) | UNIQUE, NOT NULL | NIT Colombia (sin DV) |
| size | ENUM | NOT NULL | MICRO, SMALL, MEDIUM, LARGE (<10, 10-50, 51-200, >200 trabajadores) |
| risk_level | ENUM | NOT NULL | I, II, III, IV, V (según clase de riesgo ARL) |
| economic_activity | VARCHAR(10) | NOT NULL | Código CIIU (Ej: "4711" para comercio al por menor) |
| num_workers | INTEGER | NOT NULL | Número de trabajadores |
| logo_url | TEXT | NULL | URL del logo en S3 |
| address | TEXT | NULL | Dirección |
| city | VARCHAR(100) | NULL | Ciudad |
| phone | VARCHAR(20) | NULL | Teléfono |
| schema_name | VARCHAR(63) | UNIQUE, NOT NULL | Nombre del schema en DB (company_1, company_2, ...) |
| is_active | BOOLEAN | DEFAULT true | Empresa activa |
| subscription_start | DATE | NOT NULL | Inicio de suscripción |
| subscription_end | DATE | NULL | Fin de suscripción (NULL = activa) |
| created_by | UUID | FK → users.id | Super Admin que creó la empresa |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_companies_nit` (nit)
- `idx_companies_schema_name` (schema_name)

**Relaciones:**
- companies.created_by → users.id
- users.company_id → companies.id (inversa)

---

### 2.3 Tabla: audit_logs_global

**Descripción:** Bitácora de acciones a nivel plataforma (login, creación de empresas, etc.)

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | BIGSERIAL | PK | ID autoincremental |
| user_id | UUID | FK → users.id, NULL si acción del sistema | Usuario que ejecutó la acción |
| action | VARCHAR(50) | NOT NULL | LOGIN, LOGOUT, CREATE_COMPANY, DELETE_COMPANY, etc. |
| entity_type | VARCHAR(50) | NULL | Company, User, etc. |
| entity_id | UUID | NULL | ID de la entidad afectada |
| metadata | JSONB | NULL | Detalles adicionales (IP, user agent, cambios) |
| timestamp | TIMESTAMP | DEFAULT now() | Fecha/hora de la acción |

**Índices:**
- `idx_audit_logs_global_user_id` (user_id)
- `idx_audit_logs_global_timestamp` (timestamp)
- `idx_audit_logs_global_action` (action)

---

### 2.4 Tabla: super_admin_settings

**Descripción:** Configuración global de la plataforma

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único |
| key | VARCHAR(100) | UNIQUE, NOT NULL | Ej: "smtp_host", "default_logo_url" |
| value | TEXT | NOT NULL | Valor de la configuración |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

---

## 3. TABLAS EN SCHEMA COMPANY_X (Por Empresa)

### 3.1 Tabla: activities

**Descripción:** Actividades del SG-SST organizadas por módulo PHVA

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la actividad |
| phva_module | ENUM | NOT NULL | PLAN, DO, CHECK, ACT (módulo del ciclo PHVA) |
| code | VARCHAR(20) | UNIQUE | Código único (Ej: "P-01", "H-03") |
| name | VARCHAR(255) | NOT NULL | Nombre de la actividad |
| description | TEXT | NULL | Descripción detallada |
| legal_requirement | TEXT | NULL | Requisito legal asociado (Ej: "Res. 0312/2019, Estándar 1.1.1") |
| responsible_id | UUID | FK → public.users.id, NULL | Usuario responsable |
| backup_responsible_id | UUID | FK → public.users.id, NULL | Responsable suplente |
| periodicity | ENUM | NOT NULL | MONTHLY, QUARTERLY, BIANNUAL, ANNUAL, EVENT_BASED |
| status | ENUM | NOT NULL | NOT_STARTED, IN_PROGRESS, IN_REVIEW, CLOSED, NOT_APPLICABLE |
| start_date | DATE | NULL | Fecha de inicio |
| due_date | DATE | NULL | Fecha de vencimiento |
| completed_date | DATE | NULL | Fecha de cierre real |
| requires_mandatory_evidence | BOOLEAN | DEFAULT false | ¿Requiere evidencia obligatoria? |
| evidence_type | VARCHAR(100) | NULL | Tipo de evidencia requerida (Ej: "Acta firmada", "Certificado") |
| priority | ENUM | DEFAULT 'MEDIUM' | LOW, MEDIUM, HIGH, CRITICAL |
| tags | TEXT[] | DEFAULT '{}' | Etiquetas para filtrado (Ej: ["capacitación", "comité"]) |
| parent_activity_id | UUID | FK → activities.id, NULL | Actividad padre (si es sub-actividad) |
| created_by | UUID | FK → public.users.id | Usuario que creó |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_activities_phva_module` (phva_module)
- `idx_activities_status` (status)
- `idx_activities_responsible_id` (responsible_id)
- `idx_activities_due_date` (due_date)

**Relaciones:**
- activities.responsible_id → public.users.id
- activities.parent_activity_id → activities.id (auto-referencia)

---

### 3.2 Tabla: evidences

**Descripción:** Evidencias (archivos, links) asociadas a actividades o generales

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la evidencia |
| activity_id | UUID | FK → activities.id, NULL si es general | Actividad asociada |
| filename | VARCHAR(255) | NOT NULL | Nombre original del archivo |
| file_size | BIGINT | NOT NULL | Tamaño en bytes |
| mime_type | VARCHAR(100) | NOT NULL | Tipo MIME (application/pdf, image/jpeg) |
| s3_key | TEXT | UNIQUE, NOT NULL | Ruta en S3 (empresa-{id}/evidencias/{activity-id}/{timestamp}-{filename}) |
| s3_url | TEXT | NOT NULL | URL firmada (pre-signed) temporal |
| file_type | ENUM | NOT NULL | DOCUMENT, IMAGE, LINK, OTHER |
| evidence_category | VARCHAR(100) | NULL | Categoría (Acta, Certificado, Inspección, Reporte, etc.) |
| description | TEXT | NULL | Descripción del documento |
| tags | TEXT[] | DEFAULT '{}' | Etiquetas |
| version | INTEGER | DEFAULT 1 | Versión del documento (si se reemplaza) |
| previous_version_id | UUID | FK → evidences.id, NULL | ID de la versión anterior |
| expiration_date | DATE | NULL | Fecha de vencimiento (para certificados, licencias) |
| is_expired | BOOLEAN | DEFAULT false | Computed field (expiration_date < today) |
| uploaded_by | UUID | FK → public.users.id | Usuario que subió |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_evidences_activity_id` (activity_id)
- `idx_evidences_s3_key` (s3_key)
- `idx_evidences_expiration_date` (expiration_date) para alertas
- `idx_evidences_tags` (tags) GIN index para búsqueda por tags

**Relaciones:**
- evidences.activity_id → activities.id
- evidences.previous_version_id → evidences.id (auto-referencia)

---

### 3.3 Tabla: assessments

**Descripción:** Autoevaluaciones según Resolución 0312/2019

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la autoevaluación |
| name | VARCHAR(255) | NOT NULL | Nombre (Ej: "Autoevaluación Q1 2026") |
| assessment_date | DATE | NOT NULL | Fecha de la evaluación |
| total_score | DECIMAL(5,2) | NULL | Puntaje total obtenido (0-100) |
| expected_score | DECIMAL(5,2) | NULL | Puntaje esperado según parametrización |
| compliance_percentage | DECIMAL(5,2) | NULL | % de cumplimiento |
| status | ENUM | NOT NULL | DRAFT, IN_PROGRESS, COMPLETED | | assessment_type | VARCHAR(50) | DEFAULT '0312' | Tipo de evaluación (0312, auditoría interna, etc.) |
| evaluated_by | UUID | FK → public.users.id | Usuario evaluador |
| approved_by | UUID | FK → public.users.id, NULL | Usuario que aprobó |
| notes | TEXT | NULL | Observaciones generales |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_assessments_status` (status)
- `idx_assessments_assessment_date` (assessment_date)

---

### 3.4 Tabla: assessment_items

**Descripción:** Ítems individuales de la autoevaluación 0312 (checklist)

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único del ítem |
| assessment_id | UUID | FK → assessments.id | Autoevaluación a la que pertenece |
| standard_number | INTEGER | NOT NULL | Número del estándar (1-21) |
| standard_name | VARCHAR(255) | NOT NULL | Nombre del estándar (Ej: "Recursos") |
| item_code | VARCHAR(20) | NOT NULL | Código del ítem (Ej: "1.1.1") |
| item_description | TEXT | NOT NULL | Descripción del criterio |
| weight | DECIMAL(5,2) | NOT NULL | Peso del ítem en el puntaje total |
| response | ENUM | NULL | YES, NO, PARTIAL, NOT_APPLICABLE |
| score | DECIMAL(5,2) | NULL | Puntaje obtenido en este ítem |
| evidence_id | UUID | FK → evidences.id, NULL | Evidencia asociada |
| comments | TEXT | NULL | Comentarios del evaluador |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_assessment_items_assessment_id` (assessment_id)
- `idx_assessment_items_standard_number` (standard_number)

**Relaciones:**
- assessment_items.assessment_id → assessments.id
- assessment_items.evidence_id → evidences.id

---

### 3.5 Tabla: improvement_plans

**Descripción:** Planes de mejoramiento generados desde autoevaluaciones o auditorías

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único del plan |
| name | VARCHAR(255) | NOT NULL | Nombre del plan (Ej: "Plan de Mejora Q1 2026") |
| source_type | ENUM | NOT NULL | ASSESSMENT, AUDIT, MANUAL (origen del plan) |
| source_id | UUID | NULL | ID de la autoevaluación o auditoría origen |
| status | ENUM | NOT NULL | ACTIVE, IN_PROGRESS, COMPLETED, CANCELLED |
| start_date | DATE | NOT NULL | Fecha de inicio |
| target_completion_date | DATE | NOT NULL | Fecha objetivo de finalización |
| actual_completion_date | DATE | NULL | Fecha real de cierre |
| responsible_id | UUID | FK → public.users.id | Responsable general del plan |
| created_by | UUID | FK → public.users.id | |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_improvement_plans_status` (status)
- `idx_improvement_plans_responsible_id` (responsible_id)

---

### 3.6 Tabla: improvement_actions

**Descripción:** Acciones correctivas/preventivas dentro de un plan de mejora

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la acción |
| plan_id | UUID | FK → improvement_plans.id | Plan al que pertenece |
| action_number | INTEGER | NOT NULL | Número secuencial dentro del plan |
| description | TEXT | NOT NULL | Descripción de la acción |
| action_type | ENUM | NOT NULL | CORRECTIVE, PREVENTIVE, IMPROVEMENT |
| gap_identified | TEXT | NULL | Brecha identificada (de la autoevaluación) |
| root_cause | TEXT | NULL | Análisis de causa raíz |
| root_cause_method | VARCHAR(50) | NULL | Método usado (5_WHY, ISHIKAWA, etc.) |
| responsible_id | UUID | FK → public.users.id | Responsable de la acción |
| start_date | DATE | NOT NULL | Fecha de inicio |
| due_date | DATE | NOT NULL | Fecha compromiso |
| completed_date | DATE | NULL | Fecha real de cierre |
| status | ENUM | NOT NULL | PENDING, IN_PROGRESS, COMPLETED, OVERDUE, CANCELLED |
| progress_percentage | INTEGER | DEFAULT 0 | % de avance (0-100) |
| effectiveness_verified | BOOLEAN | DEFAULT false | ¿Se verificó eficacia? |
| effectiveness_notes | TEXT | NULL | Notas sobre verificación de eficacia |
| evidence_id | UUID | FK → evidences.id, NULL | Evidencia de cierre |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_improvement_actions_plan_id` (plan_id)
- `idx_improvement_actions_status` (status)
- `idx_improvement_actions_responsible_id` (responsible_id)
- `idx_improvement_actions_due_date` (due_date)

**Relaciones:**
- improvement_actions.plan_id → improvement_plans.id
- improvement_actions.evidence_id → evidences.id

---

### 3.7 Tabla: sites (Sedes/Ubicaciones)

**Descripción:** Sedes o ubicaciones físicas de la empresa

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la sede |
| name | VARCHAR(255) | NOT NULL | Nombre de la sede (Ej: "Planta Bogotá", "Oficina Medellín") |
| address | TEXT | NOT NULL | Dirección completa |
| city | VARCHAR(100) | NOT NULL | Ciudad |
| department | VARCHAR(100) | NOT NULL | Departamento |
| is_main | BOOLEAN | DEFAULT false | ¿Es la sede principal? |
| num_workers | INTEGER | DEFAULT 0 | Trabajadores en esta sede |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_sites_city` (city)

---

### 3.8 Tabla: processes

**Descripción:** Procesos organizacionales (para estructura del SG-SST)

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único del proceso |
| name | VARCHAR(255) | NOT NULL | Nombre del proceso (Ej: "Producción", "Ventas", "Recursos Humanos") |
| description | TEXT | NULL | Descripción |
| responsible_id | UUID | FK → public.users.id, NULL | Líder del proceso |
| site_id | UUID | FK → sites.id, NULL | Sede principal del proceso |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

---

### 3.9 Tabla: notifications

**Descripción:** Notificaciones in-app para usuarios

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la notificación |
| user_id | UUID | FK → public.users.id | Usuario destinatario |
| type | VARCHAR(50) | NOT NULL | ACTIVITY_DUE, EVIDENCE_EXPIRED, FINDING_ASSIGNED, etc. |
| title | VARCHAR(255) | NOT NULL | Título de la notificación |
| message | TEXT | NOT NULL | Mensaje |
| related_entity_type | VARCHAR(50) | NULL | Activity, Evidence, etc. |
| related_entity_id | UUID | NULL | ID de la entidad relacionada |
| is_read | BOOLEAN | DEFAULT false | ¿Leída? |
| created_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_notifications_user_id` (user_id)
- `idx_notifications_is_read` (is_read)
- `idx_notifications_created_at` (created_at)

---

### 3.10 Tabla: audit_logs_company

**Descripción:** Bitácora de acciones dentro de la empresa (cambios en actividades, evidencias, etc.)

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | BIGSERIAL | PK | ID autoincremental |
| user_id | UUID | FK → public.users.id | Usuario que ejecutó la acción |
| action | VARCHAR(50) | NOT NULL | CREATE, UPDATE, DELETE, STATUS_CHANGE, etc. |
| entity_type | VARCHAR(50) | NOT NULL | Activity, Evidence, Assessment, etc. |
| entity_id | UUID | NOT NULL | ID de la entidad afectada |
| changes | JSONB | NULL | Objeto con campos modificados (before/after) |
| ip_address | INET | NULL | IP del usuario |
| user_agent | TEXT | NULL | User agent del navegador |
| timestamp | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_audit_logs_company_user_id` (user_id)
- `idx_audit_logs_company_entity` (entity_type, entity_id)
- `idx_audit_logs_company_timestamp` (timestamp)

---

## 4. TABLAS FASE 2 (Auditorías Internas)

### 4.1 Tabla: audits

**Descripción:** Auditorías internas programadas

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único de la auditoría |
| name | VARCHAR(255) | NOT NULL | Nombre (Ej: "Auditoría Interna Q2 2026") |
| audit_date | DATE | NOT NULL | Fecha de la auditoría |
| scope | TEXT | NOT NULL | Alcance (procesos, sedes auditadas) |
| auditors | UUID[] | NOT NULL | IDs de usuarios auditores |
| status | ENUM | NOT NULL | PLANNED, IN_PROGRESS, COMPLETED, CANCELLED |
| report_generated | BOOLEAN | DEFAULT false | ¿Se generó informe? |
| report_url | TEXT | NULL | URL del informe en S3 |
| created_by | UUID | FK → public.users.id | |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

---

### 4.2 Tabla: findings

**Descripción:** Hallazgos de auditorías internas

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| id | UUID | PK | ID único del hallazgo |
| audit_id | UUID | FK → audits.id | Auditoría que generó el hallazgo |
| finding_number | VARCHAR(20) | NOT NULL | Número del hallazgo (Ej: "H-2026-001") |
| type | ENUM | NOT NULL | OBSERVATION, MINOR_NC, MAJOR_NC (No Conformidad) |
| description | TEXT | NOT NULL | Descripción del hallazgo |
| requirement | TEXT | NOT NULL | Requisito incumplido |
| area_affected | VARCHAR(255) | NULL | Área o proceso afectado |
| responsible_id | UUID | FK → public.users.id | Responsable de atender el hallazgo |
| status | ENUM | NOT NULL | OPEN, IN_PROGRESS, CLOSED, VERIFIED |
| corrective_action_id | UUID | FK → improvement_actions.id, NULL | Acción correctiva asociada |
| evidence_id | UUID | FK → evidences.id, NULL | Evidencia del hallazgo |
| created_at | TIMESTAMP | DEFAULT now() | |
| updated_at | TIMESTAMP | DEFAULT now() | |

**Índices:**
- `idx_findings_audit_id` (audit_id)
- `idx_findings_status` (status)
- `idx_findings_responsible_id` (responsible_id)

---

## 5. ENUMERADOS (ENUMS)

### Role
```sql
CREATE TYPE user_role AS ENUM (
  'SUPER_ADMIN',
  'COMPANY_ADMIN',
  'AUDITOR',
  'AREA_LEADER',
  'EMPLOYEE',
  'CLIENT_READONLY'
);
```

### CompanySize
```sql
CREATE TYPE company_size AS ENUM ('MICRO', 'SMALL', 'MEDIUM', 'LARGE');
```

### RiskLevel
```sql
CREATE TYPE risk_level AS ENUM ('I', 'II', 'III', 'IV', 'V');
```

### PHVAModule
```sql
CREATE TYPE phva_module AS ENUM ('PLAN', 'DO', 'CHECK', 'ACT');
```

### ActivityStatus
```sql
CREATE TYPE activity_status AS ENUM (
  'NOT_STARTED',
  'IN_PROGRESS',
  'IN_REVIEW',
  'CLOSED',
  'NOT_APPLICABLE'
);
```

### Periodicity
```sql
CREATE TYPE periodicity AS ENUM (
  'MONTHLY',
  'QUARTERLY',
  'BIANNUAL',
  'ANNUAL',
  'EVENT_BASED'
);
```

### AssessmentResponse
```sql
CREATE TYPE assessment_response AS ENUM ('YES', 'NO', 'PARTIAL', 'NOT_APPLICABLE');
```

### ActionType
```sql
CREATE TYPE action_type AS ENUM ('CORRECTIVE', 'PREVENTIVE', 'IMPROVEMENT');
```

### ActionStatus
```sql
CREATE TYPE action_status AS ENUM (
  'PENDING',
  'IN_PROGRESS',
  'COMPLETED',
  'OVERDUE',
  'CANCELLED'
);
```

### FindingType
```sql
CREATE TYPE finding_type AS ENUM ('OBSERVATION', 'MINOR_NC', 'MAJOR_NC');
```

---

## 6. RELACIONES PRINCIPALES

### Diagrama Conceptual (ERD Textual)

```
public.users ────┬──── 1:N ────> company_X.activities (responsible)
                 │
                 └──── N:1 ────> public.companies

public.companies ──── 1:1 ────> PostgreSQL Schema (company_X)

company_X.activities ──── 1:N ────> company_X.evidences
                     └──── 1:1 ────> company_X.activities (parent_activity)

company_X.assessments ──── 1:N ────> company_X.assessment_items
                      └──── 1:N ────> company_X.improvement_plans (source)

company_X.improvement_plans ──── 1:N ────> company_X.improvement_actions

company_X.improvement_actions ──── N:1 ────> company_X.evidences

company_X.audits ──── 1:N ────> company_X.findings
                 └──── 1:1 ────> company_X.evidences (informe)

company_X.findings ──── 1:1 ────> company_X.improvement_actions (corrective_action)
```

---

## 7. ÍNDICES DE PERFORMANCE

### Búsquedas Frecuentes

1. **Actividades por vencer (alertas):**
```sql
CREATE INDEX idx_activities_due_date_status 
ON company_X.activities(due_date, status) 
WHERE status IN ('NOT_STARTED', 'IN_PROGRESS');
```

2. **Evidencias próximas a expirar:**
```sql
CREATE INDEX idx_evidences_expiring 
ON company_X.evidences(expiration_date) 
WHERE expiration_date IS NOT NULL AND is_expired = false;
```

3. **Full-Text Search en evidencias:**
```sql
CREATE INDEX idx_evidences_fulltext 
ON company_X.evidences 
USING GIN(to_tsvector('spanish', filename || ' ' || COALESCE(description, '')));
```

4. **Acciones de mejora vencidas:**
```sql
CREATE INDEX idx_improvement_actions_overdue 
ON company_X.improvement_actions(due_date, status) 
WHERE status IN ('PENDING', 'IN_PROGRESS');
```

---

## 8. TRIGGERS AUTOMÁTICOS

### 8.1 Actualizar updated_at
```sql
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Aplicar a todas las tablas
CREATE TRIGGER update_activities_timestamp
BEFORE UPDATE ON company_X.activities
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- (Repetir para otras tablas)
```

### 8.2 Crear entrada en audit_logs automáticamente
```sql
CREATE OR REPLACE FUNCTION log_activity_change()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO company_X.audit_logs_company (
    user_id, action, entity_type, entity_id, changes
  ) VALUES (
    current_setting('app.current_user_id')::UUID,
    TG_OP,
    'Activity',
    NEW.id,
    jsonb_build_object('before', to_jsonb(OLD), 'after', to_jsonb(NEW))
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_activity_changes
AFTER UPDATE OR DELETE ON company_X.activities
FOR EACH ROW EXECUTE FUNCTION log_activity_change();
```

### 8.3 Calcular compliance_percentage en assessments
```sql
CREATE OR REPLACE FUNCTION calculate_assessment_compliance()
RETURNS TRIGGER AS $$
DECLARE
  total_score DECIMAL;
  expected_score DECIMAL;
BEGIN
  -- Suma de puntajes obtenidos
  SELECT COALESCE(SUM(score), 0) INTO total_score
  FROM company_X.assessment_items
  WHERE assessment_id = NEW.id;

  -- Suma de puntajes esperados (weight)
  SELECT SUM(weight) INTO expected_score
  FROM company_X.assessment_items
  WHERE assessment_id = NEW.id;

  -- Actualizar assessment
  UPDATE company_X.assessments
  SET 
    total_score = total_score,
    expected_score = expected_score,
    compliance_percentage = (total_score / expected_score) * 100
  WHERE id = NEW.id;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_assessment_score
AFTER INSERT OR UPDATE ON company_X.assessment_items
FOR EACH ROW EXECUTE FUNCTION calculate_assessment_compliance();
```

---

## 9. QUERIES CLAVE DE EJEMPLO

### 9.1 Actividades vencidas de un usuario
```sql
SELECT 
  a.id,
  a.name,
  a.due_date,
  a.status,
  u.first_name || ' ' || u.last_name AS responsible_name
FROM company_X.activities a
JOIN public.users u ON a.responsible_id = u.id
WHERE 
  a.responsible_id = $1
  AND a.status IN ('NOT_STARTED', 'IN_PROGRESS')
  AND a.due_date < CURRENT_DATE
ORDER BY a.due_date ASC;
```

### 9.2 Dashboard de cumplimiento (KPIs)
```sql
WITH stats AS (
  SELECT 
    COUNT(*) FILTER (WHERE status = 'CLOSED') AS completed,
    COUNT(*) FILTER (WHERE status IN ('NOT_STARTED', 'IN_PROGRESS') AND due_date < CURRENT_DATE) AS overdue,
    COUNT(*) FILTER (WHERE status IN ('NOT_STARTED', 'IN_PROGRESS') AND due_date >= CURRENT_DATE) AS pending,
    COUNT(*) AS total
  FROM company_X.activities
  WHERE status != 'NOT_APPLICABLE'
)
SELECT 
  completed,
  overdue,
  pending,
  total,
  ROUND((completed::DECIMAL / NULLIF(total, 0)) * 100, 2) AS compliance_percentage
FROM stats;
```

### 9.3 Evidencias próximas a expirar (30 días)
```sql
SELECT 
  e.id,
  e.filename,
  e.expiration_date,
  a.name AS activity_name,
  CURRENT_DATE - e.expiration_date AS days_until_expiry
FROM company_X.evidences e
LEFT JOIN company_X.activities a ON e.activity_id = a.id
WHERE 
  e.expiration_date IS NOT NULL
  AND e.expiration_date BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '30 days'
  AND e.is_expired = false
ORDER BY e.expiration_date ASC;
```

### 9.4 Plan de mejora con progreso
```sql
SELECT 
  p.id,
  p.name,
  COUNT(a.id) AS total_actions,
  COUNT(a.id) FILTER (WHERE a.status = 'COMPLETED') AS completed_actions,
  ROUND(
    (COUNT(a.id) FILTER (WHERE a.status = 'COMPLETED')::DECIMAL / NULLIF(COUNT(a.id), 0)) * 100, 
    2
  ) AS progress_percentage
FROM company_X.improvement_plans p
LEFT JOIN company_X.improvement_actions a ON p.id = a.plan_id
WHERE p.status IN ('ACTIVE', 'IN_PROGRESS')
GROUP BY p.id, p.name;
```

---

## 10. MIGRACIONES Y SEED DATA

### Script de Creación de Schema por Empresa

```sql
-- Función para crear schema automáticamente
CREATE OR REPLACE FUNCTION create_company_schema(company_id UUID, schema_name TEXT)
RETURNS VOID AS $$
BEGIN
  -- Crear schema
  EXECUTE format('CREATE SCHEMA %I', schema_name);

  -- Crear todas las tablas en el nuevo schema
  EXECUTE format('
    CREATE TABLE %I.activities (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      phva_module phva_module NOT NULL,
      code VARCHAR(20) UNIQUE,
      name VARCHAR(255) NOT NULL,
      description TEXT,
      legal_requirement TEXT,
      responsible_id UUID,
      backup_responsible_id UUID,
      periodicity periodicity NOT NULL,
      status activity_status NOT NULL DEFAULT ''NOT_STARTED'',
      start_date DATE,
      due_date DATE,
      completed_date DATE,
      requires_mandatory_evidence BOOLEAN DEFAULT false,
      evidence_type VARCHAR(100),
      priority VARCHAR(20) DEFAULT ''MEDIUM'',
      tags TEXT[] DEFAULT ''{}'',
      parent_activity_id UUID REFERENCES %I.activities(id),
      created_by UUID,
      created_at TIMESTAMP DEFAULT NOW(),
      updated_at TIMESTAMP DEFAULT NOW()
    );
  ', schema_name, schema_name);

  -- (Crear resto de tablas: evidences, assessments, etc.)
  
  -- Seed de actividades precargadas según parametrización
  -- (Esto se hace desde el backend al crear empresa)
  
END;
$$ LANGUAGE plpgsql;
```

### Seed de Actividades PHVA Base

Cuando se crea una empresa, se cargan actividades pre-definidas según:
- Tamaño de empresa
- Nivel de riesgo
- Actividad económica

**Ejemplo:** Empresa Mediana, Riesgo III → Se cargan ~60 actividades estándar:

```json
[
  {
    "phva_module": "PLAN",
    "code": "P-01",
    "name": "Política de SST",
    "description": "Elaborar y publicar política de SST firmada por alta dirección",
    "legal_requirement": "Res. 0312/2019, Estándar 1.1.1",
    "periodicity": "ANNUAL",
    "requires_mandatory_evidence": true,
    "evidence_type": "Documento firmado"
  },
  {
    "phva_module": "PLAN",
    "code": "P-02",
    "name": "Objetivos de SST",
    "description": "Definir objetivos medibles de SST",
    "legal_requirement": "Res. 0312/2019, Estándar 1.1.2",
    "periodicity": "ANNUAL"
  },
  // ... (continuar con ~58 actividades más)
]
```

---

## 11. MANTENIMIENTO Y OPTIMIZACIÓN

### Vacuum y Analyze Automático
```sql
-- PostgreSQL tiene autovacuum habilitado por defecto
-- Configurar para tablas grandes:
ALTER TABLE company_X.activities SET (autovacuum_vacuum_scale_factor = 0.05);
ALTER TABLE company_X.evidences SET (autovacuum_vacuum_scale_factor = 0.05);
```

### Particionado (Futuro, si >1000 empresas)
```sql
-- Considerar particionar audit_logs por timestamp (mensual)
CREATE TABLE company_X.audit_logs_company_2026_01 PARTITION OF company_X.audit_logs_company
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

---

**FIN DEL MODELO DE DATOS**

---

## PRÓXIMO ENTREGABLE
1. ✅ PRD  
2. ✅ Arquitectura Técnica  
3. ✅ Modelo de Datos (ERD)  
4. 🔄 API Design (Endpoints + Payloads)  
5. ⏳ UX/Wireframes  
6. ⏳ Backlog MVP Priorizado
