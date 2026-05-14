# Defense Score: Tu Primera Métrica Real de Seguridad

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 7 de 15**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- El **Defense Score** es el porcentaje de técnicas MITRE ATT&CK que tu stack detecta actualmente
- Se calcula en tiempo real a partir de los resultados almacenados en Elasticsearch
- Un score del 100% no existe — el objetivo es mejorar continuamente y conocer tus gaps
- Los filtros por host, táctica, severidad y fecha permiten segmentar el análisis
- El score es la métrica que llevas al CISO y al auditor: **evidencia, no promesas**

---

## El Problema con las Métricas de Seguridad Tradicionales

```
Métricas típicas en muchos SOCs:

→ "Tenemos Defender P2 + Sentinel"         (herramienta, no resultado)
→ "Respondimos 847 alertas este mes"       (actividad, no cobertura)
→ "98.7% de tiempo de uptime del SIEM"     (disponibilidad, no efectividad)
→ "Cumplimos ISO 27001"                    (proceso, no validación técnica)

Pregunta que ninguna de estas responde:
"¿Cuántas de las técnicas que usa APT29 detectarías HOY?"
```

El **Defense Score** responde exactamente esa pregunta.

---

## Cómo Se Calcula el Defense Score

```
Defense Score = (Tests protegidos / Tests totales ejecutados) × 100

Donde:
  Tests protegidos = Tests con exit_code 1 (bloqueados/detectados)
  Tests totales    = Tests con exit_code 0 o 1 (excluye exit_code 2: errores)

Ejemplo:
  Tests ejecutados este mes:    312
  Tests con exit_code 1 (✅):   228
  Tests con exit_code 0 (❌):    84
  Tests con exit_code 2 (⚠️):    12  ← excluidos del cálculo

  Defense Score = 228 / 312 × 100 = 73.1%
```

---

## Los Hero Metrics: El Dashboard Principal

```
┌────────────────────────────────────────────────────────────────────┐
│                         ACHILLES ANALYTICS                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│   Defense Score      Tests Totales    Tests Fallidos    Endpoints  │
│                                                                    │
│     73.1%              1,847              312              23      │
│    ↑ 4.2% ↑           Últimos 30d        16.9%           online   │
│   ████████░░░                                                      │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Defense Score**: el número que importa. La barra de progreso te da visión inmediata.

**Tests Fallidos**: no son "errores" — son tests donde `exit_code 0` indicó que el ataque no fue detectado. Son tus **gaps de cobertura**.

---

## Segmentar el Score: Filtros Disponibles

El Defense Score global no siempre es el dato más útil. Los filtros permiten ir al detalle:

### Por Host

```
Analytics → Filtro: Host = "DESKTOP-SRV01"

Defense Score DESKTOP-SRV01: 81.3%  ↑
Defense Score DESKTOP-SRV02: 62.4%  ↓

→ SRV02 tiene configuración diferente o política más permisiva
→ Necesita revisión prioritaria
```

### Por Táctica MITRE

```
Analytics → Filtro: Táctica = "Defense Evasion"

Defense Score en Defense Evasion: 41.8%  🔴 CRÍTICO

→ Tienes gaps graves en la táctica más usada por APTs
→ 78 técnicas en esta táctica: solo detectas 33

Vs.

Defense Score en Initial Access: 88.2%  🟢
→ Tu perímetro está bien defendido
```

### Por Severidad

```
Analytics → Filtro: Severidad = "Critical"

Defense Score en Critical: 67.3%

→ De las técnicas más dañinas, detectas 2 de cada 3
→ 1 de cada 3 técnicas críticas pasaría desapercibida
```

### Por Período de Tiempo

```
Analytics → Filtros de fecha: Últimos 7 días vs Últimos 30 días

Últimos 7 días:  Defense Score 76.4%  ↑
Últimos 30 días: Defense Score 73.1%
Últimos 90 días: Defense Score 68.9%

→ Tendencia positiva: +7.5% en 3 meses
→ El trabajo de hardening tiene impacto medible
```

---

## El Trend Chart: Evolución en el Tiempo

```
Defense Score (últimos 90 días)

85% ┤
    │
80% ┤                                              ╭─────
    │                                         ╭───╯
75% ┤                               ╭─────────╯
    │                    ╭──────────╯
70% ┤               ╭────╯
    │     ╭──────────╯
65% ┤─────╯
    │
60% ┼─────────────────────────────────────────────────────
    Feb              Mar              Abr            May

Eventos anotados:
→ Mar 15: Implementamos Script Block Logging      +3.2%
→ Abr 02: Actualizamos reglas EDR (APT29 package) +4.1%
→ Abr 28: Habilitamos ASR Rules                   +2.8%
```

Cada acción de hardening tiene un impacto medible en el score. Esto es lo que le presentas al CISO.

---

## Defense Score por Host: Detectar los Más Vulnerables

```
Analytics → Defense Score por Host (bar chart)

DESKTOP-SRV01  ████████████████████  84.2%  🟢
DESKTOP-SRV02  █████████████████     72.1%  🟡
LINUX-PROD-01  ████████████████      67.8%  🟡
DESKTOP-OLD01  ████████████          51.3%  🔴  ← Prioridad
MACBOOK-DEV01  █████████████████     70.4%  🟡
```

Un host con Defense Score bajo puede indicar:
- Versión de OS desactualizada (sin parches de seguridad)
- Configuración de EDR incompleta (políticas heredadas)
- Excepción de AV/EDR (demasiado "permissive")
- Máquina no incluida en las políticas de grupo

---

## Top Gaps: Los 10 Tests Más Fallidos

```
Analytics → Top Controls (más fallidos)

