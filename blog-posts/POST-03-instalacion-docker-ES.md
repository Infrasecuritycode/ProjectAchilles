# Instalación de Project Achilles en 5 Minutos con Docker Compose

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 3 de 15**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Docker Compose levanta el stack completo: backend, frontend y Elasticsearch
- El perfil `elasticsearch` incluye datos sintéticos de seed para probar Analytics
- Las variables de entorno en `backend/.env` controlan autenticación y configuración
- Primera ejecución: backend en `:3000`, frontend en `:5173`, ES en `:9200`
- Tiempo real desde cero hasta dashboard con datos: **5 minutos**

---

## Prerequisitos

Antes de empezar, necesitas:

```
✅ Docker Desktop (o Docker Engine + Docker Compose v2)
   → Versión mínima: Docker 24.x
   → Verificar: docker --version && docker compose version

✅ Git
   → Para clonar el repositorio

✅ Cuenta de Clerk (gratis)
   → Para la autenticación de usuarios
   → https://clerk.com → crear app nueva → copiar las keys

✅ (Opcional) Instancia de Elasticsearch
   → O usa el perfil incluido en Docker Compose
```

---

## Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/projectachilles/achilles
cd achilles

# Verificar la estructura
ls -la
# CLAUDE.md  README.md  docker-compose.yml
# agent/  backend/  frontend/  scripts/  docs/
```

---

## Paso 2: Configurar Variables de Entorno

Achilles necesita dos archivos `.env`:

### Backend: `backend/.env`

```bash
cp backend/.env.example backend/.env
```

Edita `backend/.env` con los siguientes valores:

```bash
# ── Clerk (autenticación de usuarios) ──────────────────────────
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ── Seguridad ────────────────────────────────────────────────────
# Clave para cifrar configuración sensible (AES-256-GCM)
# Genera una: openssl rand -hex 32
ENCRYPTION_SECRET=a1b2c3d4e5f6...64chars...

# ── Base de datos ────────────────────────────────────────────────
# SQLite se crea automáticamente en ~/.projectachilles/agents.db
# No necesitas configurar nada aquí para desarrollo

# ── Elasticsearch ────────────────────────────────────────────────
# Usando el ES incluido en Docker Compose:
ELASTICSEARCH_URL=http://elasticsearch:9200
# ELASTICSEARCH_API_KEY=  (vacío si usas ES sin seguridad)

# ── CORS ─────────────────────────────────────────────────────────
CORS_ORIGIN=http://localhost:5173

# ── URL del servidor de agentes ──────────────────────────────────
# Esta URL deben alcanzarla los agentes instalados en endpoints
AGENT_SERVER_URL=http://localhost:3000
```

### Frontend: `frontend/.env.local`

```bash
# Clerk (misma publishable key que en backend)
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Puerto del backend (Vite proxia /api → backend)
VITE_BACKEND_PORT=3000
```

### Cómo obtener las keys de Clerk

```
1. Ve a https://clerk.com y crea una cuenta (gratis)
2. Crea una nueva aplicación
3. Elige "Email + Password" como método de autenticación
4. En el dashboard de Clerk → API Keys
5. Copia "Publishable key" (pk_test_...) y "Secret key" (sk_test_...)
```

---

## Paso 3: Levantar el Stack

### Opción A: Solo backend + frontend (sin ES)

Si ya tienes Elasticsearch en otra máquina o en la nube:

```bash
docker compose up -d

# Verifica que los servicios estén corriendo:
docker compose ps
# NAME          STATUS    PORTS
# achilles-backend   Up      0.0.0.0:3000->3000/tcp
# achilles-frontend  Up      0.0.0.0:5173->5173/tcp
```

### Opción B: Stack completo con Elasticsearch + datos de prueba ⭐ Recomendado

```bash
docker compose --profile elasticsearch up -d

# Esto levanta:
# → Backend Express en :3000
# → Frontend React en :5173
# → Elasticsearch 8.17 en :9200 (single-node, sin TLS)
# → Seed de 1,000 resultados sintéticos para Analytics
```

```bash
# Verifica los logs mientras arranca:
docker compose logs -f

# Cuando veas esto, está listo:
# achilles-backend  | ✓ Server running on port 3000
# achilles-frontend | Local:   http://localhost:5173/
# elasticsearch     | Cluster health: green
```

---

## Paso 4: Primer Login

Abre `http://localhost:5173` en tu navegador.

```
Pantalla de bienvenida de Achilles:
┌──────────────────────────────────────┐
│           PROJECT ACHILLES           │
│                                      │
│   [   Sign Up   ]  [   Sign In   ]   │
│                                      │
│   Powered by Clerk                   │
└──────────────────────────────────────┘
```

**Si es la primera vez:**
1. Click en "Sign Up"
2. Ingresa tu email y contraseña
3. Verifica el email (Clerk envía un link)
4. Eres redirigido al dashboard principal

---

## Paso 5: Configurar Analytics

Si usaste el perfil `elasticsearch` con datos sintéticos, el Analytics ya tiene datos. Necesitas conectarlo:

```
1. Ve a Settings → Integrations → Analytics
2. Elasticsearch URL: http://localhost:9200
3. (Sin API key para la instancia local)
4. Click "Test Connection"
   → ✓ Connected · 1,000 documents found
5. Click "Save"
```

Ahora ve a Analytics (`/analytics`) — deberías ver:

```
Defense Score:  ~71%
Tests:          1,000
Hosts:          23 (simulados)
Técnicas:       312 cubiertas
```

