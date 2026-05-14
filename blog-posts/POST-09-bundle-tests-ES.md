# Bundle Tests: Valida 30 Controles de Seguridad en una Sola Ejecución

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 9 de 15**

**Tiempo de lectura:** 9 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Los **bundle tests** ejecutan múltiples controles de seguridad en una sola tarea
- El resultado es un `bundle_results.json` que se "fan-out" en documentos ES individuales por control
- En la UI aparecen como filas colapsables: "X/Y Protected" con drill-down por control
- Ideal para auditorías de hardening completas (cyber-hygiene, identity, ransomware baseline)
- Un bundle de 30 controles te da en 5 minutos lo que un auditor tarda días en verificar

---

## ¿Por Qué Bundle Tests?

Los tests standalone son perfectos para validar una técnica específica. Pero a veces necesitas responder preguntas más amplias:

```
Pregunta: "¿Están nuestros Windows hardeneados según el CIS Benchmark?"

Con standalone tests:
→ Asignar 23 tests individuales uno por uno
→ Esperar 23 ejecuciones separadas
→ Correlacionar 23 resultados manualmente
→ Tiempo: 30-60 minutos de trabajo manual

Con bundle tests:
→ Asignar 1 bundle: cyber-hygiene
→ Se ejecutan 23 controles en secuencia
→ El resultado aparece agrupado en la UI
→ Tiempo: 5 minutos, cero trabajo manual
```

---

## Anatomía de un Bundle

Un bundle es un conjunto de controles relacionados que se ejecutan como una unidad:

```
Bundle: cyber-hygiene
ID: 7659eeba-f315-440e-9882-4aa015d68b27
Nombre: Windows Cyber Hygiene Baseline
Plataforma: Windows

Controles:
┌──────────────────────────────────────────────────────────────────┐
│ ID          Nombre                              Severidad        │
├──────────────────────────────────────────────────────────────────┤
│ CH-DEF-001  Credential Guard habilitado         CRITICAL         │
│ CH-DEF-002  LSASS Protection (PPL) activa       CRITICAL         │
│ CH-DEF-003  ASR Rules configuradas              HIGH             │
│ CH-DEF-004  PowerShell Constrained Language     HIGH             │
│ CH-IEP-001  Script Block Logging activo         MEDIUM           │
│ CH-IEP-002  Module Logging activo               MEDIUM           │
│ CH-IEP-003  Transcription Logging activo        LOW              │
│ CH-NET-001  SMB Signing habilitado              CRITICAL         │
│ CH-NET-002  LLMNR deshabilitado                 HIGH             │
│ CH-NET-003  NetBIOS deshabilitado               HIGH             │
│ CH-APP-001  AppLocker/WDAC configurado          CRITICAL         │
│ ... (23 controles total)                                         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Cómo Funciona la Ejecución

### El flujo técnico

```
1. Backend compila el bundle:
   → build_all.sh genera un binario Go por cada control
   → Todos firmados si hay certificado activo
   → Se empaquetan en un ZIP firmado

2. Agente descarga y ejecuta el bundle:
   → Extrae los binarios en un directorio temporal
   → Ejecuta cada control en secuencia
   → Cada control escribe su resultado

3. Resultado consolidado:
   → El orquestador escribe bundle_results.json
   → Agente lee el archivo y valida que el bundle_id coincide
   → Agente envía el JSON completo al backend

4. Backend "fan-out":
   → Detecta bundle_results.controls en el payload
   → Crea UN documento ES por cada control individual
   → Cada documento tiene su propio exit_code, técnicas, severity
