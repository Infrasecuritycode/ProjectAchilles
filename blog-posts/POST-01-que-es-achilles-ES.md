# Project Achilles: El BAS Open-Source que Todo Equipo de Seguridad Necesita

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 1 de 15**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- **Project Achilles** es una plataforma open-source de **Continuous Security Testing (CST)**
- Despliega un agente Go ligero en tus endpoints para ejecutar tests reales contra APT
- Mide tu cobertura de detección en tiempo real con Elasticsearch
- Es el equivalente open-source de AttackIQ, SafeBreach y Picus — pero gratuito
- Licencia Apache 2.0: self-host en tu infraestructura o usa el PaaS por $8/mes

---

## El Problema: $4M en Seguridad y Cero Evidencia de que Funciona

Imagina la conversación con tu CISO después de un incidente:

```
Auditor: ¿Cómo saben que su EDR detectaría un ataque APT29?
CISO:    Tenemos la mejor solución del mercado.
Auditor: ¿Tienen evidencia?
CISO:    ...tenemos el contrato de mantenimiento.
```

**La realidad de la mayoría de organizaciones:**

```
Gasto en seguridad:
→ EDR enterprise:          $800K/año
→ SIEM + Elasticsearch:    $500K/año  
→ Firewall next-gen:       $300K/año
→ Pentest anual:           $150K/año
→ Formación Blue Team:     $200K/año
─────────────────────────────────────
Total:                   ~$2M/año

Preguntas que nadie puede responder:
→ ¿Cuántas técnicas MITRE ATT&CK detectamos HOY?
→ ¿Qué gaps de cobertura tiene nuestro SIEM?
→ ¿Cuánto tardamos en detectar movimiento lateral?
→ ¿Nuestra configuración de EDR es correcta?
```

El problema no es la inversión. Es que **no existe un ciclo de validación continua**.

---

## La Solución: Continuous Security Testing

**Continuous Security Testing (CST)** cierra ese loop ejecutando tests reales contra tus defensas de forma continua — no anualmente.

```
┌─────────────────────────────────────────────────────────────┐
│              CICLO DE VALIDACIÓN CONTINUA                   │
│                                                             │
│   Intel de Amenazas → Tests → Ejecución → Medición         │
│         ↑                                        │          │
│         └──────── Mejora Continua ───────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

**La diferencia con un pentest tradicional:**

| | Pentest Tradicional | Continuous Security Testing |
|---|---|---|
| Frecuencia | Anual | Continuo (24/7) |
| Cobertura | 20-30 técnicas | 500+ técnicas MITRE |
| Tiempo de respuesta | Meses | Minutos |
| Coste | $50K-150K | Gratis (open-source) |
| Evidencia | Informe PDF | Métricas en tiempo real |

---

## ¿Qué es Project Achilles?

> Project Achilles es la plataforma open-source de CST para cualquier organización, de cualquier tamaño. Valida cada control contra tradecraft APT real — mapeado a MITRE ATT&CK, DORA, TIBER-EU, ISO 27001 y CIS.

**En una frase:** Achilles despliega un agente en tus máquinas, ejecuta ataques simulados, y mide cuántos detecta tu stack defensivo.

### Tres módulos principales:

```
┌──────────────────────────────────────────────────────────────┐
│                     PROJECT ACHILLES                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │   BROWSER    │  │   ANALYTICS  │  │     AGENT        │   │
│  │              │  │              │  │                  │   │
│  │ Librería de  │  │ 30+ endpoints│  │ Go binary        │   │
│  │ 500+ tests   │  │ Elasticsearch│  │ Win/Lin/Mac      │   │
│  │ MITRE mapped │  │ Heatmaps     │  │ Token enrol      │   │
│  │ Build & sign │  │ Defense Score│  │ Heartbeat        │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## Cómo Funciona: El Pipeline Completo

### Etapa 1: Ingesta de Inteligencia de Amenazas

```bash
# CISA publica advisory sobre nueva campaña APT
# Achilles ya tiene los tests mapeados en su librería

achilles browser list --tactic "lateral-movement"
# T1021.001  Remote Desktop Protocol     [CRITICAL]
# T1021.002  SMB/Windows Admin Shares    [HIGH]  
# T1021.006  Windows Remote Management  [HIGH]
```

