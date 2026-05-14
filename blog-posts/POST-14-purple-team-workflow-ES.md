# Purple Team Workflow Completo: De Intel de Amenazas a Evidencia

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 14 de 15**

**Tiempo de lectura:** 12 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- Un workflow de purple team con Achilles tiene 5 fases: Intel → Tests → Ejecución → Análisis → Hardening
- El ciclo completo tarda entre 2-4 horas para una campaña nueva
- Achilles automatiza fases 2, 3 y parte de 4 — el equipo se enfoca en inteligencia y hardening
- El output final es evidencia técnica objetiva: Defense Score antes/después + heatmap + lista de gaps

---

## ¿Qué Es un Ejercicio de Purple Team?

Un ejercicio de purple team une al Red Team y Blue Team para medir y mejorar la cobertura de detección:

```
Red Team:   "Simulamos las técnicas que usaría un atacante real"
Blue Team:  "Medimos si las detectamos y cuánto tardamos"
Purple Team: "Hacemos ambas cosas juntos, de forma colaborativa"

Sin Achilles (manual):                Con Achilles:
→ 2-3 semanas de preparación         → 2-4 horas para campaña nueva
→ 5-10 técnicas por ejercicio         → 50-150 técnicas por ejercicio
→ Informe PDF subjetivo               → Métricas objetivas en Elasticsearch
→ Evidencia difícil de reproducir     → Reproducible, comparable, auditable
```

---

## Fase 1: Inteligencia de Amenazas — El Input

Todo ejercicio purple team empieza con una pregunta: **¿Contra qué nos preparamos?**

### Opciones de fuentes de intel

```
A) Advisory CISA reciente
   Ejemplo: CISA AA26-135A — "APT targets financial sector with..."
   → Identificar técnicas MITRE mencionadas
   → Buscar en el Browser de Achilles

B) Campaña APT específica
   Ejemplo: prepararse para APT29 (SVR ruso) si eres entidad financiera europea
   → Bundle apt29-campaign ya disponible en Achilles

C) Sector-specific threat intel
   Ejemplo: FS-ISAC report para banca, H-ISAC para salud
   → Extraer técnicas top del sector este trimestre

D) Post-mortem de incidente reciente
   Ejemplo: "Necesitamos validar que la misma campaña que afectó a
             un competidor no funcionaría contra nosotros"
```

### Extraer técnicas MITRE del intel

```
Ejemplo: CISA Advisory AA26-135A menciona:
→ T1566.001 (Spearphishing Attachment) — Initial Access
→ T1059.001 (PowerShell) — Execution
→ T1547.001 (Registry Run Keys) — Persistence
→ T1078 (Valid Accounts) — Defense Evasion / Privilege Escalation
→ T1021.001 (Remote Desktop Protocol) — Lateral Movement
→ T1041 (Exfiltration Over C2 Channel) — Exfiltration
→ T1486 (Data Encrypted for Impact) — Impact

Total: 7 técnicas clave de esta campaña
```

---

## Fase 2: Seleccionar y Compilar Tests

### Búsqueda en el Browser

```
Browser → buscar técnicas identificadas en el intel:

T1566.001  Spearphishing Attachment    HIGH   Windows
T1059.001  PowerShell Encoded          HIGH   Windows
T1059.001  PowerShell AMSI Bypass      CRITICAL Windows
T1547.001  Registry Run Key            MEDIUM Windows
T1078.002  Domain Accounts             CRITICAL Windows/Linux
T1021.001  RDP Authentication          HIGH   Windows
T1041      Exfil over C2               HIGH   Windows/Linux
T1486      Ransomware Simulation       CRITICAL Windows

Total seleccionado: 8 tests para la campaña
```

### Compilar el bundle de campaña

```
Browser → seleccionar los 8 tests → [Compilar como campaña]

Nombre: "APT AA26-135A Campaign"
Plataformas: Windows x64

[ Compilar todos ]
→ 8 binarios compilados y firmados
→ Bundle: apt-AA26-135A-win-amd64.zip (28 MB)
→ Listo para asignación
```

---

## Fase 3: Planificación y Ejecución

### Seleccionar la flota de test

```
Para un ejercicio realista, selecciona máquinas representativas:
→ 1-2 workstations de usuarios estándar (sin privilegios de admin)
→ 1 servidor de archivos
→ 1 domain controller (con mucho cuidado)
→ 1 máquina de desarrollo

Evitar:
→ Producción crítica durante horas pico
→ Máquinas con datos reales sensibles (usa entornos de test si es posible)
```

