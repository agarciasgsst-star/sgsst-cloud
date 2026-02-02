# API Design
# Sistema SG-SST Multi-Tenant

**Versión:** 1.0  
**Base URL:** `https://api.sgsst-platform.com/v1`  
**Protocolo:** REST (JSON)  
**Autenticación:** JWT (Bearer Token)

---

## 1. CONVENCIONES GENERALES

### Headers Estándar
```http
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
X-Company-ID: <uuid>  # Opcional, para super admin que opera múltiples empresas
```

### Códigos de Respuesta
- **200 OK:** Operación exitosa
- **201 Created:** Recurso creado
- **204 No Content:** Operación exitosa sin respuesta (DELETE)
- **400 Bad Request:** Datos inválidos
- **401 Unauthorized:** Token inválido o expirado
- **403 Forbidden:** Sin permisos para el recurso
- **404 Not Found:** Recurso no encontrado
- **409 Conflict:** Conflicto (ej: email duplicado)
- **422 Unprocessable Entity:** Validación de negocio fallida
- **429 Too Many Requests:** Rate limit excedido
- **500 Internal Server Error:** Error del servidor

### Paginación
```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "perPage": 20,
    "total": 150,
    "totalPages": 8
  },
  "links": {
    "first": "/activities?page=1",
    "prev": null,
    "next": "/activities?page=2",
    "last": "/activities?page=8"
  }
}
```

### Formato de Errores
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Los datos proporcionados son inválidos",
    "details": [
      {
        "field": "email",
        "message": "El email ya está registrado"
      }
    ]
  }
}
```

---

## 2. AUTENTICACIÓN

### 2.1 POST /auth/login
**Descripción:** Iniciar sesión

**Request:**
```json
{
  "email": "admin@empresa.com",
  "password": "SecurePass123!"
}
```

**Response 200:**
```json
{
  "user": {
    "id": "uuid-user-123",
    "email": "admin@empresa.com",
    "firstName": "Juan",
    "lastName": "Pérez",
    "role": "COMPANY_ADMIN",
    "companyId": "uuid-company-456"
  },
  "tokens": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 900  // 15 minutos
  }
}
```

---

### 2.2 POST /auth/refresh
**Descripción:** Renovar access token usando refresh token

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response 200:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 900
}
```

---

### 2.3 POST /auth/logout
**Descripción:** Cerrar sesión (invalida refresh token)

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response 204:** (No Content)

---

### 2.4 GET /auth/me
**Descripción:** Obtener información del usuario actual

**Response 200:**
```json
{
  "id": "uuid-user-123",
  "email": "admin@empresa.com",
  "firstName": "Juan",
  "lastName": "Pérez",
  "role": "COMPANY_ADMIN",
  "company": {
    "id": "uuid-company-456",
    "legalName": "Empresa Demo S.A.S.",
    "tradeName": "Demo Corp",
    "size": "MEDIUM",
    "riskLevel": "III"
  },
  "permissions": ["activities:read", "activities:write", "users:manage", ...]
}
```

---

## 3. COMPANIES (Empresas)

### 3.1 GET /companies
**Descripción:** Listar empresas (solo para SUPER_ADMIN)

