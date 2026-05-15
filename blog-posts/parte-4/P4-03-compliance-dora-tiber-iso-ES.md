# Compliance: Evidencia para DORA, TIBER-EU e ISO 27001

> **Serie: Pro — Parte 3 de 3**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Avanzado 🔴

---

## TL;DR

- Los reguladores no piden una promesa de seguridad — piden evidencia de que la has medido y mejorado
- Achilles genera exactamente el tipo de evidencia que piden DORA, TIBER-EU e ISO 27001: tests ejecutados, resultados, fechas, mejoras
- DORA exige TLPT (Threat-Led Penetration Testing) cada 3 años para entidades significativas — Achilles es la capa de validación continua entre ejercicios
- TIBER-EU usa el mismo modelo con el regulador como árbitro — los datos de Achilles son la línea base y el cierre
- ISO 27001 Anexo A tiene controles técnicos que se demuestran con tests de detección — Achilles los mapea directamente

---

## Por Qué los Reguladores Piden Evidencia de Tests

Históricamente, la seguridad se "demostraba" con documentos:

```
Auditoría tradicional:
→ "¿Tienes política de contraseñas?"        → Sí (muestra el documento)
→ "¿Tienes EDR instalado?"                  → Sí (muestra la licencia)
→ "¿Tu EDR detecta ataques?"                → ... (silencio)
```

DORA, TIBER-EU y la evolución de ISO 27001 cambian esa lógica:

```
Auditoría moderna:
→ "¿Tu EDR detecta estas técnicas específicas?"
→ Evidencia requerida: tests ejecutados, resultados, timestamps, mejoras

Achilles responde exactamente eso.
```

---

## DORA — Digital Operational Resilience Act (UE)

### ¿A quién aplica?

DORA es obligatoria para entidades financieras reguladas en la UE desde enero 2025:
- Bancos, aseguradoras, gestoras de activos, plataformas de criptoactivos
- Proveedores de TIC críticos que les dan servicio

### Qué pide DORA en materia de testing

```
Artículo 24-27 — Programa de pruebas de resiliencia operativa digital:

Básico (todas las entidades):
→ Tests de herramientas y sistemas TIC al menos una vez al año
→ Evaluaciones de vulnerabilidades y tests de código

Avanzado — TLPT (Threat-Led Penetration Testing):
→ Entidades significativas: al menos cada 3 años
→ Basado en intel de amenazas real
→ Scope definido con el regulador
→ Resultados documentados con plan de remediación
```

### Dónde encaja Achilles en DORA

```
TLPT (cada 3 años)           Validación continua (entre TLPTs)
─────────────────            ──────────────────────────────────
Ejercicio formal con          Achilles ejecutando las mismas
Red Team externo              técnicas del TLPT cada semana

El regulador lo supervisa     Tu equipo mide la cobertura
                              y detecta degradaciones

Resultado: snapshot           Resultado: evidencia de que
formal cada 3 años            la remediación del TLPT se mantiene
```

### Evidencia de Achilles útil para DORA

```
1. Registro de tests ejecutados (Artículo 24.2.b):

   Analytics → Executions → Exportar → Rango: último año

   Columnas exportadas:
   - Fecha y hora exacta de cada test
   - Técnica ATT&CK probada
   - Máquina objetivo
   - Resultado (Protegido / No detectado)
   - Exit code del agente

2. Evolución del Defense Score (Artículo 24.2.c):

   Analytics → Defense Score → Tendencia 12 meses → Exportar

   Demuestra que la cobertura se mantiene o mejora con el tiempo.

3. Correlación con alertas reales (Artículo 25 — "evaluación de efectividad"):

   Analytics → Defender → Cross-Correlation → Exportar

   Muestra qué técnicas generaron alertas en el SIEM real.
   Esto conecta el test con la capacidad operacional del SOC.

4. Campañas de remediación (plan de mejora):

   Analytics → Campañas → [nombre campaña] → Comparativa antes/después

   Antes: cobertura X% — Después: cobertura Y%
   Demuestra que los gaps identificados fueron remediados.
```

---

## TIBER-EU — Threat Intelligence-Based Ethical Red-Teaming

### ¿Qué es TIBER-EU?

Framework del BCE (Banco Central Europeo) para tests de resiliencia en infraestructuras financieras críticas. Algunos países lo llaman TIBER-NL (Países Bajos), TIBER-DE (Alemania), TIBER-DK (Dinamarca), etc.