### Etapa 2: Seleccionar y Construir Tests

```bash
# Build del bundle de tests firmado para Windows
achilles tests build --bundle apt29-campaign \
  --platform windows-amd64 \
  --sign
# ✓ Compilando 23 binarios Go...
# ✓ Firmando con certificado Authenticode...
# ✓ Bundle listo: apt29-campaign-win-amd64.zip
```

### Etapa 3: Desplegar y Ejecutar

```bash
# Asignar tarea a endpoint o flota entera
achilles tasks create \
  --agent "DESKTOP-SRV01" \
  --test "T1021.001-rdp-brute" \
  --scheduled "2026-05-15T02:00:00Z"
# ✓ Tarea creada → se ejecutará en horario fuera de producción
```

### Etapa 4: Medir la Cobertura

```
Dashboard Analytics:

Defense Score:  73%  ↑ 4% vs semana pasada
Técnicas cubiertas:   312/427
Gaps críticos:        15 técnicas sin cobertura

Top 5 gaps detectados:
1. T1055 - Process Injection       ❌ No detectado
2. T1059.001 - PowerShell          ⚠️  Detección parcial
3. T1021.001 - RDP Brute           ✅ Detectado
4. T1078 - Valid Accounts           ❌ No detectado
5. T1486 - Data Encrypted (Ransom) ✅ Detectado
```

---

## El Agente Go: Ligero, Seguro, Multiplataforma

El corazón de Achilles es un agente Go compilado estáticamente que corre en cualquier endpoint.

```
Plataformas soportadas:
✅ Windows x64
✅ Linux x64  
✅ macOS x64
✅ macOS ARM64 (Apple Silicon)

Características de seguridad:
✅ Firma de código (Authenticode en Windows, ad-hoc en macOS)
✅ Comunicación AES-256-GCM
✅ Autenticación Ed25519
✅ Self-update automático
✅ Sin dependencias externas (binario estático)

Tamaño del binario: ~8 MB
```

---

## Comparativa: Achilles vs Soluciones Enterprise

Los competidores directos de Achilles son:

| Plataforma | Precio/año | Open Source | Self-host | DORA/TIBER |
|---|---|---|---|---|
| **Achilles** | **Gratis** | **✅ Apache 2.0** | **✅ Completo** | **✅** |
| AttackIQ | $200K+ | ❌ | ❌ | Parcial |
| SafeBreach | $150K+ | ❌ | ❌ | Parcial |
| Picus | $180K+ | ❌ | ❌ | ❌ |

> "BAS empresarial sin la factura empresarial."

---

## Frameworks de Cumplimiento Soportados

Achilles mapea cada test a los frameworks regulatorios más relevantes:

```
MITRE ATT&CK v15    → 500+ técnicas cubiertas
DORA (Art. 25)      → Evidencia para TLPT y pruebas de resiliencia
TIBER-EU            → Threat-Led Penetration Testing
ISO 27001:2022      → Controles A.8.8, A.8.29, A.5.37
CIS Controls v8     → Grupos de implementación IG1-IG3
```

Si trabajas en banca, seguros, o infraestructura crítica en Europa, esta alineación con DORA y TIBER-EU es especialmente relevante.

---

## Purple Team: La Filosofía Detrás del Diseño

Achilles es una herramienta de **purple team** — diseñada para que Blue Team y Red Team trabajen juntos:

```
RED TEAM usa Achilles para:               BLUE TEAM usa Achilles para:
→ Simular campañas APT reales             → Medir cobertura de detección
→ Identificar gaps de detección           → Priorizar reglas SIEM
→ Validar técnicas de evasión             → Justificar inversiones de seguridad
→ Generar evidencia de eficacia           → Preparar auditorías DORA/ISO
```

No es una herramienta de ataque. Es una herramienta de **medición**.

---

## Casos de Uso Reales

### SOC de Banca Regional
```
Antes de Achilles:
- 0 métricas de cobertura MITRE
- Pentest anual = $120K
- Tiempo para detectar gaps: semanas

Después (3 meses con Achilles):
- Defense Score: 68% → 81%
- 47 reglas SIEM nuevas creadas
- Evidencia DORA lista para auditoría
```

