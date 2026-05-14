# La Arquitectura de Project Achilles: 3 Módulos, 1 Objetivo

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 2 de 15**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Achilles tiene **3 módulos**: Browser (librería de tests), Analytics (métricas), Agent (gestión de endpoints)
- **Stack**: React 19 + TypeScript (frontend) / Express + TypeScript (backend) / Go (agente) / Elasticsearch (datos)
- **Autenticación triple**: Clerk (usuarios) + API key (agentes) + Elasticsearch credentials
- El agente Go es estático, compilado para 6 plataformas, firmado, con self-update
- Puedes desplegar en Docker Compose, Render, Fly.io o Vercel (serverless)

---

## El Stack Completo de un Vistazo

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENTE (NAVEGADOR)                        │
│                  React 19 · TypeScript · Vite 7                 │
│              Tailwind CSS v4 · Redux Toolkit · Clerk            │
└─────────────────────────────┬───────────────────────────────────┘
                              │ HTTPS / JWT
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKEND (API REST)                           │
│              Express · TypeScript · Node 22                     │
│                  SQLite (better-sqlite3, WAL)                   │
│           Clerk middleware · Rate limiting · AsyncHandler       │
└──────────────┬──────────────────────────────┬───────────────────┘
               │ ES client                    │ Ed25519 + HMAC
               ▼                              ▼
┌──────────────────────────┐    ┌─────────────────────────────────┐
│     ELASTICSEARCH        │    │          AGENTES GO              │
│   achilles-results       │    │  Windows / Linux / macOS        │
│   achilles-defender      │    │  Heartbeat · Tasks · Self-update │
│   (heatmaps, scores)     │    │  AES-256-GCM · Ed25519           │
└──────────────────────────┘    └─────────────────────────────────┘
```

---

## Módulo 1: Browser — La Librería de Tests

El Browser es la interfaz para explorar y gestionar la librería de tests de seguridad.

### ¿Qué contiene?

```
Librería de tests:
├── Por táctica MITRE ATT&CK
│   ├── Initial Access (T1078, T1190, T1566...)
│   ├── Execution (T1059, T1053, T1047...)
│   ├── Persistence (T1547, T1543...)
│   └── ... (11 tácticas en total)
├── Por severidad (Critical / High / Medium / Low)
├── Por plataforma (Windows / Linux / macOS)
└── Por tipo (standalone / bundle)
```

### Cómo se sincroniza la librería

La librería de tests vive en un repositorio Git separado (`f0_library`). El backend hace un `git pull` periódico y reindexea los tests:

```typescript
// backend: BrowserService.syncLibrary()
await git.pull()
await indexer.reindex()  // Actualiza metadatos MITRE, severidad, etc.
```

### Build & Sign

Desde el Browser puedes compilar y firmar los binarios de tests directamente en la UI:

```
[Browser UI]
Test: T1059.001 - PowerShell Execution
Plataformas: ✅ Windows x64  ☐ Linux  ☐ macOS

[ Compilar y Firmar ]
  → Cross-compile Go → dist/T1059.001-win-amd64.exe
  → Authenticode sign con certificado activo
  → Disponible para descarga o asignación directa
```

---

## Módulo 2: Analytics — Las Métricas que Importan

El módulo de Analytics consume datos de Elasticsearch y genera visualizaciones en tiempo real.

### Los 4 indicadores principales

```
┌────────────────────────────────────────────────────────────────┐
│  HERO METRICS                                                  │
│                                                                │
│  Defense Score    Tests Ejecutados   Tests Fallidos   Hosts   │
│     73.2%              1,847             312            23     │
│    ↑ 4.1% ↑          30 días           16.9%        activos  │
└────────────────────────────────────────────────────────────────┘
```

**Defense Score**: porcentaje de técnicas MITRE ATT&CK que tu stack detectó.

```
Defense Score = (Tests detectados / Tests ejecutados) × 100

