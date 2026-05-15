# Build & Sign: Compilar y Firmar tus Propios Agentes

> **Serie: Pro — Parte 2 de 3**

**Tiempo de lectura:** 9 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- El agente descargado desde el dashboard es el binario oficial — para la mayoría de entornos es suficiente
- Si tu EDR bloquea binarios no firmados, necesitas compilar y firmar con tu propio certificado
- El proceso usa Go + `make build-all` y produce binarios para Windows, Linux y macOS en un solo comando
- Windows requiere firma Authenticode (certificado PFX) y macOS usa firma ad-hoc
- Una vez subido el certificado al dashboard, el build firmado está disponible para descarga en un click

---

## ¿Cuándo Necesitas Hacer Esto?

```
No necesitas compilar tú mismo si:
✅ Tus máquinas tienen Windows Defender estándar
✅ Usas Defender for Endpoint sin políticas personalizadas de firma
✅ Tu EDR no bloquea binarios sin firma Authenticode de tu organización

Sí necesitas compilar y firmar si:
→ Tu EDR tiene una política de "solo ejecutar binarios firmados por nosotros"
→ Tu organización exige que todos los ejecutables tengan el certificado corporativo
→ Haces pentesting en entornos con AppLocker o WDAC (Windows Defender Application Control)
→ Quieres evitar que el hash del binario cambie entre versiones sin tu control
```

---

## Lo Que Necesitas Antes de Empezar

### 1. Go 1.24 o superior

```bash
go version
# go version go1.24.x linux/amd64  ✓
```

Si no lo tienes:
```bash
# Linux:
wget https://go.dev/dl/go1.24.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.24.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# macOS (con Homebrew):
brew install go

# Windows:
# Descarga el instalador desde https://go.dev/dl/
```

### 2. El código fuente de Achilles

```bash
git clone https://github.com/projectachilles/achilles
cd achilles/agent
```

### 3. Herramientas de firma (solo si necesitas firmar)

**Windows Authenticode** — `osslsigncode`:
```bash
# Linux (para firmar binarios Windows desde Linux):
sudo apt install osslsigncode

# macOS:
brew install osslsigncode

# Verificar:
osslsigncode --version
# osslsigncode 2.x.x  ✓
```

**macOS ad-hoc** — `rcodesign`:
```bash
# macOS (con Homebrew):
brew install rcodesign

# o descarga desde https://github.com/indygreg/apple-platform-rs
```

---

## Paso 1: Compilar para Todas las Plataformas

El Makefile incluido compila todo en un comando:

```bash
cd agent
make build-all
```

Esto produce (en `agent/dist/`):

```
dist/
├── achilles-agent-windows-amd64.exe    # Windows 64-bit
├── achilles-agent-linux-amd64          # Linux 64-bit
├── achilles-agent-macos-amd64          # macOS Intel
└── achilles-agent-macos-arm64          # macOS Apple Silicon
```

El proceso de compilación usa CGO desactivado para generar binarios estáticos — no necesitan librerías externas en la máquina destino:

```bash
# Lo que hace make build-all internamente:
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build \
  -ldflags="-X main.version=$(VERSION)" \
  -o dist/achilles-agent-windows-amd64.exe .

CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
  -ldflags="-X main.version=$(VERSION)" \
  -o dist/achilles-agent-linux-amd64 .

# ... y así para cada combinación OS/arquitectura
```

### Especificar la versión

```bash
make build-all VERSION=1.5.0-corp
```

Esto incrusta la versión en el binario — aparece en el dashboard bajo la columna "Versión" de cada agente.

---

## Paso 2A: Firmar para Windows (Authenticode)

La firma Authenticode en Windows le dice al sistema operativo "este ejecutable viene de quien dice ser". Sin ella, SmartScreen y algunos EDR muestran advertencias o bloquean directamente.

### Lo que necesitas: un certificado PFX

```
Opción A — Certificado de Code Signing comercial:
→ Emitido por DigiCert, Sectigo, GlobalSign, etc.
→ Cuesta ~$200-400/año
→ Requiere validación de identidad de la empresa
→ SmartScreen lo reconoce inmediatamente

Opción B — Certificado autofirmado (interno):
→ Gratis, lo generas tú
→ Solo funciona en máquinas donde hayas instalado
  el certificado raíz manualmente (GPO/Intune)
→ SmartScreen no lo reconoce, pero tu EDR puede
  estar configurado para confiar en tu CA interna
→ Adecuado para entornos corporativos cerrados

Opción C — Certificado existente de tu organización:
→ El que ya usas para firmar software interno
→ La opción más común en entornos enterprise
```

### Generar un certificado autofirmado (si lo necesitas)

```powershell
# Windows PowerShell (como Administrador):
$cert = New-SelfSignedCertificate `
  -Type CodeSigningCert `
  -Subject "CN=Achilles Security Tool, O=TuEmpresa S.L." `
  -KeyAlgorithm RSA `
  -KeyLength 4096 `
  -CertStoreLocation Cert:\LocalMachine\My `
  -NotAfter (Get-Date).AddYears(3)

# Exportar como PFX:
$pwd = ConvertTo-SecureString -String "TuContraseñaSegura" -Force -AsPlainText
Export-PfxCertificate `
  -Cert $cert `
  -FilePath C:\achilles-signing.pfx `
  -Password $pwd