```

### El archivo `bundle_results.json`

```json
{
  "bundle_id": "7659eeba-f315-440e-9882-4aa015d68b27",
  "bundle_name": "Windows Cyber Hygiene Baseline",
  "executed_at": "2026-05-14T02:01:00Z",
  "controls": [
    {
      "control_id": "CH-DEF-001",
      "control_name": "Credential Guard habilitado",
      "exit_code": 1,
      "techniques": ["T1003.001"],
      "severity": "critical",
      "output": "Credential Guard: Enabled (enforced)"
    },
    {
      "control_id": "CH-DEF-002",
      "control_name": "LSASS Protection activa",
      "exit_code": 0,
      "techniques": ["T1003.001"],
      "severity": "critical",
      "output": "RunAsPPL: 0 (not configured)"
    },
    ...
  ]
}
```

---

## La Executions Table: Resultados Agrupados

En la UI de Analytics, los bundle tests aparecen de forma especial:

### Vista colapsada (por defecto)

```
┌─────────────────────────────────────────────────────────────────────┐
│ EXECUTIONS TABLE                                                     │
├──────┬────────────────────────────┬───────────────┬─────────────────┤
│  ▶   │ cyber-hygiene bundle       │ 17/23 🛡️      │ DESKTOP-SRV01   │
│      │ Windows Cyber Hygiene      │ Protected     │ hace 2 horas    │
├──────┼────────────────────────────┼───────────────┼─────────────────┤
│  ▶   │ apt29-campaign bundle      │ 12/19 🛡️      │ DESKTOP-SRV01   │
│      │ APT29 Campaign Simulation  │ Protected     │ hace 2 horas    │
├──────┼────────────────────────────┼───────────────┼─────────────────┤
│ —    │ T1059.001-powershell-exec  │ ✅ Protegido   │ DESKTOP-SRV01   │
│      │ (standalone)               │               │ hace 3 horas    │
└──────┴────────────────────────────┴───────────────┴─────────────────┘
```

### Vista expandida (click en ▶)

```
├──────┬────────────────────────────────────────────────────────────────┤
│  ▼   │ cyber-hygiene bundle  [17/23 controles] 🛡️ Protected          │
├──────┼────────────────────────────────────────────────────────────────┤
│      │   ✅  CH-DEF-001  Credential Guard habilitado      CRITICAL    │
│      │   ❌  CH-DEF-002  LSASS Protection (PPL) activa   CRITICAL    │
│      │   ✅  CH-DEF-003  ASR Rules configuradas           HIGH        │
│      │   ✅  CH-DEF-004  PowerShell Constrained Language  HIGH        │
│      │   ✅  CH-IEP-001  Script Block Logging activo      MEDIUM      │
│      │   ✅  CH-IEP-002  Module Logging activo             MEDIUM      │
│      │   ✅  CH-IEP-003  Transcription Logging activo     LOW         │
│      │   ❌  CH-NET-001  SMB Signing habilitado           CRITICAL    │
│      │   ✅  CH-NET-002  LLMNR deshabilitado               HIGH        │
│      │   ❌  CH-NET-003  NetBIOS deshabilitado             HIGH        │
│      │   ❌  CH-APP-001  AppLocker/WDAC configurado       CRITICAL    │
│      │   ... (12 más)                                                 │
└──────┴────────────────────────────────────────────────────────────────┘
```

**El badge `17/23 Protected`** te da el resumen ejecutivo. Los sub-rows te dan el detalle técnico exacto de qué corregir.

---

## Tipos de Bundles Disponibles

### cyber-hygiene (Windows)
```
Qué valida: hardening básico de Windows
Número de controles: 23
Casos de uso: auditoría inicial, benchmark CIS L1
Tiempo de ejecución: ~3 minutos
```

### identity-endpoint
```
Qué valida: protección de credenciales y cuentas privilegiadas
Controles incluidos: Credential Guard, LSASS PPL, Protected Users,
                     PAW policy, Admin Tier Model
Casos de uso: auditoría de Active Directory, pre-red-team
Tiempo de ejecución: ~5 minutos
```

### ransomware-baseline
```
Qué valida: controles antiransomware esenciales
Controles incluidos: VSS protection, backup integrity,
                     Controlled Folder Access, network share ACLs
Casos de uso: evaluación de resiliencia frente a ransomware
Tiempo de ejecución: ~4 minutos
```

### apt29-campaign
```
Qué valida: técnicas usadas por APT29 (Cozy Bear / SVR)
Controles incluidos: 19 etapas del ataque desde Initial Access hasta Exfil
Tipo: multi-stage intel-driven (no cyber-hygiene)
Badge muestra: "X/Y stages" (no "X/Y controls")
Casos de uso: simulación de amenaza dirigida
```

---

## Diferencia: cyber-hygiene vs intel-driven bundles

Hay un detalle importante en cómo se interpretan los resultados:

### Bundles cyber-hygiene

```
Cada control valida un estado de configuración:
→ ¿Está habilitado Credential Guard?
→ ¿Está configurado SMB Signing?

exit_code 0 = configuración correcta ✅ (protegido)
exit_code 1 = configuración ausente ❌ (gap)

