# Build & Sign: Compilando Agentes Firmados para 6 Plataformas

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 12 de 15**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- Achilles compila binarios Go para **6 targets** (Windows/Linux/macOS × amd64/arm64)
- Windows usa **Authenticode** con certificado PFX (via `osslsigncode`)
- macOS usa **firma ad-hoc** (via `rcodesign`) — no requiere Apple Developer ID
- Los certificados se gestionan en `~/.projectachilles/certs/` (máximo 5, rotación automática)
- Los binarios sin firma funcionan, pero los EDR los tratarán con más suspicacia — lo cual es útil información

---

## Por Qué Importa la Firma de Código en Testing

Los tests de Achilles simulan APTs reales. Los APTs reales firman sus binarios:

```
Binario sin firma:
→ Windows SmartScreen bloquea al primer uso
→ Defender puntúa como "suspicious unsigned binary"
→ Muchos EDRs añaden telemetría adicional
→ Resultado: detectarías el binario por ser unsigned, no por su comportamiento

Binario firmado (Authenticode):
→ Pasa el filtro de SmartScreen
→ Los EDRs evalúan el comportamiento, no la firma
→ Simula mejor el comportamiento de un malware real
→ Resultado: measurement más preciso de tu cobertura conductual
```

La firma no es evasión — es **metodología de medición correcta**.

---

## Los 6 Targets de Compilación

```
GOOS=windows GOARCH=amd64 → achilles-agent-windows-amd64.exe  (firma Authenticode)
GOOS=linux   GOARCH=amd64 → achilles-agent-linux-amd64        (sin firma)
GOOS=darwin  GOARCH=amd64 → achilles-agent-darwin-amd64       (firma ad-hoc)
GOOS=darwin  GOARCH=arm64 → achilles-agent-darwin-arm64       (firma ad-hoc)
GOOS=windows GOARCH=arm64 → achilles-agent-windows-arm64.exe  (firma Authenticode)
GOOS=linux   GOARCH=arm64 → achilles-agent-linux-arm64        (sin firma)
```

Go hace cross-compilation nativa — no necesitas máquinas Windows/macOS para compilar para esas plataformas.

```bash
# Ejemplo: compilar el agente para todos los targets
cd agent && make build-all

# O individual:
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 \
  go build -ldflags="-X main.version=1.4.2" \
  -o dist/achilles-agent-windows-amd64.exe .
```

`CGO_ENABLED=0` produce un binario estático sin dependencias — funciona en cualquier Windows sin instalar nada.

---

## Paso 1: Obtener un Certificado de Firma

### Para Windows (Authenticode)

Necesitas un certificado de firma de código:

```
Opciones:
A) Certificado comercial OV/EV (recomendado para producción)
   → DigiCert, Sectigo, GlobalSign
   → Coste: $300-500/año
   → EV requiere validación de organización
   → Mayor confianza en SmartScreen

B) Certificado self-signed (para lab/testing interno)
   → Gratis, sin validación
   → SmartScreen lo bloqueará en máquinas no configuradas
   → Funciona si añades el cert como Trusted Publisher en las máquinas de test

C) Sin certificado
   → Achilles compila igualmente sin firma
   → Útil para medir cuántos endpoints bloquean unsigned binaries
```

### Para macOS (ad-hoc)

macOS usa firma ad-hoc — **no requiere Apple Developer ID**:

```bash
# rcodesign firma in-place sin certificado externo
rcodesign sign --code-signature-flags adhoc achilles-agent-darwin-arm64
```

La firma ad-hoc satisface las restricciones de Gatekeeper en modo de testing pero no en distribución pública.

---

## Paso 2: Subir el Certificado a Achilles

```
Settings → Tests → Certificates → [+ Nuevo Certificado]

Archivo PFX:  [Seleccionar archivo...]
Contraseña:   [**************]
Nombre:       "Certificado EV 2026 - DigiCert"

[ Subir ]
→ ✅ Certificado válido
→ Common Name: Achilles Security Testing
→ Expira: 2027-06-15
→ Almacenado en: ~/.projectachilles/certs/cert-1715000000/
```

### Gestión de múltiples certificados

Achilles soporta hasta 5 certificados simultáneos:

```
Settings → Tests → Certificates

[★ Activo] Certificado EV 2026     Expira: 2027-06-15  ✅ Válido
[  ] Certificado OV 2025            Expira: 2026-08-20  ✅ Válido  
[  ] Certificado Self-Signed       Expira: 2027-01-01  ✅ Válido

[ Establecer como activo ] [ Descargar ] [ Eliminar ]
```

El certificado **activo** es el que se usa en todos los builds nuevos. Cuando expire, activas el siguiente sin interrupción.

---

## Cómo Funciona el Build Pipeline

### Para tests standalone

```
Browser → T1059.001 → [Compilar]

Build service:
1. go build -o T1059.001-win-amd64.exe (GOOS=windows GOARCH=amd64)
2. Si hay certificado activo y plataforma es Windows:
   osslsigncode sign \
     -pkcs12 ~/.projectachilles/certs/cert-active/cert.pfx \
     -pass-file /tmp/cert-pass-XXXXX \      ← archivo temporal, permisos 0600
     -in T1059.001-win-amd64.exe \
     -out T1059.001-win-amd64-signed.exe
   rm /tmp/cert-pass-XXXXX                  ← limpieza en finally block
3. Si plataforma es macOS:
   rcodesign sign --code-signature-flags adhoc <binary>
4. Binario firmado disponible para descarga o asignación directa
```

