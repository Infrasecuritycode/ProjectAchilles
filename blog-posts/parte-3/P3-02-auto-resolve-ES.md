# Auto-Resolve: Que Achilles Limpie las Alertas de Tests por Ti

> **Serie: Integraciones y Automatización — Parte 2 de 3**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Cuando Achilles ejecuta tests, Defender genera alertas reales — eso confirma que detecta el ataque
- El problema: esas alertas llenan el queue de tu equipo con "falsos positivos" de pruebas
- **Auto-Resolve** las cierra automáticamente en Defender marcándolas como "security testing"
- Tiene tres modos: desactivado → prueba → activo — siempre empieza en modo prueba
- Requiere un permiso adicional en Azure y tener la integración con Defender activa (P3-01)

---

## El Problema que Resuelve

Imagina que tu equipo de seguridad llega el lunes por la mañana:

```
Bandeja de alertas de Defender: 47 alertas HIGH sin revisar

Analista revisa la primera:
→ "Suspicious PowerShell Activity en DESKTOP-SRV01"
→ Investiga 10 minutos
→ "Es de un test de Achilles del domingo... otra vez"

Analista revisa la segunda:
→ "LSASS Memory Access en DESKTOP-SRV02"
→ Investiga 10 minutos
→ "También es de Achilles"

40 minutos después: revisó 4 alertas, todas de tests
```

El equipo empieza a quejarse del ruido. La tentación es ejecutar menos tests — y eso es exactamente lo contrario de lo que necesitas.

**Auto-Resolve resuelve ese problema** cerrando automáticamente las alertas que Achilles sabe que generó él mismo.

---

## Cómo Sabe Achilles Qué Alertas Cerrar

Achilles no cierra alertas al azar. El proceso de correlación es muy específico:

```
1. Achilles ejecuta test T1059.001 en DESKTOP-SRV01 a las 02:01:15

2. Defender genera alerta:
   → Técnica: T1059.001 (PowerShell)
   → Host: DESKTOP-SRV01
   → Timestamp: 02:01:23

3. Achilles correlaciona:
   → ¿Hay un test de T1059.001 en DESKTOP-SRV01 en los últimos 5 minutos?
   → Sí → Esta alerta es del test

4. Auto-Resolve cierra la alerta en Defender:
   → Estado: Resolved
   → Clasificación: Informational / Expected Activity
   → Determinación: Security Testing
   → Comentario: "Closed by Achilles — test T1059.001"
```

Solo se cierran alertas que coinciden exactamente: misma técnica, mismo host, mismo período de tiempo.

---

## Paso 1: Añadir el Permiso Adicional en Azure

Auto-Resolve necesita poder **escribir** en Defender, no solo leer. Eso requiere un permiso adicional en la aplicación de Azure que creaste en P3-01.

```
Azure Portal → Microsoft Entra ID → App registrations
→ Abre tu aplicación "Achilles Security Integration"
→ API permissions → Add a permission
→ Microsoft Graph → Application permissions
→ Busca y selecciona:

   ✅ SecurityAlert.ReadWrite.All

→ Add permissions
→ Grant admin consent ✅
```

Si no puedes o no quieres dar permiso de escritura, Auto-Resolve tiene un **modo prueba** que simula las resoluciones sin ejecutarlas — puedes auditar el comportamiento antes de decidir.

---

## Paso 2: Activar en Achilles

```
Settings → Integrations → Microsoft Defender
→ Sección "Auto-Resolve"

Modo actual: [ Desactivado ]

Cambia a:    [ Modo prueba  ]

[ Guardar ]
```

**¿Por qué empezar en modo prueba?** Porque así puedes ver exactamente qué alertas resolvería Achilles antes de que realmente las cierre. Tienes 24-48 horas para revisar y confirmar que el comportamiento es correcto.

---

## Los Tres Modos Explicados

### Desactivado (default)
```
→ Achilles no toca ninguna alerta de Defender
→ Tu equipo las ve y gestiona todas manualmente
→ Igual que antes de activar la integración
```

### Modo prueba ← Empieza aquí
```
→ Achilles calcula qué alertas cerraría
→ Registra esa decisión internamente (un "recibo")
→ NO cierra nada en Defender
→ Puedes revisar: "¿tiene sentido lo que hubiera cerrado?"

Ver los resultados:
Analytics → Defender → Auto-Resolve

"Modo prueba activo — resultados de las últimas 48h:
 31 alertas hubieran sido cerradas automáticamente
 3 alertas excluidas (baja confianza de correlación)"
```

