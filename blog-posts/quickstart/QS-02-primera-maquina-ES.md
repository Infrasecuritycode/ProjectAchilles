# Conectar Tu Primera Máquina: El Agente en 10 Minutos

> **Serie: Empezando con Project Achilles — Parte 2 de 6**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- El "agente" es un programa pequeño (~8 MB) que instalas en cada máquina que quieres monitorear
- Una sola vez, publicas el binario del agente en tu servidor (`Settings → Agent → Build Binary`); luego enrolas todas las máquinas que quieras
- El proceso de enrolamiento tiene 3 pasos: generar un token → copiar el comando → ejecutarlo en la máquina
- Funciona en Windows, Linux y macOS
- Una vez instalado, la máquina aparece como "online" en el dashboard en menos de 1 minuto
- No necesitas abrir puertos ni configurar firewalls — el agente se conecta hacia afuera (como un navegador)

---

## ¿Qué Es el Agente?

El agente de Achilles es un programa pequeño que vive en cada máquina que quieres proteger y validar. Hace tres cosas:

1. **Avisa a Achilles que está activo** (cada 60 segundos manda un "sigo aquí")
2. **Ejecuta las simulaciones de ataque** cuando le toca
3. **Reporta los resultados** de vuelta al dashboard

Es un programa liviano. No consume recursos notables. No afecta el rendimiento de la máquina. Y para desinstalarlo, un comando y listo.

---

## Prerrequisito: Publicar el Binario del Agente (una sola vez)

Antes de poder enrolar máquinas, tu servidor Achilles necesita tener **publicado al menos un binario del agente** para cada plataforma que vayas a usar (Windows, Linux, macOS). El comando de instalación descarga ese binario desde tu servidor — si no hay ninguno publicado, la descarga falla con un error `404: "No version available for this platform"`.

Es un paso de administrador que haces **una sola vez** por plataforma (y lo repites solo cuando saques una versión nueva del agente). Hay dos formas de hacerlo — usa la que te funcione.

### Opción A — Construir en el servidor (recomendado)

Desde el dashboard:

```
Settings → pestaña "Agent" → tarjeta "Build Agent Binary"
```

Rellena los tres campos y haz click en **Build Binary**:

```
Version:            0.5.0            ← número de versión (el formulario sugiere el siguiente)
Operating System:   Windows          ← Linux | Windows | macOS
Architecture:       x86_64 (amd64)   ← x86_64 (amd64) | ARM64

[ 🔨 Build Binary ]
```

El servidor cross-compila el agente desde el código fuente (puede tardar hasta un minuto). Cuando termina, el binario aparece en la tarjeta **"Registered Versions"** de la misma página, listo para descargar.

> **Si desplegaste con el instalador de DigitalOcean** (`scripts/deploy-do/`), el droplet ya tiene Go instalado automáticamente (lo hace la Phase 10 del deployer), así que el "Build from Source" funciona de entrada — no necesitas instalar Go a mano. En servidores donde montaste Achilles por tu cuenta, el build requiere que Go (≥ la versión de `agent/go.mod`) esté instalado en el backend; si falta, el build falla con `Command failed: go ... spawn go ENOENT` y tendrás que instalarlo o usar la Opción B.

> **Repite el build por cada plataforma que vayas a enrolar.** Si tienes máquinas Windows y Linux, construye `Windows / amd64` y `Linux / amd64` por separado. La arquitectura ARM64 solo aplica a servidores con CPU ARM (algunas VMs en la nube, Macs Apple Silicon, Raspberry Pi).

> **Verificación rápida:** en la tarjeta "Registered Versions" debes ver al menos una fila con tu versión, OS y arquitectura. Si está vacía, el build no se completó — revisa el mensaje de error en la tarjeta de build.

### Opción B — Construir localmente y subir (si el build en el servidor falla)

El "Build from Source" del servidor puede fallar por dos motivos: en servidores con poca RAM (por ejemplo, un droplet de 1 GB), la cross-compilación de Go agota la memoria y el proceso muere; o si Go no está instalado en el backend (build manual, sin el deployer de DigitalOcean) verás un error `spawn go ENOENT`. En cualquiera de los dos casos la alternativa es **compilar el binario en tu máquina** y subirlo ya hecho.

