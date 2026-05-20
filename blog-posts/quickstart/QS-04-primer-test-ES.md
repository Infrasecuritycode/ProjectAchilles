# Ejecutar Tu Primer Test: Ver Achilles en Acción

> **Serie: Empezando con Project Achilles — Parte 4 de 6**

**Tiempo de lectura:** 7 minutos | **Dificultad:** Principiante 🟢

---

## TL;DR

- Un "test" en Achilles es una simulación de técnica de ataque que se ejecuta en tu máquina
- El proceso toma menos de 2 minutos: seleccionar test → asignar → ver resultado
- El resultado es simple: ✅ Protegido o ❌ No detectado
- El test recomendado para empezar: PowerShell con comando oculto (uno de los más usados por hackers reales)
- Los tests son seguros, no dañan la máquina ni los datos

---

## ¿Qué Es un Test de Achilles?

Cuando Achilles "ejecuta un test", lo que pasa es esto:

1. El agente descarga un pequeño programa a la máquina
2. Ese programa ejecuta una acción específica que un hacker real haría (por ejemplo: intentar ejecutar un comando de PowerShell oculto)
3. Si tu seguridad lo detecta y bloquea → ✅ **Protegido**
4. Si pasa sin que nadie lo note → ❌ **No detectado** (tienes un gap)
5. El resultado llega al dashboard en segundos

Los tests no hacen daño. No roban datos. No modifican nada permanente. Son como un simulacro: hacen la acción de forma controlada para ver si la alarma suena.

---

## El Test Recomendado para Empezar

Para tu primera prueba, te recomendamos:

**PowerShell con comando codificado en base64**

¿Por qué este? Porque es una de las técnicas más usadas por hackers reales en los últimos 5 años. Si tu seguridad no lo detecta, tienes un gap real y común.

En términos simples: es como si alguien intentara hablarle a tu computadora en un idioma cifrado para que el antivirus no entienda qué está haciendo. Si tu defensa lo detecta, bien. Si no, es algo que deberías corregir.

---

## Paso 1: Ir al Browser

En el menú principal, click en **Browser**.

```
PROJECT ACHILLES    [Analytics] [Endpoints] [Browser] [Settings]
```

Verás la biblioteca de tests disponibles:

```
BROWSER DE TESTS                        512 tests disponibles

Buscar: [_________________________________]

Filtros: [Todas las tácticas ▼] [Todas las plataformas ▼] [Todas las severidades ▼]

─────────────────────────────────────────────────────────────
PowerShell - Encoded Command           ALTO     Windows
PowerShell - AMSI Bypass               CRÍTICO  Windows
Remote Desktop Protocol Brute          ALTO     Windows
Process Injection via DLL              CRÍTICO  Windows
...
```

Busca "PowerShell Encoded" o simplemente "T1059" en el buscador.

---

## Paso 2: Ver los Detalles del Test

Click en el test "PowerShell - Encoded Command":

```
┌─────────────────────────────────────────────────────────────┐
│  PowerShell - Encoded Command                               │
│  T1059.001 · Ejecución · ALTO                               │
├─────────────────────────────────────────────────────────────┤
│  Qué hace:                                                  │
│  Ejecuta un comando de PowerShell codificado en base64,     │
│  técnica usada por Cobalt Strike, Emotet y APT29.           │
│                                                             │
│  Qué debería detectar tu seguridad:                         │
│  → Windows Defender: alerta de "Suspicious PowerShell"      │
│  → SIEM: Event ID 4104 (Script Block Logging)               │
│                                                             │
│  Plataformas: Windows                                       │
│  Severidad:   ALTO                                          │
├─────────────────────────────────────────────────────────────┤
│  [ Compilar ]    [ Asignar a Agente ]                       │
└─────────────────────────────────────────────────────────────┘
```

Esto te dice exactamente qué va a pasar y qué debería detectarlo. Transparencia total.

---

## Paso 3: Compilar el Test

Antes de ejecutarlo, Achilles necesita preparar el archivo del test para tu plataforma:

```
Click en [ Compilar ]

Plataforma: ● Windows 64-bit  ○ Linux  ○ macOS

[ Compilar ]

Preparando...
✓ Compilando para Windows...   (1.2 segundos)
✓ Listo para ejecutar
```

Solo tarda unos segundos. Achilles genera el binario específico para tu sistema operativo.

---

## Paso 4: Asignar a Tu Máquina

```
Click en [ Asignar a Agente ]

Selecciona la máquina:
● MI-PC-01 (Windows · Online)

¿Cuándo ejecutar?
● Ahora mismo
○ Programar para más tarde

[ Asignar ]

✓ Test asignado. Se ejecutará en los próximos 60 segundos.
```

