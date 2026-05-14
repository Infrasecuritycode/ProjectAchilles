# Tu Primer Test de Seguridad con Achilles: De Cero a Resultado

> **Serie: Validación Continua de Seguridad con Project Achilles — Parte 6 de 15**

**Tiempo de lectura:** 10 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Un test en Achilles toma 3 pasos: seleccionar test → asignar a agente → ver resultado
- El agente descarga el binario, lo ejecuta, y reporta el resultado en segundos
- El resultado va directo a Elasticsearch y actualiza el Defense Score en tiempo real
- Puedes ver la salida del test, el exit_code, y el documento ES generado
- Primera prueba recomendada: `T1059.001-powershell-exec` (bajo impacto, alto valor informativo)

---

## Escenario: Tu Primera Validación Real

Tienes Achilles instalado y un agente corriendo en `DESKTOP-SRV01` (Windows). Quieres responder esta pregunta:

> **¿Detecta nuestro SIEM la ejecución de PowerShell con comandos codificados en base64?**

Esta es una de las técnicas más usadas por APTs reales (APT29, Cobalt Strike, Emotet). Si no la detectas, tienes un gap crítico.

---

## Paso 1: Compilar el Test

Antes de asignar un test a un agente, necesitas compilarlo para la plataforma del endpoint.

### Desde el Browser

```
1. Ve a Browser → busca "T1059.001"
2. Click en el test: "PowerShell Execution - Encoded Command"
3. Click en [ Compilar ]

Opciones de compilación:
  Plataforma: ✅ Windows x64
  Firma: ✅ Authenticode (requiere certificado activo)
  
  [ Compilar y Firmar ]
```

### ¿Qué pasa al compilar?

```
Backend recibe la solicitud de build:
→ Cross-compila con: GOOS=windows GOARCH=amd64 go build
→ Firma con osslsigncode usando el certificado activo
→ Almacena el binario firmado: T1059.001-win-amd64.exe

Build log visible en tiempo real:
✓ Dependencias resueltas
✓ go build completado (1.2s)
✓ osslsigncode: firma aplicada
✓ Binario listo: 3.2 MB
```

**Si no tienes certificado**: puedes compilar sin firma. El test funciona igual, pero algunos EDRs pueden bloquearlo por binario sin firma (lo cual es en sí mismo una señal útil).

---

## Paso 2: Asignar el Test a un Agente

### Opción A: Asignación directa desde el Browser

```
Browser → T1059.001 → [ Asignar a Agente ]

Destino: [DESKTOP-SRV01 ▼]
Horario: ● Ahora  ○ Programar

[ Asignar ]
```

### Opción B: Desde el Dashboard de Endpoints

```
Endpoints → DESKTOP-SRV01 → [ + Nueva Tarea ]

Test:     [T1059.001-powershell-exec ▼]
Horario:  [Ahora ▼]

[ Crear Tarea ]
→ Tarea creada: task_id = abc-123
→ Estado: Pendiente
```

### Opción C: Via API

```bash
curl -X POST http://localhost:3000/api/agent/admin/tasks \
  -H "Authorization: Bearer $CLERK_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "agentId": "uuid-desktop-srv01",
    "testId": "T1059.001-powershell-exec",
    "scheduledAt": null
  }'

# Respuesta:
{
  "success": true,
  "data": {
    "taskId": "abc-123-def-456",
    "status": "pending",
    "estimatedExecution": "< 60 segundos"
  }
}
```

---

## Paso 3: El Agente Ejecuta el Test

Esto pasa automáticamente — no necesitas hacer nada. El agente hace poll del backend cada 60 segundos:

```
[DESKTOP-SRV01 / Agente]

tick... → GET /api/agent/tasks/pending
           ← { tasks: [ {id: "abc-123", binary: "url...", signature: "..."} ] }

→ Descarga binario: T1059.001-win-amd64.exe
→ Verifica firma Ed25519
→ Ejecuta: C:\Users\TEMP\achilles\T1059.001-win-amd64.exe
→ Espera resultado (timeout: 30s)
→ exit_code: 1

→ POST /api/agent/tasks/abc-123/result
  {
    "exit_code": 1,
    "output": "Access denied. PowerShell execution policy blocked.",
    "duration_ms": 234,
    "executed_at": "2026-05-14T10:45:23Z"
  }
```

