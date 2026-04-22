# Pattern: MCP Secrets Management

> Cómo gestionar tokens y credenciales de MCP servers locales sin exponerlos en los archivos de configuración de los clientes MCP (Claude Desktop, Claude Code y otros).

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)
**Complejidad:** ⭐⭐

---

## 📋 Índice

- [Cuándo usarlo](#-cuándo-usarlo)
- [Clientes MCP soportados](#-clientes-mcp-soportados)
- [El problema](#-el-problema)
- [La solución: wrapper + archivo .env](#-la-solución-wrapper--archivo-env)
- [Estructura](#-estructura)
- [Implementación paso a paso](#-implementación-paso-a-paso)
- [Flujo de rotación de secrets](#-flujo-de-rotación-de-secrets)
- [Anti-patrones](#-anti-patrones)
- [Extensiones](#-extensiones)

---

## 🎯 Cuándo usarlo

Usa este pattern si:

- Tu MCP server necesita **tokens, API keys o secrets** (GitHub PAT, API keys de servicios propios, credenciales de bases de datos).
- Quieres evitar que los secrets **queden en plaintext** en los archivos de configuración de tus clientes MCP.
- Usás **más de un cliente MCP** (ej: Claude Desktop + Claude Code) y no querés duplicar tokens en cada archivo.
- Necesitas un flujo limpio para **rotar secrets** sin tener que tocar múltiples configs cada vez.
- Trabajás con asistentes de IA que podrían leer los archivos de config y dejar los tokens expuestos en logs o contextos de conversación.

---

## 🖥️ Clientes MCP soportados

Este pattern **funciona con cualquier cliente MCP** porque el protocolo es agnóstico. Hoy cubre los clientes de Anthropic, pero la misma infraestructura (wrappers + `~/.mcp/secrets.env`) sirve para Cursor, Zed y cualquier otro cliente que aparezca.

| Cliente MCP                                   | Archivo de configuración                                             | Estructura en el JSON                         |
| --------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------- |
| **Claude Desktop** (app de escritorio)        | `~/Library/Application Support/Claude/claude_desktop_config.json`    | `mcpServers.<nombre>.command` + `args`        |
| **Claude Code** (CLI / agente)                | `~/.claude.json`                                                     | `mcpServers.<nombre>.command` + `args` + `env`|
| **Cursor**, **Zed**, **VSCode extensions**... | (cada uno con su config propia)                                      | Similar: `command` + `args` a un ejecutable   |

> ⚠️ **Si tenés varios clientes instalados, hay que actualizar cada archivo por separado.** Rotar un token requiere cambiar solo `~/.mcp/secrets.env` (una sola vez), pero inicialmente **todos los archivos de config de todos los clientes deben apuntar al mismo wrapper**.

---

## ⚠️ El problema

El formato estándar de registro de un MCP server local expone los secrets directamente en el JSON. Ambos clientes sufren el mismo problema:

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

**Claude Code** (`~/.claude.json`):

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

Si tenés ambos clientes y el mismo MCP server, el token termina **duplicado** en dos archivos distintos. Rotar = editar los dos. Olvidar uno = errores de autenticación silenciosos.

### Por qué esto es problemático

| Riesgo                          | Impacto                                                                                                        |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Lectura por agentes de IA**   | Cualquier asistente que lea los archivos de config arrastra el token a su context                              |
| **Backups accidentales**        | Tools de sync (iCloud, Time Machine) copian los archivos con los tokens adentro                                |
| **Logs y debugging**            | `cat` del archivo en terminal queda en el scrollback o en logs de sesión                                       |
| **Rotación inconveniente**      | Rotar implica editar cada archivo de config de cada cliente (fricción → postergás la rotación)                 |
| **Desincronización silenciosa** | Rotás en un cliente pero olvidás otro → errores intermitentes difíciles de debuggear                           |
| **Versionado imposible**        | No podés versionar los archivos en Git con tokens adentro                                                      |

### Limitación técnica

> **Los clientes MCP actuales NO soportan expansión de variables de entorno en sus archivos de configuración.**
> Si escribís `"${GITHUB_TOKEN}"` en el `env`, lo reciben como **string literal**, no resuelven la variable. Por eso necesitás un **wrapper script** que cargue el `.env` y luego ejecute el comando real — la misma solución sirve para todos los clientes.

---

## 💡 La solución: wrapper + archivo .env

Tres capas que separan las responsabilidades. **El wrapper es compartido entre todos los clientes MCP** — cada cliente apunta al mismo script:

```
┌─────────────────────────────────────────────────────────┐
│  Clientes MCP (todos apuntando al mismo wrapper):      │
│  • Claude Desktop → claude_desktop_config.json         │
│  • Claude Code    → ~/.claude.json                     │  ← ninguno tiene secrets
│  • (Cursor, Zed, ...)                                  │
└─────────────────┬───────────────────────────────────────┘
                  │ ejecutan
                  ▼
┌─────────────────────────────────────────────────────────┐
│  ~/.local/bin/mcp-<X>-wrapper.sh                        │
│  ├─ source ~/.mcp/secrets.env                           │  ← carga secrets
│  ├─ export <VAR>="$TOKEN_DEL_ENV"                       │  ← mapea al nombre correcto
│  └─ exec npx -y @provider/mcp-server                    │  ← arranca el server
└─────────────────┬───────────────────────────────────────┘
                  │ reemplaza el proceso con
                  ▼
┌─────────────────────────────────────────────────────────┐
│  MCP server corriendo con el token en su env           │
└─────────────────────────────────────────────────────────┘
```

### Por qué funciona

- **Los archivos de config de cada cliente** solo conocen la ruta del wrapper. Sin secrets.
- **El wrapper** es el único lugar que toca el `.env` y monta los env vars en el proceso.
- **`exec`** reemplaza el proceso de bash con el MCP server, preservando el canal STDIO intacto — ningún cliente se entera de la capa intermedia.
- **Una sola fuente de verdad** (`~/.mcp/secrets.env`) sirve para todos los clientes. Rotar = editar un solo archivo.

---

## 🏗️ Estructura

```
~/
├── .mcp/                                         ← carpeta dedicada a MCP (chmod 700)
│   └── secrets.env                               ← secrets (chmod 600)
│
├── .local/bin/
│   ├── mcp-github-wrapper.sh                     ← wrapper (chmod 755)
│   ├── mcp-github-leonobitech-wrapper.sh         ← 1 wrapper por MCP con secrets
│   └── mcp-<otro>-wrapper.sh
│
├── .claude.json                                  ← Claude Code (CLI)
│                                                   └─ mcpServers.<X>.command apunta al wrapper
│
└── Library/Application Support/Claude/
    ├── claude_desktop_config.json                ← Claude Desktop (app)
    │                                               └─ mcpServers.<X>.command apunta al wrapper
    └── claude_desktop_config.json.backup-YYYY-MM-DD  ← backup antes del cambio
```

> 💡 **Por qué `~/.mcp/` y no `~/.mcp-secrets.env`** — MCP es un protocolo agnóstico al cliente (Claude Desktop, Cursor, Zed, futuros clientes). Una carpeta dedicada sigue la convención Unix (`~/.ssh/`, `~/.aws/`, `~/.gnupg/`) y deja espacio para futuros archivos compartidos (ej: `~/.mcp/trusted-hosts.yaml`).

---

## 🔧 Implementación paso a paso

### 1. Hacer backup de los archivos de config

Antes de tocar nada, siempre backup. **Hacé backup de todos los clientes que vayas a modificar.**

**Claude Desktop:**
```bash
cp "$HOME/Library/Application Support/Claude/claude_desktop_config.json" \
   "$HOME/Library/Application Support/Claude/claude_desktop_config.json.backup-$(date +%Y-%m-%d)"
```

**Claude Code:**
```bash
cp "$HOME/.claude.json" "$HOME/.claude.json.backup-$(date +%Y-%m-%d)"
```

### 2. Crear `~/.mcp/secrets.env`

Con **placeholders**, nunca con valores reales pegados desde chat. El archivo se edita directamente en tu IDE.

```bash
# ~/.mcp/secrets.env
#
# ⚠️  NUNCA committear este archivo ni pegar su contenido en chat.
# ⚠️  Editar directamente desde tu IDE.

# GitHub — PAT principal
export GITHUB_PAT_MAIN="PEGA_TU_TOKEN_AQUI_MAIN"

# GitHub — PAT secundario
export GITHUB_PAT_LEONOBITECH="PEGA_TU_TOKEN_AQUI_LEONOBITECH"

# Otro servicio
export ANOTHER_API_KEY="PEGA_TU_KEY_AQUI"
```

Permisos restrictivos (solo tu usuario puede leer):

```bash
chmod 600 ~/.mcp/secrets.env
```

### 3. Crear el wrapper

Un wrapper por MCP server que necesite secrets. **El mismo wrapper sirve para todos los clientes** (Claude Desktop, Claude Code, etc.) — no hay que duplicarlo. Template:

```bash
#!/bin/bash
# MCP <nombre> wrapper — propósito
#
# Consumido por cualquier cliente MCP que apunte a este script como command.
# Ejemplos: claude_desktop_config.json, ~/.claude.json, config de Cursor, etc.
# El cliente lo ejecuta como proceso hijo al arrancar.

set -euo pipefail

# Asegurar que npx/node estén disponibles (los clientes MCP arrancan con PATH limpio)
export PATH="$HOME/.local/bin:$PATH"

# Cargar secrets
if [[ ! -f "$HOME/.mcp/secrets.env" ]]; then
  echo "Error: $HOME/.mcp/secrets.env no existe" >&2
  exit 1
fi
source "$HOME/.mcp/secrets.env"

# Mapear al nombre de env var que espera el MCP server
export GITHUB_PERSONAL_ACCESS_TOKEN="$GITHUB_PAT_MAIN"

# Reemplazar este proceso con el MCP server (mantiene STDIO intacto)
exec npx -y @modelcontextprotocol/server-github
```

Permisos ejecutables:

```bash
chmod 755 ~/.local/bin/mcp-github-wrapper.sh
```

### 4. Actualizar los archivos de config de cada cliente

Reemplazar la entrada del MCP server para apuntar al wrapper. **Tenés que hacerlo en cada cliente donde uses ese MCP.**

#### 4a. Claude Desktop (`~/Library/Application Support/Claude/claude_desktop_config.json`)

**Antes:**
```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
    }
  }
}
```

**Después:**
```json
{
  "github": {
    "command": "/Users/<tu-user>/.local/bin/mcp-github-wrapper.sh"
  }
}
```

#### 4b. Claude Code (`~/.claude.json`)

Claude Code usa la misma key `mcpServers` pero con campos adicionales (`type`, `env`). Se actualiza igual:

**Antes:**
```json
{
  "github": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
    }
  }
}
```

**Después:**
```json
{
  "github": {
    "type": "stdio",
    "command": "/Users/<tu-user>/.local/bin/mcp-github-wrapper.sh",
    "args": [],
    "env": {}
  }
}
```

Edición quirúrgica con `jq` (útil porque `~/.claude.json` es grande y tiene mucha más config que MCPs):

```bash
jq '.mcpServers.github = {
      type: "stdio",
      command: "/Users/<tu-user>/.local/bin/mcp-github-wrapper.sh",
      args: [],
      env: {}
    }' ~/.claude.json > ~/.claude.json.new && mv ~/.claude.json.new ~/.claude.json
```

> ℹ️ **Ruta absoluta obligatoria.** Los clientes MCP no expanden `~` ni `$HOME` — usá el path completo.

### 5. Pegar los tokens reales y reiniciar

1. Abrí `~/.mcp/secrets.env` en tu IDE
2. Reemplazá los `PEGA_TU_TOKEN_AQUI_*` por los valores reales (directo en el editor, **nunca por chat**)
3. Guardá el archivo
4. Reiniciá cada cliente modificado:
   - **Claude Desktop**: `Cmd+Q` → volver a abrir
   - **Claude Code**: salir de la sesión (`Cmd+D`) → `claude` de nuevo
5. Verificá que cada cliente cargue el MCP correctamente:
   - **Claude Desktop**: `Settings → Desktop App → Developer` → status `Running`
   - **Claude Code**: `claude mcp list` → aparece conectado

---

## 🔄 Flujo de rotación de secrets

Rotar un token ahora es un proceso limpio y reproducible. **Aunque uses varios clientes MCP, solo tocás un archivo** (`~/.mcp/secrets.env`):

```
1. Revocar el token viejo en el provider
   (ej: https://github.com/settings/tokens)
         │
         ▼
2. Generar un token nuevo
         │
         ▼
3. Abrir ~/.mcp/secrets.env en el IDE
   y pegar el nuevo valor (sin pasarlo por chat)
         │
         ▼
4. Reiniciar los clientes MCP que tengas activos:
   • Claude Desktop (Cmd+Q → abrir)
   • Claude Code    (salir de sesión → claude)
   • Otros clientes (según corresponda)
         │
         ▼
5. ✅ Rotación completa — ningún archivo de config tocado
```

> 🛡️ **Nunca** pegues el token nuevo en una conversación con un LLM. Pégalo directamente en tu IDE, en el archivo `~/.mcp/secrets.env`.

> 💡 **La gran ventaja de esta arquitectura:** rotás en 1 archivo y afecta a N clientes simultáneamente. Antes del pattern, rotar implicaba editar cada archivo de config de cada cliente por separado.

---

## 🚫 Anti-patrones

### ❌ NO pongas secrets directo en los archivos de config

```json
// NUNCA — aplica a claude_desktop_config.json, ~/.claude.json, config de Cursor, etc.
{
  "mcpServers": {
    "github": {
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx" }
    }
  }
}
```

Los archivos los leen agentes de IA, los sincronizan tools de backup, y rotar se vuelve fricción.

### ❌ NO olvides actualizar un cliente cuando tenés varios

Si tenés Claude Desktop **y** Claude Code (o cualquier otra combinación), ambos archivos de config tienen que apuntar al wrapper. Olvidar uno causa el síntoma clásico: **"todo funciona menos este cliente"** → errores de autenticación silenciosos difíciles de debuggear.

**Checklist post-rotación:**
- [ ] Reinicié Claude Desktop
- [ ] Reinicié Claude Code (nueva sesión)
- [ ] Validé con `claude mcp list` (Claude Code)
- [ ] Validé status `Running` en Settings → Developer (Claude Desktop)

### ❌ NO versiones `~/.mcp/secrets.env` en Git

Incluso en un repo privado. Si el repo se clona en otra máquina o se abre en un IDE con asistente, los secrets quedan en context. Agregar a `.gitignore` globalmente:

```bash
echo ".mcp/" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

### ❌ NO hardcodees secrets en los wrappers

El wrapper debe **leer** del `.env`, nunca contener el token directamente:

```bash
# MAL
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_xxxx"

# BIEN
source "$HOME/.mcp/secrets.env"
export GITHUB_PERSONAL_ACCESS_TOKEN="$GITHUB_PAT_MAIN"
```

### ❌ NO pegues tokens en chats con asistentes

Aunque sea "solo para probar". Todo mensaje queda en el context de la conversación y posiblemente en logs. Si necesitas pegarle un token a un asistente, cambia el flujo para que el asistente lo lea desde el `.env` en vez de recibirlo por mensaje.

---

## 🚀 Extensiones

### Múltiples MCPs, un solo `.env`

El archivo `~/.mcp/secrets.env` es compartido por todos los wrappers. Añadir un MCP nuevo con secrets:

1. Agregá la variable al `.env` (ej: `export ANTHROPIC_API_KEY="..."`)
2. Creá un nuevo wrapper `~/.local/bin/mcp-<nuevo>-wrapper.sh` que haga `source` del mismo `.env`
3. Registrá el wrapper en el archivo de config de cada cliente donde vayas a usarlo (Claude Desktop, Claude Code, etc.)

### Secrets compartidos con otros tools

Si querés que el mismo `~/.mcp/secrets.env` cargue en tu shell para uso fuera de Claude Desktop, agregá esto a `~/.zshrc`:

```bash
# Carga secrets de MCP si el archivo existe
[[ -f "$HOME/.mcp/secrets.env" ]] && source "$HOME/.mcp/secrets.env"
```

Así las variables están disponibles también en tu terminal interactiva.

### Manejo con password manager

Para setups más robustos, reemplazá el `.env` por una integración con `1Password CLI`, `Bitwarden CLI` o `macOS Keychain`:

```bash
# Wrapper con 1Password CLI
export GITHUB_PERSONAL_ACCESS_TOKEN="$(op read 'op://Personal/GitHub PAT/credential')"
exec npx -y @modelcontextprotocol/server-github
```

Ventaja: el secret nunca toca el filesystem en plaintext.

---

## 🔗 Referencias

- [LocalDev en el ecosistema MCP](../00-foundations/03-customize/02-conectores.md#️-localdev--el-mcp-server-que-declaras-tú) — contexto de foundations
- [MCP Specification](https://spec.modelcontextprotocol.io/) — protocolo oficial de Anthropic