**Query Params:**
- `page` (default: 1)
- `perPage` (default: 20, max: 100)
- `search` (búsqueda por nombre o NIT)
- `isActive` (true/false)

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-company-1",
      "legalName": "Empresa ABC S.A.S.",
      "tradeName": "ABC Corp",
      "nit": "900123456",
      "size": "MEDIUM",
      "riskLevel": "III",
      "numWorkers": 120,
      "isActive": true,
      "subscriptionStart": "2026-01-01",
      "createdAt": "2026-01-01T10:00:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 20, "total": 15, "totalPages": 1 }
}
```

---

### 3.2 POST /companies
**Descripción:** Crear nueva empresa (SUPER_ADMIN)

**Request:**
```json
{
  "legalName": "Nueva Empresa S.A.S.",
  "tradeName": "Nueva Corp",
  "nit": "900987654",
  "size": "SMALL",  // MICRO, SMALL, MEDIUM, LARGE
  "riskLevel": "II",  // I, II, III, IV, V
  "economicActivity": "4711",  // Código CIIU
  "numWorkers": 25,
  "address": "Calle 123 #45-67",
  "city": "Bogotá",
  "phone": "+57 1 234 5678"
}
```

**Response 201:**
```json
{
  "id": "uuid-company-new",
  "legalName": "Nueva Empresa S.A.S.",
  "tradeName": "Nueva Corp",
  "nit": "900987654",
  "size": "SMALL",
  "riskLevel": "II",
  "economicActivity": "4711",
  "numWorkers": 25,
  "schemaName": "company_15",
  "isActive": true,
  "subscriptionStart": "2026-02-02",
  "createdAt": "2026-02-02T14:30:00Z"
}
```

**Nota:** Al crear empresa, se ejecuta automáticamente:
1. Creación de schema en DB (`company_15`)
2. Creación de tablas base
3. Seed de actividades PHVA según parametrización
4. Creación de usuario administrador inicial (enviado por email)

---

### 3.3 GET /companies/:id
**Descripción:** Obtener detalle de empresa

**Response 200:**
```json
{
  "id": "uuid-company-1",
  "legalName": "Empresa ABC S.A.S.",
  "tradeName": "ABC Corp",
  "nit": "900123456",
  "size": "MEDIUM",
  "riskLevel": "III",
  "economicActivity": "4711",
  "numWorkers": 120,
  "logoUrl": "https://cdn.sgsst.com/logos/company-1.png",
  "address": "Carrera 7 #32-16",
  "city": "Bogotá",
  "phone": "+57 1 234 5678",
  "isActive": true,
  "subscriptionStart": "2026-01-01",
  "subscriptionEnd": null,
  "createdAt": "2026-01-01T10:00:00Z",
  "settings": {
    "notificationsEnabled": true,
    "weeklyDigest": true,
    "alertDaysBefore": 7
  }
}
```

---

### 3.4 PATCH /companies/:id
**Descripción:** Actualizar información de empresa

**Request:**
```json
{
  "tradeName": "ABC Corporation",
  "numWorkers": 135,
  "phone": "+57 1 987 6543"
}
```

**Response 200:** (misma estructura que GET /companies/:id)

---

### 3.5 PATCH /companies/:id/settings
**Descripción:** Actualizar configuración de empresa

**Request:**
```json
{
  "notificationsEnabled": true,
  "weeklyDigest": false,
  "alertDaysBefore": 14
}
```

**Response 200:**
```json
{
  "notificationsEnabled": true,
  "weeklyDigest": false,
  "alertDaysBefore": 14,
  "updatedAt": "2026-02-02T15:00:00Z"
}
```

---

## 4. USERS (Usuarios)

### 4.1 GET /companies/:companyId/users
**Descripción:** Listar usuarios de una empresa

**Query Params:**
- `page`, `perPage`
- `role` (filtrar por rol)
- `isActive`

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-user-1",
      "email": "user@empresa.com",
      "firstName": "María",
      "lastName": "González",
      "role": "AREA_LEADER",
      "isActive": true,
      "lastLogin": "2026-02-01T09:30:00Z",
      "createdAt": "2026-01-15T10:00:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 20, "total": 12, "totalPages": 1 }
}
```

---

### 4.2 POST /companies/:companyId/users
**Descripción:** Invitar nuevo usuario a la empresa

**Request:**
```json
{
  "email": "nuevo@empresa.com",
  "firstName": "Carlos",
  "lastName": "Rodríguez",
  "role": "AUDITOR"
}
```

**Response 201:**
```json
{
  "id": "uuid-user-new",
  "email": "nuevo@empresa.com",
  "firstName": "Carlos",
  "lastName": "Rodríguez",
  "role": "AUDITOR",
  "companyId": "uuid-company-1",
  "isActive": true,
  "invitationSent": true,
  "invitationExpires": "2026-02-09T14:30:00Z",
  "createdAt": "2026-02-02T14:30:00Z"
}
```

