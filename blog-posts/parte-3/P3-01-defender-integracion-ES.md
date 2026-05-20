# Conectar Achilles con Microsoft Defender: Ve Todo en un Solo Lugar

> **Serie: Integraciones y Automatización — Parte 1 de 3**

**Tiempo de lectura:** 9 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Si usas Microsoft Defender for Endpoint, Achilles puede conectarse a él y mostrar ambos en el mismo dashboard
- Verás tu **Defense Score** (Achilles) y tu **Secure Score** (Microsoft) juntos, y la diferencia entre los dos es información muy valiosa
- Achilles importa las alertas reales de Defender y las cruza con tus tests: "¿este ataque que simulamos generó una alerta?"
- Configurarlo toma unos 20 minutos y requiere acceso al portal de Azure
- Si no usas Defender o no tienes acceso a Azure, puedes saltar este post

---

## Por Qué Conectar Defender con Achilles

Ya tienes el Defense Score de Achilles. Microsoft también tiene su propio score de seguridad llamado **Secure Score**. Son dos números diferentes que miden cosas distintas:

```
Defense Score (Achilles): 73%
"De los ataques que simulamos, detectamos el 73%"
→ Basado en pruebas reales ejecutadas en tus máquinas

Secure Score (Microsoft): 62%
"Tienes el 62% de las configuraciones recomendadas por Microsoft"
→ Basado en checkboxes de configuración
```

El gap entre los dos es revelador:

```
Si Secure Score es alto pero Defense Score es bajo:
→ Tienes la configuración "correcta" en papel,
  pero en la práctica no estás detectando los ataques
→ Algo entre la configuración y la detección real no funciona

Si ambos son similares:
→ Las configuraciones recomendadas por Microsoft realmente
  se están traduciendo en detección efectiva — bien
```

Sin la integración, tienes dos números separados. Con la integración, los ves juntos y la comparación cobra sentido.

---

## Lo Que Necesitas Antes de Empezar

```
✅ Microsoft Defender for Endpoint activo en tu organización
   (Plan 1 o Plan 2 — también funciona con Microsoft 365 E3/E5)

✅ Acceso al portal de Azure (portal.azure.com)
   (necesitas permisos de administrador o que alguien con permisos
    lo configure contigo)

✅ Achilles instalado y funcionando (con al menos un agente)
```

Si no tienes acceso a Azure, comparte este post con tu administrador de sistemas o el equipo de Microsoft 365. El proceso toma unos 15 minutos para alguien con acceso.

---

## Paso 1: Crear la Aplicación en Azure (15 minutos)

Achilles se conecta a Defender a través de la API de Microsoft. Para eso necesitas registrar una "aplicación" en Azure, básicamente darle permiso a Achilles para leer datos de Defender.

### 1.1 Registrar la aplicación

```
1. Ve a https://portal.azure.com
2. Busca "Microsoft Entra ID" (antes se llamaba Azure Active Directory)
3. En el menú izquierdo: "App registrations"
4. Click "+ New registration"

   Nombre:                 Achilles Security Integration
   Supported account types: Accounts in this organizational directory only
   Redirect URI:           (dejar vacío)

5. Click "Register"
```

Ahora estás en la página de tu nueva aplicación. Copia estos dos valores, los necesitarás más adelante:

```
Application (client) ID:  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Directory (tenant) ID:    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### 1.2 Dar permisos para leer datos de seguridad

```
En tu aplicación → "API permissions" → "Add a permission"
→ Microsoft Graph
→ Application permissions (no Delegated)
→ Busca y selecciona:

   ✅ SecurityEvents.Read.All

→ Click "Add permissions"
→ Click "Grant admin consent for [tu organización]"

Estado final: ✅ Granted (verde)
```

⚠️ El paso "Grant admin consent" requiere ser administrador global o administrador de seguridad. Si el botón está en gris, necesitas pedirle a alguien con ese rol que lo apruebe.

### 1.3 Crear la contraseña de la aplicación

```
En tu aplicación → "Certificates & secrets"
→ "Client secrets" → "+ New client secret"

   Description: Achilles Integration
   Expires:     24 months

→ Click "Add"

⚠️ IMPORTANTE: Copia el "Value" ahora — solo se muestra esta vez.
   Si cierras la página sin copiarlo, tendrás que crear otro.

Client secret value: ~XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

---

## Paso 2: Configurar en Achilles (5 minutos)

Con los tres valores copiados, ve al dashboard de Achilles:

```
Settings → Integrations → Microsoft Defender

Tenant ID:      xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Client ID:      xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Client Secret:  ~XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

[ Test Connection ]
```

Si todo está bien:

```
✅ Conectado
   Organización: Tu Empresa S.L.
   Secure Score actual: 68.5 / 100
   Alertas disponibles: 1,247
```

Click **[ Guardar ]**.

Las credenciales se guardan cifradas. Nunca se almacenan en texto plano.

---

## Paso 3: Primera Sincronización

Después de guardar, Achilles empieza a importar datos de Defender:

```
Settings → Integrations → Microsoft Defender
→ [ Sincronizar ahora ]

Importando...
✓ Secure Score: 68.5 pts
✓ Control profiles: 89 controles
✓ Alertas: 1,247 alertas (últimos 30 días)

Última sincronización: hace 30 segundos
```

La sincronización automática se ejecuta:
- **Alertas**: cada 5 minutos
- **Secure Score y controles**: cada 6 horas

---

## Lo Que Ves Ahora en el Dashboard

Ve a **Analytics**, verás una pestaña nueva: **Defender**.

### Los scores juntos

```
┌──────────────────────────────────────────────────────────────┐
│  Defense Score (Achilles)       Secure Score (Microsoft)     │
│         73%                            68%                   │
│    ████████░░░                    ███████░░░░                │
│  Basado en ataques reales        Basado en configuración     │
└──────────────────────────────────────────────────────────────┘
```

### Las alertas recientes de Defender

```
ALERTAS DEFENDER (últimas 24 horas)

🔴 HIGH    PowerShell sospechoso         DESKTOP-SRV01   10:45
🔴 HIGH    Acceso a memoria de LSASS     DESKTOP-SRV02   09:12
🟠 MEDIUM  Cuenta admin con uso inusual  LINUX-PROD-01   08:33
🟡 LOW     Escaneo de red detectado      DESKTOP-OLD01   07:15
```

---

## La Parte Más Valiosa: La Correlación

Achilles cruza automáticamente los tests que ejecutaste con las alertas que generó Defender:

```
Analytics → Defender → Cross-Correlation

Tests ejecutados este mes:              312
Tests que generaron alerta en Defender: 189  (60.6%) ✅
Tests sin alerta en Defender:           123  (39.4%) ❌

→ 4 de cada 10 técnicas que probamos
  NO generaron alerta en Defender
```

Esta es la lista más accionable que puedes tener: técnicas confirmadas como gaps en Defender específicamente.

### La línea de tiempo

Para cada test, puedes ver si Defender alertó y cuánto tardó:

```
02:01:15  Achilles ejecuta T1059.001 en DESKTOP-SRV01
02:01:23  Defender genera alerta: "Suspicious PowerShell"
          → Tiempo de detección: 8 segundos ✅

02:01:45  Achilles ejecuta T1021.001 en DESKTOP-SRV01
          → Sin alerta en los siguientes 5 minutos ❌
          → Gap confirmado en Defender para esta técnica
```

---

## Errores Comunes y Cómo Resolverlos

### "Insufficient privileges" (Error 403)

```
Causa: El admin consent no fue aprobado

Solución:
Azure → App Registration → API Permissions
→ Verifica que SecurityEvents.Read.All dice "✅ Granted"
→ Si dice "Not granted": Click "Grant admin consent"
  (necesitas ser Global Admin o Security Admin)
```

### "Invalid client secret" (Error 401)

```
Causa: El secreto expiró o fue copiado incorrectamente

Solución:
Azure → App Registration → Certificates & secrets
→ El secreto puede haber expirado
→ Crea uno nuevo y actualízalo en Achilles Settings
```

### Las alertas no aparecen o están desactualizadas

```
Settings → Integrations → Defender
→ Revisa "Última sincronización"
→ Si hace más de 10 minutos: click [ Sincronizar ahora ]
→ Si sigue fallando: click [ Test Connection ] para ver el error exacto
```

---

## ¿Qué Pasa con mis Credenciales de Azure?

Una pregunta válida. Las credenciales se almacenan:

```
→ Cifradas con AES-256-GCM
→ En el servidor de Achilles (no en ningún servicio externo)
→ Solo el backend de Achilles puede descifrarlas
→ Nunca aparecen en logs ni en texto plano
→ Si usas self-hosted: se guardan en ~/.projectachilles/integrations.json
```

El permiso que diste (`SecurityEvents.Read.All`) es de **solo lectura**. Achilles no puede modificar nada en tu tenant de Microsoft.

---

## Puntos Clave

✅ Conectar Defender toma ~20 minutos con acceso a Azure
✅ Ves Defense Score (Achilles) + Secure Score (Microsoft) juntos
✅ La correlación confirma qué técnicas generan alertas y cuáles no
✅ El permiso es de solo lectura. Achilles no puede modificar nada en Microsoft
✅ Credenciales cifradas localmente, nunca en texto plano
✅ Sincronización automática: alertas cada 5 min, scores cada 6h

---

## Próximo Post

**P3-02: "Auto-Resolve. Que Achilles Limpie las Alertas de Tests por Ti"**

Cuando ejecutas tests, Defender genera alertas reales. Eso es bueno, confirma que funciona. Pero esas alertas llenan la bandeja de tu equipo de seguridad. Auto-Resolve las cierra automáticamente.

---

**Etiquetas:** #ProjectAchilles #MicrosoftDefender #Azure #Integración #CiberSeguridad #SOC #DefenseScore

---

*Parte 1 de 3 en la serie "Integraciones y Automatización con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
