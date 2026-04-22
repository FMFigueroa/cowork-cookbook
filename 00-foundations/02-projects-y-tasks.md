# Projects en Cowork — donde vive y se ejecuta el trabajo

> Documento vivo. Detalla la unidad fundamental de Cowork — el Project — y cómo ejecuta el trabajo a través del Task Loop.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es un Project?](#-qué-es-un-project)
- [Anatomía de un Project](#️-anatomía-de-un-project)
- [Flujo: cómo se ejecuta un Project](#-flujo-cómo-se-ejecuta-un-project)
- [Contexto ≠ Output](#️-contexto--output)
- [El Task Loop: cómo Cowork ejecuta cada tarea](#-el-task-loop-cómo-cowork-ejecuta-cada-tarea)
- [Los 4 pasos del loop](#-los-4-pasos-del-loop)
- [Bajo el capó: 5 pilares de ejecución](#-bajo-el-capó-5-pilares-de-ejecución)
- [Steering: redirigir en tiempo real](#️-steering-redirigir-en-tiempo-real)
- [Entorno aislado: qué sí y qué no toca Cowork](#️-entorno-aislado-qué-sí-y-qué-no-toca-cowork)
- [Mindset de revisión](#-mindset-de-revisión)
- [Walkthrough end-to-end](#-walkthrough-end-to-end)
- [Tips de prompting](#️-tips-de-prompting)

---

## 💡 ¿Qué es un Project?

Un **Project** es la **unidad fundamental de Cowork**: el contenedor que agrupa todo lo que el agente necesita para trabajar en un dominio o iniciativa tuya. Es donde configuras cómo debe comportarse Cowork y a qué información tiene acceso.

**Cowork no trabaja sin un Project.** Todo lo que ejecuta — desde un briefing diario hasta un pipeline multi-agente — vive dentro de un Project.

### ¿Para qué sirve?

- **Agrupar el contexto** de una iniciativa en un solo lugar (archivos, links, memoria persistente).
- **Definir cómo se comporta el agente** en ese dominio específico (tono, restricciones, qué Skills usar).
- **Programar tareas** que corren bajo demanda o en cadencia (diarias, semanales, manuales).
- **Aislar el trabajo** — los outputs de un Project no se mezclan con los de otro.

### ¿Por qué Cowork se organiza por Projects y no por tareas sueltas?

Porque las tareas **rara vez son aisladas** — una investigación de mercado, un cierre financiero, la preparación de leadership briefs: todas involucran múltiples ejecuciones sobre el mismo pool de archivos y bajo las mismas reglas. Un Project te deja:

- Configurar el contexto una sola vez y reutilizarlo en N tareas.
- Evolucionar las Instructions sin re-explicarte cada vez que lanzas trabajo.
- Mantener una memoria persistente entre ejecuciones.

---

## 🏗️ Anatomía de un Project

Un Project se compone de **3 elementos fundamentales**:

```
┌─────────────────────────────────────────────────────────┐
│                     📂 PROJECT                          │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Instructions │  │   Schedule   │  │   Contexto   │   │
│  │              │  │  (las Tasks) │  │              │   │
│  │ system       │  │              │  │ read-only    │   │
│  │ prompt +     │  │ bajo demanda │  │ fuentes +    │   │
│  │ recursos     │  │ o recurrente │  │ memoria      │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1. Instructions

Es el **system prompt** del Project. Aquí defines cómo debe comportarse Cowork para este Project específico:

- Tono y estilo de comunicación.
- Restricciones (qué no hacer, qué aprobaciones necesita).
- Qué Skills priorizar cuando aplique.
- Cómo estructurar los outputs.
- Dónde dejar los resultados.

Es la capa donde **haces referencia a los recursos globales** (Skills y Conectores) que quieres que Cowork priorice en este Project.

### 2. Schedule — las Tasks del Project

La lista de **tareas planificadas** del Project. Una Task es la unidad ejecutable: un prompt + configuración de cuándo y cómo corre.

Cada tarea se configura con los campos del diálogo _Create scheduled task_ de Cowork:

| Campo                 | Qué es                                                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Name**              | Identificador de la tarea (ej. `daily-briefing`)                                                                 |
| **Description**       | Resumen en una línea de lo que hace                                                                              |
| **Prompt**            | Las instrucciones completas que ejecutará la tarea                                                               |
| **Work in a project** | El folder del Project donde corre la tarea (heredado del Project seleccionado, aparece como `from project`)      |
| **Approval mode**     | `Ask for approvals` (default, pausa ante cada tool call) o `Skip all approvals` (ejecuta autónomo sin preguntar) |
| **Model**             | El modelo de Claude (por defecto o específico por tarea)                                                         |
| **Frequency**         | `Manual` (bajo demanda), `Hourly`, `Daily`, `Weekdays` (lunes a viernes) o `Weekly`                              |

Cada tarea queda **vinculada al Project** y, por extensión, al directorio de tu máquina asociado a ese Project.

> 🛡️ **Approval mode es una decisión de riesgo.** `Ask for approvals` te da control fino pero rompe la promesa de automatización (tienes que estar presente). `Skip all approvals` es el modo headless real — úsalo solo cuando confíes completamente en el prompt + contexto de la tarea.

### 3. Contexto

La **información base** del Project. Los agentes **leen** este contexto pero **no lo modifican** — es read-only.

El Contexto se alimenta de 3 fuentes externas + 1 memoria interna:

| Fuente                  | Qué es                                                                            |
| ----------------------- | --------------------------------------------------------------------------------- |
| **Carpeta local**       | Una carpeta en tu ordenador con los archivos base del Project                     |
| **Links / URLs**        | Referencias a sitios web que aportan contexto                                     |
| **Google Drive**        | Documentos en la nube                                                             |
| **Memoria del Project** | Un espacio de memoria reservado donde el agente persiste información entre tareas |

---

## 🔄 Flujo: cómo se ejecuta un Project

```
Instructions   (system prompt: cómo comportarse)
    +
Schedule       (cuándo: bajo demanda o recurrente)
    +
Contexto       (qué sabe: fuente de verdad read-only)
    +
User prompt    (la acción específica a ejecutar)
    ↓
Agentes orquestados trabajan en el entorno virtual
    ↓
Output escrito en la ubicación que definiste en Instructions
```

Las 3 capas del Project se combinan con el **prompt específico de cada tarea** para producir ejecuciones concretas. Las Instructions y el Contexto se mantienen constantes; lo único que cambia entre tareas es el prompt y los outputs.

---

## 🛡️ Contexto ≠ Output

**El contexto no se toca.** Cowork **lee** los archivos de contexto y solo **crea archivos nuevos** en la ubicación de salida que hayas especificado en las Instructions.

Tus documentos base quedan intactos; el Project genera siempre outputs separados, sin riesgo de sobreescribir tu fuente de verdad.

Esto significa que puedes ejecutar una Task 10 veces sobre el mismo Contexto sin preocuparte de erosionar tus archivos originales. Cada ejecución produce outputs en paralelo, nunca mutaciones del contexto.

---

## 🔁 El Task Loop: cómo Cowork ejecuta cada tarea

Hasta acá vimos **dónde vive** el trabajo (el Project). Ahora veamos **cómo se ejecuta** cada tarea dentro de ese Project.

> 📖 **Del curso (Anthropic Academy):** "Claude proposes a plan and waits for your approval before taking action. You adjust if needed, approve, and Claude executes. This is the pattern for every Cowork task."

El **Task Loop** es el ciclo de 4 pasos que Cowork sigue para ejecutar cualquier tarea. Empieza cuando describes lo que necesitas y termina cuando el output final aterriza en tu filesystem.

### ¿Para qué existe como loop y no como ejecución directa?

Porque Cowork es un **agente autónomo** que trabaja sobre archivos, APIs y sistemas reales. Si ejecutara sin alineamiento, correrías riesgos:

- Interpretar mal el outcome y producir algo inútil.
- Tomar una aproximación técnica que no se alinea con tu contexto.
- Modificar o borrar archivos que no esperabas.

El loop te da **puntos de intervención explícitos** antes y durante la ejecución.

---

## 🔢 Los 4 pasos del loop

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   1. DESCRIBE         2. REVISA PLAN      3. MONITOREA       │
│   ──────────          ─────────────       (o te vas)         │
│   qué quieres         Cowork arma         ────────────       │
│   de vuelta           plan + hace         Cowork ejecuta,    │
│         │             preguntas           subagentes         │
│         │             si faltan datos     paralelos          │
│         ▼                    │                  │            │
│   [tu prompt]           [alineamiento]    [ejecución]        │
│                              │                  │            │
│                              ▼                  ▼            │
│                                                              │
│                  ┌──── steering en vivo ────┐                │
│                  │  (chat para redirigir)   │                │
│                  └──────────┬───────────────┘                │
│                             │                                │
│                             ▼                                │
│                     4. ABRE EL OUTPUT                        │
│                     ─────────────────                        │
│                     archivo aterriza                         │
│                     en tu filesystem                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Paso 1 — Describe el outcome

Tu prompt funciona bien cuando le da a Cowork tres piezas:

| Pieza                   | Ejemplo                                                            |
| ----------------------- | ------------------------------------------------------------------ |
| **Qué mirar**           | "Los transcripts están en mi folder junto con el modelo."          |
| **Qué quieres de vuelta** | "Actualiza la research note. Flag lo que cambia nuestras assumptions." |
| **Dónde debe ir**       | "Guárdalo en el mismo folder" / "Mándalo por email" / "Subido al Drive" |

> 💡 **No tienes que ingenierear el prompt perfecto.** Cowork hace follow-up questions para lo que dejes fuera.

### Paso 2 — Revisa el plan (y responde preguntas si las hay)

Con base en tu prompt y lo que encontró al escanear, Cowork:

1. **Hace preguntas de clarificación** si le faltan datos críticos (qué approach priorizar, cómo debe verse el output final).
2. **Arma un plan estructurado** con los pasos que va a ejecutar.
3. **Te muestra el plan y espera tu aprobación.**

Tú decides:

- **Aprobar el plan tal cual** y dejar que ejecute.
- **Pedir ajustes** ("agrega un PDF además del PowerPoint", "no toques los archivos originales").
- **Redirigir el approach** completamente.

> ⚠️ **Nota sobre este paso:** el curso a veces lo divide en "paso 2: preguntas" + "paso 3: revisar plan". En la práctica son parte del mismo **momento de alineamiento pre-ejecución** — Cowork puede hacer ambas cosas antes de arrancar.

### Paso 3 — Monitorea o te vas

Un **panel de progreso** te muestra cada paso: qué archivos está leyendo, qué está construyendo, qué subagentes están corriendo en paralelo.

Dos modos de trabajar:

- **Step away** — dejas el task corriendo y vuelves cuando termine. Ideal para tareas largas o multi-etapas.
- **Step in** — escribes en el chat para redirigir si ves algo yendo por mal camino. No tienes que esperar a que termine para corregir.

### Paso 4 — Abre el output

El resultado aterriza donde lo indicaste:

- **Por defecto** → en el folder que Cowork leyó, junto a los archivos originales.
- **Si el prompt pidió otro destino** ("draft como email en Gmail", "guardar en Drive compartido") → ahí aparece.

**Formato real:** si pediste un `.pptx`, recibes un PowerPoint real con charts editables — no una imagen o un PDF estático.

---

## 🔧 Bajo el capó: 5 pilares de ejecución

No necesitas conocer la arquitectura para usar Cowork, pero entenderla te ayuda a:

- Escribir mejores prompts.
- Reconocer cuándo una tarea es un buen fit para Cowork.
- Saber cuándo dividir trabajo en varias tareas.

### Los 5 pilares

| # | Pilar                     | Qué hace                                                              |
| - | ------------------------- | --------------------------------------------------------------------- |
| 1 | **Analyzes request**      | Lee tu prompt y crea un plan estructurado                             |
| 2 | **Breaks into subtasks**  | Divide trabajo complejo en piezas manejables                          |
| 3 | **Executes in VM**        | Corre en un entorno aislado (VM) en tu computadora                    |
| 4 | **Parallelizes**          | Subagentes coordinan múltiples workstreams simultáneos                |
| 5 | **Delivers output**       | Los archivos finalizados aterrizan directamente en tu filesystem      |

> 💎 **Transparency built in.** Cowork expone su razonamiento en cada paso. Puedes jumpear a hacer course-correction o dar dirección adicional en cualquier momento.

### Subagentes en detalle

Cuando una tarea tiene piezas independientes, Cowork **lanza workstreams separados** que corren al mismo tiempo, cada uno con su propio job y su propio contexto fresh.

**Ejemplo del curso:** comparar 4 vendors.

```
┌─────────────────────────────────────────────────────────┐
│  Tarea: "Comparar 4 vendors: pricing, integraciones,    │
│          reviews"                                        │
└───────────────────────┬─────────────────────────────────┘
                        │
         ┌──────────────┼──────────────┬──────────────┐
         ▼              ▼              ▼              ▼
  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │ Subagente │  │ Subagente │  │ Subagente │  │ Subagente │
  │ Vendor 1  │  │ Vendor 2  │  │ Vendor 3  │  │ Vendor 4  │
  │           │  │           │  │           │  │           │
  │ Context:  │  │ Context:  │  │ Context:  │  │ Context:  │
  │ solo V1   │  │ solo V2   │  │ solo V3   │  │ solo V4   │
  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
        │              │              │              │
        └──────────────┴──────┬───────┴──────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  Síntesis en 1 output │
                  │  (comparativa final)  │
                  └───────────────────────┘
```

**Ventajas concretas:**

- **Tareas grandes que no cabrían en una sola conversación** se pueden dividir en piezas que sí caben.
- Cada pieza recibe **atención focalizada** — el análisis del Vendor 3 no se diluye con los detalles del Vendor 1.
- La **velocidad total se reduce** porque los 4 subagentes trabajan en paralelo, no secuencialmente.

---

## 🎛️ Steering: redirigir en tiempo real

Mientras Cowork ejecuta, tienes dos señales de control:

### El panel de progreso (visibilidad)

Muestra en vivo:

- Qué paso del plan está activo.
- Qué archivos está leyendo o modificando.
- Qué subagentes están corriendo.
- Qué outputs parciales ya generó.

Puedes seguirlo de cerca o ignorarlo — Cowork sigue su camino sin que mires.

### El chat (intervención)

Si ves algo yendo por mal camino, **escribe en el chat** para redirigir. No tienes que esperar a que termine la ejecución.

Ejemplos de intervenciones en vivo:

| Situación                                               | Cómo intervenir                                                      |
| ------------------------------------------------------- | -------------------------------------------------------------------- |
| Cowork está usando un archivo que no debería            | "No uses `old-draft.md`, ese ya está obsoleto. Usa `current.md`."    |
| El approach es correcto pero quieres más detalle        | "Expande la sección de business case con ROI proyectado a 3 años."   |
| El output se está desviando del formato esperado        | "Haz una tabla markdown en vez del texto corrido."                   |
| Necesitas abortar y replantearlo                        | "Para. Cambié de opinión — no queremos PowerPoint, haz un Notion."  |

---

## 🛡️ Entorno aislado: qué sí y qué no toca Cowork

Todo el trabajo ocurre en un **entorno virtual aislado (VM)** dentro de tu computadora, separado de tu sistema principal.

### Qué puede tocar

| Recurso                      | Condición                                                         |
| ---------------------------- | ----------------------------------------------------------------- |
| **Archivos del folder**       | Solo los folders que explícitamente compartiste con el Project    |
| **Conectores MCP**           | Solo los que activaste en Customize (Drive, GitHub, Slack, etc.)  |
| **Internet (si aplica)**      | Solo cuando el Skill o Conector lo requiere                       |

### Qué NO puede tocar

- Tu sistema de archivos fuera de los folders compartidos.
- Apps, tools o configuraciones que no estén conectadas.
- Datos de otras sesiones o tareas.

### Deletion gated

> 🛡️ **Cowork pregunta antes de borrar permanentemente cualquier archivo.** Tú apruebas o declinas cada operación destructiva. No hay borrado silencioso.

Esto aplica a:

- `rm` o equivalente sobre archivos existentes.
- Sobre-escritura de archivos con versiones nuevas (pide confirmación si cambia contenido significativo).
- Operaciones sobre conectores externos (borrar issue en GitHub, mover archivo en Drive, etc.).

---

## 🧠 Mindset de revisión

> 📖 **Del curso:** "Treat the result as a draft. Read it the way you'd read a first pass from a capable colleague: good work that's still yours to shape before you send it on."

**Traducido al flujo real:**

| ❌ No pienses así                               | ✅ Piensa así                                                        |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| "Cowork lo hizo, está listo para producción."   | "Cowork lo drafteó. Yo lo valido antes de enviar."                  |
| "Si el output no es perfecto, Cowork falló."    | "Un draft imperfecto es normal — ahora lo afino con mi criterio."   |
| "Le paso el output al stakeholder sin revisar." | "Reviso, ajusto tono/datos/énfasis, y entonces lo mando."           |

**Analogía clave:** Cowork es como un colega junior competente que te entrega una primera pasada. El trabajo es bueno — pero la decisión final, el último toque y la responsabilidad ante tu audiencia siguen siendo tuyas.

---

## 🎬 Walkthrough end-to-end

Así se ve el loop en práctica. La estructura es la misma para cualquier tarea donde jalas inputs de archivos para producir algo finalizado.

### El starting point

Un Project folder con la acumulación típica:

- Notas de meetings
- Un checklist
- Un timeline en spreadsheet
- Emails guardados
- Una matriz de comparación

**Formatos mixtos, organización suelta.** El goal: convertir esto en una presentación lista para leadership.

### Paso 1 — Describe

```
"Revisa todo en este folder y arma una presentación para leadership
sobre el tooling review. Incluye los resultados de evaluación de vendors,
timeline, business case y riesgos abiertos. Output en PowerPoint."
```

### Paso 2 — Revisa el plan

Cowork te muestra algo como:

```
Plan:
1. Leer todos los archivos del folder
2. Sintetizar la propuesta de vendor ganador
3. Armar el business case con costos/beneficios
4. Generar el deck en .pptx
5. Review final del resultado

¿Agregamos algo? (ej: PDF paralelo al PowerPoint)
```

Si quieres algo más (un PDF además del pptx, una nota ejecutiva previa, etc.) **lo dices aquí**.

### Paso 3 — Déjalo correr

Ves el panel si quieres, o te vas a una reunión. El trabajo sigue.

### Paso 4 — Abres el archivo

El deck es un **PowerPoint real**. Los charts son elementos editables — clickeas y ajustas. Es un draft que refinas desde un punto muchísimo más cerca de terminado que armarlo desde cero.

---

## ✍️ Tips de prompting

### Sé específico con el outcome

| ❌ Vago                          | ✅ Específico                                     |
| ------------------------------- | ------------------------------------------------ |
| "Limpia mis archivos."           | "Organiza mi carpeta Downloads por tipo y fecha." |
| "Haz un resumen."                | "Resume los 3 transcripts en una research note de máximo 2 páginas, destacando cambios de assumptions." |
| "Ayúdame con los vendors."       | "Compara los 4 vendors en precio, integraciones y reviews. Output: tabla markdown con recomendación final." |

### Dale las 3 piezas clave

1. **Qué mirar** (archivos, folders, conectores).
2. **Qué quieres de vuelta** (formato, nivel de detalle, criterios).
3. **Dónde debe ir** (mismo folder, email, Drive, Notion).

### No busques el prompt perfecto

Cowork **te preguntará lo que falte**. Empieza con lo que tienes claro y deja que el paso 2 del loop complete el resto.

### Empieza chico

> 📖 **Del curso:** "Begin with tasks that have clear boundaries — organizing a folder, synthesizing a set of documents. Build your intuition for what Cowork handles well, then scale up."

**Tareas ideales para arrancar:**

- Organizar un folder con muchos archivos.
- Sintetizar un set de documentos en un resumen.
- Comparar items (vendors, opciones, drafts) y recomendar.
- Generar un deliverable simple (doc, deck, sheet) desde notas.

**Tareas para después:**

- Pipelines multi-etapa con dependencias complejas.
- Automatizaciones recurrentes con conectores múltiples.
- Workflows multi-agente con orquestación fina.

---

## 🔗 Referencias

- [Introducción a Cowork](./01-introduccion.md) — qué es Cowork + mapa del cookbook
- [Customize](./03-customize/) — las herramientas transversales que le agregas al agente (Skills, Conectores, Plugins)
- [Anthropic Academy — The task loop](https://www.anthropic.com/learn/cowork) — lección oficial del curso