**Nota:** Se envía email con link de activación de cuenta

---

### 4.3 PATCH /companies/:companyId/users/:userId
**Descripción:** Actualizar usuario

**Request:**
```json
{
  "role": "COMPANY_ADMIN",
  "isActive": false
}
```

**Response 200:** (misma estructura que GET)

---

## 5. ACTIVITIES (Actividades del SG-SST)

### 5.1 GET /companies/:companyId/activities
**Descripción:** Listar actividades

**Query Params:**
- `page`, `perPage`
- `phvaModule` (PLAN, DO, CHECK, ACT)
- `status` (NOT_STARTED, IN_PROGRESS, IN_REVIEW, CLOSED, NOT_APPLICABLE)
- `responsibleId` (UUID del usuario)
- `dueDateFrom`, `dueDateTo` (filtro por rango de fechas)
- `priority` (LOW, MEDIUM, HIGH, CRITICAL)
- `search` (búsqueda por nombre o descripción)

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-activity-1",
      "phvaModule": "PLAN",
      "code": "P-01",
      "name": "Política de SST",
      "description": "Elaborar y publicar política de SST firmada por alta dirección",
      "legalRequirement": "Res. 0312/2019, Estándar 1.1.1",
      "responsible": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "periodicity": "ANNUAL",
      "status": "CLOSED",
      "startDate": "2026-01-05",
      "dueDate": "2026-01-31",
      "completedDate": "2026-01-28",
      "requiresMandatoryEvidence": true,
      "evidenceType": "Documento firmado",
      "evidencesCount": 2,
      "priority": "HIGH",
      "tags": ["política", "requisito-legal"],
      "createdAt": "2026-01-01T10:00:00Z",
      "updatedAt": "2026-01-28T16:45:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 20, "total": 58, "totalPages": 3 }
}
```

---

### 5.2 GET /companies/:companyId/activities/:id
**Descripción:** Obtener detalle de actividad

**Response 200:**
```json
{
  "id": "uuid-activity-1",
  "phvaModule": "PLAN",
  "code": "P-01",
  "name": "Política de SST",
  "description": "Elaborar y publicar política de SST firmada por alta dirección. Debe estar firmada por el representante legal y comunicada a todos los niveles de la organización.",
  "legalRequirement": "Resolución 0312 de 2019, Estándar 1.1.1 - Política de SST firmada y fechada",
  "responsible": {
    "id": "uuid-user-1",
    "email": "admin@empresa.com",
    "firstName": "Juan",
    "lastName": "Pérez",
    "role": "COMPANY_ADMIN"
  },
  "backupResponsible": {
    "id": "uuid-user-2",
    "firstName": "María",
    "lastName": "González"
  },
  "periodicity": "ANNUAL",
  "status": "CLOSED",
  "startDate": "2026-01-05",
  "dueDate": "2026-01-31",
  "completedDate": "2026-01-28",
  "requiresMandatoryEvidence": true,
  "evidenceType": "Documento firmado (PDF con firma del representante legal)",
  "priority": "HIGH",
  "tags": ["política", "requisito-legal", "alta-dirección"],
  "evidences": [
    {
      "id": "uuid-evidence-1",
      "filename": "Politica_SST_2026_Firmada.pdf",
      "fileSize": 524288,
      "uploadedBy": "Juan Pérez",
      "createdAt": "2026-01-28T15:30:00Z"
    }
  ],
  "parentActivity": null,
  "subActivities": [],
  "history": [
    {
      "action": "STATUS_CHANGE",
      "from": "IN_PROGRESS",
      "to": "CLOSED",
      "user": "Juan Pérez",
      "timestamp": "2026-01-28T16:45:00Z",
      "comment": "Política aprobada y publicada en cartelera"
    }
  ],
  "createdAt": "2026-01-01T10:00:00Z",
  "updatedAt": "2026-01-28T16:45:00Z"
}
```

---

### 5.3 POST /companies/:companyId/activities
**Descripción:** Crear nueva actividad

**Request:**
```json
{
  "phvaModule": "DO",
  "code": "H-15",
  "name": "Capacitación en trabajo en alturas",
  "description": "Capacitación para personal que realiza trabajo en alturas superiores a 1.5m",
  "legalRequirement": "Res. 1409 de 2012",
  "responsibleId": "uuid-user-3",
  "periodicity": "ANNUAL",
  "dueDate": "2026-03-31",
  "requiresMandatoryEvidence": true,
  "evidenceType": "Lista de asistencia + Certificados",
  "priority": "CRITICAL",
  "tags": ["capacitación", "trabajo-en-alturas"]
}
```

**Response 201:** (misma estructura que GET /:id)

---

### 5.4 PATCH /companies/:companyId/activities/:id
**Descripción:** Actualizar actividad

**Request:**
```json
{
  "dueDate": "2026-04-15",
  "priority": "HIGH",
  "description": "Capacitación para personal que realiza trabajo en alturas. Incluye práctica en campo."
}
```

**Response 200:** (misma estructura que GET /:id)

---

### 5.5 PATCH /companies/:companyId/activities/:id/status
**Descripción:** Cambiar estado de actividad

**Request:**
```json
{
  "status": "CLOSED",
  "comment": "Actividad completada. Evidencias cargadas."
}
```

**Response 200:**
```json
{
  "id": "uuid-activity-1",
  "status": "CLOSED",
  "completedDate": "2026-02-02",
  "updatedAt": "2026-02-02T15:00:00Z"
}
```

**Validaciones:**
- Si `requiresMandatoryEvidence = true` y no hay evidencias → 422 Error
- Si estado es NOT_APPLICABLE → requiere `justification` en el payload

---

### 5.6 GET /companies/:companyId/activities/:id/history
**Descripción:** Obtener bitácora de cambios de la actividad

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-log-1",
      "action": "STATUS_CHANGE",
      "entityType": "Activity",
      "entityId": "uuid-activity-1",
      "user": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "changes": {
        "before": { "status": "IN_PROGRESS" },
        "after": { "status": "CLOSED", "completedDate": "2026-01-28" }
      },
      "comment": "Política aprobada y publicada",
      "timestamp": "2026-01-28T16:45:00Z"
    }
  ]
}
```

