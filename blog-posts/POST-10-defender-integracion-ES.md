# Integración con Microsoft Defender: Correlación Automática de Tests y Alertas

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 10 de 15**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- La integración con Defender requiere una **Azure AD App Registration** con `SecurityEvents.Read.All`
- Achilles sincroniza Secure Score, alertas y control profiles desde Microsoft Graph API
- La **cross-correlación** compara qué técnicas detecta Achilles vs qué alertas genera Defender
- El **Defender Tab** en Analytics muestra Secure Score + Defense Score juntos
- Todo self-hosted: las credenciales se cifran con AES-256-GCM en `~/.projectachilles/`

---

## Por Qué Esta Integración Importa

Achilles mide tu **Defense Score** (% de técnicas que detectas). Microsoft mide tu **Secure Score** (% de controles configurados). Son medidas complementarias — y frecuentemente divergen:

```
Organización típica:

Secure Score (Microsoft):  72%
"Tienes la configuración correcta según Microsoft"

Defense Score (Achilles):  58%
"Solo detectas el 58% de técnicas APT en la práctica"

Brecha: 14 puntos
→ La configuración "correcta" no garantiza detección efectiva
→ Hay exclusiones, fallos de política, gaps de reglas SIEM que solo tests reales revelan
```

La integración pone ambas métricas en el mismo dashboard para hacer esa brecha visible.

---

## Paso 1: Crear la App Registration en Azure AD

Necesitas una Azure AD Application con permisos de lectura sobre security.

### En el portal de Azure

```
1. Azure Portal → Microsoft Entra ID → App registrations
2. Click "+ New registration"
   Name: Achilles Security Integration
   Supported account types: Single tenant
   Redirect URI: (dejar vacío)
3. Click "Register"

Copia estos valores (los necesitarás en Achilles):
→ Application (client) ID:  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
→ Directory (tenant) ID:    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Configurar permisos de API

```
App Registration → API Permissions → Add a permission
→ Microsoft Graph → Application permissions

Añadir:
  ✅ SecurityEvents.Read.All    (requerido: leer alertas)
  ✅ SecurityIncident.Read.All  (recomendado: contexto de incidentes)

Click "Grant admin consent for [tu organización]"
Estado: ✅ Granted
```

⚠️ Los permisos de tipo **Application** (no Delegated) requieren **admin consent**. Sin esto, las llamadas a la API fallarán con 403.

### Crear Client Secret

```
App Registration → Certificates & secrets → Client secrets
→ "+ New client secret"
  Description: Achilles Integration
  Expires: 24 months (recomendado)
→ Click "Add"

Copia el valor AHORA — solo se muestra una vez:
Value: ~XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

---

## Paso 2: Configurar la Integración en Achilles

### Desde la UI

```
Settings → Integrations → Microsoft Defender

Tenant ID:     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Client ID:     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Client Secret: ~XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

[ Test Connection ]
→ ✅ Connected · Microsoft 365 E5 · Tenant: contoso.onmicrosoft.com
→ Secure Score: 68.5/100
→ Alertas disponibles: 1,247

[ Guardar ]
```

Las credenciales se almacenan cifradas con AES-256-GCM en `~/.projectachilles/integrations.json`. **Nunca en texto plano, nunca en la base de datos**.

### Variables de entorno (alternativa)

```bash
# Para entornos CI/CD o Docker:
DEFENDER_TENANT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
DEFENDER_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
DEFENDER_CLIENT_SECRET=~XXXXXXXXXX

# Añadir a backend/.env
```

---

## Paso 3: Sincronización de Datos

Una vez configurado, Achilles sincroniza datos de Defender en segundo plano:

```
Sincronización automática:
→ Secure Score y Control Profiles: cada 6 horas
→ Alertas: cada 5 minutos

Primera sincronización:
→ Manual: Settings → Defender → [Sincronizar ahora]
→ O espera el primer ciclo automático (~5 min para alertas)
```

### Qué datos se importan

```
Secure Score:
→ Score actual (puntos y porcentaje)
→ Evolución histórica
→ Breakdown por control profile

Control Profiles:
→ Lista de controles de Microsoft (MFA, device compliance, etc.)
→ Estado: implemented / partial / notImplemented
→ Puntos disponibles por control

Alertas:
→ Alertas activas de Microsoft Defender for Endpoint
→ Severity: high / medium / low / informational
→ Título, descripción, técnicas MITRE asociadas
→ Hosts afectados, timestamps
```

Todos los datos se almacenan en el índice ES `achilles-defender` con un campo `doc_type` que distingue entre `secure_score`, `control_profile` y `alert`.

---

## El Defender Tab en Analytics

Una vez sincronizado, aparece la pestaña Defender en el dashboard:

```
┌─────────────────────────────────────────────────────────────────────┐
│  ANALYTICS — [Overview] [Defender] [Risk Acceptance]               │
├─────────────────────────────────────────────────────────────────────┤
│  DEFENDER TAB                                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Secure Score           Defense Score       Delta                    │
│     68.5/100               73.1%          +4.6% Achilles>MS         │
│    ██████░░░░           ███████░░░                                  │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│  ALERTAS DEFENDER (últimas 24h)                                     │
│                                                                      │
│  🔴 HIGH    Suspicious PowerShell Activity   DESKTOP-SRV01  10:45  │
│  🔴 HIGH    LSASS Memory Access Detected     DESKTOP-SRV02  09:12  │
│  🟠 MEDIUM  Unusual Admin Account Usage      LINUX-PROD-01  08:33  │
│  🟡 LOW     Network Scan Detected            DESKTOP-OLD01  07:15  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## La Cross-Correlación: El Dato Más Valioso

La cross-correlación compara qué técnicas Achilles ejecutó vs cuáles generaron alerta en Defender:

```
Analytics → Defender Tab → Cross-Correlation

