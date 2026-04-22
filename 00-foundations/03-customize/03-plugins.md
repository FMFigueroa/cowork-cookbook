# 🧩 Plugins

> Kits curados del marketplace que combinan Skills + Conectores + Subagents bajo un mismo nombre.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es un Plugin?](#-qué-es-un-plugin)
- [¿Para qué sirve?](#-para-qué-sirve)
- [Origen: Built-in y Personal](#-origen-built-in-y-personal)
- [Anatomía](#-anatomía)
- [Activación por Conector](#-activación-por-conector)
- [Invocación: slash commands](#-invocación-slash-commands)
- [Estructura en filesystem](#-estructura-en-filesystem-detalle-técnico)
- [Anatomía de un Subagent](#-anatomía-de-un-subagent)
- [Editabilidad y personalización](#-editabilidad-y-personalización)
- [Opciones de instalación y origen](#-opciones-de-instalación-y-origen)
- [Cómo encajan Plugin, Skill, Conector y Subagent](#-cómo-encajan-plugin-skill-conector-y-subagent)

---

## 💡 ¿Qué es un Plugin?

Un **Plugin** es un **kit curado** del marketplace de Claude que empaqueta tres tipos de extensiones bajo un mismo nombre: **Skills** (habilidades locales), **Conectores MCP** (bridges externos) y **Subagents** (agentes especializados). Se instala en un solo paso y convierte al agente de Cowork en un **experto para una función específica** — ventas, marketing, productos, finanzas, legal, producción, soporte, etc.

> 📖 **Definición del curso (Anthropic Academy):** "Los plugins agrupan habilidades, conectores y subagentes en un solo paquete. Al instalar un plugin, Claude se convierte en un experto para esa función."

**Fórmula:**

```
Plugin = Skill(s) + Conector(es) MCP + Subagent(s)
       → Claude se convierte en experto de una función
         (curado por Anthropic o partners)
```

## 🎯 ¿Para qué sirve?

- **Convierte al agente en un especialista de dominio.** Al instalar un Plugin, Claude adopta el conocimiento, las conexiones y los agentes internos necesarios para ejecutar una función entera (ventas, marketing, finanzas, legal, etc.).
- **Agrupa una capacidad completa** — reúne las habilidades locales, las conexiones externas y los subagentes que juntos cubren un dominio de trabajo.
- **Viene curado** — la combinación de Skills + Conectores + Subagents está pensada por Anthropic o sus partners para un caso de uso específico.
- **Instalación en un paso** — en vez de montar cada pieza por separado, un Plugin los trae empaquetados y listos para activar e invocar.

## 📚 Origen: Built-in y Personal

Los Plugins se agrupan en dos categorías:

### 🛠️ Built-in (incluido con Cowork)

Plugin de sistema pre-instalado por Anthropic. No se descarga — ya viene al abrir Cowork. Actualmente hay **1**:

- **`cowork-plugin-management`** — meta-plugin que provee las herramientas para crear, personalizar y administrar el resto de los Plugins. Contiene las skills `cowork-plugin-customizer` (edita plugins existentes) y `create-cowork-plugin` (crea plugins nuevos desde cero).

Es el plugin que habilita las opciones de instalación **"Carpetas propias"** y **"URL de GitHub"** descritas más adelante.

> 👁️ **Este plugin no aparece en el sidebar de Customize.** El panel `Customize → Personal plugins` solo lista los Plugins Personales. `cowork-plugin-management` opera invisible en el background, habilitando las capacidades de edición y creación de plugins desde dentro del agente.

### 📦 Personal (tu colección)

Plugins que incorporas a tu entorno desde el marketplace oficial, carpetas propias o repositorios de GitHub. Aparecen en tu panel **Customize → Plugins** con toggle de activación por cada uno.

Los 14 Personal plugins disponibles actualmente en el marketplace `knowledge-work-plugins` de Anthropic cubren estos dominios:

`design` · `engineering` · `marketing` · `sales` · `finance` · `legal` · `operations` · `data` · `productivity` · `product-management` · `human-resources` · `brand-voice` · `enterprise-search` · `pdf-viewer`

## 🔍 Anatomía

| Elemento | Estructura |
|---|---|
| **Plugin** | Tiene un nombre semántico que identifica al kit completo |
| **Skills** | Cada Skill dentro del Plugin es una carpeta con su propio nombre semántico; dentro de esa carpeta vive un archivo markdown con las instrucciones para el agente |
| **Conectores MCP** | Listados en la UI con su ícono (del partner o aplicación) y un toggle de activación individual |

## 🎛️ Activación por Conector

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

## ⚡ Invocación: slash commands

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

## 🗂️ Estructura en filesystem (detalle técnico)

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

## 🤖 Anatomía de un Subagent

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

### Diferencia clave: Skill vs Subagent

| | Skill | Subagent |
|---|---|---|
| **Qué es** | Habilidad que el agente principal adopta y ejecuta él mismo | Agente independiente al que se le delega trabajo |
| **Corre en** | La sesión del agente principal | Su propia sesión paralela |
| **Tools** | Hereda los del agente principal | Lista propia declarada en frontmatter |
| **Modelo** | El del agente principal | Propio (declarado en frontmatter) |
| **Visibilidad en UI** | Sí, panel Customize | No — solo en filesystem |

## 🛠️ Editabilidad y personalización

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

## 📥 Opciones de instalación y origen

Un Plugin se puede incorporar a Cowork por tres caminos:

| Opción | Descripción | Respaldo |
|---|---|---|
| **Marketplace oficial** | Plugins curados por Anthropic o sus partners. Se instalan con un clic desde el marketplace dentro de Cowork. | ✅ Certificado por Anthropic |
| **Carpetas propias** | Creas tu propio Plugin manualmente en el filesystem (`plugin.json` + `skills/` + `.mcp.json` + lo que necesites). | ⚠️ Bajo tu responsabilidad — Anthropic no se hace responsable |
| **URL de repositorio GitHub** | Apuntas a un repo (público o privado) con la estructura de Plugin. Cowork lo clona e instala. | ⚠️ Bajo tu responsabilidad — no hay certificación oficial |

La opción del **marketplace** es la vía segura — son Plugins mantenidos con garantía por Anthropic. Las otras dos son **más flexibles pero sin respaldo oficial**: puedes crear tu propio Plugin o compartir el de tu equipo vía repositorio, pero Anthropic no asegura calidad, seguridad ni compatibilidad futura.

## 🧩 Cómo encajan Plugin, Skill, Conector y Subagent

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

## 🔗 Referencias

- [Overview de Customize](./README.md) — los 3 tipos de extensiones y por qué vienen en "segundo plano"
- [Skills](./01-skills.md) — habilidades locales (uno de los componentes de un Plugin)
- [Conectores](./02-conectores.md) — bridges MCP (otro componente de un Plugin)