---

## 6. EVIDENCES (Evidencias)

### 6.1 GET /companies/:companyId/evidences
**Descripción:** Listar evidencias

**Query Params:**
- `page`, `perPage`
- `activityId` (filtrar por actividad)
- `fileType` (DOCUMENT, IMAGE, LINK, OTHER)
- `category` (Acta, Certificado, Inspección, etc.)
- `search` (por filename o descripción)
- `uploadedBy` (UUID del usuario)

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-evidence-1",
      "filename": "Acta_Comite_SST_Enero_2026.pdf",
      "fileSize": 1048576,
      "mimeType": "application/pdf",
      "fileType": "DOCUMENT",
      "category": "Acta",
      "description": "Acta de reunión del comité de SST - Enero 2026",
      "tags": ["comité", "acta", "enero-2026"],
      "activity": {
        "id": "uuid-activity-5",
        "name": "Reunión mensual Comité SST"
      },
      "version": 1,
      "expirationDate": null,
      "uploadedBy": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "createdAt": "2026-01-31T10:00:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 20, "total": 45, "totalPages": 3 }
}
```

---

### 6.2 POST /companies/:companyId/evidences
**Descripción:** Subir nueva evidencia

**Request:** (multipart/form-data)
```
file: [archivo binario]
activityId: uuid-activity-1
category: Certificado
description: Certificado de capacitación en SST
tags: ["capacitación", "sst", "certificado"]
expirationDate: 2027-02-02  // Opcional
```

**Response 201:**
```json
{
  "id": "uuid-evidence-new",
  "filename": "Certificado_Capacitacion_SST.pdf",
  "fileSize": 524288,
  "mimeType": "application/pdf",
  "fileType": "DOCUMENT",
  "category": "Certificado",
  "s3Key": "company-1/evidences/activity-1/1675340000-Certificado_Capacitacion_SST.pdf",
  "s3Url": "https://sgsst-files.s3.amazonaws.com/...",  // Pre-signed URL (válida 1h)
  "activityId": "uuid-activity-1",
  "description": "Certificado de capacitación en SST",
  "tags": ["capacitación", "sst", "certificado"],
  "version": 1,
  "expirationDate": "2027-02-02",
  "uploadedBy": {
    "id": "uuid-user-1",
    "firstName": "Juan",
    "lastName": "Pérez"
  },
  "createdAt": "2026-02-02T15:30:00Z"
}
```

---

### 6.3 GET /companies/:companyId/evidences/:id/download
**Descripción:** Obtener URL de descarga (pre-signed S3 URL válida 1h)

**Response 200:**
```json
{
  "downloadUrl": "https://sgsst-files.s3.amazonaws.com/company-1/evidences/...?signature=...",
  "expiresAt": "2026-02-02T16:30:00Z"
}
```

---

### 6.4 DELETE /companies/:companyId/evidences/:id
**Descripción:** Eliminar evidencia (soft delete)

**Response 204:** (No Content)

**Nota:** No se elimina físicamente de S3, solo se marca como deleted en DB

---

### 6.5 GET /companies/:companyId/evidences/search
**Descripción:** Búsqueda full-text de evidencias

**Query Params:**
- `q` (query string)

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-evidence-3",
      "filename": "Informe_Inspeccion_Equipos_Feb_2026.pdf",
      "description": "Inspección de equipos de protección contra caídas",
      "highlights": [
        "...inspección de <strong>equipos</strong> de protección..."
      ],
      "relevanceScore": 0.85,
      "createdAt": "2026-02-01T14:00:00Z"
    }
  ]
}
```