### Las tres fases de TIBER-EU

```
FASE 1 — Preparación:
→ El banco contrata un proveedor de Threat Intelligence (TI Provider)
→ El TI Provider entrega un informe de amenazas específico para el banco
→ Define el scope del ejercicio

FASE 2 — Testing:
→ El banco contrata un equipo Red Team externo
→ El Red Team ejecuta los ataques basados en la intel
→ El Blue Team NO sabe cuándo (ejercicio blind)
→ TIBER Cyber Team (regulador) supervisa

FASE 3 — Cierre:
→ El Red Team entrega el informe completo
→ El banco entrega el plan de remediación al regulador
→ Retest formal para verificar las remedaciones
```

### Dónde encaja Achilles en TIBER-EU

```
ANTES del ejercicio TIBER:
→ Usa Achilles para establecer la línea base
→ "Antes de que llegue el Red Team, nuestra cobertura es X%"
→ El informe de Achilles sirve como contexto para el TI Provider

DURANTE el ejercicio TIBER:
→ Achilles no interfiere — el ejercicio es blind para el Blue Team
→ Continúa ejecutando los schedules automáticos normales

DESPUÉS del ejercicio TIBER:
→ El Red Team entregó el informe con las técnicas usadas
→ Mapeas esas técnicas en Achilles
→ Ejecutas la campaña de retest formal
→ Achilles genera la evidencia cuantitativa de la remediación
→ "Post-TIBER: cobertura en técnicas del ejercicio pasó de 41% a 89%"
```

### Evidencia de Achilles útil para TIBER-EU

```
Documento: Línea base pre-ejercicio
  → Analytics → Defense Score → Snapshot en fecha [dd/mm/yyyy]
  → Técnicas TIBER relevantes identificadas en la librería
  → Cobertura actual de esas técnicas específicas

Documento: Plan de remediación post-TIBER
  → Gaps identificados por el Red Team → cruzados con tests en Achilles
  → Campaña de remediación con timestamps

Documento: Evidencia de cierre
  → Comparativa antes/después para cada técnica TIBER
  → Defense Score evolution desde el día del ejercicio
  → Frecuencia de tests desde la remediación (schedules activos)
```

---

## ISO 27001:2022 — Anexo A

### Los controles relevantes

ISO 27001:2022 tiene 93 controles en el Anexo A. Los que Achilles cubre directamente:

```
Control 8.8 — Gestión de vulnerabilidades técnicas
  Exige: identificar y gestionar vulnerabilidades en tiempo oportuno
  Achilles aporta: tests que confirman si las vulnerabilidades conocidas
  son explotables en tu entorno (vs solo existencia teórica)

Control 8.20 — Seguridad en redes
  Exige: protección adecuada de la red
  Achilles aporta: tests de técnicas de Lateral Movement y Discovery
  que validan si la segmentación de red funciona como se supone

Control 8.29 — Pruebas de seguridad en el desarrollo
  Exige: pruebas de seguridad durante el ciclo de vida
  Achilles aporta: entornos de staging/QA con agentes desplegados
  y tests ejecutados antes de pasar a producción

Control 8.8 — Registro de eventos / logs
  Exige: registrar eventos y conservarlos
  Achilles aporta: evidencia de que los logs realmente capturan
  eventos de seguridad (vía correlación con Defender/SIEM)

Control 5.37 — Procedimientos operativos documentados
  Exige: documentar procedimientos de operación seguros
  Achilles aporta: schedules automáticos = evidencia de que
  el procedimiento de validación está sistematizado, no es manual
```

### Cómo estructurar la evidencia para ISO 27001

```
Auditor: "¿Cómo gestionáis las vulnerabilidades técnicas?"

Respuesta con evidencia:
1. Mostramos el schedule de validación semanal (imagen del dashboard)
2. Mostramos el Defense Score del último trimestre (exportado)
3. Mostramos las campañas de remediación cuando se identificaron gaps
4. Mostramos que los gaps críticos fueron resueltos en < 30 días

Esto demuestra que hay un proceso continuo y medible,
no una validación puntual anual.
```

---

## Construir el Paquete de Evidencia

### Qué incluir en el paquete

