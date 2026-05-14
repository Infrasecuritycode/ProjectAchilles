# El Defense Score: Qué Significa Ese Número y Qué Hacer Con Él

> **Serie: Empezando con Project Achilles — Parte 5 de 6**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- El Defense Score es un porcentaje: cuántos ataques simulados detectó tu seguridad
- No existe el 100% real — el objetivo es mejorar continuamente desde donde estás
- Un score bajo no es un fracaso: es información que antes no tenías
- Puedes filtrar el score por máquina, tipo de ataque o período de tiempo
- El score te da el lenguaje para hablar de seguridad con tu dirección

---

## La Pregunta que Responde

Antes de Achilles, si alguien te preguntaba "¿cuán seguros estamos?", la respuesta era una opinión.

Con Achilles, la respuesta es un número:

```
Defense Score: 73%

Traducción directa:
"De cada 100 técnicas de ataque que probamos,
 nuestras defensas detectaron 73.
 Las otras 27 pasarían desapercibidas hoy."
```

Es una medición, no una estimación. Y se actualiza cada vez que ejecutas tests.

---

## Cómo Se Calcula (Sin Tecnicismos)

El cálculo es simple:

```
Defense Score = (Tests detectados ÷ Tests totales) × 100

Ejemplo:
Tests ejecutados este mes: 150
Tests donde tu seguridad dijo "¡alerta!": 109
Tests donde nadie notó nada: 41

Defense Score = 109 ÷ 150 × 100 = 72.7%
```

Nota: Los tests con errores técnicos (el agente no pudo ejecutar) no cuentan — solo se miden los que llegaron a ejecutarse correctamente.

---

## ¿Es 73% un Buen Score?

Depende del contexto, pero como orientación general:

```
< 50%     Exposición significativa. Hay gaps fundamentales que atender urgentemente.
50% - 65% Nivel básico. Muchas organizaciones empiezan aquí. Hay trabajo claro por hacer.
65% - 75% Razonable. Cobertura aceptable con oportunidades visibles de mejora.
75% - 85% Bueno. Cobertura sólida. Los gaps que quedan son técnicos y específicos.
> 85%     Excelente. Madurez avanzada. Optimización continua.
```

**Lo más importante no es el número de hoy — es la dirección.**

Un score de 58% que sube a 65% en un mes es mejor que un score de 80% que lleva 6 meses sin moverse.

---

## Tu Primera Medición Siempre Sorprende

Es completamente normal que tu primer Defense Score sea más bajo de lo que esperabas.

¿Por qué?

```
Razones comunes por las que el score es más bajo de lo esperado:

1. Configuración incompleta del EDR
   → Defender activo pero sin todas las políticas aplicadas
   → Exclusiones demasiado amplias que dejaron gaps

2. Inconsistencia entre máquinas
   → Algunas máquinas tienen políticas de seguridad, otras no
   → Equipos más antiguos con configuraciones legacy

3. Técnicas avanzadas que pocas herramientas detectan por defecto
   → Process Injection, Living-off-the-Land requieren configuración específica

4. Nunca se había medido
   → Los gaps siempre estuvieron ahí, simplemente nadie los había visto
```

Un score de 55% en tu primera medición no significa que estás en peligro inmediato. Significa que ahora tienes información concreta para mejorar.

---

## Los Filtros: Dónde Está Realmente el Problema

El score global es útil, pero los filtros revelan dónde está el trabajo:

### Por máquina

```
Analytics → Filtro: Máquina = "SERVER-ARCHIVOS"

Defense Score SERVER-ARCHIVOS: 48%  🔴

Defense Score promedio flota: 73%

→ Este servidor específico está muy por debajo del promedio
→ Probable causa: configuración de seguridad diferente o desactualizada
```

### Por tipo de ataque

```
Analytics → Filtro: Táctica = "Evasión de defensas"

Defense Score en evasión: 38%  🔴

→ Esta categoría es donde más gaps tienes
→ Los ataques avanzados que intentan esconderse de tu seguridad
   tienen alta probabilidad de pasar sin ser detectados
```

### Por severidad

