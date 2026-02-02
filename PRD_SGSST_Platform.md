# Product Requirements Document (PRD)
# Sistema de Gestión SG-SST Multi-Tenant

**Versión:** 1.0  
**Fecha:** Febrero 2026  
**Producto:** Plataforma SaaS para Administración de SG-SST en Colombia  
**Modelo de Negocio:** Cobro por empresa/cliente gestionada

---

## 1. VISIÓN DEL PRODUCTO

### 1.1 Propósito
Plataforma SaaS que permite a consultores y auditores de SG-SST administrar múltiples empresas/clientes, facilitando el seguimiento operativo del Sistema de Gestión de Seguridad y Salud en el Trabajo bajo el ciclo PHVA, con trazabilidad completa, gestión de evidencias y cumplimiento normativo (Resolución 0312/2019, Decreto 1072/2015).

### 1.2 Problema a Resolver
Los clientes (PYMEs colombianas) ya tienen su SG-SST diseñado pero necesitan:
- Administración operativa diaria del sistema
- Trazabilidad de actividades y evidencias
- Seguimiento de cumplimiento normativo
- Gestión de planes de mejora
- Reportes para auditorías
- Control de vencimientos y alertas

Los consultores necesitan:
- Gestionar múltiples clientes desde una sola plataforma
- Generar valor recurrente (SaaS por empresa gestionada)
- Sistema "audit-ready" con bitácora completa

### 1.3 Propuesta de Valor
- **Para Consultores:** Plataforma multi-tenant que permite administrar N clientes, facturación recurrente, valor agregado en seguimiento operativo
- **Para Empresas:** Sistema centralizado para operación diaria del SG-SST, evidencias organizadas, alertas automáticas, reportes listos para auditoría

---

## 2. USUARIOS Y ROLES

### 2.1 Actores del Sistema

| Rol | Descripción | Permisos Clave |
|-----|-------------|----------------|
| **Super Admin** | Dueño de la plataforma (consultor) | Gestiona todas las empresas, usuarios, configuración global |
| **Admin Empresa** | Responsable SG-SST de la empresa cliente | Gestiona su empresa: usuarios, sedes, procesos, configuración |
| **Auditor Interno** | Realiza auditorías y revisiones | Acceso lectura/escritura a auditorías, no conformidades, planes de mejora |
| **Líder de Área/Proceso** | Responsable de actividades específicas | Ejecuta actividades asignadas, carga evidencias, actualiza estados |
| **Empleado/Contratista** | Usuario final | Consulta información, recibe capacitaciones, reporta incidentes (fase 2) |
| **Cliente Solo Lectura** | Gerencia/dirección | Dashboard ejecutivo, reportes, sin edición |

---

## 3. ÉPICAS Y FUNCIONALIDADES CORE

### ÉPICA 1: Gestión Multi-Tenant y Empresas
**Valor:** Permite al consultor administrar múltiples clientes desde una plataforma centralizada