---

## Paso 5: Ver el Resultado

Vuelve a **Endpoints → MI-PC-01** y mira el historial:

```
Historial de Tests — MI-PC-01

PowerShell Encoded Command   ✅ PROTEGIDO   hace 45 seg
```

O ve directo a **Analytics → Executions** para ver todos los resultados:

```
Test                        Máquina     Resultado    Hora
─────────────────────────────────────────────────────────
PowerShell Encoded Command  MI-PC-01   ✅ Protegido  10:45:23
```

---

## Interpretar el Resultado

### ✅ PROTEGIDO — Buenas noticias

```
Tu seguridad detectó y bloqueó la simulación.

¿Qué puede significar?
→ Windows Defender bloqueó el proceso antes de que se ejecutara
→ Una política de seguridad impidió el comando
→ El EDR interceptó el comportamiento

¿Qué hacer?
→ Verifica si generó una alerta en tu herramienta de seguridad
→ ¿Alguien la habría visto? ¿O está sepultada entre cientos de alertas?
→ Detectado SIN alerta visible = protegido en silencio (también hay que revisar)
```

### ❌ NO DETECTADO — Información valiosa

```
La simulación se ejecutó sin que nadie lo notara.

¿Qué significa?
→ Esta técnica pasaría desapercibida en un ataque real contra esta máquina

¿Qué hacer?
→ Revisar la configuración de PowerShell Logging en esa máquina
→ Verificar si Script Block Logging está activado
→ Revisar políticas de PowerShell en la política de grupo (GPO)

¿Es una emergencia?
→ No. Ahora lo sabes. Antes no lo sabías. Eso es el valor de Achilles.
```

---

## Ejecutar Tu Segunda Prueba: Buscar un Gap Real

Después de ver cómo funciona con PowerShell, prueba algo que típicamente tiene gaps en muchas organizaciones:

**Process Injection**, técnica donde un programa malicioso se "mete" dentro de otro proceso legítimo para esconderse.

```
Browser → busca "Process Injection"
→ Selecciona el primer resultado
→ Compila → Asignar → Observa

Resultado esperado en la mayoría de organizaciones:
❌ No detectado — es una técnica más avanzada que requiere
                   configuración específica del EDR para detectar
```

Si obtienes ❌ aquí, acabas de encontrar un gap real que vale la pena corregir.

---

## Ejecutar Varios Tests de una Vez

No tienes que hacerlo uno por uno. Puedes seleccionar varios tests y asignarlos juntos:

```
Browser → selecciona varios tests con ☐
☑ PowerShell Encoded Command
☑ Process Injection
☑ Remote Desktop Brute Force

[ Asignar todos a MI-PC-01 ]
→ 3 tests asignados
→ Se ejecutarán en los próximos 3 minutos
```

O mejor aún: Achilles tiene **bundles**, colecciones de tests relacionados que se ejecutan juntos. Los exploramos en la Parte 2 de esta serie.

---

## Programar Tests Automáticos

Ejecutar tests manualmente es útil para empezar. Pero el verdadero valor es la **automatización**:

```
Endpoints → Schedules → [+ Nueva Programación]

Test:     PowerShell Encoded Command
Máquina:  MI-PC-01
Horario:  Cada lunes a las 02:00 AM

[ Guardar ]
```

Con esto, Achilles valida automáticamente cada semana sin que hagas nada. Si algo cambia (una actualización de Defender, un cambio de política), lo verás reflejado en el Defense Score.

---

## Puntos Clave

✅ Un test = una simulación de técnica de ataque + resultado en segundos
✅ ✅ Protegido = tu defensa lo detectó · ❌ No detectado = tienes un gap
✅ El proceso toma menos de 2 minutos: compilar → asignar → ver resultado
✅ Los tests son seguros, no dañan datos ni modifican configuraciones
✅ Empieza con PowerShell Encoded Command, fácil de interpretar y muy relevante

---

## Próximo Post

**QS-05: "Entender el Defense Score. Qué Significa Ese Número"**

Profundizamos en el número más importante de Achilles: qué es exactamente, cómo se calcula, y cómo usarlo para tomar decisiones.

---

**Etiquetas:** #ProjectAchilles #CiberSeguridad #Tutorial #PrimerTest #Principiantes

---

*Parte 4 de 6 en la serie "Empezando con Project Achilles".*

**Autor:** Kendra Mazara | **Fecha:** Mayo 2026
