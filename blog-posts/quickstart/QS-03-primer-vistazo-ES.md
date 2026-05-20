# Lo Que Ves Cuando Abres Achilles por Primera Vez

> **Serie: Empezando con Project Achilles — Parte 3 de 6**

**Tiempo de lectura:** 6 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- El dashboard de Achilles tiene 4 secciones principales: Analytics, Endpoints, Browser y Settings
- El número más importante que verás es el **Defense Score**, tu porcentaje de protección
- La primera vez que lo abres no verás datos reales hasta conectar al menos una máquina
- Puedes explorar el dashboard con datos de ejemplo antes de conectar nada
- Este post es un tour completo: qué significa cada cosa y dónde hacer click primero

---

## Antes de Abrir: El Login

Cuando entras a Achilles por primera vez, verás una pantalla de login.

```
┌──────────────────────────────────────┐
│         PROJECT ACHILLES             │
│                                      │
│   Email: [____________________]      │
│   Contraseña: [________________]     │
│                                      │
│   [ Iniciar sesión ]                 │
│   ¿No tienes cuenta? Regístrate      │
└──────────────────────────────────────┘
```

Crea una cuenta con tu email. La autenticación la maneja Clerk (el mismo sistema que usan muchas aplicaciones SaaS modernas), no necesitas configurar nada especial.

---

## La Pantalla Principal: Analytics

Después del login, llegas al **Dashboard de Analytics**. Este es el corazón de Achilles.

```
┌─────────────────────────────────────────────────────────────────┐
│  PROJECT ACHILLES         [Analytics] [Endpoints] [Browser]     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌────────┐ │
│  │ Defense Score│  │    Tests     │  │  Fallos  │  │Máquinas│ │
│  │    73%       │  │   1,847      │  │   312    │  │   23   │ │
│  │  ↑ 4% ↑     │  │  este mes    │  │  (17%)   │  │ online │ │
│  └──────────────┘  └──────────────┘  └──────────┘  └────────┘ │
│                                                                 │
│  [Gráfica de tendencia del Defense Score últimos 30 días]      │
│                                                                 │
│  TOP PROBLEMAS DETECTADOS                                       │
│  🔴 Process Injection        23 máquinas    CRÍTICO            │
│  🔴 Cuentas con acceso raro  23 máquinas    CRÍTICO            │
│  🟡 PowerShell sospechoso    11 máquinas    ALTO               │
└─────────────────────────────────────────────────────────────────┘
```

**Si es tu primera vez y no tienes datos todavía**, Achilles puede mostrarte datos de ejemplo para que explores. Los encontrarás en Settings → Analytics → "Cargar datos de muestra".

---

## Los 4 Números del Dashboard

### 1. Defense Score — El número más importante

```
Defense Score: 73%
```

Este porcentaje te dice: de todas las técnicas de ataque que Achilles probó, ¿cuántas detectó tu seguridad?

- **73% significa:** por cada 100 intentos de ataque simulados, tu defensa detectó 73.
- **No existe el 100%** en la práctica, siempre habrá algo que mejorar.
- **Un buen objetivo inicial:** subir del baseline que tengas hoy.

La flecha (↑ 4%) te dice si estás mejorando o empeorando respecto a la semana anterior.

### 2. Tests ejecutados — El volumen de validación

```
Tests: 1,847
```

Cuántas simulaciones de ataque ha ejecutado Achilles en el período seleccionado. Más tests = más confianza en tu Defense Score.

### 3. Fallos — Los gaps que encontró

```
Fallos: 312 (17%)
```

Cuántas simulaciones pasaron sin ser detectadas. Cada "fallo" es una técnica de ataque que hoy no detectarías. Son tu lista de tareas de seguridad.

### 4. Máquinas online — Tu flota activa

```
Máquinas: 23 online
```

Cuántos equipos tienen el agente instalado y activo. Más máquinas = vista más completa de tu organización.

---

## El Menú Principal: Cuatro Secciones

### Analytics — Tus métricas

```
[Analytics]
```

Donde pasas la mayor parte del tiempo. Aquí ves el Defense Score, la tendencia histórica, qué ataques no estás detectando y en qué máquinas.

