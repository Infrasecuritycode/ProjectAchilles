# Purple Team Workflow Completo: De Intel de Amenazas a Evidencia

> **Serie: Pro — Parte 1 de 3**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- Un ejercicio Purple Team es Red y Blue trabajando juntos — no un Red Team atacando en secreto
- El flujo completo tiene 5 fases: intel → selección de tests → ejecución → análisis → remediación con retest
- Achilles estructura todo ese flujo: importas la intel, ejecutas los tests, mides la cobertura y generas la evidencia
- El resultado es un informe defendible: "probamos estas técnicas, esto detectamos, esto remediamos, esto mejoró"
- Este flujo es la base de los ejercicios TIBER-EU y DORA para instituciones financieras

---

## ¿Qué es Purple Team y por Qué Importa?

```
Red Team (tradicional):        Purple Team:
→ Ataca en secreto             → Red y Blue trabajan juntos
→ El Blue no sabe nada         → El Blue sabe qué se va a probar
→ Resultado: "os entramos"     → Resultado: "mejoramos la detección
→ ¿Y ahora qué?                  en estas 8 técnicas"
```

El Purple Team no reemplaza al Red Team tradicional. Son para cosas distintas:

- **Red Team**: valida que el Blue Team reaccione ante un ataque real (no saben cuándo ni cómo)
- **Purple Team**: optimiza la cobertura de detección de forma sistemática (con evidencia para auditorías)

Achilles es una plataforma de Purple Team continuo — puedes hacer este flujo cada semana, no solo una vez al año.

---

## Las 5 Fases del Flujo

```
FASE 1         FASE 2          FASE 3          FASE 4         FASE 5
Intel de    →  Selección   →  Ejecución   →  Análisis    →  Remediación
amenazas       de tests        controlada      de gaps         + retest

"¿Qué        "¿Cuáles de     "Ejecutamos     "¿Qué no       "Arreglamos,
técnicas       esas técnicas   los tests       detectó        ejecutamos
usa ese        tiene           en la flota"    el stack?"     otra vez,
actor?"        Achilles?"                                     score sube"
```

---

## Fase 1: Intel de Amenazas

El punto de partida es siempre una pregunta: **¿contra quién nos estamos preparando?**

### Fuentes de intel

```
Opción A — Tu sector específico:
→ CISA advisories (cisa.gov/known-exploited-vulnerabilities)
→ ISAC de tu sector (FS-ISAC para financiero, H-ISAC para salud, etc.)
→ Informes anuales de vendors (CrowdStrike, Mandiant, Microsoft MSTIC)

Opción B — Amenazas activas reportadas recientemente:
→ Busca: "[tu sector] ransomware 2025 TTPs"
→ Busca: "APT [nombre] MITRE techniques"
→ Resultado: una lista de IDs tipo T1059.001, T1486, T1003.001

Opción C — Framework regulatorio (si aplica):
→ TIBER-EU: el regulador te da el threat landscape
→ DORA: TLPT (Threat-Led Penetration Testing) define el scope
```

### Salida de la fase 1

```
Lista de técnicas objetivo para este ejercicio:

Técnica          Fase ATT&CK          Fuente intel
─────────────────────────────────────────────────────
T1059.001        Execution            APT29 campaign Q1 2025
T1003.001        Credential Access    CISA AA25-040A
T1486            Impact               Ransomware group "Lockbit 4"
T1021.001        Lateral Movement     FS-ISAC Q4 2024 advisory
T1562.001        Defense Evasion      APT29 campaign Q1 2025
T1078.002        Persistence          CrowdStrike 2025 Global Report
```

---

## Fase 2: Selección de Tests en Achilles

Con la lista de técnicas, vas al Browser:

```
Browser → Filtro: Técnica ATT&CK

Buscar: "T1059.001"
→ PowerShell Encoded Command        CRÍTICO  Windows  ✅ disponible
→ PowerShell AMSI Bypass            CRÍTICO  Windows  ✅ disponible
→ PowerShell Constrained Mode       ALTO     Windows  ✅ disponible

Buscar: "T1003.001"
→ LSASS Memory Access               CRÍTICO  Windows  ✅ disponible
→ LSASS via Task Manager            ALTO     Windows  ✅ disponible

Buscar: "T1486"
→ Ransomware File Encryption Sim    CRÍTICO  Windows  ✅ disponible

Buscar: "T1021.001"
→ Remote Desktop Protocol Brute     ALTO     Windows  ✅ disponible

Buscar: "T1562.001"
→ Windows Defender Disable          CRÍTICO  Windows  ✅ disponible
→ Audit Policy Modification         MEDIO    Windows  ✅ disponible

Buscar: "T1078.002"
→ Domain Admin Account Abuse        CRÍTICO  Windows  ✅ disponible
```

### Crear una campaña con los tests seleccionados

```
Analytics → [+ Nueva Campaña]

Nombre: "Purple Team — APT29 Q2 2026"

Tests seleccionados (10 tests):
  ☑ PowerShell Encoded Command
  ☑ PowerShell AMSI Bypass
  ☑ LSASS Memory Access
  ☑ LSASS via Task Manager
  ☑ Ransomware File Encryption Sim
  ☑ RDP Brute Force
  ☑ Windows Defender Disable
  ☑ Audit Policy Modification
  ☑ Domain Admin Account Abuse
  ☑ PowerShell Constrained Mode

Destino:
  ● Todas las máquinas Windows (15 máquinas)

Horario:
  ● Esta noche a las 02:00 AM

[ Crear Campaña ]

→ 150 tareas creadas (10 tests × 15 máquinas)
→ Duración estimada: 45 minutos
```

---

## Fase 3: Ejecución Controlada

El día del ejercicio, el equipo completo está coordinado:

```
ANTES de ejecutar:
  Blue Team → abre dashboard de alertas (Defender/SIEM)
  Blue Team → confirma que todos los sistemas de logging están activos
  Red Team  → confirma que los agentes están online en las máquinas objetivo
  Ambos     → acuerdan la ventana de ejecución (ej. 02:00-04:00 AM)

DURANTE la ejecución:
  → Los tests corren automáticamente (campaña programada)
  → Blue Team observa qué alertas llegan y cuándo
  → Ningún equipo interfiere — solo observan

DESPUÉS:
  → Blue Team registra manualmente las detecciones que vio en el SIEM
  → Achilles registra automáticamente los resultados de cada test
  → Se comparan las dos fuentes
```

---

## Fase 4: Análisis de Gaps

Al día siguiente, el análisis:

```
Analytics → Campaña "Purple Team — APT29 Q2 2026" → Ver Resultados

Resumen:
  Tests ejecutados:    150 (10 técnicas × 15 máquinas)
  Protegidos:          87   (58%) ✅
  No detectados:       63   (42%) ❌

Por técnica:
Técnica                    Protegido   No detectado   Cobertura
──────────────────────────────────────────────────────────────
PowerShell Encoded Cmd     15/15       0/15           100% ✅
PowerShell AMSI Bypass     9/15        6/15            60% 🟡
LSASS Memory Access        12/15       3/15            80% 🟡
LSASS via Task Manager     4/15        11/15           27% 🔴
Ransomware Encryption      15/15       0/15           100% ✅
RDP Brute Force            10/15       5/15            67% 🟡
Defender Disable           3/15        12/15           20% 🔴  ← CRÍTICO
Audit Policy Mod.          2/15        13/15           13% 🔴  ← CRÍTICO
Domain Admin Abuse         7/15        8/15            47% 🟡
PS Constrained Mode        10/15       5/15            67% 🟡
```

### Cruzar con las alertas del Blue Team

```
Comparativa: lo que detectó Achilles vs lo que llegó al SIEM

Técnica                 Achilles   SIEM    Diferencia
───────────────────────────────────────────────────────
PowerShell Encoded      100%       100%    ✅ Sin gaps
AMSI Bypass              60%        45%    ⚠️ SIEM pierde 15%
LSASS Memory Access      80%        80%    ✅ Sin gaps
Defender Disable         20%         0%    🔴 SIEM no detecta nada
```

Esta comparativa es la columna vertebral del informe: dónde hay gaps técnicos y dónde hay gaps de visibilidad.

---