Tests ejecutados (últimos 30 días):    312
Tests con alerta Defender correlacionada: 189  (60.6%)
Tests SIN alerta Defender:             123  (39.4%)  ← GAPS

Técnica Overlap MITRE (técnicas en común):
→ Técnicas en tests Achilles:     47 técnicas únicas
→ Técnicas en alertas Defender:   31 técnicas únicas
→ Overlap:                        22 técnicas (46.8%)
→ Solo en Achilles (no en alertas): 25 técnicas → GAPS de Defender
→ Solo en Defender (no en tests):   9 técnicas → Actividad orgánica
```

### Technique Overlap Chart

```
Técnicas presentes en AMBOS (Achilles + Defender alertas):
T1059.001 ██████████████████████  22 matches
T1078.002 ████████████████        16 matches
T1021.001 ████████████            12 matches

Técnicas presentes SOLO en tests Achilles (gap de alerta):
T1055.001 ████████████████████    20 tests  → ❌ Defender no alertó
T1134.001 ██████████████          14 tests  → ❌ Defender no alertó
T1140     ████████████            12 tests  → ❌ Defender no alertó
```

**Esto es el insight más poderoso**: no solo sabes que tienes un gap, sino que puedes confirmar si Defender lo detectaría o no.

---

## Test vs Alert Timeline

La línea de tiempo de correlación muestra la relación temporal entre tests y alertas:

```
Timeline (24 horas): DESKTOP-SRV01

02:01:15  [Achilles] T1059.001 ejecutado
02:01:23  [Defender] ALERTA: Suspicious PowerShell Activity  ← correlacionado ✅
          Latencia de detección: 8 segundos

02:01:45  [Achilles] T1021.001 ejecutado
          (sin alerta Defender en los siguientes 5 minutos)   ← gap ❌

02:02:20  [Achilles] T1547.001 ejecutado
02:02:31  [Defender] ALERTA: Registry persistence mechanism   ← correlacionado ✅
          Latencia de detección: 11 segundos
```

La **latencia de detección** es un KPI secundario importante: ¿cuánto tarda Defender en alertar desde que ocurre la técnica?

---

## Alerts Summary: Panorama General

```
Analytics → Defender Tab → Alerts Summary

Resumen de alertas (últimos 30 días):

Severidad:  HIGH: 47  MEDIUM: 128  LOW: 312  INFO: 892
                                              
Top 5 técnicas en alertas:
T1059.001  PowerShell  ████████████████████  87 alertas
T1078.002  Valid Acc   ████████████          54 alertas
T1547.001  Persistence ████████              38 alertas
T1021.001  Lat.Move    ██████                27 alertas
T1055.001  Injection   ████                  18 alertas

Hosts más alertados:
DESKTOP-OLD01  ████████████████  42 alertas  ← revisar configuración
DESKTOP-SRV02  ████████████      31 alertas
```

---

## Secure Score en el Tiempo

```
Secure Score histórico (6 meses)

100 ┤
    │
80  ┤                                              ╭─────
    │                                    ╭─────────╯
70  ┤──────────────────────────────────╯
    │    68.5 → 68.8 → 69.1 → 70.2 → 72.4 → 73.1
60  ┤
    │
50  ┼────────────────────────────────────────────────
    Nov  Dic  Ene  Feb  Mar  Abr  May

Hitos anotados:
→ Ene: Implementamos Conditional Access (MFA) → +2.1 puntos
→ Mar: Actualizamos políticas de device compliance → +1.8 puntos
→ May: Habilitamos Advanced Hunting → +0.7 puntos
```

---

## Solución de Problemas Comunes

### Error 403: "Insufficient privileges"

```
Síntoma: La conexión test falla con 403
Causa: Admin consent no otorgado para los permisos de API

Solución:
Azure Portal → App Registration → API Permissions
→ Verifica que SecurityEvents.Read.All tenga "✅ Granted"
→ Si no: Click "Grant admin consent for [organización]"
→ Requiere rol de Global Administrator o Security Administrator
```

### Error 401: "Invalid client secret"

```
Síntoma: La conexión test falla con 401
Causa: El client secret expiró o fue ingresado incorrectamente

Solución:
Azure Portal → App Registration → Certificates & secrets
→ El secreto puede haber expirado
→ Crear uno nuevo y actualizar en Achilles Settings
```

### Alertas no aparecen o están desactualizadas

```
Síntoma: La pestaña Defender no muestra alertas recientes
Causa: Sincronización no ha corrido o hay error silencioso

Verificar:
Settings → Defender → "Última sincronización: hace X horas"
→ Si es > 10 minutos: [Sincronizar ahora] manualmente
→ Ver logs backend: docker compose logs achilles-backend | grep defender
```

---

## Puntos Clave

✅ Requiere Azure AD App Registration con `SecurityEvents.Read.All` + admin consent
✅ Credenciales cifradas localmente — nunca en texto plano
✅ Sincronización: alertas cada 5 min, scores/controls cada 6h
✅ Cross-correlación: tests Achilles vs alertas Defender por técnica MITRE
✅ Technique Overlap Chart: identifica gaps que Defender no cubriría
✅ Test vs Alert Timeline: mide latencia de detección

---

## Próximo Post

**POST 11: "Auto-Resolve — Automatiza el Cierre de Alertas de Tests en el SOC"**

Cómo usar el pillar de Auto-Resolve para que Achilles cierre automáticamente las alertas generadas por tests válidos, sin contaminar el queue del SOC.

---

**Etiquetas:** #ProjectAchilles #MicrosoftDefender #Graph #Azure #SOC #CrossCorrelation #CiberSeguridad #MITRE

---

*Parte 10 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
