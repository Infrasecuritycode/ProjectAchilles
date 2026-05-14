# Heatmap MITRE y Treemap de Cobertura: Visualizar tus Gaps de Seguridad

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 8 de 15**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- El **Heatmap MITRE** muestra tu cobertura sobre la matriz ATT&CK — verde/amarillo/rojo por técnica
- El **Coverage Treemap** prioriza visualmente por área y criticidad (más grande = más importante)
- La **Severity Breakdown** chart descompone tus gaps por nivel de riesgo
- El **Defense Score by Host** identifica los endpoints con peor cobertura de un vistazo
- Todas las visualizaciones están filtradas por el mismo selector de fecha y host

---

## Por Qué las Visualizaciones Importan

Los números son poderosos, pero una imagen comunica patrones que una tabla no puede:

```
Tabla de gaps (difícil de procesar):
T1055, T1078, T1134, T1140, T1218, T1036, T1562, T1070...

Heatmap (instantáneo):
Ves de un vistazo que "Defense Evasion" tiene un bloque rojo masivo
mientras "Initial Access" está mayormente verde
→ Priorizas en segundos, no en minutos
```

---

## El Heatmap MITRE ATT&CK

### ¿Qué muestra?

El heatmap reproduce la estructura de la matriz MITRE ATT&CK con tus resultados superpuestos:

```
          T1190  T1078  T1566  T1059  T1053  T1055  T1547  T1021  T1486
Init.Acc  [🟢]   [🔴]   [🟢]   [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Execution [ ]    [ ]    [ ]    [🟡]   [🟢]   [ ]    [ ]    [ ]    [ ]
Persiste. [ ]    [🔴]   [ ]    [ ]    [🟢]   [ ]    [🔴]   [ ]    [ ]
Priv.Esc  [ ]    [🔴]   [ ]    [ ]    [ ]    [🔴]   [🔴]   [ ]    [ ]
Def.Evas. [ ]    [ ]    [ ]    [🟡]   [ ]    [🔴]   [ ]    [ ]    [ ]
Cred.Acc  [ ]    [🔴]   [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Discovery [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Lat.Move. [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [🟢]   [ ]
Collect.  [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Exfil.    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
Impact    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [🔴]

Leyenda:
🟢 Protegido (exit_code 1 en todos los tests de esta técnica)
🟡 Parcial   (mix de exit_code 0 y 1)
🔴 Gap       (exit_code 0 en la mayoría de tests)
[ ] Sin testear
```

### Cómo leer el heatmap

```
Un patrón rojo en una columna completa (ej: T1078 Valid Accounts):
→ Tienes un gap sistémico con cuentas comprometidas
→ Afecta Initial Access, Persistence, Privilege Escalation, Credential Access
→ Acción: implementar MFA + revisar política de cuentas privilegiadas

Un patrón rojo en una fila completa (ej: Defense Evasion):
→ Tu stack no detecta técnicas de evasión en general
→ Los atacantes podrían operar en tu red sin ser vistos
→ Acción: revisar configuración de behavioral detection en EDR

Células amarillas (parcial):
→ Algunos hosts detectan, otros no
→ Heterogeneidad en la configuración
→ Acción: estandarizar la configuración de todos los endpoints
```

### Filtros del heatmap

```
Filtros disponibles:
[Todos] [Solo Protegidos] [Solo Parciales] [Solo Gaps]

→ "Solo Gaps" muestra únicamente las celdas donde fallas
→ Ideal para presentaciones a CISO: "estos son exactamente nuestros riesgos"
```

---

## El Coverage Treemap

### ¿Qué muestra?

El treemap visualiza tu cobertura dividida en rectángulos donde:
- **Tamaño** del rectángulo = criticidad/impacto de esa área
- **Color** = estado de protección (verde → rojo)

```
┌──────────────────────────────────────────────────────────────────┐
│ COVERAGE TREEMAP                                                  │
├────────────────────────────┬─────────────────────────────────────┤
│                            │                │                    │
│   Defense Evasion          │   Execution    │ Credential Access  │
│   🔴  41%                  │   🟡  68%      │ 🔴  52%           │
│                            │                │                    │
│   (área grande = tácticas  ├────────────────┼────────────────────┤
│    con más técnicas y      │    Persist.    │  Lat. Movement     │
│    mayor criticidad)       │   🟡  61%      │  🟢  79%          │
│                            ├────────────────┴────────────────────┤
│                            │  Initial Access │ Discovery  │ ...  │
│                            │  🟢  88%        │ 🟡  64%    │      │
└────────────────────────────┴─────────────────────────────────────┘
```

**Lectura instantánea**: el rectángulo rojo más grande es tu prioridad #1.

### Por qué el treemap es más útil que una lista

Una lista de 427 técnicas es inmanejable. El treemap colapsa esa complejidad:

```
Vista → Dashboard CISO → [Treemap]

Lo que ven en 5 segundos:
→ Defense Evasion: rojo grande → prioridad crítica
→ Initial Access: verde → no es urgente
→ Lateral Movement: verde → bien cubierto
→ Credential Access: rojo → segundo problema

Próxima reunión de seguridad:
"Proponemos invertir en behavioral detection para cubrir
 Defense Evasion y Credential Access — el heatmap lo demuestra."
```