## Fase 5: Remediación y Retest

Con los gaps identificados, el equipo prioriza:

```
Prioridad 1 — CRÍTICO (detección < 25%):
  Defender Disable (20%)
  → Acción: Habilitar "Tamper Protection" en Defender
  → Responsable: sysadmin
  → Deadline: esta semana

  Audit Policy Modification (13%)
  → Acción: Configurar alertas en SIEM para cambios de auditoría
  → Responsable: SOC engineer
  → Deadline: esta semana

Prioridad 2 — ALTO (detección < 50%):
  LSASS via Task Manager (27%)
  → Acción: Activar Credential Guard en los 3 servidores que no lo tienen
  → Responsable: sysadmin
  → Deadline: próxima semana
```

### Retest después de la remediación

```
Endpoints → [+ Nueva Campaña]

Nombre: "Retest APT29 — Semana 2"
Tests: Solo las técnicas con cobertura < 50% (4 técnicas)
Destino: Mismas 15 máquinas
Horario: Próximas 24h

→ Ejecutar → Comparar con resultados previos
```

```
Evolución después de la remediación:

Técnica                Antes    Después   Mejora
────────────────────────────────────────────────
Defender Disable        20%      93%      +73% ✅
Audit Policy Mod.       13%      87%      +74% ✅
LSASS via Task Mgr      27%      80%      +53% ✅
Domain Admin Abuse      47%      71%      +24% 🟡
```

---

## El Informe Final

El ejercicio produce un documento defendible con cuatro secciones:

```
INFORME PURPLE TEAM — APT29 Q2 2026

1. Scope
   → 10 técnicas basadas en intel de APT29 (ver CISA AA25-040A)
   → 15 máquinas Windows del entorno de producción
   → Ventana de ejecución: 15 Mayo 2026 02:00–04:00 AM

2. Baseline (antes del ejercicio)
   → Defense Score general: 62%
   → Cobertura de técnicas APT29: 58%
   → Gaps críticos identificados: 3

3. Remediación
   → Tamper Protection activada en 15/15 máquinas
   → 3 reglas SIEM nuevas para Defense Evasion
   → Credential Guard desplegado en 15/15 máquinas

4. Post-remediación
   → Defense Score general: 79%  (+17 puntos)
   → Cobertura de técnicas APT29: 86%  (+28 puntos)
   → Gaps críticos resueltos: 3/3  ✅

Evidencia adjunta:
  → Capturas de pantalla de Achilles Analytics
  → Timestamps de cada test ejecutado
  → Historial de alertas del SIEM durante la ventana
```

---

## Cadencia Recomendada

```
Continuo (automático):
→ Schedules semanales ejecutando técnicas de tu lista de amenazas
→ El Defense Score sube/baja te avisa si algo cambia

Trimestral:
→ Ejercicio Purple Team completo con nueva intel de amenazas
→ Informe formal para dirección o reguladores

Anual:
→ Red Team tradicional (sin aviso previo al Blue)
→ Valida que el Purple Team continuo se traduce en
  detección real de ataques no anticipados
```

---

## Puntos Clave

✅ Purple Team = Red y Blue juntos para optimizar la cobertura, no atacar en secreto
✅ El flujo completo: intel → selección → campaña → análisis → remediación → retest
✅ Las campañas de Achilles estructuran el ejercicio: mismo test en toda la flota a la vez
✅ El cruce Achilles vs SIEM revela tanto gaps técnicos como gaps de visibilidad
✅ El resultado es evidencia cuantitativa y fechada: scores antes/después, técnicas probadas
✅ El mismo flujo satisface los requisitos de TIBER-EU y DORA TLPT

---

## Próximo Post

**P4-02: "Build & Sign — Compilar y Firmar tus Propios Agentes"**

Cómo compilar el agente desde el código fuente, firmarlo con un certificado propio, y por qué importa para entornos con EDR enterprise que bloquean binarios sin firma.

---

**Etiquetas:** #ProjectAchilles #PurpleTeam #MITREATTACK #ThreatIntel #CiberSeguridad #SOC #RedTeam #BlueTeam

---

*Parte 1 de 3 en la serie "Pro — Flujos Avanzados con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
