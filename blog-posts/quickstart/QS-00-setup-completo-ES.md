# Instala Project Achilles en 30 Minutos: Guía Paso a Paso

> **Serie: Empezando con Project Achilles — Guía de Instalación**

**Tiempo de lectura:** 12 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Achilles necesita dos cosas: un lugar donde correr la plataforma + al menos una máquina con el agente
- Puedes instalar en tu máquina local (Opción A, gratis) o en un VPS en la nube como DigitalOcean (Opción B, ~$8/mes)
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

### Opción A: Self-hosted con Docker — Gratis 🐳

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

### Opción B: Servidor en la nube (ej. DigitalOcean) — ~$8/mes ☁️

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



**Para quién:** Quieres que el dashboard sea accesible desde cualquier lugar y que los agentes puedan conectarse desde fuera de tu red local.

```
Pros:
✅ Accesible desde cualquier lugar — URL pública para ti y los agentes
✅ No consume recursos de tu máquina local
✅ Los datos se quedan en TU servidor, no en un tercero

Contras:
→ Cuesta ~$8/mes (droplet básico en DigitalOcean o equivalente)
→ Tú gestionas el servidor (actualizaciones, backups)
→ Requiere los mismos pasos de instalación que la opción local
```

**Cómo empezar:** Sigue la **Sección 1B** de este post.

> **Nota:** Achilles es open-source — no existe una versión "cloud gestionada".
> Tú instalas, tú controlas. La nube es simplemente dónde eliges correrlo.

---

## Sección 1A: Instalar en Local (Docker en tu propia máquina)

*(Si elegiste la Opción B — DigitalOcean — salta a la Sección 1B)*

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
# Docker sirve el frontend en el puerto 80 (http://localhost)
CORS_ORIGIN=http://localhost
```

Crea el archivo `.env` en la raíz del proyecto (Docker Compose lo lee para pasar variables al contenedor del frontend):

```bash
# En la raíz del proyecto (junto a docker-compose.yml)
cp .env.example .env
```

Abre `.env` y añade tu Clerk publishable key:

```bash
CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxx
```

> **¿Por qué dos archivos?**
> `backend/.env` lo inyecta el backend directamente. El contenedor del frontend recibe su configuración del `.env` raíz, que Docker Compose interpola al arrancar.

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
# achilles-frontend   Up   0.0.0.0:80->80/tcp
# elasticsearch       Up   0.0.0.0:9200->9200/tcp
```

### Paso 5: Abrir el dashboard

Abre tu navegador en: **http://localhost**

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

## Sección 1B: Instalar en DigitalOcean (VPS en la nube)

*(Si elegiste la Opción A — Docker local — ya terminaste con la Sección 1A, salta a la Sección 2)*

### Paso 1: Crear el droplet en DigitalOcean

```
1. Ve a https://digitalocean.com → "Sign Up" (o inicia sesión)
2. Click "Create" → "Droplets"
3. Elige la configuración:
   Imagen:    Ubuntu 22.04 LTS x64
   Plan:      Basic → Regular → $8/mes (1 vCPU, 2 GB RAM, 50 GB disco)
   Región:    La más cercana a tus máquinas (ej. NYC, AMS, FRA)
   Autenticación: SSH key (recomendado) o Password
4. Nombre del droplet: "achilles-server"
5. Click "Create Droplet"
6. Espera ~1 minuto — DigitalOcean te mostrará la IP pública
   Ejemplo: 167.99.123.45
```

### Paso 2: Conectarte al servidor

```bash
# Desde tu terminal local (Mac/Linux)
ssh root@167.99.123.45

# Windows: usa PuTTY o Windows Terminal con:
# ssh root@167.99.123.45
```

### Paso 3: Instalar Docker y Git

```bash
# Actualizar el sistema
apt update && apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com | sh

# Verificar que Docker está corriendo
docker --version
docker compose version

# Instalar Git
apt install -y git
```

### Paso 4: Descargar Achilles

```bash
git clone https://github.com/projectachilles/achilles
cd achilles
```

