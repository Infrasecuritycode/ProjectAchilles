# ¿Qué Sigue? Sacar el Máximo Partido a Achilles

> **Serie: Empezando con Project Achilles — Parte 6 de 6**

**Tiempo de lectura:** 6 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Con el quickstart completado ya tienes lo esencial: agente instalado, tests ejecutados, Defense Score visible
- El siguiente nivel son los **bundles** — colecciones de tests que validan áreas completas de seguridad
- Configura **alertas** para que Achilles te avise cuando algo falla sin que tengas que revisar el dashboard
- Conecta **Microsoft Defender** si lo usas para cruzar datos y cerrar alertas de tests automáticamente
- Esta serie tiene 9 posts más en profundidad — este post es el mapa de lo que viene

---

## Lo Que Ya Tienes

Si llegaste hasta aquí, completaste el ciclo básico:

```
✅ Achilles instalado y funcionando
✅ Al menos una máquina conectada con el agente
✅ Primer test ejecutado y resultado visible
✅ Defense Score en el dashboard
✅ Sabes qué significa cada número y cómo filtrarlos
```

Con eso ya tienes algo que la mayoría de organizaciones no tiene: **una medición objetiva de cuántos ataques detectarías hoy**.

Ahora la pregunta es: ¿cómo hacerlo más poderoso?

---

## El Siguiente Paso Natural: Los Bundles

Hasta ahora ejecutaste tests uno por uno. Los **bundles** son la versión turbo: colecciones de 20-30 checks relacionados que se ejecutan juntos en una sola operación.

Por ejemplo, el bundle **cyber-hygiene** verifica en 5 minutos:

```
¿Está activa la protección de LSASS?         ✅ / ❌
¿Está habilitado el Script Block Logging?    ✅ / ❌
¿Tiene SMB Signing configurado?              ✅ / ❌
¿Está deshabilitado LLMNR?                   ✅ / ❌
¿Hay AppLocker o WDAC configurado?           ✅ / ❌
... (25 checks más)

Resultado: 17/23 controles correctamente configurados
```

En lugar de pasar horas revisando configuraciones manualmente, tienes el resultado en minutos. Esto lo cubrimos en detalle en el **Post P2-02** de esta serie.

---

## Ponle Automatización: Las Alertas

Revisar el dashboard todos los días no es sostenible. Lo que quieres es que Achilles te avise cuando algo cambia.

Achilles puede enviarte una notificación a Slack o email cuando:

```
→ El Defense Score cae más de X% en una semana
   (señal de que algo cambió: una actualización, una política nueva)

→ Se detecta un gap en una técnica crítica
   (algo que antes funcionaba dejó de funcionar)

→ Una máquina lleva más de 5 minutos sin conexión
   (el agente se cayó o la máquina está apagada)
```

Con las alertas configuradas, el flujo es:

```
Antes: Tú revisas el dashboard → ves si algo cambió
Después: Achilles te avisa → tú actúas solo cuando importa
```

Esto lo cubrimos en el **Post P2-03**.

---

## Si Usas Microsoft Defender: La Integración Vale la Pena

Si tu organización usa Microsoft Defender for Endpoint, hay una integración directa que añade dos cosas muy útiles:

**1. Ver ambos scores juntos:**
```
Defense Score (Achilles):   73%   ← cuántos ataques detectas en práctica
Secure Score (Microsoft):   68%   ← cuán bien configurados estás según MS

Brecha: 5 puntos
→ La configuración "correcta" de Microsoft no garantiza
  que detectes todos los ataques en la práctica
```

**2. Cerrar alertas de tests automáticamente:**
Cuando ejecutas tests, Defender genera alertas reales. Eso es bueno (significa que funciona), pero contamina la bandeja de tu equipo de seguridad con "falsos" positivos de pruebas. Achilles puede cerrarlos automáticamente para que el SOC solo vea amenazas reales.

Esto lo cubrimos en los **Posts P2-04 y P3-02**.

---

## El Mapa Completo de la Serie

Ahora que tienes la base, estos son los posts que siguen:

**Parte 2 — Usando la Herramienta en Profundidad**

```
P2-01  Ver tus gaps visualmente: el Heatmap y el Treemap
       → Entender tu cobertura de un vistazo
       → Comunicar los gaps a dirección con una imagen

P2-02  Bundle tests: 30 controles en una sola ejecución
       → El cyber-hygiene bundle para auditorías de hardening
       → Resultado agrupado: X/Y controles protegidos

P2-03  Alerting: que Achilles te avise cuando algo falla
       → Slack y email en 10 minutos
       → Cuándo y cómo configurar los umbrales

P2-04  La librería completa: 500+ técnicas organizadas
       → Cómo navegar y filtrar el Browser
       → Qué es MITRE ATT&CK y por qué importa
```

**Parte 3 — Integraciones y Automatización**

```
P3-01  Integración con Microsoft Defender
P3-02  Auto-Resolve: reducir el ruido del SOC
P3-03  El agente Go: gestión de flota avanzada
```

**Parte 4 — Pro**

```
P4-01  Purple Team workflow completo
P4-02  Build & Sign — compilar agentes firmados
P4-03  Evidencia para DORA, TIBER-EU e ISO 27001
```

---

## Una Cadencia Realista para Empezar

No necesitas implementar todo a la vez. Esta es una progresión razonable:

```
Semana 1 (ya lo tienes):
→ Agente en 1-2 máquinas
→ Primeros tests manuales
→ Defense Score baseline documentado

Semana 2:
→ Agentes en toda la flota (o al menos los sistemas críticos)
→ Primer bundle ejecutado (cyber-hygiene)
→ Alertas de Slack configuradas

Semana 3-4:
→ Programar tests automáticos semanales
→ Revisar y priorizar los top 5 gaps
→ Implementar las primeras correcciones

Mes 2:
→ Medir mejora vs baseline
→ Presentar Defense Score a dirección
→ Explorar integración con Defender si aplica
```

---

## La Pregunta que Ahora Puedes Responder

Volvamos a donde empezamos en el Post QS-00:

> **Director:** "¿Estamos protegidos?"

Antes de Achilles, la respuesta era una opinión.

Ahora puedes decir:

> "Nuestro Defense Score es 73%. Detectamos 73 de cada 100 técnicas de ataque que probamos. Los 27 gaps principales son estos, y tenemos plan de acción para los 5 más críticos. En 30 días esperamos estar en 80%."

Eso es seguridad medible. Eso es lo que Achilles te da.

---

## Recursos

- 🔗 Documentación oficial: https://docs.projectachilles.io
- 🔗 GitHub (código fuente): https://github.com/projectachilles/ProjectAchilles
- 🔗 Comunidad Discord: [Link en bio]
- 📚 Continúa con el **Post P2-01** sobre visualización de gaps

---

## Puntos Clave

✅ Con el quickstart tienes la base: agente + tests + Defense Score
✅ Los bundles son el siguiente nivel: 20-30 checks en una operación
✅ Las alertas hacen la validación continua sin esfuerzo manual diario
✅ La integración con Defender añade correlación y reduce ruido del SOC
✅ Hay 9 posts más en la serie para profundizar — avanza a tu ritmo

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #Tutorial #Quickstart #Principiantes #DefensaDigital

---

*Parte 6 de 6 en la serie "Empezando con Project Achilles". ¡Gracias por llegar hasta aquí!*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