Ejemplo:
- Tests ejecutados esta semana: 150
- Tests que generaron alerta: 109
- Defense Score: 72.7%
```

### Las visualizaciones clave

**Heatmap MITRE ATT&CK**
```
Táctica        T1059  T1078  T1021  T1055  T1547
Execution      [🟢]   [  ]   [  ]   [  ]   [  ]
Persistence    [  ]   [🟡]   [  ]   [  ]   [🔴]
Lat.Movement   [  ]   [  ]   [🟢]   [  ]   [  ]
Defense Evasion[  ]   [  ]   [  ]   [🔴]   [  ]

🟢 Protegido  🟡 Parcial  🔴 Gap  [ ] Sin testear
```

**Coverage Treemap**: visualización por área del framework con tamaño proporcional a la criticidad.

**Trend Chart**: Defense Score en el tiempo — ¿sube o baja con cada cambio de configuración?

**Executions Table**: cada test ejecutado, con resultado, host, duración y técnica MITRE.

### Integración con Microsoft Defender

El Analytics tiene una pestaña dedicada a Defender que muestra:
- Secure Score de Microsoft
- Alertas activas correlacionadas con tests Achilles
- Cross-correlation: ¿cuántas técnicas detecta Achilles Y Defender?
- Auto-Resolve: cierre automático de alertas generadas por tests válidos

---

## Módulo 3: Agent — La Flota de Endpoints

El módulo de Agent gestiona el ciclo de vida completo de los endpoints.

### El flujo de enrolamiento

```
1. Admin genera token de enrolamiento
   → POST /api/agent/admin/tokens
   → Token: eyJhY2hpbGxlcyI6...

2. Instalar agente en endpoint
   → Descargar binario firmado
   → Ejecutar: achilles-agent.exe --token eyJhY2hpbGxlcyI6...

3. El agente se registra automáticamente
   → POST /api/agent/enroll
   → Backend valida token, crea registro en SQLite
   → El agente recibe su API key permanente

4. Heartbeat continuo
   → Cada 60 segundos: POST /api/agent/heartbeat
   → Estado: hostname, OS, versión, IP
   → Backend marca agente como "online"
```

### El ciclo de vida de una tarea

```
Admin asigna tarea → Backend encola tarea → Agente hace poll
→ Agente descarga test binary → Ejecuta test
→ Resultado: exit_code + evidence → POST /api/agent/tasks/{id}/result
→ Backend ingesta en Elasticsearch → Analytics actualizado
```

### La base de datos SQLite

```
Tablas principales:
agents          → hostname, OS, IP, último heartbeat, estado
enrollment_tokens → tokens, expiración, usos
tasks           → tarea asignada, estado, resultado
schedules       → tareas programadas por cron
agent_versions  → historial de versiones desplegadas
```

---

## Seguridad del Agente: Tres Capas

```
Capa 1: AUTENTICACIÓN
  → Token de enrolamiento (single-use, expira)
  → API key permanente (HMAC-SHA256, almacenada hasheada)
  → Verificación Ed25519 de binarios de test

Capa 2: TRANSPORTE
  → TLS en todo el tráfico
  → AES-256-GCM para payload de resultados
  → Protección anti-replay (ventana de 5 minutos)

Capa 3: INTEGRIDAD
  → Binarios firmados (Authenticode / ad-hoc macOS)
  → Self-update verifica firma antes de aplicar
  → Rate limiting por endpoint en el backend
```

---

## El Backend: Organización del Código

```
backend/src/
├── api/              → Route handlers (*.routes.ts)
├── services/
│   ├── agent/        → enrollment, heartbeat, tasks, schedules
│   ├── analytics/    → Elasticsearch queries, mappings
│   ├── browser/      → Git sync, test indexing
│   ├── tests/        → Build service, cert management
│   └── defender/     → Microsoft Graph, auto-resolve
├── middleware/        → Auth (Clerk), error handling, rate limiting
└── server.ts          → Entry point
```

### Patrón estándar de rutas

```typescript
// Todas las rutas async usan asyncHandler + AppError
router.get('/agents', asyncHandler(async (req, res) => {
  const agents = await agentService.listAgents();
  res.json({ success: true, data: agents });
}));

