# 🔌 Conectores

> MCP servers que conectan al agente con aplicaciones y servicios externos.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es un Conector?](#-qué-es-un-conector)
- [¿Para qué sirve?](#-para-qué-sirve)
- [Transport: la ley física del MCP](#-transport-la-ley-física-del-mcp)
- [Taxonomía oficial del marketplace](#-taxonomía-oficial-del-marketplace)
- [Dónde se administran desde la UI](#-dónde-se-administran-desde-la-ui)
- [Badges en la UI](#-badges-en-la-ui)
- [LocalDev — el MCP server que declaras tú](#️-localdev--el-mcp-server-que-declaras-tú)
- [Desktop Extension — el paquete Node empaquetado](#-desktop-extension--el-paquete-node-empaquetado)
- [LocalDev vs Desktop Extension](#-localdev-vs-desktop-extension)
- [¿Por qué existen dos modalidades y no una sola?](#-por-qué-existen-dos-modalidades-y-no-una-sola)
- [Web — los MCP servers remotos](#️-web--los-mcp-servers-remotos)
- [¿Dónde vive el `.mcp.json`?](#-dónde-vive-el-mcpjson)
- [Conectores custom](#-conectores-custom)
- [Tool permissions](#-tool-permissions)
- [Modelo de invocación](#-modelo-de-invocación)

---

## 💡 ¿Qué es un Conector?

Un **Conector** es un **MCP server** (Model Context Protocol) que actúa como puente entre el agente de Cowork y un recurso externo. A través de un conector, Claude puede interactuar con archivos de tu máquina, APIs, aplicaciones en la nube y servicios de terceros, ampliando el alcance del entorno virtual más allá de su propio sandbox.

## 🎯 ¿Para qué sirve?

- Extender las capacidades del agente permitiéndole leer y escribir sobre sistemas reales.
- Integrar un Project con el stack donde ya vive tu trabajo (filesystem local, Google Drive, GitHub, Slack, bases de datos, APIs propias, etc.).
- Habilitar workflows multi-agente que ejecutan acciones sobre datos y aplicaciones externas, no solo sobre texto.

## 🚚 Transport: la ley física del MCP

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

## 🏷️ Taxonomía oficial del marketplace

En **Browse Connectors** de Claude, Anthropic clasifica los conectores autorizados (oficiales + partners) en **3 tipos**:

### 🔵 `Interactive`

MCP servers (generalmente remotos) que exponen **tools interactivas** sobre aplicaciones productivas. Son la mayoría de integraciones con SaaS colaborativos.

**Ejemplos:** Monday, Figma, Canva, Slack, Asana.

### 🖥️ `Desktop`

MCP servers que corren **localmente en tu máquina** empaquetados como *Desktop Extensions* (transport `stdio`). Operan sobre archivos, herramientas instaladas y procesos del entorno local.

**Ejemplos:** Filesystem, PDF Viewer, Control Chrome, Word, PowerPoint, Figma, Kubernetes, Control Your Mac, Desktop Commander, Metabase, AWS API MCP Server, Tableau, Cloudinary, SAP, Shadcn UI, Blockchain Query, Revolut.

### ☁️ `Web`

MCP servers **remotos** que corren en la nube de Anthropic o de sus partners (transport `http`). Es el ecosistema **más rico y amplio** — cubre aplicaciones web populares del mercado.

**Ejemplos:** Google Drive, Google Calendar, Gmail, Zoom, Monday, HubSpot, Figma, Intercom, Atlassian, Microsoft 365, Notion, Slack, Airtable, SAP, Asana, Fandom, Adobe, Google Cloud BigQuery, n8n, Stripe, Apollo, Hugging Face, Zoho, Wix, Fin.

> ℹ️ **Cross-categoría.** Algunos nombres aparecen en varias categorías (ej: `Figma`, `Slack`, `Monday`). Significa que Anthropic/partners ofrecen **más de una implementación** del mismo servicio, con características distintas.

## 🖥️ Dónde se administran desde la UI

Claude Desktop tiene **dos caminos** para llegar a los conectores. Entender esta separación elimina la confusión más común del ecosistema:

| UI                                         | Qué administra                                                | Quién los crea                        |
| ------------------------------------------ | ------------------------------------------------------------- | ------------------------------------- |
| **Customize** (antes `Settings → Connectors`) | Marketplace oficial completo (`Interactive`, `Desktop`, `Web`) | Anthropic + partners                  |
| **Settings → Desktop App → Developer**     | Solo conectores **locales** (`LocalDev` + `Desktop`)          | Tú (`LocalDev`) o Anthropic (`Desktop`) |

**Por qué los remotos (`Web`, `Interactive`) NO aparecen en `Developer`:** porque corren en la nube, no hay proceso local que monitorear. El panel `Developer` muestra status `Running` y logs en tiempo real — solo tiene sentido para servidores que corren en tu máquina.

## 🏷️ Badges en la UI

Cada conector puede llevar uno o más badges que indican su origen o tipo:

| Badge         | Qué significa                                                                                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `INCLUDED`    | Conector pre-instalado por Cowork (sistema). Ej: `Claude in Chrome`.                                                                                                      |
| `DESKTOP`     | Desktop Extension instalada desde el registry oficial. Vive en `~/Library/Application Support/Claude/Claude Extensions/`. Ej: `Filesystem`.                               |
| `LOCAL DEV`   | Conector definido manualmente en `claude_desktop_config.json` — típicamente para desarrollo local o paquetes no oficiales. Ej: `context7`, `github`, `playwright`.        |
| `Interactive` | El conector expone tools que modifican estado (no solo lectura).                                                                                                          |

---

## 🛠️ LocalDev — el MCP server que declaras tú

Un **LocalDev** es un conector que **tú registras manualmente** editando `claude_desktop_config.json`. No pasa por el marketplace — corre lo que sea que pongas en el `command`.

### Anatomía del JSON

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

> 🛡️ **Secrets en producción.** Meter tokens directo en el JSON es cómodo para probar, pero es un anti-patrón: queda en plaintext, lo leen agentes de IA que acceden al archivo, y rotarlo es fricción. Para uso real, seguí el patrón [MCP Secrets Management](../../01-patterns/mcp-secrets-management.md) — wrapper script + archivo `.env` con permisos restrictivos.

### Dónde viven físicamente

No se instalan como tal — se **descargan on-demand** desde npm al cache de tu Mac:

```
~/.npm/_npx/
├── 3dfbf5a9eea4a1b3/     ← cada hash = un paquete diferente
├── 9833c18b2d85bc59/     ← aquí está @playwright/mcp
└── ...
```

La primera vez que Claude Desktop ejecuta `npx -y @playwright/mcp@latest`, npx **baja el paquete** y lo guarda en `~/.npm/_npx/<hash>/`. En arranques siguientes, si hay cache válido, lo reutiliza.

### Ciclo de vida completo

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

### Cómo se usan las tools (el flujo invisible)

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

## 📦 Desktop Extension — el paquete Node empaquetado

Un **Desktop Extension** es un conector **instalado oficialmente** desde el marketplace (categoría `Desktop`). Es un paquete Node.js **autocontenido** que vive en disco con su código, sus dependencias y un manifest que declara todo.

### Árbol de archivos

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

### El archivo clave: `manifest.json`

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

### Qué declara cada sección

| Sección                                     | Para qué sirve                                                                                                         |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `manifest_version`, `name`, `version`       | Metadata de identificación del extension                                                                               |
| `display_name`, `description`, `icon`, `author` | Lo que ves en la UI cuando exploras el extension en el marketplace                                                 |
| `tools[]`                                   | **Declaración estática** de las tools. La UI las lista antes incluso de correr el server.                              |
| `server.type` + `entry_point` + `mcp_config` | Cómo arrancar el proceso (equivalente al `command`/`args` del LocalDev)                                                |
| `user_config`                               | Campos configurables **desde la UI**. Claude Desktop auto-genera un form gráfico (file picker, text input, etc.)       |
| `compatibility`                             | Versión mínima de Claude Desktop, OS soportados, runtime requerido                                                     |

### Template vars del `mcp_config`

Los placeholders en `args` se resuelven al vuelo antes de ejecutar:

- `${__dirname}` → la ruta absoluta de la carpeta del extension
- `${user_config.<campo>}` → lo que el usuario llenó en la UI

**Resultado final (ejemplo):**

```bash
node "/Users/felix/Library/Application Support/Claude/Claude Extensions/ant.dir.ant.anthropic.filesystem/server/index.js" "/Users/felix/Documents"
```

### Ciclo de vida

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

## 🆚 LocalDev vs Desktop Extension

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

## 🤔 ¿Por qué existen dos modalidades y no una sola?

Si te preguntas por qué no unificaron todo como Desktop Extensions (o todo como LocalDev), la respuesta es que **atacan audiencias y casos de uso complementarios**. Eliminar uno rompería al otro.

### Contexto histórico

**LocalDev fue primero.** Cuando Anthropic lanzó el protocolo MCP, la única forma de correr un server era manualmente con `claude_desktop_config.json`. Era ideal para devs que ya tenían Node/Python/etc. instalado.

**Desktop Extensions llegaron después** como evolución productizada — para que un usuario no-dev pudiera instalar un MCP sin abrir jamás una terminal.

### Para quién está pensado cada uno

| Dimensión               | LocalDev                                   | Desktop Extension                 |
| ----------------------- | ------------------------------------------ | --------------------------------- |
| **Audiencia**           | Devs que construyen/experimentan MCPs      | Usuarios finales no-devs          |
| **Filosofía**           | "Dame control total"                       | "Dame simplicidad"                |
| **Setup**               | Editas JSON a mano + CLI                   | Un click desde la UI              |
| **Secrets**             | Env vars en plaintext (PATs, API keys)     | Inputs gestionados con forms      |
| **Validación Anthropic**| Ninguna (corres lo que quieras)            | Revisión oficial antes de publicar|

### Las diferencias técnicas que justifican la separación

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

### Por qué NO unificar todo

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

### Analogía que cierra el caso

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

## ☁️ Web — los MCP servers remotos

Los conectores tipo `Web` (y la mayoría de `Interactive`) son la **otra cara** del ecosistema: corren en la nube de Anthropic o de sus partners, no en tu máquina.

### Anatomía del JSON

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

### Por qué NO aparecen en `Developer`

El panel `Settings → Desktop App → Developer` muestra status `Running` y logs en tiempo real — solo tiene sentido para servidores que corren en tu máquina. Los remotos **no son tu responsabilidad operacional**: si están caídos, es problema del partner. Tú solo consumes el endpoint.

### Autenticación: OAuth

La mayoría de conectores Web oficiales usan **OAuth** — al instalarlos desde el marketplace, Claude Desktop abre el flow de autorización en tu browser. No pegas API keys en un JSON.

### Por origen (Oficiales vs Custom)

| Modalidad     | Quién los construye      | Cómo se obtienen                                                                               |
| ------------- | ------------------------ | ---------------------------------------------------------------------------------------------- |
| **Oficiales** | Anthropic o sus partners | Se instalan desde el marketplace de Claude                                                     |
| **Custom**    | Tú mismo                 | Los desarrollas cuando no existe un conector oficial para la aplicación que necesitas integrar |

---

## 📍 ¿Dónde vive el `.mcp.json`?

Resumen de todas las ubicaciones donde Claude Desktop busca conectores:

| Badge       | Ubicación                                                                        | Qué conectores contiene                                                                                                    |
| ----------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| —           | `<plugin>/.mcp.json`                                                             | Conectores embebidos en un Plugin específico. Se registran al instalar el Plugin.                                          |
| `LOCAL DEV` | `~/Library/Application Support/Claude/claude_desktop_config.json` → `mcpServers` | Conectores globales definidos manualmente. Ej: `context7`, `github`, `playwright`.                                          |
| `DESKTOP`   | `~/Library/Application Support/Claude/Claude Extensions/`                        | Desktop Extensions del registry oficial. Cada extension tiene su propio folder con su config. Ej: `Filesystem`.             |

---

## 🧩 Conectores custom

Cuando ningún conector del marketplace cubre la aplicación que necesitas, la vía es construir tu propio **MCP server custom**. Ese server expone los tools específicos que tu caso de uso requiere y, una vez registrado, queda disponible para que Claude los invoque desde el entorno virtual de Cowork al ejecutar las tareas automatizadas del Project.

Esto te permite automatizar flujos contra aplicaciones propietarias, APIs internas de tu empresa o servicios que aún no tienen presencia oficial en el marketplace.

---

## 🔐 Tool permissions

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

## 🔗 Modelo de invocación

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

## 🔗 Referencias

- [Overview de Customize](./README.md) — los 3 tipos de extensiones y por qué vienen en "segundo plano"
- [Skills](./01-skills.md) — el tipo anterior de extensión
- [Plugins](./03-plugins.md) — kits curados que combinan Skills + Conectores + Subagents
- [MCP Secrets Management](../../01-patterns/mcp-secrets-management.md) — patrón para manejar tokens sin exponerlos