**User Stories:**
- **US-1.1:** Como Super Admin, quiero crear nuevas empresas en el sistema para agregar clientes a mi cartera
- **US-1.2:** Como Super Admin, quiero parametrizar cada empresa (tamaño, riesgo, #trabajadores, actividad económica) para aplicar requisitos mínimos según Res. 0312
- **US-1.3:** Como Admin Empresa, quiero gestionar sedes y procesos de mi organización para estructurar el SG-SST
- **US-1.4:** Como Super Admin, quiero ver dashboard consolidado de todas mis empresas con métricas clave
- **US-1.5:** Como Admin Empresa, quiero invitar usuarios con diferentes roles para delegar responsabilidades

**Criterios de Aceptación MVP:**
- Creación de empresa con wizard de parametrización (5 pasos)
- Gestión de usuarios por empresa con RBAC
- Dashboard multi-empresa con tarjetas de estado (cumplimiento %, actividades vencidas)

---

### ÉPICA 2: Estructura PHVA y Actividades
**Valor:** Organiza el SG-SST según ciclo PHVA con seguimiento de actividades y responsables

**User Stories:**
- **US-2.1:** Como Admin Empresa, quiero ver la estructura PHVA (Planear-Hacer-Verificar-Actuar) con actividades precargadas según mi parametrización
- **US-2.2:** Como Líder de Área, quiero ver mis actividades asignadas con fechas de vencimiento y prioridad
- **US-2.3:** Como Líder de Área, quiero actualizar el estado de una actividad (No iniciado → En curso → En revisión → Cerrado) con comentarios
- **US-2.4:** Como Líder de Área, quiero cargar evidencias (archivos, links, fotos) en actividades específicas
- **US-2.5:** Como Admin Empresa, quiero configurar periodicidad de actividades (mensual/trimestral/anual/por evento) con alertas automáticas
- **US-2.6:** Como cualquier usuario, quiero ver la bitácora de cambios de una actividad (quién, cuándo, qué cambió)

**Criterios de Aceptación MVP:**
- Vista de módulos PHVA con navegación lateral (similar a referencia)
- Tabla de actividades filtrable por estado, responsable, módulo
- Carga de evidencias con metadatos (fecha, autor, tipo)
- Bitácora automática en toda modificación
- Alertas por email 7 días antes de vencimiento

---

### ÉPICA 3: Autoevaluación 0312 y Plan de Mejora
**Valor:** Genera automáticamente brechas de cumplimiento y plan de mejoramiento basado en Res. 0312

**User Stories:**
- **US-3.1:** Como Admin Empresa, quiero ejecutar autoevaluación 0312 con checklist de estándares mínimos según mi parametrización
- **US-3.2:** Como Admin Empresa, quiero ver puntaje obtenido vs esperado por cada estándar (1-21) con semáforo
- **US-3.3:** Como Admin Empresa, quiero generar automáticamente un plan de mejora con acciones para cerrar brechas identificadas
- **US-3.4:** Como Auditor, quiero asignar responsables y fechas compromiso a cada acción del plan de mejora
- **US-3.5:** Como Líder de Área, quiero actualizar progreso de acciones de mejora asignadas y cargar evidencias de cierre
- **US-3.6:** Como Admin Empresa, quiero exportar reporte de autoevaluación en PDF con branding de mi empresa

**Criterios de Aceptación MVP:**
- Checklist 0312 dinámico según parametrización (21 estándares, 60 ítems aprox)
- Cálculo automático de puntaje (ponderado según tamaño y riesgo)
- Identificación de brechas (criterios no cumplidos)
- Generación de plan de mejora con template de acciones
- Exportación PDF con logo empresa, fecha, responsable

---

### ÉPICA 4: Gestión Documental y Evidencias
**Valor:** Repositorio centralizado con control de acceso, versionado y trazabilidad

**User Stories:**
- **US-4.1:** Como Líder de Área, quiero subir documentos/evidencias con etiquetas (tipo: acta, certificado, inspección, etc.)
- **US-4.2:** Como Admin Empresa, quiero organizar evidencias por carpetas (por módulo PHVA, por sede, por proceso)
- **US-4.3:** Como Auditor, quiero ver historial de versiones de un documento con fecha y autor de cada cambio
- **US-4.4:** Como Admin Empresa, quiero configurar documentos con fecha de expiración y recibir alertas de renovación
- **US-4.5:** Como cualquier usuario con permisos, quiero buscar evidencias por palabra clave, fecha, responsable o actividad asociada

**Criterios de Aceptación MVP:**
- Carga de archivos con límite 25MB por archivo
- Metadatos: fecha, autor, tipo, etiquetas, actividad relacionada
- Versionado automático al reemplazar archivo con mismo nombre
- Alertas 30 días antes de expiración de documentos
- Búsqueda por texto completo en metadatos

---

### ÉPICA 5: Calendario y Plan de Trabajo Anual
**Valor:** Visibilidad de cronograma de actividades con alertas proactivas

**User Stories:**
- **US-5.1:** Como Admin Empresa, quiero ver calendario anual con todas las actividades del SG-SST distribuidas según periodicidad
- **US-5.2:** Como Líder de Área, quiero ver vista semanal/mensual de mis actividades asignadas con estado
- **US-5.3:** Como Admin Empresa, quiero recibir resumen semanal por email con actividades próximas a vencer
- **US-5.4:** Como Líder de Área, quiero marcar actividades como "No aplica" con justificación cuando corresponda
- **US-5.5:** Como Admin Empresa, quiero exportar cronograma a Excel para presentación a dirección

**Criterios de Aceptación MVP:**
- Vista calendario (mensual) con color por estado de actividad
- Filtros por responsable, módulo PHVA, sede
- Email semanal automático con resumen de vencimientos
- Exportación a Excel con formato estándar

---

### ÉPICA 6: Indicadores y Dashboard
**Valor:** Visibilidad en tiempo real del cumplimiento y salud del SG-SST

**User Stories:**
- **US-6.1:** Como Admin Empresa, quiero ver dashboard con KPIs principales (% cumplimiento global, actividades vencidas, evidencias cargadas)
- **US-6.2:** Como Super Admin, quiero ver dashboard consolidado de todas mis empresas con alertas críticas
- **US-6.3:** Como Cliente Solo Lectura, quiero ver gráficas de tendencia de cumplimiento (últimos 6 meses)
- **US-6.4:** Como Admin Empresa, quiero configurar umbrales de alerta (ej: cumplimiento <70% = crítico)
- **US-6.5:** Como Auditor, quiero ver reporte de hallazgos de auditorías internas con estados de cierre

**Criterios de Aceptación MVP:**
- Dashboard con 6 KPIs principales (cumplimiento %, actividades al día, vencidas, evidencias actualizadas, hallazgos abiertos, acciones de mejora en curso)
- Gráficas: tendencia mensual, cumplimiento por módulo PHVA, distribución por estado
- Semáforos configurables (verde >85%, amarillo 70-85%, rojo <70%)

---

### ÉPICA 7: Auditorías y No Conformidades
**Valor:** Registro formal de hallazgos y seguimiento de acciones correctivas

**User Stories:**
- **US-7.1:** Como Auditor, quiero crear una auditoría interna con fecha, alcance y auditores asignados
- **US-7.2:** Como Auditor, quiero registrar hallazgos clasificados (observación, no conformidad menor, NC mayor) con evidencias
- **US-7.3:** Como Líder de Área, quiero recibir notificación de hallazgos asignados y proponer acción correctiva
- **US-7.4:** Como Auditor, quiero validar análisis de causa raíz (5 porqués/Ishikawa) y aprobar/rechazar acciones propuestas
- **US-7.5:** Como Admin Empresa, quiero generar informe de auditoría con hallazgos, acciones y plazos en PDF

**Criterios de Aceptación MVP:**
- CRUD de auditorías con plantilla de informe
- Registro de hallazgos con clasificación y evidencias
- Flujo de aprobación: hallazgo → análisis causa raíz → acción correctiva → verificación eficacia
- Exportación de informe en PDF con formato estándar

---

### ÉPICA 8: Reportes y Exportación
**Valor:** Generación automática de reportes para stakeholders y auditorías externas

**User Stories:**
- **US-8.1:** Como Admin Empresa, quiero generar informe ejecutivo mensual con resumen de cumplimiento y estado de acciones
- **US-8.2:** Como Super Admin, quiero generar reporte consolidado de todas mis empresas para facturación
- **US-8.3:** Como Auditor, quiero generar informe de auditoría interna con plantilla personalizable
- **US-8.4:** Como Admin Empresa, quiero exportar matriz de requisitos legales actualizada a Excel
- **US-8.5:** Como Admin Empresa, quiero configurar logo y branding de reportes para mi empresa

**Criterios de Aceptación MVP:**
- 3 templates de reportes: ejecutivo mensual, autoevaluación 0312, auditoría interna
- Exportación a PDF y Excel
- Personalización de logo y datos de empresa en footer

---

## 4. REQUISITOS NO FUNCIONALES

### 4.1 Seguridad
- Autenticación JWT con refresh tokens
- RBAC (Role-Based Access Control) a nivel empresa y recurso
- Cifrado TLS 1.3 en tránsito
- Cifrado AES-256 en reposo (archivos sensibles)
- Bitácora de auditoría (audit log) de todas las acciones de usuarios
- MFA opcional para Super Admins
- Rate limiting en APIs (100 req/min por IP)
- Backups diarios automáticos con retención 30 días

### 4.2 Performance
- Tiempo de carga inicial <2s
- Respuesta API <500ms (p95)
- Soporte 100 empresas concurrentes en MVP
- Carga de archivos optimizada (streaming, compresión)
- Paginación en listas >50 elementos

### 4.3 Escalabilidad
- Arquitectura modular (microservicios light)
- Horizontal scaling en backend (stateless)
- DB particionada por tenant (schema por empresa)
- CDN para assets estáticos
- Queue para procesamiento asíncrono (reportes, emails)

### 4.4 Usabilidad
- Interfaz en español (es-CO)
- Responsive design (mobile-first para consultas)
- Accesibilidad WCAG 2.1 AA (contraste, navegación teclado)
- Onboarding guiado para nuevas empresas
- Tooltips contextuales en formularios complejos

### 4.5 Cumplimiento
- Manejo de datos personales con buenas prácticas (anonimización en logs, consentimiento explícito)
- Retención de datos configurable (90 días a 7 años)
- Exportación de datos de empresa (portabilidad)

---

## 5. ROADMAP Y FASES

### Fase 1: MVP (4-6 semanas) ✅ PRIORIDAD MÁXIMA
**Objetivo:** Plataforma operativa para gestionar 5-10 empresas piloto

**Entregables:**
1. Autenticación y gestión de usuarios/empresas
2. Estructura PHVA con actividades precargadas
3. Gestión de actividades (estados, asignación, vencimientos)
4. Carga de evidencias básica
5. Autoevaluación 0312 con generación de plan de mejora
6. Dashboard con 6 KPIs principales
7. Exportación básica (PDF de autoevaluación, Excel de cronograma)
8. Alertas por email (vencimientos, resumen semanal)

**Fuera de Scope MVP:**
- Auditorías internas (va a Fase 2)
- Análisis de causa raíz complejo
- Reportes avanzados
- Gestión de incidentes
- Matriz de riesgos

---

### Fase 2: Auditorías y Mejora Continua (2-3 semanas)
**Objetivo:** Herramientas para auditores y cierre del ciclo PHVA completo

**Entregables:**
1. Módulo de auditorías internas
2. Registro de no conformidades y hallazgos
3. Análisis de causa raíz (5 porqués/Ishikawa)
4. Seguimiento de acciones correctivas con eficacia
5. Reportes de auditoría con plantillas
6. Integración hallazgos → plan de mejora automático

---

### Fase 3: Reportes Avanzados y Analytics (2 semanas)
**Objetivo:** Generación automática de informes para stakeholders

**Entregables:**
1. Informe ejecutivo mensual automático
2. Reporte de indicadores con tendencias
3. Dashboard consolidado multi-empresa (Super Admin)
4. Exportación masiva a Excel/PDF
5. Personalización de branding por empresa

---

### Fase 4: Gestión de Riesgos e Incidentes (3-4 semanas)
**Objetivo:** Módulos complementarios para madurez del SG-SST

**Entregables:**
1. Matriz de identificación de peligros y riesgos
2. Controles asociados a riesgos
3. Registro de incidentes/accidentes
4. Investigación de incidentes con metodología
5. Indicadores de accidentalidad

---

### Fase 5: Integraciones y Automatización (Futuro)
**Objetivo:** Ecosistema conectado

**Entregables:**
1. API pública para integraciones
2. Conectores con nómina (importación de empleados)
3. Firma electrónica para documentos
4. Notificaciones push (PWA)
5. App móvil nativa (iOS/Android)

---

## 6. REGLAS DE NEGOCIO CLAVE

### 6.1 Parametrización por Empresa (Res. 0312/2019)
La autoevaluación y requisitos mínimos dependen de:

| Parámetro | Valores | Impacto |
|-----------|---------|---------|
| **Tamaño** | Micro (<10 trab), Pequeña (10-50), Mediana (51-200), Grande (>200) | Define estándares aplicables |
| **Riesgo** | I, II, III, IV, V | Pondera puntaje de cumplimiento |
| **Actividad Económica** | CIIU (División 2 dígitos) | Determina riesgos específicos |

**Ejemplo:** Empresa Mediana (100 trabajadores), Riesgo III (Comercio) → Aplican 21 estándares con 60 ítems, puntaje mínimo esperado: 86%

---

### 6.2 Estados de Actividades
Flujo de estados con transiciones controladas:

```
No Iniciado → En Curso → En Revisión → Cerrado
             ↓                      ↓
          No Aplica (requiere justificación)
```

**Reglas:**
- Solo responsable asignado puede cambiar a "En Curso"
- Para "Cerrado" se requiere evidencia obligatoria si está marcada
- "No Aplica" requiere aprobación de Admin Empresa
- Cambio de estado genera entrada en bitácora

---

### 6.3 Evidencias Obligatorias
Según tipo de actividad, algunas evidencias son mandatorias:

| Actividad | Evidencia Obligatoria |
|-----------|-----------------------|
| Capacitación | Lista de asistencia + Certificado |
| Inspección | Registro fotográfico + Formato diligenciado |
| Auditoría | Acta de auditoría + Hallazgos |
| Comité SST | Acta firmada |

**Regla:** Actividad no puede cerrarse si falta evidencia obligatoria

---

### 6.4 Alertas y Vencimientos
Sistema de notificaciones proactivo:

| Evento | Timing | Canal |
|--------|--------|-------|
| Actividad próxima a vencer | 7 días antes | Email + notificación in-app |
| Actividad vencida | Al día siguiente | Email diario con resumen |
| Documento expirado | 30 días antes | Email semanal |
| Resumen semanal | Lunes 8am | Email con actividades de la semana |
| Hallazgo asignado | Inmediato | Email + notificación in-app |

---

### 6.5 Puntaje Autoevaluación 0312
Fórmula de cálculo:

```
Puntaje Total = Σ (Puntaje Estándar × Peso según Tamaño/Riesgo)

Estándares 1-7 (Recursos): Peso 10%
Estándares 8-21 (Gestión): Peso 90%

Clasificación:
- Crítico: <60%
- Moderado: 60-85%
- Aceptable: >85%
```

**Ejemplo:**
- Empresa Pequeña, Riesgo II
- Estándar 1 (Recursos): 8/10 pts
- Estándar 8 (Política): 5/5 pts
- Estándar 9 (Objetivos): 3/5 pts
- ... (continúa con 21 estándares)
- Puntaje Final: 78% → MODERADO → Genera 15 acciones de mejora

---

### 6.6 Bitácora de Auditoría
Toda acción crítica se registra con:
- Usuario (ID + nombre)
- Timestamp (UTC-5 Colombia)
- Acción (CREATE, UPDATE, DELETE, LOGIN, EXPORT)
- Entidad afectada (Actividad, Evidencia, Usuario, etc.)
- Valores anteriores vs nuevos (JSON diff)
- IP origen

**Retención:** 2 años mínimo, configurable hasta 7 años

---

## 7. MÉTRICAS DE ÉXITO (KPIs del Producto)

### Para el Negocio (Consultor)
- **MRR (Monthly Recurring Revenue):** $X USD por empresa activa
- **Churn Rate:** <5% mensual
- **Empresas activas:** 20 empresas en 3 meses post-MVP
- **Tasa de adopción:** >80% de usuarios activos semanalmente

### Para los Clientes (Empresas)
- **Cumplimiento normativo:** >85% en autoevaluación 0312
- **Actividades al día:** >90% completadas en fecha
- **Evidencias actualizadas:** 100% de actividades cerradas con evidencia
- **Tiempo promedio de cierre de hallazgos:** <30 días

### Técnicos
- **Uptime:** >99.5%
- **Performance:** P95 <500ms en APIs
- **Errores:** <0.1% de requests fallidos
- **Satisfacción:** NPS >50

---

## 8. SUPUESTOS Y RESTRICCIONES

### Supuestos
- Clientes tienen conocimiento básico de SG-SST (no requieren asesoría en diseño del sistema)
- Infraestructura inicial soporta hasta 100 empresas concurrentes
- Consultor es responsable de configuración inicial de cada empresa (onboarding asistido)
- Internet estable (no modo offline en MVP)

### Restricciones
- Budget inicial limitado → MVP con funcionalidad core
- Equipo pequeño (1-2 devs) → Priorización estricta
- Lanzamiento en 6 semanas → Stack probado (no experimentación)
- Normatividad específica Colombia → No expansión internacional en MVP

---

## 9. OUT OF SCOPE (No va en MVP)

❌ Gestión de capacitaciones con LMS  
❌ Matriz IPER completa con metodología GTC-45  
❌ Gestión de EPPs y inventarios  
❌ Integración con nómina/ERP  
❌ App móvil nativa  
❌ Modo offline  
❌ Firma electrónica  
❌ Reportes personalizables con query builder  
❌ BI avanzado con Machine Learning  
❌ Gamificación  

---

## 10. APÉNDICE: GLOSARIO

- **SG-SST:** Sistema de Gestión de Seguridad y Salud en el Trabajo
- **PHVA:** Planear-Hacer-Verificar-Actuar (ciclo de mejora continua)
- **Res. 0312/2019:** Resolución que define estándares mínimos del SG-SST en Colombia
- **Decreto 1072/2015:** Decreto Único Reglamentario del Sector Trabajo
- **No Conformidad:** Incumplimiento de requisito legal o estándar
- **Hallazgo:** Resultado de auditoría (observación, NC menor, NC mayor)
- **Acción Correctiva:** Medida para eliminar causa raíz de no conformidad
- **Evidencia:** Documento, registro o archivo que demuestra cumplimiento de actividad
- **Tenant:** Empresa/cliente en arquitectura multi-tenant
- **RBAC:** Role-Based Access Control (control de acceso basado en roles)

---

**FIN DEL PRD**

---

## PRÓXIMO ENTREGABLE
1. ✅ PRD  
2. 🔄 Arquitectura Técnica y Diagrama  
3. ⏳ Modelo de Datos (ERD)  
4. ⏳ API Design  
5. ⏳ UX/Wireframes  
6. ⏳ Backlog MVP Priorizado
