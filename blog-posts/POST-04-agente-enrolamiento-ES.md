# El Agente Go de Achilles: Enrolamiento de Endpoints y Gestión de Flota

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 4 de 15**

**Tiempo de lectura:** 9 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- El agente Achilles es un binario Go estático que corre como servicio en Windows, Linux y macOS
- Enrolamiento en 3 pasos: generar token → descargar binario → ejecutar con token
- El agente hace heartbeat cada 60 segundos y hace poll de tareas pendientes
- El backend verifica la firma del binario y la API key antes de aceptar cualquier resultado
- Self-update automático: el agente se actualiza solo cuando hay nueva versión

---

## El Agente: Qué Es y Qué Hace

El agente de Achilles es un binario Go de ~8 MB que se instala como servicio del sistema operativo. Una vez instalado, hace tres cosas:

```
Loop principal del agente:

Cada 60 segundos:
  1. HEARTBEAT → POST /api/agent/heartbeat
     → Reporta: hostname, OS, versión, IP, timestamp
     → Mantiene el agente "online" en el dashboard

  Cuando hay tareas:
  2. POLL → GET /api/agent/tasks/pending
     → Recibe: binario de test a ejecutar + parámetros
  3. EXECUTE → Ejecuta el test localmente
  4. REPORT → POST /api/agent/tasks/{id}/result
     → Envía: exit_code, output, bundle_results.json
```

---

## Paso 1: Descargar el Agente

### Desde la UI (recomendado)

```
1. Ve a Settings → Agent
2. Selecciona plataforma: Windows / Linux / macOS
3. Click "Download Agent"
   → Descarga el binario firmado para esa plataforma
```

### Plataformas disponibles

```
achilles-agent-windows-amd64.exe    → Windows 64-bit (firma Authenticode)
achilles-agent-linux-amd64          → Linux 64-bit (sin firma, static CGO_DISABLED)
achilles-agent-darwin-amd64         → macOS Intel (firma ad-hoc)
achilles-agent-darwin-arm64         → macOS Apple Silicon (firma ad-hoc)
```

---

## Paso 2: Generar Token de Enrolamiento

El token de enrolamiento es un one-time-use credential que el agente usa para registrarse.

### Desde la UI

```
Endpoints → Tokens → [+ Nuevo Token]

Opciones:
- Nombre: "Token para servidor de producción"
- Expira en: 1 hora / 24 horas / 7 días / sin expiración
- Usos máximos: 1 (un solo enrolamiento)

Resultado:
eyJhY2hpbGxlcyI6InRydWUiLCJ0b2tlbklkIjoiYWJjMTIzIn0...
```

### Desde la API

```bash
curl -X POST http://localhost:3000/api/agent/admin/tokens \
  -H "Authorization: Bearer $CLERK_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Token prod-server-01",
    "expiresIn": "24h",
    "maxUses": 1
  }'

# Respuesta:
{
  "success": true,
  "data": {
    "token": "eyJhY2hpbGxlcyI6...",
    "expiresAt": "2026-05-16T10:00:00Z"
  }
}
```

---

## Paso 3: Enrollar el Endpoint

### Windows (como servicio)

```powershell
# 1. Copia el binario al endpoint
Copy-Item achilles-agent-windows-amd64.exe C:\achilles\achilles-agent.exe

# 2. Instalar y enrolar (requiere PowerShell como Administrador)
C:\achilles\achilles-agent.exe install `
  --server https://tu-servidor.achilles.io `
  --token eyJhY2hpbGxlcyI6...

# El agente:
# → Se registra con el servidor
# → Recibe su API key permanente
# → Se instala como servicio de Windows: "AchillesAgent"
# → Inicia automáticamente

# Verificar que el servicio está corriendo:
Get-Service AchillesAgent
# Status: Running
```

### Linux (como servicio systemd)

```bash
# 1. Copia y da permisos de ejecución
sudo cp achilles-agent-linux-amd64 /usr/local/bin/achilles-agent
sudo chmod +x /usr/local/bin/achilles-agent

# 2. Enrolar e instalar servicio
sudo achilles-agent install \
  --server https://tu-servidor.achilles.io \
  --token eyJhY2hpbGxlcyI6...

# El agente crea automáticamente:
# /etc/systemd/system/achilles-agent.service
# Y ejecuta: systemctl enable --now achilles-agent