### Activo
```
→ Igual que el modo prueba + ejecuta el cierre real en Defender
→ La alerta desaparece del queue activo del equipo
→ Queda registrada como "Resolved / Security Testing"
→ El historial de auditoría está completo
```

---

## Activar el Modo Completo

Después de revisar el modo prueba y confirmar que el comportamiento es correcto:

```
Settings → Integrations → Defender → Auto-Resolve

Modo actual: [ Modo prueba ]
Cambiar a:   [ Activo      ]

[ Guardar ]

→ "Auto-Resolve activo. Las alertas de tests de Achilles
   se cerrarán automáticamente en Defender."
```

A partir de ahora, cada vez que Achilles ejecuta un test y Defender genera una alerta, Achilles la cierra en los siguientes 5 minutos.

---

## Ver el Historial de lo que Se Cerró

Cada cierre queda registrado y es auditable:

```
Analytics → Defender → Auto-Resolve → Ver Historial

Alerta              Test               Host          Cerrada
─────────────────────────────────────────────────────────────
a1234-xxx           T1059.001-exec     SRV01         02:06 ✅
b5678-xxx           T1055.001-inject   SRV02         02:07 ✅
c9012-xxx           T1078.002-accounts SRV01         02:08 ✅
d3456-xxx           T1021.001-rdp      SRV02         Excluida ⚠️
```

La última fila (excluida) significa que Achilles no tuvo suficiente confianza en la correlación y decidió no cerrarla — el analista la revisará manualmente.

---

## Lo Que NO Hace Auto-Resolve

Es importante entender los límites:

```
Auto-Resolve NO cierra alertas de amenazas reales.
Solo cierra alertas donde puede confirmar que son de un test de Achilles.

Auto-Resolve NO modifica tus resultados de tests ni el Defense Score.
Si el test generó alerta = protegido. Siempre. Independiente de si
la alerta se cierra después.

Auto-Resolve tiene un límite de 30 cierres por pasada (cada 5 minutos).
Esto evita problemas si hay un bug — nunca cierra miles de alertas de golpe.

Si hay un error en Defender (403, timeout):
→ Para limpiamente y lo reintenta en la próxima pasada
→ No deja alertas en estado inconsistente
```

---

## El Efecto en el SOC

Con Auto-Resolve activo, el cambio para el equipo de seguridad es inmediato:

```
Antes:
Bandeja del lunes: 47 alertas HIGH sin revisar
→ 31 son de tests de Achilles (ruido)
→ 16 son actividad real que necesita atención
→ El equipo pierde tiempo separando los dos

Después:
Bandeja del lunes: 16 alertas HIGH sin revisar
→ Todas son actividad real
→ El equipo se enfoca en lo que importa
```

---

## ¿Es Seguro Cerrar Alertas Automáticamente?

La pregunta que todos hacen. La respuesta corta: sí, con las protecciones correctas.

Las protecciones de Auto-Resolve:

```
1. Solo cierra alertas que coinciden exactamente con un test ejecutado
   (misma técnica + mismo host + ventana de 5 minutos)

2. Nunca cierra si la confianza de correlación es baja

3. Cada cierre queda registrado con timestamp y razón
   (auditable para reguladores)

4. Límite de 30 por pasada — protección contra errores masivos

5. Modo prueba para auditar antes de activar

6. Se puede desactivar en cualquier momento desde Settings
```

Para organizaciones bajo DORA o ISO 27001: el historial de Auto-Resolve es evidencia de que el proceso de testing está bajo control y separado de la operación real del SOC.

---

## Puntos Clave

✅ Auto-Resolve cierra en Defender las alertas generadas por tests de Achilles
✅ Siempre empieza en modo prueba — audita 24-48h antes de activar
✅ Requiere permiso adicional en Azure: `SecurityAlert.ReadWrite.All`
✅ Solo cierra alertas con alta confianza de correlación — nunca amenazas reales
✅ Defense Score y resultados de tests son inmutables — Auto-Resolve no los toca
✅ Historial completo de cierres para auditorías

---

## Próximo Post

**P3-03: "Gestión de Flota — Administrar Muchos Agentes a la Vez"**

Cuando tienes más de 5-10 máquinas con el agente, la gestión cambia. Cómo organizar tu flota, actualizar agentes remotamente y programar campañas de tests a escala.

---

**Etiquetas:** #ProjectAchilles #AutoResolve #MicrosoftDefender #SOC #Automatización #CiberSeguridad

---

*Parte 2 de 3 en la serie "Integraciones y Automatización con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
