# Auto-Resolve: Cierra Automáticamente las Alertas de Tests en el SOC

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 11 de 15**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- **Auto-Resolve** cierra automáticamente en Defender las alertas generadas por tests de Achilles
- Evita que el SOC procese falsas alertas de tests válidos (noise reduction)
- Tres modos: `disabled` → `dry_run` → `enabled` — siempre empieza en dry_run
- Requiere el permiso adicional `SecurityAlert.ReadWrite.All` en Azure AD
- El Defense Score y los datos de tests **NUNCA** se modifican — solo las alertas de Defender

---

## El Problema: Los Tests Contaminan el Queue del SOC

Cuando Achilles ejecuta tests, los tests exitosos generan alertas reales en Microsoft Defender. Eso es bueno — significa que tu defensa funciona. Pero tiene un efecto secundario:

```
Escenario sin Auto-Resolve:

07:00 AM - El analista de turno llega al SOC
07:05 AM - Abre Defender: 47 alertas HIGH pendientes
07:15 AM - Investiga la primera: "Suspicious PowerShell Activity"
07:25 AM - Después de 10 minutos: "Es un test de Achilles... otra vez"
07:35 AM - Repite para 12 alertas más del mismo origen
07:45 AM - 40 minutos perdidos en ruido de validación

La queja del SOC: "Achilles genera demasiado ruido en Defender"
→ Presión para ejecutar los tests con menos frecuencia
→ O peor: añadir excepciones en Defender que afectan a detecciones reales
```

Auto-Resolve elimina ese ruido automáticamente.

---

## Cómo Funciona Auto-Resolve

### El flujo técnico

```
1. Achilles ejecuta test T1059.001 en DESKTOP-SRV01

2. Defender genera alerta:
   ID: a1234567-xxxx
   Título: "Suspicious PowerShell Activity"
   Técnica: T1059.001
   Host: DESKTOP-SRV01
   Timestamp: 2026-05-14T02:01:15Z

3. Achilles correlaciona la alerta con el test ejecutado:
   → Mismo host, misma técnica, mismo timestamp ± 5 minutos
   → f0rtika.achilles_correlated: true
   → f0rtika.achilles_test_uuid: "T1059.001-powershell-exec"

4. Auto-Resolve procesa la alerta (modo enabled):
   → PATCH /v1.0/security/alerts/{alertId}
   → { status: "resolved",
       classification: "informationalExpectedActivity",
       determination: "securityTesting",
       comment: "Auto-resolved by Achilles (bundle: T1059.001-powershell-exec)" }

5. La alerta desaparece del queue activo del SOC
```

---

## Los Tres Modos

### `disabled` (default)

```
→ Auto-Resolve no hace nada
→ Los tests no tienen anotaciones de correlación
→ Las alertas permanecen en Defender sin tocar
→ El SOC las procesa manualmente
```

### `dry_run` ⭐ Empieza aquí

```
→ Achilles calcula qué alertas resolvería
→ Registra un "recibo" (receipt) en el documento ES de la alerta
→ NO hace PATCH en Defender — la alerta permanece abierta
→ Puedes auditar el comportamiento antes de activarlo

Ver el resultado en Analytics → Defender Tab → Auto-Resolve:
"Dry run: 23 alertas hubieran sido resueltas hoy"
"Sin falsos positivos detectados en revisión de muestra"
```

### `enabled`

```
→ Igual que dry_run + ejecuta el PATCH en Defender
→ La alerta se cierra automáticamente
→ El auditor puede ver: status=resolved, determination=securityTesting
→ El SOC solo ve alertas de actividad REAL
```

---

## Configuración: Paso a Paso

### Paso 1: Añadir el permiso adicional en Azure AD

Auto-Resolve requiere un permiso de escritura adicional:

```
Azure Portal → App Registration (tu app de Achilles)
→ API Permissions → Add a permission
→ Microsoft Graph → Application permissions
→ Buscar: SecurityAlert.ReadWrite.All
→ Add permissions
→ Grant admin consent ✅
```

Si el permiso no está otorgado, Auto-Resolve detectará el 403 y mostrará el nombre exacto del scope requerido en el mensaje de error.

### Paso 2: Activar en Settings

```
Settings → Integrations → Microsoft Defender
→ Scroll hasta "Auto-Resolve"

Modo actual: [ disabled ▼]

Cambiar a: [ dry_run ]
[ Guardar ]

→ "Auto-Resolve en modo dry_run. Los receipts se registran
   pero no se ejecutan PATCH en Defender."
```

### Paso 3: Auditar el dry_run

Después de 24-48 horas en dry_run:

```
Analytics → Defender Tab → Auto-Resolve

Estadísticas dry_run (últimas 48h):
→ Alertas correlacionadas con tests: 31
→ Alertas que serían auto-resueltas: 28
→ Alertas excluidas (baja confianza): 3

Receipts de muestra:
ID           Test UUID           Host          Acción
a1234567     T1059.001-exec      SRV01         Hubiera resuelto ✓
b2345678     T1055.001-inject    SRV02         Hubiera resuelto ✓  
c3456789     T1078.002-acc       SRV01         Excluida (baja confianza)

¿Se ven correctas estas acciones?
```

Si todo se ve bien, cambias a `enabled`.

---

## Las Garantías de Seguridad

Auto-Resolve tiene límites deliberados para evitar consecuencias no deseadas:

### Límite de 30 PATCH por pasada

```
Cada pasada de Auto-Resolve (cada 5 minutos) resuelve como máximo 30 alertas.
→ Evita rate-limiting del tenant de Microsoft
→ Protege contra un bug que resolvería todo masivamente
```

### El Defense Score nunca se toca

```
INVARIANTE CRÍTICO:
Auto-Resolve SOLO escribe en documentos de alertas de Defender.
NUNCA modifica documentos de test results.
NUNCA modifica el Defense Score.

Resultado:
→ Defense Score sigue siendo 100% evidencia objetiva
→ "¿Esta alerta se auto-resolvió?" no afecta si el test fue detectado
→ La correlación es independiente de la resolución
```

### Gestión de errores

```
403 → Permiso insuficiente → para la pasada completamente (no intenta las demás)
404 → Alerta ya no existe  → escribe receipt "skip-forever", no reintenta
5xx → Error transitorio    → no escribe receipt, reintenta en próxima pasada
```

---

## Ver el Estado de Auto-Resolve

### Stat Tile en el Dashboard

```
Analytics → Defender Tab

Auto-Resolve          Receipts
    28                   156
 esta semana          últimos 30d
```

### Receipts: El Registro de Auditoría

Cada acción de Auto-Resolve genera un receipt:

```
GET /api/integrations/defender/auto-resolve/receipts

[
  {
    "alertId": "a1234567-xxxx",
    "testUuid": "T1059.001-powershell-exec",
    "host": "DESKTOP-SRV01",
    "resolvedAt": "2026-05-14T02:06:33Z",
    "mode": "enabled",
    "status": "resolved",
    "determination": "securityTesting"
  },
  {
    "alertId": "c3456789-xxxx",
    "testUuid": "T1078.002-valid-accounts",
    "host": "DESKTOP-SRV01",
    "resolvedAt": null,
    "mode": "enabled",
    "status": "skip-forever",
    "reason": "404 - Alert not found"
  }
]
```

Los receipts son inmutables — proporcionan un trail de auditoría completo de todas las resoluciones automáticas.

---

## Cuándo NO Usar Auto-Resolve

No todos los entornos son apropiados para Auto-Resolve:

```
NO usar si:
→ Tu SOC tiene SLA de revisión de TODAS las alertas (auditoría estricta)
→ Utilizas Defender en modo investigation-only sin triaje activo
→ No tienes confianza en la correlación (entorno muy dinámico)
→ Los tests se ejecutan desde IPs/hosts que también tienen actividad real sospechosa

SÍ usar si:
→ Los tests generan suficiente ruido para afectar el triaje del SOC
→ Tienes un programa de purple team activo con ejecuciones frecuentes
→ Necesitas separar "noise de validación" de "amenazas reales" en el queue
→ Quieres evidencia limpia de correlación para reportes de auditoría
```

---

## El Contexto Regulatorio

Para organizaciones bajo DORA, el trail de Auto-Resolve también es evidencia:

```
Evidencia para regulador:
"Nuestros tests de resiliencia generan alertas reales en Defender
(evidencia de detección). Las alertas son clasificadas automáticamente
como 'SecurityTesting' y resueltas con audit trail completo.
El SOC puede distinguir actividad de testing de amenazas reales."

→ Demuestra: el proceso está bajo control
→ Demuestra: las detecciones son reales (no suprimidas antes de alertar)
→ Demuestra: hay separación entre testing y operación real del SOC
```

---

## Puntos Clave

✅ Auto-Resolve resuelve alertas de Defender generadas por tests de Achilles
✅ Siempre empieza en dry_run — audita antes de activar
✅ Requiere `SecurityAlert.ReadWrite.All` en Azure AD
✅ Defense Score y test data son 100% inmutables — Auto-Resolve no los toca
✅ Límite de 30 PATCHes por pasada — protección contra errores masivos
✅ Receipts inmutables = audit trail completo para reguladores

---

## Próximo Post

**POST 12: "Build & Sign — Compilando Agentes Firmados para 6 Plataformas"**

Cómo funciona el pipeline de compilación cruzada y firma de código de Achilles para Windows, Linux y macOS.

---

**Etiquetas:** #ProjectAchilles #AutoResolve #MicrosoftDefender #SOC #NoiseReduction #PurpleTeam #DORA #CiberSeguridad

---

*Parte 11 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