### Para bundles multi-binario

Los bundles más complejos (cyber-hygiene, identity-endpoint) tienen una arquitectura multi-binario:

```
bundle cyber-hygiene/
├── build_all.sh         ← orquestador del build
├── CH-DEF-001/
│   └── main.go
├── CH-DEF-002/
│   └── main.go
└── ... (23 controles)

build_all.sh:
  for each control in bundle:
    go build -o binaries/control_id.exe ./control_id/
    osslsigncode sign ... (si hay certificado)
  zip -r bundle.zip binaries/
```

El backend pasa el certificado al build via variables de entorno:

```bash
F0_SIGN_CERT_PATH=/path/to/cert.pfx
F0_SIGN_CERT_PASS_FILE=/tmp/cert-pass-XXXXX
```

El script puede entonces firmar cada binario individual antes de empaquetar.

---

## El Agent Build: Versionar el Agente

El flujo de construcción del agente es similar pero incluye versionado:

```bash
# Settings → Agent → [Compilar nueva versión]

VERSION=1.4.3
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 \
  go build \
  -ldflags="-X main.version=${VERSION} -X main.serverUrl=${AGENT_SERVER_URL}" \
  -o dist/achilles-agent-windows-amd64-${VERSION}.exe

# Firma
osslsigncode sign \
  -pkcs12 ${CERT_PATH} \
  -pass-file ${PASS_FILE} \
  -in dist/achilles-agent-windows-amd64-${VERSION}.exe \
  -out dist/achilles-agent-windows-amd64-${VERSION}-signed.exe
```

La versión se inyecta via LDFLAGS en tiempo de compilación — no hay archivo de configuración, el binario sabe su propia versión.

---

## Verificar que el Binario Está Firmado

### Windows: Verificar firma Authenticode

```powershell
# PowerShell:
Get-AuthenticodeSignature .\achilles-agent-windows-amd64.exe

SignerCertificate : [Subject: CN=Achilles Security Testing, O=...]
Status            : Valid
StatusMessage     : Signature verified.
```

### macOS: Verificar firma ad-hoc

```bash
codesign -dv achilles-agent-darwin-arm64
# Executable=./achilles-agent-darwin-arm64
# Identifier=achilles-agent-darwin-arm64
# Format=Mach-O thin (arm64)
# CodeDirectory v=20400 size=... flags=0x2(adhoc)
# Signature=adhoc
```

La firma ad-hoc tiene el flag `adhoc` — no hay un certificado de Identidad de Desarrollador Apple, pero el binario tiene integridad verificable.

---

## Errores de Firma No Son Fatales

Una filosofía importante del build system de Achilles:

```
Si osslsigncode falla:
→ El build CONTINÚA con el binario sin firma
→ Se registra un warning en el build log
→ El binario sin firma es descargable/asignable igualmente

Por qué:
→ Un fallo de firma no debería bloquear una campaña de testing
→ Los tests con binario unsigned son igualmente válidos (con caveat)
→ El analista puede decidir si continuar o resolver el cert issue primero
```

El caveat: si un EDR bloquea el binario unsigned, `exit_code 2` (error) aparece en los resultados. Eso en sí mismo es información: tu EDR bloquea binarios sin firma de código, lo cual puede ser una política válida.

---

## Rotación de Certificados

Cuando el certificado activo está próximo a expirar:

```
1. Subir nuevo certificado (Settings → Tests → Certificates → [+ Nuevo])
2. El nuevo certificado se añade a la lista (max 5)
3. Click "Establecer como activo"
4. Los nuevos builds usan el nuevo certificado automáticamente
5. Los binarios ya compilados con el certificado anterior siguen siendo válidos

Zero downtime:
→ Los agentes existentes no necesitan ser actualizados por un cambio de cert
→ Solo los nuevos binarios compilados post-rotación usan el nuevo cert
→ Los old binaries siguen funcionando hasta que los recompiles
```

---

## Puntos Clave

✅ 6 targets de compilación: Windows/Linux/macOS × amd64/arm64
✅ Compilación cruzada nativa en Go — no necesitas VMs de cada OS
✅ Windows: Authenticode con certificado PFX (osslsigncode)
✅ macOS: firma ad-hoc gratuita (rcodesign, sin Apple Developer ID)
✅ La firma no es evasión — es medición correcta (igual que haría un APT real)
✅ Fallos de firma son no-fatales: el build continúa, el analista decide

---

## Próximo Post

**POST 13: "Alerting — Notificaciones cuando tus Defensas Fallan"**

Cómo configurar alertas de Slack y email cuando el Defense Score cae o se detectan gaps críticos.

---

**Etiquetas:** #ProjectAchilles #CodeSigning #Go #CrossCompilation #Authenticode #Build #CiberSeguridad #DevSecOps

---

*Parte 12 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