// Errores con formato estándar
if (!agent) throw new AppError('Agent not found', 404);
// → { success: false, error: "Agent not found" }
```

---

## El Frontend: Arquitectura React

```
frontend/src/
├── pages/
│   ├── browser/    → Librería de tests
│   ├── analytics/  → Dashboard de métricas
│   ├── endpoints/  → Gestión de agentes y tareas
│   └── settings/   → Configuración, certs, integraciones
├── hooks/
│   └── useAuthenticatedApi  → Inyecta JWT automáticamente
├── store/          → Redux slices por módulo
└── services/api/   → Clientes HTTP por módulo
```

### Tres capas de autenticación en frontend

```
1. Clerk (global): <RequireAuth> wrapper en todas las rutas
2. Analytics: AnalyticsAuthProvider → redirige a /analytics/setup
3. Agente Admin: JWT de Clerk requerido para todas las operaciones
```

---

## Opciones de Despliegue

Achilles tiene 5 targets de despliegue soportados:

```
Opción 1: Docker Compose (más fácil)
docker compose up -d
→ Backend + Frontend + Elasticsearch
→ SQLite en volumen
→ Recomendado para: lab, demo, producción pequeña

Opción 2: Render (PaaS simple)
→ Persistent disk para SQLite
→ ~$15/mes (starter)
→ Recomendado para: equipos sin DevOps

Opción 3: Fly.io (más barato)
→ Volume para SQLite
→ ~$8/mes
→ Recomendado para: producción a bajo coste

Opción 4: Railway
→ Deploy desde GitHub
→ ~$10/mes
→ Recomendado para: simplicidad máxima

Opción 5: Vercel (serverless)
→ Turso (LibSQL) en lugar de SQLite
→ Vercel Blob para almacenamiento
→ Recomendado para: escala automática
```

---

## El Agente Go: Por Dentro

```
agent/internal/
├── config/       → Configuración y persistencia local
├── enrollment/   → Registro con el backend
├── executor/     → Ejecución de tests y lectura de resultados
├── httpclient/   → Cliente HTTP con retry y backoff
├── poller/       → Long-polling de tareas pendientes
├── reporter/     → Envío de resultados al backend
├── service/      → Integración con SCM/systemd/launchd
├── store/        → Almacenamiento local (API key, estado)
├── sysinfo/      → Información del sistema (OS, hostname, IP)
└── updater/      → Self-update con verificación de firma
```

El agente usa build tags de Go para código específico de plataforma:

```go
//go:build darwin
// Gestión de servicio via launchd plist

//go:build windows  
// Gestión via Windows Service Control Manager (sc.exe)

//go:build linux
// Gestión via systemd
```

---

## Flujo de Datos Completo

```
TEST EXECUTION → RESULT INGESTION → ANALYTICS

1. Agente ejecuta test binary
2. Test escribe resultado en c:\F0\bundle_results.json
3. Executor.go lee y valida el JSON
4. Agente POST → /api/agent/tasks/{id}/result
5. Backend detecta tipo (standalone vs bundle)
6. results.service.ts → bulk ingest en Elasticsearch
7. Índice achilles-results actualizado
8. Frontend Analytics consulta ES vía /api/analytics/*
9. Defense Score recalculado en tiempo real
```

---

## Puntos Clave

✅ Stack moderno: React 19 + Express + Go + Elasticsearch
✅ Tres módulos independientes pero integrados: Browser, Analytics, Agent
✅ Seguridad en capas: Clerk + Ed25519 + AES-256-GCM + TLS
✅ Agente Go estático: sin dependencias, funciona en cualquier endpoint
✅ 5 opciones de despliegue desde Docker Compose hasta Vercel
✅ Toda la data en tu infraestructura (self-hosted)

---

## Próximo Post

**POST 03: "Instalación en 5 Minutos con Docker Compose"**

Paso a paso: desde cero hasta tu primera sesión de Analytics con datos reales.

---

**Etiquetas:** #ProjectAchilles #Arquitectura #Go #React #Elasticsearch #CiberSeguridad #MITRE #PurpleTeam #OpenSource

---

*Parte 2 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
