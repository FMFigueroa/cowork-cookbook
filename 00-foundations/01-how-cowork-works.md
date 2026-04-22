# Cómo funciona Claude Cowork

> Documento vivo. Captura la arquitectura real del ecosistema Cowork + Claude para no perderse al construir workflows.

**Última actualización:** 2026-04-22
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

Un Skill es una carpeta en tu disco. Al instalarlo desde el marketplace queda descargado localmente en tu Mac, dentro de la jerarquía que administra la app de Cowork:

```
~/                                              ← tu carpeta personal (Home)
└── Library/                                    ← carpeta del sistema (oculta por defecto)
    └── Application Support/
        └── Claude/                             ← datos de la app Cowork
            └── local-agent-mode-sessions/      ← sesiones locales del agente
                └── skills-plugin/
                    └── <uuid-instalación>/     ← ID único de tu instalación
                        └── <uuid-sesión>/      ← ID único de la sesión activa
                            └── skills/         ← aquí viven todos tus Skills
                                ├── brand-guidelines/
                                ├── canvas-design/
                                ├── doc-coauthoring/
                                ├── mcp-builder/
                                ├── skill-creator/
                                └── <nombre-skill>/   ← una carpeta por Skill
```

> 💡 **¿Por qué no ves `Library` en el Finder?**
> macOS esconde `~/Library` por defecto. Para verla:
> - **Atajo rápido (one-shot):** abre Finder → `Cmd + Shift + .` → se muestran todas las carpetas ocultas (mismo atajo para volver a ocultarlas).
> - **Mostrarla siempre:** ve a tu Home (`Cmd + Shift + H`) → `Cmd + J` → marca *"Show Library Folder"*.

> ⚠️ **Los dos UUIDs son únicos por máquina.** Cowork los genera al instalarse y al abrir cada sesión — no los busques en la documentación oficial, son tuyos. Si reinstalas Cowork, cambian.

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

| Campo            | Obligatorio | Qué hace                                                                                              |
| ---------------- | ----------- | ----------------------------------------------------------------------------------------------------- |
| `name`           | ✅          | Identificador del Skill. Se convierte también en el nombre del slash command (`/<name>`).             |
| `description`    | ✅          | Trigger para auto-activación — Cowork matchea esto contra el prompt/contexto para decidir si activar el Skill. |
| `argument-hint`  | ❌          | Declara los argumentos que el Skill acepta cuando se invoca como slash command.                       |
| `license`        | ❌          | Metadata de licencia (Anthropic usa "Proprietary" típicamente).                                       |

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

### 🚚 Transport: la ley física del MCP

Antes de entrar en tipos, hay una decisión técnica que determina **dónde puede vivir físicamente un conector**: el **transport**, declarado en el campo `type` del `.mcp.json`.

| `type`  | Canal de comunicación                      | Dónde vive el server               |
| ------- | ------------------------------------------ | ---------------------------------- |
| `stdio` | Pipes del OS (stdin/stdout del proceso)    | **Solo en tu máquina**             |
| `http`  | Red HTTPS                                  | **Cualquier parte de internet**    |
| `sse`   | Server-Sent Events (variante HTTP)         | Remoto (streaming de eventos)      |

**Por qué es así (razón física):**

- **STDIO** son pipes del kernel entre procesos del **mismo OS** — no existen "pipes por internet".
- **HTTP** viaja por la red y puede atravesar firewalls, DNS, el internet entero.

```
STDIO (local)                         HTTP (remoto)
┌────────────────────┐                ┌──────────────┐         ┌──────────────┐
│  Claude Desktop    │                │Claude Desktop│────────▶│  MCP server  │
│   ↕ stdin/stdout   │                │ (tu Mac)     │◀────────│  (en la nube)│
│  MCP server        │                └──────────────┘  HTTPS  └──────────────┘
└────────────────────┘
Ambos en la MISMA máquina             Pueden estar en cualquier lugar del mundo
```

> 🧠 **Regla mental:** si el server **vive en tu máquina** → `stdio`. Si **vive en la nube** → `http`. Esta ley amarra la taxonomía que viene ahora.

### Taxonomía oficial del marketplace

En **Browse Connectors** de Claude, Anthropic clasifica los conectores autorizados (oficiales + partners) en **3 tipos**:

#### 🔵 `Interactive`

MCP servers (generalmente remotos) que exponen **tools interactivas** sobre aplicaciones productivas. Son la mayoría de integraciones con SaaS colaborativos.

**Ejemplos:** Monday, Figma, Canva, Slack, Asana.

#### 🖥️ `Desktop`

MCP servers que corren **localmente en tu máquina** empaquetados como *Desktop Extensions* (transport `stdio`). Operan sobre archivos, herramientas instaladas y procesos del entorno local.

**Ejemplos:** Filesystem, PDF Viewer, Control Chrome, Word, PowerPoint, Figma, Kubernetes, Control Your Mac, Desktop Commander, Metabase, AWS API MCP Server, Tableau, Cloudinary, SAP, Shadcn UI, Blockchain Query, Revolut.

#### ☁️ `Web`

MCP servers **remotos** que corren en la nube de Anthropic o de sus partners (transport `http`). Es el ecosistema **más rico y amplio** — cubre aplicaciones web populares del mercado.

**Ejemplos:** Google Drive, Google Calendar, Gmail, Zoom, Monday, HubSpot, Figma, Intercom, Atlassian, Microsoft 365, Notion, Slack, Airtable, SAP, Asana, Fandom, Adobe, Google Cloud BigQuery, n8n, Stripe, Apollo, Hugging Face, Zoho, Wix, Fin.

> ℹ️ **Cross-categoría.** Algunos nombres aparecen en varias categorías (ej: `Figma`, `Slack`, `Monday`). Significa que Anthropic/partners ofrecen **más de una implementación** del mismo servicio, con características distintas.

### Dónde se administran desde la UI

Claude Desktop tiene **dos caminos** para llegar a los conectores. Entender esta separación elimina la confusión más común del ecosistema:

| UI                                         | Qué administra                                                | Quién los crea                        |
| ------------------------------------------ | ------------------------------------------------------------- | ------------------------------------- |
| **Customize** (antes `Settings → Connectors`) | Marketplace oficial completo (`Interactive`, `Desktop`, `Web`) | Anthropic + partners                  |
| **Settings → Desktop App → Developer**     | Solo conectores **locales** (`LocalDev` + `Desktop`)          | Tú (`LocalDev`) o Anthropic (`Desktop`) |

**Por qué los remotos (`Web`, `Interactive`) NO aparecen en `Developer`:** porque corren en la nube, no hay proceso local que monitorear. El panel `Developer` muestra status `Running` y logs en tiempo real — solo tiene sentido para servidores que corren en tu máquina.

### Badges en la UI

Cada conector puede llevar uno o más badges que indican su origen o tipo:

| Badge         | Qué significa                                                                                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `INCLUDED`    | Conector pre-instalado por Cowork (sistema). Ej: `Claude in Chrome`.                                                                                                      |
| `DESKTOP`     | Desktop Extension instalada desde el registry oficial. Vive en `~/Library/Application Support/Claude/Claude Extensions/`. Ej: `Filesystem`.                               |
| `LOCAL DEV`   | Conector definido manualmente en `claude_desktop_config.json` — típicamente para desarrollo local o paquetes no oficiales. Ej: `context7`, `github`, `playwright`.        |
| `Interactive` | El conector expone tools que modifican estado (no solo lectura).                                                                                                          |

---

### 🛠️ LocalDev — el MCP server que declaras tú

Un **LocalDev** es un conector que **tú registras manualmente** editando `claude_desktop_config.json`. No pasa por el marketplace — corre lo que sea que pongas en el `command`.

#### Anatomía del JSON

Viven bajo la key `mcpServers`. Cada entrada declara el conector con transport `stdio` + cómo arrancarlo:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

Si el server necesita secrets (API keys, tokens), se pasan vía `env`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<tu-PAT>"
      }
    }
  }
}
```

#### Dónde viven físicamente

No se instalan como tal — se **descargan on-demand** desde npm al cache de tu Mac:

```
~/.npm/_npx/
├── 3dfbf5a9eea4a1b3/     ← cada hash = un paquete diferente
├── 9833c18b2d85bc59/     ← aquí está @playwright/mcp
└── ...
```

La primera vez que Claude Desktop ejecuta `npx -y @playwright/mcp@latest`, npx **baja el paquete** y lo guarda en `~/.npm/_npx/<hash>/`. En arranques siguientes, si hay cache válido, lo reutiliza.

#### Ciclo de vida completo

```
1. ABRES Claude Desktop
         │
         ▼