---

## 7. ASSESSMENTS (Autoevaluación 0312)

### 7.1 GET /companies/:companyId/assessments
**Descripción:** Listar autoevaluaciones

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-assessment-1",
      "name": "Autoevaluación Q4 2025",
      "assessmentDate": "2025-12-15",
      "status": "COMPLETED",
      "totalScore": 78.5,
      "expectedScore": 100,
      "compliancePercentage": 78.5,
      "evaluatedBy": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "createdAt": "2025-12-01T10:00:00Z"
    }
  ]
}
```

---

### 7.2 POST /companies/:companyId/assessments
**Descripción:** Crear nueva autoevaluación

**Request:**
```json
{
  "name": "Autoevaluación Q1 2026",
  "assessmentDate": "2026-03-31"
}
```

**Response 201:**
```json
{
  "id": "uuid-assessment-new",
  "name": "Autoevaluación Q1 2026",
  "assessmentDate": "2026-03-31",
  "status": "DRAFT",
  "totalScore": null,
  "expectedScore": 100,
  "compliancePercentage": null,
  "items": [
    {
      "id": "uuid-item-1",
      "standardNumber": 1,
      "standardName": "Recursos",
      "itemCode": "1.1.1",
      "itemDescription": "Responsable del Sistema de Gestión de SST",
      "weight": 0.5,
      "response": null,
      "score": null
    },
    // ... (60 items aprox según parametrización)
  ],
  "createdAt": "2026-02-02T16:00:00Z"
}
```

**Nota:** Los items se generan automáticamente según parametrización de la empresa (tamaño, riesgo)

---

### 7.3 PATCH /companies/:companyId/assessments/:id/items/:itemId
**Descripción:** Responder un ítem de la autoevaluación

**Request:**
```json
{
  "response": "YES",  // YES, NO, PARTIAL, NOT_APPLICABLE
  "evidenceId": "uuid-evidence-1",  // Opcional
  "comments": "Se cuenta con responsable de SG-SST certificado"
}
```

**Response 200:**
```json
{
  "id": "uuid-item-1",
  "standardNumber": 1,
  "itemCode": "1.1.1",
  "response": "YES",
  "score": 0.5,  // Calculado automáticamente (weight si YES, 0.5*weight si PARTIAL, 0 si NO)
  "evidenceId": "uuid-evidence-1",
  "comments": "Se cuenta con responsable de SG-SST certificado",
  "updatedAt": "2026-02-02T16:15:00Z"
}
```

---

### 7.4 POST /companies/:companyId/assessments/:id/calculate
**Descripción:** Calcular puntaje final de autoevaluación

**Response 200:**
```json
{
  "id": "uuid-assessment-1",
  "totalScore": 78.5,
  "expectedScore": 100,
  "compliancePercentage": 78.5,
  "status": "COMPLETED",
  "breakdown": [
    {
      "standardNumber": 1,
      "standardName": "Recursos",
      "scoreObtained": 4.5,
      "scoreExpected": 5.0,
      "percentage": 90.0
    },
    {
      "standardNumber": 2,
      "standardName": "Gestión Integral del SG-SST",
      "scoreObtained": 3.2,
      "scoreExpected": 4.5,
      "percentage": 71.1
    }
    // ... (21 estándares)
  ],
  "gaps": [
    {
      "itemCode": "2.3.1",
      "itemDescription": "Plan de trabajo anual actualizado",
      "response": "NO",
      "scoreGap": 0.5
    }
    // ... (items con respuesta NO o PARTIAL)
  ]
}
```

---

### 7.5 POST /companies/:companyId/assessments/:id/generate-improvement-plan
**Descripción:** Generar automáticamente plan de mejora basado en brechas

**Response 201:**
```json
{
  "improvementPlan": {
    "id": "uuid-plan-new",
    "name": "Plan de Mejora - Q1 2026 (Autoevaluación)",
    "sourceType": "ASSESSMENT",
    "sourceId": "uuid-assessment-1",
    "status": "ACTIVE",
    "startDate": "2026-02-03",
    "targetCompletionDate": "2026-06-30",
    "actions": [
      {
        "id": "uuid-action-1",
        "actionNumber": 1,
        "description": "Actualizar Plan de Trabajo Anual del SG-SST",
        "actionType": "CORRECTIVE",
        "gapIdentified": "Plan de trabajo anual no está actualizado (ítem 2.3.1)",
        "responsibleId": "uuid-user-1",
        "dueDate": "2026-03-15",
        "status": "PENDING"
      }
      // ... (acciones generadas automáticamente para cada brecha)
    ]
  }
}
```

---

### 7.6 GET /companies/:companyId/assessments/:id/export
**Descripción:** Exportar reporte de autoevaluación en PDF

**Query Params:**
- `format` (pdf | excel) - default: pdf

**Response 200:**
```json
{
  "jobId": "uuid-job-123",
  "status": "QUEUED",
  "message": "El reporte se está generando. Use /reports/:jobId/status para consultar el progreso."
}
```

---

## 8. IMPROVEMENT PLANS (Planes de Mejora)

### 8.1 GET /companies/:companyId/improvement-plans
**Descripción:** Listar planes de mejora

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-plan-1",
      "name": "Plan de Mejora Q1 2026",
      "sourceType": "ASSESSMENT",
      "status": "IN_PROGRESS",
      "startDate": "2026-02-03",
      "targetCompletionDate": "2026-06-30",
      "actionsTotal": 12,
      "actionsCompleted": 3,
      "progressPercentage": 25.0,
      "responsible": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "createdAt": "2026-02-02T17:00:00Z"
    }
  ]
}
```

