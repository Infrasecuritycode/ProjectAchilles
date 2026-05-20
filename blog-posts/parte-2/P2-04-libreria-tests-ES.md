# La Librería de Tests: Navegar 500+ Técnicas de Ataque

> **Serie: Usando Achilles en Profundidad — Parte 4 de 4**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- La librería de Achilles tiene más de 500 simulaciones organizadas por tipo de ataque
- Cada test está vinculado a una técnica real documentada de hackers reales (MITRE ATT&CK)
- Puedes filtrar por: fase del ataque, sistema operativo, nivel de riesgo y tipo de test
- El Browser te dice exactamente qué debería detectar cada test en tu stack de seguridad
- La librería se actualiza automáticamente cuando salen nuevos advisories de ciberseguridad

---

## La Librería: Qué Hay Dentro

Cuando abres el Browser, estás mirando un catálogo de simulaciones de ataque. Cada entrada representa algo que un hacker real podría hacer:

```
BROWSER                                    512 tests disponibles

Búsqueda: [_____________________________]

Filtros activos: ninguno

PowerShell - Encoded Command               ALTO      Windows
PowerShell - AMSI Bypass                   CRÍTICO   Windows
Remote Desktop Protocol Brute Force        ALTO      Windows
Process Injection via DLL                  CRÍTICO   Windows
Registry Run Key Persistence               MEDIO     Windows
SMB Share Enumeration                      BAJO      Windows/Linux
LSASS Memory Access                        CRÍTICO   Windows
DNS Tunneling Exfiltration                 ALTO      Windows/Linux
...
```

Cada línea es un test diferente. 512 formas en que un atacante podría intentar comprometer tus sistemas, con el test correspondiente para verificar si tu defensa lo detectaría.

---

## Cómo Está Organizado: Las Fases del Ataque

Los tests están organizados por la "fase del ataque" en que ocurren. Esto viene del marco MITRE ATT&CK que vimos en el post anterior (P2-01).

Piénsalo como el "guión de un hacker":

```
FASE 1 — ¿Cómo entró?
  Initial Access: phishing, exploits web, credenciales robadas
  → 32 tests disponibles

FASE 2 — ¿Qué ejecutó?
  Execution: PowerShell, scripts, macros de Office
  → 67 tests disponibles (el área con más variantes)

FASE 3 — ¿Cómo se quedó?
  Persistence: registro de Windows, tareas programadas, servicios
  → 48 tests disponibles

FASE 4 — ¿Cómo escaló permisos?
  Privilege Escalation: explotar configuraciones incorrectas
  → 31 tests disponibles

FASE 5 — ¿Cómo se escondió?
  Defense Evasion: técnicas para no ser detectado
  → 78 tests disponibles  ← el área con más gaps en la mayoría de organizaciones

FASE 6 — ¿Cómo robó contraseñas?
  Credential Access: LSASS, keyloggers, hash stealing
  → 38 tests disponibles

FASE 7 — ¿Cómo exploró la red?
  Discovery: escaneo de red, enumerar usuarios y sistemas
  → 52 tests disponibles

FASE 8 — ¿Cómo se movió por la red?
  Lateral Movement: RDP, SMB, WMI
  → 37 tests disponibles

FASE 9 — ¿Qué robó?
  Collection / Exfiltration: archivos, capturas, datos cifrados
  → 64 tests disponibles

FASE 10 — ¿Qué destruyó?
  Impact: ransomware, borrado de datos, sabotaje
  → 30 tests disponibles
```

---

## Los Filtros: Encontrar los Tests que Te Importan

### Por fase del ataque

```
Filtro: Táctica = "Lateral Movement"
→ 37 tests sobre cómo los hackers se mueven entre máquinas
→ Útil si te preocupa que un hacker que entró por un PC
  llegue al servidor de archivos o al controlador de dominio
```

### Por sistema operativo

```
Filtro: Plataforma = "Windows"
→ Tests específicos para Windows
→ La mayoría están aquí porque Windows es el objetivo más común

Filtro: Plataforma = "Linux"
→ Tests específicos para servidores Linux
→ Importante si tienes servidores web, bases de datos, etc.
```

### Por nivel de riesgo

```
Filtro: Severidad = "Crítico"
→ 58 tests de las técnicas más dañinas
→ Si solo puedes probar un subconjunto, empieza por estos
```

### Por tipo: Standalone vs Bundle

```
Filtro: Tipo = "Standalone"
→ Tests individuales, uno por uno
→ Más de 400 disponibles

Filtro: Tipo = "Bundle"
→ Colecciones de tests relacionados
→ 6 bundles disponibles (cyber-hygiene, identity, ransomware...)
```

---

## Ver los Detalles de un Test

Click en cualquier test para ver la ficha completa:

```
┌──────────────────────────────────────────────────────────────┐
│  LSASS Memory Access                                         │
│  T1003.001 · Credential Access · CRÍTICO                     │
├──────────────────────────────────────────────────────────────┤
│  Qué es:                                                     │
│  LSASS (Local Security Authority Subsystem Service) es el   │
│  proceso de Windows que almacena contraseñas en memoria.    │
│  Los hackers intentan leer ese proceso para robar las       │
│  credenciales activas.                                      │
│                                                             │
│  Qué simula este test:                                      │
│  Intenta acceder a la memoria de LSASS con las mismas       │
│  técnicas que usa Mimikatz (la herramienta más usada para   │
│  robo de contraseñas en Windows).                          │
│                                                             │
│  Qué debería detectarlo:                                    │
│  → Windows Defender: Credential theft attempt               │
│  → Si tienes Credential Guard activo: bloqueado en origen  │
│  → EDR enterprise: Memory scanning rule para LSASS         │
│                                                             │
│  Plataforma: Windows · Severidad: CRÍTICO                   │
│  Fuente: CISA Advisory AA23-187A                            │
├──────────────────────────────────────────────────────────────┤
│  [ Compilar ]    [ Asignar a Agente ]                        │
└──────────────────────────────────────────────────────────────┘
```

La ficha te explica en lenguaje claro:
1. Qué es la técnica
2. Qué simula el test
3. Qué de tu stack de seguridad debería detectarlo

---

## La Fuente de los Tests: Ataques Documentados

Cada test en Achilles está basado en ataques reales documentados. Verás referencias como:

```
Fuente: CISA Advisory AA23-187A
→ Advisory del gobierno americano sobre una campaña de hackers real

Fuente: APT29 campaign (SVR)
→ Técnicas usadas por el grupo de hackers vinculado a los servicios
  de inteligencia rusos

Fuente: TIBER-EU threat intelligence
→ Framework europeo de pruebas de resiliencia para banca
```

Esto significa que cuando ejecutas un test de Achilles, estás probando contra técnicas que se usan en ataques reales hoy, no escenarios hipotéticos.

---

## Estrategia: ¿Por Dónde Empezar?

Con 500+ tests, ¿cuáles priorizar?

### Si acabas de instalar Achilles

```
Semana 1: Bundle cyber-hygiene
→ Valida el hardening básico antes de explorar técnicas individuales
→ Te da un mapa rápido de tu estado de configuración
```

### Si ya tienes el cyber-hygiene completo

```
Semana 2-3: Técnicas CRÍTICAS de Execution y Credential Access
→ PowerShell variants (T1059.001)
→ LSASS Access (T1003.001)
→ Son las más usadas en ataques reales y las que más impacto tienen
```

### Si tu sector tiene amenazas específicas

```
Busca por la fuente del advisory de tu sector:
→ Banca/Finanzas: filtrar por "TIBER" o "DORA"
→ Salud: filtrar por "ransomware" + "Impact"
→ Gobierno: filtrar por campañas APT documentadas
```

### Si quieres probar defensa en profundidad

```
Ejecuta el mismo test en todas las máquinas y compara:
¿Por qué DESKTOP-SRV01 lo detecta y SERVER-ARCHIVOS no?
→ Identifica inconsistencias de configuración entre equipos
```

---

## La Librería Se Actualiza Sola

La librería de tests se sincroniza automáticamente desde el repositorio oficial de Achilles:

```
Browser → última actualización: hace 2 horas · 512 tests indexados
```

Cuando CISA publica un nuevo advisory, cuando se documenta una nueva campaña APT, o cuando aparece una nueva variante de malware, los tests correspondientes llegan a tu librería automáticamente.

No necesitas hacer nada. Los tests relevantes para las amenazas de hoy están disponibles cuando los necesites.

---

## Buscador: Encontrar un Test Específico

Si sabes exactamente qué quieres probar, el buscador es la forma más rápida:

```
Buscar: "T1059"        → Todos los tests de PowerShell/Scripting
Buscar: "ransomware"   → Tests relacionados con ransomware
Buscar: "credential"   → Tests de robo de credenciales
Buscar: "rdp"          → Tests de Remote Desktop Protocol
Buscar: "LSASS"        → Tests específicos de LSASS
```

---

## Puntos Clave

✅ 500+ tests organizados por fase del ataque, todos basados en técnicas documentadas reales
✅ Filtros por fase, plataforma, severidad y tipo para encontrar lo que importa
✅ Cada test tiene ficha explicativa: qué es, qué simula, qué debería detectarlo
✅ Los tests vienen de advisories CISA, campañas APT documentadas y frameworks como TIBER
✅ La librería se actualiza automáticamente, siempre tienes los tests de las amenazas actuales
✅ Empieza con bundles, luego técnicas CRÍTICAS, luego explora por tu sector

---

## ¿Qué Sigue en la Serie?

Con la Parte 2 completada tienes:
- Visualizaciones de gaps (Heatmap y Treemap)
- Auditorías automáticas con Bundles
- Alertas automáticas cuando algo falla
- Conocimiento de toda la librería de tests

La **Parte 3** cubre integraciones avanzadas:
- Conectar con Microsoft Defender para correlación automática
- Auto-Resolve: cerrar alertas de tests para no contaminar el SOC
- Gestión avanzada de flota de agentes

---

**Etiquetas:** #ProjectAchilles #MITRE #LibreríaTests #CiberSeguridad #Browser #Tutorial

---

*Parte 4 de 4 en la serie "Usando Achilles en Profundidad".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
