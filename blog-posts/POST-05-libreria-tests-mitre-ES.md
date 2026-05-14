# La Librería de Tests de Achilles: 500+ Técnicas MITRE ATT&CK

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 5 de 15**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- La librería contiene **500+ tests** mapeados a técnicas MITRE ATT&CK v15
- Cada test es un binario Go compilado — no scripts, no PowerShell: **ejecutables firmados**
- Los tests se organizan en dos tipos: **standalone** (1 técnica) y **bundle** (múltiples controles)
- El Browser te permite filtrar por táctica, plataforma, severidad y buscar por ID de técnica
- Los tests se sincronizan desde un repositorio Git — siempre actualizados con nuevas campañas APT

---

## ¿Qué Es un Test de Achilles?

A diferencia de frameworks como Atomic Red Team (scripts PowerShell/Bash), los tests de Achilles son **binarios Go compilados y firmados**.

```
Test tradicional (Atomic Red Team):
→ Script PowerShell .ps1
→ Puede ser bloqueado por AMSI
→ Sin firma de código
→ Fácil de detectar por AV/EDR como "script sospechoso"

Test de Achilles:
→ Binario Go compilado (EXE firmado en Windows)
→ Simula comportamiento de APT real
→ Firmado con Authenticode (Windows) o ad-hoc (macOS)
→ Representa tradecraft más realista
```

**El objetivo es medir detección real**, no simplemente que tu AV bloquee un script obviamente malicioso.

---

## Anatomía de un Test

Cada test en la librería tiene los siguientes metadatos:

```json
{
  "id": "T1059.001-powershell-exec",
  "name": "PowerShell Execution - Encoded Command",
  "technique_id": "T1059.001",
  "technique_name": "Command and Scripting Interpreter: PowerShell",
  "tactic": "Execution",
  "severity": "high",
  "platform": ["windows"],
  "description": "Simula ejecución de PowerShell con comando codificado en base64, técnica común en loaders APT.",
  "mitre_url": "https://attack.mitre.org/techniques/T1059/001/",
  "source": "CISA AA23-187A, APT29 campaign",
  "type": "standalone"
}
```

### Campos clave

| Campo | Descripción |
|---|---|
| `technique_id` | ID MITRE ATT&CK (e.g. T1059.001) |
| `tactic` | Táctica MITRE (Execution, Persistence...) |
| `severity` | critical / high / medium / low |
| `platform` | windows / linux / darwin |
| `type` | standalone / bundle |
| `source` | Origen: CISA advisory, threat intel report, etc. |

---

## Las 11 Tácticas MITRE Cubiertas

```
┌──────────────────────────────────────────────────────────────────┐
│                    COBERTURA POR TÁCTICA                         │
├──────────────────────────────────────────────────────────────────┤
│  Initial Access        ████████████  32 tests                    │
│  Execution             █████████████████████  67 tests           │
│  Persistence           ████████████████  48 tests                │
│  Privilege Escalation  ██████████  31 tests                      │
│  Defense Evasion       ████████████████████████  78 tests        │
│  Credential Access     ████████████  38 tests                    │
│  Discovery             ████████████████  52 tests                │
│  Lateral Movement      ████████████  37 tests                    │
│  Collection            ████████████  39 tests                    │
│  Exfiltration          ████████  25 tests                        │
│  Impact                ██████████  30 tests                      │
└──────────────────────────────────────────────────────────────────┘
```

---

## El Browser: Cómo Navegar la Librería

### Filtros disponibles

```
┌─────────────────────────────────────────────────────────────────┐
│  BROWSER DE TESTS                                               │
├─────────────────────────────────────────────────────────────────┤
│  Buscar: [T1059___________________]                             │
│                                                                 │
│  Táctica: [Todas ▼]    Plataforma: [Windows ▼]                 │
│  Severidad: [Critical ▼]    Tipo: [Standalone ▼]               │
│                                                                 │
│  Resultados: 23 tests                                           │
├─────────────────────────────────────────────────────────────────┤
│  T1059.001  PowerShell Exec - Encoded        HIGH  Windows      │
│  T1059.001  PowerShell - AMSI Bypass         CRITICAL Windows   │
│  T1059.003  Windows Command Shell            HIGH  Windows      │
│  T1059.004  Unix Shell - Reverse Connection  HIGH  Linux        │
│  T1059.005  VBScript Execution               MEDIUM Windows     │
│  ...                                                            │
└─────────────────────────────────────────────────────────────────┘
```

