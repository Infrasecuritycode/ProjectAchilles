# ¿Para Qué Sirve Project Achilles? La Respuesta Honesta

> **Serie: Empezando con Project Achilles — Parte 1 de 6**

**Tiempo de lectura:** 5 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Achilles responde una pregunta que la mayoría de organizaciones no puede responder: **¿Tus defensas realmente funcionan?**
- No es un antivirus, no es un SIEM, no reemplaza tu seguridad actual — la **verifica**
- Lo instalas en 5 minutos, despliega un agente en tus máquinas, y te dice qué ataques detectarías y cuáles no
- Es gratuito y open-source (Apache 2.0)
- Si tienes un equipo de IT o seguridad de cualquier tamaño, esto es para ti

---

## La Conversación que Todos Hemos Tenido

¿Te suena familiar esta situación?

> **Director:** "¿Estamos protegidos contra ransomware?"
>
> **IT/Seguridad:** "Sí, tenemos Defender, tenemos el SIEM, tenemos el firewall..."
>
> **Director:** "Pero... ¿cómo lo sabemos?"
>
> **IT/Seguridad:** "..."

Tienes las herramientas. Pagaste por ellas. Pero nadie puede responder con certeza si esas herramientas realmente detectarían un ataque real.

Eso no es culpa tuya. Hasta hace poco, la única forma de saberlo era contratar un equipo externo de pentesters una vez al año por $50,000-$150,000.

**Project Achilles cambia eso.**

---

## Qué Hace Achilles en Términos Simples

Achilles instala un pequeño programa en tus máquinas (el "agente") que simula técnicas que usarían hackers reales. Después de cada simulación, Achilles mide si tu seguridad lo detectó o no.

El resultado es un número: tu **Defense Score**.

```
Defense Score: 73%

Esto significa:
→ De 100 técnicas de ataque simuladas,
  tu stack de seguridad detectó 73.
→ 27 pasarían desapercibidas en un ataque real.
```

No es una opinión. No es una estimación. Es una medición.

---

## ¿Qué NO Es Achilles?

Es importante aclarar lo que Achilles no hace:

**No es un antivirus.**
Achilles no protege tus máquinas. Mide si tu protección actual funciona.

**No es un escáner de vulnerabilidades.**
Herramientas como Nessus buscan agujeros en tu software. Achilles prueba si tu defensa detectaría que alguien los explota.

**No reemplaza nada de lo que ya tienes.**
Funciona sobre tu seguridad actual. Si tienes Defender, Achilles prueba si Defender detectaría un ataque. Si tienes un SIEM, Achilles genera alertas en él para ver si alguien las vería.

**No hace daño.**
Las simulaciones son controladas y seguras. El agente no tiene acceso a tus datos, no roba credenciales, no se mueve por tu red. Es como un simulacro de incendio — real enough para medir la respuesta, seguro para el edificio.

---

## Un Ejemplo Real

Imagina que tienes Windows Defender activo en todas tus máquinas.

Sin Achilles, asumes que Defender detecta los ataques comunes. Con Achilles, lo pruebas:

```
Simulación ejecutada: PowerShell con comando oculto
(técnica favorita de muchos hackers reales)

Resultado en DESKTOP-VENTAS-01:
→ ✅ Defender detectó el intento — PROTEGIDO

Resultado en SERVER-ARCHIVOS:
→ ❌ Nadie detectó nada — GAP ENCONTRADO
```

¿Por qué el servidor no lo detectó? Quizás Defender no está bien configurado ahí. Quizás tiene una excepción vieja. Quizás nunca se aplicó la política correcta.

Ahora lo sabes. Antes, no lo sabías.

---

## ¿Para Quién Es Achilles?

Achilles es útil si te identificas con alguna de estas situaciones:

**"Somos una empresa mediana y no tenemos presupuesto para un pentest anual."**
Achilles es gratis. Lo instalas tú mismo y obtienes métricas continuas.

**"Tenemos auditorías de seguridad y necesitamos evidencia real."**
Achilles genera registros técnicos de cada test ejecutado — documentación concreta para auditores.

**"Nuestro equipo de IT gestiona la seguridad pero no son expertos en ciberataques."**
No necesitas saber cómo funciona un ataque para usar Achilles. La herramienta lo hace por ti.

**"Tenemos un equipo de seguridad y queremos medir mejor nuestros controles."**
Achilles es el benchmarking continuo que falta en la mayoría de programas de seguridad.

---

## Cómo Se Ve en la Práctica

Cuando abres Achilles, ves algo así:

```
┌────────────────────────────────────────────────────┐
│  Defense Score    Tests ejecutados   Máquinas      │
│     73%               1,847            23          │
│   ↑ 4% esta semana                   online        │
├────────────────────────────────────────────────────┤
│  Técnicas que no estás detectando:                 │
│  🔴 Process Injection        23 máquinas afectadas │
│  🔴 Uso de cuentas válidas   23 máquinas afectadas │
│  🟡 PowerShell oculto        11 de 23 máquinas     │
└────────────────────────────────────────────────────┘
```

Un número claro. Una lista de lo que falla. Sin ambigüedades.

---

## Lo Que Necesitas para Empezar

Para empezar a usar Achilles solo necesitas:

- Una computadora o servidor donde instalar Achilles (o usar su versión en la nube por $8/mes)
- Al menos una máquina donde instalar el agente (puede ser la misma)
- 30 minutos la primera vez

No necesitas:
- Saber programar
- Conocer MITRE ATT&CK (aunque lo aprenderás por el camino)
- Un equipo de seguridad dedicado

---

## Puntos Clave

✅ Achilles mide si tu seguridad actual detectaría ataques reales
✅ El resultado es un número: Defense Score — claro, objetivo y comparable
✅ No reemplaza tu seguridad, la verifica
✅ Las simulaciones son seguras y controladas
✅ Gratuito y open-source

---

## Próximo Post

**QS-02: "Lo Que Ves Cuando Abres Achilles por Primera Vez"**

Un tour completo del dashboard: qué significa cada número y dónde ir primero.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #DefensaDigital #OpenSource #Tutorial #Principiantes

---

*Parte 1 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