```

### Firmar el binario

```bash
# Desde Linux o macOS, usando osslsigncode:
osslsigncode sign \
  -pkcs12 /ruta/a/tu-certificado.pfx \
  -pass "TuContraseña" \
  -n "Achilles Security Agent" \
  -i "https://projectachilles.io" \
  -in  dist/achilles-agent-windows-amd64.exe \
  -out dist/achilles-agent-windows-signed.exe

# Verificar la firma:
osslsigncode verify dist/achilles-agent-windows-signed.exe
# Signature verification: ok  ✓
```

O usando el Makefile si tienes el certificado configurado:

```bash
make sign-windows \
  CERT_PATH=/ruta/a/certificado.pfx \
  CERT_PASS=TuContraseña
```

---

## Paso 2B: Firmar para macOS (Ad-hoc)

En macOS, la firma ad-hoc no requiere un certificado de Apple — es una firma local que identifica el binario y evita que macOS lo trate como "software desconocido" en muchos contextos.

```bash
# Firma ad-hoc con rcodesign (no requiere cuenta de Apple Developer):
rcodesign sign \
  --code-signature-flags adhoc \
  dist/achilles-agent-macos-arm64

rcodesign sign \
  --code-signature-flags adhoc \
  dist/achilles-agent-macos-amd64

# Verificar:
rcodesign print-signature-info dist/achilles-agent-macos-arm64
```

O con el Makefile:

```bash
make sign-darwin
```

> **Nota:** La firma ad-hoc no pasa Gatekeeper si el binario viene de Internet. Para distribución interna esto no es un problema — el agente se despliega por scripts/Intune/JAMF, no se descarga por el usuario.

### Linux

Los binarios de Linux no se firman. Los sistemas Linux corporativos usan otros mecanismos de control (SELinux, AppArmor, políticas de repositorios) que no dependen de firma de ejecutables.

---

## Paso 3: Subir el Certificado al Dashboard

Una vez que tienes el PFX, súbelo a Achilles para que el sistema firme automáticamente en los builds futuros:

```
Settings → Tests → Certificados → [+ Subir Certificado]

Archivo:      achilles-signing.pfx
Contraseña:   (tu contraseña del PFX)
Nombre:       "Certificado corporativo 2026"

[ Subir ]

✅ Certificado instalado
   Válido hasta: 2028-05-15
   Subject: CN=Achilles Security Tool, O=TuEmpresa S.L.
   [ Establecer como activo ]
```

A partir de ahora, cuando cualquier test se compila desde el Browser, Achilles usa este certificado para firmar el binario resultante automáticamente.

---

## Paso 4: Subir los Binarios Compilados

Con los binarios firmados, súbelos al dashboard para que estén disponibles para descarga:

```
Settings → Agent → Versiones → [+ Subir Nueva Versión]

Windows (64-bit):  achilles-agent-windows-signed.exe
Linux (64-bit):    achilles-agent-linux-amd64
macOS ARM:         achilles-agent-macos-arm64
macOS Intel:       achilles-agent-macos-amd64

Versión: 1.5.0-corp
Notas:   "Build corporativo con certificado TuEmpresa"

[ Subir ]

✅ Versión 1.5.0-corp disponible para descarga
```

Ahora cuando tus usuarios van a Settings → Agent → Descargar, descargan el binario firmado con tu certificado.

---

## Verificar que la Firma Funciona en Windows

Antes de desplegar masivamente, verifica en una máquina de prueba:

```powershell
# Verificar la firma Authenticode:
Get-AuthenticodeSignature C:\achilles\achilles-agent-windows.exe

# Resultado esperado:
# SignerCertificate : [Datos de tu certificado]
# Status            : Valid
# StatusMessage     : Signature verified.
```

Y en el Explorador de Windows:
```
Clic derecho en el .exe → Propiedades → Firma digital
→ Debería ver el nombre de tu organización
→ "La firma digital es correcta"
```

---

## Actualizar Agentes Existentes con el Nuevo Binario

Después de subir la nueva versión, los agentes existentes pueden actualizarse de forma remota:

```
Endpoints → [ Actualizar todos ] → Actualizar a v1.5.0-corp

"15 agentes serán actualizados de v1.4.2 a v1.5.0-corp"
[ Confirmar ]
```

El agente descarga el nuevo binario, verifica la firma digital, y se reinicia. Si la verificación falla, rechaza la actualización y queda en la versión anterior.

---

## Puntos Clave

✅ `make build-all` compila para Windows, Linux y macOS en un comando
✅ Windows requiere firma Authenticode — `osslsigncode` con un PFX
✅ macOS usa firma ad-hoc — `rcodesign` sin necesidad de certificado de Apple
✅ Linux no requiere firma — usa otros mecanismos de control de acceso
✅ Subir el PFX al dashboard automatiza la firma de todos los builds futuros
✅ Los agentes verifican la firma antes de aplicar actualizaciones remotas

---

## Próximo Post

**P4-03: "Compliance — Evidencia para DORA, TIBER-EU e ISO 27001"**

Cómo exportar los resultados de Achilles como evidencia para auditorías regulatorias. Qué necesita cada marco, qué aporta Achilles y cómo estructurar el paquete de evidencia.

---

**Etiquetas:** #ProjectAchilles #BuildSign #CodeSigning #Go #Authenticode #CiberSeguridad #Enterprise #DevSecOps

---

*Parte 2 de 3 en la serie "Pro — Flujos Avanzados con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
