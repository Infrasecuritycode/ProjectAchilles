# Instala Project Achilles en 30 Minutos: Guía Paso a Paso

> **Serie: Empezando con Project Achilles — Guía de Instalación**

**Tiempo de lectura:** 12 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Achilles necesita dos cosas: un lugar donde correr la plataforma + al menos una máquina con el agente
- Puedes instalar la plataforma tú mismo (gratis) o usar la versión en la nube ($8/mes)
- El agente se instala en 3 comandos en Windows, Linux o macOS
- Desde cero hasta ver tu primer Defense Score: 30 minutos
- Este post te lleva paso a paso por la opción que elijas

---

## Lo Que Vas a Instalar

Achilles tiene dos partes:

```
PARTE 1: La plataforma Achilles
→ El dashboard web que usas para ver resultados
→ La base de datos donde se guardan los tests
→ El servidor al que se conectan los agentes

PARTE 2: El agente (en cada máquina que quieras monitorear)
→ Un programa pequeño (~8 MB)
→ Ejecuta las simulaciones de ataque
→ Reporta los resultados al dashboard
```

Primero instalas la plataforma. Luego instalas el agente en tus máquinas.

---

## Elige Tu Opción para la Plataforma

### Opción A: Versión en la nube — $8/mes ☁️

**Para quién:** Quieres empezar rápido sin gestionar servidores.

```
Pros:
✅ Listo en 2 minutos — sin instalación
✅ Sin mantenimiento de servidor
✅ Actualizaciones automáticas
✅ Accesible desde cualquier lugar

Contras:
→ Cuesta $8/mes
→ Los datos de resultados van a la nube
   (los tests se ejecutan en TUS máquinas locales,
    pero los resultados se almacenan externamente)
```

**Cómo empezar:**
```
1. Ve a https://projectachilles.io
2. Click "Get started"
3. Crea tu cuenta
4. Salta directo a la Sección 2 de este post (instalar el agente)
```

---

### Opción B: Self-hosted con Docker — Gratis 🐳

**Para quién:** Quieres que todo quede en tu infraestructura, o no quieres pagar.

```
Pros:
✅ Completamente gratis
✅ Todos los datos se quedan en tu red
✅ Control total

Contras:
→ Necesitas una máquina dedicada (puede ser la misma donde
   instalas el agente si es un equipo de prueba)
→ Tú gestionas las actualizaciones
```

**Requisitos mínimos del servidor:**
```
Sistema operativo: Linux, macOS o Windows con Docker Desktop
RAM: 2 GB mínimo (4 GB recomendado)
Disco: 10 GB libres
Docker: versión 24 o superior
```

**¿Tienes Docker instalado?**
```bash
docker --version
# Docker version 24.x.x — ✓ listo

docker compose version
# Docker Compose version v2.x.x — ✓ listo
```

Si no tienes Docker:
- Windows/Mac: descarga **Docker Desktop** desde https://docker.com/products/docker-desktop
- Linux: sigue la guía oficial de tu distribución (Ubuntu: `apt install docker.io`)

---

## Sección 1A: Instalar la Plataforma con Docker

*(Si elegiste la opción cloud, salta a la Sección 2)*

### Paso 1: Descargar Achilles

```bash
git clone https://github.com/projectachilles/achilles
cd achilles
```

Si no tienes git: descarga el ZIP desde GitHub → "Code" → "Download ZIP", extrae y entra a la carpeta.

### Paso 2: Crear tu cuenta de Clerk (autenticación gratuita)

Achilles usa Clerk para el login de usuarios. Necesitas crear una app gratuita:

```
1. Ve a https://clerk.com → "Start building for free"
2. Crea una cuenta (es gratis)
3. Crea una nueva aplicación:
   Nombre: "Achilles"
   Método de login: Email + Password
4. En el dashboard de Clerk → "API Keys"
5. Copia estos dos valores:
   → Publishable key: pk_test_xxxxxxxxxx...
   → Secret key:      sk_test_xxxxxxxxxx...
```

### Paso 3: Configurar las variables de entorno

```bash
# Copia el archivo de ejemplo
cp backend/.env.example backend/.env
```

Abre `backend/.env` con cualquier editor de texto y rellena:

```bash
# ── Clerk ────────────────────────────────────────
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxx
CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxx

# ── Seguridad (genera una clave aleatoria) ────────
# En Mac/Linux ejecuta: openssl rand -hex 32
# En Windows PowerShell: [System.Web.Security.Membership]::GeneratePassword(64,0)
ENCRYPTION_SECRET=pon-aqui-una-clave-de-64-caracteres-aleatoria

# ── URL de tu servidor ────────────────────────────
# Si el agente corre en la misma red: usa la IP local
# Si el agente corre fuera: usa tu IP pública o dominio
AGENT_SERVER_URL=http://192.168.1.100:3000

# ── CORS ─────────────────────────────────────────
CORS_ORIGIN=http://localhost:5173
```

Crea el archivo `frontend/.env.local`:

```bash
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxx
VITE_BACKEND_PORT=3000
```

### Paso 4: Arrancar todo

```bash
# Stack completo con Elasticsearch y datos de ejemplo
docker compose --profile elasticsearch up -d
```

Espera 1-2 minutos mientras los contenedores arrancan. Verifica que todo está corriendo:

```bash
docker compose ps

# Deberías ver:
# achilles-backend    Up   0.0.0.0:3000->3000/tcp
# achilles-frontend   Up   0.0.0.0:5173->5173/tcp
# elasticsearch       Up   0.0.0.0:9200->9200/tcp
```

### Paso 5: Abrir el dashboard

Abre tu navegador en: **http://localhost:5173**

```
┌──────────────────────────────────────────┐
│           PROJECT ACHILLES               │
│                                          │
│  Email:      [____________________]      │
│  Contraseña: [____________________]      │
│                                          │
│  [ Iniciar sesión ]  [ Registrarse ]     │
└──────────────────────────────────────────┘
```

Crea tu primera cuenta con "Registrarse". Usa tu email — Clerk te enviará un código de verificación.

### Paso 6: Conectar Elasticsearch

```
Settings → Integrations → Analytics

Elasticsearch URL: http://elasticsearch:9200
(dentro de Docker, usa este hostname)

[ Test Connection ] → ✅ Connected · 1,000 documents

[ Guardar ]
```

Ve a **Analytics** — deberías ver el dashboard con datos de ejemplo ya cargados.

---

## Sección 2: Instalar el Agente en una Máquina

Esta sección aplica **tanto si elegiste cloud como self-hosted**.

El agente es el programa que corre en las máquinas que quieres validar.

### Paso 1: Generar un token de instalación

```
Desde el dashboard → Endpoints → Tokens → [+ Nuevo Token]

Nombre:     "Mi primera máquina"
Válido por: 24 horas

[ Crear ]

Token generado:
eyJhY2hpbGxlcyI6InRydWUiLCJ0b2tlbklkIjoiYWJjMTIzIn0...

→ Copia este valor
```

### Paso 2: Descargar el agente

```
Settings → Agent → Descargar Agente

Elige tu sistema:
● Windows (64-bit)      → achilles-agent-windows.exe
○ Linux (64-bit)        → achilles-agent-linux
○ macOS Intel           → achilles-agent-macos-intel
○ macOS Apple Silicon   → achilles-agent-macos-arm

[ Descargar ]
```

Copia el archivo descargado a la máquina donde lo quieres instalar.

---

### Instalar en Windows

Abre **PowerShell como Administrador**:

```powershell
# Crear carpeta para el agente
New-Item -ItemType Directory -Path "C:\achilles" -Force

# Mover el ejecutable
Move-Item .\achilles-agent-windows.exe C:\achilles\

# Instalar como servicio de Windows
C:\achilles\achilles-agent-windows.exe install `
  --server http://IP-DE-TU-SERVIDOR:3000 `
  --token eyJhY2hpbGxlcyI6...

# Verificar que está corriendo
Get-Service AchillesAgent
```

```
Status   Name            DisplayName
------   ----            -----------
Running  AchillesAgent   Achilles Security Agent  ✓
```

**¿Cuál es la IP de tu servidor?**
- Si instalaste con Docker en la misma máquina: `http://localhost:3000`
- Si Docker está en otra máquina de tu red: `http://192.168.1.X:3000`
- Si usas la versión cloud: la URL que te dio Achilles al registrarte

---

### Instalar en Linux

```bash
# Dar permisos de ejecución
chmod +x achilles-agent-linux

# Instalar como servicio del sistema (requiere sudo)
sudo ./achilles-agent-linux install \
  --server http://IP-DE-TU-SERVIDOR:3000 \
  --token eyJhY2hpbGxlcyI6...

# Verificar que está corriendo
systemctl status achilles-agent
```

```
● achilles-agent.service - Achilles Security Agent
   Active: active (running) since ...  ✓
```

---

### Instalar en macOS

