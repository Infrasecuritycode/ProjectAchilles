# Ver Tus Gaps de Seguridad de un Vistazo: El Heatmap y el Treemap

> **Serie: Usando Achilles en Profundidad — Parte 1 de 4**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- El **Heatmap** te muestra tu cobertura sobre el mapa de técnicas de ataque más conocido del mundo (MITRE ATT&CK)
- El **Treemap** te muestra lo mismo pero priorizando visualmente lo más crítico (más grande = más urgente)
- Ambas visualizaciones están en Analytics y se actualizan con cada test ejecutado
- Son la mejor herramienta para comunicar gaps a dirección o a un auditor con una sola imagen
- No necesitas saber qué es MITRE ATT&CK para leerlos, este post te lo explica

---

## Primero: ¿Qué Es MITRE ATT&CK?

Antes de ver el heatmap, necesitas entender qué estás mirando.

**MITRE ATT&CK** es básicamente un catálogo, una lista organizada de todas las técnicas que los hackers reales usan para atacar organizaciones. Lo mantiene una organización sin ánimo de lucro llamada MITRE, con información real de incidentes documentados en todo el mundo.

El catálogo organiza las técnicas en fases del ataque:

```
¿Cómo entra el hacker?  → Initial Access (ej: phishing, exploit web)
¿Cómo ejecuta código?   → Execution (ej: PowerShell, scripts)
¿Cómo se queda?        → Persistence (ej: registro de Windows, tareas programadas)
¿Cómo escala permisos? → Privilege Escalation
¿Cómo se esconde?      → Defense Evasion  ← donde más gaps suele haber
¿Cómo roba contraseñas?→ Credential Access
¿Cómo espía la red?    → Discovery
¿Cómo se mueve?        → Lateral Movement
¿Cómo roba datos?      → Collection / Exfiltration
¿Cómo daña?            → Impact (ej: ransomware)
```

Achilles tiene tests para la mayoría de estas técnicas. El Heatmap te muestra cuáles de ellas detectas tú.

---

## El Heatmap: Tu Radiografía de Seguridad

```
Analytics → [pestaña Overview] → scroll hasta "Coverage Heatmap"
```

Verás una cuadrícula de colores:

```
                T1059  T1078  T1021  T1055  T1547
Initial Access  [🟢]   [🔴]   [ ]    [ ]    [ ]
Execution       [🟢]   [ ]    [ ]    [ ]    [ ]
Persistence     [ ]    [🔴]   [ ]    [ ]    [🔴]
Defense Evasion [ ]    [ ]    [ ]    [🔴]   [ ]
Credential Acc  [ ]    [🔴]   [ ]    [ ]    [ ]
Lateral Movement[ ]    [ ]    [🟢]   [ ]    [ ]

🟢 Detectado    🟡 A veces    🔴 No detectado    [ ] Sin probar
```

**Cómo leer esto:**

- Cada **fila** es una fase del ataque
- Cada **columna** es una técnica específica
- El **color** de cada celda dice si la detectas o no

**Lo que buscar:**
- Un bloque rojo grande en una fila → tienes una fase del ataque mal cubierta
- Una columna toda roja → una técnica que afecta a múltiples fases de ataque
- Muchas celdas grises/vacías → muchas técnicas que nunca has probado

---

## Los Filtros del Heatmap

Los botones encima del heatmap te dejan cambiar la vista:

```
[ Todos ] [ Solo protegidos ] [ Solo gaps ] [ Solo parciales ]
```

**El más útil:** "Solo gaps", muestra únicamente dónde fallas. Ideal para una reunión de seguridad o para presentar al CISO.

---

## El Treemap: Priorizar en 5 Segundos

```
Analytics → scroll hasta "Coverage Treemap"
```

El treemap es el mismo dato pero en otro formato:

```
┌──────────────────────────────────────────────────────┐
│                                │              │      │
│   Defense Evasion              │  Execution   │ Cred │
│   🔴  41%                      │  🟡  68%     │ 🔴   │
│   (rectángulo grande           │              │ 52%  │
│    = más técnicas críticas      ├──────────────┤      │
│    sin cobertura)              │ Persistence  │      │
│                                │ 🟡  61%      │      │
├────────────────────────────────┴──────────────┴──────┤
│ Initial Access 🟢 88%  │ Lateral Movement 🟢 79%    │
└──────────────────────────────────────────────────────┘
```