### Programar la ejecución

```
Endpoints → Campaña → [+ Nueva Campaña]

Tests: [Bundle AA26-135A — 8 tests]
Agentes: 
  ✅ DESKTOP-SRV01 (workstation estándar)
  ✅ DESKTOP-SRV02 (workstation con acceso privilegiado)
  ✅ FILESERVER-01  (servidor de archivos)

Horario: Mañana, 02:00 AM (fuera de horas pico)

Notificación: Slack #security-alerts cuando termine

[ Programar campaña ]
→ 24 tareas creadas (8 tests × 3 agentes)
→ Ejecución estimada: 45 minutos
```

### Durante la ejecución (02:00 - 02:45 AM)

```
El Blue Team monitoriza su SIEM en tiempo real durante la ejecución:
→ ¿Qué alertas se generan?
→ ¿En cuánto tiempo?
→ ¿Son claras y accionables?
→ ¿Se correlacionan correctamente?

El Red Team documenta las técnicas ejecutadas:
→ Qué parámetros se usaron
→ Qué comportamiento genera cada técnica
→ Qué evasiones serían posibles en un ataque real
```

---

## Fase 4: Análisis de Resultados

### Defense Score antes vs después

```
Defense Score BASE (pre-campaña):  73.1%
Defense Score POST-campaña:        68.4%  ← cayó 4.7%

Interpretación:
Las nuevas técnicas del advisory tienen menor cobertura que el promedio
→ Hay un gap específico para esta campaña APT
```

### Heatmap de la campaña

```
Táctica           T1566  T1059  T1547  T1078  T1021  T1041  T1486
                                                              
Initial Access    [🔴]   [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Execution         [ ]    [🟡]   [ ]    [ ]    [ ]    [ ]    [ ]
Persistence       [ ]    [ ]    [🟢]   [ ]    [ ]    [ ]    [ ]
Defense Evasion   [ ]    [ ]    [ ]    [🔴]   [ ]    [ ]    [ ]
Lateral Movement  [ ]    [ ]    [ ]    [ ]    [🟢]   [ ]    [ ]
Exfiltration      [ ]    [ ]    [ ]    [ ]    [ ]    [🔴]   [ ]
Impact            [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [🟢]

Resumen de la campaña AA26-135A:
  ✅ Protegido (3/8):  T1547, T1021, T1486
  🟡 Parcial   (1/8):  T1059
  ❌ Gap        (4/8):  T1566, T1078, T1041, AMSI bypass
  
Defense Score campaña específica: 37.5% — CRÍTICO
```

### Análisis gap por gap

```
T1566.001 — Spearphishing Attachment (❌ 3/3 hosts sin cobertura)
  Hosts: DESKTOP-SRV01, DESKTOP-SRV02, FILESERVER-01
  Por qué falla: El EDR detecta el payload pero no el attachment inicial
  Acción: Revisar reglas de email scanning + sandbox de attachments
  
T1078 — Valid Accounts (❌ 3/3 hosts sin cobertura)  
  Hosts: todos
  Por qué falla: No tenemos MFA en todas las cuentas privilegiadas
  Acción: URGENTE - Implementar MFA para cuentas Domain Admin
  
T1041 — Exfil over C2 (❌ 2/3 hosts sin cobertura)
  Hosts: DESKTOP-SRV01, FILESERVER-01
  Por qué falla: Regla de DNS tunneling no configurada en SIEM
  Acción: Añadir regla Sigma sigma/dns_tunneling_by_nslookup.yml

T1059.001 — PowerShell (🟡 2/3 hosts)
  Detectado en: DESKTOP-SRV01, DESKTOP-SRV02
  No detectado en: FILESERVER-01
  Por qué: FILESERVER-01 no tiene Script Block Logging habilitado
  Acción: Aplicar GPO de PowerShell logging a todos los servidores
```

---

## Fase 5: Hardening y Cierre del Loop

### Priorizar acciones

```
Prioridad CRÍTICA (esta semana):
  1. T1078: MFA para Domain Admin accounts
     Owner: IT → Deadline: Viernes
     
  2. T1059 en FILESERVER-01: Aplicar GPO de PS logging  
     Owner: IT → Deadline: Miércoles

Prioridad HIGH (próximas 2 semanas):
  3. T1566.001: Revisar sandbox de email attachments
     Owner: Security Engineering
     
  4. T1041: Implementar regla DNS tunneling en SIEM
     Owner: SOC → Deadline: Semana que viene

Prioridad MEDIUM (próximo mes):
  5. T1059 AMSI Bypass: Actualizar configuración AMSI
     Owner: Security Engineering
```

