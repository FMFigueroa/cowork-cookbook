# 🧠 Skills

> Habilidades especializadas que el agente adopta para ejecutar tareas dentro de su entorno local.

**Última actualización:** 2026-04-22
**Autor:** Felix M. Figueroa · [@FMFigueroa](https://github.com/FMFigueroa)

---

## 📋 Índice

- [¿Qué es un Skill?](#-qué-es-un-skill)
- [¿Para qué sirve?](#-para-qué-sirve)
- [Origen: Built-in y Personal](#-origen-built-in-y-personal)
- [Anatomía](#-anatomía)

---

## 💡 ¿Qué es un Skill?

Un **Skill** es una **habilidad especializada** que se le suministra al agente de Cowork para ejecutar las tareas de un Project dentro de su entorno local. Funciona como una base de conocimiento autocontenida: combina instrucciones en markdown (tipo system prompt), archivos estáticos (HTML, assets, referencias) y/o código ejecutable (Python, JavaScript, bash, shell scripts) que el agente lee, consulta e invoca — todo sin salir al exterior.

## 🎯 ¿Para qué sirve?

- Le da al agente **una habilidad específica** para abordar un tipo concreto de tarea (ej. crear un `.docx`, diseñar un poster, redactar comunicaciones internas).
- Funciona como **base de conocimiento autocontenida**: combina instrucciones, scripts, plantillas, referencias y assets en una sola unidad.
- **Opera localmente**: el agente ejecuta la habilidad dentro de su entorno virtual, sin necesidad de conectarse a recursos externos.
- **Tiene dos formas de invocación**:
  - **Auto-activación** — cuando el prompt o el contexto coinciden con el trigger declarado en `description`.
  - **Invocación explícita** — escribiendo `/<nombre-del-skill>` en el prompt, opcionalmente con argumentos (ej. `/accessibility-review <Figma URL>`).

## 📚 Origen: Built-in y Personal

En el panel **Skills** de Customize los Skills aparecen agrupados en dos categorías:

### 🛠️ Built-in skills (pre-instaladas por Anthropic)

Skills nativas del entorno Cowork. Vienen incorporadas de fábrica y habilitan capacidades base del sistema. No se descargan — ya están ahí al abrir Cowork.

Las 3 actuales: `schedule`, `setup-cowork`, `context`.

### 📦 Personal skills (tu colección)

Skills que tú has incorporado a tu entorno. Tienen dos orígenes posibles:

- **Descargadas del marketplace oficial de Anthropic** — eliges cuáles instalar según las necesidades de tus Projects.
- **Creadas por ti** — Skills custom que construyes desde cero.

Ambos casos aparecen bajo la etiqueta "Personal" en la UI y quedan disponibles globalmente para todos tus Projects.

Personal skills actualmente descargadas del marketplace (10):

`algorithmic-art` · `brand-guidelines` · `canvas-design` · `doc-coauthoring` · `internal-comms` · `mcp-builder` · `skill-creator` · `slack-gif-creator` · `theme-factory` · `web-artifacts-builder`

## 🔍 Anatomía

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

### Lo único obligatorio: `SKILL.md`

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

### Lo que puede haber alrededor (varía por Skill)

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

### Rango real de complejidad

Ejemplos de Skills instalados localmente:

- **Mínimo** — `doc-coauthoring` → solo `SKILL.md`
- **Medio** — `algorithmic-art` → `SKILL.md` + `templates/` + `LICENSE.txt`
- **Rico** — `skill-creator` → `SKILL.md` + `agents/` + `assets/` + `eval-viewer/` + `references/` + `scripts/` + `LICENSE.txt`

---

## 🔗 Referencias

- [Overview de Customize](./README.md) — los 3 tipos de extensiones y por qué vienen en "segundo plano"
- [Conectores](./02-conectores.md) — el siguiente tipo de extensión
- [Plugins](./03-plugins.md) — kits curados que combinan Skills + Conectores + Subagents
