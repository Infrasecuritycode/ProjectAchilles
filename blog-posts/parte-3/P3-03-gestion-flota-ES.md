# Gestión de Flota: Administrar Muchos Agentes a la Vez

> **Serie: Integraciones y Automatización — Parte 3 de 3**

**Tiempo de lectura:** 8 minutos | **Dificultad:** Intermedio 🟡

---

## TL;DR

- Con 1-2 máquinas gestionas el agente manualmente. Con 10+ necesitas un flujo diferente
- Puedes ver el estado de toda tu flota de un vistazo: quién está online, qué versión tiene, cuándo fue su último test
- Los agentes se actualizan a distancia con un click — sin tocar cada máquina
- Las **campañas de tests** te permiten asignar tests a toda la flota o grupos de máquinas a la vez
- Los **schedules** automatizan los tests para que corran solos cada semana sin que hagas nada

---

## Del Piloto a Producción

En el quickstart conectaste 1-2 máquinas. Cuando empiezas a escalar, las preguntas cambian:

```
Con 2 máquinas:               Con 20 máquinas:
→ ¿Está online mi PC?         → ¿Cuáles de las 20 están offline?
→ ¿Ejecuto un test?           → ¿Cómo ejecuto tests en todas a la vez?
→ ¿Actualizo el agente?       → ¿Cómo actualizo 20 agentes sin SSH?
→ ¿Qué resultados tengo?      → ¿Qué máquinas tienen el peor score?
```

Este post cubre exactamente ese salto.

---

## Vista de Flota: El Estado de Todo de un Vistazo

```
Endpoints → (vista tabla)

ENDPOINTS                                              23 online · 2 offline

Hostname          Sistema    Versión   IP              Estado    Últ. Test
──────────────────────────────────────────────────────────────────────────
DESKTOP-SRV01     Windows    1.4.2     192.168.1.45   🟢 Online   hace 2h
DESKTOP-SRV02     Windows    1.4.2     192.168.1.46   🟢 Online   hace 2h
LINUX-PROD-01     Linux      1.4.2     10.0.0.12      🟢 Online   hace 2h
SERVER-ARCHIVOS   Windows    1.3.0     192.168.1.78   🟡 Online   hace 5d  ← versión vieja
DESKTOP-OLD01     Windows    1.4.2     192.168.1.90   🔴 Offline  hace 3h
MACBOOK-DEV01     macOS      1.4.2     192.168.1.100  🟢 Online   hace 2h
...
```

Las señales que buscar:

```
🔴 Offline → El agente no reportó en los últimos 3 minutos
   → Máquina apagada, agente caído, o problema de red

🟡 Versión vieja → Tiene una versión anterior del agente
   → Actualizar para tener las últimas capacidades

"Últ. test: hace 5d" → Esta máquina no se testea hace días
   → Revisar si tiene schedules asignados
```

---

## Actualizar Agentes Remotamente

Cuando hay una nueva versión del agente, no necesitas tocar cada máquina:

### Actualizar una máquina

```
Endpoints → click en "SERVER-ARCHIVOS" → [ Actualizar Agente ]

"¿Actualizar de v1.3.0 a v1.4.2?"
[ Confirmar ]

→ El agente descarga la nueva versión
→ Verifica que el archivo es legítimo (firma digital)
→ Se reinicia automáticamente
→ En 60 segundos: "SERVER-ARCHIVOS — v1.4.2 🟢 Online"
```

### Actualizar toda la flota a la vez

```
Endpoints → [ Actualizar todos ] → Actualizar los que tengan versión vieja

"3 agentes serán actualizados de v1.3.0 a v1.4.2"
[ Confirmar ]

→ Los 3 agentes se actualizan en paralelo
→ Sin interrumpir el servicio en los que ya están en v1.4.2
```

El agente verifica la firma digital antes de aplicar cualquier actualización. Si la verificación falla, el agente rechaza la actualización y queda en la versión anterior — nunca aplica algo que no sea legítimo.

---

## Campañas de Tests: Lanzar Tests a Toda la Flota

En lugar de asignar tests máquina por máquina, las **campañas** te permiten asignar en bloque:

### Campaña sencilla: mismo test en todas las máquinas

```
Analytics → [+ Nueva Campaña]

Tests a ejecutar:
  ☑ PowerShell Encoded Command
  ☑ Process Injection
  ☑ LSASS Memory Access

Destino:
  ● Todas las máquinas Windows online  (15 máquinas)
  ○ Grupo específico
  ○ Máquinas seleccionadas

Horario:
  ○ Ahora mismo
  ● Esta noche a las 02:00 AM

[ Crear Campaña ]

→ 45 tareas creadas (3 tests × 15 máquinas)
→ Ejecución programada: 2026-05-15 02:00 AM
→ Duración estimada: 20 minutos
```

Al día siguiente, todos los resultados están en Analytics — una vista unificada de cómo se comportaron todas las máquinas frente a los mismos tests.

---

## Schedules: Validación Automática Semanal

El siguiente nivel es que los tests se ejecuten solos, sin que tengas que hacer nada.

### Crear un schedule

```
Endpoints → Schedules → [+ Nuevo Schedule]

Nombre:   "Validación semanal - Windows"

Tests:
  ● Bundle cyber-hygiene (23 controles)
  ○ Tests individuales

Destino:
  ● Todas las máquinas Windows

Frecuencia:
  ● Semanal → Todos los lunes a las 02:00 AM

Notificación al terminar:
  ✅ Enviar resumen a Slack #seguridad-resultados

[ Guardar ]
```