---

## Severity Breakdown Chart

```
Distribution of Test Results by Severity (últimos 30 días)

CRITICAL  ████████████░░░░░░░░  312 tests  61.5% protected
HIGH      ██████████████░░░░░░  518 tests  71.2% protected
MEDIUM    ████████████████░░░░  412 tests  81.4% protected
LOW       ███████████████████░  605 tests  94.2% protected
```

**La lectura correcta**: tus defensas mejoran conforme baja la severidad.

```
Esto es normal y esperado:
→ Técnicas LOW (Discovery, básicas) son fáciles de detectar
→ Técnicas CRITICAL (Process Injection, Living-off-the-Land) son más evasivas

Lo que no debería pasar:
→ CRITICAL < 40%: exposición severa a ataques avanzados
→ CRITICAL < LOW: algo está muy mal configurado
```

---

## Defense Score by Host Chart

```
Defense Score por Endpoint (bar chart horizontal)

DESKTOP-SRV01    ████████████████████  84.2% 🟢
MACBOOK-DEV01    ████████████████      70.4% 🟡
LINUX-PROD-01    ████████████████      67.8% 🟡
DESKTOP-SRV02    ████████████████      72.1% 🟡
DESKTOP-OLD01    ████████████          51.3% 🔴
LINUX-OLD-02     ████████              39.1% 🔴

Insight inmediato:
→ DESKTOP-OLD01 y LINUX-OLD-02 son prioridades de hardening
→ Posiblemente hardware legado sin soporte de EDR moderno
```

---

## Test Activity Timeline

La línea de tiempo de actividad correlaciona cuándo ejecutaste tests con cambios en el score:

```
Tests ejecutados + Defense Score (últimas 4 semanas)

Tests/día
   30 ┤   ██        ██                        ██
   20 ┤   ██  ██    ██    ██        ██        ██
   10 ┤   ██  ██    ██    ██    ██  ██    ██  ██
    0 ┼───────────────────────────────────────────
        L   M  X  J  V  S  D  L  M  X  J  V  S

Defense Score
   78% ┤                                    ╭────
   76% ┤                               ╭────╯
   74% ┤                    ╭──────────╯
   72% ┤         ╭──────────╯
   70% ┤─────────╯

Correlación visible:
→ Después de cada batch de tests → ajustes de configuración → score sube
→ El equipo responde a los gaps con acciones concretas
```

---

## El Error Type Distribution

```
Error Type Pie Chart

Tests con exit_code 0 (GAP): ████████████░░░░  40.3%
Tests con exit_code 1 (✅):  ████████████████  57.1%
Tests con exit_code 2 (⚠️):  ░░░  2.6%
```

El `exit_code 2` (error técnico) merece atención:
- Puede indicar que el binario de test fue bloqueado antes de ejecutar (que es en sí mismo protección)
- O que hay un problema técnico con el agente o el entorno
- Investigar los casos individuales en la Executions Table

---

## Cómo Usar las Visualizaciones en la Práctica

### Reunión semanal de seguridad

```
Agenda tipo:
1. [2 min] Hero metrics — ¿subió o bajó el Defense Score?
2. [5 min] Heatmap — ¿hay nuevos gaps desde la semana pasada?
3. [5 min] Top Gaps — ¿qué tres técnicas priorizamos esta semana?
4. [3 min] Defense Score by Host — ¿algún host se degradó?
```

### Preparar evidencia para auditoría

```
Para auditor DORA/ISO:
1. Exportar heatmap como imagen → incluir en informe
2. Capturar Defense Score histórico (trend chart)
3. Documentar cada gap con plan de remediación
4. Mostrar evolución mensual: "mejoramos 7.5% en Q1 2026"
```

### Identificar regresiones

```
Si el Defense Score baja entre semanas:
1. Filtrar por la semana anterior vs esta semana
2. Comparar heatmaps: ¿qué celdas cambiaron de verde a rojo?
3. Probable causa: actualización de OS/EDR que cambió política
   o exclusión nueva añadida por el equipo de IT

→ El score como "canario": detecta degradaciones de configuración
   antes de que un atacante real las descubra
```

---

## Puntos Clave

✅ Heatmap MITRE: cobertura sobre la matriz ATT&CK — patrones visuales inmediatos
✅ Treemap: priorización visual por criticidad y área — decisiones en 5 segundos
✅ Severity Breakdown: normal que CRITICAL sea más bajo que LOW
✅ Defense Score by Host: identifica endpoints con peor cobertura
✅ Test Activity Timeline: correlaciona acciones de hardening con mejoras en el score
✅ Exportable para informes de auditoría DORA / ISO 27001

---

## Próximo Post

**POST 09: "Bundle Tests — Validación de Controles Completos"**

Cómo los bundle tests validan docenas de controles en una sola ejecución, y cómo interpretar los resultados agrupados en la Executions Table.

---

**Etiquetas:** #ProjectAchilles #MITRE #Heatmap #Visualizaciones #PurpleTeam #SOC #CiberSeguridad #Analytics

---

*Parte 8 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