# Verificar:
systemctl status achilles-agent
# ● achilles-agent.service - Achilles Security Agent
#    Active: active (running) since ...
```

### macOS (como servicio launchd)

```bash
# 1. Copia el binario
sudo cp achilles-agent-darwin-arm64 /usr/local/bin/achilles-agent
sudo chmod +x /usr/local/bin/achilles-agent

# 2. Enrolar e instalar
sudo achilles-agent install \
  --server https://tu-servidor.achilles.io \
  --token eyJhY2hpbGxlcyI6...

# El agente crea un plist en:
# /Library/LaunchDaemons/io.achilles.agent.plist
# Y ejecuta: launchctl load ...

# Verificar:
sudo launchctl list | grep achilles
# -  0  io.achilles.agent
```

---

## Lo Que Pasa Detrás del Enrolamiento

```
[Agente]                              [Backend]
   │                                      │
   │ POST /api/agent/enroll               │
   │ { token: "eyJ...", hostname: "...",  │
   │   os: "windows", arch: "amd64" }     │
   │─────────────────────────────────────>│
   │                                      │
   │                        Valida token  │
   │                  Crea registro agent │
   │               Genera API key (HMAC)  │
   │                    Almacena en SQLite│
   │                                      │
   │<─────────────────────────────────────│
   │ { apiKey: "ach_xxxxxxxxxxxxxxxx",    │
   │   agentId: "uuid-xxx" }              │
   │                                      │
   │ Almacena API key localmente          │
   │ (cifrada con AES-256-GCM)            │
   │                                      │
   │ Inicia heartbeat loop ──────────────>│
```

La **API key** que recibe el agente es la credencial permanente. El token de enrolamiento se invalida inmediatamente después del primer uso.

---

## El Dashboard de Endpoints

Después del enrolamiento, el agente aparece en el dashboard:

```
┌─────────────────────────────────────────────────────────────────┐
│  ENDPOINTS                                              23 online│
├─────────────────────────────────────────────────────────────────┤
│  Hostname         OS         Versión   IP             Estado    │
├─────────────────────────────────────────────────────────────────┤
│  DESKTOP-SRV01    Windows    1.4.2     192.168.1.45   🟢 Online │
│  linux-prod-01    Linux      1.4.2     10.0.0.12      🟢 Online │
│  macbook-kendra   macOS ARM  1.4.2     192.168.1.100  🟢 Online │
│  DESKTOP-OLD01    Windows    1.3.0     192.168.1.78   🔴 Offline│
└─────────────────────────────────────────────────────────────────┘
```

**Estado online/offline**: el backend marca un agente como offline si no recibe heartbeat en los últimos 3 minutos.

---

## Ver Detalles de un Agente

Click en cualquier agente para ver su página de detalles:

```
┌─────────────────────────────────────────────────────────────────┐
│  DESKTOP-SRV01                                    🟢 Online      │
├─────────────────────────────────────────────────────────────────┤
│  OS:          Windows 11 Pro 23H2                               │
│  Arquitectura: amd64                                            │
│  Versión:     1.4.2                                             │
│  IP:          192.168.1.45                                      │
│  Último heartbeat: hace 23 segundos                             │
│  Enrolado:    2026-05-14 09:15:33                               │
├─────────────────────────────────────────────────────────────────┤
│  HISTORIAL DE TAREAS                                            │
│                                                                 │
│  T1059.001   Ejecutado   exit:0  ✅  2026-05-14 02:01:15       │
│  T1021.001   Ejecutado   exit:1  ❌  2026-05-14 02:01:45       │
│  T1547.001   Ejecutado   exit:0  ✅  2026-05-14 02:02:20       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Gestión de Flota: Operaciones Comunes

### Ver todos los agentes (API)

```bash
curl http://localhost:3000/api/agent/admin/agents \
  -H "Authorization: Bearer $CLERK_JWT"

# Respuesta:
{
  "success": true,
  "data": [
    {
      "id": "uuid-xxx",
      "hostname": "DESKTOP-SRV01",
      "os": "windows",
      "version": "1.4.2",
      "lastHeartbeat": "2026-05-14T10:23:45Z",
      "status": "online"
    },
    ...
  ]
}
```

### Revocar un agente

```bash
# Desde la UI: Endpoints → [agente] → "Revocar acceso"
# O via API:
curl -X DELETE http://localhost:3000/api/agent/admin/agents/{id} \
  -H "Authorization: Bearer $CLERK_JWT"
```