---

### 8.2 GET /companies/:companyId/improvement-plans/:id/actions
**Descripción:** Listar acciones de un plan de mejora

**Query Params:**
- `status` (PENDING, IN_PROGRESS, COMPLETED, OVERDUE, CANCELLED)

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-action-1",
      "actionNumber": 1,
      "description": "Actualizar Plan de Trabajo Anual del SG-SST",
      "actionType": "CORRECTIVE",
      "gapIdentified": "Plan de trabajo no actualizado",
      "rootCause": null,
      "responsible": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez"
      },
      "startDate": "2026-02-03",
      "dueDate": "2026-03-15",
      "completedDate": null,
      "status": "IN_PROGRESS",
      "progressPercentage": 40,
      "effectivenessVerified": false,
      "createdAt": "2026-02-02T17:00:00Z"
    }
  ]
}
```

---

### 8.3 PATCH /companies/:companyId/improvement-plans/:planId/actions/:actionId
**Descripción:** Actualizar acción de mejora

**Request:**
```json
{
  "status": "COMPLETED",
  "completedDate": "2026-03-10",
  "progressPercentage": 100,
  "evidenceId": "uuid-evidence-5",
  "effectivenessNotes": "Plan de trabajo actualizado y aprobado por dirección"
}
```

**Response 200:** (misma estructura que GET)

---

## 9. REPORTS (Reportes)

### 9.1 POST /companies/:companyId/reports/executive
**Descripción:** Generar informe ejecutivo mensual

**Request:**
```json
{
  "month": "2026-01",  // YYYY-MM
  "includeGraphs": true
}
```

**Response 202:** (Accepted - procesamiento asíncrono)
```json
{
  "jobId": "uuid-job-456",
  "status": "QUEUED",
  "estimatedTime": 30  // segundos
}
```

---

### 9.2 GET /companies/:companyId/reports/:jobId/status
**Descripción:** Consultar estado de generación de reporte

**Response 200:**
```json
{
  "jobId": "uuid-job-456",
  "status": "COMPLETED",  // QUEUED, PROCESSING, COMPLETED, FAILED
  "progress": 100,
  "startedAt": "2026-02-02T18:00:00Z",
  "completedAt": "2026-02-02T18:00:25Z",
  "downloadUrl": "https://sgsst-files.s3.amazonaws.com/reports/..."  // Si status = COMPLETED
}
```

---

### 9.3 POST /companies/:companyId/reports/calendar
**Descripción:** Exportar cronograma de actividades a Excel

**Request:**
```json
{
  "year": 2026,
  "includeCompleted": false
}
```

**Response 202:** (misma estructura que 9.1)

---

## 10. NOTIFICATIONS (Notificaciones)

### 10.1 GET /notifications
**Descripción:** Obtener notificaciones del usuario actual

**Query Params:**
- `isRead` (true/false)
- `page`, `perPage`

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid-notif-1",
      "type": "ACTIVITY_DUE",
      "title": "Actividad próxima a vencer",
      "message": "La actividad 'Capacitación en trabajo en alturas' vence el 15/03/2026",
      "relatedEntityType": "Activity",
      "relatedEntityId": "uuid-activity-15",
      "isRead": false,
      "createdAt": "2026-02-02T08:00:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 20, "total": 5, "totalPages": 1 },
  "unreadCount": 3
}
```