```
Analytics → Filtro: Severidad = "Crítico"

Defense Score en técnicas críticas: 61%

→ De las técnicas más dañinas, detectas solo 6 de cada 10
→ Prioridad: subir este número específico
```

---

## La Tendencia: Tu Historia de Seguridad

Más importante que el número de hoy es ver si sube o baja con el tiempo.

```
Defense Score — últimos 3 meses

80% ┤                                              ╭──────
    │                                    ╭─────────╯
75% ┤                         ╭──────────╯
    │              ╭──────────╯
70% ┤   ╭──────────╯
    │───╯
65% ┤
    └────────────────────────────────────────────────
    Ene           Feb           Mar           Abr

Eventos anotados por el equipo:
→ Feb 10: Activamos Script Block Logging en todos los equipos  +3%
→ Mar 02: Actualizamos políticas del EDR                       +4%
→ Mar 25: Aplicamos parche de configuración a servidores       +3%
```

Cada acción de mejora de seguridad tiene un impacto medible. Eso es lo que presenta tu equipo de IT a dirección: no "hicimos cambios", sino "mejoramos 10 puntos en 3 meses".

---

## Cómo Usar el Score para Priorizar Trabajo

El Defense Score solo es útil si lo conviertes en acción. El flujo es:

```
1. Revisar el score global
   → ¿Subió o bajó esta semana?

2. Identificar los gaps más urgentes
   Analytics → "Top problemas detectados"
   → ¿Cuáles son los rojos con más máquinas afectadas?

3. Enfocarse en 1-3 problemas por semana
   → No intentar resolver todo a la vez
   → Priorizar: crítico + muchas máquinas afectadas = primero

4. Implementar la mejora
   → Configurar la política de seguridad correspondiente
   → Aplicar el cambio a las máquinas afectadas

5. Re-ejecutar el test
   → ¿El score mejoró en esas máquinas?
   → Si sí: siguiente problema
   → Si no: revisar si la mejora se aplicó correctamente
```

---

## Presentar el Score a Dirección

El Defense Score convierte un tema técnico en algo comprensible para cualquiera:

```
Sin Achilles:
Dirección: "¿Estamos protegidos?"
IT: "Sí, tenemos Defender y el firewall activados..."
Dirección: [No puede evaluar la respuesta]

Con Achilles:
Dirección: "¿Estamos protegidos?"
IT: "Nuestro Defense Score actual es 73%.

     Eso significa que de 100 técnicas de ataque probadas,
     detectamos 73. Las 27 restantes son gaps conocidos.

     Este trimestre mejoramos 11 puntos gracias a:
     → Activar logging de PowerShell
     → Actualizar políticas del EDR
     → Parchear 3 servidores con configuración legacy

     Objetivo próximo trimestre: llegar al 80%."

Dirección: [Puede tomar decisiones informadas]
```

---

## Lo Que el Score No Mide

Para ser honestos sobre las limitaciones:

```
El Defense Score mide si detectas las técnicas probadas.
NO mide:

→ Si tus backups funcionan
→ Si tu equipo respondería correctamente a una alerta
→ Vulnerabilidades en tu código o aplicaciones web
→ Ataques de ingeniería social (phishing, llamadas falsas)
→ Ataques físicos

Es una pieza importante del rompecabezas de seguridad,
no el rompecabezas completo.
```

---

## Puntos Clave

✅ Defense Score = % de ataques simulados que tu seguridad detectó
✅ No existe el 100% — el objetivo es mejorar continuamente
✅ Tu primer score bajo es normal — ahora tienes datos para actuar
✅ Los filtros por máquina/tipo de ataque revelan dónde está el problema real
✅ La tendencia importa más que el número de hoy
✅ El score te da el lenguaje para justificar inversiones de seguridad a dirección

---

## Próximo Post

**QS-06: "¿Qué Sigue? Sacar el Máximo Partido a Achilles"**

Ahora que tienes la base, qué features explorar a continuación y cómo construir un programa de validación continua.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #DefenseScore #Tutorial #Métricas #Principiantes

---

*Parte 5 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
