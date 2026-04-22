# Pattern: MCP Secrets Management

> Cómo gestionar tokens y credenciales de MCP servers locales (`LOCAL DEV`) sin exponerlos en `claude_desktop_config.json`.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)
**Complejidad:** ⭐⭐

---

## 📋 Índice

- [Cuándo usarlo](#-cuándo-usarlo)
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
- Quieres evitar que los secrets **queden en plaintext** en `claude_desktop_config.json`.
- Necesitas un flujo limpio para **rotar secrets** sin tocar la config de Claude Desktop cada vez.
- Trabajas con asistentes de IA (Claude Code, Copilot) que podrían leer el archivo de config y dejar los tokens expuestos en logs o contextos de conversación.

---

## ⚠️ El problema

El formato estándar de registro de un MCP server `LOCAL DEV` expone los secrets directamente en el JSON:

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

### Por qué esto es problemático

| Riesgo                          | Impacto                                                                                       |
| ------------------------------- | --------------------------------------------------------------------------------------------- |
| **Lectura por agentes de IA**   | Cualquier asistente que lea `claude_desktop_config.json` arrastra el token a su context       |
| **Backups accidentales**        | Tools de sync (iCloud, Time Machine) copian el archivo con los tokens adentro                  |
| **Logs y debugging**            | `cat` del archivo en terminal queda en el scrollback o en logs de sesión                       |
| **Rotación inconveniente**      | Cada vez que rotas un token hay que editar el JSON (fricción → postergás la rotación)          |
| **Versionado imposible**        | No podés versionar el archivo en Git con tokens adentro                                       |

### Limitación técnica

> **Claude Desktop NO soporta expansión de variables de entorno en `claude_desktop_config.json`.**
> Si escribís `"${GITHUB_TOKEN}"`, lo recibe como **string literal**, no resuelve la variable. Por eso necesitás un **wrapper script** que cargue el `.env` y luego ejecute el comando real.

---

## 💡 La solución: wrapper + archivo .env

Tres capas que separan las responsabilidades:

```
┌─────────────────────────────────────────────────────────┐
│  claude_desktop_config.json                             │
│  └─ command: /Users/<user>/.local/bin/mcp-<X>-wrapper.sh│  ← no tiene secrets
└─────────────────┬───────────────────────────────────────┘
                  │ ejecuta
                  ▼
┌─────────────────────────────────────────────────────────┐
│  ~/.local/bin/mcp-<X>-wrapper.sh                        │
│  ├─ source ~/.mcp-secrets.env                           │  ← carga secrets
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

- **`claude_desktop_config.json`** solo conoce la ruta del wrapper. Sin secrets.
- **El wrapper** es el único lugar que toca el `.env` y monta los env vars en el proceso.
- **`exec`** reemplaza el proceso de bash con el MCP server, preservando el canal STDIO intacto — Claude Desktop ni se entera de la capa intermedia.

---

## 🏗️ Estructura

```
~/
├── .mcp-secrets.env                              ← secrets (chmod 600)
│
├── .local/bin/
│   ├── mcp-github-wrapper.sh                     ← wrapper (chmod 755)
│   ├── mcp-github-leonobitech-wrapper.sh         ← 1 wrapper por MCP con secrets
│   └── mcp-<otro>-wrapper.sh
│
└── Library/Application Support/Claude/
    ├── claude_desktop_config.json                ← apunta a los wrappers
    └── claude_desktop_config.json.backup-YYYY-MM-DD  ← backup antes del cambio
```

---

## 🔧 Implementación paso a paso

### 1. Hacer backup del `claude_desktop_config.json`

Antes de tocar nada, siempre backup:

```bash
cp "$HOME/Library/Application Support/Claude/claude_desktop_config.json" \
   "$HOME/Library/Application Support/Claude/claude_desktop_config.json.backup-$(date +%Y-%m-%d)"
```

### 2. Crear `~/.mcp-secrets.env`

Con **placeholders**, nunca con valores reales pegados desde chat. El archivo se edita directamente en tu IDE.

```bash
# ~/.mcp-secrets.env
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
chmod 600 ~/.mcp-secrets.env
```

### 3. Crear el wrapper

Un wrapper por MCP server que necesite secrets. Template:

```bash
#!/bin/bash
# MCP <nombre> wrapper — propósito
#
# Consumido por: ~/Library/Application Support/Claude/claude_desktop_config.json
# Claude Desktop lo ejecuta como proceso hijo al arrancar.

set -euo pipefail

# Asegurar que npx/node estén disponibles (Claude Desktop arranca con PATH limpio)
export PATH="$HOME/.local/bin:$PATH"

# Cargar secrets
if [[ ! -f "$HOME/.mcp-secrets.env" ]]; then
  echo "Error: $HOME/.mcp-secrets.env no existe" >&2
  exit 1
fi
source "$HOME/.mcp-secrets.env"

# Mapear al nombre de env var que espera el MCP server
export GITHUB_PERSONAL_ACCESS_TOKEN="$GITHUB_PAT_MAIN"

# Reemplazar este proceso con el MCP server (mantiene STDIO intacto)
exec npx -y @modelcontextprotocol/server-github
```

Permisos ejecutables:

```bash
chmod 755 ~/.local/bin/mcp-github-wrapper.sh
```

### 4. Actualizar `claude_desktop_config.json`

Reemplazar la entrada del MCP server para apuntar al wrapper:

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

> ℹ️ **Ruta absoluta obligatoria.** Claude Desktop no expande `~` ni `$HOME` — usa el path completo.

### 5. Pegar los tokens reales y reiniciar

1. Abre `~/.mcp-secrets.env` en tu IDE
2. Reemplaza los `PEGA_TU_TOKEN_AQUI_*` por los valores reales
3. Guarda el archivo
4. Reinicia Claude Desktop (Cmd+Q → volver a abrir)
5. Verifica en `Settings → Desktop App → Developer` que el MCP server aparezca con status `Running`

---

## 🔄 Flujo de rotación de secrets

Rotar un token ahora es un proceso limpio y reproducible:

```
1. Revocar el token viejo en el provider
   (ej: https://github.com/settings/tokens)
         │
         ▼
2. Generar un token nuevo
         │
         ▼
3. Abrir ~/.mcp-secrets.env en el IDE
   y pegar el nuevo valor (sin pasarlo por chat)
         │
         ▼
4. Reiniciar Claude Desktop
         │
         ▼
5. ✅ Rotación completa — claude_desktop_config.json intacto
```

> 🛡️ **Nunca** pegues el token nuevo en una conversación con un LLM. Pégalo directamente en tu IDE, en el archivo `.mcp-secrets.env`.

---

## 🚫 Anti-patrones

### ❌ NO pongas secrets directo en `claude_desktop_config.json`

```json
// NUNCA
{
  "mcpServers": {
    "github": {
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx" }
    }
  }
}
```

El archivo lo leen agentes de IA, lo sincronizan tools de backup, y rotar se vuelve fricción.

### ❌ NO versiones `~/.mcp-secrets.env` en Git

Incluso en un repo privado. Si el repo se clona en otra máquina o se abre en un IDE con asistente, los secrets quedan en context. Agregar a `.gitignore` globalmente:

```bash
echo ".mcp-secrets.env" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

### ❌ NO hardcodees secrets en los wrappers

El wrapper debe **leer** del `.env`, nunca contener el token directamente:

```bash
# MAL
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_xxxx"

# BIEN
source "$HOME/.mcp-secrets.env"
export GITHUB_PERSONAL_ACCESS_TOKEN="$GITHUB_PAT_MAIN"
```

### ❌ NO pegues tokens en chats con asistentes

Aunque sea "solo para probar". Todo mensaje queda en el context de la conversación y posiblemente en logs. Si necesitas pegarle un token a un asistente, cambia el flujo para que el asistente lo lea desde el `.env` en vez de recibirlo por mensaje.

---

## 🚀 Extensiones

### Múltiples MCPs, un solo `.env`

El archivo `~/.mcp-secrets.env` es compartido por todos los wrappers. Añadir un MCP nuevo con secrets:

1. Agrega la variable al `.env` (ej: `export ANTHROPIC_API_KEY="..."`)
2. Crea un nuevo wrapper `~/.local/bin/mcp-<nuevo>-wrapper.sh` que haga `source` del mismo `.env`
3. Registra el wrapper en `claude_desktop_config.json`

### Secrets compartidos con otros tools

Si querés que el mismo `.mcp-secrets.env` cargue en tu shell para uso fuera de Claude Desktop, agregá esto a `~/.zshrc`:

```bash
# Carga secrets de MCP si el archivo existe
[[ -f "$HOME/.mcp-secrets.env" ]] && source "$HOME/.mcp-secrets.env"
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

- [LocalDev en el ecosistema MCP](../00-foundations/01-how-cowork-works.md#️-localdev--el-mcp-server-que-declaras-tú) — contexto de foundations
- [MCP Specification](https://spec.modelcontextprotocol.io/) — protocolo oficial de Anthropic