---

## Paso 4: Ver el Resultado

### En el Dashboard de Endpoints

```
Endpoints → DESKTOP-SRV01 → Historial de Tareas

T1059.001-powershell-exec   ✅ PROTEGIDO   exit:1   2026-05-14 10:45:23
```

**exit_code: 1 = PROTEGIDO** — tu defensa bloqueó el intento de ejecutar PowerShell encodado. ¡Buenas noticias!

### En Analytics

El resultado ya está en Elasticsearch. Puedes verlo en:

```
Analytics → Executions Table

Test:       T1059.001-powershell-exec
Host:       DESKTOP-SRV01
Resultado:  ✅ Protegido
Táctica:    Execution
Severidad:  HIGH
Timestamp:  2026-05-14 10:45:23
Exit code:  1
```

### En Elasticsearch (directamente)

```bash
curl http://localhost:9200/achilles-results/_search \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "match": { "f0rtika.test_uuid": "T1059.001-powershell-exec" }
    },
    "sort": [{ "@timestamp": "desc" }],
    "size": 1
  }'
```

```json
{
  "_source": {
    "f0rtika": {
      "test_name": "T1059.001-powershell-exec",
      "technique_id": "T1059.001",
      "tactic": "Execution",
      "severity": "high",
      "exit_code": 1,
      "protected": true,
      "hostname": "DESKTOP-SRV01",
      "platform": "windows"
    },
    "@timestamp": "2026-05-14T10:45:23Z"
  }
}
```

---

## Interpretando los Resultados

### exit_code 1: PROTEGIDO ✅

```
El test fue bloqueado antes de completarse.

¿Qué puede significar?
→ Windows Defender bloqueó el proceso
→ EDR interceptó la ejecución
→ Política de PowerShell bloqueó el comando
→ AppLocker/WDAC impidió la ejecución del binario

Próximo paso: Correlacionar con alertas en el SIEM
→ ¿Hay un evento generado? ¿El analista sería notificado?
→ Detectado + alerta = PROTEGIDO COMPLETO ✅
→ Detectado sin alerta = PROTEGIDO SILENCIOSO ⚠️ (gap en visibilidad)
```

### exit_code 0: GAP DE DETECCIÓN ❌

```
El test se ejecutó sin ser bloqueado.

Interpretación: la técnica pasaría desapercibida en un ataque real.

Próximo paso: Acción correctiva
→ Revisar configuración de EDR para esta técnica
→ Crear/ajustar regla SIEM (Event ID 4104 para PS Script Block)
→ Habilitar PowerShell Constrained Language Mode
→ Verificar cobertura en MITRE ATT&CK Navigator
```

### El Defense Score después del primer test

```
Antes del primer test:
Defense Score: N/A (sin datos)

Después de T1059.001 → exit_code 1:
Defense Score: 100% (1/1 tests protegidos)

Conforme añades más tests:
T1078.002 → exit_code 0 (GAP)
T1021.001 → exit_code 1 (protegido)
T1055.001 → exit_code 0 (GAP)

Defense Score: 50% (2/4 tests protegidos)
→ Tienes gaps en Valid Accounts y Process Injection
```

---

## Segundo Test: Detectar un Gap Real

Ahora que sabes que el PowerShell está protegido, probemos algo que típicamente NO está configurado en la mayoría de organizaciones.

### T1055.001 — Process Injection

```
1. Browser → busca "T1055"
2. Test: "Process Injection - DLL Injection"
3. Compilar para Windows x64
4. Asignar a DESKTOP-SRV01

Resultado esperado en ~80% de organizaciones:
→ exit_code 0 (GAP) — la mayoría de EDRs requieren configuración adicional
                       para detectar todas las variantes de injection
```

Si obtienes `exit_code 0` aquí, tienes evidencia concreta de un gap que necesita acción.