Nota: la lógica está invertida vs técnicas individuales.
En cyber-hygiene, 0 = bien configurado = protegido.
```

### Bundles intel-driven (APT29, APT41...)

```
Cada etapa ejecuta la técnica de ataque real:
→ Stage 1: Initial Access via spear-phishing attachment
→ Stage 2: Execution via PowerShell download cradle
→ Stage 3: Persistence via registry run key
...

exit_code 0 = técnica ejecutada sin detección ❌ (gap)
exit_code 1 = técnica detectada/bloqueada ✅ (protegido)

Algunas etapas pueden ser "Skipped" si dependen de etapas anteriores.
Los stages skipped no cuentan en el score.
```

---

## Composite test_uuid: Identificar Controles Individuales

Cada control dentro de un bundle tiene un identificador único compuesto:

```
Formato: <bundle-uuid>::<control-id>

Ejemplo:
7659eeba-f315-440e-9882-4aa015d68b27::CH-DEF-002

→ Este es el test_uuid del documento ES para ese control específico
→ Puedes buscar este ID exacto en Elasticsearch para ver el documento completo

curl http://localhost:9200/achilles-results/_search \
  -d '{ "query": { "match": { 
    "f0rtika.test_uuid": "7659eeba-f315-440e-9882-4aa015d68b27::CH-DEF-002"
  }}}'
```

---

## Ejecutar un Bundle: Paso a Paso

```
1. Browser → Bundles → selecciona "cyber-hygiene"
2. Click [Compilar Bundle]
   → Build de 23 binarios + ZIP firmado (~2 minutos)
3. [Asignar a Agente]
   → Destino: DESKTOP-SRV01
   → Horario: Esta noche, 01:00 AM
4. El agente ejecuta todos los controles en secuencia
5. Analytics → Executions Table → resultado agrupado
```

---

## Interpretar Resultados del Bundle para Hardening

```
Bundle cyber-hygiene resultado: 17/23 Protected

Gaps identificados (6 controles fallidos):
CRITICAL: CH-DEF-002 - LSASS PPL no configurado
CRITICAL: CH-NET-001 - SMB Signing deshabilitado
CRITICAL: CH-APP-001 - AppLocker/WDAC no configurado
HIGH:     CH-NET-003 - NetBIOS activo
HIGH:     CH-LAN-002 - WDigest Authentication activo
MEDIUM:   CH-MON-003 - Audit Policy incompleta

Plan de remediación priorizado:
┌────────────────────────────────────────────────────────────────┐
│ 1. [HOY]    CH-DEF-002: reg add HKLM\...\LSASS /v RunAsPPL   │
│             /t REG_DWORD /d 1                                  │
│ 2. [ESTA SEMANA] CH-NET-001: Set-SmbServerConfiguration       │
│                  -RequireSecuritySignature $true               │
│ 3. [ESTA SEMANA] CH-APP-001: Desplegar GPO con WDAC policy    │
│ 4. [PRÓXIMO MES] CH-APP-001+: WDAC Managed Installer rule     │
└────────────────────────────────────────────────────────────────┘
```

---

## Bundle Tests en Auditorías DORA/ISO

Los bundles son especialmente valiosos para evidencia de auditoría:

```
Evidencia para auditor DORA Art. 25:
"Ejecutamos el bundle 'cyber-hygiene' el 2026-05-14 contra 23 endpoints.
Resultado: 17/23 controles validados como protegidos (73.9%).
Los 6 gaps están documentados con plan de remediación adjunto.
Re-ejecución programada post-remediación para verificar corrección."

→ Evidencia técnica, objetiva, reproducible
→ No "tenemos el control implementado" sino "aquí está la prueba"
```

---

## Puntos Clave

✅ Bundle tests = múltiples controles en 1 ejecución → ahorro masivo de tiempo
✅ El resultado se "fan-out" en documentos ES individuales — granularidad total
✅ La Executions Table agrupa bundle results en filas colapsables
✅ cyber-hygiene bundles: exit_code 0 = protegido (lógica de hardening)
✅ intel-driven bundles: exit_code 0 = gap (lógica de técnica de ataque)
✅ Composite test_uuid (`bundle::control`) permite búsquedas exactas en ES

---

## Próximo Post

**POST 10: "Integración con Microsoft Defender — Correlación Automática"**

Cómo conectar Achilles con el Microsoft Graph API para correlacionar tus test results con las alertas reales de Defender.

---

**Etiquetas:** #ProjectAchilles #BundleTests #MITRE #CiberSeguridad #Hardening #CIS #DORA #PurpleTeam

---

*Parte 9 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