### Ver detalles de un test

```
T1059.001 - PowerShell Execution (Encoded Command)
══════════════════════════════════════════════════

Táctica:    Execution
Severidad:  HIGH  
Plataforma: Windows x64
Tipo:       Standalone

Descripción:
Simula la ejecución de PowerShell con un comando codificado en base64,
patrón observado en campañas de Cobalt Strike, Emotet y APT29.

Técnica MITRE: https://attack.mitre.org/techniques/T1059/001/

Fuente: CISA Advisory AA23-187A (May 2023)

Qué debería detectar tu stack:
→ Event ID 4104 (Script Block Logging)
→ Event ID 4688 (Process Creation con -EncodedCommand)
→ Regla Sigma: proc_creation_win_powershell_encoded_cmd.yml

[  Compilar  ]  [  Asignar a Agente  ]  [  Ver Código Fuente  ]
```

---

## Tests Standalone vs Bundle

### Tests Standalone

Un test standalone valida **una sola técnica MITRE**:

```
Nombre:    T1021.001-rdp-brute
Técnica:   T1021.001 (Remote Desktop Protocol)
Táctica:   Lateral Movement
Ejecuta:   1 binario → 1 resultado → 1 documento en ES

Ideal para:
→ Validar una regla SIEM específica
→ Responder "¿detectamos THIS técnica?"
→ Tests puntuales durante hardening
```

### Bundle Tests

Un bundle valida **múltiples controles relacionados** en una sola ejecución:

```
Bundle: cyber-hygiene
ID: 7659eeba-f315-440e-9882-4aa015d68b27

Controles:
  CH-DEF-001  Credential Guard habilitado        CRITICAL
  CH-DEF-002  LSASS Protection activada          CRITICAL
  CH-DEF-003  ASR Rules configuradas             HIGH
  CH-DEF-004  PowerShell Constrained Language    HIGH
  CH-IEP-001  Script Block Logging activo        MEDIUM
  CH-IEP-002  Module Logging activo              MEDIUM
  CH-IEP-003  Transcription Logging activo       LOW
  ...

Resultado: X/Y controles protected
           Documento ES por cada control individual
```

**¿Cuándo usar cada uno?**

```
Standalone → Validación quirúrgica de una técnica específica
Bundle     → Auditoría completa de una categoría de controles
             (cyber-hygiene, identity protection, network security...)
```

---

## Categorías de Bundles Disponibles

```
cyber-hygiene          → Controles de hardening de Windows
identity-endpoint      → Protección de credenciales y cuentas
network-security       → Detección de movimiento lateral y tráfico
apt29-campaign         → Simulación de campaña APT29 (Cozy Bear)
apt41-campaign         → Simulación de campaña APT41 (doble nexo)
ransomware-baseline    → Controles antiransomware esenciales
```

---

## Cómo Están Organizados los Tests en Git

La librería vive en el repositorio `f0_library` y se sincroniza automáticamente:

```
f0_library/
├── standalone/
│   ├── T1059/
│   │   ├── T1059.001-powershell-exec/
│   │   │   ├── main.go          → Código del test
│   │   │   ├── metadata.json    → Metadatos MITRE
│   │   │   └── README.md        → Qué detecta y cómo
│   │   └── T1059.003-cmd-shell/
│   │       └── ...
│   └── T1021/
│       └── ...
└── bundles/
    ├── cyber-hygiene/
    │   ├── build_all.sh         → Script de build multi-binario
    │   ├── metadata.json        → Metadatos del bundle
    │   ├── CH-DEF-001/
    │   │   └── main.go
    │   └── CH-DEF-002/
    │       └── main.go
    └── apt29-campaign/
        └── ...
```

### Sincronización automática

