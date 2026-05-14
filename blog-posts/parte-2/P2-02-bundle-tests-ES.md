# Bundle Tests: Auditar 30 Controles de Seguridad en 5 Minutos

> **Serie: Usando Achilles en Profundidad — Parte 2 de 4**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Un **bundle** es una colección de tests relacionados que se ejecutan juntos en una sola operación
- El resultado aparece agrupado: "17/23 controles protegidos" — expandible para ver cada uno
- El bundle **cyber-hygiene** valida el hardening básico de Windows en 5 minutos
- Ideal para auditorías periódicas y para responder "¿estamos bien configurados según mejores prácticas?"
- No necesitas conocimiento técnico previo para interpretar los resultados

---

## El Problema que Resuelven los Bundles

Imagina que quieres responder esta pregunta:

> "¿Están bien configuradas nuestras máquinas Windows según las mejores prácticas de seguridad?"

Para responder eso con tests individuales necesitarías:
- Asignar 23 tests uno por uno
- Esperar a que ejecuten en secuencia
- Abrir 23 resultados separados
- Correlacionarlos manualmente

Con un bundle:
- Asignas 1 cosa
- Esperas 5 minutos
- Ves el resultado: **17/23 controles protegidos**
- Expandes para ver exactamente cuáles fallaron

La diferencia entre 45 minutos de trabajo manual y 5 minutos automáticos.

---

## ¿Qué Contiene un Bundle?

Un bundle es una lista de "preguntas de seguridad" relacionadas. Cada pregunta es un control.

Por ejemplo, el bundle **cyber-hygiene** hace preguntas como:

```
¿Está activa la protección de LSASS?            → CH-DEF-001
¿Tiene configurado PPL para LSASS?              → CH-DEF-002
¿Están activas las reglas de reducción          → CH-DEF-003
  de superficie de ataque (ASR)?
¿Está habilitado el modo restringido            → CH-DEF-004
  de PowerShell?
¿Está activo el Script Block Logging?           → CH-IEP-001
¿Tiene SMB Signing habilitado?                  → CH-NET-001
¿Está deshabilitado LLMNR?                      → CH-NET-002
¿Hay AppLocker o WDAC configurado?              → CH-APP-001
... (23 controles en total)
```

Cada control es una verificación técnica de configuración.

---

## Ejecutar Tu Primer Bundle

```
Browser → pestaña "Bundles" → cyber-hygiene

┌────────────────────────────────────────────────────────────┐
│  cyber-hygiene                                             │
│  Windows Cyber Hygiene Baseline · 23 controles             │
├────────────────────────────────────────────────────────────┤
│  Verifica el hardening básico de Windows según CIS         │
│  Benchmark Level 1. Ideal como auditoría inicial.          │
│                                                            │
│  Tiempo estimado: ~5 minutos                               │
│  Plataforma: Windows                                       │
└────────────────────────────────────────────────────────────┘

[ Compilar Bundle ]  [ Asignar a Máquina ]
```

Una vez compilado (tarda ~2 minutos la primera vez):

```
[ Asignar a Máquina ]

Selecciona: ● DESKTOP-SRV01
¿Cuándo?    ● Ahora mismo

[ Asignar ]
→ Bundle asignado. Resultados en ~5 minutos.
```

---

## Ver los Resultados: La Vista Agrupada

Cuando el bundle termina, ve a **Analytics → Executions Table**:

```
EXECUTIONS TABLE

▶  cyber-hygiene bundle      [17/23 🛡️ Protegidos]    DESKTOP-SRV01   hace 8 min
—  T1059.001 (standalone)    ✅ Protegido              DESKTOP-SRV01   hace 2 h
```

El bundle aparece como una sola fila con el resumen. Click en ▶ para expandir:

```
▼  cyber-hygiene bundle      [17/23 🛡️ Protegidos]    DESKTOP-SRV01

   ✅  CH-DEF-001  Protección LSASS activa           CRÍTICO
   ❌  CH-DEF-002  PPL para LSASS no configurado     CRÍTICO    ← gap
   ✅  CH-DEF-003  ASR Rules activas                 ALTO
   ✅  CH-DEF-004  PowerShell modo restringido        ALTO
   ✅  CH-IEP-001  Script Block Logging activo        MEDIO
   ✅  CH-IEP-002  Module Logging activo              MEDIO
   ✅  CH-IEP-003  Transcription Logging activo       BAJO
   ❌  CH-NET-001  SMB Signing deshabilitado          CRÍTICO    ← gap
   ✅  CH-NET-002  LLMNR deshabilitado                ALTO
   ❌  CH-NET-003  NetBIOS activo                     ALTO       ← gap
   ❌  CH-APP-001  Sin AppLocker ni WDAC              CRÍTICO    ← gap
   ... (12 más, 13 protegidos)
```

**17 de 23 está bien.** Los 6 gaps son exactamente lo que necesitas corregir.

---

## ¿Qué Hacer con los Gaps del Bundle?

Cada control fallido es una tarea concreta. Aquí los más comunes y cómo resolverlos:

### CH-DEF-002 — PPL para LSASS no configurado

