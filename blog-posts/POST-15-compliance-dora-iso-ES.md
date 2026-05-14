# Compliance con Achilles: DORA, TIBER-EU e ISO 27001

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 15 de 15**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- DORA (Art. 25), TIBER-EU e ISO 27001 requieren **evidencia técnica de pruebas de resiliencia**
- Achilles genera esa evidencia automáticamente: Defense Score histórico + heatmaps + test logs
- El workflow de compliance tiene 3 etapas: baseline → ejecución continua → paquete de evidencia
- No reemplaza un pentest formal, pero proporciona **evidencia continua entre engagements**
- Apache 2.0: sin costes de licencia que justificar ante el auditor

---

## El Contexto Regulatorio

### DORA (Digital Operational Resilience Act)

Para entidades financieras en la UE (banca, seguros, exchanges, fintechs):

```
DORA Art. 25 — Threat-Led Penetration Testing (TLPT):
"Las entidades financieras significativas deberán realizar
pruebas de penetración basadas en amenazas (TLPT) al menos
cada tres años."

Requisitos clave:
→ Tests basados en inteligencia de amenazas real
→ Cobertura de sistemas críticos
→ Documentación y evidencia del proceso
→ Informe para la autoridad competente (BCE, BdE, etc.)

¿Cómo ayuda Achilles?
→ Proporciona el ciclo continuo de validación entre TLPTs
→ Genera evidencia técnica del estado de la resiliencia
→ Los tests están mapeados a técnicas APT reales (el requisito de TLPT)
```

### TIBER-EU

Marco europeo para pruebas de resiliencia del sector financiero:

```
TIBER-EU fases:
1. Threat Intelligence Phase (TI): contratar proveedor de intel
2. Red Team Phase (RT): ejercicio de red team basado en el intel
3. Purple Team Phase (PT): mejora y validación

Achilles encaja en la fase Purple Team:
→ Valida que las técnicas del Red Team no volverían a funcionar
→ Evidencia del "test-fix-verify" loop
→ Métricas continuas post-TIBER
```

### ISO 27001:2022

Controles relevantes donde Achilles genera evidencia:

```
A.8.8  Management of technical vulnerabilities
→ Achilles: Tests de técnicas de explotación de vulnerabilidades

A.8.29 Security testing in development and acceptance
→ Achilles: Validación antes de producción

A.5.37 Documented operating procedures  
→ Achilles: Logs automáticos de todos los tests ejecutados

A.5.23 Information security for use of cloud services
→ Achilles: Validación de controles en infraestructura cloud
```

---

## Fase 1: Establecer el Baseline

Antes de poder demostrar mejora, necesitas un punto de referencia.

### Semana 1: Primera ejecución completa

```
Objetivo: Medir el estado actual sin cambiar nada

1. Instalar agentes en todos los sistemas críticos
2. Ejecutar la suite completa de técnicas MITRE
3. Documentar el Defense Score baseline

Resultado inicial típico:
  Defense Score: 62.3%
  Técnicas cubiertas: 189/427
  Gaps críticos: 47
  Hosts evaluados: 28
```

### Documentar el baseline

```markdown
# BASELINE DE SEGURIDAD — ISO 27001 / DORA
Organización: [Nombre]
Fecha: 2026-01-15
Evaluador: Achilles v1.4.2

## Métricas Base
- Defense Score global: 62.3%
- Técnicas evaluadas: 427
- Técnicas con cobertura: 189 (44.3%)
- Hosts evaluados: 28
- Plataformas: 24 Windows, 4 Linux

## Distribución por Severidad
- CRITICAL: 41.2% protegido (7/17 técnicas críticas)
- HIGH: 58.7% protegido (54/92 técnicas)
- MEDIUM: 68.1% protegido (89/131 técnicas)
- LOW: 82.3% protegido (79/96 técnicas)

## Top 10 Gaps Críticos
[Lista de técnicas con mayor riesgo]

## Periodo de referencia para auditoría
Este baseline es la referencia para medir el progreso
durante el período de certificación ISO 27001.
```

---

## Fase 2: Ejecución Continua (el corazón del compliance)

La evidencia de compliance no es una foto — es una película. Los auditores quieren ver continuidad.