Con esto, cada lunes a las 2 de la mañana Achilles:
1. Ejecuta el bundle en todas las máquinas Windows
2. Recoge los resultados
3. Actualiza el Defense Score
4. Envía un resumen a Slack

Tú llegas el lunes y tienes los resultados esperándote — sin haber hecho nada.

### Schedules recomendados para empezar

```
Semanal (lunes 02:00 AM):
→ Bundle cyber-hygiene en todos los Windows
→ Resultado: estado del hardening de la flota

Semanal (martes 02:00 AM — escalonado para no saturar la red):
→ Técnicas críticas de Execution y Credential Access
→ Resultado: cobertura de las técnicas más usadas por atacantes

Mensual (primer lunes del mes, 02:00 AM):
→ Suite completa — todos los tests disponibles
→ Resultado: benchmark mensual completo del Defense Score
```

---

## Filtrar y Segmentar la Flota

Con muchas máquinas, los filtros de Achilles permiten segmentar el análisis:

### Por sistema operativo

```
Analytics → Filtro: Sistema = "Windows"
→ Defense Score solo para tus Windows: 71%

Analytics → Filtro: Sistema = "Linux"
→ Defense Score solo para tus Linux: 68%
→ Los servidores Linux tienen peor cobertura → prioridad
```

### Por grupo (si tienes máquinas etiquetadas)

```
Endpoints → [máquina] → Editar → Etiquetas: "produccion", "servidores"

Analytics → Filtro: Etiqueta = "produccion"
→ Defense Score de los servidores de producción específicamente
```

### Por Defense Score más bajo

```
Analytics → Defense Score por Máquina (gráfico barras)
→ Ordenado de menor a mayor

Las 3 peores máquinas esta semana:
1. SERVER-ARCHIVOS    48%  🔴
2. DESKTOP-OLD01      52%  🔴
3. LINUX-BACKUP-01    61%  🟡

→ Estas tres son la prioridad de hardening esta semana
```

---

## Revocar Acceso de una Máquina

Cuando una máquina sale de uso, la das de baja en Achilles:

```
Endpoints → click en la máquina → [ Revocar acceso ]

"¿Revocar acceso a DESKTOP-OLD01? El agente dejará de reportar."
[ Confirmar ]

→ La API key del agente queda invalidada
→ Aunque el agente siga instalado en la máquina, no puede conectarse
→ La máquina aparece como "Revocada" en el historial
```

El agente sigue instalado en la máquina hasta que lo desinstales manualmente, pero no puede hacer nada — está bloqueado por el backend.

Para desinstalar completamente:

```bash
# Windows (PowerShell como Administrador):
C:\achilles\achilles-agent-windows.exe uninstall

# Linux:
sudo achilles-agent uninstall

# macOS:
sudo achilles-agent-macos-arm uninstall
```

---

## Consejos para Flotas Grandes

### Automatizar el enrolamiento inicial

Si tienes que instalar el agente en 50 máquinas, hazlo con tus herramientas existentes:

```powershell
# Con SCCM / Intune (Windows):
# Crea un paquete de software con este script:

$token = "eyJhY2hpbGxlcyI6..."  # Token con múltiples usos permitidos
$server = "http://achilles.tuempresa.com:3000"

New-Item -ItemType Directory -Path "C:\achilles" -Force
# (descarga el binario desde un share interno o URL)
Invoke-WebRequest -Uri "$server/agent/download/windows" -OutFile "C:\achilles\achilles-agent.exe"
C:\achilles\achilles-agent.exe install --server $server --token $token
```

```bash
# Con Ansible (Linux):
- name: Install Achilles agent
  shell: |
    chmod +x /tmp/achilles-agent-linux
    /tmp/achilles-agent-linux install \
      --server http://achilles.tuempresa.com:3000 \
      --token {{ achilles_token }}
```

### Tokens para múltiples máquinas

Por defecto los tokens son de un solo uso. Para enrolamiento masivo:

```
Endpoints → Tokens → [+ Nuevo Token]
Usos máximos: 50  (o el número que necesites)
Válido por: 7 días
```

El token permite hasta 50 enrolamientos antes de expirar.

---

## Puntos Clave

✅ La vista de flota muestra estado, versión y último test de cada máquina
✅ Actualizaciones remotas con un click — sin SSH ni acceso físico a las máquinas
✅ Las campañas asignan tests a toda la flota o grupos en una operación
✅ Los schedules ejecutan tests automáticamente cada semana sin trabajo manual
✅ Revocar acceso invalida la API key del agente inmediatamente
✅ Para 50+ máquinas: automatizar el enrolamiento con SCCM, Ansible o scripts

---

## Cierre de la Parte 3

Con la Parte 3 completada tienes:
- ✅ Defender integrado y correlacionando alertas con tests
- ✅ Auto-Resolve limpiando el ruido del SOC
- ✅ Flota de agentes gestionada a escala con schedules automáticos

La **Parte 4 — Pro** cubre los flujos avanzados:
- Purple Team workflow completo (de intel de amenazas a evidencia)
- Compilar y firmar tus propios agentes
- Evidencia para auditorías DORA, TIBER-EU e ISO 27001

---

**Etiquetas:** #ProjectAchilles #GestiónDeFlota #Automatización #Agentes #CiberSeguridad #Tutorial #DevSecOps

---

*Parte 3 de 3 en la serie "Integraciones y Automatización con Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