2. Claude lee ~/Library/Application Support/Claude/claude_desktop_config.json
         │
         ▼
3. Para cada entrada en "mcpServers", ejecuta el comando:
   ┌──────────────────────────────────────────────┐
   │  npx -y @playwright/mcp@latest               │
   │  ↓                                           │
   │  a) npm descarga el paquete (si no hay cache)│
   │  b) Lo guarda en ~/.npm/_npx/<hash>/         │
   │  c) Lanza el server como PROCESO HIJO        │
   └──────────────────────────────────────────────┘
         │
         ▼
4. El proceso abre un canal STDIO con Claude Desktop
         │
         ▼
5. Claude pregunta: "¿qué tools expones?" → server responde
         │
         ▼
6. EL PROCESO QUEDA VIVO todo el rato que Claude está abierto
   (por eso ves "Running" en Developer + logs en tiempo real)
```

#### Cómo se usan las tools (el flujo invisible)

**Tú NO corres `npx` manualmente cada vez que usas el MCP.** Ya quedó vivo al abrir Claude. Lo que pasa cuando invocas una tool:

```
┌───────────────────────────────────────────────────┐
│  Tú:  "Tómame un screenshot de google.com"        │
└────────────────────┬──────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────────────┐
│  Claude (cerebro):                                │
│  "Necesito la tool browser_screenshot             │
│   del MCP playwright"                             │
└────────────────────┬──────────────────────────────┘
                     │ JSON-RPC por stdio
                     ▼
┌───────────────────────────────────────────────────┐
│  Proceso playwright-mcp (ya corriendo)            │
│  → navega, toma screenshot, devuelve imagen       │
└────────────────────┬──────────────────────────────┘
                     │ respuesta por stdio
                     ▼
┌───────────────────────────────────────────────────┐
│  Claude recibe el resultado y te lo muestra       │
└───────────────────────────────────────────────────┘
```

---

### 📦 Desktop Extension — el paquete Node empaquetado

Un **Desktop Extension** es un conector **instalado oficialmente** desde el marketplace (categoría `Desktop`). Es un paquete Node.js **autocontenido** que vive en disco con su código, sus dependencias y un manifest que declara todo.

#### Árbol de archivos

Cada extension es una carpeta propia dentro de `Claude Extensions/`:

```
~/Library/Application Support/Claude/Claude Extensions/
└── ant.dir.ant.anthropic.filesystem/
    ├── icon.png              ← ícono para la UI
    ├── manifest.json         ← contrato con Claude Desktop
    ├── package.json          ← deps de Node (estándar npm)
    ├── node_modules/         ← deps instaladas (pre-resueltas)
    └── server/
        └── index.js          ← entry point del MCP server