### Programación automática

```
Schedules configurados para compliance continuo:

Diario (low-impact, silencioso):
→ Suite de Discovery y Collection
→ Bajo impacto operacional, alta frecuencia de datos

Semanal (core coverage):
→ Suite completa de técnicas MITRE
→ Todos los hosts, todos los lunes a las 02:00 AM
→ Resultado: Defense Score actualizado cada semana

Mensual (técnicas sector-specific):
→ Bundle relevante para el sector (banca, salud, gobierno)
→ Primero de cada mes

Trimestral (simulaciones APT completas):
→ Bundle de campaña APT completa
→ Ejercicio formal con informe
```

### El registro de actividad (el audit trail)

Cada test ejecutado genera un documento ES inmutable con:

```json
{
  "f0rtika": {
    "test_uuid": "T1059.001-powershell-exec",
    "technique_id": "T1059.001",
    "tactic": "Execution",
    "severity": "high",
    "exit_code": 1,
    "protected": true,
    "hostname": "DESKTOP-SRV01",
    "platform": "windows",
    "agent_version": "1.4.2"
  },
  "@timestamp": "2026-03-15T02:01:23Z",
  "executed_by": "achilles-backend",
  "framework_tags": ["DORA-Art25", "MITRE-T1059", "CIS-IG2"]
}
```

Este documento es el **registro de evidencia** — reproducible, con timestamp, inmutable.

---

## Fase 3: Construir el Paquete de Evidencia

Cuando llega la auditoría, el paquete de evidencia Achilles contiene:

### Evidencia 1: Defense Score histórico

```
Exportar: Analytics → Defense Score Trend → [Exportar CSV/PDF]

Período: Enero 2026 → Mayo 2026

Mes     Defense Score    Cambio
───────────────────────────────
Ene     62.3%            baseline
Feb     65.8%            +3.5%    Nuevas reglas SIEM
Mar     71.2%            +5.4%    EDR policies actualizadas
Abr     73.8%            +2.6%    Hardening Windows
May     77.1%            +3.3%    MFA implementado

Mejora total: +14.8% en 5 meses
```

### Evidencia 2: Heatmap MITRE en el tiempo

```
[Heatmap Enero 2026 - antes]
Muchas celdas rojas en Execution, Defense Evasion, Credential Access

[Heatmap Mayo 2026 - después]
Significativamente más verde, especialmente en las áreas trabajadas

→ Visualización de la mejora en cobertura de detección
```

### Evidencia 3: Log de tests ejecutados

```
Exportar: Analytics → Executions → [Exportar últimos 6 meses]

Total tests ejecutados: 15,847
Tests únicos validados: 427 técnicas MITRE
Hosts evaluados:        28
Frecuencia media:       182 tests/día

→ Evidencia de que el proceso es continuo, no puntual
```

### Evidencia 4: Plan de remediación y acciones

```
Tabla de gaps identificados y cerrados:

Técnica      Gap detectado    Acción tomada          Verificado
T1078        2026-01-20      MFA implementado        2026-02-05 ✅
T1055.001    2026-01-20      EDR config actualizada  2026-03-10 ✅
T1059 AMSI   2026-02-15      AMSI config revisada    2026-04-01 ✅
T1041        2026-01-20      Regla SIEM añadida      En progreso ⏳
```

---

## Para Auditorías DORA Art. 25

El argumento para el auditor:

```
Organización: "Nuestro programa de TLPT está soportado por validación
               continua entre los ejercicios formales."

Evidencia:
1. Tests ejecutados: 15,847 en los últimos 6 meses
2. Fuentes de intel usadas: CISA, TIBER, threat reports sectoriales
3. Defense Score histórico: 62.3% → 77.1% (+14.8%)
4. Cobertura de técnicas MITRE: 44.3% → 68.7%
5. Gaps identificados y resueltos: 23 gaps cerrados, 3 en progreso
6. Todos los tests mapeados a técnicas MITRE ATT&CK v15

Marco de referencia: TIBER-EU Purple Team Phase documentation
Herramienta: Project Achilles (Apache 2.0, self-hosted)
```

---

## Para Auditorías ISO 27001