```
¿Qué significa?
LSASS es el proceso de Windows que guarda contraseñas en memoria.
PPL (Protected Process Light) lo protege de que otros programas
lo lean. Sin PPL, herramientas como Mimikatz pueden robar contraseñas.

Cómo corregirlo (en Windows 10/11 y Server 2016+):
Abrir regedit → ir a:
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
→ Crear DWORD: RunAsPPL = 1
→ Reiniciar la máquina

O via GPO si tienes Active Directory:
Computer Config → Windows Settings → Security Settings →
Local Policies → Security Options → 
"LSASS Protection (RunAsPPL)" → Enabled
```

### CH-NET-001 — SMB Signing deshabilitado

```
¿Qué significa?
SMB Signing previene ataques de tipo "relay" donde un atacante
intercepta comunicaciones entre máquinas Windows. Sin firma,
un atacante en la red puede impersonar servidores.

Cómo corregirlo via PowerShell:
Set-SmbServerConfiguration -RequireSecuritySignature $true -Force
Set-SmbClientConfiguration -RequireSecuritySignature $true -Force
```

### CH-APP-001 — Sin AppLocker ni WDAC

```
¿Qué significa?
AppLocker y WDAC (Windows Defender Application Control) controlan
qué programas pueden ejecutarse. Sin ellos, cualquier ejecutable
puede correr — incluido malware.

Cómo empezar:
→ AppLocker es más fácil de configurar para empezar
→ Panel de Control → Herramientas Administrativas →
   Directiva de Seguridad Local → Application Control Policies
→ Empieza en modo "Audit" para ver qué bloquearía antes de activarlo
```

---

## Los Bundles Disponibles

```
cyber-hygiene
→ Hardening básico de Windows (23 controles)
→ Ideal para: primera auditoría, benchmark CIS

identity-endpoint
→ Protección de credenciales y cuentas (18 controles)
→ Ideal para: organizaciones con Active Directory

ransomware-baseline
→ Controles antiransomware esenciales (15 controles)
→ Ideal para: evaluar resiliencia frente a ransomware

apt29-campaign
→ Simula técnicas de un grupo de hackers reales (APT29)
→ 19 etapas del ataque completo
→ Más avanzado — para después de tener buena cobertura básica
```

---

## Programar Bundles Automáticamente

El verdadero valor de los bundles es ejecutarlos de forma recurrente:

```
Endpoints → Schedules → [+ Nueva Programación]

Bundle:   cyber-hygiene
Máquinas: Todas las Windows
Horario:  Primer lunes de cada mes, 02:00 AM

[ Guardar ]
```

Con esto, tienes una auditoría de hardening automática mensual. Si alguien deshabilita una configuración por error o una actualización la sobreescribe, el bundle lo detecta sin que tengas que hacer nada.

---

## Bundles vs Tests Individuales: ¿Cuándo Usar Cada Uno?

```
Usa tests individuales cuando:
→ Quieres validar una técnica específica que te preocupa
→ Después de un cambio de configuración puntual
→ Quieres explorar el catálogo a tu ritmo

Usa bundles cuando:
→ Quieres hacer una auditoría completa de un área
→ Quieres programar validación recurrente automática
→ Necesitas un informe de "estado general" del hardening
→ Estás preparando evidencia para una auditoría externa
```

---

## El Resultado Como Evidencia

El resultado del bundle es ideal para documentar:

```
Auditoría cyber-hygiene — DESKTOP-SRV01
Fecha: 2026-05-14 · Ejecutado por: Achilles v1.4.2

Resultado: 17/23 controles protegidos (73.9%)

Controles fallidos (acción requerida):
  CH-DEF-002 CRÍTICO → Plan: habilitar PPL via GPO (Owner: IT, Fecha: 2026-05-21)
  CH-NET-001 CRÍTICO → Plan: activar SMB Signing (Owner: IT, Fecha: 2026-05-21)
  CH-APP-001 CRÍTICO → Plan: evaluar AppLocker (Owner: Security Eng, Fecha: 2026-06-01)
  CH-NET-003 ALTO    → Plan: deshabilitar NetBIOS (Owner: IT, Fecha: 2026-05-28)

Re-auditoría programada: 2026-06-14
```

Esto es exactamente lo que un auditor externo quiere ver: evidencia técnica + plan de acción + seguimiento.

---

## Puntos Clave

✅ Un bundle = múltiples controles en 1 operación → resultado "X/Y protegidos"
✅ El resultado es expandible: ves exactamente qué falló y qué pasó
✅ Empieza con cyber-hygiene para el hardening básico de Windows
✅ Programa bundles mensuales para auditoría automática sin trabajo manual
✅ Los gaps del bundle son tareas concretas con pasos de resolución claros
✅ El resultado del bundle es evidencia directamente usable en auditorías

---

## Próximo Post

**P2-03: "Alerting — Que Achilles Te Avise Cuando Algo Falla"**

Cómo configurar notificaciones de Slack o email para no tener que revisar el dashboard manualmente.

---

**Etiquetas:** #ProjectAchilles #BundleTests #Hardening #CIS #CiberSeguridad #Auditoría #Windows

---

*Parte 2 de 4 en la serie "Usando Achilles en Profundidad".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