### Paso 5: Exponer los puertos al exterior

El `docker-compose.yml` por defecto enlaza los puertos solo a `localhost` (seguro para máquinas locales, pero inaccesible desde internet en un VPS). Crea un archivo de sobreescritura para abrirlos:

```bash
cat > docker-compose.override.yml << 'EOF'
services:
  backend:
    ports:
      - "0.0.0.0:3000:3000"
  frontend:
    ports:
      - "0.0.0.0:80:80"
EOF
```

Docker Compose fusiona este archivo automáticamente al hacer `docker compose up` — no necesitas modificar el `docker-compose.yml` original.

### Paso 6: Crear tu cuenta de Clerk

Sigue exactamente el **Paso 2** de la Sección 1A (más arriba).
El proceso es idéntico — Clerk es gratis y funciona igual en local o en la nube.

### Paso 7: Configurar las variables de entorno

```bash
cp backend/.env.example backend/.env
nano backend/.env   # o usa: vi backend/.env
```

Rellena con tu IP pública del droplet (ej. `167.99.123.45`):

```bash
# ── Clerk ────────────────────────────────────────
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxx
CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxx

# ── Seguridad ─────────────────────────────────────
# Genera la clave con: openssl rand -hex 32
ENCRYPTION_SECRET=pon-aqui-una-clave-de-64-caracteres-aleatoria

# ── URL pública del servidor ──────────────────────
# Los agentes usarán esta URL para conectarse
AGENT_SERVER_URL=http://167.99.123.45:3000

# ── CORS ─────────────────────────────────────────
# Usa la IP pública, no localhost
CORS_ORIGIN=http://167.99.123.45
```

Crea el `.env` raíz:

```bash
cp .env.example .env
nano .env
```

Añade tu Clerk publishable key:

```bash
CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxx
```

### Paso 8: Abrir los puertos en el firewall de DigitalOcean

```
En el panel de DigitalOcean → tu droplet → "Networking" → "Firewalls"
→ Create Firewall → añade estas reglas de entrada (Inbound):

  Tipo    Puerto    Origen
  ─────────────────────────────────────
  HTTP    80        All IPv4, All IPv6
  Custom  3000      All IPv4, All IPv6

→ Aplica el firewall al droplet "achilles-server"
```

> Si prefieres usar `ufw` desde el servidor:
> ```bash
> ufw allow 80/tcp
> ufw allow 3000/tcp
> ufw enable
> ```

### Paso 9: Arrancar todo

```bash
docker compose --profile elasticsearch up -d
```

Espera 1-2 minutos y verifica:

```bash
docker compose ps
# achilles-backend    Up   0.0.0.0:3000->3000/tcp
# achilles-frontend   Up   0.0.0.0:80->80/tcp
# elasticsearch       Up   0.0.0.0:9200->9200/tcp
```

### Paso 10: Abrir el dashboard

Desde tu navegador (en cualquier computadora): **http://167.99.123.45**

Crea tu cuenta con "Registrarse" y sigue el **Paso 6** de la Sección 1A para conectar Elasticsearch.

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
- Si Achilles está en DigitalOcean: `http://IP-PUBLICA-DEL-DROPLET:3000`

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
OPCIÓN B — DigitalOcean (~$8/mes):
  1. Crear droplet Ubuntu + instalar Docker   (10 min)
  2. Clonar repo + configurar .env con IP pública (10 min)
  3. docker compose up + abrir puertos        (5 min)
  4. Crear cuenta + conectar Elasticsearch    (5 min)
  5. Instalar agente + primer test            (10 min)
  Total: ~40 minutos

OPCIÓN A — Local (gratis):
  1. Clonar repo + configurar .env            (10 min)
  2. docker compose up                        (5 min)
  3. Crear cuenta + conectar Elasticsearch    (5 min)
  4. Instalar agente + primer test            (10 min)
  Total: ~30 minutos
```

---

## Puntos Clave

✅ Dos opciones: local con Docker (gratis, Opción A) o VPS en la nube como DigitalOcean (~$8/mes, Opción B)
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