**1) Compila el agente en tu máquina** (necesitas [Go](https://go.dev/dl/) instalado, versión ≥ la indicada en `agent/go.mod`):

```bash
cd agent

# Windows (x86_64)
make build-windows        # genera dist/achilles-agent-windows-amd64.exe

# Otras plataformas, según necesites:
make build-linux          # dist/achilles-agent-linux-amd64
make build-darwin-arm64   # dist/achilles-agent-darwin-arm64   (Mac Apple Silicon)
make build-darwin-amd64   # dist/achilles-agent-darwin-amd64   (Mac Intel)
make build-all            # las cuatro plataformas de una vez
```

> El número de versión sale del `Makefile` (línea `VERSION := ...`) — anótalo, lo necesitas al subir.

**2) Sube el binario** en el dashboard:

```
Settings → pestaña "Agent" → tarjeta "Upload Agent Binary"

   Version:            0.6.2            ← la misma del Makefile
   Operating System:   Windows          ← Linux | Windows | macOS
   Architecture:       x86_64 (amd64)   ← x86_64 (amd64) | ARM64
   File:               achilles-agent-windows-amd64.exe

   [ Upload ]
```

Igual que en la Opción A, debe aparecer en **"Registered Versions"**.

> **Firma:** el binario compilado así va **sin firmar** a menos que uses `make sign-windows` (requiere un certificado configurado). Un agente sin firma puede ser bloqueado por Windows Defender en la máquina objetivo — ver el troubleshooting al final de este post.

---

Una vez publicado el binario (por cualquiera de las dos opciones), sigue con los 3 pasos de enrolamiento.

---

## Paso 1: Generar un Token de Instalación

El token es un código único que le dice al agente "perteneces a esta cuenta de Achilles".

Desde el dashboard:

```
Endpoints → Agents → botón "Enroll Agent" (arriba a la derecha)
```

Se despliega un formulario con dos campos:

```
TTL (hours):  24      ← cuántas horas tiene de vida el token
Max Uses:      1      ← cuántas máquinas pueden usarlo

[ Generate Token ]
```

Deja los valores por defecto (24 horas, 1 uso) y haz click en **Generate Token**.

El dashboard muestra inmediatamente el token y los comandos de instalación listos para copiar, uno por plataforma. No necesitas descargar nada por separado.

---

## Paso 2: Copiar y Ejecutar el Comando

Al generar el token, el dashboard muestra los comandos con el token y la URL de tu servidor ya rellenos. Solo tienes que copiar el de tu plataforma y ejecutarlo en la máquina objetivo.

### Windows

Abre **PowerShell como Administrador** y ejecuta el comando que aparece bajo **"Windows (PowerShell)"**:

```powershell
Invoke-WebRequest -Uri "http://<tu-servidor>/api/agent/download?os=windows&arch=amd64" `
  -OutFile achilles-agent.exe; `
  .\achilles-agent.exe --enroll <TOKEN> --server http://<tu-servidor> --install
```

> **¿No sabes qué es PowerShell como Administrador?** Click derecho en el menú Inicio → "Windows PowerShell (Administrador)".

El comando descarga el agente, lo registra en tu cuenta y lo instala como servicio de Windows con arranque automático, todo en un paso.

### Linux

Ejecuta el comando bajo **"Linux (amd64)"** o **"Linux (arm64)"** según tu arquitectura:

```bash
curl -fSL "http://<tu-servidor>/api/agent/download?os=linux&arch=amd64" \
  -o achilles-agent && \
  chmod +x achilles-agent && \
  sudo ./achilles-agent --enroll <TOKEN> --server http://<tu-servidor> --install
```

### macOS

Ejecuta el comando bajo **"macOS (Apple Silicon)"** o **"macOS (Intel)"**:

```bash
# Apple Silicon (M1/M2/M3)
curl -fSL "http://<tu-servidor>/api/agent/download?os=darwin&arch=arm64" \
  -o achilles-agent && \
  chmod +x achilles-agent && \
  sudo ./achilles-agent --enroll <TOKEN> --server http://<tu-servidor> --install

# Intel
curl -fSL "http://<tu-servidor>/api/agent/download?os=darwin&arch=amd64" \
  -o achilles-agent && \
  chmod +x achilles-agent && \
  sudo ./achilles-agent --enroll <TOKEN> --server http://<tu-servidor> --install
```

> **`<tu-servidor>`** es la URL que configuraste en QS-01:
> - Instalación local: `http://localhost:3000`
> - DigitalOcean: `http://<IP-pública>:3000`
>
> El dashboard ya rellena esto automáticamente en los comandos que muestra — solo copia y pega.

---

## Paso 3: Verificar que Apareció en el Dashboard

Vuelve al dashboard y ve a **Endpoints → Agents**:

```
AGENTS                                             1 online

Hostname        Sistema    Versión   IP            Estado
─────────────────────────────────────────────────────────
MI-PC-01        Windows    1.4.2     192.168.1.45  🟢 Online
```

¡Ahí está! La máquina aparece en menos de 60 segundos después de la instalación.

Si no aparece en 2 minutos, revisa la sección de troubleshooting al final de este post.

---

## ¿Qué Información Ve Achilles de Mi Máquina?

Una pregunta válida. El agente reporta:

```
✅ Nombre de la máquina (hostname)
✅ Sistema operativo y versión
✅ Dirección IP
✅ Versión del agente instalada
✅ Resultados de los tests ejecutados (exit code + output)

❌ NO reporta archivos o documentos
❌ NO reporta contraseñas ni credenciales
❌ NO tiene acceso a tu correo ni aplicaciones
❌ NO graba pantalla ni keystrokes
```

El agente solo ejecuta los tests que le asignas y reporta si fueron detectados o no.

---

## Añadir Más Máquinas

Una vez que viste que funciona en la primera, el proceso para las demás es idéntico:

1. Genera un token nuevo (cada token tiene `Max Uses: 1` por defecto)
2. Copia el comando para esa plataforma
3. Ejecútalo en la máquina objetivo
4. Repite

Para una flota grande (20+ máquinas), puedes aumentar el `Max Uses` al generar el token — así un solo token sirve para varias máquinas sin tener que generar uno por cada una. Útil si despliegas con Ansible, SCCM o similar.

---

## Gestionar Tus Máquinas

Desde **Endpoints → Agents** puedes ver el detalle de cada máquina:

```
Click en "MI-PC-01"

┌──────────────────────────────────────────────────────┐
│  MI-PC-01                               🟢 Online    │
├──────────────────────────────────────────────────────┤
│  Sistema:   Windows 11 Pro                           │
│  IP:        192.168.1.45                             │
│  Agente:    v1.4.2                                   │
│  Online desde: hace 3 minutos                        │
├──────────────────────────────────────────────────────┤
│  Historial de tests: (vacío — aún no has ejecutado)  │
└──────────────────────────────────────────────────────┘
```

Desde aquí también puedes:
- **Desasignar la máquina** si ya no quieres monitorizarla
- **Forzar actualización** del agente si hay una versión nueva
- **Ver el historial** de todos los tests ejecutados

---

## Troubleshooting: Si No Aparece

**La máquina no aparece después de 2 minutos:**

```
Causa más común: el agente no puede llegar al servidor Achilles

Windows — verificar el servicio:
  Get-Service AchillesAgent
  → Si Status: Stopped, el servicio no arrancó

  Ver logs:
  Get-EventLog -LogName Application -Source AchillesAgent -Newest 10

Linux — verificar el servicio:
  systemctl status achilles-agent

macOS — verificar el servicio:
  sudo launchctl list | grep achilles

Causa habitual: la URL del servidor está mal o hay un firewall bloqueando
el puerto 3000 entre la máquina y el servidor Achilles.
```

**La descarga falla con "404: No version available for this platform":**
```
No has publicado el binario del agente para esa plataforma todavía.
Ve al prerrequisito al inicio de este post y publícalo:
  - Opción A: Settings → Agent → Build Agent Binary (construye en el servidor)
  - Opción B: compila local con `make build-<plataforma>` y súbelo en
              Settings → Agent → Upload Agent Binary (si el build del servidor falla)

Verifica que aparezca en "Registered Versions" antes de reintentar.
```

**El token dice "inválido" o "expirado":**
```
Los tokens expiran según el TTL configurado.
Genera uno nuevo en:
  Endpoints → Agents → "Enroll Agent" → Generate Token
```

**Windows Defender bloqueó el ejecutable:**
```
Esto puede pasar porque es un binario nuevo sin firma reconocida.

Opciones:
1. Añade una exclusión en Defender para la carpeta donde guardaste el agente
2. O usa la versión firmada — requiere configurar un certificado en
   Settings → Tests → Certificates (cubre QS-04)

Nota: que Defender bloquee el agente sin firma es en sí mismo
información útil — significa que tu defensa detecta binarios sin firma.
```

---

## Puntos Clave

✅ 3 pasos: generar token → copiar el comando del dashboard → ejecutarlo en la máquina
✅ El dashboard genera los comandos listos para copiar, con el token y la URL ya rellenos
✅ La máquina aparece en el dashboard en menos de 60 segundos
✅ Funciona igual en Windows, Linux y macOS
✅ El agente no accede a tus datos personales ni archivos

---

## Próximo Post

**QS-03: "Un Primer Vistazo al Dashboard"**

Ya tienes una máquina conectada. Exploramos el dashboard de Achilles: qué muestra cada módulo y cómo orientarte antes de ejecutar tu primer test.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #Tutorial #Instalación #Agente #Principiantes

---

*Parte 2 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