```bash
# El backend hace git pull periódicamente
# También puedes forzar sincronización desde la UI:
# Settings → Browser → [Sincronizar Librería]

# Estado de sincronización visible en:
# Browser → última actualización: hace 2 horas
# 512 tests indexados
```

---

## El Código de un Test: Por Dentro

Para entender qué ejecutas, aquí hay un ejemplo simplificado de cómo luce el código de un test:

```go
// T1059.001 - PowerShell Execution
// Simula ejecución de PowerShell con comando codificado
package main

import (
    "encoding/base64"
    "os"
    "os/exec"
)

func main() {
    // Comando inofensivo codificado en base64
    // (en tests reales: payload de simulación controlada)
    cmd := "Write-Output 'Achilles-Test-Execution'"
    encoded := base64.StdEncoding.EncodeToString([]byte(cmd))

    // Ejecuta PowerShell con -EncodedCommand (la técnica real que usaría un APT)
    out, err := exec.Command(
        "powershell.exe",
        "-EncodedCommand", encoded,
    ).CombinedOutput()

    if err != nil {
        // Si PowerShell no pudo ejecutar → posible protección activa
        os.Exit(1)  // Señal: DETECTADO/BLOQUEADO
    }

    _ = out
    os.Exit(0)  // Señal: ejecutado sin bloqueo (GAP en defensa)
}
```

**Convención de exit codes:**

```
exit_code 0 → El test se ejecutó sin que la defensa lo bloqueara
              Interpretación: GAP de detección (el ataque pasaría)
              
exit_code 1 → El test fue bloqueado o no pudo ejecutar
              Interpretación: PROTEGIDO (la defensa funcionó)
              
exit_code 2 → Error en el propio test (problema técnico)
              No cuenta para el Defense Score
```

⚠️ **Nota importante**: `exit_code 0` significa que el test se ejecutó — no significa que la defensa falló en cada caso. Para algunos tests, el comportamiento es inofensivo y la "detección" se mide por las alertas que genera en el SIEM, no por si el proceso fue bloqueado.

---

## Estrategia de Testing: ¿Por Dónde Empezar?

### Para Blue Teams (SOC / Defenders)

```
Prioridad 1: Cyber-hygiene bundle
→ Valida controles básicos de hardening
→ Bajo riesgo, alto valor de información

Prioridad 2: Técnicas con mayor prevalencia en tu sector
→ Banca: T1059, T1078, T1021
→ Salud: T1486 (ransomware), T1190, T1133
→ Gobierno: T1566 (phishing), T1078, T1071

Prioridad 3: Técnicas de campañas activas
→ Filtra por fuente: "CISA AA23-..." o "TIBER 2025"
→ Tests directamente derivados de campañas reales
```

### Para Red Teams (Pentesters / Purple Team)

```
Usar Achilles como benchmark previo al pentest:
1. Ejecuta suite completa antes del engagement
2. Documenta Defense Score base: 68%
3. Ejecuta pentest tradicional
4. Post-engagement: vuelve a ejecutar suite
5. Mide mejora: 68% → 79% (+11%)

El Defense Score da evidencia objetiva del valor del pentest.
```

---

## Puntos Clave

✅ 500+ tests mapeados a MITRE ATT&CK v15 — binarios Go compilados y firmados
✅ Dos tipos: standalone (1 técnica) y bundle (múltiples controles)
✅ `exit_code 0` = ejecutado sin bloqueo, `exit_code 1` = bloqueado/detectado
✅ Librería sincronizada desde Git — nuevos tests con cada advisory CISA
✅ Filtros en el Browser por táctica, plataforma, severidad y tipo
✅ Cada test documenta qué regla SIEM o Event ID debería activarse

---

## Próximo Post

**POST 06: "Tu Primer Test de Seguridad con Achilles"**

Paso a paso: desde seleccionar un test en el Browser hasta ver el resultado en el dashboard de Analytics.

---

**Etiquetas:** #ProjectAchilles #MITRE #ATTandCK #PurpleTeam #BlueTeam #RedTeam #CiberSeguridad #SOC

---

*Parte 5 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