### Forzar actualización de agente

```bash
# Desde la UI: Endpoints → [agente] → "Actualizar agente"
# El backend envía una tarea especial de tipo "update"
# El agente descarga el nuevo binario, verifica la firma y se reinicia
```

---

## Self-Update: Cómo Funciona

El agente puede actualizarse automáticamente sin intervención manual:

```
1. Agente consulta: GET /api/agent/updates/latest
   → Respuesta: { version: "1.4.3", downloadUrl: "...", signature: "..." }

2. Si versión actual < versión disponible:
   → Descarga el nuevo binario
   → Verifica firma Ed25519 antes de aplicar

3. Aplica la actualización:
   → Reemplaza el binario actual
   → Reinicia el servicio (Windows: SCM, Linux: systemd, macOS: launchd)

4. Post-update: primer heartbeat confirma la nueva versión
```

Si la verificación de firma falla, el agente descarta la actualización y registra el error — **nunca aplica un binario sin firma válida**.

---

## Seguridad del Agente: Detalle

### La API key

```
Formato: ach_<64-chars-random>
Almacenamiento en backend: HMAC-SHA256 hash (nunca en claro)
Almacenamiento en agente: cifrado AES-256-GCM en config local

Cada request del agente incluye:
Authorization: Bearer ach_xxxxxxxxxxxxxxxxxxxxx
```

### Protección anti-replay

```
Cada request incluye un timestamp.
El backend rechaza requests con más de 5 minutos de diferencia.
Esto previene replay attacks si se captura tráfico.
```

### Verificación de binarios de test

```
Antes de ejecutar cualquier test binary:
1. Backend firma el hash del binario con su clave privada Ed25519
2. Agente verifica la firma antes de ejecutar
3. Si la firma no es válida → rechaza ejecución + alerta al backend
```

---

## El Agente Como Herramienta Purple Team

El agente no es solo un "ejecutor de tests". Desde la perspectiva del blue team:

```
Red Team perspective:
→ "El agente simula un adversario en el endpoint"
→ Ejecuta las mismas técnicas que un APT real usaría

Blue Team perspective:
→ "El agente es nuestra sonda de validación"
→ Confirma que las reglas SIEM están activas y funcionan
→ Genera evidencia objetiva de cobertura
```

La diferencia clave: el agente **no tiene acceso** a credenciales de dominio, no se mueve lateralmente, y todas sus acciones están registradas y son predecibles. Es un APT bajo control.

---

## Troubleshooting del Agente

### El agente no aparece como online

```bash
# Windows: verificar servicio
Get-Service AchillesAgent
sc query AchillesAgent

# Linux: verificar servicio
systemctl status achilles-agent
journalctl -u achilles-agent -n 50

# macOS: verificar launchd
sudo launchctl list io.achilles.agent

# Causa más común: AGENT_SERVER_URL incorrecto
# El agente no puede llegar al backend desde la red del endpoint
```

### "Token inválido o expirado"

```bash
# Los tokens son single-use y expiran
# Genera un nuevo token y reintenta el enrolamiento:
# Settings → Agent → Tokens → [+ Nuevo Token]
```

### El agente muestra versión desactualizada

```bash
# Forzar actualización desde la UI:
# Endpoints → [agente] → "Actualizar"

# O desinstalar y reinstalar:
# Windows:
achilles-agent.exe uninstall
achilles-agent.exe install --server ... --token ...
```

---

## Puntos Clave

✅ El agente es un binario Go estático: cero dependencias, funciona en cualquier endpoint
✅ Enrolamiento en 3 pasos: token → download → install
✅ Heartbeat cada 60 segundos mantiene visibilidad de la flota
✅ API key permanente (hasheada en backend), token de enrolamiento one-time
✅ Self-update automático con verificación de firma Ed25519
✅ Funciona como servicio nativo: Windows SCM, systemd, launchd

---

## Próximo Post

**POST 05: "La Librería de Tests — 500+ Técnicas MITRE ATT&CK"**

Cómo navegar la librería, entender la estructura de los tests, y seleccionar los tests correctos para tu entorno.

---

**Etiquetas:** #ProjectAchilles #Go #PurpleTeam #CiberSeguridad #MITRE #EndpointSecurity #Tutorial

---

*Parte 4 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