```
Control A.8.8 — Gestión de vulnerabilidades técnicas:
"Achilles ejecuta 427+ tests mapeados a técnicas de explotación.
Defense Score: 77.1% (técnicas detectadas exitosamente).
Proceso de remediación documentado para cada gap identificado."

Control A.8.29 — Pruebas de seguridad:
"Validación técnica ejecutada semanalmente contra todos los
sistemas en alcance. Logs disponibles en Elasticsearch."

Control A.5.37 — Procedimientos operativos documentados:
"CLAUDE.md documenta el proceso de build, sign, deploy y
measurement. Achilles genera audit trail automático por cada
ejecución."
```

---

## Preguntas Frecuentes de Auditores

**"¿Esto reemplaza el pentest anual?"**
```
No. Achilles complementa el pentest:
→ Pentest: perspectiva externa, creatividad humana, vulnerabilidades
           de negocio, ingeniería social
→ Achilles: cobertura continua, métricas objetivas, validación de defensas

Los mejores programas usan ambos.
```

**"¿Los tests pueden afectar sistemas de producción?"**
```
Los tests son binarios que simulan técnicas APT.
→ Diseñados para ser inofensivos en sistemas de producción
→ No modifican datos, no escalan privilegios realmente
→ Ejecutados en ventanas de mantenimiento configurables
→ Pueden ser revertidos inmediatamente via gestión de agentes
```

**"¿Cómo verificamos la integridad de los resultados?"**
```
→ Los documentos ES tienen timestamps inmutables
→ Los binarios de test están firmados (Authenticode/ad-hoc)
→ El agente verifica la firma antes de ejecutar
→ El backend almacena el hash del binario ejecutado
→ Todo el tráfico agente→backend va firmado con Ed25519
```

---

## Achilles vs Alternativas para Compliance

```
Evidencia para DORA/TIBER/ISO:

AttackIQ ($200K+/año):
→ Sí, genera evidencia comparable
→ Vendor lock-in
→ Difícil de auditar el proceso interno (caja negra parcial)

Atomic Red Team (gratis):
→ Scripts, no binarios firmados
→ Sin métricas automáticas
→ Require mucho trabajo manual para generar evidencia
→ Menos representativo de APT real

Achilles (gratis, Apache 2.0):
→ Binarios firmados que simulan APT real
→ Métricas automáticas en Elasticsearch
→ Código abierto: el auditor puede verificar el proceso
→ Self-hosted: los datos no salen de tu infraestructura
```

---

## Cierre de la Serie

Has llegado al final de la serie "Validación Continua de Seguridad con Project Achilles". Repasamos:

```
POST 01: ¿Qué es Project Achilles?
POST 02: Arquitectura del stack
POST 03: Instalación con Docker Compose
POST 04: El agente Go — enrolamiento y flota
POST 05: Librería de tests MITRE ATT&CK
POST 06: Tu primer test de seguridad
POST 07: Defense Score — la métrica clave
POST 08: Heatmap y treemap de cobertura
POST 09: Bundle tests
POST 10: Integración Microsoft Defender
POST 11: Auto-Resolve — reducir noise del SOC
POST 12: Build & Sign para 6 plataformas
POST 13: Alerting — notificaciones proactivas
POST 14: Purple Team workflow completo
POST 15: DORA, TIBER-EU, ISO 27001 (estás aquí)
```

**El mensaje central de la serie:**

No tienes que esperar un breach para saber si tus defensas funcionan. Con Project Achilles, puedes saberlo hoy, mañana, y cada semana — con evidencia objetiva, reproducible, y sin gastar $200K en una solución enterprise.

> "Para de esperar que tus defensas funcionen. Empieza a probarlo."

---

## Recursos Finales

- 🔗 GitHub: https://github.com/projectachilles/achilles
- 🔗 Documentación: https://projectachilles.io
- 🔗 Discord (comunidad): [Link en bio]
- 🔗 DORA compliance guide: docs/compliance/DORA.md

---

**Etiquetas:** #ProjectAchilles #DORA #TIBER #ISO27001 #Compliance #CiberSeguridad #PurpleTeam #MITRE #Regulación #Banca

---

*Parte 15 de 15 en la serie "Validación Continua de Seguridad con Project Achilles". ¡Gracias por leer!*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
