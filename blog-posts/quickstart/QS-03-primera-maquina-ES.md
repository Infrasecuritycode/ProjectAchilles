# Conectar Tu Primera Máquina: El Agente en 10 Minutos

> **Serie: Empezando con Project Achilles — Parte 3 de 6**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- El "agente" es un programa pequeño (~8 MB) que instalas en cada máquina que quieres monitorear
- El proceso tiene 3 pasos: generar un token → descargar el agente → instalarlo
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

## Paso 1: Generar un Token de Instalación

El token es un código único que le dice al agente "perteneces a esta cuenta de Achilles". Es de un solo uso por seguridad.

```
Desde el dashboard:
Endpoints → Tokens → [+ Nuevo Token]

Opciones:
  Nombre:    "Mi primera máquina de prueba"
  Válido por: 24 horas (suficiente para instalarlo ahora)

[ Crear Token ]

Resultado:
eyJhY2hpbGxlcyI6InRydWUiLCJ0b2tlbklkIjoiYWJjMTIzIn0...

→ Copia este token, lo necesitas en el Paso 3
```

**Tip:** El token expira en 24 horas. Si tardas más, solo crea uno nuevo — es instantáneo.

---

## Paso 2: Descargar el Agente

```
Settings → Agent → Descargar Agente

Elige tu plataforma:
  ● Windows (64-bit)     → achilles-agent-windows.exe
  ○ Linux (64-bit)       → achilles-agent-linux
  ○ macOS (Intel)        → achilles-agent-macos-intel
  ○ macOS (Apple Silicon)→ achilles-agent-macos-arm

[ Descargar ]
```

El archivo descargado pesa unos 8 MB. Es un ejecutable que no necesita instalador, no necesita Java, no necesita nada más.

---

## Paso 3: Instalar en la Máquina

### Windows

Abre **PowerShell como Administrador** y ejecuta:

```powershell
# 1. Mueve el archivo a una carpeta permanente
New-Item -ItemType Directory -Path "C:\achilles" -Force
Move-Item achilles-agent-windows.exe C:\achilles\

# 2. Instala como servicio de Windows (arranque automático)
C:\achilles\achilles-agent-windows.exe install `
  --server https://tu-instancia.achilles.io `
  --token eyJhY2hpbGxlcyI6...

# Verifica que está corriendo:
Get-Service AchillesAgent
# Status: Running ✓
```

¿No sabes qué es PowerShell como Administrador? Haz click derecho en el menú Inicio → "Windows PowerShell (Administrador)".

### Linux

```bash
# Da permisos de ejecución al archivo
chmod +x achilles-agent-linux

# Instala como servicio del sistema
sudo ./achilles-agent-linux install \
  --server https://tu-instancia.achilles.io \
  --token eyJhY2hpbGxlcyI6...

# Verifica que está corriendo:
systemctl status achilles-agent
# Active: active (running) ✓
```

### macOS

```bash
# Da permisos de ejecución
chmod +x achilles-agent-macos-arm  # o -intel según tu Mac

# Instala (requiere sudo)
sudo ./achilles-agent-macos-arm install \
  --server https://tu-instancia.achilles.io \
  --token eyJhY2hpbGxlcyI6...
```

---

## Paso 4: Verifica que Apareció en el Dashboard

Vuelve al dashboard y ve a **Endpoints**:

```
ENDPOINTS                                          1 online

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

1. Genera un token nuevo (cada token es para una sola máquina)
2. Descarga el agente para esa plataforma
3. Instala con el token
4. Repite

Para una flota grande (20+ máquinas), puedes automatizar con tu herramienta de gestión existente (SCCM, Ansible, Chef, etc.) — el comando de instalación es siempre el mismo, solo cambia el token.

---

## Gestionar Tus Máquinas

Desde **Endpoints** puedes ver información de cada máquina:

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

Verificar en Windows:
Get-Service AchillesAgent
→ Si Status: Stopped → el servicio no arrancó

→ Ver logs:
   Get-EventLog -LogName Application -Source AchillesAgent -Newest 10

Causa común: la URL del servidor está mal o hay firewall bloqueando
```

**El token dice "inválido":**
```
Los tokens expiran. Genera uno nuevo en:
Endpoints → Tokens → [+ Nuevo Token]
```

**Windows Defender bloqueó el ejecutable:**
```
Esto puede pasar porque es un binario nuevo.

Opciones:
1. Añade una exclusión en Defender para C:\achilles\
2. O descarga la versión firmada desde Settings → Agent → "Descarga firmada"
   (requiere que hayas configurado un certificado)

Nota: que Defender bloquee el agente sin firma es en sí mismo
información útil — significa que tu defensa detecta binarios sin firma.
```

---

## Puntos Clave

✅ 3 pasos: generar token → descargar → instalar
✅ La máquina aparece en el dashboard en menos de 60 segundos
✅ Funciona igual en Windows, Linux y macOS
✅ El agente no accede a tus datos personales ni archivos
✅ Para más máquinas: mismo proceso, token diferente por máquina

---

## Próximo Post

**QS-04: "Ejecutar Tu Primer Test — Ver Achilles en Acción"**

Ya tienes una máquina conectada. Ahora ejecutaremos el primer test de seguridad y veremos el resultado en tiempo real.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #Tutorial #Instalación #Agente #Principiantes

---

*Parte 3 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