**El tamaño del rectángulo** = qué tan importante es esa área (basado en cantidad de técnicas y criticidad).

**La lectura es instantánea:** el rectángulo rojo más grande es tu prioridad #1.

En el ejemplo de arriba: Defense Evasion es el área más crítica con menor cobertura, ahí es donde enfocar el trabajo.

---

## Por Qué el Treemap Es Mejor para Presentar a Dirección

Una tabla de 427 técnicas es imposible de procesar para alguien no técnico. El treemap colapsa esa complejidad en una imagen:

```
Antes:
"Tenemos gaps en T1055.001, T1134.001, T1218.011, T1036.005, T1562.001..."
[El director desconecta]

Con el treemap:
"Este cuadro rojo grande es Defense Evasion — nuestra área más débil.
Los cuadros verdes son áreas bien cubiertas. Vamos a enfocar este mes en
reducir ese rojo."
[El director entiende y puede tomar decisiones]
```

---

## El Gráfico de Severidad: ¿Qué Tan Críticos Son Tus Gaps?

Debajo del treemap hay un gráfico de barras que divide los resultados por nivel de riesgo:

```
Distribución por severidad:

CRÍTICO  ██████████░░░░░░░░░   58% protegido   42% con gaps
ALTO     ████████████░░░░░░░   67% protegido   33% con gaps
MEDIO    █████████████████░░   84% protegido   16% con gaps
BAJO     ████████████████████  94% protegido    6% con gaps
```

Esto es esperado y normal: las técnicas más críticas (las que usan los hackers más avanzados) suelen ser las más difíciles de detectar.

**La señal de alarma:** si CRÍTICO está por debajo de 40%, es prioritario.

---

## Usar el Heatmap para Decidir Qué Tests Ejecutar

El heatmap también sirve como guía de planificación:

```
Muchas celdas vacías en "Defense Evasion"
→ Nunca has probado esas técnicas
→ Ir al Browser → filtrar por "Defense Evasion"
→ Seleccionar 5-10 tests para ejecutar esta semana
→ Ver cuántos rojos aparecen en el heatmap
```

Las celdas vacías no son "verde", son desconocido. El objetivo es convertir desconocidos en verde (o al menos saber que son rojos y actuar).

---

## Defense Score por Máquina

También en Analytics, un gráfico de barras horizontales muestra el Defense Score desglosado por endpoint:

```
DESKTOP-SRV01    ████████████████████  84%  🟢
MACBOOK-KENDRA   ████████████████      70%  🟡
LINUX-PROD-01    ████████████████      67%  🟡
SERVER-ARCHIVOS  ████████████          51%  🔴  ← Prioridad

→ SERVER-ARCHIVOS está muy por debajo del promedio
→ Revisar su configuración de seguridad específicamente
```

---

## Cómo Usar Estas Visualizaciones en la Práctica

**Reunión semanal de 15 minutos:**
```
1. ¿El Defense Score global subió o bajó? (2 min)
2. ¿Hay nuevas celdas rojas en el heatmap? (5 min)
3. ¿Qué 1-2 problemas atacamos esta semana? (8 min)
```

**Preparar evidencia para una auditoría:**
```
1. Exportar screenshot del heatmap actual
2. Exportar el trend del Defense Score (últimos 6 meses)
3. Documentar: "Estas son las áreas con gaps y este es el plan de mejora"
```

---

## Puntos Clave

✅ El Heatmap muestra tu cobertura sobre el mapa de técnicas de ataque reales
✅ El Treemap prioriza visualmente: el cuadro rojo más grande = prioridad #1
✅ Celdas vacías = desconocido, no verde, hay que probar esas técnicas
✅ El gráfico por severidad te dice cuán críticos son tus gaps
✅ Defense Score por máquina identifica los endpoints con peor cobertura
✅ Una imagen del treemap comunica la situación de seguridad mejor que cualquier tabla

---

## Próximo Post

**P2-02: "Bundle Tests. Auditar 30 Controles de Seguridad en 5 Minutos"**

Cómo usar los bundles de Achilles para hacer auditorías completas de hardening sin trabajo manual.

---

**Etiquetas:** #ProjectAchilles #MITRE #Heatmap #Visualización #CiberSeguridad #Dashboard #Gaps

---

*Parte 1 de 4 en la serie "Usando Achilles en Profundidad".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