Técnica         Fallos  Hosts afectados   Severidad
──────────────────────────────────────────────────
T1055.001        18/23       23           CRITICAL
T1078.002        15/23       23           CRITICAL
T1134.001        14/23       19           HIGH
T1140            12/23       23           HIGH
T1218.011        11/23       20           HIGH
T1036.005        10/23       17           MEDIUM
T1562.001         9/23       14           HIGH
T1070.004         9/23       16           MEDIUM
T1083             8/23       21           LOW
T1012             7/23       18           LOW
```

Esta tabla es tu **backlog de hardening priorizado**. Empieza por los CRITICAL con más hosts afectados.

---

## Defense Score vs Secure Score de Microsoft

Si tienes la integración con Defender activa (POST 10), verás ambas métricas juntas:

```
┌─────────────────────────────────────────────────────────────────┐
│  Defense Score (Achilles)        Secure Score (Microsoft)       │
│         73.1%                            62.0%                  │
│      ████████░░░                      ██████░░░░                │
│   Basado en técnicas MITRE         Basado en controles          │
│   ejecutadas contra tu flota       de configuración             │
└─────────────────────────────────────────────────────────────────┘
```

**¿Cuál es más importante?**

```
Defense Score (Achilles):
→ Mide si detectas ataques REALES ejecutados contra tu entorno
→ Basado en evidencia empírica
→ Más difícil de subir (requiere hardening técnico)

Secure Score (Microsoft):
→ Mide si sigues las recomendaciones de configuración de Microsoft
→ Basado en checkboxes de configuración
→ Más fácil de subir (implementar recomendaciones)

Correlación ideal: ambos scores suben juntos
Si Secure Score alto + Defense Score bajo: las recomendaciones no bastan
Si Defense Score alto + Secure Score bajo: configuración no ortodoxa pero funcional
```

---

## Presentar el Defense Score al CISO

El Defense Score convierte un tema técnico en una conversación ejecutiva:

```
Antes de Achilles:
CISO: "¿Cómo estamos de seguros?"
SOC:  "Tenemos Defender P2 y 847 alertas este mes..."
CISO: [No tiene con qué tomar decisiones]

Con Achilles:
CISO: "¿Cómo estamos de seguros?"
SOC:  "Defense Score actual: 73.1%
       ↑ 4.2% vs mes pasado gracias a las nuevas reglas EDR.
       
       Gaps críticos pendientes:
       - Process Injection (T1055): 23/23 hosts vulnerables
       - Valid Accounts (T1078): requiere MFA urgente
       
       Inversión propuesta: $15K en formación para subir a ~82%
       Impacto estimado: cubrir las 15 técnicas críticas restantes"

CISO: [Puede tomar una decisión informada]
```

---

## Defense Score para Auditorías DORA/ISO 27001

Para organizaciones financieras bajo DORA (art. 25) o con ISO 27001, el Defense Score es evidencia tangible:

```
Evidencia para auditor DORA:
"Ejecutamos 312 tests de validación técnica en los últimos 30 días,
mapeados a técnicas MITRE ATT&CK. El 73.1% fueron detectadas exitosamente.
Los 26.9% restantes están documentados como gaps con plan de remediación."

→ Más sólido que: "Tenemos un SIEM y un EDR"
→ Satisface el requisito de "pruebas de resiliencia operativa"
```

---

## Benchmarks: ¿Qué Score Es Bueno?

No existe un "número mágico", pero estos rangos son orientativos:

```
< 50%   🔴 Crítico     → Gaps fundamentales, riesgo alto de no detección
50-65%  🟠 Bajo        → Configuración básica, muchas áreas sin cobertura  
65-75%  🟡 Moderado    → Línea base aceptable, oportunidades claras de mejora
75-85%  🟢 Bueno       → Cobertura sólida, enfoque en gaps específicos
85-95%  🟢 Excelente   → Madurez avanzada, optimización continua
> 95%   🔵 Elite       → Very rare. Indica profundidad + amplitud de cobertura
```

El objetivo no es llegar al 100% — es **mejorar continuamente** y **conocer exactamente dónde están tus gaps**.

---

## Puntos Clave

✅ Defense Score = % de técnicas MITRE que tu stack detecta hoy
✅ Calculado en tiempo real desde Elasticsearch — siempre actualizado
✅ Filtra por host, táctica, severidad y fecha para análisis granular
✅ El Trend Chart convierte el hardening en evidencia visual de mejora
✅ Top Gaps = backlog priorizado de acciones de seguridad
✅ Lenguaje ejecutivo: métrica clara para CISOs y auditores DORA/ISO

---

## Próximo Post

**POST 08: "Heatmap MITRE y Treemap de Cobertura — Visualizar tus Gaps"**

Cómo usar las visualizaciones avanzadas de Achilles para entender tu cobertura de un vistazo y comunicarla a cualquier audiencia.

---

**Etiquetas:** #ProjectAchilles #DefenseScore #MITRE #CISO #Métricas #SOC #DORA #ISO27001 #CiberSeguridad

---

*Parte 7 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
