# Alerting en Achilles: Notificaciones Cuando tus Defensas Fallan

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 13 de 15**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Achilles envía alertas a **Slack** (Block Kit) y/o **Email** (SMTP) cuando el Defense Score cae
- Configuras **umbrales**: % de caída relativa + suelo de score absoluto
- Las alertas se activan desde el pipeline de ingesta de resultados — en tiempo real
- La **Notification Bell** en el dashboard muestra alertas sin leer
- Útil para: detectar regresiones después de actualizaciones de OS o cambios de política EDR

---

## El Problema que Resuelve

Sin alerting, los gaps se descubren tarde:

```
Escenario sin alerting:

Lunes:    IT actualiza las políticas de grupo (GPO) en toda la flota
Martes:   Una exclusión nueva en EDR deja T1055 sin cobertura
Miércoles: Tests nocturnos detectan el gap — Defense Score cae 8%
Jueves:   El analista revisa el dashboard y ve la caída
Viernes:  IT investiga — identifican el GPO como causa
Semana siguiente: Se corrige

Tiempo de detección: 4 días

Con alerting:

Miércoles 02:15 AM: Tests detectan el gap
Miércoles 02:15 AM: Achilles envía alerta a Slack: "Defense Score cayó 8%"
Miércoles 09:00 AM: El analista ve la alerta y abre ticket
Miércoles 11:00 AM: IT identifica el GPO como causa y revierte

Tiempo de detección: 7 horas
```

---

## Configuración: Slack

### Paso 1: Crear un Incoming Webhook en Slack

```
1. Ve a https://api.slack.com/apps
2. Crea una nueva app (o usa una existente)
3. Incoming Webhooks → Activate Incoming Webhooks → ON
4. Add New Webhook to Workspace
5. Selecciona el canal: #security-alerts
6. Copia la URL del webhook:
   https://hooks.slack.com/services/<WORKSPACE_ID>/<CHANNEL_ID>/<TOKEN>
```

### Paso 2: Configurar en Achilles

```
Settings → Integrations → Alerting → Slack

Webhook URL: https://hooks.slack.com/services/<WORKSPACE_ID>/...

[ Test ] → Achilles envía un mensaje de prueba al canal
→ ✅ Mensaje de prueba enviado correctamente

[ Guardar ]
```

### Formato del mensaje Slack (Block Kit)

```
┌─────────────────────────────────────────────────────────────────┐
│ 🚨 Achilles Security Alert                                       │
├─────────────────────────────────────────────────────────────────┤
│ Defense Score cayó por encima del umbral configurado            │
│                                                                 │
│  Score anterior:  81.2%                                         │
│  Score actual:    73.4%                                         │
│  Caída:           7.8 puntos                                    │
│                                                                 │
│ Técnica con mayor impacto:                                      │
│  T1055.001 - Process Injection (23 hosts afectados)             │
│                                                                 │
│ [ Ver Dashboard ] [ Ver Gaps ]                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Configuración: Email

### Requisitos SMTP

Achilles usa Nodemailer para envío de email. Necesitas acceso SMTP:

```
Opciones comunes:
→ Gmail: smtp.gmail.com:587 (requiere App Password)
→ Outlook/Office 365: smtp.office365.com:587
→ SendGrid: smtp.sendgrid.net:587 (con API key)
→ Servidor SMTP propio: el que tengas en tu infraestructura
```

### Configuración en Achilles

```
Settings → Integrations → Alerting → Email

SMTP Host:    smtp.gmail.com
SMTP Port:    587
Usuario:      security@tuempresa.com
Contraseña:   [App Password]
TLS:          ✅ STARTTLS

Destinatarios:
  soc@tuempresa.com
  ciso@tuempresa.com

[ Test ] → Envía email de prueba a los destinatarios configurados
→ ✅ Email de prueba enviado

[ Guardar ]
```

### Formato del email

```
De: Achilles Security <security-noreply@tuempresa.com>
Para: soc@tuempresa.com
Asunto: 🚨 Achilles: Defense Score cayó 7.8% — Acción requerida

────────────────────────────────────────────────────────────────

ALERTA DE DEFENSE SCORE

Score anterior: 81.2%
Score actual:   73.4%
Caída detectada: 7.8 puntos

UMBRALES ACTIVADOS
✗ Caída relativa > 5% (actual: 7.8%)
✓ Score por encima del suelo mínimo (umbral: 60%)

TOP GAPS DETECTADOS
1. T1055.001 Process Injection           23 hosts  CRITICAL
2. T1134.001 Token Impersonation          19 hosts  HIGH
3. T1218.011 Signed Proxy Execution      17 hosts  HIGH

PRÓXIMOS PASOS SUGERIDOS
→ Revisar cambios de configuración recientes en EDR
→ Verificar actualizaciones de políticas de grupo (GPO)
→ Acceder al dashboard: https://tu-instancia.achilles.io/analytics