### Verificación post-hardening

```
Después de implementar las correcciones:

Semana siguiente → Re-ejecutar la misma campaña AA26-135A:

Defense Score campaña AA26-135A:
  Antes:  37.5%  ❌
  Después: 81.2%  ✅

Cambio por técnica:
  T1566.001  🔴 → 🟢  (sandbox activo)
  T1078      🔴 → 🟢  (MFA implementado)
  T1041      🔴 → 🟡  (regla parcial, mejorar)
  T1059 AMSI 🔴 → 🟢  (configuración actualizada)
  T1059 PS   🟡 → 🟢  (GPO aplicado a todos los servidores)
```

**+43.7 puntos en una semana** — con evidencia objetiva de cada mejora.

---

## El Informe de Evidencia

El output final del ejercicio es un paquete de evidencia para el CISO o el regulador:

```
INFORME DE EJERCICIO PURPLE TEAM
Campaña: APT AA26-135A
Fecha: 2026-05-14 → 2026-05-21
Equipo: Blue Team + Red Team + Achilles

1. CONTEXTO
   Fuente: CISA Advisory AA26-135A (Mayo 2026)
   Técnicas simuladas: 8 técnicas MITRE ATT&CK
   Endpoints evaluados: 3 (workstations + fileserver)
   
2. RESULTADOS PRE-HARDENING
   Defense Score (campaña): 37.5%
   Gaps críticos identificados: 4
   [Incluir screenshot heatmap]
   
3. ACCIONES TOMADAS
   [Tabla de acciones con owner, fecha y estado]
   
4. RESULTADOS POST-HARDENING
   Defense Score (campaña): 81.2%  (+43.7%)
   Gaps críticos resueltos: 3/4
   1 gap en progreso (T1041: implementación parcial)
   [Incluir screenshot heatmap post]
   
5. EVIDENCIA
   - Logs de ejecución de Achilles (exportar desde Analytics)
   - Documentos Elasticsearch (test_uuid por técnica)
   - Alertas Defender correlacionadas
   
6. PLAN DE CONTINUIDAD
   - Re-ejecutar campaña en 30 días
   - Programar validación mensual de técnicas críticas
   - Próximo ejercicio: campaña ransomware-baseline
```

---

## Cadencias Recomendadas

```
Semanal (automatizado):
→ Suite de técnicas críticas contra flota completa
→ Defense Score monitoring con alerting configurado
→ Tiempo requerido del equipo: 0 (automatizado)

Mensual (manual):
→ Revisión del heatmap completo
→ Identificar y priorizar 3-5 gaps nuevos
→ Ejecutar bundle de sector-specific threats
→ Tiempo: 2-3 horas

Trimestral (ejercicio purple team formal):
→ Campaña completa contra 1-2 nuevas amenazas relevantes
→ Ciclo completo: intel → tests → hardening → verificación
→ Output: informe de evidencia para CISO / auditor
→ Tiempo: 1-2 días

Anual:
→ Benchmark completo de las 427+ técnicas MITRE
→ Comparativa año vs año
→ Evidencia para certificaciones ISO 27001, DORA, SOC2
```

---

## Puntos Clave

✅ 5 fases: Intel → Tests → Ejecución → Análisis → Hardening
✅ Achilles automatiza fases 2-4: el equipo se enfoca en inteligencia y corrección
✅ Defense Score antes/después es evidencia objetiva del valor del ejercicio
✅ El heatmap comunica los gaps a cualquier audiencia — técnica o ejecutiva
✅ El loop cierra cuando re-ejecutas la misma campaña y mides la mejora
✅ El informe de evidencia satisface requisitos de auditoría DORA/ISO

---

## Próximo Post

**POST 15: "Compliance con Achilles — DORA, TIBER-EU e ISO 27001"**

El post final de la serie: cómo usar Achilles para construir el paquete de evidencia de cumplimiento regulatorio.

---

**Etiquetas:** #ProjectAchilles #PurpleTeam #BlueTeam #RedTeam #MITRE #CiberSeguridad #ThreatIntel #CISA #SOC

---

*Parte 14 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