```
Paquete estándar de evidencia (válido para los tres marcos):

1. RESUMEN EJECUTIVO (1 página)
   → Período cubierto: [fecha inicio] - [fecha fin]
   → Número de tests ejecutados: X
   → Defense Score inicio / fin del período
   → Número de gaps identificados y resueltos

2. REGISTRO DE ACTIVIDAD
   → Export de Executions (CSV/JSON)
   → Columnas: fecha, técnica, máquina, resultado
   → Firmado con la fecha de export

3. EVOLUCIÓN DEL DEFENSE SCORE
   → Gráfico de tendencia 12 meses (captura de pantalla)
   → Defense Score desglosado por tácticas ATT&CK

4. GAPS Y REMEDIACIÓN
   → Lista de técnicas con cobertura < 70% identificadas
   → Acciones de remediación tomadas (con fecha)
   → Retest: cobertura post-remediación

5. CORRELACIÓN CON ALERTAS REALES (si tienes Defender)
   → % de tests que generaron alerta en el SIEM
   → Validación de que la detección es operacional, no solo técnica

6. CONFIGURACIÓN DEL PROCESO
   → Captura de los schedules activos (evidencia de continuidad)
   → Política de respuesta a caídas del Defense Score (thresholds)
```

### Exportar desde el dashboard

```
Analytics → Executions → Exportar

Formato:      CSV  (para Excel/auditorías) o JSON (para sistemas externos)
Rango:        Últimos 12 meses
Incluir:      Todos los tests / Solo fallidos / Solo críticos

[ Exportar ]

→ achilles-executions-2025-05-15.csv  (ejemplo: 3,450 filas)
```

```
Analytics → Defense Score → Exportar resumen

Incluye:
→ Score por semana (52 puntos de datos)
→ Score desglosado por táctica ATT&CK
→ Número de tests y máquinas en cada período

[ Exportar ]

→ achilles-defense-score-2025-2026.json
```

---

## Frecuencias Recomendadas por Marco

```
Marco       Frecuencia mínima exigida    Recomendado con Achilles
──────────────────────────────────────────────────────────────────
ISO 27001   Anual (Control 8.8)          Semanal (schedule automático)
DORA TLPT   Cada 3 años (formal)         Continuo entre TLPTs
DORA básico Anual                        Mensual + alertas de caída
TIBER-EU    Cada 2-3 años               Continuo entre ejercicios
```

La diferencia entre "cumplir el mínimo" y "estar bien protegido" es exactamente esa: los marcos exigen mínimos anuales, pero los ataques reales ocurren cada semana.

---

## Puntos Clave

✅ DORA exige TLPT cada 3 años + validación continua — Achilles cubre la parte continua
✅ TIBER-EU: Achilles establece la línea base antes del ejercicio y la evidencia de cierre después
✅ ISO 27001 Anexo A: los controls 8.8, 8.20 y 8.29 se demuestran directamente con los exports de Achilles
✅ El paquete de evidencia tiene 6 secciones: resumen, actividad, score, gaps/remediación, correlación, proceso
✅ Los exports CSV/JSON de Analytics son la evidencia — fechados, completos, exportables
✅ La diferencia entre cumplir el mínimo y estar protegido es la frecuencia: semanal, no anual

---

## Cierre de la Serie

Con la Parte 4 completada, tienes el cuadro completo:

```
Parte 1 — Quickstart:
  ✅ Achilles instalado y funcionando
  ✅ Primera máquina conectada y primer test ejecutado
  ✅ Defense Score entendido

Parte 2 — En Profundidad:
  ✅ Heatmap de gaps y visualización de cobertura
  ✅ Bundle tests para auditorías automáticas
  ✅ Alertas automáticas cuando algo baja
  ✅ Librería completa de 500+ técnicas

Parte 3 — Integraciones:
  ✅ Microsoft Defender correlacionado
  ✅ Auto-Resolve para el SOC
  ✅ Flota de agentes gestionada a escala

Parte 4 — Pro:
  ✅ Purple Team workflow completo con evidencia
  ✅ Agentes compilados y firmados con tu certificado
  ✅ Paquetes de evidencia para DORA, TIBER-EU e ISO 27001
```

---

**Etiquetas:** #ProjectAchilles #DORA #TIBEEU #ISO27001 #Compliance #Regulatorio #CiberSeguridad #PurpleTeam #Evidencia

---

*Parte 3 de 3 en la serie "Pro — Flujos Avanzados con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