---

### 10.2 PATCH /notifications/:id/read
**Descripción:** Marcar notificación como leída

**Response 200:**
```json
{
  "id": "uuid-notif-1",
  "isRead": true,
  "updatedAt": "2026-02-02T18:30:00Z"
}
```

---

### 10.3 POST /notifications/mark-all-read
**Descripción:** Marcar todas las notificaciones como leídas

**Response 200:**
```json
{
  "markedAsRead": 5
}
```

---

## 11. DASHBOARD (Métricas)

### 11.1 GET /companies/:companyId/dashboard
**Descripción:** Obtener KPIs y métricas del dashboard

**Response 200:**
```json
{
  "complianceOverview": {
    "totalActivities": 58,
    "activitiesCompleted": 45,
    "activitiesOverdue": 3,
    "activitiesPending": 10,
    "compliancePercentage": 77.6
  },
  "evidencesStats": {
    "totalEvidences": 92,
    "evidencesExpiring30Days": 4,
    "evidencesExpired": 1
  },
  "improvementPlans": {
    "activePlans": 2,
    "totalActions": 18,
    "actionsCompleted": 7,
    "actionsOverdue": 2
  },
  "lastAssessment": {
    "id": "uuid-assessment-1",
    "name": "Autoevaluación Q4 2025",
    "compliancePercentage": 78.5,
    "assessmentDate": "2025-12-15"
  },
  "complianceTrend": [
    { "month": "2025-09", "percentage": 72.0 },
    { "month": "2025-10", "percentage": 75.0 },
    { "month": "2025-11", "percentage": 76.5 },
    { "month": "2025-12", "percentage": 78.5 },
    { "month": "2026-01", "percentage": 77.6 }
  ],
  "phvaDistribution": [
    { "module": "PLAN", "activitiesTotal": 15, "activitiesCompleted": 12 },
    { "module": "DO", "activitiesTotal": 20, "activitiesCompleted": 16 },
    { "module": "CHECK", "activitiesTotal": 18, "activitiesCompleted": 13 },
    { "module": "ACT", "activitiesTotal": 5, "activitiesCompleted": 4 }
  ]
}
```