### Endpoints — Tus máquinas

```
[Endpoints]
```

La lista de todas las máquinas con el agente instalado. Desde aquí puedes:
- Ver qué máquinas están online u offline
- Asignar pruebas a máquinas específicas
- Ver el historial de pruebas de cada máquina

### Browser — El catálogo de pruebas

```
[Browser]
```

La biblioteca de simulaciones disponibles. Tiene más de 500 técnicas de ataque, organizadas por tipo y nivel de riesgo. Desde aquí seleccionas qué pruebas ejecutar.

### Settings — La configuración

```
[Settings]
```

Donde configuras todo: conectar Elasticsearch (la base de datos donde Achilles guarda resultados), integrar Microsoft Defender, configurar notificaciones de Slack o email.

---

## Lo Que Verás en Tu Primera Semana

**Día 1: Sin datos**
El dashboard está vacío. Normal. Necesitas instalar el agente en al menos una máquina primero (lo hacemos en QS-03).

**Día 2-3: Primeros resultados**
Después de instalar el agente y ejecutar los primeros tests, el Defense Score aparece. Probablemente entre 50-70%, eso es normal para una primera medición.

**Día 7+: Tendencia visible**
Empiezas a ver si la línea sube o baja. Cada cambio de configuración que hagas en tu seguridad se refleja en el score.

---

## La Vista de Tendencia: Tu Historia de Seguridad

Debajo de los 4 números principales hay una gráfica de línea:

```
Defense Score — últimas 4 semanas

85% ┤
    │                                    ╭────
80% ┤                               ╭───╯
    │                    ╭───────────╯
75% ┤         ╭──────────╯
    │─────────╯
70% ┤
    └───────────────────────────────────────
    Sem 1      Sem 2      Sem 3      Sem 4
```

Esta gráfica es tu historia de seguridad. Cada vez que tu equipo hace mejoras (actualizar configuraciones, añadir reglas, aplicar parches), deberías ver la línea subir. Si baja, algo cambió para peor.

---

## La Lista de Problemas: Tu Prioridad de Hoy

En la parte inferior del dashboard verás algo como esto:

```
TOP PROBLEMAS DETECTADOS (ordenados por impacto)

🔴 Process Injection        23/23 máquinas   CRÍTICO
🔴 Cuentas con acceso raro  23/23 máquinas   CRÍTICO
🟡 PowerShell sospechoso    11/23 máquinas   ALTO
🟡 Movimiento lateral       8/23 máquinas    ALTO
🟢 Escaneo de red           2/23 máquinas    BAJO
```

**🔴 Rojo** = Achilles lo probó y tu seguridad no lo detectó en ninguna máquina.
**🟡 Amarillo** = Lo detectaste en algunas máquinas pero no en todas.
**🟢 Verde** = Tu seguridad lo está manejando bien.

Esta lista es directamente tu backlog: empieza por los rojos.

---

## El Filtro de Fechas: Ver Períodos Específicos

En la parte superior del dashboard hay un selector de fechas:

```
[Últimos 7 días ▼]  [Todas las máquinas ▼]
```

Puedes filtrar por:
- Período: últimos 7 días, 30 días, 90 días, o rango personalizado
- Máquina específica: ver solo los resultados de un servidor o computadora

Útil cuando quieres saber "¿qué pasó después de que IT actualizó las políticas el martes pasado?"

---

## Puntos Clave

✅ El Defense Score es el número más importante, tu porcentaje de protección
✅ Analytics, Endpoints, Browser y Settings son las 4 secciones principales
✅ Los "Fallos" son tu lista de problemas a resolver, empieza por los rojos
✅ La gráfica de tendencia muestra si estás mejorando o empeorando
✅ Sin agente instalado no hay datos, eso lo resolvemos en el próximo post

---

## Próximo Post

**QS-04: "Ejecutar tu Primer Test"**

Paso a paso: cómo instalar el agente en una máquina Windows, Linux o macOS y verla aparecer en el dashboard.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #Tutorial #Dashboard #Principiantes

---

*Parte 3 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
