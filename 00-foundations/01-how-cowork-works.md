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
- **Se activa automáticamente** cuando el prompt o el contexto coinciden con el trigger declarado en el Skill.

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

Todo Skill tiene un archivo `SKILL.md` en la raíz. Empieza con un frontmatter YAML que Cowork usa para decidir cuándo activarlo:

```yaml
---
name: docx
description: "Use this skill whenever the user wants to create, read, edit, or manipulate Word documents..."
license: Proprietary. LICENSE.txt has complete terms
---
```

El resto del archivo es markdown con instrucciones para Claude: overview, quick reference, workflows y ejemplos de comandos.

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

Los conectores se clasifican según **dónde corren** y, dentro de los de la nube, según **quién los construye**.

#### Por ubicación

**🖥️ Locales.** MCP servers que corren en tu propia máquina. Sirven para ejecutar tareas multi-agénticas sobre archivos y sistemas locales — tu filesystem, herramientas instaladas en tu equipo, procesos propios del entorno.

**☁️ En la nube.** MCP servers remotos que corren como servicios externos y exponen aplicaciones online al agente. Son la vía para conectar Cowork con APIs, SaaS y plataformas de terceros.

#### Por origen (aplica a los de la nube)

| Modalidad     | Quién los construye      | Cómo se obtienen                                                                               |
| ------------- | ------------------------ | ---------------------------------------------------------------------------------------------- |
| **Oficiales** | Anthropic o sus partners | Se instalan desde el marketplace de Claude                                                     |
| **Custom**    | Tú mismo                 | Los desarrollas cuando no existe un conector oficial para la aplicación que necesitas integrar |

### Modelo de invocación

```
┌──────────────────────────┐
│  Aplicación / Servicio   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│  MCP server (Conector)   │   ← Local o Cloud · Oficial o Custom
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

---

## 🧩 Plugins

Un **Plugin** es un **kit curado** del marketplace de Claude que empaqueta dos tipos de extensiones bajo un mismo nombre: **Skills** (habilidades locales) y **Conectores MCP** (bridges externos). Se instala en un solo paso y entrega al agente una capacidad completa — habilidad + alcance externo — sin tener que montar cada Skill y cada Conector por separado.

**Fórmula:**

```
Plugin = Skill(s) + Conector(es) MCP
       (curado por Anthropic o partners)
```

### ¿Para qué sirve?

- **Agrupa una capacidad completa** — reúne las habilidades locales y las conexiones externas que juntas cubren un dominio de trabajo.
- **Viene curado** — la combinación de Skills + Conectores está pensada por Anthropic o sus partners para un caso de uso específico.
- **Instalación en un paso** — en vez de instalar Skills y Conectores uno por uno, un Plugin los trae empaquetados y listos para activar.

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

### Origen

Los Plugins publicados en el marketplace de Claude provienen de dos orígenes:

| Modalidad    | Quién los construye                 |
| ------------ | ----------------------------------- |
| **Oficiales**| Anthropic                           |
| **Partners** | Proveedores certificados por Anthropic |

### Cómo encajan Plugin, Skill y Conector

```
Skill          = habilidad local autocontenida
Conector MCP   = bridge externo hacia una aplicación
Plugin         = kit curado que combina Skills + Conectores
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