```

#### El archivo clave: `manifest.json`

Es el ADN del extension — declara TODO lo que Claude Desktop necesita saber. Mucho más rico que el entry de `mcpServers` de un LocalDev:

```json
{
  "manifest_version": "0.2",
  "name": "Filesystem",
  "version": "0.2.0",
  "description": "Let Claude access your filesystem to read and write files.",
  "author": { "name": "Anthropic", "url": "https://www.claude.ai" },
  "icon": "icon.png",

  "tools": [
    { "name": "read_file",       "description": "Read the contents of a file" },
    { "name": "write_file",      "description": "Write content to a file" },
    { "name": "edit_file",       "description": "Edit the contents of a file" },
    { "name": "list_directory",  "description": "List contents of a directory" }
    // ... 7 tools más
  ],

  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": [
        "${__dirname}/server/index.js",
        "${user_config.allowed_directories}"
      ]
    }
  },

  "user_config": {
    "allowed_directories": {
      "type": "directory",
      "title": "Allowed Directories",
      "multiple": true,
      "required": true
    }
  },

  "compatibility": {
    "claude_desktop": ">=0.10.0",
    "platforms": ["darwin", "win32", "linux"],
    "runtimes": { "node": ">=16.0.0" }
  }
}
```

#### Qué declara cada sección

| Sección                                     | Para qué sirve                                                                                                         |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `manifest_version`, `name`, `version`       | Metadata de identificación del extension                                                                               |
| `display_name`, `description`, `icon`, `author` | Lo que ves en la UI cuando exploras el extension en el marketplace                                                 |
| `tools[]`                                   | **Declaración estática** de las tools. La UI las lista antes incluso de correr el server.                              |
| `server.type` + `entry_point` + `mcp_config` | Cómo arrancar el proceso (equivalente al `command`/`args` del LocalDev)                                                |
| `user_config`                               | Campos configurables **desde la UI**. Claude Desktop auto-genera un form gráfico (file picker, text input, etc.)       |
| `compatibility`                             | Versión mínima de Claude Desktop, OS soportados, runtime requerido                                                     |

#### Template vars del `mcp_config`

Los placeholders en `args` se resuelven al vuelo antes de ejecutar:

- `${__dirname}` → la ruta absoluta de la carpeta del extension
- `${user_config.<campo>}` → lo que el usuario llenó en la UI

**Resultado final (ejemplo):**

```bash
node "/Users/felix/Library/Application Support/Claude/Claude Extensions/ant.dir.ant.anthropic.filesystem/server/index.js" "/Users/felix/Documents"
```

#### Ciclo de vida

```
1. ABRES Claude Desktop
         │
         ▼