### Equipo IT de Gobierno
```
Requisito: Cumplir ENS (Esquema Nacional de Seguridad)
Solución: Achilles self-hosted en infraestructura aislada
Resultado: Mapa de cobertura completo antes de auditoría
```

### MSSP / Proveedor de Servicios
```
Modelo: Multi-tenant con Achilles
Valor: Informe mensual de Defense Score para cada cliente
Diferenciación: Métrica objetiva vs "recomendaciones"
```

---

## Qué Necesitas para Empezar

### Requisitos mínimos:
- Docker & Docker Compose
- Un endpoint Windows, Linux o macOS para instalar el agente
- Elasticsearch (incluido en Docker Compose)

### Tiempo de setup: **5 minutos**

```bash
git clone https://github.com/projectachilles/achilles
cd achilles
docker compose up -d
# ✓ Backend iniciado en :3000
# ✓ Frontend iniciado en :5173  
# ✓ Elasticsearch iniciado en :9200
```

---

## Hoja de Ruta de la Serie

Esta es la **Parte 1 de 15** en la serie de Validación Continua de Seguridad:

**Módulo 1: Fundamentos (Posts 1-3)**
- ✅ POST 01: ¿Qué es Project Achilles? (estás aquí)
- 📅 POST 02: La arquitectura: 3 módulos, 1 objetivo
- 📅 POST 03: Instalación en 5 minutos con Docker Compose

**Módulo 2: El Agente Go (Posts 4-5)**
- 📅 POST 04: Enrolamiento de endpoints y gestión de flota
- 📅 POST 05: La librería de tests — 500+ técnicas MITRE

**Módulo 3: Ejecutar y Medir (Posts 6-8)**
- 📅 POST 06: Tu primer test de seguridad
- 📅 POST 07: Defense Score — tu primera métrica real
- 📅 POST 08: Heatmap MITRE y Treemap de cobertura

**Módulo 4: Capacidades Avanzadas (Posts 9-11)**
- 📅 POST 09: Bundle Tests — validación de controles completos
- 📅 POST 10: Integración con Microsoft Defender
- 📅 POST 11: Auto-Resolve — automatiza el cierre de alertas

**Módulo 5: Producción (Posts 12-15)**
- 📅 POST 12: Build & Sign — compilando agentes firmados
- 📅 POST 13: Alerting — notificaciones cuando tus defensas fallan
- 📅 POST 14: Purple Team workflow completo
- 📅 POST 15: Compliance — evidencia lista para DORA y TIBER-EU

---

## Puntos Clave

✅ Achilles = BAS (Breach & Attack Simulation) open-source y gratuito
✅ Agente Go ligero para Windows, Linux y macOS
✅ Defense Score en tiempo real con Elasticsearch
✅ Mapeado a MITRE ATT&CK, DORA, TIBER-EU, ISO 27001
✅ Self-host completo o PaaS por $8/mes
✅ Apache 2.0 — sin licencias, sin procurement

---

## Discusión

**Preguntas para ti:**

1. ¿Tu organización hace algún tipo de validación continua hoy?
2. ¿Cuántas técnicas MITRE ATT&CK crees que cubre tu EDR?
3. ¿Tienes requisitos DORA o TIBER-EU que cumplir?

---

## Recursos

- 🔗 GitHub: https://github.com/projectachilles/achilles
- 🔗 Sitio web: https://projectachilles.io
- 🔗 Discord: [Link en bio]

---

## Próximo Post

**POST 02: "La Arquitectura de Project Achilles: 3 Módulos, 1 Objetivo"**

Desmontaremos el stack completo: el agente Go, el backend Express, Elasticsearch y cómo se conectan entre sí.

---

**Etiquetas:** #ProjectAchilles #ContinuousSecurityTesting #BAS #MITRE #PurpleTeam #OpenSource #CiberSeguridad #DORA #TIBER #SOC #BlueTeam #RedTeam

---

*Parte 1 de 15 en la serie "Validación Continua de Seguridad con Project Achilles". Sígueme para actualizaciones semanales.*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
