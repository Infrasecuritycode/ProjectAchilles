# Blog Posts — Project Achilles

Serie completa de posts en español sobre cómo usar Project Achilles para validación continua de seguridad. Desde la instalación hasta flujos avanzados de Purple Team y compliance regulatorio.

**Idioma:** Español | **Audiencia:** Blue Team, Red Team, SecOps | **Total:** 17 posts

---

## Quickstart — Empezando con Project Achilles

Para quien instala Achilles por primera vez. Sin conocimientos previos requeridos.

| # | Post | Descripción |
|---|------|-------------|
| QS-00 | [Instala Project Achilles en 30 Minutos](quickstart/QS-00-setup-completo-ES.md) | Guía completa de instalación: cloud ($8/mes) o self-hosted con Docker. Agente en Windows, Linux y macOS. |
| QS-01 | [¿Para Qué Sirve Project Achilles?](quickstart/QS-01-para-que-sirve-ES.md) | El problema que resuelve, qué es el Defense Score y qué NO hace Achilles. |
| QS-02 | [Primer Vistazo al Dashboard](quickstart/QS-02-primer-vistazo-ES.md) | Tour por las 4 secciones principales: Analytics, Endpoints, Browser y Settings. |
| QS-03 | [Conectar tu Primera Máquina](quickstart/QS-03-primera-maquina-ES.md) | Instalar el agente paso a paso en Windows, Linux y macOS. Solución de problemas. |
| QS-04 | [Ejecutar tu Primer Test](quickstart/QS-04-primer-test-ES.md) | Buscar un test, compilarlo, asignarlo a un agente y leer el resultado. |
| QS-05 | [Entender el Defense Score](quickstart/QS-05-defense-score-ES.md) | Qué mide, cómo se calcula, benchmarks por madurez y cómo presentarlo. |
| QS-06 | [¿Qué Sigue?](quickstart/QS-06-que-sigue-ES.md) | Próximos pasos: bundles, alertas, Defender. Hoja de ruta completa de la serie. |

---

## Parte 2 — Usando Achilles en Profundidad

Sacarle más partido a Analytics, tests avanzados y automatización de alertas.

| # | Post | Descripción |
|---|------|-------------|
| P2-01 | [Heatmap y Gaps de Cobertura](parte-2/P2-01-heatmap-gaps-ES.md) | Leer el MITRE ATT&CK heatmap y el Coverage Treemap para identificar qué fases del ataque tienen peor cobertura. |
| P2-02 | [Bundle Tests y Auditorías Automáticas](parte-2/P2-02-bundle-tests-ES.md) | El bundle cyber-hygiene (23 controles), cómo leer los resultados expandidos y remediar los gaps más comunes. |
| P2-03 | [Alertas Automáticas](parte-2/P2-03-alerting-ES.md) | Configurar notificaciones por Slack y email cuando el Defense Score baja. Umbrales y ventanas de mantenimiento. |
| P2-04 | [La Librería de 500+ Tests](parte-2/P2-04-libreria-tests-ES.md) | Navegar las 10 fases del ataque, filtros, ficha de cada test y estrategia de priorización. |

---

## Parte 3 — Integraciones y Automatización

Conectar Achilles con tu stack de seguridad existente y gestionar flotas de agentes.

| # | Post | Descripción |
|---|------|-------------|
| P3-01 | [Conectar Microsoft Defender](parte-3/P3-01-defender-integracion-ES.md) | Azure App Registration, permisos, Defense Score vs Secure Score juntos, correlación de alertas. |
| P3-02 | [Auto-Resolve para el SOC](parte-3/P3-02-auto-resolve-ES.md) | Cerrar automáticamente en Defender las alertas generadas por tests de Achilles. Modos: dry-run → activo. |
| P3-03 | [Gestión de Flota a Escala](parte-3/P3-03-gestion-flota-ES.md) | Vista de flota, actualizaciones remotas, campañas de tests, schedules automáticos y enrolamiento masivo. |

---

## Parte 4 — Pro: Flujos Avanzados

Purple Team estructurado, compilar agentes firmados y evidencia para reguladores.

| # | Post | Descripción |
|---|------|-------------|
| P4-01 | [Purple Team Workflow Completo](parte-4/P4-01-purple-team-workflow-ES.md) | Las 5 fases: intel de amenazas → campaña → análisis de gaps → remediación → retest con evidencia antes/después. |
| P4-02 | [Build & Sign — Compilar tus Propios Agentes](parte-4/P4-02-build-sign-agentes-ES.md) | Compilar desde fuente con Go, firmar para Windows (Authenticode) y macOS (ad-hoc), desplegar con tu certificado. |
| P4-03 | [Compliance: DORA, TIBER-EU e ISO 27001](parte-4/P4-03-compliance-dora-tiber-iso-ES.md) | Qué evidencia pide cada marco, cómo exportarla desde Achilles y cómo estructurar el paquete de auditoría. |

---

## Ruta de Lectura Recomendada

```
¿Acabas de instalar Achilles?
→ Lee el Quickstart completo (QS-00 a QS-06)

¿Quieres profundizar en Analytics?
→ Parte 2: P2-01 → P2-02 → P2-03 → P2-04

¿Tienes Microsoft Defender o más de 10 máquinas?
→ Parte 3: P3-01 → P3-02 → P3-03

¿Haces Purple Team o tienes requisitos de compliance?
→ Parte 4: P4-01 → P4-02 → P4-03
```

---

**Autor:** Kendra Mazara | **Última actualización:** Mayo 2026