```bash
# Dar permisos de ejecución
chmod +x achilles-agent-macos-arm  # o -intel según tu Mac

# Instalar (requiere sudo)
sudo ./achilles-agent-macos-arm install \
  --server http://IP-DE-TU-SERVIDOR:3000 \
  --token eyJhY2hpbGxlcyI6...

# Verificar
sudo launchctl list | grep achilles
# -  0  io.achilles.agent  ✓
```

---

## Paso 3: Confirmar que la Máquina Aparece en el Dashboard

Vuelve al dashboard → **Endpoints**:

```
ENDPOINTS                                    1 online

Hostname        Sistema    Versión   IP              Estado
────────────────────────────────────────────────────────────
MI-PC-01        Windows    1.4.2     192.168.1.45    🟢 Online
```

Debería aparecer en menos de **60 segundos**.

Si no aparece, revisa la sección de solución de problemas al final.

---

## Paso 4: Tu Primer Test (5 minutos)

Con la máquina conectada, ejecuta tu primera simulación:

```
Browser → busca "PowerShell Encoded" → click en el resultado

[ Compilar ]  →  espera ~10 segundos

[ Asignar a Agente ]
Máquina: MI-PC-01
Horario: Ahora mismo
[ Asignar ]
```

Espera 60 segundos y ve a **Analytics → Executions**:

```
PowerShell Encoded Command   MI-PC-01   ✅ Protegido   hace 45 seg
```

¡Listo! Ejecutaste tu primer test de seguridad real.

---

## Solución de Problemas Comunes

### El agente no aparece en el dashboard

```
Causa más común: no llega al servidor

Verificar conectividad (desde la máquina del agente):
  Windows:  Test-NetConnection -ComputerName IP-SERVIDOR -Port 3000
  Linux/Mac: curl http://IP-SERVIDOR:3000/health

Si falla: revisar firewall en el servidor
  → El puerto 3000 debe estar abierto para la red interna

Ver logs del agente:
  Windows:  Get-EventLog -LogName Application -Source AchillesAgent -Newest 10
  Linux:    journalctl -u achilles-agent -n 20
  macOS:    log show --predicate 'subsystem == "io.achilles.agent"' --last 5m
```

### "Token inválido o expirado"

```
Los tokens son de un solo uso y expiran en 24 horas.
Genera uno nuevo:
Endpoints → Tokens → [+ Nuevo Token]
```

### Windows Defender bloqueó el ejecutable

```
Achilles genera el bloqueo porque el binario es nuevo para Defender.
Dos opciones:

1. Permitir manualmente:
   Windows Security → Virus & threat protection → Protection history
   → Encuentra el bloqueo → "Allow on device"

2. O añadir exclusión de carpeta:
   Windows Security → Virus & threat protection settings
   → Add or remove exclusions → Add folder → C:\achilles\

Nota: que Defender bloquee el agente sin firma es información útil
en sí misma — significa que tu EDR detecta binarios sin firma.
```

### El dashboard está vacío después del login

```
Falta conectar Elasticsearch:
Settings → Integrations → Analytics
→ URL: http://elasticsearch:9200 (si usas Docker)
→ [ Test Connection ] → [ Guardar ]
```

---

## Resumen: Los 4 Pasos

```
OPCIÓN CLOUD:
  1. Crear cuenta en projectachilles.io       (2 min)
  2. Generar token de instalación             (1 min)
  3. Descargar e instalar el agente           (5 min)
  4. Ejecutar primer test                     (2 min)
  Total: ~10 minutos

OPCIÓN SELF-HOSTED:
  1. Clonar repo + configurar .env            (10 min)
  2. docker compose up                        (5 min)
  3. Crear cuenta + conectar Elasticsearch    (5 min)
  4. Instalar agente + primer test            (10 min)
  Total: ~30 minutos
```

---

## Puntos Clave

✅ Dos opciones: cloud ($8/mes, 10 min) o self-hosted (gratis, 30 min)
✅ El agente se instala con 1 comando en Windows, Linux y macOS
✅ La máquina aparece en el dashboard en menos de 60 segundos
✅ El primer test tarda 2 minutos en ejecutarse y dar resultado
✅ Si algo no funciona, los logs del agente explican exactamente qué pasa

---

## Próximo Post

**QS-01: "¿Para Qué Sirve Project Achilles?"**

Ya tienes Achilles instalado. Ahora entendamos qué hace exactamente, qué significa el Defense Score y cómo usarlo para mejorar tu seguridad.

---

**Etiquetas:** #ProjectAchilles #Instalación #Docker #Tutorial #CiberSeguridad #Paso a Paso #Principiantes

---

*Guía de instalación de la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