---

## Paso 6: Verificar que Todo Funciona

### Check 1: Backend API

```bash
curl http://localhost:3000/health
# {"status":"ok","version":"1.x.x"}
```

### Check 2: Elasticsearch

```bash
curl http://localhost:9200
# {"name":"...", "cluster_name":"docker-cluster", ...}

# Ver índices de Achilles:
curl http://localhost:9200/_cat/indices/achilles*
# green open achilles-results  ...  1,000 docs
```

### Check 3: Frontend

```bash
# Abre en el navegador:
open http://localhost:5173

# Navegación esperada:
# /          → Landing page
# /browser   → Librería de tests (vacía hasta que sincronices)
# /analytics → Dashboard con datos sintéticos
# /endpoints → Sin agentes aún (los añadiremos en POST 04)
# /settings  → Configuración
```

---

## Estructura de Datos: ¿Qué Genera el Seed?

Los 1,000 documentos sintéticos representan tests ejecutados contra una flota simulada:

```json
{
  "f0rtika": {
    "test_uuid": "7659eeba-f315-440e-9882-4aa015d68b27",
    "test_name": "T1059.001-powershell-exec",
    "technique_id": "T1059.001",
    "tactic": "Execution",
    "severity": "high",
    "exit_code": 0,
    "hostname": "DESKTOP-SRV01",
    "platform": "windows",
    "protected": true
  },
  "@timestamp": "2026-05-01T14:23:00Z"
}
```

**exit_code** es el indicador clave:
```
exit_code: 0  → Test ejecutado, defensa DETECTÓ el ataque ✅
exit_code: 1  → Test ejecutado, defensa NO detectó el ataque ❌
exit_code: 2  → Error en ejecución del test ⚠️
```

---

## Solución de Problemas Comunes

### El frontend muestra "Authentication Error"

```bash
# Problema: Keys de Clerk incorrectas o no configuradas
# Solución: Verifica frontend/.env.local
cat frontend/.env.local
# VITE_CLERK_PUBLISHABLE_KEY debe comenzar con pk_test_ o pk_live_

# Reconstruye el frontend después de cambiar .env:
docker compose restart achilles-frontend
```

### Analytics muestra "Connection Error"

```bash
# Problema: Elasticsearch no está corriendo o mal configurado
docker compose --profile elasticsearch ps
# Si elasticsearch no aparece: olvidaste --profile elasticsearch

# Verifica la URL en Settings → Analytics:
# ✓ http://elasticsearch:9200  (dentro de Docker network)
# ✗ http://localhost:9200      (solo funciona fuera de Docker)
```

### El backend no arranca

```bash
# Ver logs del backend:
docker compose logs achilles-backend

# Problema común: ENCRYPTION_SECRET no configurado
# Solución:
echo $(openssl rand -hex 32)  # Genera una clave
# Añade al backend/.env: ENCRYPTION_SECRET=<resultado>
docker compose restart achilles-backend
```

### Ports ya en uso

```bash
# Si el puerto 5173 o 3000 está ocupado:
# backend/.env → PORT=3001
# frontend/.env.local → VITE_BACKEND_PORT=3001

# O mata el proceso que usa el puerto:
lsof -ti:5173 | xargs kill -9
```

---

## Comandos Útiles de Operación

```bash
# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f achilles-backend

# Reiniciar un servicio
docker compose restart achilles-backend

# Parar todo
docker compose --profile elasticsearch down

# Parar y limpiar volúmenes (⚠️ borra datos)
docker compose --profile elasticsearch down -v

# Ver estado
docker compose ps

# Actualizar imágenes
docker compose pull && docker compose up -d
```

---

## Desarrollo Local (Sin Docker)

Si prefieres desarrollo con hot-reload:

```bash
# Script todo-en-uno (detecta puertos automáticamente)
./scripts/start.sh -k --daemon
# -k: mata procesos existentes en esos puertos
# --daemon: corre en background

# Para cuando termines:
./scripts/start.sh --stop

# O manualmente:
cd backend  && npm install && npm run dev  # Puerto 3000
cd frontend && npm install && npm run dev  # Puerto 5173
```

---

## Verificación Final: Checklist

```
✅ docker compose ps → todos los servicios "Up"
✅ http://localhost:3000/health → {"status":"ok"}
✅ http://localhost:9200 → respuesta de Elasticsearch
✅ http://localhost:5173 → login de Achilles visible
✅ Primer login exitoso con Clerk
✅ Analytics → Defense Score visible (si usaste --profile elasticsearch)
✅ Settings → Analytics → "Connection successful"
```

---

## Puntos Clave

✅ `docker compose --profile elasticsearch up -d` levanta el stack completo
✅ Dos archivos `.env` necesarios: `backend/.env` y `frontend/.env.local`
✅ Clerk es gratis y necesario para la autenticación
✅ Los datos sintéticos de seed permiten explorar Analytics inmediatamente
✅ `exit_code: 0` = detectado, `exit_code: 1` = no detectado

---

## Próximo Post

**POST 04: "El Agente Go — Enrolamiento de Endpoints y Gestión de Flota"**

Cómo generar tokens de enrolamiento, instalar el agente en un endpoint real y verlo aparecer en el dashboard.

---

**Etiquetas:** #ProjectAchilles #Docker #DevOps #CiberSeguridad #Tutorial #Instalación #Elasticsearch #OpenSource

---

*Parte 3 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
