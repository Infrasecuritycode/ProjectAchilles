# Alerting: Que Achilles Te Avise Cuando Algo Falla

> **Serie: Usando Achilles en Profundidad — Parte 3 de 4**

**Tiempo de lectura:** 6 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Achilles puede enviarte alertas a **Slack** o **email** cuando el Defense Score cae o se detecta un gap crítico
- Se configura en menos de 10 minutos
- Puedes establecer dos umbrales: caída porcentual (ej: "avísame si cae más del 5%") y suelo mínimo (ej: "avísame si baja de 60%")
- Las alertas llegan en tiempo real, en segundos después de detectar el cambio
- Con alertas configuradas no necesitas revisar el dashboard todos los días

---

## Por Qué las Alertas Son Importantes

Sin alertas, el flujo es así:

```
Tests automáticos se ejecutan lunes 02:00 AM
↓
Un cambio de política de IT afectó la configuración del EDR
↓
El Defense Score cayó 8%
↓
...nadie lo sabe hasta que alguien revise el dashboard
↓
Jueves: el analista abre el dashboard y ve la caída
↓
Viernes: se investiga la causa
↓
Lunes siguiente: se corrige
Tiempo total: 7 días
```

Con alertas:

```
Tests ejecutados lunes 02:00 AM
↓
Defense Score cayó 8%
↓
02:03 AM: alerta en Slack: "Defense Score cayó 8% esta noche"
↓
Lunes 09:00 AM: el analista ve el mensaje y abre ticket
↓
Mismo día: se investiga y corrige
Tiempo total: horas, no días
```

---

## Configurar Alertas de Slack

### Paso 1: Crear el webhook en Slack (5 minutos)

Un webhook es una URL especial que permite a aplicaciones externas enviar mensajes a tu canal de Slack.

```
1. Ve a https://api.slack.com/apps
2. Click "Create New App" → "From scratch"
   Nombre: Achilles Security
   Workspace: el tuyo
3. En el menú izquierdo: "Incoming Webhooks"
4. Activa el toggle: "Activate Incoming Webhooks" → ON
5. Click "Add New Webhook to Workspace"
6. Selecciona el canal: #seguridad-alertas (o el que prefieras)
7. Click "Allow"

Copia la URL que aparece:
https://hooks.slack.com/services/<tu-workspace>/<tu-canal>/<tu-token>
```

### Paso 2: Configurar en Achilles

```
Settings → Integrations → Alerting → Slack

Webhook URL: [pega la URL copiada]

[ Probar conexión ]
→ Debería llegar un mensaje de prueba a tu canal de Slack ✓

[ Guardar ]
```

### Cómo luce una alerta en Slack

```
🚨 Achilles Security Alert

Defense Score cayó por encima del umbral configurado

  Score anterior:  81%
  Score actual:    73%
  Caída:           8 puntos

Técnica con más impacto:
  Process Injection — 23 máquinas afectadas

[ Ver Dashboard ]  [ Ver Gaps ]
```

---

## Configurar Alertas de Email

Si no usas Slack, Achilles también envía por email.

```
Settings → Integrations → Alerting → Email

SMTP Host:   smtp.gmail.com           (o el de tu empresa)
SMTP Puerto: 587
Usuario:     tu-email@empresa.com
Contraseña:  [tu contraseña o App Password]

Destinatarios:
  soc@empresa.com
  it-security@empresa.com

[ Probar conexión ]
→ Debería llegar un email de prueba ✓

[ Guardar ]
```

**Si usas Gmail**, necesitas usar un "App Password" en lugar de tu contraseña normal:
Google Account → Seguridad → Verificación en 2 pasos → Contraseñas de aplicaciones.

---

## Configurar los Umbrales

Los umbrales definen cuándo se dispara una alerta. Dos tipos:

```
Settings → Integrations → Alerting → Umbrales

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Umbral 1: Caída relativa

Alertar si el Defense Score cae más de: [5%]
en las últimas [24 horas]

Ejemplo: 80% → 74% en una noche → ALERTA
Ejemplo: 80% → 78% en una noche → sin alerta
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Umbral 2: Score mínimo absoluto

Alertar si el Defense Score cae por debajo de: [60%]

Ejemplo: Score = 58% → ALERTA CRÍTICA
Ejemplo: Score = 65% → sin alerta
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Canales: ✅ Slack  ✅ Email

[ Guardar ]
```

### ¿Qué umbrales poner?

```
Si llevas menos de 1 mes con Achilles:
→ Caída relativa: 10% (el score fluctúa más al principio)
→ Suelo: 50%
→ Objetivo: aprender el baseline, no saturarte de alertas

Si llevas 1-3 meses:
→ Caída relativa: 7%
→ Suelo: 60%

Si llevas más de 3 meses y el score es estable:
→ Caída relativa: 5%
→ Suelo: 70%
→ Con el baseline estable, cualquier caída es una señal real
```

---

## La Campana de Notificaciones en el Dashboard

Además de Slack y email, el dashboard tiene una campana de notificaciones:

```
PROJECT ACHILLES   [Analytics] [Endpoints] [Browser]   🔔 3
                                                         ↑
                                                  3 alertas sin leer
```

Click en la campana para ver todas las alertas recientes, aunque no hayas configurado Slack ni email todavía. Útil para cuando empiezas.

---

## Tipos de Alerta

Achilles envía cuatro tipos de notificación:

```
1. Caída del Defense Score
   → El score bajó más del umbral configurado
   → Incluye: score anterior, score actual, técnica más impactada

2. Nuevo gap crítico detectado
   → Una técnica CRÍTICA que antes se detectaba ya no se detecta
   → Señal de que algo cambió en la configuración de seguridad

3. Máquina desconectada
   → Un agente lleva más de 5 minutos sin reportar
   → Puede significar: máquina apagada, agente caído, o problema de red

4. Spike de errores técnicos
   → Muchos tests fallan con error técnico (no el resultado de seguridad, sino el test en sí)
   → Puede indicar problemas con el agente o el entorno
```

---

## Ventanas de Mantenimiento: Silenciar Durante Cambios Planificados

Si vas a hacer una actualización grande (nuevo EDR, cambio de políticas GPO), el score puede fluctuar temporalmente. Para no recibir falsas alarmas:

```
Settings → Alerting → Ventanas de Mantenimiento

[+ Nueva ventana]
Inicio:  Mañana, 22:00
Fin:     Pasado mañana, 06:00
Nota:    "Actualización de políticas de grupo y EDR"

[ Guardar ]
→ Durante esa ventana no se envían alertas
→ Los tests siguen ejecutándose y guardando datos
→ Al terminar la ventana, el nuevo baseline es el punto de referencia
```

---

## Puntos Clave

✅ Slack: crear webhook → pegar URL en Achilles → listo en 5 min
✅ Email: configurar SMTP → añadir destinatarios → listo
✅ Dos umbrales: caída relativa (%) y suelo absoluto
✅ Empieza con umbrales amplios y ajusta conforme conoces tu baseline
✅ Ventanas de mantenimiento evitan falsas alarmas durante cambios planificados
✅ La campana del dashboard funciona sin configurar nada adicional

---

## Próximo Post

**P2-04: "La Librería Completa. Navegar 500+ Técnicas de Ataque"**

Cómo explorar el catálogo de tests de Achilles, qué es MITRE ATT&CK en detalle, y cómo elegir qué probar en tu organización.

---

**Etiquetas:** #ProjectAchilles #Alerting #Slack #Notificaciones #CiberSeguridad #Tutorial

---

*Parte 3 de 4 en la serie "Usando Achilles en Profundidad".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