---

## Programar Tests Recurrentes

Los tests manuales son útiles para auditorías. Los tests recurrentes son más valiosos para validación continua:

```
Endpoints → Schedules → [+ Nueva Programación]

Test:       T1059.001-powershell-exec
Agente:     DESKTOP-SRV01
Frecuencia: Cada lunes a las 02:00 AM
            (fuera de horario de producción)

[ Guardar ]
→ Se ejecutará automáticamente cada semana
→ El Defense Score se actualiza solo
→ Cualquier regresión es visible inmediatamente
```

**Por qué programar fuera del horario de producción**: los tests generan procesos y actividad de red. Aunque son inofensivos, es mejor ejecutarlos cuando el impacto en la experiencia de usuario es mínimo.

---

## Tests en Lote: Asignar a Toda la Flota

Quieres ejecutar el mismo test en todos los endpoints:

```
Analytics → [+ Nueva Campaña de Tests]

Tests:   Seleccionar todos en Execution (67 tests)
Agentes: ✅ Todos los Windows online
Horario: Esta noche, 01:00 AM

[ Crear campaña ]
→ 67 tareas creadas para 15 agentes = 1,005 ejecuciones
→ Duración estimada: 45 minutos
→ Resultados visibles en Analytics al terminar
```

---

## Correlación con tu SIEM

El valor real de un test no es solo el exit_code — es correlacionar ese resultado con las alertas de tu SIEM.

### Flujo de correlación manual

```
1. Ejecutas T1059.001-powershell-exec → exit_code 1 (bloqueado)
2. Vas a tu SIEM (Elasticsearch, Splunk, QRadar...)
3. Buscas eventos cerca del timestamp del test en DESKTOP-SRV01
4. Encuentras:
   - Event ID 4104 (PowerShell Script Block Logging)
   - Alert de Defender: "Suspicious PowerShell Command"
5. Conclusión: detectado Y alertado → PROTECCIÓN COMPLETA ✅

Si encuentras:
- Bloqueado pero sin alerta → detectado pero sin visibilidad
- No bloqueado y sin alerta → gap crítico
```

### Con la integración Defender (POST 10)

Cuando configures la integración con Microsoft Defender, esta correlación es automática:

```
Analytics → Defender Tab → Cross-Correlation

Test ejecutados este mes:     312
Alertas Defender correlacionadas: 189 (60.6%)
Sin correlación (gap):          123 (39.4%)

→ 39.4% de tus tests no generaron alerta en Defender
→ Lista ordenada por severidad para priorizar acción
```

---

## Checklist: Primer Test Exitoso

```
✅ Agente instalado y online en al menos 1 endpoint
✅ Test T1059.001 compilado para la plataforma del endpoint
✅ Tarea asignada y ejecutada (< 60 segundos)
✅ Resultado visible en Endpoints → Historial de Tareas
✅ Resultado visible en Analytics → Executions Table
✅ Documento creado en Elasticsearch (achilles-results)
✅ Defense Score actualizado
```

---

## Puntos Clave

✅ 3 pasos: compilar → asignar → ver resultado
✅ El agente ejecuta automáticamente en su próximo tick (< 60s)
✅ `exit_code 1` = protegido, `exit_code 0` = gap de detección
✅ El resultado va directo a ES y actualiza el Defense Score en tiempo real
✅ Programar tests recurrentes = validación continua real
✅ Correlacionar con SIEM = diferencia entre "bloqueado" y "detectado+alertado"

---

## Próximo Post

**POST 07: "Defense Score — Tu Primera Métrica Real de Seguridad"**

Qué es exactamente el Defense Score, cómo se calcula, cómo interpretarlo, y cómo usarlo para priorizar inversiones de seguridad.

---

**Etiquetas:** #ProjectAchilles #PurpleTeam #MITRE #CiberSeguridad #SOC #Tutorial #BAS #BlueTeam

---

*Parte 6 de 15 en la serie "Validación Continua de Seguridad con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026 | **Serie:** Validación Continua con Achilles