---

### 11.2 GET /dashboard/super-admin
**Descripción:** Dashboard consolidado para SUPER_ADMIN (todas las empresas)

**Response 200:**
```json
{
  "totalCompanies": 15,
  "activeCompanies": 14,
  "companiesOverview": [
    {
      "companyId": "uuid-company-1",
      "companyName": "Empresa ABC S.A.S.",
      "compliancePercentage": 78.5,
      "activitiesOverdue": 3,
      "status": "MODERATE"  // CRITICAL, MODERATE, GOOD
    }
  ],
  "alerts": [
    {
      "companyId": "uuid-company-5",
      "companyName": "Empresa XYZ",
      "alertType": "LOW_COMPLIANCE",
      "message": "Cumplimiento bajo: 62%",
      "severity": "HIGH"
    }
  ]
}
```

---

## 12. AUDIT LOGS (Bitácora)

### 12.1 GET /companies/:companyId/audit-logs
**Descripción:** Obtener bitácora de auditoría de la empresa

**Query Params:**
- `userId` (filtrar por usuario)
- `action` (CREATE, UPDATE, DELETE, etc.)
- `entityType` (Activity, Evidence, etc.)
- `dateFrom`, `dateTo`
- `page`, `perPage`

**Response 200:**
```json
{
  "data": [
    {
      "id": 12345,
      "user": {
        "id": "uuid-user-1",
        "firstName": "Juan",
        "lastName": "Pérez",
        "email": "admin@empresa.com"
      },
      "action": "UPDATE",
      "entityType": "Activity",
      "entityId": "uuid-activity-1",
      "changes": {
        "before": { "status": "IN_PROGRESS" },
        "after": { "status": "CLOSED", "completedDate": "2026-01-28" }
      },
      "ipAddress": "192.168.1.100",
      "timestamp": "2026-01-28T16:45:00Z"
    }
  ],
  "meta": { "page": 1, "perPage": 50, "total": 523, "totalPages": 11 }
}
```

---

## 13. WEBHOOKS (Fase 2)

### 13.1 POST /companies/:companyId/webhooks
**Descripción:** Configurar webhook para eventos

**Request:**
```json
{
  "url": "https://external-app.com/webhook/sgsst",
  "events": ["activity.completed", "evidence.uploaded", "assessment.completed"],
  "secret": "webhook_secret_key_123"
}
```

**Payload enviado al webhook:**
```json
{
  "event": "activity.completed",
  "timestamp": "2026-02-02T19:00:00Z",
  "companyId": "uuid-company-1",
  "data": {
    "activityId": "uuid-activity-1",
    "activityName": "Política de SST",
    "completedBy": "uuid-user-1",
    "completedDate": "2026-01-28"
  },
  "signature": "sha256=..."  // HMAC con secret
}
```

---

**FIN DEL API DESIGN**

---

## PRÓXIMO ENTREGABLE
1. ✅ PRD  
2. ✅ Arquitectura Técnica  
3. ✅ Modelo de Datos  
4. ✅ API Design  
5. 🔄 UX/Wireframes + Backlog MVP Priorizado  
6. 🔄 Prototipo Interactivo
