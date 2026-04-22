# Cómo funciona Claude Cowork

> Documento vivo. Captura la arquitectura real del ecosistema Cowork + Claude para no perderse al construir workflows.

**Última actualización:** 2026-04-21
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es y para qué sirve?](#-qué-es-y-para-qué-sirve)
- [Cómo trabaja](#️-cómo-trabaja)
- [Customize](#-customize)
- [Skills](#-skills)
- [Conectores](#-conectores)
- [Plugins](#-plugins)
- [Projects](#-projects)

---

## 💡 ¿Qué es y para qué sirve?

**Cowork es el módulo de Claude Desktop que te permite automatizar un trabajo con Claude mientras trabajas en otra cosa.**

Su arquitectura es similar al sistema agéntico de Claude Code.

**Básicamente, Cowork es un agente que vive en tu ordenador.** Corre en un entorno virtual y ejecuta trabajos de forma automatizada, sin que tengas que intervenir.

---

## ⚙️ Cómo trabaja

Piénsalo como crear un proyecto:

1. **Planificas** lo que quieres obtener.
2. **Ejecutas** el plan.
3. **Obtienes** el resultado.

Puedes ver el proceso de ejecución y modificarlo en tiempo real si hace falta.

**Cowork es el project manager de tu proyecto.** Distribuye el trabajo en diferentes tareas y las ejecuta en paralelo a través de una orquestación de agentes.

### Elementos de cada tarea

Para ejecutar cada tarea, Cowork combina 3 elementos fundamentales:

- **Programación opcional** — cada tarea puede correr bajo demanda o en una cadencia que tú definas.
- **Entorno aislado** — todo el trabajo ocurre dentro del entorno virtual de Cowork, sin tocar tus archivos originales.
- **Herramientas conectadas** — hacen que cada tarea sea autónoma. El resultado se genera en la ubicación de tu equipo que hayas definido.

---

## 🎨 Customize

**Customize** es la sección de Cowork donde viven las extensiones que potencian y personalizan al agente. Es el área de mayor valor del entorno: todo lo que hace a Cowork útil para tu caso específico se configura aquí.

Customize agrupa tres tipos de extensiones:

- **🧠 Skills** — Habilidades especializadas que el agente adopta para ejecutar tareas dentro de su entorno local.
- **🔌 Conectores** — MCP servers que conectan al agente con aplicaciones y servicios externos.
- **🧩 Plugins** — Kits curados del marketplace que combinan Skills y Conectores MCP bajo un mismo nombre.

Las siguientes secciones profundizan en cada uno.

---

## 🧠 Skills

Un **Skill** es una **habilidad especializada** que se le suministra al agente de Cowork para ejecutar las tareas de un Project dentro de su entorno local. Funciona como una base de conocimiento autocontenida: combina instrucciones en markdown (tipo system prompt), archivos estáticos (HTML, assets, referencias) y/o código ejecutable (Python, JavaScript, bash, shell scripts) que el agente lee, consulta e invoca — todo sin salir al exterior.

### ¿Para qué sirve?

- Le da al agente **una habilidad específica** para abordar un tipo concreto de tarea (ej. crear un `.docx`, diseñar un poster, redactar comunicaciones internas).
- Funciona como **base de conocimiento autocontenida**: combina instrucciones, scripts, plantillas, referencias y assets en una sola unidad.
- **Opera localmente**: el agente ejecuta la habilidad dentro de su entorno virtual, sin necesidad de conectarse a recursos externos.
- **Tiene dos formas de invocación**:
  - **Auto-activación** — cuando el prompt o el contexto coinciden con el trigger declarado en `description`.
  - **Invocación explícita** — escribiendo `/<nombre-del-skill>` en el prompt, opcionalmente con argumentos (ej. `/accessibility-review <Figma URL>`).

### Origen: Built-in y Personal

En el panel **Skills** de Customize los Skills aparecen agrupados en dos categorías:

#### 🛠️ Built-in skills (pre-instaladas por Anthropic)

Skills nativas del entorno Cowork. Vienen incorporadas de fábrica y habilitan capacidades base del sistema. No se descargan — ya están ahí al abrir Cowork.

Las 3 actuales: `schedule`, `setup-cowork`, `context`.

#### 📦 Personal skills (tu colección)

Skills que tú has incorporado a tu entorno. Tienen dos orígenes posibles:

- **Descargadas del marketplace oficial de Anthropic** — eliges cuáles instalar según las necesidades de tus Projects.
- **Creadas por ti** — Skills custom que construyes desde cero.

Ambos casos aparecen bajo la etiqueta "Personal" en la UI y quedan disponibles globalmente para todos tus Projects.

Personal skills actualmente descargadas del marketplace (10):

`algorithmic-art` · `brand-guidelines` · `canvas-design` · `doc-coauthoring` · `internal-comms` · `mcp-builder` · `skill-creator` · `slack-gif-creator` · `theme-factory` · `web-artifacts-builder`

### Anatomía

Un Skill es una carpeta en tu disco. Al instalarlo desde el marketplace queda descargado localmente en:

```
~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/.../skills/<nombre>/
```

#### Lo único obligatorio: `SKILL.md`

Todo Skill tiene un archivo `SKILL.md` en la raíz. Empieza con un frontmatter YAML que Cowork usa para decidir cuándo activarlo y cómo invocarlo:

```yaml
---
name: accessibility-review
description: "Run a WCAG 2.1 AA accessibility audit on a design or page. Trigger with 'audit accessibility', 'check a11y', 'is this accessible?'..."
argument-hint: "<Figma URL, URL, or description>"
license: Proprietary. LICENSE.txt has complete terms
---
```

**Campos del frontmatter:**

| Campo | Obligatorio | Qué hace |
|---|---|---|
| `name` | ✅ | Identificador del Skill. Se convierte también en el nombre del slash command (`/<name>`). |
| `description` | ✅ | Trigger para auto-activación — Cowork matchea esto contra el prompt/contexto para decidir si activar el Skill. |
| `argument-hint` | ❌ | Declara los argumentos que el Skill acepta cuando se invoca como slash command. |
| `license` | ❌ | Metadata de licencia (Anthropic usa "Proprietary" típicamente). |

El resto del archivo es markdown con instrucciones para Claude: overview, quick reference, workflows y ejemplos de comandos. Puede usar placeholders como `$ARGUMENTS` o `$1` para referenciar los argumentos pasados en la invocación.

#### Lo que puede haber alrededor (varía por Skill)

| Carpeta / Archivo | Qué contiene                                          |
| ----------------- | ----------------------------------------------------- |
| `scripts/`        | Python, shell u otros ejecutables que el Skill invoca |
| `templates/`      | Plantillas base para generar output                   |
| `assets/`         | Archivos estáticos (imágenes, fuentes, datos)         |
| `agents/`         | Definiciones de subagentes especializados             |
| `references/`     | Specs, docs adicionales, ejemplos                     |
| `eval-viewer/`    | Herramientas para medir calidad del Skill             |
| Otros `.md`       | Docs temáticos (ej. `FORMS.md`, `REFERENCE.md`)       |
| `LICENSE.txt`     | Licencia (Anthropic usa Proprietary típicamente)      |

#### Rango real de complejidad

Ejemplos de Skills instalados localmente:

- **Mínimo** — `doc-coauthoring` → solo `SKILL.md`
- **Medio** — `algorithmic-art` → `SKILL.md` + `templates/` + `LICENSE.txt`
- **Rico** — `skill-creator` → `SKILL.md` + `agents/` + `assets/` + `eval-viewer/` + `references/` + `scripts/` + `LICENSE.txt`

---

## 🔌 Conectores

Un **Conector** es un **MCP server** (Model Context Protocol) que actúa como puente entre el agente de Cowork y un recurso externo. A través de un conector, Claude puede interactuar con archivos de tu máquina, APIs, aplicaciones en la nube y servicios de terceros, ampliando el alcance del entorno virtual más allá de su propio sandbox.

### ¿Para qué sirve?

- Extender las capacidades del agente permitiéndole leer y escribir sobre sistemas reales.
- Integrar un Project con el stack donde ya vive tu trabajo (filesystem local, Google Drive, GitHub, Slack, bases de datos, APIs propias, etc.).
- Habilitar workflows multi-agente que ejecutan acciones sobre datos y aplicaciones externas, no solo sobre texto.

### Taxonomía

En el panel **Customize → Connectors** los conectores aparecen agrupados en dos secciones que reflejan **dónde corren**:

#### ☁️ Web (remotos)

MCP servers que corren como servicios externos en la nube y exponen aplicaciones online al agente. Son la vía para conectar Cowork con APIs, SaaS y plataformas de terceros. Ejemplos: Canva, Figma, GitHub Integration, Google Calendar, Google Drive, Notion.

#### 🖥️ Desktop (locales)

MCP servers que corren en tu propia máquina. Sirven para ejecutar tareas multi-agénticas sobre archivos y sistemas locales — tu filesystem, herramientas instaladas, procesos propios del entorno. Ejemplos: Filesystem, Spotify (AppleScript), playwright, context7.

#### Badges en la UI

Cada conector puede llevar uno o más badges que indican su origen o tipo:

| Badge | Qué significa |
|---|---|
| `INCLUDED` | Conector pre-instalado por Cowork (sistema). Ej: `Claude in Chrome`. |
| `LOCAL DEV` | Conector definido manualmente en `claude_desktop_config.json` — típicamente para desarrollo local. Ej: `context7`, `github`, `playwright`. |
| `Interactive` | El conector expone tools que modifican estado (no solo lectura). |

#### Por origen (aplica a los Web)

| Modalidad     | Quién los construye      | Cómo se obtienen                                                                               |
| ------------- | ------------------------ | ---------------------------------------------------------------------------------------------- |
| **Oficiales** | Anthropic o sus partners | Se instalan desde el marketplace de Claude                                                     |
| **Custom**    | Tú mismo                 | Los desarrollas cuando no existe un conector oficial para la aplicación que necesitas integrar |

### Cómo se declaran: `.mcp.json`

Los conectores se registran como entradas dentro de un archivo `.mcp.json`. Cada entrada bajo la key `mcpServers` declara un conector con su nombre, tipo y endpoint o comando.

**Conector Web (remoto):**

```json
{
  "mcpServers": {
    "<nombre-conector>": {
      "type": "http",
      "url": "https://<mcp-endpoint>"
    }
  }
}
```

**Conector Desktop (local):**

```json
{
  "mcpServers": {
    "<nombre-conector>": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@provider/server-name"]
    }
  }
}
```

#### ¿Dónde vive el `.mcp.json`?

| Ubicación | Qué conectores contiene |
|---|---|
| `<plugin>/.mcp.json` | Conectores embebidos en un Plugin específico. Se registran al instalar el Plugin. |
| `~/Library/Application Support/Claude/claude_desktop_config.json` → `mcpServers` | Conectores globales de Claude Desktop (aparecen con badge `LOCAL DEV`). |
| `~/Library/Application Support/Claude/Claude Extensions/` | Desktop Extensions del registry oficial (ej: Filesystem, Spotify). Cada extensión tiene su propio folder con su config. |

### Modelo de invocación

```
┌──────────────────────────┐
│  Aplicación / Servicio   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  MCP server (Conector)   │   ← Desktop o Web · Oficial o Custom
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Claude en Cowork        │   ← Invoca los tools del conector
│  (entorno virtual)       │      al ejecutar las tareas del Project
└──────────────────────────┘
```

### Conectores custom

Cuando ningún conector del marketplace cubre la aplicación que necesitas, la vía es construir tu propio **MCP server custom**. Ese server expone los tools específicos que tu caso de uso requiere y, una vez registrado, queda disponible para que Claude los invoque desde el entorno virtual de Cowork al ejecutar las tareas automatizadas del Project.

Esto te permite automatizar flujos contra aplicaciones propietarias, APIs internas de tu empresa o servicios que aún no tienen presencia oficial en el marketplace.

### Tool permissions

Al seleccionar un conector en el panel, Cowork muestra los tools que expone agrupados por categoría. Cada tool puede configurarse con permisos granulares.

**Categorías típicas de tools:**

- **Interactive tools** — tools que ejecutan acciones (crear, modificar, generar).
- **Read-only tools** — tools que solo leen información, sin efectos de escritura.

**Niveles de permiso por tool:**

| Nivel | Icono | Comportamiento |
|---|---|---|
| **Always allow** | ✓ | El conector usa el tool sin pedir aprobación cada vez. |
| **Ask** | ✋ | Cada invocación del tool requiere aprobación explícita del usuario. |
| **Never** | ✗ | El tool queda bloqueado para ese conector. |

A nivel de categoría existe un dropdown (`Always allow` / `Always ask` / `Never`) que aplica el mismo permiso a todas las tools del grupo.

> 🛡️ **Mínimo privilegio.** La granularidad por-tool permite habilitar solo lo estrictamente necesario. Para tareas sensibles (escrituras a producción, acciones destructivas) mantén `Ask`. Para tools read-only de bajo riesgo, `Always allow` reduce fricción sin abrir superficie de ataque.

---

## 🧩 Plugins

Un **Plugin** es un **kit curado** del marketplace de Claude que empaqueta tres tipos de extensiones bajo un mismo nombre: **Skills** (habilidades locales), **Conectores MCP** (bridges externos) y **Subagents** (agentes especializados). Se instala en un solo paso y convierte al agente de Cowork en un **experto para una función específica** — ventas, marketing, productos, finanzas, legal, producción, soporte, etc.

> 📖 **Definición del curso (Anthropic Academy):** "Los plugins agrupan habilidades, conectores y subagentes en un solo paquete. Al instalar un plugin, Claude se convierte en un experto para esa función."

**Fórmula:**

```
Plugin = Skill(s) + Conector(es) MCP + Subagent(s)
       → Claude se convierte en experto de una función
         (curado por Anthropic o partners)
```

### ¿Para qué sirve?

- **Convierte al agente en un especialista de dominio.** Al instalar un Plugin, Claude adopta el conocimiento, las conexiones y los agentes internos necesarios para ejecutar una función entera (ventas, marketing, finanzas, legal, etc.).
- **Agrupa una capacidad completa** — reúne las habilidades locales, las conexiones externas y los subagentes que juntos cubren un dominio de trabajo.
- **Viene curado** — la combinación de Skills + Conectores + Subagents está pensada por Anthropic o sus partners para un caso de uso específico.
- **Instalación en un paso** — en vez de montar cada pieza por separado, un Plugin los trae empaquetados y listos para activar e invocar.

### Origen: Built-in y Personal

Los Plugins se agrupan en dos categorías:

#### 🛠️ Built-in (incluido con Cowork)

Plugin de sistema pre-instalado por Anthropic. No se descarga — ya viene al abrir Cowork. Actualmente hay **1**:

- **`cowork-plugin-management`** — meta-plugin que provee las herramientas para crear, personalizar y administrar el resto de los Plugins. Contiene las skills `cowork-plugin-customizer` (edita plugins existentes) y `create-cowork-plugin` (crea plugins nuevos desde cero).

Es el plugin que habilita las opciones de instalación **"Carpetas propias"** y **"URL de GitHub"** descritas más adelante.

> 👁️ **Este plugin no aparece en el sidebar de Customize.** El panel `Customize → Personal plugins` solo lista los Plugins Personales. `cowork-plugin-management` opera invisible en el background, habilitando las capacidades de edición y creación de plugins desde dentro del agente.

#### 📦 Personal (tu colección)

Plugins que incorporas a tu entorno desde el marketplace oficial, carpetas propias o repositorios de GitHub. Aparecen en tu panel **Customize → Plugins** con toggle de activación por cada uno.

Los 14 Personal plugins disponibles actualmente en el marketplace `knowledge-work-plugins` de Anthropic cubren estos dominios:

`design` · `engineering` · `marketing` · `sales` · `finance` · `legal` · `operations` · `data` · `productivity` · `product-management` · `human-resources` · `brand-voice` · `enterprise-search` · `pdf-viewer`

### Anatomía

| Elemento | Estructura |
|---|---|
| **Plugin** | Tiene un nombre semántico que identifica al kit completo |
| **Skills** | Cada Skill dentro del Plugin es una carpeta con su propio nombre semántico; dentro de esa carpeta vive un archivo markdown con las instrucciones para el agente |
| **Conectores MCP** | Listados en la UI con su ícono (del partner o aplicación) y un toggle de activación individual |

### Activación por Conector

Cada Conector dentro de un Plugin tiene su **propio toggle de activación**. Al instalar un Plugin tú eliges cuáles de los conectores incluidos quieres habilitar, según lo que tus Projects necesiten.

```
┌──────────────────────────────────────────┐
│        PLUGIN: <nombre-semántico>        │
│                                          │
│   Skills/                                │
│   ├── skill-a/                           │
│   │   └── skill-a.md                     │
│   └── skill-b/                           │
│       └── skill-b.md                     │
│                                          │
│   Conectores MCP                         │
│   ├── [icon] Notion       [ON ]          │
│   ├── [icon] GitHub       [OFF]          │
│   └── [icon] Slack        [ON ]          │
└──────────────────────────────────────────┘
```

### Invocación: slash commands

Los Plugins se usan desde el prompt mediante **slash commands**. Cada Skill del Plugin se invoca escribiendo `/<nombre-del-skill>` — donde `<nombre-del-skill>` es literalmente el nombre de la carpeta del Skill.

**Regla directa:**

```
skills/<nombre-skill>/  ──▶  /<nombre-skill>
```

No hay registro manual ni mapeo adicional. Cowork lee la carpeta `skills/` del Plugin y cada subcarpeta queda expuesta automáticamente como un comando disponible. **Correr el slash command equivale a ejecutar el Skill** — son lo mismo.

**Argumentos:** el `SKILL.md` puede declarar en su frontmatter un `argument-hint` que describe qué inputs acepta el comando. Los argumentos se pasan al invocar y se referencian en el body con `$ARGUMENTS` o `$1`:

```
/<nombre-skill> <argumento>
```

**Al correr el comando**, el Skill se ejecuta dentro del contexto del Plugin y puede:

- Usar sus propias instrucciones, scripts, plantillas y referencias.
- Activar los **Conectores MCP** del Plugin.
- Delegar trabajo a los **Subagents** del Plugin.

**Ejemplos conceptuales (plugin de ventas):**

- `/prep-call` — prepara el briefing de una llamada comercial.
- `/deal-summary` — estructura un resumen de acuerdo.
- `/territory-report` — genera un informe de territorio.

**Flujo de invocación:**

```
Usuario escribe /prep-call
    ↓
Cowork busca el comando en los plugins instalados
    ↓
Encuentra commands/prep-call.md en el plugin de ventas
    ↓
Ejecuta el comando, que puede activar:
  - Skills del plugin
  - Conectores MCP (Notion, Salesforce, etc.)
  - Subagents especializados
    ↓
Entrega el resultado al usuario
```

### Estructura en filesystem (detalle técnico)

A nivel de **UI de Cowork**, un Plugin solo expone **Skills** y **Conectores** — es la abstracción visible en el panel Customize. Pero cuando el Plugin se instala en tu máquina, la estructura real en disco es más rica.

**Ubicación de los plugins de Cowork:**

```
~/Library/Application Support/Claude/local-agent-mode-sessions/<user-uuid>/<device-uuid>/rpm/
├── manifest.json              ← índice de plugins instalados (mapea ID → nombre)
└── plugin_<id>/               ← cada plugin tiene un ID tipo hash
```

> ⚠️ **No confundir con `~/.claude/plugins/`**, que es la ubicación de plugins de **Claude Code** (no de Cowork). Son dos sistemas de plugins separados con ubicaciones distintas.

**Estructura interna de un plugin de Cowork:**

```
plugin_<id>/
├── .claude-plugin/
│   └── plugin.json          ← manifest del plugin (name, version, description, author)
├── .mcp.json                 ← config de Conectores MCP (expuesto en UI)
├── CONNECTORS.md             ← docs de los conectores (opcional)
├── README.md                 ← docs del plugin (opcional)
├── skills/                   ← Skills del plugin (expuesto en UI)
│   └── <nombre-skill>/SKILL.md
├── agents/                   ← Subagents (no expuesto en UI, opcional)
│   └── <agent>.md
└── commands/                 ← Slash commands auxiliares (no expuesto en UI, opcional)
    └── <command>.md
```

Solo `plugin.json` es obligatorio. El resto de componentes son opcionales.

**`plugin.json` — manifest del plugin (ejemplo real):**

```json
{
  "name": "design",
  "version": "1.2.0",
  "description": "Accelerate design workflows — critique, design system, UX writing, accessibility audits...",
  "author": { "name": "Anthropic" }
}
```

**`.mcp.json` — config de los Conectores MCP:**

Los Conectores del plugin se declaran aquí. Cada conector se registra bajo `mcpServers` con su nombre + tipo + URL (o comando, para locales):

```json
{
  "mcpServers": {
    "<nombre-conector>": {
      "type": "http",
      "url": "<mcp-endpoint>"
    }
  }
}
```

**Ejemplo real — los 9 conectores del plugin `design`:**

```json
{
  "mcpServers": {
    "figma":           { "type": "http", "url": "https://mcp.figma.com/mcp" },
    "slack":           { "type": "http", "url": "https://mcp.slack.com/mcp" },
    "notion":          { "type": "http", "url": "https://mcp.notion.com/mcp" },
    "linear":          { "type": "http", "url": "https://mcp.linear.app/mcp" },
    "asana":           { "type": "http", "url": "https://mcp.asana.com/v2/mcp" },
    "atlassian":       { "type": "http", "url": "https://mcp.atlassian.com/v1/mcp" },
    "intercom":        { "type": "http", "url": "https://mcp.intercom.com/mcp" },
    "google-calendar": { "type": "http", "url": "https://gcal.mcp.claude.com/mcp" },
    "gmail":           { "type": "http", "url": "https://gmail.mcp.claude.com/mcp" }
  }
}
```

Cada entrada es un MCP server remoto. Los conectores locales usarían `"type": "stdio"` con un comando ejecutable en lugar de `url`.

### Anatomía de un Subagent

Los **Subagents** son agentes secundarios definidos dentro de un Plugin. No aparecen en la UI de Cowork, pero existen en la carpeta `agents/` como archivos markdown. El agente principal (orquestador) les delega trabajo cuando una tarea se beneficia de paralelización o requiere un rol especializado con tools y modelo específicos.

Cada Subagent es un `.md` con **frontmatter YAML + system prompt**:

```yaml
---
name: <nombre-del-subagente>
description: <qué hace — usado por el orquestador para decidir cuándo delegarle>
tools: <lista de tools disponibles, separados por coma>
model: <modelo de Claude — sonnet, opus, haiku>
color: <tag visual>
---

[System prompt del subagente: responsabilidades, formato de output, criterios]
```

**Campos del frontmatter:**

| Campo | Qué hace |
|---|---|
| `name` | Identificador del subagente |
| `description` | Qué hace — el orquestador lo matchea contra la tarea para decidir si delegarle |
| `tools` | Lista de tools disponibles (Read, Grep, WebFetch, Bash, etc.) |
| `model` | Modelo de Claude que correrá el subagente |
| `color` | Tag visual (pa' distinguir en logs o traces de ejecución) |

#### Diferencia clave: Skill vs Subagent

| | Skill | Subagent |
|---|---|---|
| **Qué es** | Habilidad que el agente principal adopta y ejecuta él mismo | Agente independiente al que se le delega trabajo |
| **Corre en** | La sesión del agente principal | Su propia sesión paralela |
| **Tools** | Hereda los del agente principal | Lista propia declarada en frontmatter |
| **Modelo** | El del agente principal | Propio (declarado en frontmatter) |
| **Visibilidad en UI** | Sí, panel Customize | No — solo en filesystem |

### Editabilidad y personalización

Todos los archivos de un Plugin son **texto plano** y Cowork los lee directamente desde la carpeta — **sin compilación ni build step**. Esto convierte a los Plugins en un sistema abiertamente personalizable: puedes modificar cualquiera de sus 3 capas (Skills, Conectores, Subagents) para que coincidan con las necesidades específicas de tu equipo o Project.

| Acción | Cómo hacerlo |
|---|---|
| Cambiar cómo funciona una habilidad | Abres el `SKILL.md` del skill y lo editas |
| Añadir una habilidad nueva | Creas una carpeta en `skills/` con su `SKILL.md` |
| Añadir o modificar un subagente | Creas o editas un `.md` en `agents/` |
| Modificar conectores | Editas `.mcp.json` |
| Agregar un slash command | Creas un `.md` en `commands/` |

> 📖 **Del curso (Anthropic Academy):** "Para cambiar cómo funciona una habilidad, abra su archivo y edítelo. Para añadir una nueva habilidad, añade una carpeta en `skills/`. No hay ningún paso de compilación — Cowork lee la carpeta directamente."

Los Plugins son, en la práctica, **open source dentro de tu máquina**: los descargas del marketplace pero a partir de ese momento son tuyos para ajustar.

### Opciones de instalación y origen

Un Plugin se puede incorporar a Cowork por tres caminos:

| Opción | Descripción | Respaldo |
|---|---|---|
| **Marketplace oficial** | Plugins curados por Anthropic o sus partners. Se instalan con un clic desde el marketplace dentro de Cowork. | ✅ Certificado por Anthropic |
| **Carpetas propias** | Creas tu propio Plugin manualmente en el filesystem (`plugin.json` + `skills/` + `.mcp.json` + lo que necesites). | ⚠️ Bajo tu responsabilidad — Anthropic no se hace responsable |
| **URL de repositorio GitHub** | Apuntas a un repo (público o privado) con la estructura de Plugin. Cowork lo clona e instala. | ⚠️ Bajo tu responsabilidad — no hay certificación oficial |

La opción del **marketplace** es la vía segura — son Plugins mantenidos con garantía por Anthropic. Las otras dos son **más flexibles pero sin respaldo oficial**: puedes crear tu propio Plugin o compartir el de tu equipo vía repositorio, pero Anthropic no asegura calidad, seguridad ni compatibilidad futura.

### Cómo encajan Plugin, Skill, Conector y Subagent

```
Skill          = habilidad local autocontenida
Conector MCP   = bridge externo hacia una aplicación
Subagent       = agente especializado que ejecuta una parte del trabajo
Slash command  = comando que invoca una acción del Plugin desde el prompt
Plugin         = kit curado que combina Skills + Conectores + Subagents
                 y los expone mediante slash commands
                 (capacidad completa en una instalación)
```

---

## 📂 Projects

Un **Project** es el contenedor que agrupa todo lo que Cowork necesita para trabajar en un dominio o iniciativa tuya. Es donde configuras cómo debe comportarse el agente y a qué información tiene acceso.

### Anatomía de un Project

Un Project se compone de **3 elementos fundamentales**:

#### 1. Instructions

Es el **system prompt** del Project. Aquí defines cómo debe comportarse Cowork para este Project específico: tono, restricciones, qué Skills invocar, cómo estructurar el output, dónde dejarlo. Es la capa donde haces referencia a los recursos globales (como los Skills y Conectores) que quieres que Cowork priorice para este Project.

#### 2. Schedule

La lista de **tareas planificadas** del Project. Cada tarea se configura con los siguientes campos (tal como aparecen en el diálogo _Create scheduled task_ de Cowork):

| Campo                 | Qué es                                                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Name**              | Identificador de la tarea (ej. `daily-briefing`)                                                                 |
| **Description**       | Resumen en una línea de lo que hace                                                                              |
| **Prompt**            | Las instrucciones completas que ejecutará la tarea                                                               |
| **Work in a project** | El folder del Project donde corre la tarea (heredado del Project seleccionado, aparece como `from project`)      |
| **Approval mode**     | `Ask for approvals` (default, pausa ante cada tool call) o `Skip all approvals` (ejecuta autónomo sin preguntar) |
| **Model**             | El modelo de Claude (por defecto o específico por tarea)                                                         |
| **Frequency**         | `Manual` (bajo demanda), `Hourly`, `Daily`, `Weekdays` (lunes a viernes) o `Weekly`                              |

Cada tarea queda vinculada al Project y, por extensión, al directorio de tu máquina asociado a ese Project.

> 🛡️ **Approval mode es una decisión de riesgo.** `Ask for approvals` te da control fino pero rompe la promesa de automatización (tienes que estar presente). `Skip all approvals` es el modo headless real — úsalo solo cuando confíes completamente en el prompt + contexto de la tarea.

#### 3. Contexto

La **información base** del Project. Los agentes **leen** este contexto pero **no lo modifican** — es read-only.

El Contexto se alimenta de 3 fuentes externas + 1 memoria interna:

| Fuente                  | Qué es                                                                            |
| ----------------------- | --------------------------------------------------------------------------------- |
| **Carpeta local**       | Una carpeta en tu ordenador con los archivos base del Project                     |
| **Links / URLs**        | Referencias a sitios web que aportan contexto                                     |
| **Google Drive**        | Documentos en la nube                                                             |
| **Memoria del Project** | Un espacio de memoria reservado donde el agente persiste información entre tareas |

### El flujo completo

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

### Contexto ≠ Output

**El contexto no se toca.** Cowork **lee** los archivos de contexto y solo **crea archivos nuevos** en la ubicación de salida que hayas especificado en las Instructions.

Tus documentos base quedan intactos; el Project genera siempre outputs separados, sin riesgo de sobreescribir tu fuente de verdad.

---