2. Claude escanea ~/Library/Application Support/Claude/Claude Extensions/*/
   y lee cada manifest.json
         │
         ▼
3. Para cada extension activada, resuelve las template vars
   y ejecuta el command:
   ┌──────────────────────────────────────────────┐
   │  node /ruta/absoluta/server/index.js <args>  │
   │  ↓                                           │
   │  a) Node carga node_modules/ LOCALES         │
   │     (no baja nada de internet)               │
   │  b) index.js instancia el MCP server         │
   │  c) Queda vivo comunicándose por STDIO       │
   └──────────────────────────────────────────────┘
         │
         ▼
4. Claude conecta con el proceso y lista sus tools
         │
         ▼
5. EL PROCESO QUEDA VIVO (igual que el LocalDev)
```

> 💡 **Dato clave:** los Desktop Extensions **también son servidores corriendo en tiempo real** en tu máquina, exactamente como los LocalDev. Si vas a `Settings → Desktop App → Developer`, los verás con status `Running` y sus logs en vivo. La diferencia está en cómo llegaron a tu máquina y cómo se configuran, no en cómo ejecutan.

---

### 🆚 LocalDev vs Desktop Extension

| Aspecto                       | LocalDev                                        | Desktop Extension                                  |
| ----------------------------- | ----------------------------------------------- | -------------------------------------------------- |
| **Dónde se declara**          | `claude_desktop_config.json` (1 archivo global) | `Claude Extensions/<nombre>/manifest.json` (1 carpeta por extension) |
| **Código del server**         | Cache de npx (`~/.npm/_npx/<hash>/`)            | Bundle local (`Claude Extensions/<nombre>/server/`) |
| **Instalación**               | Manual: editas el JSON a mano                   | UI: `Settings → Extensions` con botón de install   |
| **Dependencias**              | Resueltas on-demand por npx                     | Pre-instaladas en `node_modules/` del extension    |
| **Internet al arrancar**      | Sí (npx valida/descarga)                        | No (todo local)                                    |
| **Config de usuario**         | Hardcodeada en el JSON (ej: `env` vars)         | UI gráfica con form (`user_config` del manifest)   |
| **Tools declaradas**          | Se descubren al correr el server                | Declaradas estáticamente en `manifest.json`        |
| **Comando típico**            | `npx -y <paquete>`                              | `node server/index.js`                             |
| **Updates**                   | Tú actualizas el `args`/`command`               | Claude Desktop gestiona updates explícitos vía UI  |
| **Aparece en marketplace**    | Nunca (es 100% manual)                          | Sí (si es oficial de Anthropic/partners)           |
| **Badge en UI**               | `LOCAL DEV`                                     | `DESKTOP`                                          |

---

### 🤔 ¿Por qué existen dos modalidades y no una sola?

Si te preguntas por qué no unificaron todo como Desktop Extensions (o todo como LocalDev), la respuesta es que **atacan audiencias y casos de uso complementarios**. Eliminar uno rompería al otro.

#### Contexto histórico

**LocalDev fue primero.** Cuando Anthropic lanzó el protocolo MCP, la única forma de correr un server era manualmente con `claude_desktop_config.json`. Era ideal para devs que ya tenían Node/Python/etc. instalado.

**Desktop Extensions llegaron después** como evolución productizada — para que un usuario no-dev pudiera instalar un MCP sin abrir jamás una terminal.

#### Para quién está pensado cada uno

| Dimensión               | LocalDev                                   | Desktop Extension                 |
| ----------------------- | ------------------------------------------ | --------------------------------- |
| **Audiencia**           | Devs que construyen/experimentan MCPs      | Usuarios finales no-devs          |
| **Filosofía**           | "Dame control total"                       | "Dame simplicidad"                |
| **Setup**               | Editas JSON a mano + CLI                   | Un click desde la UI              |
| **Secrets**             | Env vars en plaintext (PATs, API keys)     | Inputs gestionados con forms      |
| **Validación Anthropic**| Ninguna (corres lo que quieras)            | Revisión oficial antes de publicar|

#### Las diferencias técnicas que justifican la separación

**1. Flexibilidad de runtime**

- **LocalDev** corre **cualquier cosa**: `python`, `node`, `bun`, `deno`, `uv`, `docker run`, un binario compilado en Rust… cualquier comando que tu OS entienda.
- **Desktop Extension** solo corre **Node.js** (según el spec actual del manifest).

**2. Distribución**

- **LocalDev**: no hay registry. Tú descubres paquetes en npm/GitHub y los sumas a mano.
- **Desktop Extension**: marketplace oficial curado con search, categorías, descripciones largas, validación.

**3. Configuración de usuario**

- **LocalDev**: si tu server necesita un API key, se hardcodea en `env` del JSON o esperas que el user lo exporte en su shell.
- **Desktop Extension**: declarativo en el manifest (`user_config`) → Claude Desktop **auto-genera un form gráfico**.

**4. Lifecycle de versión**

- **LocalDev** con `npx -y <paquete>@latest`: cada vez que abres Claude, npx **re-valida contra npm** y puede traer una versión nueva silenciosamente.
- **Desktop Extension**: versión pineada en `package.json`. Updates son explícitos — Claude te notifica en la UI y tú decides.

**5. Seguridad**

- **LocalDev**: tú eres 100% responsable. Puedes correr un MCP malicioso sin saberlo si copiaste el comando de un README cualquiera.
- **Desktop Extension**: Anthropic **revisa el código** antes de publicar en el marketplace.

#### Por qué NO unificar todo

**¿Por qué no todo como Desktop Extension?** Perderías:
- Cualquier runtime (Python, Rust, Go, Bun, Docker)
- MCPs privados/experimentales (no quieres publicarlos al marketplace solo para probarlos)
- Env vars con secrets dinámicos
- Iteración rápida en scripts en desarrollo

**¿Por qué no todo como LocalDev?** Excluirías al 95% de usuarios que:
- No saben qué es una terminal
- No tienen Node instalado
- No entienden `npx`
- Necesitan un marketplace confiable para descubrir qué existe

#### Analogía que cierra el caso

Es el mismo split que tienes en macOS:

| LocalDev                                        | Desktop Extension                        |
| ----------------------------------------------- | ---------------------------------------- |
| Correr un binario con `./mi-programa` en terminal | Instalar una `.app` desde Mac App Store  |
| Full flexibilidad, requiere conocimiento técnico | Un click, sandboxed, validado por Apple  |
| Cualquier lenguaje/runtime                      | Solo formatos aprobados                  |
| Tú eres responsable de la seguridad             | Apple revisa antes de publicar           |

**Ambas coexisten porque atacan necesidades complementarias.** Ni Anthropic ni Apple van a eliminar una — son dos caras de la misma moneda.

```
LocalDev           → "API de bajo nivel": poder + responsabilidad
Desktop Extension  → "API de alto nivel":   simplicidad + constraints
```

---

### ☁️ Web — los MCP servers remotos

Los conectores tipo `Web` (y la mayoría de `Interactive`) son la **otra cara** del ecosistema: corren en la nube de Anthropic o de sus partners, no en tu máquina.

#### Anatomía del JSON

Transport `http`, sin `command` ni `args` — solo un endpoint:

```json
{
  "mcpServers": {
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

No hay proceso que arrancar — Claude Desktop solo hace requests HTTPS al endpoint remoto.

#### Por qué NO aparecen en `Developer`

El panel `Settings → Desktop App → Developer` muestra status `Running` y logs en tiempo real — solo tiene sentido para servidores que corren en tu máquina. Los remotos **no son tu responsabilidad operacional**: si están caídos, es problema del partner. Tú solo consumes el endpoint.

#### Autenticación: OAuth

La mayoría de conectores Web oficiales usan **OAuth** — al instalarlos desde el marketplace, Claude Desktop abre el flow de autorización en tu browser. No pegas API keys en un JSON.

#### Por origen (Oficiales vs Custom)

| Modalidad     | Quién los construye      | Cómo se obtienen                                                                               |
| ------------- | ------------------------ | ---------------------------------------------------------------------------------------------- |
| **Oficiales** | Anthropic o sus partners | Se instalan desde el marketplace de Claude                                                     |
| **Custom**    | Tú mismo                 | Los desarrollas cuando no existe un conector oficial para la aplicación que necesitas integrar |

---

### ¿Dónde vive el `.mcp.json`?

Resumen de todas las ubicaciones donde Claude Desktop busca conectores:

| Badge       | Ubicación                                                                        | Qué conectores contiene                                                                                                    |
| ----------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| —           | `<plugin>/.mcp.json`                                                             | Conectores embebidos en un Plugin específico. Se registran al instalar el Plugin.                                          |
| `LOCAL DEV` | `~/Library/Application Support/Claude/claude_desktop_config.json` → `mcpServers` | Conectores globales definidos manualmente. Ej: `context7`, `github`, `playwright`.                                          |
| `DESKTOP`   | `~/Library/Application Support/Claude/Claude Extensions/`                        | Desktop Extensions del registry oficial. Cada extension tiene su propio folder con su config. Ej: `Filesystem`.             |

---

### Conectores custom

Cuando ningún conector del marketplace cubre la aplicación que necesitas, la vía es construir tu propio **MCP server custom**. Ese server expone los tools específicos que tu caso de uso requiere y, una vez registrado, queda disponible para que Claude los invoque desde el entorno virtual de Cowork al ejecutar las tareas automatizadas del Project.

Esto te permite automatizar flujos contra aplicaciones propietarias, APIs internas de tu empresa o servicios que aún no tienen presencia oficial en el marketplace.

---

### Tool permissions

Al seleccionar un conector en el panel, Cowork muestra los tools que expone agrupados por categoría. Cada tool puede configurarse con permisos granulares.

**Categorías típicas de tools:**

- **Interactive tools** — tools que ejecutan acciones (crear, modificar, generar).
- **Read-only tools** — tools que solo leen información, sin efectos de escritura.

**Niveles de permiso por tool:**

| Nivel            | Icono | Comportamiento                                                       |
| ---------------- | ----- | -------------------------------------------------------------------- |
| **Always allow** | ✓     | El conector usa el tool sin pedir aprobación cada vez.               |
| **Ask**          | ✋    | Cada invocación del tool requiere aprobación explícita del usuario.  |
| **Never**        | ✗     | El tool queda bloqueado para ese conector.                           |

A nivel de categoría existe un dropdown (`Always allow` / `Always ask` / `Never`) que aplica el mismo permiso a todas las tools del grupo.

> 🛡️ **Mínimo privilegio.** La granularidad por-tool permite habilitar solo lo estrictamente necesario. Para tareas sensibles (escrituras a producción, acciones destructivas) mantén `Ask`. Para tools read-only de bajo riesgo, `Always allow` reduce fricción sin abrir superficie de ataque.

---

### Modelo de invocación

```
┌──────────────────────────┐
│  Aplicación / Servicio   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  MCP server (Conector)   │   ← Web (http) · Desktop (stdio) · LocalDev (stdio)
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  Claude en Cowork        │   ← Invoca los tools del conector
│  (entorno virtual)       │      al ejecutar las tareas del Project
└──────────────────────────┘
```

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