────────────────────────────────────────────────────────────────
Project Achilles · Validación Continua de Seguridad
```

---

## Configuración de Umbrales

Los umbrales determinan cuándo se dispara una alerta:

```
Settings → Integrations → Alerting → Thresholds

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Umbral 1: Caída porcentual relativa

  Alertar si el Defense Score cae más de: [5%] en las últimas [24h]

  Ejemplo: 81.2% → 76.0% (caída de 5.2%) → ALERTA ✓
  Ejemplo: 81.2% → 78.5% (caída de 2.7%) → sin alerta ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Umbral 2: Score mínimo absoluto

  Alertar si el Defense Score cae por debajo de: [60%]

  Ejemplo: Score = 58.3% → ALERTA CRÍTICA ✓
  Ejemplo: Score = 65.1% → sin alerta ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Canales de notificación:
  ✅ Slack  ✅ Email

[ Guardar ]
```

### Umbrales recomendados por madurez

```
Organización nueva con Achilles (primeros 3 meses):
→ Caída relativa: 10% (el score fluctúa más al principio)
→ Suelo absoluto: 50%
→ Objetivo: aprender el baseline, no inundarse de alertas

Organización con 3-6 meses usando Achilles:
→ Caída relativa: 7%
→ Suelo absoluto: 60%

Organización madura (>6 meses):
→ Caída relativa: 5%
→ Suelo absoluto: 70%
→ Objetivo: detectar cualquier regresión significativa rápidamente
```

---

## La Notification Bell

La campana de notificaciones en el dashboard muestra alertas sin leer en tiempo real:

```
Header del Dashboard:
[Project Achilles]  [Analytics]  [Endpoints]  [Browser]  🔔 3  [Settings]
                                                            ↑
                                                     3 alertas sin leer
```

Click en la campana:

```
┌──────────────────────────────────────────────────────────────┐
│ NOTIFICACIONES (3)                                 [Marcar todas leídas] │
├──────────────────────────────────────────────────────────────┤
│ 🚨 Defense Score cayó 7.8%         hace 2 horas              │
│    Score: 73.4% (era 81.2%)                                  │
│    [Ver Dashboard →]                                         │
├──────────────────────────────────────────────────────────────┤
│ ⚠️  Nuevo gap detectado: T1055.001    hace 2 horas            │
│    23 hosts afectados · CRITICAL                             │
│    [Ver Tests →]                                             │
├──────────────────────────────────────────────────────────────┤
│ ℹ️  Agente DESKTOP-OLD01 offline      hace 6 horas            │
│    Sin heartbeat desde 02:15 AM                              │
│    [Ver Endpoints →]                                         │
└──────────────────────────────────────────────────────────────┘
```

---

## Cuándo Se Disparan las Alertas

Las alertas se evalúan **inmediatamente después de cada ingesta de resultados**:

```
Timeline de una alerta:

02:01 AM  → Agente ejecuta batch de tests nocturnos
02:03 AM  → Backend ingesta 150 resultados en Elasticsearch
02:03 AM  → results.service.ts llama a alertingService.evaluate()
02:03 AM  → Defense Score recalculado: 81.2% → 73.4%
02:03 AM  → Umbral activado: caída > 5%
02:03 AM  → alertingService.dispatch()
             → slack.service.ts: POST a webhook
             → email.service.ts: SMTP send
02:04 AM  → Analista de turno recibe alerta en Slack
```

No hay polling ni delay artificial — la alerta llega en segundos.

---

## Tipos de Alerta

```
Tipo 1: Defense Score Drop
→ Disparador: score cae más de X% o debajo de umbral absoluto
→ Incluye: score anterior, score actual, top gaps

Tipo 2: New Critical Gap
→ Disparador: una técnica CRITICAL pasa de "protegido" a "gap"
→ Incluye: técnica, número de hosts afectados, recomendación

Tipo 3: Agent Offline
→ Disparador: un agente no hace heartbeat en > 5 minutos
→ Incluye: hostname, última vez visto, acciones sugeridas

Tipo 4: Test Error Spike
→ Disparador: > 10% de tests tienen exit_code 2 (error técnico)
→ Incluye: tests fallidos, posibles causas
```

---

## Desactivar Alertas en Ventanas de Mantenimiento

Si sabes que habrá cambios que afectarán el score (actualización de OS, cambio de política):

```
Settings → Alerting → Maintenance Window

[+ Nueva ventana de mantenimiento]
Inicio:  2026-05-20 22:00
Fin:     2026-05-21 06:00
Motivo:  "Actualización de Windows y nuevas políticas GPO"

→ Durante esa ventana, las alertas se suprimen
→ Los resultados de tests siguen ingresándose normalmente
→ Después de la ventana, el score nuevo es el nuevo baseline
```

---

## Puntos Clave

✅ Dos canales: Slack (Block Kit rico) y Email (SMTP con Nodemailer)
✅ Dos tipos de umbrales: caída relativa (%) y suelo absoluto
✅ Las alertas se disparan en tiempo real desde el pipeline de ingesta
✅ La Notification Bell centraliza todas las alertas en el dashboard
✅ Maintenance Windows evita alertas durante cambios planificados
✅ El alerting cierra el loop: detección de regresiones sin revisar el dashboard manualmente

---

## Próximo Post

**POST 14: "Purple Team Workflow Completo — De Intel de Amenazas a Evidencia"**

El workflow end-to-end de una campaña de purple team con Achilles: desde el advisory CISA hasta el informe de evidencia para el regulador.

---

**Etiquetas:** #ProjectAchilles #Alerting #Slack #Email #SOC #Monitoring #CiberSeguridad #PurpleTeam

---

*Parte 13 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
